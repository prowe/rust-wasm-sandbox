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

Let's work through (Wasm by Example)[https://wasmbyexample.dev]. Specifically, the [Hello World Rust Example](https://wasmbyexample.dev/examples/hello-world/hello-world.rust.en-us.html).

I decided I didn't want to do a bunch of setup on my machine, so I fired up [VS Code](https://code.visualstudio.com) and opened a folder using a Rust [Dev Container](https://code.visualstudio.com/docs/remote/containers).

I won't repeat the instructions for the example here. I'm just following along with the Wasm by Example site.

Installing packages via cargo (the rust package manager) takes a while as it seems to download and then compile them all. Good time to go for a walk or make some coffee.

It seems that the gist of the WASM architecture is to compile down a bunch of code (ie. Rust) into a WASM module and that module "exports" functions that can be called from Javascript. This reminds me of the way [Java invokes native methods using JNI](https://en.wikipedia.org/wiki/Java_Native_Interface).

Looking at the `pkg` directory that was created. It automatically created a `.gitignore` file in there. I like that I don't have to remember to ignore these build artifacts. It also created a type definition file for the exproted rust functions. That's handy too.

I can't just open `index.html` in a browser because Safari will refuse to load local resources for security reasons. When working on Javascript projects I usually run a simple webserver using `npx` but I didn't want to install that in the container or have to run a mix of in and out of continer stuff. So I went looking for a Rust based web server. Gave up quicky but found the dev container has Python so I ran `python -m SimpleHTTPServer` that I found on [Stack Overflow](https://unix.stackexchange.com/a/32200) and then forwarded port 8000.

Now I'm getting a 404 in the browser. Turns out the example had the wrong file names (mine are prefixed with `rust_`) so fixed those names and now we're cooking.

It seems a bit odd that we import a JS module, call init on it (an async method) and we have to pass the wasm module to init. I can't think of a scenerio where I would ever want to pass a different wasm module to init since it is specific to my exported functions.

Fun fact: my wasm module is only 180 bytes. Pretty small so it must not be packaging a lot of extra boilerplate. The JS binding file is 1.7k though but it is pretty printed and not minified.

With that complete, I'm moving on to [WASM Exports](https://wasmbyexample.dev/examples/exports/exports.rust.en-us.html).

