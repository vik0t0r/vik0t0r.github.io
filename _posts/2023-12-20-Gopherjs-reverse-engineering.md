---
layout: post
title: Gopherjs reverse engineering
date: 2023-12-20 16:50 +0200
categories: [Hacking, Software]
tags: [hacking, software, javascript, golang, reverse engineering, android]
media_subpath: /assets/img/gopherjs/
---
## Introduction
<a href="https://github.com/gopherjs/gopherjs" target="_blank">Gopherjs</a>
is a compiler for the go language which compiles to pure javascript, it can
be used to write go code which is executed on a webrowser, or where I found it, on a <a href="https://cordova.apache.org/" target="_blank">cordova</a> app, while I was trying to reverse engineer an android apk. Usually reverse engineering android apps is quite trivial thanks to the great tools which exists out ther like <a href="https://apktool.org/" target="_blank">apktool</a> or <a href="https://github.com/skylot/jadx" target="_blank">jadx</a>. But on a cordova app, where all user code is written in javascript, this tools are rendered useless.

In this cases, understanding how javascript works is vital. However, compared with the strictity of java, where you can easily identify which methods are calling which methods and renaming symbols does not brake the hole codebase, statically analyzing javascript is much more difficult.

Well, enough chattering, lets see how gopherjs works, a great start is <a href="https://www.youtube.com/watch?v=HObqhDgMdgk" target="_blank">this</a> video, where you can see some exammples of how go code is translated into javascript code. Take into account that, in production, the code will be minified.

## First approach: map files
When compiling a project whith gopherjs two files are usually generated, one .js with the code for execution and one .js.map which serves for debugging purposes and maps some statements of the javascript file to its original place in the go files. This way, in case of exception, developers can know in which place the error took place.
More info about source maps can be found <a href="https://github.com/ryanseddon/source-map/wiki/Source-maps:-languages,-tools-and-other-info" target="_blank">here</a>

On a first approach, I though about using this map files to recover the source code. But even though usually map files contain the original source code. This is not the case for gopherjs and the only info which can be extracted are file paths and little more. Thats why we cannot use tools like <a href="https://github.com/denandz/sourcemapper" target="_blank">this one</a> because all they do is extracted the source code from the map itself, in fact, they do not use the minified code for anything. 

Even though, I tried seeing what I could recover from this map files and I wrote the following python script:

```python
import sourcemap
import os
import shutil
from pathlib import Path
from sourcemap.decoder import SourceMapIndex, Token

# Load the original minified JavaScript file and its source map
minified = open('hello-world.js').readlines()
index: SourceMapIndex = sourcemap.load(open("hello-world.js.map"))

# Remove tokens with no associated source file
index.tokens = [x for x in index if x.src is not None]

def get_token(token: Token):
    line: str = minified[token.dst_line]
    end = line.find(";", token.dst_col)
    return line[token.dst_col: end]

output_files = {}

# Iterate through each token in the source map
for i in range(len(index)):
    item: Token = index[i]

    # Check if source file is already in the output dictionary
    if item.src in output_files:
        output = output_files[item.src]
    else:
        output_files[item.src] = {}
        output = output_files[item.src]

    # Place the token in its expected location in the original file
    output[item.src_line] = " " * item.src_col + get_token(item)

# Remove existing "output" directory and create a new one
shutil.rmtree("output")
os.mkdir("output")

# Write the reconstructed files to the "output" directory
for f in output_files.keys():
    path_str = "output/" + f[:-2] + "js"
    path = Path(path_str)
    path.parent.mkdir(exist_ok=True, parents=True)

    with open(path, "w") as output_file:
        output = output_files[f]
        for l_i in range(max(output.keys()) + 1):
            try:
                output_file.write(output[l_i] + "\n")
            except KeyError:
                output_file.write("\n")
```
This, recovers the following javascript from the original go code:
```go
package main

import "fmt"

type person struct {
    name string
    age  int
}
func newPerson(name string) *person {
    p := person{name: name}
    p.age = 42
    return &p
}

func main() {
    fmt.Println(person{"Bob", 20})
}
```
```javascript















     b=A.Println(new E([(a=new B.ptr("Bob",20),new a.constructor.elem(a))]))

```
As we can see, only executable statements have an entry on the map file, and with this method, we lost a ton of useful information.

## Second approach: analyzing the minified code structure

