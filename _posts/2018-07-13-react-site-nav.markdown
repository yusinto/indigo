---
published: true
title: "Building a site nav that doesn't suck"
layout: post
date: 2018-07-13 07:30
tag:
- react
- site
- nav
- navigation
- bar
- navbar
- stripe
- styled
- components
- menu
- css
- animations
- grid
- cssgrid

blog: true
---
Let's start with a question: hands up who here wakes up in the morning feeling inspired to go to work and build a nav bar?
A few hands? Maybe. I predict the majority of us would prefer more exciting stuff like graphql or react-native-dom.
Building nav bars is sooo boring just because it's been done so many times before! And all the time (or most of the time) very
badly done as well! It's a vicious cycle. 

No one cares about the nav bar. It's a neglected child. Just use the default bootstrap
one says one dev. Just build a really quick and simple one that does the job says the product manager. It's always a non-priority
item in the backlog. Animations? Usability? UX? Somehow we ignore all these things when it comes to the navbar. 
After all, who cares right?

Well I do. No more crappy nav bars. Introducing [react-site-nav](https://github.com/yusinto/react-site-nav){:target="_blank"},
a kick ass fully animated fully customisable site nav bar without compromise. Powered by [styled components](https://www.styled-components.com/){:target="_blank"},
css grid and animations and inspired by [stripe.com](https://stripe.com){:target="_blank"}.

##

## Step 1: Install

You need react ^16.4 to use ld-react.

{% highlight bash %}
yarn add ld-react
{% endhighlight %}

## Step 2: Wrap your root app withFlagProvider

The withFlagProvider hoc initialises an ldClient object on componentDidMount and sets up subscriptions to all flags.
It then uses the context api to pass flag values to consumers. 

<script src="https://gist.github.com/yusinto/a31074588c26c2ce747505bcbd49400b.js"></script>

You can also pass a user object and options as part of the second parameter in addition to clientSideId. However, they are
not mandatory.
 
## Step 3: Where you need flags, wrap that component withFlags

The withFlags hoc sets up a context consumer which passes flags to the wrapped component. Your flags will then be
available as camelCased keys under `this.props.flags.yourFeatureFlag`. 

<script src="https://gist.github.com/yusinto/195df27bbba1044c3773a9c4a86db057.js"></script>

That's it! For more, check out [github](https://github.com/yusinto/ld-react){:target="_blank"}. There is also a fully
working spa [example](https://github.com/yusinto/ld-react/tree/master/example){:target="_blank"} with react router 4 and
ssr. 

Happy coding!

---------------------------------------------------------------------------------------
