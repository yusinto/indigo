---
title: "Step 2 to React: Webpack and React"
layout: post
date: 2016-03-29 16:48
tag:
- react
- webpack
blog: true
---

If you haven't read my previous post, it might be a good idea to have a quick read. We'll build this example on that last project.
Before we can start writing react code, we'll need to setup webpack to compile react code into pure javascript that
browsers can understand.

1. npm install --save-dev webpack babel-loader@6.2.0
   npm install --save react react-dom babel-preset-react
2. Add webpack.config.dev.js. Configure webpack (basic)
3. Modify package.json:
 "prestart": "webpack --config webpack.config.dev.js --progress",
4.  Add ./src/common/component/appComponent.js
    Add ./src/client/index.js
5. Add htmlString template to server.js.
6. Test by running npm start

---

## Step 1.1: Install babel
 You'll need to install express, babel-express and babel-preset-es2015:

{% highlight js %}
npm install express babel-register babel-preset-es2015 --save
{% endhighlight %}

Where express is the standard web framework for node, babel-register will compile every file that is require'd with babel and
babel-preset-es2015 tells babel to transpile es6 code to es5.

---

## Step 1.2: Configure babel - add a .babelrc file
Create a new file called .babelrc at the root directory of your project. The file contents should look like this:

{% highlight js %}
{
    "presets": ["es2015"]
}
{% endhighlight %}

This configures babel to transpile es6 code to es5.

---

## Step 1.3: Configure an entry point in package.json

In your package.json, add a scripts/start command which tells npm what to do when you run "npm start" in the command line:

{% highlight js %}
{
...
      "scripts": {
        "start": "node src/server/index.js",
      }
...
}
{% endhighlight %}

This tells npm to execute src/server/index.js when you run "npm start" at your root project folder. In this case, index.js 
is the entry point to your app. The contents of this file should look like this: 

{% highlight js %}
require('babel-register');
require('./server');
{% endhighlight %}

---

## Step 1.4: Write es6 code
The file server.js contains all your es6 code for your app. It should look like this:

{% highlight js %}
import Express from 'express';

const PORT = 3000;
const app = Express();

app.use((req, res) => {
    res.end('hello world!');
});

app.listen(PORT, () => {
    console.log(`Listening at ${PORT}`);
});
{% endhighlight %}

Here we use import statements in place of the classic require statements, const keyword instead of var, 
arrow functions instead of inline function declarations and es6 template strings instead of string concatenations.

---

## Step 1.5: Run your app!
Run 
 
{% highlight js %}
npm start
{% endhighlight %}
 
 at your root directory and browse to localhost:3000 to see the output of your app. Download the complete source code from
 [github](https://github.com/yusinto/reactStep1).



