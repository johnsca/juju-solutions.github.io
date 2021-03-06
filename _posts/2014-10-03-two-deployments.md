---
layout: post
title: "A Tale of Two Deployments"
date: 2014-10-03 10:06
comments: true
categories: [juju,  cloudfoundry]
tags: ['framework charms', 'PAAS']
author: "Ben Saller"
author_url: "@bcsaller"
---

By: @bcsaller w/ @whitmo

At the Juju Solutions sprint in Dillon
[Chuck Butler](https://twitter.com/lazypower) demoed a nice example of
using the
[Rails charm](https://manage.jujucharms.com/charms/precise/rails) to
deploy a sample app/workload right out of github
<https://github.com/leereilly/github-high-scores.git>. The rails charm
serves as a great example of framework charm, automating many of the
common tasks around running and managing a
[Rack](http://rack.github.io/) served application.

I've spent the last few months working on using Juju to deploy and
manage a system that also helps deploy and manage applications:
CloudFoundry.  Strikingly, this charm provides a user a similar
develop and deploy experience as using a PAAS like [Cloudfoundry](http://cloudfoundry.org/index.html), [Heroku](https://manage.jujucharms.com/charms/precise/rails) or
[OpenShift](https://www.openshift.com/)

My curiousity piqued, I did an experiment to compare the the relative
experience of deploying a rails app via a framework charm vs. the
experience of deploying the same app into a PAAS.

<!-- more -->

First I juju deployed Cloudfoundry into AWS. Then I prepared the demo
app for cf deployment.

By adding a few lines (a manifest.yml) to a clean checkout I was able
to deploy the same application to a my CloudFoundry instance. Just to
show the scope of change, look at the following:

    ---
    applications:
    - name: highscore
      buildpack: https://github.com/cloudfoundry/heroku-buildpack-ruby.git
      command: ruby app.rb $PORT


Then I deployed into cf.

    cd highscore
    cf push


Pretty simple.

Here is all Cloudfoundry requires to bring a supported type of
application under management: a snippet of yaml and a command. Because
CF provides all the conventions around application management this
yaml effectively 'charms' an application. Add file, and boom, the
repository become deployable.

From this perspective I'd like to compare and contrast these two solutions to
running this simple app. With the rails charm you'd set service options to
point to the github repo. The charm is written in such a way as to pull down
the app and bootstrap it. Fairly similar to the CF deployment so far.

So what are the differences?

By default with Juju you get a new machine per Rails app you want to
run. This means that deploying the Rails app now means a bootstrap,
unit allocation and then the spin up of the configured
application. Let's say this whole process is about 15 minutes for the
first unit and about 10 for each subsequent unit. Its not fast but
you're only paying for the machines you need in a public cloud
situation.

In the CloudFoundry world you'll have to deploy the whole PaaS
first. Deployment takes about 45 minutes process, depending on your
configuration.  It will start with 1 very large instance or more
commonly about 9 medium sized ones. If you're using the whole platform
to run a simple app like this one it would be a very expensive way to
do it.

Once its running you have a platform that supports running many
isolated applications with security, user and org management. And
applications deploy very fast. In this case it took about 30 seconds
to provision the application the first time and each additional takes
less (as internally cf uses an image for the app's container).

Speed isn't the only difference though. In the Juju world you could
relate your application to anything the Rails charm supported, say
your Juju deployed database. If the Rails charm predefined a mysql
relation then you could make that available to your application and
changing that relation could trigger restarts of the app.

CloudFoundry has its own system for dealing with what we call
"relations" in juju.  While not the not exactly same, they also
support a service model and the ability to add catalogs of services
that can be used by a cf hosted app. In CF's model the service
bindings are declared as needed and the application holds the
responsibility to take advantage of them. Understand that in this
model if your Ruby code supported MongoDb and the catalog provides
that service, you could just use it.

In the Juju model if the Rails charm doesn't support MongoDB you have
to fork supposedly generic charm to add that relation to
metadata.yaml or forego using Juju to orchestrate the
application's use of Mongo.

As the purpose of a PaaS is a tailored experience for the user, an
application developer, we can see certain tradeoffs here.  CF makes
app deployment fast and easy, and flexible with it's adhoc
relations. It trades off upfront deployment effort in time and
resources, as well as limiting the complexity of deployment and the
richness of types of relationship.

Looking forward, I'm hopeful that we'll be able to provide a sensible
bridge to CF application so they can take advantage of the Juju Charm
catalog in the future, but I'm equally hopeful that extend Juju's
model in such a way that we can separate the handling of deploying a
Rails charm with its needs and requirements from those of the
application payload that it manages. This is the idea of Juju
deploying things that we deploy things onto. CF might be one example
of a Juju deployed thing we deploy other things to, but at the same
time, so is the Rails charm, so is Mesos, so is Kubernetes.

Deploying things into things we've deployed is an exciting growth path
for Juju and a topic for another day.
