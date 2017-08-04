---
published: true
title: "WebAssembly and React"
layout: post
date: 2017-08-04 07:30
tag:
- webAssembly
- react
- web
- assembly
- javascript
- c
- c++
blog: true
---
WebAssembly is the next big thing. So they say. Who knows. All I know is
that it's fast and it can make my app goes faster, like native fast. So
naturally I am interested.

I looked around for a quick guide to get WebAssembly up and running in
node, express and react but couldn't find one. So I decided to do it myself.

Let's begin!

## Goal
Run a C function from a .wasm file in a react component.

## Step 1: Install emscripten
Emscripten compiles C/C++ code to web assembly. Given a helloWorld.c file,
emscripten produces helloWorld.wasm and helloWorld.js.

The wasm is a binary file that contains your C/C++ code. You can't easily
import wasm files directly into js so emscripten also produces a js file which
acts as a proxy. You add a script reference to this file in your html so you
can use wasm in your js code! They call this js file the "glue" code. Personally I prefer proxy.

Here are the steps to install emscripten:

* Download [emscripten portable](https://s3.amazonaws.com/mozilla-games/emscripten/releases/emsdk-portable.tar.gz){:target:"_blank"}
* Unzip and cd into the dir and execute these:

{% highlight bash %}
./emsdk update
./emsdk install latest
./emsdk activate latest
source ./emsdk_env.sh
{% endhighlight %}

* Add the path to emcc executable to your /etc/paths file. Mine is
located at /your_download_dir/emsdk-portable/emscripten/1.37.16

## Step 2: Write C code
Create a file called generateRandom.c under your src folder.

#### generateRandom.c
{% highlight javascript %}
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <emscripten/emscripten.h>

int main(int argc, char ** argv) {
    printf("WebAssembly module loaded\n");
}

int EMSCRIPTEN_KEEPALIVE generateRandom() {
    srand ( time(NULL) );
    return rand();
}
{% endhighlight %}


## Step 3: Compile your C code

{% highlight bash %}
emcc generateRandom.c -s WASM=1 -o generateRandom.js -O3
{% endhighlight %}

* -s Specify settings which gets passed down to the emscripten compiler. Here
we specify we want to compile to wasm. The default is asm. This will
produce generateRandom.wasm.

* -o Specify the filename for the glue code. This will produce generateRandom.js

* -O3 The first character is the upper case letter 'O' not zero! Sets the optimisation
level for your wasm and js files. You can check the various optimisation levels
[here](https://kripken.github.io/emscripten-site/docs/tools_reference/emcc.html#emcc-o0){:target="_blank"}

## Step 4: Add a script tag to your html file

#### index.html
{% highlight html %}
<html>
   <head>
       <!-- other stuff here -->
       <script src="/dist/generateRandom.js" />
   </head>

</html>
{% endhighlight %}

## Step 5: Add an express route to serve wasm files

Express does not serve wasm files by default so we'll have to add a custom route
to serve wasm files.

#### server.js
{% highlight javascript %}
app.get('/:filename.wasm', (req, res) => {
    // add code here
});
{% endhighlight %}

## Step 6: Call wasm from react!

Finally! You can call your C function from react by prefixing an underscore
in front of its name. We included the glue code in the html template, so
all your C methods are exposed globally. This is not the best way, but it suffices
for this demo.

#### app.js
{% highlight javascript %}

export default class App extends Component {
    render() {
        // Eureka moment: call your C function with an underscore prefix!
        const random = _generateRandom();

        return <div>{random}</div>;
    }
}
{% endhighlight %}

## Conclusion
Check out the [sample code](https://github.com/yusinto/wasm-playground){:target="_blank"}
for a working example and let me know if this is useful (or not)!

---------------------------------------------------------------------------------------
