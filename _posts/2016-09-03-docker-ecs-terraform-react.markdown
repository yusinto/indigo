---
published: true
title: "Deploying react apps on docker using ecs and terraform"
layout: post
date: 2016-09-03 10:52
tag:
- react
- docker
- ecs
- terraform
- deploy
- continuous
- integration
- deployment
blog: true
---

I was tinkering on blogging about Step 5 to React: Introduction to Redux, but I decided to write something closer to my heart: 
using ecs and terraform to deploy docker react apps (can I squeeze in anymore buzzwords?). This is not an intro to docker so 
I assume you are familiar with the basics. 
 
My plan was to use docker to achieve continuous integration and ultimately continuous delivery and then deployment. But I like to
start things small so the plan was to firstly create a docker image on every master merge and save that image. 

This image
is a stable and deployable package which can be deployed and run on any environment. This is the "golden" build concept - a single build
that is used on all steps of the deployment pipeline: uat, stage and prod. This is a good practice to adopt because the exact same 
build that has gone through all the QA steps is the one that's going out to production. 

It gives you confidence the prod deployment at the end
will work as expected. This is only possible if the package that comes out of your CI build is immutable - that's where docker comes in. I don't 
know about you but I get really turned on by this kind of stuff! Let's get to it!
 
## The end game
By the end of this blog, we want to be able to create a docker image containing our react app, be able to run lint, tests and the actual app
on that image and save that image on aws (on ecr; short for ec2 container registry, which is the aws version of dockerhub). 

We will be using the codebase from my [previous blog on react router](http://www.reactjunkie.com/step-four-to-react-routing-with-react-router/){:target="_blank"}. It's 
a minimal react spa with routing, you should be able to easily substitute your own codebase and follow the steps here to use docker.

## Step 1: Install docker
I'm on a mac so download docker for mac from [here](https://download.docker.com/mac/stable/Docker.dmg). It's a 110mb download so stop reading and do it first, continue reading later.
There are some hardware & os requirements. The important ones are:
<ul>
<li>OS X 10.10.3 Yosemite or newer</li>
<li>VirtualBox 4.3.30 or newer</li>
</ul>

If you have a previous install of docker-machine, mac docker will ask if you want to migrate data from that install to your new install. I said no because I don't have anything
important to migrate as I'll be starting from scratch.

Then open terminal and type:
{% highlight bash %}
docker -v
{% endhighlight %}

You should get something like 
{% highlight bash %}
Docker version 1.12.0, build 8eab29e
{% endhighlight %}

Now we are ready to rock!

## Step 2: Create Dockerfile

We need to create a Dockerfile first. This is the sequence of instructions you tell docker to execute to create the image. 
It's akin to you manually entering a sequence of shell commands on the terminal of a new linux box when deploying your app. Except
docker runs it for you automatically, and then saves the resultant state of that linux box as an image.

So right click on the root directory of your project, add a new file call it <i>Dockerfile</i>. This It should look like this:

####DockerFile
{% highlight bash %}
# We need a base image to build upon. Use the latest node image from 
# dockerhub as the base image so we get node and npm for free
FROM node:latest
MAINTAINER Yus Ng

# Store all our app code in the /src folder, starting from package.json
# first. Why copy package.json first? So we can take advantage of 
# the docker build cache. See detailed explanation below.
COPY package.json /src/package.json

# Once we have package.json, do npm install (restricting the loglevel
# to minimise noise)
RUN cd /src && npm install --loglevel error

# Copy all our code (yes including package.json again) to /src. 
# Why can't we just do this and then npm install after this step?  
# Why do we need the two steps above?? Be patient young skywalker, see 
# explanation below regarding docker build cache.
COPY . /src

# Change directory into the /src folder so we can execute npm commands
WORKDIR /src

# Produce the bundled.js and css files.
RUN npm run build

# This is the express port on which our app runs
EXPOSE 3000

# Set entrypoint to be executable
RUN chmod 755 /src/dockerEntryPoint.sh
ENTRYPOINT ["/src/dockerEntryPoint.sh"]
{% endhighlight %}

<b>Important points</b>: 
<ul>
<li>The first argument to the COPY instruction in the Dockerfile is relative to the current local directory you are in.</li>
<li>For example the instruction "COPY package.json /src/package.json" looks for package.json in your current local directory 
and then copies it to the /src directory in the image.
</li>
<li><b>Docker build cache explanation</b> - Docker has a cache to optimise and speed up image builds. Each instruction in the dockerfile is checked against previous images' instructions
to see if there was one built using the exact same instruction. This check is a string comparison and the cache contains the output of the instruction. 
The cache will only be used if the instructions are exactly the same string.</li>
<li>This is very efficient and helps speed up the image build process. However for a command like "COPY package.json /src/package.json" clearly a simple
string check is insufficient because the contents of our package.json might have changed. If a cache copy is used from a previous image that has
a package.json with different dependencies then our build will contain extraneous/missing packages. Docker deals with this by making ADD and COPY
instructions special. Instead of a simple string comparison, a checksum is calculated for each file in the ADD/COPY instruction. This checksum is used
to compare the new files and the old files. If the checksum is different i.e. something has changed in the file, then the cache is invalidated. Once
the cache is invalidated, subsequent commands will generate new images and ignore the cache.
</li>
<li>
It is therefore prudent and highly recommended that you always copy package.json first, and then npm install so you can utilise the cache.
</li>
</ul>

For more information on docker build cache check the official doco [here](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/){:target="_blank"}.

## Step 3: Create .dockerignore

We need one more file before we can build our image. Go ahead and add a new file to the root directory of your project call it 
<i>.dockerignore</i>

Docker will exclude files and directories specified here from the image. It should look like this:

{% highlight bash %}
.git
.gitignore
node_modules
{% endhighlight %}

## Step 4: Build the image

Let's do it! Go to terminal, cd into your root project folder where your Dockerfile resides and type the following (<b>NOTE</b> the "." at the end
is very important!): 

{% highlight bash %}
docker build -t reactjunkie:v1 .
{% endhighlight %}

Docker will build an image named "reactjunkie:v1" using the Dockerfile specified in the current directory (represented by the "." at the end).

####Lill: Standard es6 class method
{% highlight c# %}
{% endhighlight %}

## What's next?
I'll be attending [NDC Sydney](http://ndcsydney.com/){:target="_blank"} on 3-5 August. If you are around, please say hello otherwise
it will be quite a lonely conference for me :(

I have also organised a reading group which will take place on Thu 25 Aug. The pick is ["If Hemingway Wrote Javascript"
by Angus Croll](https://www.nostarch.com/hemingway){:target="_blank"}. Please reach out to me if you would like to join!

Until next time, cowabunga.


---------------------------------------------------------------------------------------
