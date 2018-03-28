---
published: true
title: "Javascript Lessons: toJSON and valueOf "
layout: post
date: 2018-03-28 07:30
tag:
- javascript
- tojson
- valueof
- json
- stringify
- lessons
blog: true
---

## toJSON()
If you need a custom output when json stringifying your object, you can define
a function called toJSON() which will be invoked automatically by
JSON.stringify():

<p data-height="376" data-theme-id="dark" data-slug-hash="KoZmLa" data-default-tab="js,result" data-user="yusinto" data-embed-version="2" data-pen-title="Javascript Lessons" class="codepen">See the Pen <a href="https://codepen.io/yusinto/pen/KoZmLa/">Javascript Lessons</a> by Yusinto Ngadiman (<a href="https://codepen.io/yusinto">@yusinto</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

Without the custom toJSON() implementation, the code above will use the default
implementation and output:

{% highlight json %}
{"firstName":"Yus","lastName":"Ng"}
{% endhighlight %}

## valueOf()

<p data-height="372" data-theme-id="dark" data-slug-hash="YaYQyo" data-default-tab="js,result" data-user="yusinto" data-embed-version="2" data-pen-title="Javascript Lessons: valueOf" class="codepen">See the Pen <a href="https://codepen.io/yusinto/pen/YaYQyo/">Javascript Lessons: valueOf</a> by Yusinto Ngadiman (<a href="https://codepen.io/yusinto">@yusinto</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>


---------------------------------------------------------------------------------------
