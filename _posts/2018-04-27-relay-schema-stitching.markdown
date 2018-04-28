---
published: true
title: "Relay Schema Stitching with Pesisted Queries"
layout: post
date: 2018-04-27 07:30
tag:
- relay
- schema
- stitching
- graphql
- apollo
- react
- js
- javascript
- prisma
- graphcool
- persisted
- queries

blog: true
---

Schema stitching was introduced by Apollo, so there is a general misconception that it
only works with the apollo stack but it works with relay as well! In this blog
I'll show you how to stitch your relay schema with any remote graphql schema.

You can find the complete project in [github](https://github.com/yusinto/relay-compiler-plus/tree/master/example-stitching){:target="_blank"}.

## Goal
Given a local schema that looks like this:

<script src="https://gist.github.com/yusinto/c4c6a8f376a36f600ddb0f1f24c952df.js"></script>

Stitch it so that business details are coming from a remote graphql server, in this case from [prisma](https://eu1.prisma.sh/public-nickelwarrior-830/wendarie-prisma/dev){:target="_blank"}.

## Step 1: Install graphql-cli
We'll use graphql cli to download the remote schema:
 
{% highlight bash %}
yarn add -D graphql-cli
{% endhighlight %}

## Step 2: Download the remote schema
Graphql cli looks for a .graphqlconfig.yml by default, so we'll create that file
and it looks like this:

<script src="https://gist.github.com/yusinto/7cfbeeac5e0b6a35dde44a72e5e1a268.js"></script>

In this example, our business details are hosted in [prisma](https://eu1.prisma.sh/public-nickelwarrior-830/wendarie-prisma/dev){:target="_blank"} 
but you can use any graphql endpoint. [Prisma](https://prisma.io){:target="_blank"} is created by the guys from 
graphcool, check it out if you haven't!

Run the following to download the remote schema to ./remote.schema.graphql:
{% highlight bash %}
graphql get-schema
{% endhighlight %}
 

## Step 3: Stitch
The stitching part is standard, following [Apollo's doco](https://www.apollographql.com/docs/graphql-tools/schema-stitching.html){:target="_blank"}:

<script src="https://gist.github.com/yusinto/9ab425fc0a100f81c183e824f1405b57.js"></script>

**Important bits**:

* Line 11: Import the remote schema we downloaded from step 2 using graphql-import. This gets
fed into makeRemoteExecutableSchema.

* Line 14: Use apollo-link to resolve remote queries! If you look at the documentation, there is
a second option of using a fetcher. However doing this will forward the Document AST
to the remote server instead of the query string, which breaks most graphql servers except 
apollo-server.

* Line 20: Extend the Place type to include a reference to Business, which will be resolved to
the remote server via mergeSchemas in line 29.

* Line 29: This defines the resolver for the business field. The key here is the `delegateToSchema`
method which gives you access to the current `Place` being queried based on the fragment defined
in line 30, and access to the remote schema so you can query the remote server with any queries 
you want. In this instance, the query we want to execute on prisma is equivalent to:

<script src="https://gist.github.com/yusinto/650e2f21fc82d8e5e07577a9683e0e76.js"></script>

## Step 4: Compile
Say we have a relay query like so:

<script src="https://gist.github.com/yusinto/44be0c2447ceb723197e3a9bbf2894c9.js"></script>

We'll compile our relay queries using [relay-compiler-plus](https://github.com/yusinto/relay-compiler-plus){:target="_blank"}.
This way we'll get schema stitching plus persisted queries for free! You can still use the standard relay-compiler,
but you won't get persisted queries (not yet anyway. I have submitted a [pr](https://github.com/facebook/relay/pull/2354){:target="_blank"}) 
and you can't compile directly from graphql-js.

{% highlight bash %}
yarn add -D relay-compiler-plus
{% endhighlight %}

Add the following npm command to your package.json:
{% highlight bash %}
"rcp": "relay-compiler-plus --webpackConfig path/to/webpack.config.js --src src",
{% endhighlight %}

Ensure the webpack entry is pointing to mergedSchema.js from step 3. Then run:
{% highlight bash %}
npm run rcp
{% endhighlight %}

This should produce the standard relay query files plus a queryMap.json file under src. You'll see in queryMap.json that 
the remote query exists as if it's local!

## Conclusion
It is possible to use relay and schema stitching. It is surprisingly straight forward thanks to the tools provided
by Apollo. It is impossible to document every single step in this blog without boring everyone to death. People don't read
blogs more than 4 mins long these days... so perhaps it's best to see it in action. 

Check out the full project on [github](https://github.com/yusinto/relay-compiler-plus/tree/master/example-stitching){:target="_blank"}.

Happy stitching!

![Loui stitch?](/assets/images/loui-stitch.jpg)


---------------------------------------------------------------------------------------
