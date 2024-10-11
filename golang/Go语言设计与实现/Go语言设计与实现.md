# 介绍

来源：https://www.bookstack.cn/read/draveness-golang/cd9337bf6625a2e8.md

# 第一章 介绍

## Go 语言设计与实现

Go 语言是 Google 在 **2009 年 12 月**发布的编程语言，目前的 Go 语言在国内外的社区都非常热门，很多著名的开源框架 Kubernetes、etcd 和 prometheus 等都使用 Go 语言进行开发，近年来热门的微服务架构和云原生技术也为 Go 语言社区带来了非常多的活力。

作者目前也使用 Go 语言作为日常开发的主要语言，虽然 Go 语言没有 Lisp 系语言的开发效率和强大表达能力，但是却是一门非常容易使用并且大规模运用的工程语言，这也是作者学习和使用 Go 语言的主要原因。

> 补充：Lisp（List Processing）是一种编程语言家族，最早由约翰·麦卡锡（John McCarthy）在1958年开发。Lisp 系语言以其强大的元编程能力和函数式编程风格而闻名。
>
> Lisp 系语言的特点包括：
>
> 1. 基于列表的数据结构：Lisp 系语言使用列表（List）作为主要的数据结构，列表中的元素可以是其他列表，这种嵌套的结构使得 Lisp 系语言非常灵活。
> 2. 函数式编程：Lisp 系语言鼓励使用函数式编程的风格，函数是一等公民，可以作为参数传递、返回值返回，以及存储在变量中。这种编程风格强调不可变性和无副作用，使得代码更加清晰、可维护和可测试。
> 3. 元编程能力：Lisp 系语言具有强大的元编程能力，可以在运行时操作和修改程序的结构。这使得开发者可以编写高度抽象和灵活的代码，甚至可以编写自定义的语言扩展。
> 4. 动态类型系统：Lisp 系语言通常使用动态类型系统，变量的类型在运行时确定。这种灵活性使得开发者可以更快地进行迭代和开发，但也需要更多的测试和错误处理。
>
> Lisp 系语言的代表性成员包括 Common Lisp、Scheme 和 Clojure。它们在语法和特性上有所不同，但都继承了 Lisp 的核心思想和特点。



## 关于本书

这本书介绍的主要内容其实就是 **Go 语言内部的实现原理**，目前的大纲包含以下的八个章节，在编写的过程中还是会对内容的编排顺序和方式进行修改：

![golang-internal](Go语言设计与实现.assets/afa64721a976e9d548f5b973241109db.png)

虽然文章的目录结构可能会改变，但是你一定会通过这本书了解到 Go 语言相关的以下内容：

- 从词法语法解析、类型检查、中间代码生成以及机器码生成的**编译**相关全链路；
- 数组、哈希表和字符串等数据结构的表示以及基本操作的实现原理；
- 理解 Go 语言中的函数、方法、闭包和上下文等语言特性；
- 常见**并发编程**使用的 WaitGroup、Once 以及互斥锁等结构的实现；
- 语言中的**万能类型 interface** 的实现原理；
- 各种关键字 make、new、defer、select 以及 for 循环的处理过程；
- Goroutine 和 Channel 相关的结构、调度策略以及原理；
- …

> TBD：书中的内容将会有不少于 60% 的章节将以公开的形式发布，剩余最多 40% 的内容将通过其他的渠道收费进行发布，不过在 Go 语言的主要版本更新时，会对文章的内容进行整理和更新，历史的章节会在网站中以公开形式发布，最新的章节仍然会通过不同渠道推送给付费的订阅者。

## 其他

无论你是对目前书中不包含的 Go 语言特性的实现原理感兴趣，还是有其他的建议都可以在下面留言，欢迎持续关注本书的进展，谢谢。

![wechat-account-qrcode](Go语言设计与实现.assets/32b916ab562494e6858bc9393dc256b4.png)

本作品采用[知识共享署名-非商业性使用-禁止演绎 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-nd/4.0/)进行许可。





# 第二章 编译原理

## 2.1 概述

Go 语言是一门需要编译才能运行的编程语言，也就是说代码在运行之前**需要通过编译器生成二进制机器码**，包含二进制机器码的文件才能在目标机器上运行，如果我们想要了解 Go 语言的实现原理，理解它的编译过程就是一个没有办法绕过的事情。



这一节会先对 Go 语言编译的过程进行概述，从顶层介绍编译器执行的几个步骤，随后的几节会分别剖析各个步骤完成的工作和实现原理，同时也会对一些需要预先掌握的知识进行介绍，确保后面的章节能够被更好的理解。

### 2.1.1 预备知识

想要深入了解 Go 语言的编译过程，需要提前了解一下编译过程中涉及的一些术语和专业知识。这些知识其实在我们的日常工作和学习中比较难用到，但是对于理解编译的过程和原理还是非常重要的。这一小节会简单挑选几个常见并且重要的概念提前进行介绍，减少后面章节的理解压力。

#### 抽象语法树

