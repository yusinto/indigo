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


In this blog I'll walk you through how to set up a dev pipeline on GCP and the Kubernetes Engine.   

## Goal
During development, the dev test cycle is predictable and well-defined. You write code, it runs locally and pass all the 
tests then you push your code to git. The *intent* is clear. Pushing to git means you want to see that code running on an 
environment somewhere. This environment should also be predictable and well-defined. Without doing any more work, you
should be able to open your browser and run the code you just pushed in this well-defined environment.

The goal of this tutorial is to build a pipeline that does exactly that. At the end of this post, you should be able to:

1. Create a new feature branch off master.
2. Write code and test locally.
3. Push to git.
4. Browse to a well-defined feature url which runs the new code.

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

![Container builder trigger settings](/assets/images/build_trigger_settings.jpg)

1. Set the trigger type to branch, because we want to invoke this trigger when a branch push happens. There is also
an option to trigger on tag push, which is useful for production deployment (when you create a release tag) but we'll
cover that in future post.

2. Set a regex to match the feature branch names. Mine feature branches start with feature-[JIRA_TICKET_NUMBER]-description.
All the developers follow this branch naming convention when they create a new feature branch. Once a convention is in place, you
can set a regex expression here to match your convention.

3. You define your build steps in a yaml file called cloudbuild.yaml. By default container builder looks for *cloudbuild.yaml*
at the root of your repo. You can specify a custom location and yaml file if you want.

## Step 3: cloudbuild.yaml

So what's in this yaml file?



## Goal
Create a relay modern app with ssr with found and found relay.

## Step 1: Install npm packages
Install babel-polyfill (for async await), found and found relay

## Step 2: Create routes
create routes, makeRouteConfig, export that routeConfig object.
createBrowserRouter using routeConfig object.

## Step 3: SSR
use the same routes object
on the client bootstrap use createInitialBrowserRouter instead and await on that
u also need createRender function (TODO: investigate why?) 

On the server, await getFarceResult from found/lib/server passing in url, routeConfig and 
render method to get the output data from the requested route.

Then renderToString the element property from the output. That's it!

Without redux, it's soo much cleaner! 

Install relay-compiler-plus and the latest [graphql-js](https://github.com/graphql/graphql-js){:target="_blank"} package:

{% highlight bash %}
yarn add relay-compiler-plus
{% endhighlight %}

{% highlight bash %}
yarn upgrade graphql --latest
{% endhighlight %}

## Step 2: Compile
Add this npm command to your **package.json**:

{% highlight json %}
"scripts": {
    "rcp": "relay-compiler-plus --schema <SCHEMA_FILE_PATH> --src <SRC_DIR_PATH> -f"
},
{% endhighlight %}
    
where:<br/> 
`<SCHEMA_FILE_PATH>` is the path to your schema.graphql or schema.json file<br/>
`<SRC_DIR_PATH>` is the path to your src directory<br/>
`-f` will delete all `**/__generated__/*.graphql.js` files under `SRC_DIR_PATH` before compilation starts<br/>

Run the command to start compiling:

{% highlight bash %}
npm run rcp
{% endhighlight %}

## Step 3: Map query ids on the server
On the server, use `matchQueryMiddleware` prior to `express-graphql` to match query ids to actual queries. Note 
that `queryMap.json` is auto-generated by `relay-compiler-plus` in the previous step.

{% highlight javascript %}
import Express from 'express';
import expressGraphl from 'express-graphql';
import {matchQueryMiddleware} from 'relay-compiler-plus'; // do this
import queryMapJson from '../queryMap.json'; // do this

const app = Express();

app.use('/graphql',
  matchQueryMiddleware(queryMapJson), // do this
  expressGraphl({
    schema: graphqlSchema,
    graphiql: true,
  }));
{% endhighlight %}

## Step 4: Send query ids on the client
On the client, modify your relay network fetch implementation to pass a `queryId` parameter in the
request body instead of a `query` parameter. Note that `operation.id` is generated by `relay-compiler-plus` in step 2.

{% highlight javascript %}
function fetchQuery(operation, variables,) {
  return fetch('/graphql', {
    method: 'POST',
    headers: {
      'content-type': 'application/json'
    },
    body: JSON.stringify({
      queryId: operation.id, // do this
      variables,
    }),
  }).then(response => {
    return response.json();
  });
}
{% endhighlight %}

## Bonus
In [step 2](#step-2-compile), running relay-compiler-plus generates relay query files like the original relay-compiler,
but with a difference. Inspect a generated `ConcreteBatch` query file and you'll see that it now has an `id` assigned 
to it and that the query `text` is now `null`:

{% highlight javascript %}
const batch /*: ConcreteBatch*/ = {
  "fragment": {
    "argumentDefinitions": [],
    "kind": "Fragment",
    "metadata": null,
    "name": "client_index_Query",
    "selections": [...],
    "type": "Query"
  },
  "id": "6082095e8a45f64d38924775d047cf8c", // look ma, query id!
  "kind": "Batch",
  "metadata": {},
  "name": "client_index_Query",
  "query": {...},
  "text": null // look again ma, null query text!
};
{% endhighlight %}

The id is an md5 hash of the query text, generated by the [persistQuery](https://github.com/yusinto/relay-compiler-plus/blob/master/src/compiler/main.js){:target="_blank"} 
function. It looks like this:

{% highlight javascript %}
 function persistQuery(operationText: string): Promise<string> {
   return new Promise((resolve) => {
     const queryId = md5(operationText);
     
     // queryCache is written to disk at the end as queryMap.json
     queryCache.push({id: queryId, text: operationText});
     resolve(queryId);
   });
 }   
{% endhighlight %}

As you can see above, the hash to query text mapping is saved to an array which gets written to disk
at the end of the compilation as queryMap.json. This is used on the server side as outlined in 
[step 3](#step-3-map-query-ids-on-the-server).

## Conclusion
You can find the package at [github](https://github.com/yusinto/relay-compiler-plus){:target="_blank"} with a fully working
[example](https://github.com/yusinto/relay-compiler-plus/tree/master/example){:target="_blank"}. 

Let me know if this is useful (or not)! 

---------------------------------------------------------------------------------------
