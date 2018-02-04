---
published: true
title: "Javascript Job Queues and Promises"
layout: post
date: 2017-09-18 07:30
tag:
- javascript
- promises
- job
- queues
- js
- asynchronous
blog: true
---
So you think you know how promises work? Someone ever comes to you
with a little trivia called "what's the sequence of console logs from this
piece of async code"? And no matter how hard you try you never get
it right?

I experienced this at work last week and in process learnt something new about
promises which I would like to share.

There are few (if any) that explain the scheduling aspects of Promises though.
Perhaps because it's not a hot topic. It's too low level. You need a
shot of vodka to understand it. Bla bla.. so here goes nothing.

This is not an intro to promises or how to use them. There are plenty
of blogs out there explaining that in complete pornographic detail (give me
that vodka!). No sir, today I'll be talking about the temporal aspects
of promises i.e. "when" your code gets executed and why.

It's actually very interesting!

## Goal
Understand when parts of your promise gets executed and why.

## Step 1: Anatomy of a Promise
{% highlight javascript %}
const p = new Promise(
    // this function is called the "executor"
    (resolve, reject) => {
        console.log(1);
        resolve(2);
        console.log(3);
    }
);

console.log(4);

p.then(
    // this is the success handler
    result => console.log(result)
);

console.log(5);
{% endhighlight %}

What is the output of the code above? We'll go through this in this blog.

## Step 2: The event loop
This gives javascript that infamous single-threaded reputation. It is
an infinite while true loop that continues forever. Each iteration of
this loop is called a "tick".


## Step 2: The Queue
Each tick processes an item off the queue.

## Step 3. The Job Queue


* Download [emscripten portable](https://s3.amazonaws.com/mozilla-games/emscripten/releases/emsdk-portable.tar.gz){:target:"_blank"}
* Unzip and cd into the dir and execute these:

{% highlight bash %}
./emsdk update
./emsdk install latest
./emsdk activate latest
source ./emsdk_env.sh
{% endhighlight %}

* Add the emcc executable to your /etc/paths file. Mine is
located at /your_download_dir/emsdk-portable/emscripten/1.37.16

## Step 2: Write C code
Create a file called utils.c under your src folder.

#### utils.c
{% highlight javascript %}
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <emscripten/emscripten.h>

int main(int argc, char ** argv) {
    // gets translated to console.log
    printf("WebAssembly successfully loaded!\n");
}

// Emscripten does dead code elimination during compilation.
// This decorator ensures our code does not get removed.
EMSCRIPTEN_KEEPALIVE
int generateRandom() {
    srand ( time(NULL) );
    return rand();
}
{% endhighlight %}


## Step 3: Compile your C code

{% highlight bash %}
emcc utils.c -s WASM=1 -o utils.js -O3
{% endhighlight %}

* -s Specify settings which gets passed down to the emscripten compiler. Here
we specify we want to compile to wasm. The default is asm. This will
produce utils.wasm.

* -o Specify the filename for the glue code. This will produce utils.js.

* -O3 The first character is the upper case letter 'O' not zero! Sets the optimisation
level for your wasm and js files. You can check the various optimisation levels
[here](https://kripken.github.io/emscripten-site/docs/tools_reference/emcc.html#emcc-o0){:target="_blank"}.

## Step 4: Add the glue code to your html

{% highlight html %}
<!DOCTYPE html>
    <html>
         <head>
            <title>Hasta la vista JS!</title>
            <meta name="viewport" content="width=device-width, initial-scale=1">
          </head>
          <body>
            <div id="reactDiv"/>
            <script src="/dist/utils.js"></script>
            <script src="/dist/bundle.js"></script>
          </body>
    </html>
{% endhighlight %}

## Step 5: Add an express route to serve wasm files

Express does not serve .wasm files by default so we have to add a custom route.

#### server.js
{% highlight javascript %}
app.get('/:filename.wasm', (req, res) => {
  const wasmFilePath = path.resolve(__dirname, 
    `../../dist/${req.params.filename}.wasm`);
  
  console.log(`Wasm request ${wasmFilePath}`);

  fs.readFile(wasmFilePath, (err, data) => {
    const errorMessage = `Error ${wasmFilePath} not found. ${JSON.stringify(err)}`;
    if (err) {
      console.log(errorMessage);
      res.status(404).send(errorMessage);
      return;
    }
    res.send(data);
  });
});
{% endhighlight %}

## Step 6: Call wasm from React!

Finally! You can now use your C function from React by prefixing an underscore
in front of the C function's name. We included the glue code in our app html, so
all your C methods are exposed globally. This is not the best way, but 
in the future, webpack will rescue us. There is [wip](https://medium.com/webpack/webpack-awarded-125-000-from-moss-program-f63eeaaf4e15){:target="_blank"}
right now sponsored by Mozilla to develop first class support for WebAssembly in webpack. 
This means we'll be able to import C/C++ files directly in js files and
call wasm functions directly! 

Till that day arrives, a global script tag will have to do for now.

#### app.js
{% highlight javascript %}

export default class App extends Component {
    state = {randomNumber: -1};
    
    onClickGenerateRandom = () => {
      // EUREKA! Call our C function with an underscore prefix!
      // All the methods in utils.c are exposed globally because utils.js
      // is included as a script tag in our html.
      const randomNumber = _generateRandom();
      console.log(`onClickGenerateRandom: ${randomNumber}`);
      this.setState({randomNumber});
    }; 
      
    render() {
      return (
        <div>
          <button onClick={this.onClickGenerateRandom}>
            Generate random
          </button>
          {this.state.randomNumber}
        </div>
      );
    }
}
{% endhighlight %}

## Conclusion
The next step is to help Sean Larkin and co to get webpack support WebAssembly!

The complete code is [here](https://github.com/yusinto/wasm-playground){:target="_blank"}
as usual. Start learning C/C++. Enjoy!

---------------------------------------------------------------------------------------
