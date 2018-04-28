---
published: true
title: "Relay Schema Stitching"
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
- graphcool

blog: true
---

Schema stitching was introduced by Apollo, so there is a general misconception that it
only works with the apollo stack but it works with relay as well! In this blog
I'll show you how to stitch your relay schema with any remote graphql schema.

You can find the complete project in [github](https://github.com/yusinto/relay-compiler-plus/tree/master/example-stitching){:target="_blank"}.

## Goal
Given a local schema that looks like this:

<script src="https://gist.github.com/yusinto/c4c6a8f376a36f600ddb0f1f24c952df.js"></script>

Stitch it so that business details are coming from a remote graphql server, in this case from [prisma](https://www.prisma.io/){:target="_blank"}.

## Step 1: Install graphql-cli
We'll use the graphql cli to download the remote schema:
 
{% highlight bash %}
yarn add -D graphql-cli
{% endhighlight %}

## Step 2: Download the remote schema
Graphql cli looks for a .graphqlconfig.yml by default, so we'll create that file
and it looks like this:

<script src="https://gist.github.com/yusinto/7cfbeeac5e0b6a35dde44a72e5e1a268.js"></script>

In this example, I have set up a graphql server with [prisma](https://www.prisma.io/){:target="_blank"} 
but you can use any graphql endpoint. Prisma is created by the guys from graphcool, check it out if you haven't!

Add the following npm command to your package.json:

{% highlight bash %}
"get-remote-schema": "graphql get-schema",
{% endhighlight %}

Run it to download the remote schema to ./remote.schema.graphql:
{% highlight bash %}
npm run get-remote-schema
{% endhighlight %}
 

## Step 3: Stitch
The stitching part is pretty standard, following Apollo's doco:

<script src="https://gist.github.com/yusinto/9ab425fc0a100f81c183e824f1405b57.js"></script>


## Step 4: Compile
Lastly in this example, we'll compile our relay queries using [relay-compiler-plus](https://github.com/yusinto/relay-compiler-plus){:target="_blank"}
so we'll get schema stitching plus persisted queries for free!

Add the following npm command to your package.json:
{% highlight bash %}
"rcp": "NODE_ENV=production relay-compiler-plus --webpackConfig src/server/webpack.config.js --src src",
{% endhighlight %}



---------------------------------------------------------------------------------------
