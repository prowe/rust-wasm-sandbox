So last week I was working on a project and thinking about linting and how forcing a code style removes a lot of decision making within a team (though there are downsides as well). Later on in the day a teammate of mine posted a bash script into our Slack channel that had a couple functions defined in it.

In bash functions are called using the syntax:
```bash
functionName arg1 arg2 arg3
```
Basicly the same syntax as executing a program. This got me thinking about the idea of a new language that had a super simple syntax that avoided all the distractions of syntatic sugar and varying formats. The idea would be to have each line start with a "command" followed by 0 or more whitespace-separated arguments. If a line is indented then it is included in the line above as more arguments. So for example:
```
print "hello world"
+ 2 2
```

There is a lot missing from a language like the above. Though a language that consists mainly of function defitions and calls like [Lisp](https://en.wikipedia.org/wiki/Lisp_(programming_language)) but with a more readable style is an interesting concept. It also reminded me of another language, [Assembly](https://en.wikipedia.org/wiki/Assembly_language) (ASM) is strucutred mainly as commands (called opt codes) and arguments. That reminded me of [Web Assembly](https://webassembly.org). Sounds like a good excuse to dive into a new tech stack.

I went searching for Web Assembly introductions. I kinda wanted to write raw web assembly (like traditional ASM) but got distracted by compiling down [Rust](https://www.rust-lang.org) (an interesting language that is compiled and uses a somewhat uncommon memory model).

### Let's work through (Wasm by Example)[https://wasmbyexample.dev]. Specifically, the [Hello World Rust Example](https://wasmbyexample.dev/examples/hello-world/hello-world.rust.en-us.html).

I decided I didn't want to do a bunch of setup on my machine, so I fired up [VS Code](https://code.visualstudio.com) and opened a folder using a Rust [Dev Container](https://code.visualstudio.com/docs/remote/containers).

I won't repeat the instructions for the example here. I'm just following along with the Wasm by Example site.

Installing packages via cargo (the rust package manager) takes a while as it seems to download and then compile them all. Good time to go for a walk or make some coffee.

It seems that the gist of the WASM architecture is to compile down a bunch of code (ie. Rust) into a WASM module and that module "exports" functions that can be called from Javascript. This reminds me of the way [Java invokes native methods using JNI](https://en.wikipedia.org/wiki/Java_Native_Interface).

Looking at the `pkg` directory that was created. It automatically created a `.gitignore` file in there. I like that I don't have to remember to ignore these build artifacts. It also created a type definition file for the exproted rust functions. That's handy too.

I can't just open `index.html` in a browser because Safari will refuse to load local resources for security reasons. When working on Javascript projects I usually run a simple webserver using `npx` but I didn't want to install that in the container or have to run a mix of in and out of continer stuff. So I went looking for a Rust based web server. Gave up quicky but found the dev container has Python so I ran `python -m SimpleHTTPServer` that I found on [Stack Overflow](https://unix.stackexchange.com/a/32200) and then forwarded port 8000.

Now I'm getting a 404 in the browser. Turns out the example had the wrong file names (mine are prefixed with `rust_`) so fixed those names and now we're cooking. It seems the file names are based off of the name of the project which is based off the directory name which for me is "rust-hello-world"

It seems a bit odd that we import a JS module, call init on it (an async method) and we have to pass the wasm module to init. I can't think of a scenerio where I would ever want to pass a different wasm module to init since it is specific to my exported functions.

Fun fact: my wasm module is only 180 bytes. Pretty small so it must not be packaging a lot of extra boilerplate. The JS binding file is 1.7k though but it is pretty printed and not minified.

### With that complete, I'm moving on to [WASM Exports](https://wasmbyexample.dev/examples/exports/exports.rust.en-us.html).

So the Wasm by Example site says "Let's import the wasm initialization module from pkg/exports.js" but after running `wasm-pack build --target web` again, I don't have an `exports.js` file in my `pkg` directory. That's probably because I'm using the same project and the example probably assumes a project name of "exports". Bad name for the project on the part of the Wasm by Example site since I mistaked it for a static name that was just always created by the wasm compiler. Knowing this, I can just use the appropriate name (`rust_hello_world` for me) where appropriate.

Everything looking good. Moving on to [WebAssembly Linear Memory Overview](https://wasmbyexample.dev/examples/webassembly-linear-memory/webassembly-linear-memory.rust.en-us.html):

Waoh, nothing like dealing directly with memory via pointers to act like ice water to the face to wake us up. I also don't know much about Rust, but a code in any language inside of a keyword called ["unsafe"](https://doc.rust-lang.org/reference/unsafe-blocks.html) makes me take pause.

I'm wondering how much reading and writing to a shared buffer is necessary in the real word? Seems to work as advertised though. I wonder what would happen if I tried to read beyond the length of the array on the JS side:
```Javascript
// Printed "Beyond the end: 0" for me
console.log('Beyond the end: ', wasmMemory[bufferPointer + 2]);
```

Oh, the `new Uint8Array(rustWasm.memory.buffer);` line is Uint8Array wrapping the **entire memory space** of the WASM module...that's cool, let's print it out:
```Javascript
// Printed: ArrayBuffer {byteLength: 1114112}
console.log(rustWasm.memory.buffer);
```

I wonder if there is a way to control how much memory the WASM module gets. It would be interesting to load a WASM module into a small amount of memory (maybe a couple kb) and then just read/write the entire memory space to save state.

### Moving on to the next section in the site, [Importing Javascript Functions Into WebAssembly](https://wasmbyexample.dev/examples/importing-javascript-functions-into-webassembly/importing-javascript-functions-into-webassembly.rust.en-us.html)

Just guessing that the "extern" keyword in Rust is a way to link Rust code to an external language (I guess we're just telling rust this is C code?). Then we're defining a `log` function that is bound to the `log` function on the Javascript side. I guess `js_namespace` really means "js function on a global object with this name" since Javascript doesn't have "namespaces" in usual sense of the word (ex Java package or .Net namespace).

Works as I would expect. It's nice that we can just pass a string back (and not have to deal with some char array mess or something). I'm looking forward to the "Passing High Level Data Types" section the site keeps teasing me with. Passing an array or object would be pretty handy.

### Next it looks like we are going to do some [rendering to a canvas](https://wasmbyexample.dev/examples/reading-and-writing-graphics/reading-and-writing-graphics.rust.en-us.html) to demo the performace of WASM

Reading ahead through this module. I kinda didn't like passing around 3 8-bit integers to represent a color. It would be a lot more convinent to pass a single value. However, at the end the advantage is that it can be just handed to the canvas API directly. In fact, looking into the API I think there is a simpler way to implement the JS side of the example code. I'll resist the urge to refactor the Rust side into something more readable (large functions, especially those that are setting and then mutating variables are just screaming to have peices refactored out). I removed a bunch of unneeded canvas clearing code and simplifed the generate method to this:
```Javascript
const drawCheckerBoard = () => {
    const checkerBoardSize = 20;
    const checkerBoardByteLength = checkerBoardSize * checkerBoardSize * 4;

    // Generate a new checkboard in wasm
    rustWasm.generate_checker_board(
        getDarkValue(),
        getDarkValue(),
        getDarkValue(),
        getLightValue(),
        getLightValue(),
        getLightValue()
    );

    const bufferStart = rustWasm.get_output_buffer_pointer();
    const imageBytes = new Uint8ClampedArray(rustWasm.memory.buffer, bufferStart, checkerBoardByteLength);
    const imageData = new ImageData(imageBytes, canvasElement.width, canvasElement.height);
    canvasContext.putImageData(imageData, 0, 0);
};
```

Very Trippy

