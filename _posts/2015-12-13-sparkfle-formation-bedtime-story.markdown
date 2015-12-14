---
layout: post
title: "SparkleFormation Bedtime Story"
date: 2015-12-13 20:42
comments: true
categories: aws, cloudformation
---

#A very brief whirlwind tour of sparkles

or: How I learned to stop writing terrible [serialization](https://aws.amazon.com/cloudformation/aws-cloudformation-templates/) formats [directly](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/working-with-templates-cfn-designer-json-editor.html) and [love the dsl](http://www.sparkleformation.io/)

An imaginary story of the beginner exploring some infrastructure tooling.

## Getting started

Set env variables. Maybe create an init.sh while practicing.

``` sh
export AWS_ACCESS_KEY_ID="your key"
export AWS_SECRET_ACCESS_KEY="your sekret"
export AWS_REGION="us-east-1" # because YOLO
export NESTED_BUCKET="s3://mah_bukkit"
```

load that stuff into my current shell

`source ./init.sh`

### Create a config

`.sfn` file aready exists, thanks [@luckymike](https://github.com/luckymike). no need to create one.

## Let's make a VPC!

Initial creation results

``` sh
~/s/sparkleformation-workshops ❯❯❯ be sfn create training-vpc --file sparkleformation/vpc.rb
```

... (a bunch of cloudformation stack output)
... (like really a lot)

The important stuff:

``` sh
[Sfn]: Stack create complete: SUCCESS
[Sfn]: Stack description of training-vpc:
[Sfn]: Outputs for stack: training-vpc
[Sfn]:    Vpc Id: vpc-62950a06
[Sfn]:    Vpc Cidr: 10.0.0.0/16
[Sfn]:    Public Route Table: rtb-f45b0f90
[Sfn]:    Private Route Table: rtb-e95b0f8d
[Sfn]:    Internet Gateway: igw-625db206
[Sfn]:    Public Us East1a Subnet: subnet-bb0be8e3
[Sfn]:    Public Us East1c Subnet: subnet-ce829fe5
[Sfn]:    Public Us East1d Subnet: subnet-07ba7271
[Sfn]:    Public Us East1e Subnet: subnet-27fc781a
```

OK, lets go with that, at least I've got something. *Note to self:* Save that, you're going to need it later.

Let's update it to add an ec2 instance (Update plans are amaze!)

``` sh
~/s/sparkleformation-workshops ❯❯❯ be sfn update training-vpc --file sparkleformation/example.rb
[Sfn]: SparkleFormation: update
[Sfn]:   -> Name: training-vpc Path: /Users/sme/src/sparkleformation-workshops/sparkleformation/example.rb
[Sfn]: Stack runtime parameters:
[Sfn]: Stack Creator [sme]:
[Sfn]: Github User:
[ERROR]: Please provide a valid value
[Sfn]: Github User: webframp
[Sfn]: Hello World: How do you do!
[Sfn]: Vpc Id: vpc-62950a06
[Sfn]: Public Us East1a Subnet: subnet-bb0be8e3
[Sfn]: Pre-update resource planning report:

Update plan for: training-vpc
Resources to be removed:
[AWS::EC2::DHCPOptions]                    DhcpOptions
[AWS::EC2::InternetGateway]                InternetGateway
[AWS::EC2::VPCGatewayAttachment]           InternetGatewayAttachment
[AWS::EC2::RouteTable]                     PrivateRouteTable
[AWS::EC2::RouteTable]                     PublicRouteTable
[AWS::EC2::Route]                          PublicSubnetInternetRoute
[AWS::EC2::Subnet]                         PublicUsEast1aSubnet
[AWS::EC2::SubnetRouteTableAssociation]    PublicUsEast1aSubnetRouteTableAssociation
[AWS::EC2::Subnet]                         PublicUsEast1cSubnet
[AWS::EC2::SubnetRouteTableAssociation]    PublicUsEast1cSubnetRouteTableAssociation
[AWS::EC2::Subnet]                         PublicUsEast1dSubnet
[AWS::EC2::SubnetRouteTableAssociation]    PublicUsEast1dSubnetRouteTableAssociation
[AWS::EC2::Subnet]                         PublicUsEast1eSubnet
[AWS::EC2::SubnetRouteTableAssociation]    PublicUsEast1eSubnetRouteTableAssociation
[AWS::EC2::VPC]                            Vpc
[AWS::EC2::VPCDHCPOptionsAssociation]      VpcDhcpOptionsAssociation

Resources to be added:
[AWS::IAM::AccessKey]               CfnKeys
[AWS::IAM::User]                    CfnUser
[AWS::EC2::SecurityGroupEgress]     ExampleAllSecurityGroupEgress
[AWS::EC2::Instance]                ExampleEc2Instance
[AWS::EC2::SecurityGroupIngress]    ExampleHttpSecurityGroupIngress
[AWS::EC2::SecurityGroup]           ExampleSecurityGroup
[AWS::EC2::SecurityGroupIngress]    ExampleSshSecurityGroupIngress

[Sfn]: No resources life cycle changes detected in this update!
[Sfn]: Apply this stack update? (Y/N): Y
```

`sfn update` output:

``` sh
Time                      Resource Logical Id                         Resource Status      Resource Status Reason
2015-12-12 02:35:36 UTC   training-vpc                                CREATE_IN_PROGRESS   User Initiated
2015-12-12 02:35:46 UTC   DhcpOptions                                 CREATE_IN_PROGRESS
2015-12-12 02:35:46 UTC   InternetGateway                             CREATE_IN_PROGRESS
2015-12-12 02:35:46 UTC   Vpc                                         CREATE_IN_PROGRESS
2015-12-12 02:35:47 UTC   DhcpOptions                                 CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:35:47 UTC   InternetGateway                             CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:35:48 UTC   Vpc                                         CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:36:05 UTC   InternetGateway                             CREATE_COMPLETE
2015-12-12 02:36:05 UTC   Vpc                                         CREATE_COMPLETE
2015-12-12 02:36:06 UTC   DhcpOptions                                 CREATE_COMPLETE
2015-12-12 02:36:06 UTC   PublicUsEast1aSubnet                        CREATE_IN_PROGRESS
2015-12-12 02:36:06 UTC   PrivateRouteTable                           CREATE_IN_PROGRESS
2015-12-12 02:36:07 UTC   PublicUsEast1dSubnet                        CREATE_IN_PROGRESS
2015-12-12 02:36:07 UTC   PublicUsEast1dSubnet                        CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:36:07 UTC   PrivateRouteTable                           CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:36:08 UTC   PublicUsEast1aSubnet                        CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:36:08 UTC   InternetGatewayAttachment                   CREATE_IN_PROGRESS
2015-12-12 02:36:08 UTC   PublicUsEast1eSubnet                        CREATE_IN_PROGRESS
2015-12-12 02:36:08 UTC   PublicUsEast1cSubnet                        CREATE_IN_PROGRESS
2015-12-12 02:36:08 UTC   PrivateRouteTable                           CREATE_COMPLETE
2015-12-12 02:36:08 UTC   PublicRouteTable                            CREATE_IN_PROGRESS
2015-12-12 02:36:08 UTC   InternetGatewayAttachment                   CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:36:09 UTC   VpcDhcpOptionsAssociation                   CREATE_IN_PROGRESS
2015-12-12 02:36:09 UTC   VpcDhcpOptionsAssociation                   CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:36:09 UTC   PublicRouteTable                            CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:36:10 UTC   PublicUsEast1cSubnet                        CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:36:10 UTC   PublicUsEast1eSubnet                        CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:36:10 UTC   VpcDhcpOptionsAssociation                   CREATE_COMPLETE
2015-12-12 02:36:11 UTC   PublicRouteTable                            CREATE_COMPLETE
2015-12-12 02:36:14 UTC   PublicSubnetInternetRoute                   CREATE_IN_PROGRESS
2015-12-12 02:36:15 UTC   PublicSubnetInternetRoute                   CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:36:24 UTC   PublicUsEast1aSubnet                        CREATE_COMPLETE
2015-12-12 02:36:24 UTC   InternetGatewayAttachment                   CREATE_COMPLETE
2015-12-12 02:36:25 UTC   PublicUsEast1dSubnet                        CREATE_COMPLETE
2015-12-12 02:36:26 UTC   PublicUsEast1aSubnetRouteTableAssociation   CREATE_IN_PROGRESS
2015-12-12 02:36:27 UTC   PublicUsEast1aSubnetRouteTableAssociation   CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:36:28 UTC   PublicUsEast1dSubnetRouteTableAssociation   CREATE_IN_PROGRESS
2015-12-12 02:36:28 UTC   PublicUsEast1eSubnet                        CREATE_COMPLETE
2015-12-12 02:36:29 UTC   PublicUsEast1cSubnet                        CREATE_COMPLETE
2015-12-12 02:36:29 UTC   PublicUsEast1dSubnetRouteTableAssociation   CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:36:31 UTC   PublicUsEast1eSubnetRouteTableAssociation   CREATE_IN_PROGRESS
2015-12-12 02:36:31 UTC   PublicSubnetInternetRoute                   CREATE_COMPLETE
2015-12-12 02:36:31 UTC   PublicUsEast1cSubnetRouteTableAssociation   CREATE_IN_PROGRESS
2015-12-12 02:36:32 UTC   PublicUsEast1cSubnetRouteTableAssociation   CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:36:32 UTC   PublicUsEast1eSubnetRouteTableAssociation   CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 02:36:43 UTC   PublicUsEast1aSubnetRouteTableAssociation   CREATE_COMPLETE
2015-12-12 02:36:45 UTC   PublicUsEast1dSubnetRouteTableAssociation   CREATE_COMPLETE
2015-12-12 02:36:48 UTC   PublicUsEast1eSubnetRouteTableAssociation   CREATE_COMPLETE
2015-12-12 02:36:48 UTC   PublicUsEast1cSubnetRouteTableAssociation   CREATE_COMPLETE
2015-12-12 02:36:50 UTC   training-vpc                                CREATE_COMPLETE
2015-12-12 03:01:43 UTC   training-vpc                                UPDATE_IN_PROGRESS   User Initiated
2015-12-12 03:01:59 UTC   CfnUser                                     CREATE_IN_PROGRESS
2015-12-12 03:01:59 UTC   ExampleSecurityGroup                        CREATE_IN_PROGRESS
2015-12-12 03:02:00 UTC   CfnUser                                     CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 03:02:01 UTC   CfnUser                                     CREATE_COMPLETE
2015-12-12 03:02:04 UTC   CfnKeys                                     CREATE_IN_PROGRESS
2015-12-12 03:02:05 UTC   CfnKeys                                     CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 03:02:06 UTC   CfnKeys                                     CREATE_COMPLETE
2015-12-12 03:02:16 UTC   ExampleSecurityGroup                        CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 03:02:16 UTC   ExampleSecurityGroup                        CREATE_COMPLETE
2015-12-12 03:02:18 UTC   ExampleAllSecurityGroupEgress               CREATE_IN_PROGRESS
2015-12-12 03:02:18 UTC   ExampleSshSecurityGroupIngress              CREATE_IN_PROGRESS
2015-12-12 03:02:18 UTC   ExampleEc2Instance                          CREATE_IN_PROGRESS
2015-12-12 03:02:19 UTC   ExampleHttpSecurityGroupIngress             CREATE_IN_PROGRESS
2015-12-12 03:02:19 UTC   ExampleSshSecurityGroupIngress              CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 03:02:19 UTC   ExampleHttpSecurityGroupIngress             CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 03:02:19 UTC   ExampleAllSecurityGroupEgress               CREATE_IN_PROGRESS   Resource creation Initiated
2015-12-12 03:02:19 UTC   ExampleAllSecurityGroupEgress               CREATE_IN_PROGRESS            Resource creation Initiated
2015-12-12 03:02:19 UTC   ExampleHttpSecurityGroupIngress             CREATE_COMPLETE
2015-12-12 03:02:20 UTC   ExampleEc2Instance                          CREATE_FAILED                 Invalid availability zone: [us-west-2a]
2015-12-12 03:02:20 UTC   ExampleAllSecurityGroupEgress               CREATE_COMPLETE
2015-12-12 03:02:23 UTC   training-vpc                                UPDATE_ROLLBACK_IN_PROGRESS   The following resource(s) failed to create: [ExampleEc2Instance].
2015-12-12 03:02:33 UTC   training-vpc                                UPDATE_ROLLBACK_COMPLETE_CLEANUP_IN_PROGRESS
2015-12-12 03:02:35 UTC   ExampleHttpSecurityGroupIngress             DELETE_IN_PROGRESS
2015-12-12 03:02:36 UTC   ExampleEc2Instance                          DELETE_COMPLETE
2015-12-12 03:02:36 UTC   ExampleSshSecurityGroupIngress              DELETE_IN_PROGRESS
2015-12-12 03:02:36 UTC   ExampleAllSecurityGroupEgress               DELETE_IN_PROGRESS
2015-12-12 03:02:37 UTC   ExampleSshSecurityGroupIngress              DELETE_COMPLETE
2015-12-12 03:02:37 UTC   ExampleHttpSecurityGroupIngress             DELETE_COMPLETE
2015-12-12 03:02:39 UTC   ExampleAllSecurityGroupEgress               DELETE_COMPLETE
2015-12-12 03:02:39 UTC   CfnKeys                                     DELETE_IN_PROGRESS
2015-12-12 03:02:40 UTC   CfnKeys                                     DELETE_COMPLETE
2015-12-12 03:02:41 UTC   ExampleSecurityGroup                        DELETE_IN_PROGRESS
2015-12-12 03:02:42 UTC   CfnUser                                     DELETE_IN_PROGRESS
2015-12-12 03:02:42 UTC   ExampleSecurityGroup                        DELETE_COMPLETE
2015-12-12 03:02:43 UTC   CfnUser                                     DELETE_COMPLETE
2015-12-12 03:02:45 UTC   training-vpc                                UPDATE_ROLLBACK_COMPLETE
[FATAL]: Update of stack training-vpc: FAILED
ERROR: RuntimeError:

```

FAIL! Why is it trying to `us-west-2a` for ec2 instance when I clearly set us-east-1 ?

oh, because example.rb is using hardcoded us-west-2a...

let's change that, `emacs sparkleformation/example.rb`

``` diff
diff --git a/sparkleformation/example.rb b/sparkleformation/example.rb
index f0a8155..fd89915 100644
--- a/sparkleformation/example.rb
+++ b/sparkleformation/example.rb
@@ -1,7 +1,7 @@
     example_ec2_instance do
       type 'AWS::EC2::Instance'
       properties do
-        availability_zone 'us-west-2a'
+        availability_zone zone
```

That should work.
Now update again: `be sfn update training-vpc --file sparkleformation/example.rb`

now what?!

``` sh
2015-12-12 03:15:40 UTC   ExampleEc2Instance                          CREATE_FAILED                                  The image id '[ami-e5b8b4d5]' does not exist
```

oh. *This is not the ami you are looking for.* Every. Time.

Why is this part of ec2 still such a PITA?

(fiercly googling "e5b8b4d5")
[...](http://cloud-images.ubuntu.com/rss/trusty-release.xml)
[....](http://cloud-images.ubuntu.com/releases/trusty/release-20150724/)

Well hello, Ubuntu 14.04.2 LTS (Trusty Tahr) 64bit ebs

Next stop, [Amazon EC2 AMI Locator](https://cloud-images.ubuntu.com/locator/ec2/), searching `us-east-1`.
[Gotcha!](https://console.aws.amazon.com/ec2/home?region=us-east-1#LaunchInstanceWizard:ami=ami-7388cd19) you sneaky `ami-7388cd19`

Sure, in a perfect world I could just fire up `pry` and do this kind of junk:

``` rb
~/s/sparkle_formation ❯❯❯ pry
[1] pry(main)> require 'aws-sdk-core'
=> true
[2] pry(main)> ec2 = ::Aws::EC2::Client.new
=> #<Aws::EC2::Client>
[3] pry(main)> ec2.what? # I have no idea.
```

But we live in the real world of hostile amis that don't love you. You're on your own.
Find it yourself and edit that file.
(but at least we have the comfort of `.tuesday?` and `.saturday?` methods in `::Aws::EC2::Client`, right?)

Now, let's try that `update` again.

``` sh
~/s/sparkleformation-workshops ❯❯❯ be sfn update training-vpc --file sparkleformation/example.rb
```

(lots of cloudformation output)

But this looks good, right?

``` sh
2015-12-12 03:56:35 UTC   ExampleEc2Instance                          CREATE_IN_PROGRESS                             Received SUCCESS signal with UniqueId i-1dc9f7ad
2015-12-12 03:56:38 UTC   ExampleEc2Instance                          CREATE_COMPLETE
```

yea! maybe there's an instance somewhere in mah clouds now? ok, I guess I'll let this finish...

``` sh
2015-12-12 04:00:06 UTC   InternetGatewayAttachment                   DELETE_FAILED                                  Network vpc-62950a06 has some mapped public address(es). Please unmap
those public address(es) before detaching the gateway.
2015-12-12 04:00:13 UTC   InternetGateway                             DELETE_IN_PROGRESS
2015-12-12 04:01:03 UTC   PublicUsEast1aSubnet                        DELETE_FAILED                                  The subnet 'subnet-bb0be8e3' has dependencies and cannot be deleted.
2015-12-12 04:01:06 UTC   Vpc                                         DELETE_IN_PROGRESS
```

huh?! wait, what!!? why am I deleting this vpc and testing instances? I didn't ask for that but guess I'll wait and see what happens.

``` sh
2015-12-12 04:04:24 UTC   Vpc                                         DELETE_FAILED                                  The vpc 'vpc-62950a06' has dependencies and cannot be delete
```

Is that good or bad? I didn't ask you to delete, but maybe you are trying to save me from myself.
I give up, it's time for bed.

``` sh
bundle exec destroy training-vpc
```

Footnote:
In all seriousness, none of this is ever easy and all software is terrible. The alternatives are far, far worse and you should just be using [SparkleFormation](http://www.sparkleformation.io/)
