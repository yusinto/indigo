---
published: true
title: "Implementing feature toggle with launch darkly and react redux"
layout: post
date: 2016-09-09 10:52
tag:
- react
- redux
- feature
- toggle
- launch
- darkly
- toggling
blog: true
---

I first heard the term feature toggle from someone I interviewed when we were talking about continuous deployment (deploying to prod at anytime, continuously).
I didn't really think much of it at first, thinking that it's just a trivial bunch of if-else flags that I have to maintain
manually. I couldn't be more wrong.

Maintaining feature flags in your codebase can become messy very quickly. On top of that you need to build targeting and analytics tools that 
the business would (eventually) want. I soon realise that feature toggle is a distinct and separate supporting domain of my core domain. Much like identity and
access management. So is there anything out there we can use to integrate with our core domain?

Enter Launch Darkly. I met Edith Harbaugh the CEO of Launch Darkly at NDC Sydney 2016. She was kind enough to give a demo of launch darkly in situ and I was blown away.
Launch darkly stores feature flags only without coupling to the UI, so it works with React or any modern javascript framework with virtual dom reconciliation. 
The sdk supports almost every imaginable platform, but most importantly it's on npm [ldclient-js](https://www.npmjs.com/package/ldclient-js){:target="_blank"} and 
open source on [github](https://github.com/launchdarkly/js-client){:target="_blank"}. 

In this blog I will walk through step by step how to integrate launch darkly feature toggles with your react react-router redux app.
Note that this blog assumes prior working knowledge of redux and redux-thunk.
 
## The end game
By the end of this blog, we will have a feature-flag driven react redux app using Launch Darkly as our feature toggle provider.

## Step 1: Configure launch darkly dashboard

[insert video link]

## Step 2: Install ldclient-js
{% highlight bash %}
npm i ldclient-js --save
{% endhighlight %}

## Step 3: Create redux action to instantiate ldclient
We need to create an instance of ldclient in order to communicate with launch darkly from our app. This instantiation
should be done once at the start of the app, and the resultant client object stored in redux state to be 
re-used throughout the app. 

We'll go ahead and create the action and reducer to perform this instantiation.

####appAction.js
{% highlight c# %}
import ldClient from 'ldclient-js';

export const initialiseLD = () => {
  // use redux-thunk for async action
  return dispatch => {
    /**
     * Launch darkly provides a comprehensive suite of targeting and
     * rollout options. Targeting and rollouts are based on the user
     * viewing the page, so we must pass a user at initialisation time.
     *
     * The user object can contain these properties:
     * key, ip, firstName, lastName, country, email, avatar, name,
     * anonymous.
     *
     * The only mandatory property is "key". All the others are
     * optional. You can also specify custom properties using the
     * "custom" property name like company and authorOf properties below.
     *
     * For more info on users, check here:
     * http://docs.launchdarkly.com/docs/js-sdk-reference#users
    */
    const user = {
      "key": "deadbeef", // MANDATORY!
      "firstName": "John",
      "lastName": "Carmack",
      "email": "jcarmack@doom.com",

      // specify custom properties here. These will appear
      // automatically on the dashboard.
      "custom": {
        "company": "ID Software",
        "authorOf": ["doom", "quake"]
      }
    };

    // Actual work done here. You'll need to use your environment id as
    // configured in your dashboard
    const client = ldClient.initialize('YOUR-ENVIRONMENT-ID', user);

    // The client will emit an "ready" event when it has finished
    // initialising. At that point we want to store that client
    // object in our redux state. Do this by dispatching an action
    // with the client object as the argument to be stored in app state.
    client.on('ready', () => {
      dispatch(setLDReady(client));
    });
  };
};

// Stores launch darkly client object in app state
export const setLDReady = ldClient => {
  return {
    type: Constants.LD_READY,
    data: ldClient
  }
};
{% endhighlight %}

####appReducer.js
{% highlight c# %}
import Constants from './common/constant';

const defaultState = {
  isLDReady: false,
  ldClient: null,
};

export default function App(state = defaultState, action) {
  switch (action.type) {
    case Constants.LD_READY:
      return Object.assign({}, state, {isLDReady: true, ldClient: action.data});

    default:
      return state;
  }
}
{% endhighlight %}

## Step 4: Call initialiseLD from the root component
We want to initialise the client just once at the start of the app. The
best place to do this is at the root component's componentDidMount. 

I'll skip the appContainer snippet to keep things short.

####appComponent.js
{% highlight c# %}
import React, {Component} from 'react';

export default class App extends Component {
  componentDidMount() {
    // This will trigger ldclient initialisation
    this.props.initialiseLD();
  }

  render() {
    ...
  }
}
{% endhighlight %}

## Step 3. Fetch feature flags
So now we have the client initialised, we can fetch our flags! We do 
this by invoking the "variation" method on the ldclient object.


## Step 4: Use feature flags

## Step 5: Subscribe to feature flag changes
  
 
## Step 1: Install docker
I'm on a mac so download docker for mac from [here](https://download.docker.com/mac/stable/Docker.dmg). It's a 110mb download so stop reading and do it first, continue reading later.
There are some hardware & os requirements. The important ones are:
<ul>
<li>OS X 10.10.3 Yosemite or newer</li>
<li>VirtualBox 4.3.30 or newer</li>
</ul>

If you have a previous install of docker-machine, mac docker will ask if you want to migrate data from that install to your new install. I said no because I don't have anything
important to migrate as I'll be starting from scratch.

Once installation is done, open terminal and type:
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

So right click on the root directory of your project, add a new file call it <i>Dockerfile</i>. It should look like this:

####DockerFile
{% highlight bash %}
# We need a base image to build upon. Use the latest node image from 
# dockerhub as the base image so we get node and npm for free
FROM node:latest
MAINTAINER Yus Ng

# Store all our app code in the /src folder, starting from package.json
# first. Why copy package.json first? So we can take advantage of 
# the docker build cache. More below.
COPY package.json /src/package.json

# Once we have package.json, do npm install (restricting the loglevel
# to minimise noise)
RUN cd /src && npm install --loglevel error

# Copy all our code (yes including package.json again) to /src. 
COPY . /src

# Change directory into the /src folder so we can execute npm commands
WORKDIR /src

# This is the express port on which our app runs
EXPOSE 3000

# This is the default command to execute when docker run is issued. Only
# one CMD instruction is allowed per Dockerfile.
CMD npm start
{% endhighlight %}

<b>Important points</b>: 
<ul>
<li>Each instruction creates a new intermediate layer.</li>
<li>Docker uses the instruction string as the cache key. The result of that instruction is a new layer which gets stored as the cache value.</li>
<li>ADD and COPY instructions are special. The cache key for these are the checksum of their file contents.</li>
<li>If your package.json file does not change, the cache will be hit because the checksum matches. The next instruction npm install will also hit the cache 
because docker uses the instruction string as key which has not changed.</li>
<li>In contrast, consider what will happen if do COPY . /src and then followed by RUN npm install.</li>
<li>The COPY command does a checksum of all the files in current directory and compares that against previous layers. Some files
would have changed in the src folder, because it contains all our source code, images, config files, styles, etc. The checksum comparison would not match, hence
the cache will be invalidated. Once invalidated, all subsequent instructions will create new layers ignoring the cache.</li>
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
npm-debug.log
{% endhighlight %}

## Step 4: Build the docker image

Let's do it! Go to terminal, cd into your root project folder where your Dockerfile resides and type the following (<b>NOTE</b> the "." at the end
is very important!): 

{% highlight bash %}
docker build -t reactjunkie:v1 .
{% endhighlight %}

Docker will build an image named "reactjunkie:v1" using the Dockerfile specified in the current directory (represented by the "." at the end). You can see it
by issuing the command:

{% highlight bash %}
docker images
{% endhighlight %}

You should see two images; the latest node base image which gets downloaded when docker built our image and our reactjunkie:v1 image.

## Step 5: Run the docker container

Now we have an image, we can start a container based on that image and run our app!

{% highlight bash %}
docker run -d -p 8080:3000 reactjunkie:v1
{% endhighlight %}

This command tells docker to run the default CMD command specified in the last line of our Dockerfile above. As we will see shortly we can
override this default by issuing our own commands.

<p>The -d flag tells docker to detach from the container process after issuing the command so we regain control of our terminal window.</p>
The -p flag maps the port on your mac (the host) to the container port.
Hit [http://localhost:8080](http://localhost:8080){:target="_blank"} and you should be able to see the app running!

## Step 6: Running lint and test

So the previous step demonstrated how we can run our webapp on our docker container. However in a CI environment, we want to be able to first build our image,
run lint and tests and then save the image first prior to starting the web app. 

I've setup eslint and jest in this project (available on [github](https://github.com/yusinto/docker){:target="_blank"}). To run eslint and tests on our container, type 
the following:

{% highlight bash %}
docker run -i --rm reactjunkie:v1 npm run lint
docker run -i --rm reactjunkie:v1 npm t
{% endhighlight %}

<p>The -i flag tells docker to run in "interactive" mode so we can see eslint console output from the container.</p>
<p>The --rm flag tells docker to automatically clean up the container and remove its file system when the it exits.</p>
<p>Then npm run lint and npm t are the commands that override the default CMD instruction in our Dockerfile. Docker will start
a container based on our image, issue these commands and then cleanup and remove the container when that command is finished.</p>

## What's next?
Now we have the docker image on our local machine, we need a way to export it to a central place so it can be shared with other developers, 
build systems and so on. Docker has dockerhub which does exactly that, but I use ECR which is aws' offering.

All the code in this blog are available on [github](https://github.com/yusinto/docker){:target="_blank"}

To be continued...


---------------------------------------------------------------------------------------
