---
layout: post
title: "Growing with the SparkleFormation registry"
date: 2016-06-03 22:20
comments: true
categories: aws cloudformation
---

# Growing into SparkleFormation 

In my continued usage with [SparkleFormation](http://www.sparkleformation.io/)
I'm really growing to appreciate the convenience of having ruby available when
composing templates. Things that would be challenging in a simplistic
serialization format or lead to unmanageable duplication become easily solvable
thanks to the powerful combination of an actual programming language, which json
and yml are not, and the simplicity of the SparkleFormation DSL.

At it's simplest you can mirror the structure of any cloudformation json (and
finally have comments inline!) but you quickly discover more advanced use cases.

# SparkleFormation registries

Registries are described as: 

> "lightweight dynamics that are useful for storing items that may be used in multiple locations. "

A **dynamic** in SparkleFormation is just a reusable block of code that
generates some section of the template and can be very flexible. This is in
comparison to `components` which are meant as single use items to insert a
static block of code.

Registries and dynamics are similar in that they are just building blocks for
composing the template elements for any declarative orchestration API such as
CloudFormation or Azure Resource Manager, well any arbitrary json data structure
really. Registries should be simpler and more static than a full fledged
dynamic, useful for things like a list of AWS instance sizes as in the
documentation:

{% highlight ruby %}

SfnRegistry.register(:instance_size_default){ 'm3.medium' }

{% endhighlight %}

or even something a little more involved like shared `AWS::CloudFormation::Init`
data.

# A more dynamic registry for AWS cfn-init

Recently the need arose to provide a shared registry for the necessary init
commands to bootstrap a new ec2 instance with `chef-client`. Since we deal with
both Windows and linux nodes in ec2 it would be a bit annoying to have to
declare different registries like

{% highlight ruby %}

registry!(:windows_chef_client)
registry!(:linux_chef_client)

{% endhighlight %}

After a brief discussion on freenode irc `#sparkleformation`,
[@luckymike](https://twitter.com/luckymike) pointed me in the direction of a
technique he discovered that took advantage of the
[cfn-init configsets](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-init.html#aws-resource-init-configsets)
feature and the fact that registries can take args, just like any other
SparkleFormation dynamic.

Essentially, cloudformation init will look for an array of config names and run
them in order. This can be used in ruby to create an empty array called
`default` and then conditionally append to the array based on parameters passed
to the registry dynamic.

{% highlight ruby %}

SfnRegistry.register(:chef_client) do | _config = {} |
  metadata('AWS::CloudFormation::Init') do
    _camel_keys_set(:auto_disable)
    # take advantage of fact that cfn-init runs the `default`
    # configsets http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-init.html
    configSets do |sets|
      if _config.fetch(:platform) == 'windows'
        set.default += ["windows_config"]
      else
        set.default += ["default_config"]
      end
    end

    # Windows init config
    windows_config do
        # windows specific elements here
    end
    default_config do
        # generic elements here
    end

{% endhighlight %}

Clearly some of the code has been left out. With a fully fleshed out dynamic
using this approach it can then be used in a template by passing `:platform` as
a configuration element:

{% highlight ruby %}

registry!(:chef_client, :platform => 'windows')

{% endhighlight %}

The intent of this reads much clearer in the template code and everything needed
for the `:chef_client` registry is contained in the same block of code.

Another good example of this can be seen in the
[Sensu evaluation stack](https://github.com/sensu/sensu-eval-stack) codebase.
The RabbitMQ
[registry](https://github.com/sensu/sensu-eval-stack/blob/master/sparkleformation/registry/rabbitmq.rb#L4)
needed a
[generated password](https://github.com/sensu/sensu-eval-stack/blob/master/sparkleformation/sensu_stack.rb#L4)
that is
[passed in as an argument](https://github.com/sensu/sensu-eval-stack/blob/master/sparkleformation/sensu_stack.rb#L67)
when calling the registry in the stack template.

A simple example perhaps but this is exactly the type of thing that leads to
better code reuse and eases management of complex definitions over the lifespan
of any infrastructure as code workflow.
