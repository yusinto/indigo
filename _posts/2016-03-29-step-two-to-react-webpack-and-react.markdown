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
it might be a good idea to have a quick read before reading this one. We'll build this example based on that last post.
Let's get into it!

---

## Step 2.1: Install webpack, babel and react
{% highlight js %}
npm install --save-dev webpack babel-loader
npm install --save react react-dom babel-preset-react
{% endhighlight %}

Breaking it down part by part:
<p>
We use webpack to bundle all our javascript code into a single file. We include this file in a script tag in our html
just like any other js file and this alone will be sufficient to run our app in the browser.
</p>
<p>
Our code is in es6 and jsx (we'll talk about jsx in detail a bit later) which are not (yet) understood by browsers. So
we have to transpile (short for transform and compile) it into pure javascript which are understood by browsers. We use
babel to perform this transpilation.
</p>
<p>
react and react-dom are core react modules required to write react apps. babel-preset-react instructs babel (both
babel-register and babel-loader) to transpile jsx into pure javascript.
</p>

---

## Step 2.2: Configure webpack
Create a new file called webpack.config.dev.js at the root directory of your project. The file contents should look like this:

{% highlight js linenos %}
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

There are 3 main parts to this config:
<ul>
<li>entry: Tells webpack to start bundling from this file</li>
<li>output: Write the resultant bundled file to a dist folder in root</li>
<li>module.loaders: For all js or jsx files under src folder, use babel-loader to transpile those files prior to bundling</li>
</ul>

---

## Step 2.3: Configure an entry point in package.json

In your package.json, add a scripts.prestart command. This is a natively supported npm command, just like start.
When you run npm start, the prestart command will always get exected first, then the start command, followed by a
poststart command which we don't use here. In prestart, we tell npm to run webpack to bundle our code prior to starting
the app.

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
Finally we get to write some react code! This is the meatiest part, so it's a bit longer than the rest of the post, but
it's worth it. Let's create two new files:

<ul>
    <li>src/common/component/appComponent.js</li>
    <li>src/client/index.js</li>
</ul>

Note about directory structure: All my code are in a src folder to separate it from other elements of the project
like node_modules and dist. This allows me to target only src for transpilation in my webpack build. Under src, my files
are further organised into client, server and common. What's common? This is in preparation for building a universal
app (also known as isomorphic app) where the code being run on the client and server are one and the same ergo common
code. But we'll talk more about this more in a later post.

If you come from an OO background like me, you should be familiar with the syntax here.

{% highlight C# %}
/*
 Here we import react's package default module and assign it to a variable named React.
 Additionally we also import the Component module and assign it to a variable named Component.
*/
import React, {Component} from 'react';

/*
 Subclass React.Component and implement the render method. This method must return a single child element.
 A react component at minimum must implement the render method.
 Also set this class as the default export of this file so we can import it from other files.
*/
export default class App extends Component {
    render() {
        return (
            // Nooo this looks like inline html! Are we back in the land of classic asp and php? Short answer is no we are not.
            // These might look like plain html, but under the bonnet they are shorthand syntax to generate ReactElements.
            // A ReactElement is the primary basic type in React which constitutes the virtual DOM. In essence, you are
            // writing virtual DOM. It's called virtual because it's not the real DOM. React keeps an in-memory copy
            // of this DOM subtree and only flush changes to the real DOM in the browser if there is a props or state
            // change. We'll talk about props and state more in later posts. For now, just understand that you are writing
            // a html-like syntax called jsx which becomes part of the virtual dom.
            <div>
                <h1>Hello world in React!</h1>
                <p>
                    The time now is { (new Date()).toLocaleString() }
                </p>
            </div>
        );
    }
}
{% endhighlight %}

index.js
{% highlight C# %}
//We need the render method of the react-dom package to mount our component onto an html element
import React from 'react';
import {render} from 'react-dom';
import App from '../common/component/appComponent';

// This is the entry point into our react app on the client side. Again we use jsx to create our ReactElement to be
// mounted onto a div called reactDiv on the html template.
render(<App />, document.getElementById('reactDiv'));
{% endhighlight %}

---

## Step 2.5: Modify express to serve react
Now we need to modify server.js to serve a html page with a script reference to our dist/bundled.js generated by webpack.
We also need to add an express static middleware to serve that static bundle.js file. A middleware is just code that executes
between a request and a response. In this example, a GET request comes in from the client asking for dist/bundled.js. Our
 middleware matches the route and executes our code. We use express's built-in static middleware to do this so we get this
 for free.

{% highlight c# %}

...

// This is our html template that contains a div with id="reactDiv" for our react app to mount as well as a script
// reference to dist/bundle.js
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

// Use express's built-in static middleware to serve static files in the dist folder
app.use('/dist', Express.static('dist'));

...
{% endhighlight %}

---

## Step 2.6: Run the app!
At your root directory run "npm start" and browse to localhost:3000 to see the output.
Download the complete source code from [github](https://github.com/yusinto/reactStep2).

---------------------------------------------------------------------------------------
