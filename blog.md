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

