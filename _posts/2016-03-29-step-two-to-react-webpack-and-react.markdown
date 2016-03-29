---
published: true
title: "Step 2 to React: Webpack and React"
layout: post
date: 2016-03-29 16:48
tag:
- react
- webpack
blog: true
---

If you haven't read my [previous post](http://www.reactjunkie.com/step-one-to-react-es-6-and-express/), 
it might be a good idea to have a quick read before starting this one. We'll build this example based on that last project.
Before we can start writing react code, we'll need to setup webpack to compile react code into pure javascript that
browsers can understand.

1. npm install --save-dev webpack babel-loader
   npm install --save react react-dom babel-preset-react
2. Add webpack.config.dev.js. Configure webpack (basic)
3. Modify package.json:
 "prestart": "webpack --config webpack.config.dev.js --progress",
4.  Add ./src/common/component/appComponent.js
    Add ./src/client/index.js
5. Add htmlString template to server.js.
6. Add static serve to '/dist' to server.js
7. Run npm start

---

## Step 2.1: Install webpack, babel and react
 You'll need to install express, babel-express and babel-preset-es2015:

{% highlight js %}
npm install --save-dev webpack babel-loader
npm install --save react react-dom babel-preset-react
{% endhighlight %}

<ul>
<li>webpack is an all powerful module bundler which bundles all our react code into a single file</li>
<li>babel-loader is the transpiler that transforms jsx into javascript</li>
<li>react is the core module for react (duh)</li>
<li>react-dom contains all dom related stuff for react</li>
<li>babel-preset-react allows babel to transform jsx into javascript</li>
</ul>
---

## Step 2.2: Configure webpack
Create a new file called webpack.config.dev.js at the root directory of your project. The file contents should look like this:

{% highlight js %}
var webpack = require('webpack');
var path = require('path');

module.exports = {
    entry: [path.join(__dirname, 'src/client/index')],
    output: {
        path: path.join(__dirname, 'dist'),
        filename: 'bundle.js'
    },
    module: {
        loaders:[{
            test: /\.jsx?$/,
            loader: 'babel',
            include: path.join(__dirname, 'src')
        }]
    }
};
{% endhighlight %}

This configures babel to transpile es6 code to es5.

---

## Step 2.3: Configure an entry point in package.json

In your package.json, add a scripts/prestart command which tells npm to run a compile task prior to the main "start" task:

{% highlight js %}
{
...
      "scripts": {
        "prestart": "webpack --config webpack.config.dev.js --progress",
        "start": "node src/server/index.js",
      }
...
}
{% endhighlight %}

The "prestart" task gets run first prior to the start task. That prestart task compiles our code and dump a static js file 
onto the dist folder.

---

## Step 2.4: Write some react code
Finally we get to write some react code! Let's create two new files:

<ul>
    <li>src/common/component/appComponent.js</li>
    <li>src/client/index.js</li>
</ul>

appComponent.js

{% highlight C# %}
import React, {Component} from 'react';

export default class App extends Component {
    render() {
        return (
            <div>
                Hello world in React! 
                The time now is { (new Date()).toLocaleString() }
            </div>
        );
    }
}
{% endhighlight %}

index.js
{% highlight C# %}
import React from 'react';
import {render} from 'react-dom';
import App from '../common/component/appComponent';

render(<App />, document.getElementById('reactDiv'));
{% endhighlight %}

---

## Step 2.5: Modify express to serve react
We need to modify server.js to serve a html content with a script reference to our bundled js file.
We also need to add an express static middleware to serve that static bundle.js file.

{% highlight c# %}

...

const htmlString = `<!DOCTYPE html>
    <html>
         <head>
            <title>Webpack and React</title>
          </head>
          <body>
            <div id="reactDiv" />
            <script type="application/javascript" src="/dist/bundle.js"></script>
          </body>
    </html>`;

app.use('/dist', Express.static('dist'));

...
{% endhighlight %}

---

## Step 2.6: Run the app!
At your root directory run "npm start" and browse to localhost:3000 to see the output. 
Download the complete source code from [github](https://github.com/yusinto/reactStep2).

---------------------------------------------------------------------------------------