Compiling the previous go example with the minified flag, we get something like this: (lines have been truncated to 80 chars per line):
```javascript
"use strict";
(function() {

var $goVersion = "go1.17.9";
var $global,$module;if(Error.stackTraceLimit=1/0,"undefined"!=typeof window?$glo

$packages["github.com/gopherjs/gopherjs/js"]=(function(){var $pkg={},$init,A,B,J
$packages["runtime/internal/sys"]=(function(){var $pkg={},$init;$init=function()
$packages["runtime"]=(function(){var $pkg={},$init,B,A,D,E,M,AE,AS,AW,AX,AY,AZ,B
$packages["internal/unsafeheader"]=(function(){var $pkg={},$init,A;A=$pkg.Slice=
$packages["internal/reflectlite"]=(function(){var $pkg={},$init,C,A,B,D,E,H,N,O,
$packages["errors"]=(function(){var $pkg={},$init,A,G,H,L,E,a,F;A=$packages["int
$packages["internal/abi"]=(function(){var $pkg={},$init;$init=function(){$pkg.$i
$packages["internal/goexperiment"]=(function(){var $pkg={},$init;$init=function(
$packages["internal/itoa"]=(function(){var $pkg={},$init,C,D,A,B;C=$arrayType($U
$packages["math/bits"]=(function(){var $pkg={},$init,M,N,O,R,S,AP,AQ,AS,AX;O=fun
$packages["math"]=(function(){var $pkg={},$init,B,A,IT,IU,IV,IW,DN,FJ,FK,FL,FN;B
$packages["internal/cpu"]=(function(){var $pkg={},$init;$init=function(){$pkg.$i
$packages["internal/bytealg"]=(function(){var $pkg={},$init,A,C;A=$packages["int
$packages["unicode/utf8"]=(function(){var $pkg={},$init,B,A,C,F,G,J,K,L,M,P,Q;B=
$packages["strconv"]=(function(){var $pkg={},$init,F,C,E,D,B,A,BL,BU,CE,CI,DT,DU
$packages["internal/race"]=(function(){var $pkg={},$init,A,B,C,D,E,H,I;A=functio
$packages["sync/atomic"]=(function(){var $pkg={},$init,A,B,AJ,L,O,R,W,Z,AC,AE,AI
$packages["sync"]=(function(){var $pkg={},$init,C,A,B,D,E,F,U,V,AL,AT,AV,AW,AX,B
$packages["unicode"]=(function(){var $pkg={},$init;$init=function(){$pkg.$init=f
$packages["reflect"]=(function(){var $pkg={},$init,K,J,A,L,B,C,D,E,F,G,H,I,O,P,S
$packages["sort"]=(function(){var $pkg={},$init,A,M,Q,AJ,AK,AL,AM;A=$packages["i
$packages["internal/fmtsort"]=(function(){var $pkg={},$init,A,B,C,I,J,D,E,F,G,H;
$packages["io"]=(function(){var $pkg={},$init,A,B,N,O,Z,AA,AH,AR,BE,BF,BG,BL,M,A
$packages["internal/oserror"]=(function(){var $pkg={},$init,A;A=$packages["error
$packages["syscall"]=(function(){var $pkg={},$init,H,G,I,A,B,C,D,E,F,N,W,AB,AC,A
$packages["internal/syscall/unix"]=(function(){var $pkg={},$init,B,A,C,H;B=$pack
$packages["github.com/gopherjs/gopherjs/nosync"]=(function(){var $pkg={},$init,B
$packages["time"]=(function(){var $pkg={},$init,C,E,D,A,B,V,W,X,AF,AG,AP,AQ,AR,A
$packages["internal/poll"]=(function(){var $pkg={},$init,H,C,A,D,E,F,B,G,X,AD,AJ
$packages["internal/syscall/execenv"]=(function(){var $pkg={},$init,A;A=$package
$packages["internal/testlog"]=(function(){var $pkg={},$init,B,A,C,N,D,F,I;B=$pac
$packages["path"]=(function(){var $pkg={},$init,A,B,C;A=$packages["errors"];B=$p
$packages["io/fs"]=(function(){var $pkg={},$init,A,E,C,B,D,F,G,AE,AL,AM,AN,AP,AR
$packages["os"]=(function(){var $pkg={},$init,E,K,F,P,J,N,H,G,M,I,D,A,Q,L,O,B,C,
$packages["fmt"]=(function(){var $pkg={},$init,A,I,B,C,D,E,F,G,H,V,W,X,AK,AL,AM,
$packages["main"]=(function(){var $pkg={},$init,A,B,E,D;A=$packages["fmt"];B=$pk
$synthesizeMethods();
$initAllLinknames();
var $mainPkg = $packages["main"];
$packages["runtime"].$init();
$go($mainPkg.$init, []);
$flushConsole();

}).call(this);
//# sourceMappingURL=hello-world.js.map
```
At a glance, we can get the following info:
1. First six lines, I will call it "header", it the same for all the projects, so we can ignore them, the only useful info is the go version, which also tell us the gopherjs version as they are linked.
2. Then we have a variable number of lines, were each line is a go package, we will split each of this lines in an individual file, so the IDE those not implode (which is what happens when you try to beautify the code all at once).
3. Lastly, we have 9 lines which are all the same for all the programs too, this is where everything starts whith the call to init(), but we will ignore this part for now.

### VSCode extension

To help with the decompiling process I will be writting a VScode extension which can be found <a href="https://github.com/vik0t0r/gopherjsre.git" target="_blank">here</a>. This extension has two main commands:
1. Split files: This has to be executed insidel the .js file, and splits each package in its own file. Whats great about this is that now we can open and beautify each package without the ide imploding. This process could be improved if info in the .js.map file was used to split each package in multiple files.
2. Decompile each file: To be executed in each of the previous files, beautifies the code and renames some symbols.

