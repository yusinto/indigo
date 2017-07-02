---
published: true
title: "Writing Graphcool functions in es6 with async await and jest"
layout: post
date: 2017-06-30 08:30
tag:
- graphcool
- functions
- server
- side
- subscription
- async
- await
- es6
- graphql
- jest
blog: true
---
Graphcool is cool. Graphcool functions are even cooler. There are two types of functions:
request pipeline and server side subscription. Request pipeline function gets triggered
at a specified stage of a crud request. You write custom business logic you want to execute
as part of your api requests here. It is synchronous.

The second type of function is server side subscription. These get triggered
<b>after</b> crud operations. Your write custom business logic here to
react to crud events in your database. Server side subscriptions are
asynchronous.

In this blog, I will talk about server side subscriptions. A crud occurs
on the server and we want to execute some business logic when that happens.
The traditional solution is to do it in our own backend in node/java/.net
probably in the business logic layer. But that means we need to host our
own backend server which means we have to worry about infrastructure.
Can we do better?

Yes we can. Enter Graphcool server side subscriptions. You write your business logic directly in
the Graphcool console and specify the event which will trigger this logic.

## Goal
The goal is to create a stripe customer when a Graphcool customer is created. It's a very common
use case. The complete code is [here](https://github.com/graphcool-examples/functions/tree/master/stripe-create-customer-es6){:target="_blank"}.

Enough talk, let's code!

## Step 1: Create Graphcool schema
Create a Customer type in your Graphcool backend so we have something to crud with.

#### customer.graphql
{% highlight javascript %}
type Customer implements Node {
  email: String!
  stripeCustomerId: String
  id: ID! @isUnique
  createdAt: DateTime!
  updatedAt: DateTime!
}
{% endhighlight %}

Use the graphcool cli to create a new Graphcool project and create the Customer type.

{% highlight bash %}
npm -g install graphcool
graphcool init --schema customer.graphql
{% endhighlight %}

## Step 2: Create Graphcool server side subscription
The cli is powerful but it is still work in progress. You can't create functions
via the cli at this stage (yet). We'll create our ssr function via the console
for now. You can open the console using the cli:

{% highlight javascript %}
graphcool console
{% endhighlight %}

Go to functions -> new function -> server-side subscription -> select Customer type
as the trigger and click define function. Copy paste the subscription query
below into the left window pane under subscription query.

#### subscription.graphql
{% highlight C# %}
subscription {
  Customer(filter: {
    mutation_in: [CREATED]
  }) {
    updatedFields
    node {
      id
      email
    }
  }
}
{% endhighlight %}

Let's look at this in detail:
<ul>
<li>
The <b>subscription</b> keyword is a third operation recently added graphql.
</li>
<li>The subscription above means we are subscribing only to Customer create events.
You can also listen to UPDATED and DELETED events but we don't need those here.
</li>
<li>When a Customer is created, return the id and email of that newly created customer.</li>
</ul>

Back in the console click "Create Function". Leave inline code on the right pane
as is for now, we'll write the code for this in the next section.

## Step 3: Write code!
Finally we get to write our function code! You could have written code directly
into the console inline editor, but doing so forgoes a lot of the benefit
of your IDE. Furthermore, behind the scenes inline functions are deployed
to [webtask](https://webtask.io/){:target:"_blank"} which is cool
but does not support async await.

Optionally you can also write and host your code elsewhere (like aws lambda) and
specify that as a webhook. But this means you have to worry about hosting
your code elsewhere.

We want to be able to write es6 code with async await, linting, unit tests,
strong typing, etc in the comfort of our favourite IDE and be able to
deploy that to graphcool. To do this we have to bite the bullet and use
webpack to transpile our code. Luckily for you fellow js devs, readers and
oss fans, I've done all the hard work! I have worked out the minimal webpack
config to support async await and the latest es6 features to write Graphcool
functions.

This is not a webpack tutorial but I want to share a few interesting things I
discovered while working on this:

<ul>
<li>
The <b>subscription</b> keyword is a third operation recently added graphql.
</li>
<li>The subscription above means we are subscribing only to Customer create events.
You can also listen to UPDATED and DELETED events but we don't need those here.
</li>
<li>When a Customer is created, return the id and email of that newly created customer.</li>
</ul>

{% highlight javascript %}
const webpack = require('webpack');
const path = require('path');
const nodeExternals = require('webpack-node-externals');

// the filename of your function under ./src directory
const functionToBuild = 'createStripeCustomer';

module.exports = {
  entry: ['babel-polyfill', `./src/${functionToBuild}`],
  output: {
    path: path.join(__dirname, 'dist'),
    library: '[name]',
    libraryTarget: 'commonjs2',
    filename: `${functionToBuild}.js`
  },
  // exclude all modules from bundling because they should be provided by webtask.
  externals: [nodeExternals({
    // include these to get async await to work
    whitelist: ['babel-polyfill', 'regenerator-runtime/runtime']}
  )],
  target: 'node',
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        options: {
          presets: ['es2015'],
        }
      }
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false,
        screw_ie8: true,
        conditionals: true,
        unused: true,
        comparisons: true,
        sequences: true,
        dead_code: true,
        evaluate: true,
        join_vars: true,
        if_return: true
      },
    }),
  ]
};
{% endhighlight %}

