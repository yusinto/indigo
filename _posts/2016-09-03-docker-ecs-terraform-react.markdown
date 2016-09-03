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
 
## Objective
By the end of this blog, we want to be able to create a docker image containing our react app, be able to run lint, tests and the actual app
on that image and save that image on aws (on ecr; short for ec2 container registry, which is the aws version of dockerhub). 

We will be using the codebase from my [previous blog on react router](http://www.reactjunkie.com/step-four-to-react-routing-with-react-router/). It's 
a minimal react spa with routing, you should be able to easily substitute your own codebase and follow the steps here to use docker.

## Step 1: Install docker


## Step 2: Create a docker image

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
