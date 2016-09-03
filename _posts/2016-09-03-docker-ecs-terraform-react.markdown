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

We will be using the codebase from my [previous blog on react router](http://www.reactjunkie.com/step-four-to-react-routing-with-react-router/). It's 
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

## Step 2: Create a Dockerfile

We need to create a Dockerfile first. This is the sequence of commands you tell docker to execute to create the image. 
It's akin to you manually entering a sequence of shell commands on the terminal of a new linux box when deploying your app. Except
docker runs it for you automatically, and then saves the resultant state of that linux box as an image.

So right click on the root directory of your project, add a new file call it Dockerfile. This It should look like this:

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

Important points:
<ul>
<li>The commands in this Dockerfile is relative to the current directory you are in which should be the same as the location of 
the Dockerfile. For example when docker runs the command "COPY package.json /src/package.json" it looks for package.json in the same directory as 
the Dockerfile and then copies that file to the /src in the image.</li>
<li>Docker build cache</li>
</ul>

For more information on docker build cache check the official doco [here](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)



####Lill: Standard es6 class method
{% highlight c# %}
import React, {Component} from 'react';

export default class MethLab extends Component {
    
    constructor(props) {
        super(props);
        
        this.state = {
            methQuality: 'blueSky'
        };
        
        // manually bind "this" context of printQuality function
        // to the instance of MethLab being constructed
        this.printQuality = ::this.printQuality;
    }
    
    // Use the standard es6 class method syntax
    printQuality() {
        console.log(`${this.state.methQuality}`);
    }
}
{% endhighlight %}

## Other news
I'll be attending [NDC Sydney](http://ndcsydney.com/){:target="_blank"} on 3-5 August. If you are around, please say hello otherwise
it will be quite a lonely conference for me :(

I have also organised a reading group which will take place on Thu 25 Aug. The pick is ["If Hemingway Wrote Javascript"
by Angus Croll](https://www.nostarch.com/hemingway){:target="_blank"}. Please reach out to me if you would like to join!

Until next time, cowabunga.


---------------------------------------------------------------------------------------
