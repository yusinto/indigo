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

## Step 2: xxx

bla bla
 
## Step 3: Make it pretty

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

{% highlight c# %}
import aboutMeImage from './about-me.png';

  // ... other code omitted for brevity
  <ul>
    <li>
      <img src={aboutMeImage} height="40"/>
      <span>About me</span>
    </li>
  </ul>
{% endhighlight %}

Let's make our list item use flex so we can easily center everything:

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

## Result
![Before and after](/assets/images/before-after.png)

The complete stylesheet:

<script src="https://gist.github.com/yusinto/9a04ad983ff2b03a140683d45ef9405b.js"></script>

## Next steps
There are still loads left to do, like mobile and sizing near edges. I'll get to those in time!

For more, check out [github](https://github.com/yusinto/react-site-nav){:target="_blank"}. There are three fully
working spas including the code in this blog in the [examples](https://github.com/yusinto/react-site-nav/tree/master/examples){:target="_blank"} 
folder. The code in this blog is under [examples/cra-with-nav](https://github.com/yusinto/react-site-nav/tree/master/examples/cra-with-nav){:target="_blank"}.

Please star it if you like it! Thanks.

---------------------------------------------------------------------------------------
