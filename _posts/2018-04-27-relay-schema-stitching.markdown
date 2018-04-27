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

Schema stitching was introduced by apollo, so there is a misconception that it
only works with the apollo stack but it works with relay as well! In this blog
I'll show you how to stitch your relay schema with a remote graphql schema.


## Step 1: Install graphql-cli

{% highlight bash %}
yarn add -D graphql-cli
{% endhighlight %}

## Step 2: Download the remote schema

{% highlight bash %}
"get-remote-schema": "graphql get-schema -p remote",
{% endhighlight %}

## Step 3: Stitch
Use apollo graphql-tools to stitch your schema with the remote schema you just
downloaded:


const remoteSchema = makeRemoteExecutableSchema({
  schema: makeExecutableSchema({typeDefs: remoteTypeDefs}),
  fetcher,
});
const result = mergeSchemas({
  schemas: [rcpSchema, remoteSchema],
});

export default result;

Lastly compile it using relay compiler plus:
"remote-rcp": "npm run remote && NODE_ENV=production relay-compiler-plus --webpackConfig src/server/webpack.config.js --src src",

## toJSON()
If you need a custom output when json stringifying your object, you can define
a function called toJSON() which will be invoked automatically by
JSON.stringify():

<p data-height="376" data-theme-id="dark" data-slug-hash="KoZmLa" data-default-tab="js,result" data-user="yusinto" data-embed-version="2" data-pen-title="Javascript Lessons" class="codepen">See the Pen <a href="https://codepen.io/yusinto/pen/KoZmLa/">Javascript Lessons</a> by Yusinto Ngadiman (<a href="https://codepen.io/yusinto">@yusinto</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

Without the custom toJSON() implementation, the code above will use the default
implementation:

{% highlight json %}
{"firstName":"Yus","lastName":"Ng"}
{% endhighlight %}

## valueOf()
Object.prototype has a built-in valueOf method which returns the primitive value
of the object. You can override this method to return a custom primitive value
for your object:

<p data-height="372" data-theme-id="dark" data-slug-hash="YaYQyo" data-default-tab="js,result" data-user="yusinto" data-embed-version="2" data-pen-title="Javascript Lessons: valueOf" class="codepen">See the Pen <a href="https://codepen.io/yusinto/pen/YaYQyo/">Javascript Lessons: valueOf</a> by Yusinto Ngadiman (<a href="https://codepen.io/yusinto">@yusinto</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

The override allows us to perform arithmetic with our object. Without the override,
the valueOf myCar will be NaN (Not-a-Number).


---------------------------------------------------------------------------------------
