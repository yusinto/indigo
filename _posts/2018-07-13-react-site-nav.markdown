---
published: true
title: "Introducing react-site-nav"
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

[Stripe](https://stripe.com){:target="_blank"} has a beautiful site nav and this package is inspired by that.
Introducing react-site-nav, a beautifully animated site nav powered by [styled components](https://www.styled-components.com/){:target="_blank"}
and css animations. Play with the [live demo](https://now-evztwufdfm.now.sh){:target="_blank"} powered by [now](https://zeit.co/now){:target="_blank"}
or check out the video below.

<p align="center">
{% youtube 4fThkT_vlBE %}
</p>

## Goal
Let's use react-site-nav and add a kick ass nav to create-react-app!

## Step 1: Install

We'll create a new create-react-app project and install react-site-nav the usual way:

{% highlight bash %}
create-react-app cra-with-nav
cd cra-with-nav
yarn add react-site-nav
{% endhighlight %}

## Step 2: Adding SiteNav and ContentGroup

Good stuff. Now we are going to add two components from react-site-nav to App.js: SiteNav and ContentGroup.

<script src="https://gist.github.com/yusinto/c53edbc178d9dd3289c1a80050e9f20f.js"></script>

SiteNav is the root react component that contains ContentGroup children.
Each ContentGroup can accept 3 props: title, width and height.

<p align="center">
<img src="/assets/images/react-site-nav-content-group.png" width="400"/>
</p>


## Step 3: Making it pretty

As you can see, it's easy to get up and going, but it looks pretty ordinary. Let's make it
look better!

The withFlags hoc sets up a context consumer which passes flags to the wrapped component. Your flags will then be
available as camelCased keys under `this.props.flags.yourFeatureFlag`. 

<script src="https://gist.github.com/yusinto/195df27bbba1044c3773a9c4a86db057.js"></script>

That's it! For more, check out [github](https://github.com/yusinto/ld-react){:target="_blank"}. There is also a fully
working spa [example](https://github.com/yusinto/ld-react/tree/master/example){:target="_blank"} with react router 4 and
ssr. 

Happy coding!

---------------------------------------------------------------------------------------