If you are interested, you can check the complete webpack config [here](https://github.com/graphcool-examples/functions/blob/master/stripe-create-customer-es6/webpack.config.js){:target="_blank"}.

You need to install the following npm packages:

{% highlight javascript %}
yarn add --dev jest babel-plugin-rewire enzyme react-addons-test-utils
{% endhighlight %}

I use [jest](https://facebook.github.io/jest/){:target="_blank"} for my test framework and you should too, it kicks butt. 
I also use [enzyme](https://github.com/airbnb/enzyme){:target="_blank"} which is a utility library for testing react components. 
Enzyme requires the official [react-addons-test-utils](https://facebook.github.io/react/docs/test-utils.html){:target="_blank"} package.

In your .babelrc, add an "env" block to include babel-plugin-rewire when running tests:

#### .babelrc
{% highlight javascript %}
{
    "presets": ["es2015", "react", "stage-0"],
    
    /* etc your other config */
    
    "env": {
      "test": {
        "plugins": ["babel-plugin-rewire"]
      }
    }
}
{% endhighlight %}

## Write the tests!
Now we can write the tests! The complete file is [here](https://github.com/yusinto/test-react/blob/master/src/universal/home/home.test.js){:target="_blank"}:

#### home.test.js
{% highlight javascript %}
import React from 'react';
import {shallow} from 'enzyme';
import HomeRedux from './home';

describe('Home component tests', () => {
  // rewire injects __get__ and __set__ methods to all our modules.
  // These can then be used to extract and set top level private variables.
  // In this instance, we extract the private Home class
  const Home = HomeRedux.__get__('Home');

  it('should render correctly', () => {
    // Yayy! We can now render the presentational component directly! 
    const output = shallow(<Home randomNumber={45}/>);
    
    // Use jest snapshot testing for convenience
    expect(output).toMatchSnapshot();
  });
});
{% endhighlight %}

## Conclusion
With client side subscriptions, you'll use apollo with
the [subscriptions-transport-ws](https://github.com/apollographql/subscriptions-transport-ws){:target="_blank"}
to enable your js client app to "hot listen" to server changes. The server pushes notifications
to the client, which reacts to these notifications in real-time. It's super cool!

This approach incurs a little more time to setup, but I think it's worth it. We leave the code fully testable, encapsulation intact. 
This feels right for me. Also, you can apply the same technique to test react components wrapped in relay containers. It works! 

Check out the [sample code](https://github.com/yusinto/test-react){:target="_blank"} for a working example and let me know if this is useful (or not)!

---------------------------------------------------------------------------------------
