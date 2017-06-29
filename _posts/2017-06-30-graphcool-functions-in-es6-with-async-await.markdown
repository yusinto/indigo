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
Graphcool is cool. Graphcool subscriptions are even cooler. There are two types of subscriptions:
client side and server side subscriptions.

With client side subscriptions, you'll use apollo with
the [subscriptions-transport-ws](https://github.com/apollographql/subscriptions-transport-ws){:target="_blank"}
to enable your js client app to "hot listen" to server changes. The server pushes notifications
to the client, which reacts to these notifications in real-time. It's super cool!

In this blog, I will talk about server side subscriptions. It's all on the server, there's no client
involvement. A crud occurs on the server and we want to execute some business logic when that happens.
The traditional solution is to do it in our own backend in node/java/.net and probably in the business
logic layer. But that means we need to host our own backend server which means we have to worry about
infrastructure. Can we do better?

Yes we can. Enter Graphcool server side subscriptions. You write your business logic directly in
the Graphcool console and specify the event which will trigger this logic.

## Enough talk, show me some code
The goal is to create a stripe customer when a Graphcool customer is created. It's a very common
use case. The complete code is [here](https://github.com/graphcool-examples/functions/tree/master/stripe-create-customer-es6){:target="_blank"}.

## Step 1: Setup Graphcool schema
Create a Customer type in your Graphcool backend so we have something to crud with.
Call this file customer.graphql.

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

{% highlight javascript %}
import React, {Component} from 'react';
import {connect} from 'react-redux';
import * as Actions from './homeAction';

// private class
class Home extends Component {
  constructor(props) {
    super(props);
    this.onClickGenerateRandom = ::this.onClickGenerateRandom;
  }

  onClickGenerateRandom() {
    this.props.generateRandom();
  }

  render() {
    let homeText = 'Click button below to generate a random number!';

    return (
      <div>
        <p>{ homeText }</p>
        <div>{this.props.randomNumber}</div>
        <button onClick={this.onClickGenerateRandom}>
            Generate random number
        </button>
      </div>
    );
  }
}

const mapStateToProps = (state) => {
  const homeState = state.Home;

  return {
    randomNumber: homeState.randomNumber
  };
};

export default connect(mapStateToProps, Actions)(Home);
{% endhighlight %}

The Home class is private and that's what we want to test. It is not exported at all, so we can't access it directly. 
As mentioned above, the official redux documentation recommends exporting this class, but that breaks encapsulation.
So what do we do? Enter [babel-plugin-rewire](https://github.com/speedskater/babel-plugin-rewire){:target="_blank"}.

## Using rewire
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
This approach incurs a little more time to setup, but I think it's worth it. We leave the code fully testable, encapsulation intact. 
This feels right for me. Also, you can apply the same technique to test react components wrapped in relay containers. It works! 

Check out the [sample code](https://github.com/yusinto/test-react){:target="_blank"} for a working example and let me know if this is useful (or not)!

---------------------------------------------------------------------------------------
