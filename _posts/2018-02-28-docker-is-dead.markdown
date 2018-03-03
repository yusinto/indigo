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
Kubernetes rocks. Kubernetes engine to be exact. I was so inspired by Kelsey Hightower's
[presentation](https://www.youtube.com/watch?v=kOa_llowQ1c&feature=youtu.be){:target="_blank"} at KubeCon 2017 I
spent the next 2 weeks migrating my entire pipeline from Jenkins/AWS/Terraform to Google Cloud Platform (GCP for short) 
and the Kubernetes Engine. It's the best decision I have ever made.

Ok so the title is a little dramatic. Kubernetes uses docker so it doesn't actually really kill docker, it merely pushes
it further into the background. That means you are free to focus on the bigger picture, which is your end to end pipeline
and your code. During my migration, I found that apart from the dockerfile and a docker build, I didn't really have to
touch docker much. It's a sign that the docker stack has matured.

In this blog I'll walk you through how to set up a continuous dev pipeline on GCP and the Kubernetes Engine.   

## Goal
During development, the dev test cycle is predictable and well-defined. You write code, it runs locally and pass all the 
tests then you push your code to git. The *intent* is clear. Pushing to git means you want to see that code running on an 
environment somewhere. This environment should also be predictable and well-defined. Without doing any more work, you
should be able to open your browser and run the code you just pushed in this well-defined environment.

The goal of this tutorial is to build a pipeline that does exactly that. At the end of this post, you should be able to:

1. Create a new feature off master, code and test locally.
2. Push to git.
3. Browse to a well-defined feature url which has the new code running.

## Step 1: Create a Kubernetes cluster
Jump into google cloud console and under the main menu, go to **Compute -> Kubernetes Engine -> Kubernetes clusters** and
click on Create Cluster. You'll see a screen like below, you only need to touch 3 fields:

![Create Kubernetes cluster](/assets/images/create-cluster.png)

1. Name your cluster *feature-cluster*. When we push to git, we'll deploy our feature to nodes running on this cluster.

2. Pick a zone that's closest to you. The Zonal and Regional option refers to the location of your Kubernetes master. 
Zonal means your master service will be running in a single zone that you picked. Regional means your master service 
will be running in multiple zones in that region for redundancy. To keep things simple for this demo we'll stick with zonal.

3. Pick the latest cluster version to get all the latest features.

Leave the remaining default settings e.g. 3 nodes in the cluster on Container-Optimised OS and click create. It takes 
some time for google to create your cluster because it has to provision the nodes. While that's cooking, we'll create 
the build job.

## Step 2: Create a build trigger
In the console menu, go to **Tools -> Container Registry -> Build triggers**. Add a new trigger. Select your git source 
(I use github) and repo and authorise container builder to access it. Then you'll get to the
Edit Trigger page like below:

![Container builder trigger settings](/assets/images/create-trigger.png)

1. Give your trigger a name, I name mine *feature* because it gets triggered by a feature push and builds and deploys 
that feature.

2. Set the trigger type to branch, because we want to trigger a build when a feature branch push happens. There is also
an option to trigger on tag push, which is useful for production deployment (when you create a release tag) but we'll
cover that in a future post.

3. Set a regex to match the feature branch names. By convention I enforce the following convention for features:
**feature-[JIRA_TICKET_NUMBER]-description**. All the developers follow this naming convention when they create a new 
feature branch. Once a convention is in place, you can set a regex expression here to match your feature branches.

4. You define your build steps in the cloudbuild.yaml file. Set the location of *cloudbuild.yaml* so container builder
can find it in your repo. I set mine to the root of my repo.

## Step 3: cloudbuild.yaml
Container builder uses this file to execute a series of steps to build and deploy your app. Each build step specified
here is a container running in its own shell. For our demo, we'll use the cloudbuild.yaml below:

<script src="https://gist.github.com/yusinto/3922f40d0b8d0241b6c6ead1a9aa8f3f.js"></script>

There are 3 build steps in our *cloudbuild.yaml* file. Each step has a **name** and and an **id**. The **name** field refers to a 
docker image that the build step will pull and run to execute the step. Container builder supports a [common set of builder
images](https://github.com/GoogleCloudPlatform/cloud-builders){:target="_blank"} you can use as build steps. There are also [community
images](https://github.com/GoogleCloudPlatform/cloud-builders-community){:target="_blank"}. In this example we'll only
use google builder steps because it's more than sufficient for our needs.

The **id** field is optional but useful to specify because it will be displayed in the build logs. Otherwise you'll see the 
**name** field instead, which is not as informative. Also by specifying an id,
you can make subsequent build steps to **waitFor** this build step so those child steps can run concurrently. Speed up baby! So cool!

We build our docker image in the first step. By specifying **gcr.io/cloud-builders/docker**, container builder
will pull that image and run a docker container running the docker command. We'll pass the build arguments to this 
docker command like we would in a terminal. In the same fashion, the second step pushes the image we just created.
These steps are equivalent to:

{% highlight bash %}
docker build -t gcr.io/gke-playground/some-branch-name:latest .
docker push gcr.io/gke-playground/some-branch-name:latest
{% endhighlight %}


## Step 4: deployment-template.yaml
Let's take a look at the third step which runs **kubectl**. This step deploys our docker image to the Kubernetes cluster. Note
that this step runs the **gcr.io/cloud-builders/kubectl** image, but we specified the
entrypoint as **bash** meaning that the container will run the bash command when it starts rather
than **kubectl**. This is a useful technique if you need to pre-run some commands prior to executing
the main command. We'll come back to this in a minute after inspecting **deployment-template.yaml**:

<script src="https://gist.github.com/yusinto/9536fa7dcd28106efee7f8b217a9d06a.js"></script>

This is a standard kubernetes resource yaml specifying two resources to be created/updated in the cluster:

1. A deployment containing our pod spec

2. A service to expose our pods to the external world so that it's accessible via the internet.

We use a placeholder string **_BRANCH_NAME** which gets replaced by the real **$BRANCH_NAME** using the **sed** command
in our build step:

{% highlight bash %}
sed -e "s|_BRANCH_NAME|$BRANCH_NAME|g" deployment-template.yaml | tee deployment.yaml
{% endhighlight %}

**$BRANCH_NAME** is injected by the Container Builder as an environment variable to all build steps. This is the git 
branch name that triggers the build. There are [others](https://cloud.google.com/container-builder/docs/configuring-builds/substitute-variable-values){:target="_blank"} 
you can use.

This produces a new `deployment.yaml` which gets used by `kubectl` for deployment:

{% highlight bash %}
kubectl apply -f deployment.yaml
{% endhighlight %}

This way, we can deploy each branch to its own infrastructure mirroring our git branching strategy.

## Step 5: What's this **images** thing at the end?
At the end of **cloudbuild.yaml**, the **images** command will push the specified docker images to the remote registry. 
Wait a minute! Didn't we already push our docker image in build step #2? Yes and you'd be right my observant friend (if you
are still awake reading this long post). It is common to rely on the **images** command to push our docker images rather than
explicitly doing so through a build step. However we had to push prior to executing **kubectl** so we could not rely on the
**images** command which runs at the **end** of all build steps. So what's the point of having the **images** command in the yaml
at all? If you pushed an image explicitly in a build step AND specify the **images** command, you will see the image pushed in the
build results in the console. If you don't specify the **images** command, you won't see it in the build results. Simple! 

## Step 6: Test the app!
It takes a while for google cloud to assign an external ip to our service. You can check under **Compute -> Kubernetes Engine -> Discovery & load balancing**.
The *Endpoints* column should display a valid external ip when it's ready. Then you can hit that link and your app should be running!

## Conclusion
Deploying my app used to involve many moving parts: Jenkins, ec2 instances, load balancers, terraform, ecs. Kubernetes Engine
greatly simplifies this by encapsulating many of the moving parts. When used in conjunction with Container Builder, the 
dream of a continuous end to end build pipeline suddenly becomes somewhat easier to achieve. This is just the tip of the
iceberg, I am so excited! 

In part two of this series I'll walk you through the teardown process to complete the feature development pipeline. 
You don't want unused resources running in the cloud burning your wallet! Till next time.

---------------------------------------------------------------------------------------
