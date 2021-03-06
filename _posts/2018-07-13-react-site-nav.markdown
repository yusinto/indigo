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

Stripe has a beautiful site nav and this package is inspired by that.
Introducing react-site-nav, a beautifully animated site nav powered by styled components
and css animations. Play with the [live demo](https://react-site-nav.now.sh){:target="_blank"} powered by now
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

It takes only a few lines of code to get up and going, but it still looks very basic.
Let's make it pretty! First, set our SiteNav to debug mode so the content group stays open
when we hover over it:

{% highlight javascript %}
<SiteNav debug={true}>
{% endhighlight %}

Let's get rid of those ugly default list style, margin and padding:

{% highlight css %}
ul {
    list-style-type: none;
    margin: 0;
    padding: 0;
}
{% endhighlight %}

Next instead of a bullet point, let's have an image next to our text. Our jsx becomes:

<script src="https://gist.github.com/yusinto/840ecdba5ce0d8f4bf85fa11ae2a4e51.js"></script>

Let's use flex for our list item so we can easily center everything:

{% highlight css %}
li {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 60px;
}
{% endhighlight %}

It's starting to look good! Just some finishing touches now to make the images and text vertically
aligned and some hover effects:

{% highlight css %}
li:hover {
    opacity: 0.7;
}

li > span {
    flex: 0 0 100px;
    text-align: left;
    margin-left: 10px;
}
{% endhighlight %}

## Tadaa
![Before and after](/assets/images/before-after.png)

Check out the live demo [here](https://build-yzcyfqkxqz.now.sh){:target="_blank"}. The complete stylesheet:

<script src="https://gist.github.com/yusinto/9a04ad983ff2b03a140683d45ef9405b.js"></script>

## Next steps
There are still loads left to do, like mobile and sizing near edges. I'll get to those in time!

For more, check out [github](https://github.com/yusinto/react-site-nav){:target="_blank"}. There are three fully
working spas including the code in this blog in the [examples](https://github.com/yusinto/react-site-nav/tree/master/examples){:target="_blank"}
folder. The code in this blog is under [examples/cra-with-nav](https://github.com/yusinto/react-site-nav/tree/master/examples/cra-with-nav){:target="_blank"}.

Please star it if you like it! Thanks.

## Thanks
[Max Stoiber](https://mxstbr.com/){:target="_blank"} is awesome.
[Styled components](https://www.styled-components.com/){:target="_blank"} is awesome.
[Now](https://zeit.co/now){:target="_blank"} is awesome. Thanks.

---------------------------------------------------------------------------------------
