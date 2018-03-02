---
published: true
title: "Docker is dead long live Kubernetes"
layout: post
date: 2018-02-28 07:30
tag:
- docker
- kubernetes
- engine
- google
- cloud
- platform
- ci
- cd
- continuous
- deployment
- integration
- build
- pipeline
- end
- to
- end
blog: true
---
Kubernetes rocks. Kubernetes engine to be more exact. I was so inspired by Kelsey Hightower's
[presentation](https://www.youtube.com/watch?v=kOa_llowQ1c&feature=youtu.be){:target="_blank"} at KubeCon 2017 I
spent the next 2 weeks migrating my entire pipeline from Jenkins/AWS/Terraform to Google Cloud Platform (GCP for short) 
and the Kubernetes Engine. It's the best decision I have ever made.

Ok so the title is a little dramatic. Kubernetes uses docker so it doesn't actually really kill docker, it merely pushes
it further into the background. That means you are free to focus on the bigger picture, which is your end to end pipeline
and your code. During my migration, I found that apart from the dockerfile and a docker build, I didn't really have to
touch docker much. It's a sign that the docker stack has matured.

In this blog I'll walk you through how to set up a dev pipeline on GCP and the Kubernetes Engine.   

## Goal
During development, the dev test cycle is predictable and well-defined. You write code, it runs locally and pass all the 
tests then you push your code to git. The *intent* is clear. Pushing to git means you want to see that code running on an 
environment somewhere. This environment should also be predictable and well-defined. Without doing any more work, you
should be able to open your browser and run the code you just pushed in this well-defined environment.

The goal of this tutorial is to build a pipeline that does exactly that. At the end of this post, you should be able to:

1. Create a new feature branch off master, code and test locally.
2. Push to git.
3. Browse to a well-defined feature url which has the new code running.

## Step 1: Create a Kubernetes cluster
Jump into the google cloud console and create a new Kubernetes cluster. I call my cluster *features*. When we push a
git commit to a feature branch, we'll deploy a copy of our app running this feature code on this cluster. I left many
of the default settings e.g. 3 nodes in the cluster on Container-Optimised OS. The console is pretty straightforward
so you shouldn't have any issues.

It takes some time for google to create your cluster because it has to provision the nodes. While that's cooking,
we'll create the build job.

## Step 2:
In the console menu, go to Tools -> Container Registry -> Build triggers. Add a new trigger. You'll need to
select your source (I use github) and repo and authorise container builder to access it. Then you'll get to the
Edit Trigger page which is the interesting part. Give your trigger a name, I name mine *feature* because it
gets triggered by a feature push and builds and deploys that feature.

![Container builder trigger settings](/assets/images/trigger_settings.png)

1. Set the trigger type to branch, because we want to invoke this trigger when a branch push happens. There is also
an option to trigger on tag push, which is useful for production deployment (when you create a release tag) but we'll
cover that in future post.

2. Set a regex to match the feature branch names. Mine feature branches start with feature-[JIRA_TICKET_NUMBER]-description.
All the developers follow this branch naming convention when they create a new feature branch. Once a convention is in place, you
can set a regex expression here to match your convention.

3. You define build steps in the cloudbuild.yaml file. By default container builder looks for *cloudbuild.yaml*
at the root of your repo. 

4. **Optional** - you can specify a custom location and yaml file if you want.

## Step 3: cloudbuild.yaml
This is where you define build steps. Each build step is a container running in its own shell. 

<script src="https://gist.github.com/yusinto/3922f40d0b8d0241b6c6ead1a9aa8f3f.js"></script>

1. Build the docker image of our feature.

2. Deploy that to the Kubernetes cluster we created in [step 1](#step1).

At the end of the yaml, images specified in the *[images]* label will be pushed to the remote registry.


## Step 4: deployment.yaml
This is a standard kubernetes yaml specifying resources in the cluster. 

<script src="https://gist.github.com/yusinto/9536fa7dcd28106efee7f8b217a9d06a.js"></script>

The main points here is that:

1. We specify 2 resources: a deployment containing our pod spec and a service to expose our pods to the external world so that
it's accessible via the internet.

2. We use a placeholder string called _BRANCH_NAME which gets replaced in our build step in [cloudbuild.yaml](#step3) with the real branch name.

## Step 5: Test the app!
It takes a while for google cloud to assign an external ip to our service. You can check under Kubernetes Engine -> Discovery & load balancing.
The *Endpoints* column should display a valid ip address when it's ready. Then you can hit that link and your app should be running!


{% highlight javascript %}
{% endhighlight %}

## Conclusion
You can a full working example on [github](https://github.com/yusinto/kubernetes-engine-playground){:target="_blank"}. In part two, of this
series, I'll walk you through the teardown part of the pipeline to complete the feature development. You don't want unused resources
running in the cloud burning your wallet! Till next time.

---------------------------------------------------------------------------------------
