---
title: "The Migrating Bottleneck"
date: 2026-07-28
tags: [swamp, dynamodb, datastore, performance, aws]
description: "Four fixes into a DynamoDB sync backend, the ceiling kept moving: from a hot GSI partition to lock contention to catalog metadata no datastore could outrun."
---

I opened issue #265 against my own extension after a load test told me something I didn't want to hear. Ten workers, seven hours, and the sync rate had fallen by a factor of ten: throughput workers down from 0.10 iterations per second to 0.01, write-stress workers (one model, continuous writes) failing every single iteration once they passed number two hundred.

The extension is a DynamoDB backend for [swamp](https://swamp-club.com)'s datastore layer, and the datastore layer treats storage as a path-addressable file tree: every model's data lives at `data/{model-type}/{model-id}/{data-name}/{version}/raw`, with a metadata file alongside it. Sync follows a fixed sequence: acquire a lock, pull changed paths from remote, do the work locally, push the dirty paths back, commit an index update, release the lock. Whatever backend serves this sequence, S3, DynamoDB, or something else, sees whole-file transfers. It never sees an individual field write.

That contract is deliberate. Anything that can store a blob by key, do a conditional write for locking, and list by prefix qualifies as a backend. It also means DynamoDB has to earn its place in a shape it wasn't built for: it's a key-value database with query capabilities, not an object store, and a filesystem-shaped sync layer built on top of it lives or dies on key design, not on how much you're willing to pay for capacity.

## Why a partition key can quietly become a ceiling

A DynamoDB table has one primary key and, optionally, a handful of secondary indexes. A Global Secondary Index (GSI) is a live, re-keyed copy of your table's items around a different partition key than the base table uses. DynamoDB spreads a table's items across physical partitions by hashing the partition key, and it allocates throughput per partition, not per table. A partition key with many distinct values spreads load across many partitions. A partition key with one value puts every item on the same partition, and that partition has a ceiling around 3,000 read capacity units regardless of how much on-demand capacity you've provisioned for the table as a whole.

My original GSI used exactly one value:

```
gsi1pk = "FILE"  # every item, every model, one partition
gsi1sk = relPath  # e.g. "data/command/shell/<uuid>/result/42"
```

A single query, `gsi1pk = "FILE"`, returned every file metadata item that had ever existed, and the "what changed since my last sync" logic filtered by timestamp client-side, after DynamoDB had already charged for reading and returning the whole set. At a hundred items nobody notices. At fifteen thousand, every sync reads all fifteen thousand to find the handful that actually changed.

## The first bottleneck: a full scan on every sync

The bug report laid out the evidence plainly:

| Time | Iterations | Rate | Error Rate |
|------|-----------|------|------------|
| 0-1h | 0-70 | 0.10/s | 0% |
| 1-3h | 70-150 | 0.05/s | 0% |
| 3-5h | 150-250 | 0.02/s | ~10% |
| 5-7h | 250-380 | 0.01/s | ~20% |

`queryAllFileMeta()` was the culprit, and it read exactly the way the name suggests:

```typescript
async function queryAllFileMeta(prefix?: string): Promise<Map<string, RemoteFileMeta>> {
  // Queries GSI where gsi1pk = "FILE": returns ALL file metadata items
  // Paginates through entire result set
  // Then client-side filters by updatedAt > lastPulledAt
}
```

Every sync pull, and every sync push once the local dirty-path cache overflowed, paid for the entire GSI regardless of how little had actually changed. I sketched three ways out in the issue: add a server-side filter expression on `updatedAt` (cheap, but DynamoDB still reads every item to evaluate the filter), restructure the GSI around timestamp buckets, or give each model its own partition. I recommended shipping the filter expression first as a quick patch and treating a real redesign as a follow-up. What actually shipped, a day later, skipped straight to the redesign.

## The fix: one partition per model, time as part of the key

```
gsi1pk = "FILE#<modelType>/<modelId>"   # one partition per model
gsi1sk = "<updatedAt>|<relPath>"         # timestamp first, sortable
```

Three things fall out of this key shape. Each model's data lives in its own physical partition, so no single model's write volume can starve another's reads. A query like `gsi1sk > :lastSyncTimestamp` is a key condition, not a filter, so DynamoDB only reads the items that actually changed instead of reading everything and discarding most of it. And querying across many models becomes many independent partition queries that DynamoDB serves from different physical nodes at once, instead of one query serialized against a single hot shard.

That schema change needed supporting pieces. A rare full sync (nobody has a `lastSyncTimestamp` to scope against yet) has to discover every partition that exists, so a small registry item tracks known partition keys, updated with an atomic `UpdateItem ADD` on a string set rather than a read-modify-write, so two workers registering partitions at the same moment can't stomp on each other. A scoped push, where the caller already knows which paths are dirty, skips the GSI entirely and does direct primary-key lookups in batches. And the sync watermark, "everything has been pushed as of time T", only gets to advance on an unscoped pull; a scoped pull for one model has no way to vouch for every other model, so letting it advance the watermark would poison every other worker's next full sync.

That's the ordinary cost of making a general contract, "here's a file tree, sync it," work well against a database that rewards specific access patterns instead of generic ones.

## Testing the fix reveals the second bottleneck

The fix worked, and the first test run made that oddly hard to see. Ten workers, the redesigned GSI live, and the floor rate settled at 0.006 to 0.008 iterations per second, with 47 to 74 percent of iterations failing. Next to the pre-fix numbers, that doesn't read like a win.

The shape of the curve is what proves it is one. The old bug degraded continuously as items accumulated: 0.10 down to 0.01 over seven hours, a curve that keeps falling as long as the table grows. The new floor is flat. It reaches roughly the same low rate within about three hours and stays there, whether the table holds fifty thousand items or a hundred thousand. Every timeout in the run traced back to the same message: `Lock timeout after 60000ms on key: dynamodb://bench-ddb-gsi-perf`. One lock item, ten workers queued behind it, a 30-second TTL and a 60-second wait that the queue depth blew past on every single cycle.

Doubling the fleet to twenty workers in a second run confirmed it rather than making things worse: the floor rate held at almost exactly the same 0.007 iterations per second, with error rates a little higher. If the bottleneck were DynamoDB capacity or GSI query cost, twice the workers should have meant twice the pressure on the database. It didn't, because the lock queue saturates fast and everything past that point just fails faster. The GSI redesign had done its job. The datastore-level lock, one lock per table, acquired around every sync regardless of which model it touched, was now the only thing serializing ten workers who never touched each other's data in the first place.

## The fix: namespaces, and a bottleneck that stayed hidden behind it

Swamp's datastore layer supports a namespace setting, and namespacing does something specific: it scopes the lock key to the namespace, not just the data. Give each worker its own namespace and each worker gets its own lock. Twenty workers, twenty lock keys, confirmed by scanning the table directly, zero cross-worker contention, lock acquisition times under a millisecond when uncontended.

At launch, that showed up exactly as expected. Zero lock timeouts. Iteration rates of 0.12 to 0.18 per second per worker, a full order of magnitude above the shared-lock floor.

Four hours in, the rate had fallen anyway, to 0.02 iterations per second, with error rates climbing back toward ten percent, despite zero lock contention. The namespace fix was correct. It had also uncovered a bottleneck the lock had been hiding: every sync commits an index update, and that commit rewrites every file entry in the namespace's index, not just the ones that changed. At iteration ten, that's ninety files. At iteration three hundred fifty, it's three thousand five hundred, and rewriting three thousand five hundred index entries on every single sync is not a cheap step to repeat every few seconds.

## Disposable models, and the bottleneck underneath that one

The fix here didn't touch DynamoDB at all. It changed what the workload asked the datastore to track. Instead of one long-lived model per worker accumulating data across every iteration, each iteration now creates a fresh, disposable model instance, runs one method against it, and deletes it:

```bash
INSTANCE="w${WORKER_ID}-i${ITERATION}"
swamp model @webframp/system method run "$METHOD" "$INSTANCE" --skip-checks --skip-reports --json
swamp model delete "$INSTANCE" --force
```

Committed files per sync dropped to six and stayed at six, whether the worker was on iteration ten or iteration six hundred. Scaled up to a hundred concurrent workers, the fleet sustained roughly 17 iterations per second in aggregate, and this is where DynamoDB's own numbers are worth pausing on: 17,500 write capacity units consumed per second, average `PutItem` latency of 3.7 milliseconds, write throttling under 0.2 percent of operations, zero system errors. At the throughput and latency layer, DynamoDB did exactly what a managed database under real load is supposed to do.

And the rate still crept down, from around 0.20 to 0.35 iterations per second at launch to 0.13 to 0.22 by iteration six hundred sixty, because `model delete` removes the model's definition but leaves its data entries in the catalog, the index of everything that has ever existed in the namespace. Deleting a model doesn't delete its history. Over nine and a half hours, that catalog grew past 4,000 rows, and the export of it, required on every sync for cross-namespace visibility, grew right along with it.

## The fix that actually held: pruning

`swamp data prune --force` removes orphaned data left behind by deleted models, and adding it to the loop bounded the one metric nothing else had touched:

| Prune frequency | Catalog at hour 3 | Iterations/s at hour 3 |
|-----------------|-------------------|------------------------|
| Never | 1,500+ rows | 0.10 |
| Every 20 iterations | 300-500 rows | 0.13 |
| Every iteration | 1-8 rows | 0.12-0.15 |

Pruned every iteration, the catalog converged to single digits and the rate held there instead of continuing to fall. Everything upstream of this, the GSI, the namespaced locks, the disposable models, was necessary and none of it was sufficient on its own. The dominant cost had moved somewhere none of those fixes reached: metadata that outlives the resources it describes.

`--force` deletes data permanently, and pruning every single iteration only made sense here because I already knew every entry the benchmark created was disposable the moment its worker deleted it. That's not a production recommendation. If a catalog is growing because real resources with real lifecycles are accumulating in it, the fix is lifecycle management for those resources: expire what's actually done, keep what's still live. Running `prune --force` on a schedule against data you haven't confirmed is safe to lose is a good way to find out the hard way which entries mattered. I reached for it here to characterize how the extension behaves under a workload built to grow catalog metadata fast, not to suggest aggressive pruning is how you buy back performance in general.

## Proving it wasn't DynamoDB's fault

By this point I'd fixed four things in the DynamoDB extension and the sync coordinator that sits above it, and I wanted to know whether the fourth bottleneck was a DynamoDB problem in disguise. I ran the identical workload against an S3-backed datastore, which indexes with a completely different mechanism, shard-first manifest files instead of a GSI, and watched for the same curve.

I got it. Catalog rows grew at roughly the same rate per iteration on S3 as on DynamoDB. Iteration throughput fell from 0.11-0.16 to 0.02-0.03 per second over the same run, and pruning converged the catalog to one row the same way it had on DynamoDB. What didn't match was the absolute rate: DynamoDB's `PutItem` averages 3.7 milliseconds, while S3's `PutObject` through a VPC endpoint runs closer to three hundred, and a sync operation makes several of those round trips (lock, sync, execute, commit, delete, prune). DynamoDB workers hit a higher ceiling at the same catalog size because each operation costs less, not because the catalog problem is any smaller.

That's the finding that mattered most out of everything in this series. The degradation lives in the sync coordinator's catalog lifecycle, above both backends, not inside either one's indexing strategy. I could have spent another month re-tuning GSI partition boundaries or DynamoDB capacity modes and never touched the actual ceiling.

## Where DynamoDB earned its keep

The parts of this contract that map awkwardly onto DynamoDB, path-to-key translation, prefix listing recast as a query, and a manifest write's expansion into many index item writes, needed real key design instead of a naive port from an object store, and that's exactly what happened. The part of the contract that matters most for correctness, the lock, is stronger on DynamoDB than anything S3's conditional PUT offers. Fencing-token nonces, written with a conditional `PutItem`, give compare-and-swap safety on every lock renewal; a native TTL attribute is a backstop for a crashed holder, not the mechanism doing the work. A stale lock holder's write gets rejected on the next conditional check, TTL or no TTL. That combination, plus sub-4-millisecond writes at 560 operations a second sustained across a hundred workers, is the reason none of the first three fixes ever needed to touch DynamoDB's own performance. It had headroom the whole time, and the ceiling was always somewhere else.

## What generalizes

A GSI partition key that takes on one value for every item is a hot partition waiting for enough load to expose it, and it will look fine in every test you run below that load. Moving a filter into the key condition, rather than applying it after the read, is the difference between DynamoDB charging you for data it discards and DynamoDB doing the discarding for you. Fixing a bottleneck proves the fix is correct at its layer. It doesn't prove the system is fixed, because the next constraint up the stack was there all along, just hidden behind the one you found first. And metadata that outlives the resource it describes is a performance bug with a very long fuse: it costs nothing for the first hour and something real by the ninth, which is exactly why an hour-long benchmark would have told me the system was fine.

Four fixes, four ceilings, and a fifth test that told me the last one wasn't about DynamoDB at all: every layer I fixed was genuinely fixed, and every one of them was standing in front of another.