#### Renaming package symbols

Each package, onces beautified, starts with something like:
```javascript
$packages["main"] = (function () {
    var $pkg = {},
        $init, A, B, E, D; // package variable definitions: firstly other packages, secondly package types, and then function declarations.
    A = $packages["fmt"]; // 1
    B = $pkg.person = $newType(0, $kindStruct, "main.person", true, "main", false, function (name_, age_) { // 2
        this.$val = this;
        if (arguments.length === 0) {
            this.name = "";
            this.age = 0;
            return;
        }
        this.name = name_;
        this.age = age_;
    });
    E = $sliceType($emptyInterface); // 3
    D = function () { // 4
```
As we can see on the code above, everything starts with a list of variable declarations, this variables are in orther:
1. other packages references
2. newTypes definitions
3. built-in types definition (slices, arrays, pointers...)
4. function definitions

This makes us really easy to rename something like `A = $packages["fmt"]` to `fmt_A = $packages["fmt"]` using regular expresions.

#### Renaiming types in each package

Type definition its also easy to rename, above all new types are created with a call to $newType() with the following form: `B = $pkg.person = $newType(0, $kindSt...` which makes it possible to rename `B` to `type_person_B`. 

But we have to take into account that other built-in types, this are pointers, arrays, slices and functions, usually they are placed all together:

```javascript
BM = $sliceType($emptyInterface);
BN = $ptrType(reflect_E.rtype);
BO = $ptrType(type_buffer_AO);
BP = $arrayType($Uint8, 68);
BS = $sliceType($Uint8);
BT = $ptrType(type_ss_W);
CO = $ptrType(type_pp_AP);
```
This makes for it to be possible to rename them, I'm not really sure if my naming convention is too verbose or what but each of this types starts with `t_` then comes the type (Array, Slice, Func, Ptr) and then I put the containing types (which makes for really long names) and finally the original name. So for example, the first line would be translated to : `t_SliceEmptyInterface_BM`. As said, this can be improved.

#### Renaiming functions

Function renaiming is something which can help us to understand better the code by renaiming something like `D` to `function_D` so in the future we know with which kind of object we are dealing with. Moreover, if the function is exported outside the package, we can know its original name, because after the function declaration there is a statement like: `$pkg.println = D` which tells us that function D in originally was called println.

#### Remove goroutines boilerplate

There is no concurrency in javascript, and thats why a lot of boilerplate is generated to handle goroutines. Specifically, a switch is generated, where between each line of code, there is a case, something like:
```javascript
        s: while (true) {
            switch ($s) {
                case 0:
                    b = fmt_A.Println(new E([(a = new B.ptr("Bob", 20), new a.constructor.elem(a))])); // real line of code
                    $s = 1;
                case 1:
                    if ($c) {
                        $c = false;
                        b = b.$blk();
                    }
                    if (b && b.$blk !== undefined) {
                        break s;
                    }
                    b;
                    $s = -1;
                    return;
```

And it would be really cool to be able to diferentiate the real code from the goroutine boilerplate.
The easiest way is to comment out the code blocks just after each each case statement, so we get something like the following code improving readability:
```javascript
        s: while (true) {
            switch ($s) {
                case 0:
                    b = fmt_A.Println(new E([(a = new B.ptr("Bob", 20), new a.constructor.elem(a))])); // real line of code
                    $s = 1;
                case 1:
//                  if($c){$c=false;b=b.$blk();}if(b&&b.$blk!==undefined){break s;}
                    b;
                    $s = -1;
                    return;
```
More could be done in the future, for example by tryng to remove all the switches or something like that.

## Some ideas for the future

Use a parser like <a href="https://github.com/acornjs/acorn/tree/master" target="_blank">this one</a> to improve the decompiler extension. Or maybe run the code for its decompilation, as the $packages structure might contain really useful information, and maybe could be use for better reverse engineering.

## Conclusion

Now I have an initial understanding of how gopherjs works, and I hope this post serves as a starting point for future researchers who need to reverse engineer some golang compiled with gopherjs.

I do believe that the vscode extension, although being a little bit rudimentary, is a great point for starting reverse engineering gopherjs code.

Theres still a chance that I improve the decompiler in the future.

## References
1. <a href="https://github.com/gopherjs/gopherjs" target="_blank">Gopherjs github</a>
2. <a href="https://apktool.org/" target="_blank">Apktool webpage</a>
3. <a href="https://github.com/skylot/jadx" target="_blank">Jadx a java decompiler</a>
4. <a href="https://www.youtube.com/watch?v=HObqhDgMdgk" target="_blank">Youtube video explaining how does go code translate to javascript</a>
5. <a href="https://github.com/ryanseddon/source-map/wiki/Source-maps:-languages,-tools-and-other-info" target="_blank">Source map info</a>
6. <a href="https://github.com/denandz/sourcemapper" target="_blank">Tool for extracting code from sourcemaps</a>
7. <a href="https://github.com/vik0t0r/gopherjsre.git" target="_blank">My vscode extension for dealing with gopherjs</a>