[抽象语法树](https://en.wikipedia.org/wiki/Abstract_syntax_tree)（AST），是源代码语法的结构的一种抽象表示，它**用树状的方式表示编程语言的语法结构**[1](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fn:1)。抽象语法树中的每一个节点都表示源代码中的一个元素，每一颗子树都表示一个语法元素，例如一个 if else 语句，我们可以从 `2 * 3 + 7` 这一表达式中解析出下图所示的抽象语法树。

![abstract-syntax-tree](go语言设计与实现.assets/9b14e53fd7755501c99373c2edb27835.png)

**图 2-1 简单表达式的抽象语法树**

作为编译器常用的数据结构，抽象语法树抹去了源代码中不重要的一些字符 - 空格、分号或者括号等等。编译器在执行完语法分析之后会输出一个抽象语法树，这个抽象语法树会辅助编译器进行语义分析，我们可以用它来确定语法正确的程序是否存在一些类型不匹配或不一致的问题。



#### 静态单赋值

[静态单赋值](https://en.wikipedia.org/wiki/Static_single_assignment_form)（Static Single Assigment, **SSA**）是中间代码的一个特性，如果一个中间代码具有静态单赋值的特性，那么每个变量就只会被赋值一次[2](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fn:2)。在实践中我们通常会用添加下标的方式实现每个变量只能被赋值一次的特性，这里以下面的代码举个例子：

```
x := 1
x := 2
y := x
```

根据分析，我们其实能够发现上述的代码其实并不需要第一个将 `1` 赋值给 `x` 的表达式，也就是 `x := 1` 这一表达式在上述的代码片段中是没有作用的。

```
x1 := 1
x2 := 2
y1 := x2
```

当我们使用具有 SSA 特性的中间代码时，就可以非常清晰地发现变量 `y1` 的值和 `x1` 是完全没有任何关系的，所以在机器码生成时其实就可以省略第一步，这样就能减少需要执行的指令来优化这一段代码。(语法优化)



在中间代码中使用 SSA 的特性能够为整个程序实现以下的优化：

- 常数传播（constant propagation）
- 值域传播（value range propagation）
- 稀疏有条件的常数传播（sparse conditional constant propagation）
- 消除无用的程式码（dead code elimination）
- 全域数值编号（global value numbering）
- 消除部分的冗余（partial redundancy elimination）
- 强度折减（strength reduction）
- 寄存器分配（register allocation）因为 SSA 的主要作用是对代码进行优化，所以它是编译器后端[3](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fn:3)的一部分；当然代码编译领域除了 SSA 还有很多中间代码的优化方法，编译器生成代码的优化也是一个非常古老并且复杂的领域，这里就不会展开介绍了。

#### 指令集

最后要介绍的一个预备知识就是[指令集](https://en.wikipedia.org/wiki/Instruction_set_architecture)-[4](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fn:4)了，很多开发者都会遇到在生产环境运行的结果和本地不同的问题，导致这种情况的原因其实非常复杂，不同机器使用不同的指令也是可能的原因之一。

我们大多数开发者都会使用 x86_64 的 Macbook 作为工作上主要使用的硬件，在命令行中输入 `uname -m` 就能够获得当前机器上硬件的信息：

```
$ uname -m
x86_64
```

x86 是目前比较常见的指令集，除了 x86 之外，还有很多其他的指令集，不同的处理器使用了不同的架构和机器语言，所以很多编程语言为了在不同的机器上运行需要将源代码根据架构翻译成不同的机器代码。

复杂指令集计算机（Complex Instruction Set Computer，CISC）和精简指令集计算机（Reduced Instruction Set Computer，RISC）是目前的两种 CPU 区别，它们的在设计理念上会有一些不同，从名字我们就能看出来这两种不同的设计有什么区别：

- 复杂指令集通过增加指令的数量（复杂度）减少需要执行的指令数；
- 精简指令集能使用更少的指令完成目标的计算任务；

早期的 CPU 为了减少机器语言指令的（使用？）数量使用复杂指令集完成计算任务，这两者其实并没有绝对的好坏，它们只是在一些设计上的选择不同以达到不同的目的，我们会在后面的[机器码生成](https://www.bookstack.cn/read/draveness-golang/100aabc3275d17e5.md)一节中详细介绍指令集架构，不过各位读者也可以主动了解相关的内容。



### 2.1.2 编译原理

Go 语言编译器的源代码在 [`src/cmd/compile`](https://github.com/golang/go/tree/master/src/cmd/compile) 目录中，目录下的文件共同组成了 Go 语言的编译器。

```bash
# ll src/cmd/compile/
total 68
-rw-r--r--.  1 root root 39969 Nov 30  2023 abi-internal.md
-rw-r--r--.  1 root root 10303 Nov 30  2023 doc.go
drwxr-xr-x. 45 root root  4096 Nov 30  2023 internal
-rw-r--r--.  1 root root  1357 Nov 30  2023 main.go
-rw-r--r--.  1 root root  7076 Nov 30  2023 README.md
```

学过编译原理的人可能听说过编译器的前端和后端，**编译器的前端一般承担着词法分析、语法分析、类型检查和中间代码生成几部分工作**，而编译器**后端主要负责目标代码的生成和优化**，也就是将中间代码翻译成目标机器能够运行的二进制机器码。

![complication-process](go语言设计与实现.assets/a9ca56ee97ec2c0e4ea544fb6e806872.png)

**图 2-2 编译原理的核心过程**

> lexical analysis:词法分析（Lexical Analysis），也称为扫描（Scanning），是编译器中的一个阶段，它负责将源代码分割成一个个的词法单元（Token）。词法单元是编程语言中的最小语法单位，例如关键字、标识符、运算符、常量等。
>
> syntax analysis:语法分析（Syntax Analysis），也称为解析（Parsing），是编译器中的一个阶段，它负责根据词法分析器生成的词法单元流（Token Stream）来分析源代码的语法结构，并构建相应的语法树（Syntax Tree）或抽象语法树（Abstract Syntax Tree）。
>
> semantic analysis：语义分析（Semantic Analysis）/类型检查是编译器中的一个重要阶段，它负责对源代码进行语义检查和语义解释。语义分析的目标是确保程序在语义上是正确的，并生成中间表示（如抽象语法树、符号表等）以供后续阶段使用。
>
> IR generation :R 生成（IR Generation）是编译器中的一个阶段，它负责将源代码转换为中间表示（Intermediate Representation，简称 IR）。中间表示是一种介于源代码和目标代码之间的抽象表示形式，它捕捉了源代码的语义和结构，方便后续的优化和代码生成。
>
> code optimization：代码优化
>
> machine code generation：机器码生成



Go 的编译器在逻辑上可以被分成四个阶段：词法与语法分析、类型检查和 AST 转换、通用 SSA 生成和最后的机器代码生成，在这一节我们会使用比较少的篇幅分别介绍这四个阶段做的工作，后面的章节会具体介绍每一个阶段的具体内容。



#### 词法与语法分析

所有的编译过程其实都是从解析代码的源文件开始的，

词法分析的作用就是解析源代码文件，它**将文件中的字符串序列转换成 Token 序列**，方便后面的处理和解析，我们一般会把执行词法分析的程序称为词法解析器（lexer）。

而**语法分析的输入就是词法分析器输出的 Token 序列，这些序列会按照顺序被语法分析器进行解**析，语法的解析过程就是将词法分析生成的 Token 按照语言定义好的文法（Grammar）自下而上或者自上而下的进行规约，每一个 Go 的源代码文件最终会被归纳成一个 [`SourceFile`](https://golang.org/ref/spec#Source_file_organization) 结构[5](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fn:5)：

```
SourceFile = PackageClause ";" { ImportDecl ";" } { TopLevelDecl ";" } .
```

词法分析会返回一个不包含空格、换行等字符的 Token 序列，例如：`package`, `json`, `import`, `(`, `io`, `)`, …，而语法分析会把 Token 序列转换成有意义的结构体，也就是语法树：

```
"json.go": SourceFile {
    PackageName: "json",
    ImportDecl: []Import{
        "io",
    },
    TopLevelDecl: ...
}
```

将 Token 转换成上述『语法树』就会使用语法解析器，语法解析的结果其实就是上面介绍过的抽象语法树（AST），每一个 AST 都对应着一个单独的 Go 语言文件，这个抽象语法树中包括当前文件属于的包名、定义的常量、结构体和函数等。

> Go 语言的语法解析器使用的就是 LALR(1)[6](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fn:6) 的文法，对解析器文法感兴趣的读者可以在推荐阅读中找到编译器文法的相关资料。

![golang-files-and-ast](go语言设计与实现.assets/fc26e0c77f13a4782b7376a59b4f2510.png)

**图 2-3 从源文件到语法树**

如果在语法解析的过程中发生了任何语法错误，都会被语法解析器发现并将消息打印到标准输出上，整个编译过程也会随着错误的出现而被中止。[词法与语法分析](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md)一节会详细介绍 Go 语言的文法、词法解析和语法解析过程。



#### 类型检查

当拿到一组文件的抽象语法树之后，Go 语言的编译器会对语法树中定义和使用的类型进行检查，**类型检查分别会按照以下的顺序对不同类型的节点进行验证和处理**：

- 常量、类型和函数名及类型；
- 变量的赋值和初始化；
- 函数和闭包的主体；
- 哈希键值对的类型；
- 导入函数体；
- 外部的声明；通过对整颗抽象语法树的遍历，我们在每一个节点上都会对当前子树的类型进行验证，以保证当前节点上不会出现类型错误的问题，所有的类型错误和不匹配都会在这一个阶段被发现和暴露出来，结构体是否实现了某些接口也会在这一阶段被检查出来。

类型检查阶段不止会对**节点的类型**进行验证，还会展开和改写一些内建的函数，例如 `make` 关键字在这个阶段会根据子树的结构被替换成 `makeslice` 或者 `makechan` 等函数。

![golang-keyword-make](go语言设计与实现.assets/091a5fa198f55d436909c85c71e0e9c3.png)

**图 2-4 类型检查阶段对 make 进行改写**

类型检查这一过程在整个编译流程中还是非常重要的，Go 语言的很多关键字都依赖类型检查期间的展开和改写，后面的章节[类型检查](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md)会详细介绍这一步骤。



#### 中间代码生成

当我们将源文件转换成了抽象语法树、对整棵树的语法进行解析并进行类型检查之后，就可以认为当前文件中的代码不存在语法错误和类型错误的问题了，Go 语言的编译器就会将输入的抽象语法树转换成中间代码。

在类型检查之后，就会通过一个名为 `compileFunctions` 的函数开始对整个 Go 语言项目中的全部函数进行编译，这些函数会在一个编译队列中等待几个后端工作协程的消费（MQ)，这些并发执行的 Goroutine 会将所有函数对应的抽象语法树转换成中间代码。

![concurrency-compiling](go语言设计与实现.assets/99d2b3eb13010f4d12e41bbe4b500573.png)

**图 2-5 并发编译过程**

由于 Go 语言编译器的中间代码使用了 SSA 的特性，所以在这一阶段我们就能够分析出代码中的无用变量和片段并对代码进行优化，在[中间代码生成](https://www.bookstack.cn/read/draveness-golang/5a89e04c79706261.md)这一节会详细介绍中间代码的生成过程并简单介绍 Go 语言是如何在中间代码中使用 SSA 特性的。



#### 机器码生成

Go 语言源代码的 [`src/cmd/compile/internal`](https://github.com/golang/go/tree/master/src/cmd/compile/internal) 目录中包含了很多机器码生成相关的包，不同类型的 CPU 分别使用了不同的包生成机器码，其中包括 amd64、arm、arm64、mips、mips64、ppc64、s390x、x86 和 wasm

```bash
# ll src/cmd/compile/internal/
total 172
drwxr-xr-x. 2 root root 4096 Nov 30  2023 abi
drwxr-xr-x. 2 root root 4096 Nov 30  2023 abt
drwxr-xr-x. 2 root root 4096 Nov 30  2023 amd64
drwxr-xr-x. 2 root root 4096 Nov 30  2023 arm
drwxr-xr-x. 2 root root 4096 Nov 30  2023 arm64
drwxr-xr-x. 2 root root 4096 Nov 30  2023 base
drwxr-xr-x. 2 root root 4096 Nov 30  2023 bitvec
drwxr-xr-x. 2 root root 4096 Nov 30  2023 compare
.....
```

其中比较有趣的就是 WebAssembly（Wasm）[7](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fn:7)了。作为一种在栈虚拟机上使用的二进制指令格式，它的设计的主要目标就是在 Web 浏览器上提供一种具有高可移植性的目标语言。Go 语言的编译器既然能够生成 Wasm 格式的指令，那么就能够运行在常见的主流浏览器中。

```
$ GOARCH=wasm GOOS=js go build -o lib.wasm main.go
```

我们可以使用上述的命令将 Go 的源代码编译成能够在浏览器上运行 WebAssembly 文件，当然除了这种新兴的二进制指令格式之外，Go 语言经过编译还可以运行在几乎全部的主流机器上，不过对于除了 Linux 和 Darwin 之外的机器上兼容性上可能还是有一些问题，例如：Go Plugin 至今仍然不支持 Windows[8](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fn:8)。

![supported-hardware](go语言设计与实现.assets/d72973fdb8a8d86de90d077e5032465d.png)



**图 2-6 Go 语言支持的架构**

[机器码生成](https://www.bookstack.cn/read/draveness-golang/100aabc3275d17e5.md)一节会详细介绍将中间代码翻译到不同目标机器的过程，在这个章节中也会简单介绍不同的指令集架构的区别。



### 2.1.3 编译器入口

Go 语言的编译器入口在 [`src/cmd/compile/internal/gc/main.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/gc/main.go) 文件中，这个 600 多行的 [`Main`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/main.go#L144-L796) 函数就是 Go 语言编译器的主程序，该函数会先获取命令行传入的参数并更新编译选项和配置，随后就会开始运行 `parseFiles` 函数对输入的所有文件进行词法与语法分析得到文件对应的抽象语法树：

```
func Main(archInit func(*Arch)) {    ...    lines := parseFiles(flag.Args())
```

接下来就会分九个阶段对抽象语法树进行更新和编译，就像我们在上面介绍的，整个过程会经历类型检查、SSA 中间代码生成以及机器码生成三个部分：

- 检查常量、类型和函数的类型；
- 处理变量的赋值；
- 对函数的主体进行类型检查；
- 决定如何捕获变量；
- 检查内联函数的类型；
- 进行逃逸分析；
- 将闭包的主体转换成引用的捕获变量；
- 编译顶层函数；
- 检查外部依赖的声明；对整个编译过程有一个顶层的认识之后，我们重新回到词法和语法分析后的具体流程，在这里编译器会对生成语法树中的节点执行类型检查，除了常量、类型和函数这些顶层声明之外，它还会对变量的赋值语句、函数主体等结构进行检查：

```go
    for i := 0; i < len(xtop); i++ {
        n := xtop[i]
        if op := n.Op; op != ODCL && op != OAS && op != OAS2 && (op != ODCLTYPE || !n.Left.Name.Param.Alias) {
            xtop[i] = typecheck(n, ctxStmt)
        }
    }
    for i := 0; i < len(xtop); i++ {
        n := xtop[i]
        if op := n.Op; op == ODCL || op == OAS || op == OAS2 || op == ODCLTYPE && n.Left.Name.Param.Alias {
            xtop[i] = typecheck(n, ctxStmt)
        }
    }
    ...
```

类型检查会对遍历传入节点的全部子节点，这个过程会对 `make` 等关键字进行展开和重写，在类型检查会改变语法树中的一些节点，不会生成新的变量或者语法树，这个过程的结束也意味着源代码中已经不存在语法错误和类型错误，中间代码和机器码都可以根据抽象语法树正常生成了。

```go
    initssaconfig()
    peekitabs()
    for i := 0; i < len(xtop); i++ {
        n := xtop[i]
        if n.Op == ODCLFUNC {
            funccompile(n)
        }
    }
    compileFunctions()
    for i, n := range externdcl {
        if n.Op == ONAME {
            externdcl[i] = typecheck(externdcl[i], ctxExpr)
        }
    }
    checkMapKeys()
}
```

在主程序运行的最后，会将顶层的函数编译成中间代码并根据目标的 CPU 架构生成机器码，不过在这一阶段也有可能会再次对外部依赖进行类型检查以验证正确性。

### 2.1.4 小结

Go 语言的编译过程其实是非常有趣并且值得学习的，通过对 Go 语言四个编译阶段的分析和对编译器主函数的梳理，我们能够对 Go 语言的实现有一些基本的理解，掌握编译的过程之后，Go 语言对于我们来讲也不再是一个黑盒，所以学习其编译原理的过程还是非常让人着迷的。

- 抽象语法树 https://en.wikipedia.org/wiki/Abstract_syntax_tree[↩︎](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fnref:1)
- 静态单赋值 https://en.wikipedia.org/wiki/Static_single_assignment_form[↩︎](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fnref:2)
- 编译器一般分为前端和后端，其中前端的主要工作是将源代码编程语言无关的中间表示，而后端主要负责目标代码的优化和生成。 [↩︎](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fnref:3)
- 指令集架构是计算机的抽象模型，也被称作架构或者计算架架构 https://en.wikipedia.org/wiki/Instruction_set_architecture[↩︎](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fnref:4)
- `SourceFile` 表示一个 Go 语言源文件，它由 `package` 定义、多个 `import` 语句以及顶层的声明组成 https://golang.org/ref/spec#Source_file_organization[↩︎](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fnref:5)
- 关于 Go 语言文法的是不是 LALR(1) 的讨论 https://groups.google.com/forum/#!msg/golang-nuts/jVjbH2-emMQ/UdZlSNhd3DwJ

LALR 的全称是 Look-Ahead LR，大多数的通用编程语言都会使用 LALR 的文法 https://en.wikipedia.org/wiki/LALR_parser[↩︎](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fnref:6)

- WebAssembly 是基于栈的虚拟机的二进制指令，简称 Wasm https://webassembly.org/[↩︎](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fnref:7)
- plugin: add Windows support #19282 https://github.com/golang/go/issues/19282[↩︎](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md#fnref:8)



## 2.2 词法和语法分析

当使用[通用编程语言](https://en.wikipedia.org/wiki/General-purpose_programming_language)[1](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:1)进行编写代码时，我们一定要知道代码是写给人看的，只是恰好可以被机器编译和执行，而很难被人理解的代码是非常糟糕并且不容易维护的。代码其实就是按照约定格式编写的一堆字符串，经过训练的软件工程师可以在脑内对语言的源代码进行编译并运行目标程序，我们能对本来无意义的字符串进行分组和分析，按照约定的语法来理解源代码。

既然工程师能够按照一定的方式理解和编译 Go 语言的源代码，那么我们如何模拟人理解源代码的方式构建一个能够分析编程语言代码的程序呢。我们在这一节中将介绍词法分析和语法分析这两个重要的编译过程，这两个过程能将原本机器看来无序意义的源文件转换成更容易理解、分析并且结构化的抽象语法树，接下来我们就看一看解析器眼中的 Go 语言是什么样的。



### 2.2.1 词法分析

源代码在编译器眼中其实就是一团乱麻，一个由『无实际意义』字符组成的、无法被理解的字符串，所有的字符在编译器看来并没有什么区别，为了理解这些字符我们需要做的第一件事情就是**将字符串分组**，这样能够降低理解字符串的成本，简化分析源代码的过程。

```
make(chan int)
```

一个不懂编程的人看到上述文本的的第一反应就是将上述字符串分成几个部分 - `make`、`chan`、`int` 和括号，这个凭直觉分解文本的过程其实就是[词法分析](https://en.wikipedia.org/wiki/Lexical_analysis)，

词法分析是计算机科学中将字符序列转换为标记（token）序列的过程[2](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:2)。

#### lex

[lex](http://dinosaur.compilertools.net/lex/index.html)[3](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:3) 是用于生成词法分析器的工具，lex 生成的代码能够将一个文件中的字符分解成 Token 序列，很多语言在设计早期都会使用它快速设计出原型。词法分析作为具有固定模式的任务，出现这种更抽象的工具其实是必然的，lex 作为一个代码生成器，使用了类似 C 语言的语法，我们可以理解成 lex 使用正则匹配对输入的字符流进行扫描，下面是一个 lex 文件的示例(simplego.l)：

```
%{
#include <stdio.h>
%}
%%
package      printf("PACKAGE ");
import       printf("IMPORT ");
\.           printf("DOT ");
\{           printf("LBRACE ");
\}           printf("RBRACE ");
\(           printf("LPAREN ");
\)           printf("RPAREN ");
\"           printf("QUOTE ");
\n           printf("\n");
[0-9]+       printf("NUMBER ");
[a-zA-Z_]+   printf("IDENT ");
%%
```

这个定义好的文件能够解析 `package` 和 `import` 关键字、常见的特殊字符、数字以及标识符，虽然这里的规则可能有一些简陋和不完善，但是如果用来解析下面的这一段代码还是比较轻松的：

```
package main
import (
    "fmt"
)
func main() {
    fmt.Println("Hello")
}
```

`.l` 结尾的 lex 代码并不能直接运行，首先需要通过 `lex` 命令将上面的 `simplego.l` **『展开』成 C 语言代码**，这里可以直接执行如下的命令编译并打印当前文件中的内容：

```bash
$ lex simplego.l
$ cat lex.yy.c
...
int yylex (void) {
    ...
    while ( 1 ) {
        ...
yy_match:
        do {
            register YY_CHAR yy_c = yy_ec[YY_SC_TO_UI(*yy_cp)];
            if ( yy_accept[yy_current_state] ) {
                (yy_last_accepting_state) = yy_current_state;
                (yy_last_accepting_cpos) = yy_cp;
            }
            while ( yy_chk[yy_base[yy_current_state] + yy_c] != yy_current_state ) {
                yy_current_state = (int) yy_def[yy_current_state];
                if ( yy_current_state >= 30 )
                    yy_c = yy_meta[(unsigned int) yy_c];
                }
            yy_current_state = yy_nxt[yy_base[yy_current_state] + (unsigned int) yy_c];
            ++yy_cp;
        } while ( yy_base[yy_current_state] != 37 );
        ...
do_action:
        switch ( yy_act )
            case 0:
                ...
            case 1:
                YY_RULE_SETUP
                printf("PACKAGE ");
                YY_BREAK
            ...
}
```

[lex.yy.c](https://gist.github.com/draveness/85db6ec4a4088b63ccccf7f09424f474)[4](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:4) 的前 600 行基本都是宏和函数的声明和定义，后面生成的代码大都是为 `yylex` 这个函数服务的，这个函数使用[有限自动机（DFA）](https://en.wikipedia.org/wiki/Deterministic_finite_automaton)[5](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:5)的程序结构来分析输入的字符流，上述代码中 `while` 循环就是这个有限自动机的主体，你如果仔细看这个文件生成的代码会发现当前的文件中并不存在 `main` 函数，`main` 函数其实是在 liblex 这个库中定义的，所以在编译时其实需要添加额外的 `-ll` 选项：

```
$ cc lex.yy.c -o simplego -ll
$ cat main.go | ./simplego
```

当我们将 C 语言代码通过 **gcc 编译**成二进制代码之后，就可以使用管道将上面提到的 Go 语言代码作为输入传递到生成的词法分析器中，这个词法分析器会打印出如下的内容：

```
PACKAGE  IDENT
IMPORT  LPAREN
    QUOTE IDENT QUOTE
RPAREN
IDENT  IDENT LPAREN RPAREN  LBRACE
    IDENT DOT IDENT LPAREN QUOTE IDENT QUOTE RPAREN
RBRACE
```

从上面的输出我们能够看到 Go 源代码的影子，lex 生成的词法分析器 lexer 通过正则匹配的方式将机器原本很难理解的字符串进行分解成很多的 Token，有利于后面的继续处理。

![simplego-lex](go语言设计与实现.assets/9ec1c55f451d4ba85a52beac0cc0fadf.png)

**图 2-7 从 .l 文件到二进制**

如上图所示，到这里我们已经为各位读者展示了从定义 `.l` 文件，使用 `lex` 将 `.l` 文件编译成 C 语言代码到编译成二进制的全过程，而最后生成的词法分析器也能够将简单的 Go 语言代码进行分组，`lex` 的使用还是比较简单的，我们可以使用它快速实现词法分析器，相信各位读者对这个工具也有了一定的了解。





#### Go

> scanner

Go 语言的词法解析是通过 [`src/cmd/compile/internal/syntax/scanner.go`](https://github.com/golang/go/tree/master/src/cmd/compile/internal/syntax/scanner.go)[6](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:6) 文件中的 `scanner` 结构体实现的，这个结构体会持有当前扫描的数据源文件、启用的模式和当前被扫描到的 Token：

```go
type scanner struct {
    source
    mode   uint
    nlsemi bool
    // current token, valid after calling next()
    line, col uint
    tok       token
    lit       string
    kind      LitKind
    op        Operator
    prec      int
}
```

[`src/cmd/compile/internal/syntax/tokens.go`](https://github.com/golang/go/tree/master/src/cmd/compile/internal/syntax/tokens.go)[7](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:7) 文件中定义了 Go 语言中支持的全部 Token 类型，所有的 `token` 类型都是正整数，你可以在这个文件中找到一些常见 Token 的定义，例如：操作符、括号和关键字等：

```go
const (
    _    token = iota
    _EOF       // EOF
    // operators and operations
    _Operator // op
    // ...
    // delimiters
    _Lparen    // (
    _Lbrack    // [
    // ...
    // keywords
    _Break       // break
    // ...
    _Type        // type
    _Var         // var
    tokenCount //
)
```

从 Go 语言中定义的 Token 类型，我们可以将语言中的元素分成几个不同的类别，分别是名称和字面量、操作符、分隔符和关键字。词法分析主要是由 `scanner` 这个结构体中的 `next` 方法驱动，这个 250 行函数的主体就是一个 `switch/case` 结构：

```go
func (s *scanner) next() {
    c := s.getr()
    for c == ' ' || c == '\t' || c == '\n' || c == '\r' {
        c = s.getr()
    }
    s.line, s.col = s.source.line0, s.source.col0
    if isLetter(c) || c >= utf8.RuneSelf && s.isIdentRune(c, true) {
        s.ident()
        return
    }
    switch c {
    case -1:
        s.tok = _EOF
    case '0', '1', '2', '3', '4', '5', '6', '7', '8', '9':
        s.number(c)
    // ...
    }
}
```

`scanner` 每次都会通过 `getr` 函数获取文件中最近的未被解析的字符，然后根据当前字符的不同执行不同的 case，如果遇到了空格和换行符这些空白字符会直接跳过，如果当前字符是 `0` 就会执行 `number` 方法尝试匹配一个数字。

```
func (s *scanner) number(c rune) {
    s.startLit()
    s.kind = IntLit
    for isDigit(c) {
        c = s.getr()
    }
    s.ungetr()
    s.nlsemi = true
    s.lit = string(s.stopLit())
    s.tok = _Literal
}
```



上述的 `number` 方法省略了很多的代码，包括如何匹配浮点数、指数和复数，我们只是简单看一下词法分析匹配的逻辑。

- 在开始具体匹配之前调用 `startLit` 记录一下当前 Token 的开始位置；
- 在 for 循环中不断获取最新的字符，将字符追加到 `scanner` 持有的缓冲区中；
- 方法返回之前会通过 `ungetr` 撤销最近获取的错误字符（非数字）并将字面量和 Token 的类型传递回 `scanner`；当前包中的词法分析器 `scanner` 也只是为上层提供了 `next` 方法，词法解析的过程都是惰性的，只有在上层的解析器需要时才会调用 `next` 获取最新的 Token。

Go 语言的词法元素相对来说还是比较简单，使用这种巨大的 switch/case 进行词法解析也比较方便和顺手，早期的 Go 语言虽然使用 lex 这种工具来生成词法解析器，但是最后还是使用 Go 来实现词法分析器，用自己写的词法分析器来解析自己[8](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:8)。



### 2.2.2 语法分析

说完了编译最开始的词法分析过程，接下来就到了语法分析，[语法分析](https://en.wikipedia.org/wiki/Parsing)是根据某种特定的形式文法（Grammar）对 Token 序列构成的输入文本进行分析并确定其语法结构的一种过程[9](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:9)。

从上面的定义来看，词法分析器输出的结果 — Token 序列就是语法分析器的输入。

在语法分析的过程会使用**自顶向下**或者**自底向上**的方式进行推导，在介绍 Go 语言的语法分析的实现原理之前，我们会先来介绍语法分析中的文法和分析方法。

#### 文法

[上下文无关文法](https://en.wikipedia.org/wiki/Context-free_grammar)是用来对某种编程语言进行形式化的、精确的描述工具，我们能够通过文法定义一种语言的语法，它主要包含了一系列用于转换字符串的生产规则（production rule）[10](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:10)。上下文无关文法中的每一个生产规则都会将规则左侧的非终结符转换成右侧的字符串，文法都由以下的四个部分组成：

> 终结符是文法中无法再被展开的符号，而非终结符与之相反，还可以通过生产规则进行展开，例如 “id”、“123” 等标识或者字面量[11](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:11)。

- $N$ 有限个非终结符的集合；
- $\Sigma$ 有限个终结符的集合；
- $P$ 有限个生产规则[12](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:12)的集合；
- $S$ 非终结符集合中唯一的开始符号；

文法被定义成一个**四元组** $(N, \Sigma, P, S)$，这个元组中的几部分就是上面提到的四个符号，其中最为重要的就是生产规则了，每一个生产规则都会包含非终结符、终结符或者开始符号，我们在这里可以举一个简单的例子：

- $S \rightarrow aSb $
- $S \rightarrow ab $上述规则构成的文法就能够表示 `ab`、`aabb` 以及 `aaa..bbb` 等字符串，编程语言的文法就是由这一系列的生产规则表示的，在这里我们可以从 [`src/cmd/compile/internal/syntax/parser.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/syntax/parser.go)[13](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:13) 文件中摘抄一些 Go 语言文法的生产规则：

```go
SourceFile = PackageClause ";" { ImportDecl ";" } { TopLevelDecl ";" } .
PackageClause  = "package" PackageName .
PackageName    = identifier .

ImportDecl       = "import" ( ImportSpec | "(" { ImportSpec ";" } ")" ) .
ImportSpec       = [ "." | PackageName ] ImportPath .
ImportPath       = string_lit .

TopLevelDecl  = Declaration | FunctionDecl | MethodDecl .
Declaration   = ConstDecl | TypeDecl | VarDecl .
```

> Go 语言更详细的文法可以从 [Language Specification](https://golang.org/ref/spec)[14](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:14) 中找到，这里不仅包含语言的文法，还包含词法元素、内置函数等信息。

因为**每个 Go 源代码文件最终都会被解析成一个独立的抽象语法树**，所以语法树最顶层的结构或者开始符号其实就是 `SourceFile`：

```
SourceFile = PackageClause ";" { ImportDecl ";" } { TopLevelDecl ";" } .
```

从 `SourceFile` 相关的生产规则我们可以看出，每一个文件都包含一个 `package` 的定义以及可选的 `import` 声明和其他的顶层声明（TopLevelDecl），每一个 `SourceFile` 在编译器中都对应一个 `File` 结构体，你能从它们的定义中轻松找到两者的联系：

```
type File struct {    PkgName  *Name    DeclList []Decl    Lines    uint    node}
```

顶层声明有五大类型，分别是常量、类型、变量、函数和方法，你可以在文件 [`src/cmd/compile/internal/syntax/parser.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/syntax/parser.go) 中找到这五大类型的定义。

```go 
ConstDecl = "const" ( ConstSpec | "(" { ConstSpec ";" } ")" ) .
ConstSpec = IdentifierList [ [ Type ] "=" ExpressionList ] .

TypeDecl  = "type" ( TypeSpec | "(" { TypeSpec ";" } ")" ) .
TypeSpec  = AliasDecl | TypeDef .
AliasDecl = identifier "=" Type .
TypeDef   = identifier Type .

VarDecl = "var" ( VarSpec | "(" { VarSpec ";" } ")" ) .
VarSpec = IdentifierList ( Type [ "=" ExpressionList ] | "=" ExpressionList ) .
```

上述的文法分别定义了 Go 语言中常量、类型和变量三种常见的结构，从文法中可以看到语言中的很多关键字 `const`、`type` 和 `var`，稍微回想一下我们日常接触的 Go 语言代码就能验证这里文法的正确性。

除了三种简单的语法结构之外，函数和方法的定义就更加复杂，从下面的文法我们可以看到 `Statement` 总共可以转换成 15 种不同的语法结构，这些语法结构就包括我们经常使用的 switch/case、if/else、for 循环以及 select 等语句：

```
FunctionDecl = "func" FunctionName Signature [ FunctionBody ] .
FunctionName = identifier .
FunctionBody = Block .
MethodDecl = "func" Receiver MethodName Signature [ FunctionBody ] .
Receiver   = Parameters .
Block = "{" StatementList "}" .
StatementList = { Statement ";" } .
Statement =
    Declaration | LabeledStmt | SimpleStmt |
    GoStmt | ReturnStmt | BreakStmt | ContinueStmt | GotoStmt |
    FallthroughStmt | Block | IfStmt | SwitchStmt | SelectStmt | ForStmt |
    DeferStmt .
SimpleStmt = EmptyStmt | ExpressionStmt | SendStmt | IncDecStmt | Assignment | ShortVarDecl .
```



这些不同的语法结构共同定义了 Go 语言中能够使用的语法结构和表达式，对于 `Statement` 展开的更多内容这篇文章就不会详细介绍了，感兴趣的读者可以直接查看 [Go 语言说明书](https://golang.org/ref/spec#Statement)或者直接从 [`src/cmd/compile/internal/syntax/parser.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/syntax/parser.go) 文件中找到想要的答案。

#### 分析方法

语法分析的分析方法一般分为自顶向下和自底向上两种，这两种方式会使用不同的方式对输入的 Token 序列进行推导：

- [自顶向下分析](https://en.wikipedia.org/wiki/Top-down_parsing)：可以被看作找到当前输入流最左推导的过程，对于任意一个输入流，根据当前的输入符号，确定一个生产规则，使用生产规则右侧的符号替代相应的非终结符向下推导[15](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:15)；
- [自底向上分析](https://en.wikipedia.org/wiki/Bottom-up_parsing)：语法分析器从输入流开始，每次都尝试重写最右侧的多个符号，这其实就是说解析器会从最简单的符号进行推导，在解析的最后合并成开始符号[16](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:16)；

如果读者无法理解上述的定义也没有关系，我们会在这一节的剩余部分介绍两种不同的分析方法以及它们的具体分析过程。

##### 自顶向下

[LL 文法](https://en.wikipedia.org/wiki/LL_grammar)[17](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:17)就是一种使用自顶向上分析方法的文法，下面给出了一个常见的 LL 文法：

- $S \rightarrow aS_1$
- $S_1 \rightarrow bS_1 $
- $S_1 \rightarrow \epsilon$假设我们存在以上的生产规则和输入流 `abb`，如果这里使用自顶向上的方式进行语法分析，我们可以理解为每次解析器会通过新加入的字符判断应该使用什么方式展开当前的输入流：
- $S$ （开始符号）
- $aS_1$（规则 1)
- $abS_1$（规则 2)
- $abbS_1$（规则 2)
- $abb$（规则 3)这种分析方法一定会从开始符号分析，通过下一个即将入栈的符号判断应该如何对当前堆栈中最右侧的非终结符（$S$ 或 $S_1$）进行展开，直到整个字符串中不存在任何的非终结符，整个解析过程才会结束。

##### 自底向上

但是如果我们使用自底向上的方式对输入流进行分析时，处理过程就会完全不同了，常见的四种文法 LR(0)、SLR、LR(1) 和 LALR(1) 使用了自底向上的处理方式[18](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:18)，我们可以简单写一个与上一节中效果相同的 LR(0) 文法：

- $S \rightarrow S_1 $
- $S_1 \rightarrow S_1b $
- $S_1 \rightarrow a $使用上述等效的文法处理同样地输入流 `abb` 会使用完全不同的过程对输入流进行展开：
- $a$（入栈）
- $S_1$（规则 3）
- $S_1b$（入栈）
- $S_1$（规则 2）
- $S_1b$（入栈）
- $S_1$（规则 2）
- $S$（规则 1）自底向上的分析过程会维护一个堆栈用于存储未被归约的符号，在整个过程中会执行两种不同的操作，一种叫做入栈（shift），也就是将下一个符号入栈，另一种叫做归约（reduce），也就是对最右侧的字符串按照生产规则进行合并。

上述的分析过程和自顶向上的分析方法完全不同，这两种不同的分析方法其实也代表了计算机科学中两种不同的思想 — 从抽象到具体和从具体到抽象。

##### Lookahead

在语法分析中除了 LL 和 LR 这两种不同类型的语法分析方法之外，还存在另一个非常重要的概念，就是[向前查看（Lookahead）](https://en.wikipedia.org/wiki/Lookahead)，在不同生产规则发生冲突时，当前解析器需要通过预读一些 Token 判断当前应该用什么生产规则对输入流进行展开或者归约[19](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:19)，例如在 LALR(1) 文法中，就需要预读一个 Token 保证出现冲突的生产规则能够被正确处理。



#### Go

Go 语言的解析器就使用了 LALR(1) 的文法来解析词法分析过程中输出的 Token 序列[20](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fn:20)，最右推导加向前查看构成了 Go 语言解析器的最基本原理，也是大多数编程语言的选择。

我们在[概述](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md)中已经介绍了编译器的主函数，该函数调用的 `parseFiles` 就会对使用多个 Goroutine 来解析源文件，解析的过程就会使用 [`Parse`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/syntax/syntax.go#L58-L73) 函数，该函数初始化了一个新的 `parser` 结构体并通过 `fileOrNil` 方法开启对当前文件的词法和语法解析：

```
func Parse(base *PosBase, src io.Reader, errh ErrorHandler, pragh PragmaHandler, mode Mode) (_ *File, first error) {
    var p parser
    p.init(base, src, errh, pragh, mode)
    p.next()
    return p.fileOrNil(), p.first
}
```

[`fileOrNil`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/syntax/parser.go#L352-L425) 方法其实就是对上面介绍的 Go 语言文法的实现，该方法首先会解析文件开头的 `package` 定义：

```
// SourceFile = PackageClause ";" { ImportDecl ";" } { TopLevelDecl ";" } .
func (p *parser) fileOrNil() *File {
    f := new(File)
    f.pos = p.pos()
    if !p.got(_Package) {
        p.syntaxError("package statement must be first")
        return nil
    }
    f.PkgName = p.name()
    p.want(_Semi)
```

从上面的这一段方法中我们可以看出，当前方法会通过 `got` 来判断下一个 Token 是不是 `package` 关键字，如果是 `package` 关键字，就会执行 `name` 来匹配一个包名并将结果保存到返回的文件结构体中。

```
    for p.got(_Import) {
        f.DeclList = p.appendGroup(f.DeclList, p.importDecl)
        p.want(_Semi)
    }
```

确定了当前文件的包名之后，就开始解析可选的 `import` 声明，每一个 `import` 在解析器看来都是一个声明语句，这些声明语句都会被加入到文件的 `DeclList` 列表中。

在这之后就会根据编译器获取的关键字进入 switch 的不同分支，这些分支调用 `appendGroup` 方法并在方法中传入用于处理对应类型语句的 `constDecl`、`typeDecl` 函数。

```
    for p.tok != _EOF {
        switch p.tok {
        case _Const:
            p.next()
            f.DeclList = p.appendGroup(f.DeclList, p.constDecl)
        case _Type:
            p.next()
            f.DeclList = p.appendGroup(f.DeclList, p.typeDecl)
        case _Var:
            p.next()
            f.DeclList = p.appendGroup(f.DeclList, p.varDecl)
        case _Func:
            p.next()
            if d := p.funcDeclOrNil(); d != nil {
                f.DeclList = append(f.DeclList, d)
            }
        }
    }
    f.Lines = p.source.line
    return f
}
```

`fileOrNil` 方法使用了非常多的子方法对输入的文件进行语法分析，并在最后会返回文件最开始创建的 `File` 结构体。

读到这里的人可能会有一些疑惑，为什么没有看到词法分析的代码，这是因为词法分析器 `scanner` 作为结构体被嵌入到了 `parser` 中，所以这个方法中的 `p.next()` 实际上调用的是 `scanner` 的 `next` 方法，它会直接获取文件中的下一个 Token，所以词法分析和语法分析其实是一起进行的。

`fileOrNil` 与在这个方法中执行的其他子方法共同构成了一颗树，这棵树根节点就是 `fileOrNil`，子节点就是 `importDecl`、`constDecl` 等方法，它们与 Go 语言文法中的生产规则一一对应。


![golang-parse](go语言设计与实现.assets/8baf5c1c950701464cee9f56de1c86cd.png)

**图 2-8 Go 语言解析器的方法**

`fileOrNil`、`constDecl` 等方法对应了一个 Go 语言中的生产规则，例如 `fileOrNil` 实现的就是：

```
SourceFile = PackageClause ";" { ImportDecl ";" } { TopLevelDecl ";" } .
```

我们根据这个规则就能很好地理解语法分析器 `parser` 的实现原理 - 将编程语言的所有生产规则映射到对应的方法上，这些方法构成的树形结构最终会返回一个抽象语法树。

因为大多数方法的实现都非常相似，所以这里就仅介绍 `fileOrNil` 方法的实现了，想要了解其他方法的实现原理，读者可以自行查看 [`src/cmd/compile/internal/syntax/parser.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/syntax/parser.go) 文件，该文件包含了语法分析阶段的全部方法。



##### 辅助方法

虽然这里不会展开介绍其他类似方法的实现，但是解析器运行过程中有几个辅助方法我们还是要简单说明一下，首先就是 [`got`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/syntax/parser.go#L158-L164) 和 [`want`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/syntax/parser.go#L166-L171) 这两个常见的方法：

```
func (p *parser) got(tok token) bool {
    if p.tok == tok {
        p.next()
        return true
    }
    return false
}
func (p *parser) want(tok token) {
    if !p.got(tok) {
        p.syntaxError("expecting " + tokstring(tok))
        p.advance()
    }
}
```



`got` 其实只是用于快速判断一些语句中的关键字，如果当前解析器中的 Token 是传入的 Token 就会直接跳过该 Token 并返回 `true`；而 `want` 就是对 `got` 的简单封装了，如果当前的 Token 不是我们期望的，就会立刻返回语法错误并结束这次编译。

这两个方法的引入能够帮助工程师在上层减少判断关键字的大量重复逻辑，让上层语法分析过程的实现更加清晰。

另一个方法 [`appendGroup`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/syntax/parser.go#L469-L489) 的实现就稍微复杂了一点，它的主要作用就是找出批量的定义，我们可以简单举一个例子：

```
var (
   a int
   b int 
)
```

这两个变量其实属于同一个组（Group），各种顶层定义的结构体 `ConstDecl`、`VarDecl` 在进行语法分析时有一个额外的参数 `Group`，这个参数就是通过 [`appendGroup`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/syntax/parser.go#L469-L489) 方法传递进去的：

```
func (p *parser) appendGroup(list []Decl, f func(*Group) Decl) []Decl {
    if p.tok == _Lparen {
        g := new(Group)
        p.list(_Lparen, _Semi, _Rparen, func() bool {
            list = append(list, f(g))
            return false
        })
    } else {
        list = append(list, f(nil))
    }
    return list
}
```

`appendGroup` 方法会调用传入的 `f` 方法对输入流进行匹配并将匹配的结果追加到另一个参数 `File` 结构体中的 `DeclList` 数组中，`import`、`const`、`var`、`type` 和 `func` 声明语句都是调用 `appendGroup` 方法进行解析的。

##### 节点

语法分析器最终会使用不同的结构体来构建抽象语法树中的节点，其中根节点 `File` 我们已经在上面介绍过了，其中包含了当前文件的包名、所有声明结构的列表和文件的行数：

```
type File struct {
    PkgName  *Name
    DeclList []Decl
    Lines    uint
    node
}
```

其他节点的结构体也都在 [`src/cmd/compile/internal/syntax/nodes.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/syntax/nodes.go) 文件中定义了，文件中定义了全部声明类型的结构体，在这里简单看一下函数声明的结构：

```

type (
    Decl interface {
        Node
        aDecl()
    }
    FuncDecl struct {
        Attr   map[string]bool
        Recv   *Field
        Name   *Name
        Type   *FuncType
        Body   *BlockStmt
        Pragma Pragma
        decl
    }
}
```

从函数定义中我们可以看出，函数在语法结构上主要由接受者、函数名、函数类型和函数体几个部分组成，函数体 `BlockStmt` 是由一系列的表达式组成的，这些表达式共同组成了函数的主体：

![golang-funcdecl-struct](go语言设计与实现.assets/3e2bf1a93e46ba68fba345e48f257fcc.png)

**图 2-9 Go 语言函数定义的结构体**

函数的主体其实就是一个 `Stmt` 数组，`Stmt` 是一个接口，实现该接口的类型其实也非常多，总共有 14 种不同类型的 `Stmt` 实现：

![golang-statement](go语言设计与实现.assets/81d2633a9954fdadf3223159b3168055.png)

**图 2-9 Go 语言的 14 种声明**

这些不同类型的 `Stmt` 构成了全部命令式的 Go 语言代码，从中我们可以看到非常多熟悉的控制结构，例如 `if`、`for`、`switch` 和 `select`，这些命令式的结构在其他的编程语言中也非常常见。



### 2.2.3 小结

这一节介绍了 Go 语言的词法分析和语法分析过程，我们不仅从理论的层面介绍了词法和语法分析的原理，还从 Golang 的源代码出发详细分析 Go 语言的编译器是如何在底层实现词法和语法解析功能的。

了解 Go 语言的词法分析器 `scanner` 和语法分析器 `parser` 让我们对解析器处理源代码的过程有着比较清楚的认识，同时我们也在 Go 语言的文法和语法分析器中找到了熟悉的关键字和语法结构，加深了对 Go 语言的理解。



- 通用编程语言 General-purpose programming language https://en.wikipedia.org/wiki/General-purpose_programming_language[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:1)
- 词法分析 Lexical analysis https://en.wikipedia.org/wiki/Lexical_analysis[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:2)
- 词法分析生成器 http://dinosaur.compilertools.net/lex/index.html[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:3)
- 生成的 simplego.lex.c 文件 https://gist.github.com/draveness/85db6ec4a4088b63ccccf7f09424f474[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:4)
- 有限自动机 DFA https://en.wikipedia.org/wiki/Deterministic_finite_automaton[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:5)
- https://github.com/golang/go/tree/master/src/cmd/compile/internal/syntax/scanner.go[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:6)
- https://github.com/golang/go/tree/master/src/cmd/compile/internal/syntax/tokens.go[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:7)
- Go 1.5 Bootstrap Plan https://docs.google.com/document/d/1OaatvGhEAq7VseQ9kkavxKNAfepWy2yhPUBs96FGV28/edit[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:8)
- 语法分析 Syntactic analysis https://en.wikipedia.org/wiki/Parsing[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:9)
- https://en.wikipedia.org/wiki/Context-free_grammar[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:10)
- 终结符和非终结符 https://en.wikipedia.org/wiki/Terminal_and_nonterminal_symbols[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:11)
- 生产规则在计算机科学领域是符号替换的重写规则，S -> aSb 就是可以用右侧的 aSb 将左侧的符号进行展开 [https://en.wikipedia.org/wiki/Production_(computer_science)](https://en.wikipedia.org/wiki/Production_(computer_science))[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:12)
- https://github.com/golang/go/blob/master/src/cmd/compile/internal/syntax/parser.go[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:13)
- The Go Programming Language Specification https://golang.org/ref/spec[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:14)
- 自顶向下解析 https://en.wikipedia.org/wiki/Top-down_parsing[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:15)
- 自底向上解析 https://en.wikipedia.org/wiki/Bottom-up_parsing[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:16)
- LL 文法是一种上下文无关文法，可以使用 LL 解析器解析 https://en.wikipedia.org/wiki/LL_grammar[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:17)
- LR 解析器是一种自底向上的解析器，它有很多 SLR、LALR、等变种 https://en.wikipedia.org/wiki/LR_parser[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:18)
- https://en.wikipedia.org/wiki/Lookahead[↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:19)
- 关于 Go 语言文法的讨论 https://groups.google.com/forum/#!msg/golang-nuts/jVjbH2-emMQ/UdZlSNhd3DwJ [↩︎](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md#fnref:20)

