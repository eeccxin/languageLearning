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

![golang-internal](go语言设计与实现.assets/afa64721a976e9d548f5b973241109db.png)

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

![wechat-account-qrcode](go语言设计与实现.assets/32b916ab562494e6858bc9393dc256b4.png)

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



## 2.3 类型检查

我们在上一节中介绍了 Golang 的第一个编译阶段 — 通过[词法和语法分析器](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md)的解析得到了抽象语法树，在这里就会继续介绍编译器执行的下一个过程 — 类型检查。

提到类型检查和编程语言的类型系统，很多人都会想到几个非常模糊并且难以区分和理解的术语：强类型、弱类型、静态类型和动态类型。这几个术语有的可能在并没有被广泛认同的明确定义，但是我们既然即将谈到 Go 语言编译器的类型检查过程，就不得不讨论一下这些『类型』的含义与异同。



### 2.3.1 强弱类型

[强类型和弱类型](https://en.wikipedia.org/wiki/Strong_and_weak_typing)[1](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md#fn:1)经常会被放在一起讨论，然而这两者并没有一个学术上的严格定义，作者以前也尝试对强弱类型这两个概念进行理解，但是查阅了很多资料之后却发现理解编程语言的类型系统反而更加困难，很多资料都是相互矛盾的、用词含糊不清，也没有足够权威的资料。

![strong-and-weak-typing](go语言设计与实现.assets/c09812eb09f17a74f24492fbea269bcc.png)

**图 2-10 强类型和弱类型**

由于权威的定义的缺失，对于强弱类型来说，我们很多时候也只能根据现象和特性从直觉上进行判断 —— 强类型的编程语言在编译期间会有着更严格的类型限制，也就是编译器会在编译期间发现变量赋值、返回值和函数调用时的类型错误，而弱类型的语言在出现类型错误时可能会在运行时进行隐式的类型转换，在类型转换时可能会造成运行错误[2](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md#fn:2)。

假如我们从上面的定义出发，我们就可以认为 Java、C# 等在编译期间进行类型检查的编程语言往往都是强类型的，同样地按照这个标准，Go 语言因为会在编译期间发现类型错误，所以也应该是强类型的编程语言。

理解强弱类型这两个具有非常明确歧义并且定义不严格的概念是没有太多实际价值的，作为一种抽象的定义，我们使用它更多的时候是为了方便沟通和分类，这对于我们真正使用和理解编程语言可能没有什么帮助，相比没有明确定义的强弱类型，更应该被关注的应该是下面的这些问题：

- 类型的转换是显式的还是隐式的？
- 编译器会帮助我们推断变量的类型么？这些具体的问题在这种语境下其实更有价值，也希望各位读者能够减少和避免对强弱类型的争执。



### 2.3.2 静态与动态类型

静态类型和动态类型的编程语言其实也是两个不精确的表述，它们应该被称为使用[静态类型检查](https://en.wikipedia.org/wiki/Type_system#Static_type_checking)和[动态类型检查](https://en.wikipedia.org/wiki/Type_system#Dynamic_type_checking_and_runtime_type_information)的编程语言，这一小节会分别介绍两种类型检查的特点以及它们的区别。

> 静态类型检查

静态类型检查是基于对源代码的分析来确定运行程序类型安全的过程[3](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md#fn:3)，如果我们的代码能够通过静态类型检查，那么当前程序在一定程度上就满足了类型安全的要求，它也可以被看作是一种代码优化的方式，能够减少程序在运行时的类型检查。

作为一个开发者来说，静态类型检查能够帮助我们在编译期间发现程序中出现的类型错误，一些动态类型的编程语言都会有社区提供的工具为这些编程语言加入静态类型检查，例如 JavaScript 的 [Flow](https://flow.org/)[4](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md#fn:4)，这些工具能够在编译期间发现代码中的类型错误。

相信很多读者也都听过『动态类型一时爽，代码重构火葬场』[5](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md#fn:5)，使用很多编程语言的开发者一定对这句话深有体会，静态类型为代码在编译期间提供了一种约束，如果代码没有满足这种约束就没有办法通过编译器的检查。

在重构时这种静态的类型检查能够帮助我们节省大量的时间并且避免一些遗漏的错误，但是对于仅使用动态类型检查的语言，就需要额外写大量的测试用例保证重构不会出现类型错误了，当然在这里并不是说测试不重要，我们写的**任何代码都应该有良好的测试**，这与语言没有太多的关系。

> 动态类型检查

动态类型检查就是在运行时确定程序类型安全的过程，**这个过程需要编程语言在编译时为所有的对象加入类型标签和信息，运行时就可以使用这些存储的类型信息来实现动态派发、向下转型、反射以及其他特性[6](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md#fn:6)。**

这种类型检查的方式能够为工程师提供更多的操作空间，让我们能在运行时获取一些类型相关的上下文并根据对象的类型完成一些动态操作。

只使用动态类型检查的编程语言就叫做动态类型编程语言，常见的动态类型编程语言就包括 JavaScript、Ruby 和 PHP，这些编程语言在使用上非常灵活也不需要经过编译器的编译，但是有问题的代码该出错还是出错并不会因为更加灵活就会减少错误。

> 小结

静态类型检查和动态类型检查其实并不是两种完全冲突和对立的特点，很多编程语言都会同时使用两种类型检查，Java 就同时使用了这两种检查的方法，不仅在编译期间对类型提前检查发现类型错误，还为对象添加了类型信息，这样能够在运行时使用反射根据对象的类型动态地执行方法减少了冗余代码。



### 2.3.3 执行过程

Go 语言的编译器不仅使用静态类型检查来保证程序运行的类型安全，还会在编程期引入类型信息，让工程师能够使用反射来判断参数和变量的类型，当我们想要将 `interface` 转换成具体类型时就会进行动态类型检查，如果无法发生转换就可能会造成程序崩溃。

我们在这里还是会重点介绍编译期间的静态类型检查，在 [2.1 概述](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md)中，我们曾经介绍过 Go 语言编译器主程序中的 [`Main`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/main.go#L144-L796) 函数，其中有一段是这样的：

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
    checkMapKeys()
```



这段代码的执行过程可以分成两个部分，首先通过 [`src/cmd/compile/internal/gc/typecheck.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/gc/typecheck.go) 文件中的 `typecheck` 函数检查常量、类型、函数声明以及变量赋值语句的类型，然后使用 `checkMapKeys` 检查哈希中键的类型，我们会分几个部分对上述代码的实现原理进行分析。

编译器类型检查的主要逻辑都在 `typecheck` 和 `typecheck1` 这两个函数中，其中 `typecheck` 中逻辑不是特别多，它的主要作用就做一些类型检查之前的准备工作。而核心的类型检查逻辑都在 [typecheck1](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/typecheck.go#L327-L2081) 函数中，这是由一个巨大的 switch/case 构成的 2000 行函数：

```
func typecheck1(n *Node, top int) (res *Node) {
    switch n.Op {
    case OTARRAY:
        ...
    case OTMAP:
        ...
    case OTCHAN:
        ...
    }
    ...    
    return n
}
```

`typecheck1` 根据传入节点操作 `Op` 的不同，进入不同的分支，其中 `Op` 包括加减乘数等操作符、函数调用、方法调用等 150 多种，所有的节点操作 `Op` 都定义在 [`src/cmd/compile/internal/gc/syntax.go`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/syntax.go#L613-L800) 这个文件中，不过由于它的种类确实非常多，所以我们在这里只节选几个比较重要的案例深入分析一下。



#### 切片 OTARRAY

如果当前节点的操作类型是 `OTARRAY`，那么这个分支首先会对右节点，也就是切片或者数组中元素的类型进行类型检查：

```
    case OTARRAY:
        r := typecheck(n.Right, Etype)
        if r.Type == nil {
            n.Type = nil
            return n
        }
```

然后该分支会根据当前节点的左节点不同，分三种不同的情况更新当前 `Node` 的类型，也就是三种不同的声明方式 `[]int`、`[…]int` 和 `[3]int`，第一种相对来说比较简单，这里会直接调用 `NewSlice` 函数：

```
        if n.Left == nil {
            t = types.NewSlice(r.Type)
```

`NewSlice` 函数直接返回了一个 `TSLICE` 类型的结构体，中元素的类型信息 `r.Type` 也会存储在结构体中。当遇到 `[…]int` 这种形式的数组类型时，就会使用 `NewDDDArray` 函数创建一个存储着 `&Array{Elem: elem, Bound: -1}` 结构的 `TARRAY` 类型，`-1` 就代表当前的数组类型的大小需要进行推导：

```
        } else if n.Left.Op == ODDD {
            if top&ctxCompLit == 0 {
                if !n.Diag() {
                    n.SetDiag(true)
                    yyerror("use of [...] array outside of array literal")
                }
                n.Type = nil
                return n
            }
            // t.Extra = &Array{Elem: r.Type, Bound: -1}
            t = types.NewDDDArray(r.Type)
```

在最后，如果源代码中直接包含了数组的大小，就会调用 `NewArray` 函数初始化一个 `TARRAY` 类型的结构体，结构体存储着数组中元素的类型和数组的大小：

```
        } else {
            n.Left = indexlit(typecheck(n.Left, ctxExpr))
            l := n.Left
            v := l.Val()
            bound := v.U.(*Mpint).Int64()
            // t.Extra = &Array{Elem: r.Type, Bound: bound}
            t = types.NewArray(r.Type, bound)        }
        n.Op = OTYPE
        n.Type = t
        n.Left = nil
        n.Right = nil
```

三个不同的分支会分别处理数组和切片声明的不同形式，每一个分支都会更新 `Node` 结构体中存储的类型并修改抽象语法树中的内容。通过对这个片段的分析，我们发现数组的长度是类型检查期间确定的，而 `[…]int` 这种声明形式也只是 Go 语言为我们提供的语法糖。

哈希 OTMAP

对于哈希或者映射来说，编译器会对它的键值类型分别进行检查以验证它们类型的合法性：

```
    case OTMAP:
        n.Left = typecheck(n.Left, Etype)
        n.Right = typecheck(n.Right, Etype)
        l := n.Left
        r := n.Right
        n.Op = OTYPE
        n.Type = types.NewMap(l.Type, r.Type)
        mapqueue = append(mapqueue, n)
        n.Left = nil
        n.Right = nil
```

与处理切片时几乎完全相同，这里会通过 [`NewMap`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/types/type.go#L521-L527) 创建一个新的 `TMAP` 类型并将哈希的键值类型都存储到该结构体中：

```
func NewMap(k, v *Type) *Type {
    t := New(TMAP)
    mt := t.MapType()
    mt.Key = k
    mt.Elem = v
    return t
}
```

代表当前哈希的节点最终也会被加入 `mapqueue` 队列，编译器会在后面的阶段对哈希键的类型进行再次检查，而检查键类型调用的其实就是上面提到的 `checkMapKeys` 函数：

```
func checkMapKeys() {
    for _, n := range mapqueue {
        k := n.Type.MapType().Key
        if !k.Broke() && !IsComparable(k) {
            yyerrorl(n.Pos, "invalid map key type %v", k)
        }
    }
    mapqueue = nil
}
```

该函数会遍历 `mapqueue` 队列中等待检查的节点，判断这些类型能否作为哈希的键，如果当前类型不合法就会在类型检查的阶段直接报错中止整个检查的过程。

#### 关键字 OMAKE

最后要介绍的其实就是 Go 语言中很常见的内置函数 `make`，在类型检查阶段之前，无论是创建切片、哈希还是 Channel 用的都是 `make` 关键字，但是在类型检查阶段就会根据创建的类型将 `make` 替换成本的函数，后面[生成中间代码](https://www.bookstack.cn/read/draveness-golang/5a89e04c79706261.md)的过程就不再会处理 `OMAKE` 类型的节点了，而是会根据这里生成的更加细分的操作类型进行处理：

![golang-keyword-make](go语言设计与实现.assets/091a5fa198f55d436909c85c71e0e9c3-172863322378123.png)

**图 2-4 类型检查阶段对 make 进行改写**

这里会先对关键字 `make` 的第一个参数，也就是类型进行检查并类型的不同进入不同的分支，切片分支 `TSLICE`、哈希分支 `TMAP` 和 Channel 分支 `TCHAN`：

```
    case OMAKE:
        args := n.List.Slice()
        n.List.Set(nil)
        l := args[0]
        l = typecheck(l, Etype)
        t := l.Type
        i := 1
        switch t.Etype {
        case TSLICE:
            // ...
        case TMAP:
            // ...
        case TCHAN:
            // ...
        }
        n.Type = t
```

如果 `make` 的第一个参数是切片类型，那么就会从参数中获取切片的长度 `len` 和容量 `cap` 并对这两个参数进行校验，其中包括：

- 切片的长度参数是否被传入；
- 切片的长度必须要小于或者等于切片的容量；

```
        case TSLICE:
            if i >= len(args) {
                yyerror("missing len argument to make(%v)", t)
                n.Type = nil
                return n
            }
            l = args[i]
            i++
            l = typecheck(l, ctxExpr)
            var r *Node
            if i < len(args) {
                r = args[i]
                i++
                r = typecheck(r, ctxExpr)
            }
            if Isconst(l, CTINT) && r != nil && Isconst(r, CTINT) && l.Val().U.(*Mpint).Cmp(r.Val().U.(*Mpint)) > 0 {
                yyerror("len larger than cap in make(%v)", t)
                n.Type = nil
                return n
            }
            n.Left = l
            n.Right = r
            n.Op = OMAKESLICE
```

除了对参数的数量和合法性进行校验，这段代码最后会将当前节点的操作 `Op` 改成 `OMAKESLICE`，方便后面编译阶段的处理。

第二种情况就是 `make` 的第一个参数是 `map` 类型，在这种情况下，第二个可选的参数就是哈希的初始大小，在默认情况下它的大小是 0，当前分支最后也会改变当前节点的 `Op` 属性：

```
        case TMAP:
            if i < len(args) {
                l = args[i]
                i++
                l = typecheck(l, ctxExpr)
                l = defaultlit(l, types.Types[TINT])
                if !checkmake(t, "size", l) {
                    n.Type = nil
                    return n
                }
                n.Left = l
            } else {
                n.Left = nodintconst(0)
            }
            n.Op = OMAKEMAP
```



`make` 内置函数能够初始化的最后一种结构就是 [Channel](https://www.bookstack.cn/read/draveness-golang/c666731d5f1a2820.md) 了，从下面的代码我们可以发现第二个参数表示的就是该 Channel 的缓冲区大小，如果不存在第二个参数，那么就会创建缓冲区大小为 0 的 Channel：

```
        case TCHAN:
            l = nil
            if i < len(args) {
                l = args[i]
                i++
                l = typecheck(l, ctxExpr)
                l = defaultlit(l, types.Types[TINT])
                if !checkmake(t, "buffer", l) {
                    n.Type = nil
                    return n
                }
                n.Left = l
            } else {
                n.Left = nodintconst(0)
            }
            n.Op = OMAKECHAN
```

在类型检查的过程中，无论 `make` 的第一个参数是什么类型，都会对当前节点的 `Op` 类型进行修改并且对传入参数的合法性进行一定的验证。



### 2.3.4 小结

类型检查是 Go 语言编译的第二个阶段，在词法和语法分析之后我们得到了每个文件对应的抽象语法树，随后的类型检查会遍历抽象语法树中的节点，对每个节点的类型进行检验，找出其中存在的语法错误，在这个过程中也可能会对抽象语法树进行改写，这不仅能够去除一些不会被执行的代码、对代码进行优化以提高执行效率，而且也会修改 `make`、`new` 等关键字对应节点的操作类型。

`make` 和 `new` 这些内置函数其实并不直接对应某些函数的实现，它们会在编译期间被转换成真正存在的其他函数，我们在下一节[中间代码生成](https://www.bookstack.cn/read/draveness-golang/5a89e04c79706261.md)中会介绍编译器对它们做了什么。

------

- Strong and weak typing https://en.wikipedia.org/wiki/Strong_and_weak_typing[↩︎](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md#fnref:1)
- Weak And Strong Typing https://wiki.c2.com/?WeakAndStrongTyping[↩︎](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md#fnref:2)
- https://en.wikipedia.org/wiki/Type_system#Static_type_checking[↩︎](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md#fnref:3)
- JavaScript 静态检查工具 https://flow.org/[↩︎](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md#fnref:4)
- 为什么说“动态类型一时爽，代码重构火葬场”？ https://www.zhihu.com/question/30072490[↩︎](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md#fnref:5)
- https://en.wikipedia.org/wiki/Type_system#Dynamic_type_checking_and_runtime_type_information[↩︎](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md#fnref:6)



## 2.4 中间代码生成

前两节介绍的[词法与语法分析](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md)以及[类型检查](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md)两个部分都属于编译器前端，它们负责对源代码进行分析并检查其中存在的词法和语法错误，经过这两个阶段生成的抽象语法树已经几乎不存在结构上的错误了，从这一节开始就进入了编译器后端的工作 —— 中间代码生成。

### 2.4.1 概述

[中间代码](https://en.wikipedia.org/wiki/Intermediate_representation)是指一种应用于抽象机器的编程语言，它设计的目的，是用来帮助我们分析计算机程序。在编译的过程中，编译器会在将源代码转换成目标机器上机器码的过程中，先把源代码转换成一种中间的表述形式[1](https://www.bookstack.cn/read/draveness-golang/5a89e04c79706261.md#fn:1)。

![intermediate-representation](go语言设计与实现.assets/5a43582b0327f01696abaaf3fe71c52b.png)

**图 2-12 源代码、中间代码和机器码**

很多读者可能认为中间代码没有太多价值，我们也可以直接将源代码翻译成目标语言，这样就能省略编译的步骤，这种看起来可行的办法实际上有很多问题，它忽略了编译器需要面对的复杂场景，很多编译器可能需要将一种源代码翻译成多种机器码，想要在高级的编程语言上直接进行翻译是比较困难的，但是我们使用中间代码就可以将我们的问题简化：

- 将编程语言直接翻译成机器码的过程拆成两个简单步骤 —— 中间代码生成和机器码生成；
- 中间代码是一种更接近机器语言的表示形式，对中间代码的优化和分析相比直接分析高级编程语言更容易；Go 语言编译器的中间代码具有静态单赋值（SSA）的特性，我们在介绍 [Go 语言编译过程](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md)中曾经介绍过静态单赋值，对这个特性不了解的读者可以回到上面的章节阅读相关的内容。

我们再来回忆一下编译阶段入口的主函数 [`Main`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/main.go#L144-L796) 中关于中间代码生成的部分，在这一段代码中会初始化 SSA 生成的配置，在配置初始化结束之后会调用 `funccompile` 对函数进行编译：

```
func Main(archInit func(*Arch)) {
    // ...
    initssaconfig()
    for i := 0; i < len(xtop); i++ {
        n := xtop[i]
        if n.Op == ODCLFUNC {
            funccompile(n)
        }
    }
    compileFunctions()
}
```

这一节将分别介绍配置的初始化以及函数编译两部分内容，我们会以 `initssaconfig` 和 `funccompile` 这两个函数作为入口来分析中间代码生成的具体过程和实现原理。

### 2.4.2 配置初始化

SSA 配置的初始化过程其实就是做中间代码生成之前的准备工作，在这个过程中我们会缓存可能用到的类型指针、初始化 SSA 配置和一些之后会调用的运行时函数，例如：用于处理 `defer` 关键字的 `deferproc`、用于创建 Goroutine 的 `newproc` 和扩容切片的 `growslice` 等，除此之外还会根据当前的目标设备初始化特定的 ABI[2](https://www.bookstack.cn/read/draveness-golang/5a89e04c79706261.md#fn:2)。我们以 [`initssaconfig`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/ssa.go#L39-L172) 作为入口开始分析配置初始化的过程。

```
func initssaconfig() {
    types_ := ssa.NewTypes()
    _ = types.NewPtr(types.Types[TINTER])                             // *interface{}
    _ = types.NewPtr(types.NewPtr(types.Types[TSTRING]))              // **string
    _ = types.NewPtr(types.NewPtr(types.Idealstring))                 // **string
    _ = types.NewPtr(types.NewSlice(types.Types[TINTER]))             // *[]interface{}
    ..
    _ = types.NewPtr(types.Errortype)                                 // *error
```



这个函数的执行过程总共可以被分成三个部分，首先就是调用 [`NewTypes`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/ssa/config.go#L81-L85) 初始化一个新的 `Types` 结构体并调用 `NewPtr` 函数缓存类型的信息，`Types` 结构体中存储了所有 Go 语言中基本类型对应的指针，比如 `Bool`、`Int8`、以及 `String` 等。

![golang-type-and-pointer](go语言设计与实现.assets/324f612787109de296376b6d78c9adb8.png)

**图 2-12 类型和类型指针**

[`NewPtr`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/types/type.go#L535-L555) 函数的主要作用就是根据类型生成指向这些类型的指针，同时它会根据编译器的配置将生成的指针类型缓存在当前类型中，优化类型指针的获取效率：

```
func NewPtr(elem *Type) *Type {
    if t := elem.Cache.ptr; t != nil {
        if t.Elem() != elem {
            Fatalf("NewPtr: elem mismatch")
        }
        return t
    }
    t := New(TPTR)
    t.Extra = Ptr{Elem: elem}
    t.Width = int64(Widthptr)
    t.Align = uint8(Widthptr)
    if NewPtrCacheEnabled {
        elem.Cache.ptr = t
    }
    return t
}
```

配置初始化的第二步就是根据当前的 CPU 架构初始化 SSA 配置 `ssaConfig`，我们会向 `NewConfig` 函数传入目标机器的 CPU 架构、上述代码初始化的 `Types` 结构体、上下文信息和 Debug 配置：

```
    ssaConfig = ssa.NewConfig(thearch.LinkArch.Name, *types_, Ctxt, Debug['N'] == 0)
```

[`NewConfig`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/ssa/config.go#L197-L367) 会根据传入的 CPU 架构设置用于生成中间代码和机器码的函数，当前编译器使用的指针、寄存器大小、可用寄存器列表、掩码等编译选项：

```
func NewConfig(arch string, types Types, ctxt *obj.Link, optimize bool) *Config {
    c := &Config{arch: arch, Types: types}
    c.useAvg = true
    c.useHmul = true
    switch arch {
    case "amd64":
        c.PtrSize = 8
        c.RegSize = 8
        c.lowerBlock = rewriteBlockAMD64
        c.lowerValue = rewriteValueAMD64
        c.registers = registersAMD64[:]
        ...
    case "arm64":
    ...
    case "wasm":
    default:
        ctxt.Diag("arch %s not implemented", arch)
    }
    c.ctxt = ctxt
    c.optimize = optimize
    // ...
    return c
}
```

所有的配置项一旦被创建，在整个编译期间都是只读的并且被全部编译阶段共享，也就是中间代码生成和机器码生成这两部分都会使用这一份配置完成自己的工作。在 `initssaconfig` 方法调用的最后，会初始化一些编译器会用到的 Go 语言运行时的函数：

```
    assertE2I = sysfunc("assertE2I")
    assertE2I2 = sysfunc("assertE2I2")
    assertI2I = sysfunc("assertI2I")
    assertI2I2 = sysfunc("assertI2I2")
    deferproc = sysfunc("deferproc")
    Deferreturn = sysfunc("deferreturn")
    ...
```

这些函数会在对应的 runtime 包结构体 `Pkg` 中创建一个新的符号 `obj.LSym`，表示上述的方法已经被注册到运行时 runtime 包中，我们在后面的中间代码生成中直接使用这些方法，我们在这里看到的 `deferproc` 和 `deferreturn` 就是 Go 语言用于实现 `defer` 关键字的运行时函数，你能在后面的章节 [5.3 defer](https://www.bookstack.cn/read/draveness-golang/f434f07b7b465a9f.md) 中了解更多内容。



### 2.4.3 遍历和替换

在生成中间代码之前，我们还需要对抽象语法树中节点的一些元素进行替换，这个替换的过程就是通过 `walk` 和很多以 `walk` 开头的相关函数实现的，简单展示几个相关函数的签名：

```
func walk(fn *Node)
func walkappend(n *Node, init *Nodes, dst *Node) *Node
...
func walkrange(n *Node) *Node
func walkselect(sel *Node)
func walkselectcases(cases *Nodes) []*Node
func walkstmt(n *Node) *Node
func walkstmtlist(s []*Node)
func walkswitch(sw *Node)
```

**这些用于遍历抽象语法树的函数会将一些关键字和内建函数转换成函数调用**，例如： `panic`、`recover` 这两个内建函数就会被在 `walkXXX` 中被转换成 `gopanic` 和 `gorecover` 两个真正存在的函数，而关键字 `new` 也会在这里被转换成对 `newobject` 函数的调用。



![golang-keyword-and-builtin-mapping](go语言设计与实现.assets/dadd66f00a53f00d419e014fd6c83301.png)

**图 2-13 关键字和操作符和运行时函数的映射**

上图是从关键字或内建函数到其他实际存在的运行时函数的映射，包括 Channel、哈希相关的操作、用于创建结构体对象的 `make`、`new` 关键字以及一些控制流中的关键字 `select` 等。转换后的全部函数都属于运行时 `runtime` 包，我们能在 [`src/cmd/compile/internal/gc/builtin/runtime.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/gc/builtin/runtime.go) 文件中找到函数对应的签名和定义。

```
func makemap64(mapType *byte, hint int64, mapbuf *any) (hmap map[any]any)
func makemap(mapType *byte, hint int, mapbuf *any) (hmap map[any]any)
func makemap_small() (hmap map[any]any)
func mapaccess1(mapType *byte, hmap map[any]any, key *any) (val *any)
...
func makechan64(chanType *byte, size int64) (hchan chan any)
func makechan(chanType *byte, size int) (hchan chan any)
...
```

[`src/cmd/compile/internal/gc/builtin/runtime.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/gc/builtin/runtime.go) 文件中代码的作用只是让编译器能够找到对应符号的函数定义而已，真正的函数实现都在另一个 [`src/runtime`](https://github.com/golang/go/tree/master/src/runtime) 包中。简单总结一下，编译器会将 Go 语言关键字转换成 `runtime` 包中的函数，也就是说关键字和内置函数的功能是由语言的编译器和运行时共同完成的。

我们简单了解一下遍历节点时几个 Channel 操作是如何转换成运行时对应方法的，首先介绍向 Channel 中发送消息或者从 Channel 中接受消息两个操作，编译器会分别使用 `OSEND` 和 `ORECV` 表示发送和接收消息两个操作，在 [`walkexpr`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/walk.go#L439-L1532) 函数中会根据节点类型的不同进入不同的分支：

```
func walkexpr(n *Node, init *Nodes) *Node {
    ...
    case OSEND:
        n1 := n.Right
        n1 = assignconv(n1, n.Left.Type.Elem(), "chan send")
        n1 = walkexpr(n1, init)
        n1 = nod(OADDR, n1, nil)
        n = mkcall1(chanfn("chansend1", 2, n.Left.Type), nil, init, n.Left, n1)
    ...
}
```

当遇到 `OSEND` 操作时，会使用 `mkcall1` 创建一个操作为 `OCALL` 的节点，这个节点中包含当前调用的函数 `chansend1` 和几个参数，新的 `OCALL` 节点会替换当前的 `OSEND` 节点，这也就完成了对 `OSEND` 子树的改写。



![golang-ocall-node](go语言设计与实现.assets/4b961c6fe19eb34320d1aee1309a8bef.png)

**图 2-14 改写后的 Channel 发送操作**

在中间代码生成的阶段遇到 `ORECV` 操作时，编译器的处理与遇到 `OSEND` 时相差无几，我们也只是将 `chansend1` 换成了 `chanrecv1`，其他的参数没有发生太大的变化：

```
        n = mkcall1(chanfn("chanrecv1", 2, n.Left.Type), nil, &init, n.Left, nodnil())
```

使用 `close` 关键字的 `OCLOSE` 操作也会在 `walkexpr` 函数中被转换成调用 `closechan` 的 `OCALL` 节点：

```
func walkexpr(n *Node, init *Nodes) *Node {
    ...
    case OCLOSE:
        fn := syslook("closechan")
        fn = substArgTypes(fn, n.Left.Type)
        n = mkcall1(fn, nil, init, n.Left)
    ...
}
```



对于 Channel 的这些内置操作都会在编译期间就转换成几个运行时执行的函数，很多人都想要了解 Channel 底层的实现，但是并不知道函数的入口，经过这里的分析我们就知道`chanrecv1`、`chansend1` 和 `closechan` 几个函数分别实现了 Channel 的发送、接受和关闭操作。

2.4.4 SSA 生成

经过 `walk` 系列函数的处理之后，AST 的抽象语法树就不再会改变了，Go 语言的编译器会使用 [`compileSSA`](https://github.com/golang/go/blob/06b12e660c239541c973ea9340f00455b9c5a266/src/cmd/compile/internal/gc/pgen.go#L297-L326) 函数将抽象语法树转换成中间代码，我们可以先看一下该函数的简要实现：

```
func compileSSA(fn *Node, worker int) {
    f := buildssa(fn, worker)
    pp := newProgs(fn, worker)
    genssa(f, pp)
    pp.Flush()
}
```

`buildssa` 就是用来具有 SSA 特性的中间代码的函数，我们可以使用命令行工具来观察当前中间代码的生成过程，假设我们有以下的 Go 语言源代码，其中只包含一个非常简单的 `hello` 函数：

```
package hello
func hello(a int) int {
    c := a + 2
    return c
}
```

我们可以使用 `GOSSAFUNC` 环境变量构建上述代码并获取从源代码到最终的中间代码经历的几十次迭代，所有的数据都被存储到了 `ssa.html` 文件中：

```
$ GOSSAFUNC=hello go build hello.go
# command-line-arguments
dumped SSA to ./ssa.html
```



这个文件中包含源代码对应的抽象语法树、几十个版本的中间代码以及最终生成的 SSA，在这里截取文件中的一部分为大家展示一下，让各位读者对这个文件中的内容有更具体的印象：

![ssa-htm](go语言设计与实现.assets/b2b28676d2373310221f9989a8655bfd.png)

**图 2-15 SSA 中间代码生成过程**

如上图所示，其中最左侧就是源代码，中间是源代码生成的抽象语法树，最右侧是生成的第一轮中间代码，后面还有几十轮，感兴趣的读者可以自己尝试编译一下。`hello` 函数对应的抽象语法树会包含当前函数的 `Enter`、`NBody` 和 `Exit` 三个属性，输出这些属性的工作是由下面的函数 [`buildssa`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/ssa.go#L281-L451) 完成的，你能从这个简化的逻辑中看到上述输出的影子：

```
func buildssa(fn *Node, worker int) *ssa.Func {
    name := fn.funcname()
    var astBuf *bytes.Buffer
    var s state
    fe := ssafn{
        curfn: fn,
        log:   printssa && ssaDumpStdout,
    }
    s.curfn = fn
    s.f = ssa.NewFunc(&fe)
    s.config = ssaConfig
    s.f.Type = fn.Type
    s.f.Config = ssaConfig
    ...
    s.stmtList(fn.Func.Enter)
    s.stmtList(fn.Nbody)
    ssa.Compile(s.f)
    return s.f
}
```

`ssaConfig` 就是我们在这里的第一小节初始化的结构体，其中包含了与 CPU 架构相关的函数和配置，随后的中间代码生成其实也分成两个阶段，第一个阶段是使用 `stmtList` 以及相关函数将抽象语法树转换成中间代码，第二个阶段会调用 [`src/cmd/compile/internal/ssa`](https://github.com/golang/go/tree/master/src/cmd/compile/internal/ssa) 包的 [`Compile`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/ssa/compile.go#L29-L155) 函数对 SSA 中间代码进行多轮的迭代和转换。



#### AST 到 SSA

`stmtList` 方法的主要功能就是为传入数组中的每一个节点调用 [`stmt`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/ssa.go#L1023-L1502) 方法，在这个方法中编译器会根据节点操作符的不同将当前 AST 转换成对应的中间代码：

```
func (s *state) stmt(n *Node) {
    ...
    switch n.Op {
    case OCALLMETH, OCALLINTER:
        s.call(n, callNormal)
        if n.Op == OCALLFUNC && n.Left.Op == ONAME && n.Left.Class() == PFUNC {
            if fn := n.Left.Sym.Name; compiling_runtime && fn == "throw" ||
                n.Left.Sym.Pkg == Runtimepkg && (fn == "throwinit" || fn == "gopanic" || fn == "panicwrap" || fn == "block" || fn == "panicmakeslicelen" || fn == "panicmakeslicecap") {
                m := s.mem()
                b := s.endBlock()
                b.Kind = ssa.BlockExit
                b.SetControl(m)
            }
        }
    case ODEFER:
        s.call(n.Left, callDefer)
    case OGO:
        s.call(n.Left, callGo)
    ...
    }
}
```

从上面节选的代码中我们会发现，在遇到函数调用、方法调用、使用 defer 或者 go 关键字时都会执行 [`call`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/ssa.go#L4324-L4517) 生成调用函数的 SSA 节点，这些在开发者看来不同的概念在编译器中都会被实现成静态的函数调用，上层的关键字和方法其实都是语言为我们提供的**语法糖**：

```
func (s *state) call(n *Node, k callKind) *ssa.Value {
    ...
    var call *ssa.Value
    switch {
    case k == callDefer:
        call = s.newValue1A(ssa.OpStaticCall, types.TypeMem, deferproc, s.mem())
    case k == callGo:
        call = s.newValue1A(ssa.OpStaticCall, types.TypeMem, newproc, s.mem())
    case sym != nil:
        call = s.newValue1A(ssa.OpStaticCall, types.TypeMem, sym.Linksym(), s.mem())
    ..
    }
    ...
}
```

首先，从 AST 到 SSA 的转化过程中，编译器会生成将函数调用的参数放到栈上的中间代码，处理参数之后才会生成一条运行函数的命令 `ssa.OpStaticCall`：

- 如果这里使用的是 defer 关键字，就会插入 `deferproc` 函数；
- 如果使用 go 创建新的 Goroutine 时会插入 `newproc` 函数符号；
- 在遇到其他情况时会插入表示普通函数对应的符号；[`src/cmd/compile/internal/gc/ssa.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/gc/ssa.go) 这个拥有将近 7000 行代码的文件包含用于处理不同节点的各种方法，编译器会根据节点类型的不同在一个巨型 switch 语句处理不同的情况，这也是我们在编译器这种独特的场景下才能看到的现象。

```
compiling hello
hello func(int) int
  b1:
    v1 = InitMem <mem>
    v2 = SP <uintptr>
    v3 = SB <uintptr> DEAD
    v4 = LocalAddr <*int> {a} v2 v1 DEAD
    v5 = LocalAddr <*int> {~r1} v2 v1
    v6 = Arg <int> {a}
    v7 = Const64 <int> [0] DEAD
    v8 = Const64 <int> [2]
    v9 = Add64 <int> v6 v8 (c[int])
    v10 = VarDef <mem> {~r1} v1
    v11 = Store <mem> {int} v5 v9 v10
    Ret v11
```

上述代码就是在这个过程生成的，你可以看到中间代码主体中的每一行其实都定义了一个新的变量，这也就是我们在前面提到的具有静态单赋值（SSA）特性的中间代码，如果你使用 `GOSSAFUNC=hello go build hello.go` 命令亲自尝试一下会对这种中间代码有更深的印象。



#### 多轮转换

虽然我们在 `stmt` 以及相关方法中生成了 SSA 中间代码，但是这些中间代码仍然需要编译器进行优化以去掉无用代码并对操作数进行精简，编译器对中间代码的优化过程都是由 [`src/cmd/compile/internal/ssa`](https://github.com/golang/go/tree/master/src/cmd/compile/internal/ssa) 包的 [`Compile`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/ssa/compile.go#L29-L155) 函数执行的：

```
func Compile(f *Func) {
    if f.Log() {
        f.Logf("compiling %s\n", f.Name)
    }
    phaseName := "init"
    for _, p := range passes {
        f.pass = &p
        p.fn(f)
    }
    phaseName = ""
}
```

这是删除了很多打印日志和性能分析功能的 `Compile` 函数，SSA 需要经历的多轮处理也都保存在了 `passes` 变量中，这个变量中存储了每一轮处理的名字、使用的函数以及表示是否必要的 `required` 字段：

```
var passes = [...]pass{
    {name: "number lines", fn: numberLines, required: true},
    {name: "early phielim", fn: phielim},
    {name: "early copyelim", fn: copyelim},
    ...
    {name: "loop rotate", fn: loopRotate},
    {name: "stackframe", fn: stackframe, required: true},
    {name: "trim", fn: trim},
}
```

目前的编译器总共引入了将近 50 个需要执行的过程，我们能在 `GOSSAFUNC=hello go build hello.go` 命令生成的文件中看到每一轮处理后的中间代码，例如最后一个 `trim` 阶段就生成了如下的 SSA 代码：

```
  pass trim begin
  pass trim end [738 ns]
hello func(int) int
  b1:
    v1 = InitMem <mem>
    v10 = VarDef <mem> {~r1} v1
    v2 = SP <uintptr> : SP
    v6 = Arg <int> {a} : a[int]
    v8 = LoadReg <int> v6 : AX
    v9 = ADDQconst <int> [2] v8 : AX (c[int])
    v11 = MOVQstore <mem> {~r1} v2 v9 v10
    Ret v11
```



经过将近 50 轮处理的中间代码相比处理之前已经有了非常大的改变，执行效率会有比较大的提升，多轮的处理已经包含了一些机器特定的修改，包括根据目标架构对代码进行改写，不过这里就不会展开介绍每一轮处理的具体内容了。

### 2.4.5 小结

中间代码的生成过程其实就是从 AST 抽象语法树到 SSA 中间代码的转换过程，在这期间会对语法树中的关键字在进行一次改写，改写后的语法树会经过多轮处理转变成最后的 SSA 中间代码，这里的代码大都是巨型的 switch 语句、复杂的函数以及调用栈，阅读和分析起来也非常困难。

很多 Go 语言中的关键字和内置函数都是在这个阶段被转换成运行时包中方法的，作者在后面的章节会从具体的语言关键字和内置函数的角度介绍一些数据结构和内置函数的实现。

- 中间代码，也被翻译成中间表示，即 Intermediate representation https://en.wikipedia.org/wiki/Intermediate_representation[↩︎](https://www.bookstack.cn/read/draveness-golang/5a89e04c79706261.md#fnref:1)
- 应用程序二进制接口，Application Binary Interface（ABI） https://en.wikipedia.org/wiki/Application_binary_interface[↩︎](https://www.bookstack.cn/read/draveness-golang/5a89e04c79706261.md#fnref:2)



## 2.5 机器码生成

Go 语言编译的最后一个阶段就是根据 SSA 中间代码生成机器码了，这里谈的**机器码就是在目标 CPU 架构上能够运行的二进制代码**，[中间代码生成](https://www.bookstack.cn/read/draveness-golang/5a89e04c79706261.md)一节简单介绍的从抽象语法树到 SSA 中间代码的生成过程，将近 50 个生成中间代码的步骤中有一些过程严格上说是属于机器码生成阶段的。

机器码的生成过程其实就是对 SSA 中间代码的降级（lower）过程，在 SSA 中间代码降级的过程中，编译器将一些值重写成了目标 CPU 架构的特定值，降级的过程处理了所有机器特定的重写规则并对代码进行了一定程度的优化；在 SSA 中间代码生成阶段的最后，Go 函数体的代码会被转换成一系列的 `obj.Prog` 结构体。

### 2.5.1 指令集架构

首先需要介绍的就是指令集架构了，虽然我们在第一节[编译过程概述](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md)中曾经讲解过指令集架构的相关知识，但是在这里还是需要引入更多的指令集构知识。

![instruction-set-architecture](go语言设计与实现.assets/e4197e719afa965f29928ba920b83742.png)

**图 2-16 计算机软硬件之间的桥梁**

[指令集架构](https://en.wikipedia.org/wiki/Instruction_set_architecture)是计算机的抽象模型，在很多时候也被称作架构或者计算机架构，它是**计算机软件和硬件之间的接口和桥梁**[1](https://www.bookstack.cn/read/draveness-golang/100aabc3275d17e5.md#fn:1)；

一个为特定指令集架构编写的应用程序能够运行在所有支持这种指令集架构的机器上，也就说如果当前应用程序支持 x86 的指令集，那么就可以运行在所有使用 x86 指令集的机器上，这其实就是抽象层的作用，每一个指令集架构都定义了支持的数据结构、寄存器、管理主内存的硬件支持（例如内存一致、地址模型和虚拟内存）、支持的指令集和 IO 模型，它的引入其实就在软件和硬件之间引入了一个抽象层，让同一个二进制文件能够在不同版本的硬件上运行。

如果一个编程语言想要在所有的机器上运行，它就可以将中间代码转换成使用不同指令集架构的机器码，这可比为不同硬件单独移植要简单的太多了。



![cisc-and-ris](go语言设计与实现.assets/284e890a46f7d7c5005feaa125d14fd4.png)

**图 2-17 复杂指令集（CISC）和精简指令集（RISC）**

最常见的指令集架构分类方法就是根据指令的复杂度将其分为复杂指令集（CISC）和精简指令集（RISC），复杂指令集架构包含了很多特定的指令，但是其中的一些指令很少会被程序使用，而精简指令集只实现了经常被使用的指令，不常用的操作都会通过组合简单指令来实现。



[复杂指令集](https://en.wikipedia.org/wiki/Complex_instruction_set_computer)的特点就是指令数目多并且复杂，每条指令的字节长度并不相等，**x86 就是常见的复杂指令集处理器**，它的指令长度大小范围非常广，从 1 到 15 字节不等，对于长度不固定的指令，计算机必须额外对指令进行判断，这需要付出额外的性能损失[2](https://www.bookstack.cn/read/draveness-golang/100aabc3275d17e5.md#fn:2)。



而[精简指令集](https://en.wikipedia.org/wiki/Reduced_instruction_set_computer)对指令的数目和寻址方式做了精简，大大减少指令数量的同时更容易实现，指令集中的每一个指令都使用标准的字节长度、执行时间相比复杂指令集会少很多，处理器在处理指令时也可以流水执行，提高了对并行的支持，**作为一种常见的精简指令集处理器，amd 使用 4 个字节作为指令的固定长度**，省略了判断指令的性能损失[3](https://www.bookstack.cn/read/draveness-golang/100aabc3275d17e5.md#fn:3)，精简指令集其实就是利用了我们耳熟能详的 20/80 原则，用 20% 的基础指令和它们的组合来解决问题。



最开始的计算机使用复杂指令集是因为当时计算机的性能和内存比较有限，业界需要尽可能地减少机器需要执行的指令，所以更倾向于高度编码、长度不等以及多操作数的指令，但是随着性能的飞速提升，就出现了精简指令集这种牺牲代码密度换取简单实现的设计，不仅如此，硬件的飞速提升还带来了更多的寄存器和更高的时钟频率，软件开发人员也不再直接接触汇编代码，而是通过编译器和汇编器生成指令，复杂的机器指定对于编译器来说很难利用，所以精简的指令在这种场景下更适合。

复杂指令集和精简指令集的使用其实是一种权衡，经过这么多年的发展，两种指令集也相互借鉴和学习，与最开始刚被设计出来时已经有了较大的差别，对于软件工程师来讲，复杂的硬件设备对于我们来说已经是领域下两层的知识了，其实不太需要掌握太多，但是对指令集架构感兴趣的读者可以简单找一些资料开拓眼界。



### 2.5.2 机器码生成

机器码的生成在 Go 的编译器中主要由两部分协同工作，其中一部分是负责 SSA 中间代码降级和根据目标架构进行特定处理的 [`src/cmd/compile/internal/ssa`](https://github.com/golang/go/tree/master/src/cmd/compile/internal/ssa) 包，另一部分是负责生成机器码的 [`src/cmd/internal/obj`](https://github.com/golang/go/tree/master/src/cmd/internal/obj)[4](https://www.bookstack.cn/read/draveness-golang/100aabc3275d17e5.md#fn:4)：

- [`src/cmd/compile/internal/ssa`](https://github.com/golang/go/tree/master/src/cmd/compile/internal/ssa) 主要负责对 SSA 中间代码进行降级、执行架构特定的优化和重写并生成 `obj.Prog` 指令；
- [`src/cmd/internal/obj`](https://github.com/golang/go/tree/master/src/cmd/internal/obj) 作为一个汇编器会将这些指令最终转换成机器码完成这次的编译；

#### SSA 降级

SSA 的降级是在中间代码生成的过程中完成的，其中将近 50 轮处理的过程中，`lower` 以及后面的阶段都属于 SSA 降级这一过程，这么多轮的处理会将 SSA 转换成机器特定的操作：

```
var passes = [...]pass{
    ...
    {name: "lower", fn: lower, required: true},
    {name: "lowered deadcode for cse", fn: deadcode}, // deadcode immediately before CSE avoids CSE making dead values live again
    {name: "lowered cse", fn: cse},
    ...
    {name: "trim", fn: trim}, // remove empty blocks
}
```

SSA 降级执行的第一个阶段就是 `lower`，该阶段的入口方法就是 [`src/cmd/compile/internal/ssa/lower.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/ssa/lower.go) 文件中的 [`lower`](https://github.com/golang/go/blob/c170b14c2c1cfb2fd853a37add92a82fd6eb4318/src/cmd/compile/internal/ssa/lower.go#L8-L11) 函数，它会将 SSA 的中间代码转换成机器特定的指令：

```
func lower(f *Func) {
    applyRewrite(f, f.Config.lowerBlock, f.Config.lowerValue)
}
```

向 `applyRewrite` 传入的两个函数 `lowerBlock` 和 `lowerValue` 是在[中间代码生成](https://www.bookstack.cn/read/draveness-golang/5a89e04c79706261.md)阶段初始化 SSA 配置时确定的，这两个函数会分别转换一个函数中的代码块和代码块中的值。

假设目标机器使用 x86 的架构，最终会调用 [`rewriteBlock386`](https://github.com/golang/go/blob/c170b14c2c1cfb2fd853a37add92a82fd6eb4318/src/cmd/compile/internal/ssa/rewrite386.go#L21977-L23093) 和 [`rewriteValue386`](https://github.com/golang/go/blob/c170b14c2c1cfb2fd853a37add92a82fd6eb4318/src/cmd/compile/internal/ssa/rewrite386.go#L9-L709) 两个函数，这两个函数是两个巨大的 switch/case，前者总共有 2000 多行，后者将近 700 行，用于处理 x86 架构重写的函数总共有将近 30000 行代码，你能在 [`src/cmd/compile/internal/ssa/rewrite386.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/ssa/rewrite386.go) 这里找到文件的全部内容，我们只节选其中的一段简单展示一下：

```
func rewriteValue386(v *Value) bool {
    switch v.Op {
    case Op386ADCL:
        return rewriteValue386_Op386ADCL_0(v)
    case Op386ADDL:
        return rewriteValue386_Op386ADDL_0(v) || rewriteValue386_Op386ADDL_10(v) || rewriteValue386_Op386ADDL_20(v)
    ...
    }
}
func rewriteValue386_Op386ADCL_0(v *Value) bool {
    // match: (ADCL x (MOVLconst [c]) f)
    // cond:
    // result: (ADCLconst [c] x f)
    for {
        _ = v.Args[2]
        x := v.Args[0]
        v_1 := v.Args[1]
        if v_1.Op != Op386MOVLconst {
            break
        }
        c := v_1.AuxInt
        f := v.Args[2]
        v.reset(Op386ADCLconst)
        v.AuxInt = c
        v.AddArg(x)
        v.AddArg(f)
        return true
    }
    ...
}
```

重写的过程会将通用的 SSA 中间代码转换成目标架构特定的指令，上述的 `rewriteValue386_Op386ADCL_0` 函数就会使用 `ADCLconst` 替换 `ADCL` 和 `MOVLconst` 两条指令，它能通过对指令的压缩和优化减少在目标硬件上执行所需要的时间和资源。

我们在上一节中间代码生成中已经介绍过 `compileSSA` 中调用 `buildssa` 的执行过程，我们在这里继续介绍 `buildssa` 函数返回之后的逻辑：

```
func compileSSA(fn *Node, worker int) {
    f := buildssa(fn, worker)
    pp := newProgs(fn, worker)
    defer pp.Free()
    genssa(f, pp)
    pp.Flush()
}
```

[`genssa`](https://github.com/golang/go/blob/075c20cea8a1efda0e8d5d33a1995a220ad27b8c/src/cmd/compile/internal/gc/ssa.go#L5899-L6226) 函数会创建一个新的 `obj.Progs` 结构并将生成的 SSA 中间代码都存入新建的结构体中，我们在 2.4.4 SSA 生成中得到的 `ssa.html` 文件中就包含最后生成的中间代码：

![genssa](go语言设计与实现.assets/0410439fc43e096347f920952dca9575.png)

**图 2-18 genssa 的执行结果**

上述输出结果跟最后生成的汇编代码其实已经非常相似了，随后调用的 [`Flush`](https://github.com/golang/go/blob/075c20cea8a1efda0e8d5d33a1995a220ad27b8c/src/cmd/compile/internal/gc/gsubr.go#L92-L95) 函数就会使用 [`src/cmd/internal/obj`](https://github.com/golang/go/tree/master/src/cmd/internal/obj) 包中的汇编器将 SSA 转换成汇编代码：

```
func (pp *Progs) Flush() {
    plist := &obj.Plist{Firstpc: pp.Text, Curfn: pp.curfn}
    obj.Flushplist(Ctxt, plist, pp.NewProg, myimportpath)
}
```

`buildssa` 中的 `lower` 和随后的多个阶段会对 SSA 进行转换、检查和优化，生成机器特定的中间代码，接下来通过 `genssa` 将代码输出到 `Progs` 对象中，这也是代码进入汇编器前的最后一个步骤。



#### 汇编器

汇编器是将汇编语言翻译为机器语言的程序，Go 语言的汇编器是基于 Plan 9 汇编器的输入类型设计的，Go 语言对于汇编语言 Plan 9 和汇编器的资料十分缺乏，网上能够找到的资料也大多都含糊不清，官方对汇编器在不同处理器架构上的实现细节也没有明确定义：

> The details vary with architecture, and we apologize for the imprecision; the situation is **not well-defined**.[5](https://www.bookstack.cn/read/draveness-golang/100aabc3275d17e5.md#fn:5)

我们在研究汇编器和汇编语言时不应该陷入细节，只需要理解汇编语言的执行逻辑就能够帮助我们快速读懂汇编代码。当我们将如下的代码编译成汇编指令时，会得到如下的内容：

```
$ cat hello.go
package hello
func hello(a int) int {
    c := a + 2
    return c
}
$ GOOS=linux GOARCH=amd64 go tool compile -S main.go
"".hello STEXT nosplit size=15 args=0x10 locals=0x0
    0x0000 00000 (main.go:3)    TEXT    "".hello(SB), NOSPLIT, $0-16
    0x0000 00000 (main.go:3)    FUNCDATA    $0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
    0x0000 00000 (main.go:3)    FUNCDATA    $1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
    0x0000 00000 (main.go:3)    FUNCDATA    $3, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
    0x0000 00000 (main.go:4)    PCDATA    $2, $0
    0x0000 00000 (main.go:4)    PCDATA    $0, $0
    0x0000 00000 (main.go:4)    MOVQ    "".a+8(SP), AX
    0x0005 00005 (main.go:4)    ADDQ    $2, AX
    0x0009 00009 (main.go:5)    MOVQ    AX, "".~r1+16(SP)
    0x000e 00014 (main.go:5)    RET
    0x0000 48 8b 44 24 08 48 83 c0 02 48 89 44 24 10 c3     H.D$.H...H.D$..
...
```

上述汇编代码都是由 [`Flushplist`](https://github.com/golang/go/blob/b2482e481722357c6daa98ef074d8eaf8ac4baf3/src/cmd/internal/obj/plist.go#L22-L114) 这个函数生成的，该函数会调用架构特定的 `Preprocess` 和 `Assemble` 方法：

```
func Flushplist(ctxt *Link, plist *Plist, newprog ProgAlloc, myimportpath string) {
    ...
    for _, s := range text {
        mkfwd(s)
        linkpatch(ctxt, s, newprog)
        ctxt.Arch.Preprocess(ctxt, s, newprog)
        ctxt.Arch.Assemble(ctxt, s, newprog)
        linkpcln(ctxt, s)
        ctxt.populateDWARF(plist.Curfn, s, myimportpath)
    }
}
```

`Preprocess` 和 `Assemble` 这两个方法其实是在 Go 编译器最外层的主函数就确定了，我们在 2.1.4 中提到过的 `archInits` 函数会根据目标硬件对当前架构使用的配置进行初始化。

如果目标机器的架构是 x86，那么这两个函数最终会使用 [`preprocess`](https://github.com/golang/go/blob/b2482e481722357c6daa98ef074d8eaf8ac4baf3/src/cmd/internal/obj/x86/obj6.go#L565-L916) 和 [`span6`](https://github.com/golang/go/blob/498eaee461adefd5e578e62c134382ece94198da/src/cmd/internal/obj/x86/asm6.go#L1848-L2003)，作者在这里就不展开介绍这两个特别复杂的底层函数了，有兴趣的读者可以通过上述链接找到目标函数的位置了解预处理和汇编的处理过程，机器码的生成也都是由这两个函数组合完成的。

### 2.5.3 小结

机器码生成作为 Go 语言编译的最后一步，其实已经到了硬件和机器指令这一层，其中对于内存、寄存器的处理非常复杂并且难以阅读，想要真正掌握这里的处理的步骤和原理还是需要非常多的精力，但是作为软件工程师来说，如果不是 Go 语言编译器的开发者或者需要经常处理汇编语言和机器指令，掌握这些知识的投资回报率实在太低，没有太多的必要，我们在这里只需要对这个过程有所了解，补全知识上的盲点，这样在遇到问题时能够快速定位。

- Instruction set architecture https://en.wikipedia.org/wiki/Instruction_set_architecture[↩︎](https://www.bookstack.cn/read/draveness-golang/100aabc3275d17e5.md#fnref:1)
- 复杂指令集 Complex instruction set computer https://en.wikipedia.org/wiki/Complex_instruction_set_computer[↩︎](https://www.bookstack.cn/read/draveness-golang/100aabc3275d17e5.md#fnref:2)
- 精简指令集 Reduced instruction set computer https://en.wikipedia.org/wiki/Reduced_instruction_set_computer[↩︎](https://www.bookstack.cn/read/draveness-golang/100aabc3275d17e5.md#fnref:3)
- Introduction to the Go compiler https://github.com/golang/go/blob/master/src/cmd/compile/README.md[↩︎](https://www.bookstack.cn/read/draveness-golang/100aabc3275d17e5.md#fnref:4)
- A Quick Guide to Go's Assembler https://golang.org/doc/asm[↩︎](https://www.bookstack.cn/read/draveness-golang/100aabc3275d17e5.md#fnref:5)

## 2.6 推荐阅读

- [Introduction to the Go compiler](https://github.com/golang/go/tree/master/src/cmd/compile)
- [Go 1.5 Bootstrap Plan](https://docs.google.com/document/d/1OaatvGhEAq7VseQ9kkavxKNAfepWy2yhPUBs96FGV28/edit)
- [A Manual for the Plan 9 assembler](https://9p.io/sys/doc/asm.html)
- [Lexical Scanning in Go - Rob Pike](https://www.youtube.com/watch?v=HxaD_trXwRE)

## 2.7 总结

到这里，整个 Go 语言编译的过程也都介绍完了，从[词法与语法分析](https://www.bookstack.cn/read/draveness-golang/a81e1b5a6b7b28bc.md)、[类型检查](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md)、[中间代码生成](https://www.bookstack.cn/read/draveness-golang/5a89e04c79706261.md)到最后的[机器码生成](https://www.bookstack.cn/read/draveness-golang/100aabc3275d17e5.md)，包含的内容非常复杂，不过经过分析我们已经能够对 Go 语言编译器的原理有足够的了解，也对相关特性的实现更加清楚，后面的章节会介绍一些具体特性的原理，这些原理会依赖于编译期间的一些步骤，所以我们在深入理解 Go 语言的特性之前还是需要先了解一些编译期间完成的工作。



# 第三章 数据结构

- [3.1 数组](https://www.bookstack.cn/read/draveness-golang/79255565262cc9f6.md)
- [3.2 切片](https://www.bookstack.cn/read/draveness-golang/bc59e924b285e5e9.md)
- [3.3 哈希表](https://www.bookstack.cn/read/draveness-golang/8a6fa5746b8fbe7e.md)
- [3.4 字符串](https://www.bookstack.cn/read/draveness-golang/0bc3143426c7b7ea.md)
- [3.5 推荐阅读](https://www.bookstack.cn/read/draveness-golang/4f0f4af7cd96ea0e.md)



## 3.1 数组

数组和切片是 Go 语言中常见的数据结构，很多刚刚使用 Go 的开发者往往会混淆这两个概念，数组作为最常见的集合在编程语言中是非常重要的，除了数组之外，Go 语言引入了另一个概念 — 切片，切片与数组有一些类似，但是它们的不同之处导致使用上会产生巨大的差别。我们在这一节中会从 Go 语言的[编译期间](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md)运行时来介绍数组的底层实现原理，其中会包括数组的初始化、访问和赋值几种常见操作。



### 3.1.1 概述

数组是由相**同类型元素**的集合组成的数据结构，计算机会为数组分配一块**连续的内存**来保存其中的元素，我们可以利用数组中元素的索引快速访问元素对应的存储地址，常见的数组大多都是一维的线性数组，而多维数组在数值和图形计算领域却有比较常见的应用[1](https://www.bookstack.cn/read/draveness-golang/79255565262cc9f6.md#fn:1)。

![3D-array](go语言设计与实现.assets/7a95ec5e69ae3530797f91d20e4e05a8.jpeg)

**图 3-1 多维数组**

数组作为一种基本的数据类型，我们通常都会从两个维度描述数组，我们首先需要描述数组中存储的元素类型，还需要描述数组最大能够存储的元素个数，在 Go 语言中我们往往会使用如下所示的方式来表示数组类型：

```
[10]int
[200]interface{}
```

与很多语言不同，**Go 语言中数组在初始化之后大小就无法改变，存储元素类型相同、但是大小不同的数组类型在 Go 语言看来也是完全不同的，只有两个条件都相同才是同一个类型**。

```
func NewArray(elem *Type, bound int64) *Type {
    if bound < 0 {
        Fatalf("NewArray: invalid bound %v", bound)
    }
    t := New(TARRAY)
    t.Extra = &Array{Elem: elem, Bound: bound}
    t.SetNotInHeap(elem.NotInHeap())
    return t
}
```

编译期间的数组类型是由上述的 [`cmd/compile/internal/types.NewArray`](https://github.com/golang/go/blob/616c39f6a636166447bdaac4f0871a5ca52bae8c/src/cmd/compile/internal/types/type.go#L473-L481) 函数生成的，类型 `Array` 包含两个字段，一个是元素类型 `Elem`，另一个是数组的大小 `Bound（范围、界）`，这两个字段共同构成了数组类型，而当前数组是否应该在堆栈中初始化也在编译期就确定了。



### 3.1.2 初始化

Go 语言中的数组有两种不同的创建方式，一种是显式的指定数组的大小，另一种是使用 `[…]T` 声明数组，Go 语言会在编译期间通过源代码对数组的大小进行**推断**：

```
arr1 := [3]int{1, 2, 3}
arr2 := [...]int{1, 2, 3}
```

上述两种声明方式在运行期间得到的结果是完全相同的，**后一种声明方式在编译期间就会被『转换』成为前一种**，这也就是编译器对数组大小的推导，下面我们来介绍编译器的推导过程。

#### 上限推导

两种不同的声明方式会导致编译器做出完全不同的处理，如果我们使用第一种方式 `[10]T`，那么变量的类型在编译进行到[类型检查](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md)阶段就会被提取出来，随后会使用 [`cmd/compile/internal/types.NewArray`](https://github.com/golang/go/blob/616c39f6a636166447bdaac4f0871a5ca52bae8c/src/cmd/compile/internal/types/type.go#L473-L481) 函数创建包含数组大小的 `Array` 类型。

当我们使用 `[…]T` 的方式声明数组时，虽然在这一步也会创建一个 `Array` 类型 `Array{Elem: elem, Bound: -1}`，但是其中的数组大小上限会是 `-1`，这里的 `-1` 只是一个占位符，编译器会在后面的 [`cmd/compile/internal/gc.typecheckcomplit`](https://github.com/golang/go/blob/b7d097a4cf6b8a9125e4770b54d33826fa803023/src/cmd/compile/internal/gc/typecheck.go#L2755-L2961) 函数中对该数组的大小进行推导：

```golang
func typecheckcomplit(n *Node) (res *Node) {
    ...
    switch t.Etype {
    case TARRAY, TSLICE:
        var length, i int64
        nl := n.List.Slice()
        for i2, l := range nl {
            i++
            if i > length {
                length = i
            }
        }
        if t.IsDDDArray() {
            t.SetNumElem(length)
        }
    }
}
```

这个删减后的 [`cmd/compile/internal/gc.typecheckcomplit`](https://github.com/golang/go/blob/b7d097a4cf6b8a9125e4770b54d33826fa803023/src/cmd/compile/internal/gc/typecheck.go#L2755-L2961) 函数通过**遍历元素的方式**来计算数组中元素的数量。上述代码中的 `DDDArray` 指的就是使用 `[…]T` 声明的数组，因为声明这种数组时需要使用三个点（**Dot**），所以在编译器中就被称作 `DDDArray`。

所以我们可以看出 `[…]T{1, 2, 3}` 和 `[3]T{1, 2, 3}` 在运行时是完全等价的，`[…]T` 这种初始化方式也只是 Go 语言为我们提供的一种**语法糖**，当我们不想计算数组中的元素个数时就可以通过这种方法较少一些工作。



#### 语句转换

对于一个由字面量组成的数组，根据数组元素数量的不同，编译器会在负责初始化字面量的 [`cmd/compile/internal/gc.anylit`](https://github.com/golang/go/blob/f07059d949057f414dd0f8303f93ca727d716c62/src/cmd/compile/internal/gc/sinit.go#L875-L967) 函数中做两种不同的优化：

- 当元素数量小于或者等于 4 个时，会直接将数组中的元素放置在**栈**上；
- 当元素数量大于 4 个时，会将数组中的元素放置到**静态区**并在运行时取出；

```
func anylit(n *Node, var_ *Node, init *Nodes) {
    t := n.Type
    switch n.Op {
    case OSTRUCTLIT, OARRAYLIT:
        if n.List.Len() > 4 {
            ...
        }
        fixedlit(inInitFunction, initKindLocalCode, n, var_, init)
    ...
    }
}
```

当数组的元素**小于或者等于四个**时，[`cmd/compile/internal/gc.fixedlit`](https://github.com/golang/go/blob/f07059d949057f414dd0f8303f93ca727d716c62/src/cmd/compile/internal/gc/sinit.go#L515-L583) 会负责在函数编译之前将 `[3]{1, 2, 3}` 转换成更加原始的语句：

```
func fixedlit(ctxt initContext, kind initKind, n *Node, var_ *Node, init *Nodes) {
    var splitnode func(*Node) (a *Node, value *Node)
    ...
    for _, r := range n.List.Slice() {
        a, value := splitnode(r)
        a = nod(OAS, a, value)
        a = typecheck(a, ctxStmt)
        switch kind {
        case initKindStatic:
            genAsStatic(a)
        case initKindLocalCode:
            a = orderStmtInPlace(a, map[string][]*Node{})
            a = walkstmt(a)
            init.Append(a)
        }
    }
}
```

当数组中元素的个数小于四个时，[`cmd/compile/internal/gc.fixedlit`](https://github.com/golang/go/blob/f07059d949057f414dd0f8303f93ca727d716c62/src/cmd/compile/internal/gc/sinit.go#L515-L583) 函数接受的 `kind` 是 `initKindLocalCode`，上述代码会将原有的初始化语句 `[3]int{1, 2, 3}` 拆分成一个声明变量的表达式和几个赋值表达式，这些表达式会完成对数组的初始化：

```
var arr [3]int
arr[0] = 1
arr[1] = 2
arr[2] = 3
```

但是如果当前数组的元素大于 4 个，`anylit` 方法会先获取一个唯一的 `staticname`，然后调用 [`cmd/compile/internal/gc.fixedlit`](https://github.com/golang/go/blob/f07059d949057f414dd0f8303f93ca727d716c62/src/cmd/compile/internal/gc/sinit.go#L515-L583) 函数在静态存储区初始化数组中的元素并将临时变量赋值给当前的数组：

```
func anylit(n *Node, var_ *Node, init *Nodes) {
    t := n.Type
    switch n.Op {
    case OSTRUCTLIT, OARRAYLIT:
        if n.List.Len() > 4 {
            vstat := staticname(t)
            vstat.Name.SetReadonly(true)
            fixedlit(inNonInitFunction, initKindStatic, n, vstat, init)
            a := nod(OAS, var_, vstat)
            a = typecheck(a, ctxStmt)
            a = walkexpr(a, init)
            init.Append(a)
            break
        }
        ...
    }
}
```

假设我们在代码中初始化 `[]int{1, 2, 3, 4, 5}` 数组，那么我们可以将上述过程理解成以下的伪代码：

```
var arr [5]int
statictmp_0[0] = 1
statictmp_0[1] = 2
statictmp_0[2] = 3
statictmp_0[3] = 4
statictmp_0[4] = 5
arr = statictmp_0
```

总结起来，如果数组中元素的个数小于或者等于 4 个，那么所有的变量会直接在栈上初始化，如果数组元素大于 4 个，变量就会在静态存储区初始化然后拷贝到栈上，这些转换后的代码才会继续进入[中间代码生成](https://www.bookstack.cn/read/draveness-golang/5a89e04c79706261.md)和[机器码生成](https://www.bookstack.cn/read/draveness-golang/100aabc3275d17e5.md)两个阶段，最后生成可以执行的二进制文件。



### 3.1.3 访问和赋值

无论是在栈上还是静态存储区，**数组在内存中其实就是一连串的内存空间**，表示数组的方法就是**一个指向数组开头的指针、数组中元素的数量以及数组中元素类型占的空间大小**，如果我们不知道数组中元素的数量，访问时就可能发生越界，而如果不知道数组中元素类型的大小，就没有办法知道应该一次取出多少字节的数据，如果没有这些信息，我们就无法知道这片连续的内存空间到底存储了什么数据：

![golang-array-memory](go语言设计与实现.assets/f198479bc3f617455d5e24fcb41363d4.png)

**图 3-2 数组的内存空间**

**数组访问越界是非常严重的错误**，Go 语言中对越界的判断是可以在编译期间由静态类型检查完成的，

[`cmd/compile/internal/gc.typecheck1`](https://github.com/golang/go/blob/b7d097a4cf6b8a9125e4770b54d33826fa803023/src/cmd/compile/internal/gc/typecheck.go#L327-L2081) 函数会对访问数组的索引进行验证：

```
func typecheck1(n *Node, top int) (res *Node) {
    switch n.Op {
    case OINDEX:
        ok |= ctxExpr
        l := n.Left  // array
        r := n.Right // index
        switch n.Left.Type.Etype {
        case TSTRING, TARRAY, TSLICE:
            ...
            if n.Right.Type != nil && !n.Right.Type.IsInteger() {
                yyerror("non-integer array index %v", n.Right)
                break
            }
            if !n.Bounded() && Isconst(n.Right, CTINT) {
                x := n.Right.Int64()
                if x < 0 {
                    yyerror("invalid array index %v (index must be non-negative)", n.Right)
                } else if n.Left.Type.IsArray() && x >= n.Left.Type.NumElem() {
                    yyerror("invalid array index %v (out of bounds for %d-element array)", n.Right, n.Left.Type.NumElem())
                }
            }
        }
    ...
    }
}
```

- 访问数组的索引是非整数时会直接报错 —— `non-integer array index %v`；

- 访问数组的索引是负数时会直接报错 —— `"invalid array index %v (index must be non-negative)"`；

- 访问数组的索引越界时会直接报错 —— `"invalid array index %v (out of bounds for %d-element array)"`；

  

数组和字符串的一些简单越界错误都会在编译期间发现，比如我们直接使用整数或者常量访问数组，但是如果使用变量index去访问数组或者字符串时，编译器就无法发现对应的错误了，这时就需要 Go 语言运行时发挥作用了(panic: runtime error)：

```
arr[4]: invalid array index 4 (out of bounds for 3-element array)
arr[i]: panic: runtime error: index out of range [4] with length 3
```

Go 语言运行时在发现数组、切片和字符串的越界操作会由运行时的 `panicIndex` 和 [`runtime.goPanicIndex`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L86-L89) 函数触发程序的运行时错误并导致崩溃退出：

```
TEXT runtime·panicIndex(SB),NOSPLIT,$0-8
    MOVL    AX, x+0(FP)
    MOVL    CX, y+4(FP)
    JMP    runtime·goPanicIndex(SB)
func goPanicIndex(x int, y int) {
    panicCheck1(getcallerpc(), "index out of range")
    panic(boundsError{x: int64(x), signed: true, y: y, code: boundsIndex})
}
```

当数组的访问操作 `OINDEX` 成功通过编译器的检查之后，会被转换成几个 SSA 指令，假设我们有如下所示的 Go 语言代码，通过如下的方式进行编译会得到 `ssa.html` 文件：

```
package check
func outOfRange() int {
    arr := [3]int{1, 2, 3}
    i := 4
    elem := arr[i]
    return elem
}
$ GOSSAFUNC=outOfRange go build array.go
dumped SSA to ./ssa.html
```



`start` 阶段生成的 SSA 代码就是优化之前的第一版中间代码，下面展示的部分就是 `elem := arr[i]` 对应的中间代码，在这段中间代码中我们发现 Go 语言为数组的访问操作生成了判断数组上限的指令 `IsInBounds` 以及当条件不满足时触发程序崩溃的 `PanicBounds` 指令：

```
b1:
    ...
    v22 (6) = LocalAddr <*[3]int> {arr} v2 v20
    v23 (6) = IsInBounds <bool> v21 v11
If v23 → b2 b3 (likely) (6)
b2: ← b1-
    v26 (6) = PtrIndex <*int> v22 v21
    v27 (6) = Copy <mem> v20
    v28 (6) = Load <int> v26 v27 (elem[int])
    ...
Ret v30 (+7)
b3: ← b1-
    v24 (6) = Copy <mem> v20
    v25 (6) = PanicBounds <mem> [0] v21 v11 v24
Exit v25 (6)
```

`PanicBounds` 指令最终会被转换成上面提到的 `panicIndex` 函数，当数组下标没有越界时，编译器会先获取数组的内存地址和访问的下标，然后利用 `PtrIndex` 计算出目标元素的地址，再使用 `Load` 操作将指针中的元素加载到内存中。

当然只有当编译器无法对数组下标是否越界无法做出判断时才会加入 `PanicBounds` 指令交给运行时进行判断，在使用字面量整数访问数组下标时就会生成非常简单的中间代码，当我们将上述代码中的 `arr[i]` 改成 `arr[2]` 时，就会得到如下所示的代码：

```
b1:
    ...
    v21 (5) = LocalAddr <*[3]int> {arr} v2 v20
    v22 (5) = PtrIndex <*int> v21 v14
    v23 (5) = Load <int> v22 v20 (elem[int])
    ...
```

Go 语言对于数组的访问还是有着比较多的检查的，它不仅会在编译期间提前发现一些简单的越界错误并插入用于检测数组上限的函数调用，而在运行期间这些插入的函数会负责保证不会发生越界错误。

数组的赋值和更新操作 `a[i] = 2` 也会生成 SSA 生成期间计算出数组当前元素的内存地址，然后修改当前内存地址的内容，这些赋值语句会被转换成如下所示的 SSA 操作：

```
b1:
    ...
    v21 (5) = LocalAddr <*[3]int> {arr} v2 v19
    v22 (5) = PtrIndex <*int> v21 v13
    v23 (5) = Store <mem> {int} v22 v20 v19
    ...
```

赋值的过程中会先确定目标数组的地址，再通过 `PtrIndex` 获取目标元素的地址，最后使用 `Store` 指令将数据存入地址中，从上面的这些 SSA 代码中我们可以看出无论是数组的寻址还是赋值都是在编译阶段完成的，没有运行时的参与。

### 3.1.4 小结

数组是 Go 语言中重要的数据结构，了解它的实现能够帮助我们更好地理解这门语言，通过对其实现的分析，我们知道了对数组的访问和赋值需要同时依赖编译器和运行时，它的大多数操作在[编译期间](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md)都会转换成对内存的直接读写，在中间代码生成期间，编译器还会插入运行时方法 `panicIndex` 调用防止发生越界错误。

- Array data structure, https://en.wikipedia.org/wiki/Array_data_structure[↩︎](https://www.bookstack.cn/read/draveness-golang/79255565262cc9f6.md#fnref:1)



## 3.2 切片

我们在上一节介绍的数组在 Go 语言中没那么常用，更常用的数据结构其实是切片，**切片就是动态数组**，它的长度并不固定，我们可以随意向切片中追加元素，而切片会在容量不足时自动扩容。

在 Go 语言中，切片类型的声明方式与数组有一些相似，由于切片的长度是动态的，所以声明时只需要指定切片中的元素类型：

```
[]int
[]interface{}
```

从切片的定义我们能推测出，切片在编译期间的生成的类型只会包含切片中的元素类型，即 `int` 或者 `interface{}` 等。[`cmd/compile/internal/types.NewSlice`](https://github.com/golang/go/blob/616c39f6a636166447bdaac4f0871a5ca52bae8c/src/cmd/compile/internal/types/type.go#L484-L496) 就是编译期间用于创建 `Slice` 类型的函数：

```
func NewSlice(elem *Type) *Type {
    if t := elem.Cache.slice; t != nil {
        if t.Elem() != elem {
            Fatalf("elem mismatch")
        }
        return t
    }
    t := New(TSLICE)
    t.Extra = Slice{Elem: elem}
    elem.Cache.slice = t
    return t
}
```

上述方法返回的结构体 `TSLICE` 中的 `Extra` 字段是一个只包含切片内元素类型的 `Slice{Elem: elem}` 结构，也就是说**切片内元素的类型是在编译期间确定的**，编译器确定了类型之后，会将类型存储在 `Extra` 字段中帮助程序在运行时动态获取。



### 3.2.1 数据结构

编译期间的切片是 `Slice` 类型的，但是在运行时切片由如下的 `SliceHeader` 结构体表示，其中 **`Data` 字段是指向数组的指针**，`Len` 表示当前切片的长度，而 `Cap` 表示当前切片的容量，也就是 `Data` 数组的大小：

```
type SliceHeader struct {
    Data uintptr
    Len  int
    Cap  int
}
```

`Data` 作为一个指针指向的数组是一片连续的内存空间，这片内存空间可以用于存储切片中保存的全部元素，数组中的元素只是逻辑上的概念，底层存储其实都是连续的，所以我们可以将切片理解成一片连续的内存空间加上长度与容量的标识。

![golang-slice-struct](go语言设计与实现.assets/027b700b31af167d4f4f8c5ae7737941.png)

**图 3-3 Go 语言切片结构体**

从上图我们会发现切片与数组的关系非常密切，切片引入了一个抽象层，提供了对数组中部分片段的引用，作为数组的引用，我们可以在运行区间可以修改它的长度，如果底层的数组长度不足就会触发扩容机制，切片中的数组就会发生变化，不过在上层看来切片时没有变化的，上层只需要与切片打交道不需要关心底层的数组变化。

我们在上一节介绍过，获取数组大小、对数组中的元素的读写在编译期间就已经进行了简化，由于数组的内存固定且连续，很多操作都会变成对内存的直接读写。但是切片是运行时才会确定内容的结构，所有的操作还需要依赖 Go 语言的运行时来完成，我们接下来就会介绍切片一些常见操作的实现原理。



### 3.2.2 初始化

Go 语言中的切片有三种初始化的方式：

- 通过下标的方式获得数组或者切片的一部分；
- 使用字面量初始化新的切片；
- 使用关键字 `make` 创建切片：

```
arr[0:3] or slice[0:3]
slice := []int{1, 2, 3}
slice := make([]int, 10)
```



#### 使用下标

使用下标创建切片是最原始也最接近汇编语言的方式，它是所有方法中最为底层的一种，`arr[0:3]` 或者 `slice[0:3]` 这些操作会由编译器转换成 `OpSliceMake` 操作，我们可以通过下面的代码来验证一下：

```go
// ch03/op_slice_make.go
package opslicemake
func newSlice() []int {
    arr := [3]int{1, 2, 3}
    slice := arr[0:1]
    return slice
}
```

通过 `GOSSAFUNC` 变量编译上述代码可以得到如下所示的 SSA 中间代码，在[中间代码生成](https://www.bookstack.cn/read/draveness-golang/5a89e04c79706261.md)的 `decompose builtin` 阶段，`slice := arr[0:1]` 对应的部分：

```
v27 (+5) = SliceMake <[]int> v11 v14 v17
name &arr[*[3]int]: v11
name slice.ptr[*int]: v11
name slice.len[int]: v14
name slice.cap[int]: v17
```

`SliceMake` 这个操作会接受三个参数创建新的切片，元素类型、数组指针、切片大小和容量，这也就是我们在数据结构一节中提到的切片的几个字段。



#### 字面量

当我们使用字面量 `[]int{1, 2, 3}` 创建新的切片时，[`cmd/compile/internal/gc.slicelit`](https://github.com/golang/go/blob/f07059d949057f414dd0f8303f93ca727d716c62/src/cmd/compile/internal/gc/sinit.go#L595-L766) 函数会在编译期间将它展开成如下所示的代码片段：

```
var vstat [3]int
vstat[0] = 1
vstat[1] = 2
vstat[2] = 3
var vauto *[3]int = new([3]int)
*vauto = vstat
slice := vauto[:]
```

- 根据切片中的元素数量对底层数组的大小进行推断并创建一个数组；
- 将这些字面量元素存储到初始化的数组中；
- 创建一个同样指向 `[3]int` 类型的数组指针；
- 将静态存储区的数组 `vstat` 赋值给 `vauto` 指针所在的地址；
- 通过 `[:]` 操作获取一个底层使用 `vauto` 的切片；第 5 步中的 `[:]` 就是使用下标创建切片的方法，从这一点我们也能看出 `[:]` 操作是创建切片最底层的一种方法。



#### 关键字

如果使用字面量的方式创建切片，大部分的工作就都会在编译期间完成，但是当我们使用 `make` 关键字创建切片时，很多工作都需要运行时的参与；调用方必须在 `make` 函数中传入一个切片的大小以及可选的容量，[`cmd/compile/internal/gc.typecheck1`](https://github.com/golang/go/blob/b7d097a4cf6b8a9125e4770b54d33826fa803023/src/cmd/compile/internal/gc/typecheck.go#L327-L2126) 会对参数进行校验：

```
func typecheck1(n *Node, top int) (res *Node) {
    switch n.Op {
    ...
    case OMAKE:
        args := n.List.Slice()
        i := 1
        switch t.Etype {
        case TSLICE:
            if i >= len(args) {
                yyerror("missing len argument to make(%v)", t)
                return n
            }
            l = args[i]
            i++
            var r *Node
            if i < len(args) {
                r = args[i]
            }
            ...
            if Isconst(l, CTINT) && r != nil && Isconst(r, CTINT) && l.Val().U.(*Mpint).Cmp(r.Val().U.(*Mpint)) > 0 {
                yyerror("len larger than cap in make(%v)", t)
                return n
            }
            n.Left = l
            n.Right = r
            n.Op = OMAKESLICE
        }
    ...
    }
}
```



上述函数不仅会检查 `len` 是否传入，还会保证传入的容量 `cap` 一定大于或者等于 `len`，除了校验参数之外，当前函数会将 `OMAKE` 节点转换成 `OMAKESLICE`，随后的中间代码生成阶段在 [`cmd/compile/internal/gc.walkexpr`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/walk.go#L439-L1532) 函数中的 [`OMAKESLICE`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/walk.go#L1315) 分支依据两个重要条件对这里的 `OMAKESLICE` 进行转换：

- 切片的大小和容量是否足够小；
- 切片是否发生了逃逸，最终在堆上初始化当切片发生逃逸或者非常大时，我们需要 [`runtime.makeslice`](https://github.com/golang/go/blob/440f7d64048cd94cba669e16fe92137ce6b84073/src/runtime/slice.go#L34-L50) 函数在堆上初始化，如果当前的切片不会发生逃逸并且切片非常小的时候，`make([]int, 3, 4)` 会被直接转换成如下所示的代码：

```
var arr [4]int
n := arr[:3]
```

上述代码会初始化数组并且直接通过下标 `[:3]` 来得到数组的切片，这两部分操作都会在编译阶段完成，编译器会在栈上或者静态存储区创建数组，`[:3]` 会被转换成上一节提到的 `OpSliceMake` 操作。

分析了主要由编译器处理的分支之后，我们回到用于创建切片的运行时函数 [`runtime.makeslice`](https://github.com/golang/go/blob/440f7d64048cd94cba669e16fe92137ce6b84073/src/runtime/slice.go#L34-L50)，这个函数的实现非常简单：

```
func makeslice(et *_type, len, cap int) unsafe.Pointer {
    mem, overflow := math.MulUintptr(et.size, uintptr(cap))
    if overflow || mem > maxAlloc || len < 0 || len > cap {
        mem, overflow := math.MulUintptr(et.size, uintptr(len))
        if overflow || mem > maxAlloc || len < 0 {
            panicmakeslicelen()
        }
        panicmakeslicecap()
    }
    return mallocgc(mem, et, true)
}
```

它的主要工作就是计算当前切片占用的内存空间并在堆上申请一片连续的内存，它使用如下的方式计算占用的内存：

```
内存空间 = 切片中元素大小 x 切片容量
```

虽然大多的错误都可以在编译期间被检查出来，但是在创建切片的过程中如果发生了以下错误就会直接导致程序触发运行时错误并崩溃：

- 内存空间的大小发生了溢出；
- 申请的内存大于最大可分配的内存；
- 传入的长度小于 0 或者长度大于容量；`mallocgc` 就是用于申请内存的函数，这个函数的实现还是比较复杂，如果遇到了比较小的对象会直接初始化在 Go 语言调度器里面的 P 结构中，而大于 32KB 的一些对象会在堆上初始化，我们会在后面的章节中详细介绍 Go 语言的内存分配器，在这里就不展开分析了。

目前的 [`runtime.makeslice`](https://github.com/golang/go/blob/440f7d64048cd94cba669e16fe92137ce6b84073/src/runtime/slice.go#L34-L50) 会返回指向底层数组的指针，之前版本的 Go 语言中，数组指针、长度和容量会被合成一个 `slice` 结构并返回，但是从 [cmd/compile: move slice construction to callers of makeslice](https://github.com/golang/go/commit/020a18c545bf49ffc087ca93cd238195d8dcc411#diff-d9238ca551e72b3a80da9e0da10586a4) 这次提交[1](https://www.bookstack.cn/read/draveness-golang/bc59e924b285e5e9.md#fn:1)之后，构建结构体 `SliceHeader` 的工作就都交给 [`runtime.makeslice`](https://github.com/golang/go/blob/440f7d64048cd94cba669e16fe92137ce6b84073/src/runtime/slice.go#L34-L50) 的调用方处理了，这些调用方会在编译期间构建切片结构体：

```
func typecheck1(n *Node, top int) (res *Node) {
    switch n.Op {
    ...
    case OSLICEHEADER:
    switch 
        t := n.Type
        n.Left = typecheck(n.Left, ctxExpr)
        l := typecheck(n.List.First(), ctxExpr)
        c := typecheck(n.List.Second(), ctxExpr)
        l = defaultlit(l, types.Types[TINT])
        c = defaultlit(c, types.Types[TINT])
        n.List.SetFirst(l)
        n.List.SetSecond(c)
    ...
    }
}
```

`OSLICEHEADER` 操作会创建我们在上面介绍过的结构体 `SliceHeader`，其中包含数组指针、切片长度和容量，它也是切片在运行时的表示：

```
type SliceHeader struct {
    Data uintptr
    Len  int
    Cap  int
}
```

正是因为大多数对切片类型的操作并不需要直接操作原 `slice` 结构体，所以 `SliceHeader` 的引入能够减少切片初始化时的少量开销，这个改动能够减少 ~0.2% 的 Go 语言包大小并且能够减少 92 个 `panicindex` 的调用，占整个 Go 语言二进制的 ~3.5%。



### 3.2.3 访问元素

对切片常见的操作就是获取它的长度或者容量，这两个不同的函数 `len` 和 `cap` 被 Go 语言的编译器看成是两种特殊的操作，即 `OLEN` 和 `OCAP`，它们会在 [SSA 生成阶段](https://www.bookstack.cn/read/draveness-golang/5a89e04c79706261.md)被 [`cmd/compile/internal/gc.epxr`](https://github.com/golang/go/blob/a037582efff56082631508b15b287494df6e9b69/src/cmd/compile/internal/gc/ssa.go#L1975-L2724) 函数转换成 `OpSliceLen` 和 `OpSliceCap` 操作：

```
func (s *state) expr(n *Node) *ssa.Value {
    switch n.Op {
    case OLEN, OCAP:
        switch {
        case n.Left.Type.IsSlice():
            op := ssa.OpSliceLen
            if n.Op == OCAP {
                op = ssa.OpSliceCap
            }
            return s.newValue1(op, types.Types[TINT], s.expr(n.Left))
        ...
        }
    ...
    }
}
```

访问切片中的字段可能会触发 `decompose builtin` 阶段的优化，`len(slice)` 或者 `cap(slice)` 在一些情况下会被直接替换成切片的长度或者容量，不需要运行时从切片结构中获取：

```
(SlicePtr (SliceMake ptr _ _ )) -> ptr
(SliceLen (SliceMake _ len _)) -> len
(SliceCap (SliceMake _ _ cap)) -> cap
```

除了获取切片的长度和容量之外，访问切片中元素使用的 `OINDEX` 操作也会在中间代码生成期间转换成对地址的直接访问：

```
func (s *state) expr(n *Node) *ssa.Value {
    switch n.Op {
    case OINDEX:
        switch {
        case n.Left.Type.IsSlice():
            p := s.addr(n, false)
            return s.load(n.Left.Type.Elem(), p)
        ...
        }
    ...
    }
}
```

切片的操作基本都是在编译期间完成的，除了访问切片的长度、容量或者其中的元素之外，使用 `range` 遍历切片时也会在编译期间转换成形式更简单的代码，我们会在后面的 `range` 关键字一节中介绍使用 `range` 遍历切片的过程。





### 3.2.4 追加和扩容

向切片中追加元素应该是最常见的切片操作，在 Go 语言中我们会使用 `append` 关键字向切片追加元素，中间代码生成阶段的 [`cmd/compile/internal/gc.state.append`](https://github.com/golang/go/blob/a037582efff56082631508b15b287494df6e9b69/src/cmd/compile/internal/gc/ssa.go#L2732-L2884) 方法会拆分 `append` 关键字，该方法追加元素会根据返回值是否会覆盖原变量，分别进入两种流程，如果 `append` 返回的『新切片』不需要赋值回原有的变量，就会进入如下的处理流程：

```
// append(slice, 1, 2, 3)
ptr, len, cap := slice
newlen := len + 3
if newlen > cap {
    ptr, len, cap = growslice(slice, newlen)
    newlen = len + 3
}
*(ptr+len) = 1
*(ptr+len+1) = 2
*(ptr+len+2) = 3
return makeslice(ptr, newlen, cap)
```



我们会先对切片结构体进行解构获取它的数组指针、大小和容量，如果在追加元素后切片的大小大于容量，那么就会调用 [`runtime.growslice`](https://github.com/golang/go/blob/440f7d64048cd94cba669e16fe92137ce6b84073/src/runtime/slice.go#L76-L191) 对切片进行扩容并将新的元素依次加入切片；如果 `append` 后的切片会覆盖原切片，即 `slice = append(slice, 1, 2, 3)`， [`cmd/compile/internal/gc.state.append`](https://github.com/golang/go/blob/a037582efff56082631508b15b287494df6e9b69/src/cmd/compile/internal/gc/ssa.go#L2732-L2884) 就会使用另一种方式改写关键字：

```
// slice = append(slice, 1, 2, 3)
a := &slice
ptr, len, cap := slice
newlen := len + 3
if uint(newlen) > uint(cap) {
   newptr, len, newcap = growslice(slice, newlen)
   vardef(a)
   *a.cap = newcap
   *a.ptr = newptr
}
newlen = len + 3
*a.len = newlen
*(ptr+len) = 1
*(ptr+len+1) = 2
*(ptr+len+2) = 3
```

是否覆盖原变量的逻辑其实差不多，最大的区别在于最后的结果是不是赋值回原有的变量，如果我们选择覆盖原有的变量，也不需要担心切片的拷贝，因为 Go 语言的编译器已经对这种情况作了优化。

![golang-slice-append](go语言设计与实现.assets/e40a9d7ff091457f8f0dc7260065f52c.png)

**图 3-4 向 Go 语言的切片追加元素**



到这里我们已经通过 `append` 关键字被转换的控制流了解了在切片容量足够时如何向切片中追加元素，但是当切片的容量不足时就会调用 [`runtime.growslice`](https://github.com/golang/go/blob/440f7d64048cd94cba669e16fe92137ce6b84073/src/runtime/slice.go#L76-L191) 函数为切片扩容，**扩容就是为切片分配一块新的内存空间并将原切片的元素全部拷贝过去**，我们分几部分分析该方法：

```
func growslice(et *_type, old slice, cap int) slice {
    newcap := old.cap
    doublecap := newcap + newcap
    if cap > doublecap {
        newcap = cap
    } else {
        if old.len < 1024 {
            newcap = doublecap
        } else {
            for 0 < newcap && newcap < cap {
                newcap += newcap / 4
            }
            if newcap <= 0 {
                newcap = cap
            }
        }
    }
```

在分配内存空间之前需要先确定新的切片容量，Go 语言根据切片的当前容量选择不同的策略进行扩容：

- 如果期望容量大于当前容量的两倍就会使用期望容量；
- 如果当前切片容量小于 1024 就会将容量翻倍；
- 如果当前切片容量大于 1024 就会每次增加 25% 的容量，直到新容量大于期望容量；确定了切片的容量之后，就可以计算切片中新数组占用的内存了，计算的方法就是将目标容量和元素大小相乘，计算新容量时可能会发生溢出或者请求的内存超过上限，在这时就会直接 `panic`，不过相关的代码在这里就被省略了：

```
    var overflow bool
    var newlenmem, capmem uintptr
    switch {
    ...
    default:
        lenmem = uintptr(old.len) * et.size
        newlenmem = uintptr(cap) * et.size
        capmem, _ = math.MulUintptr(et.size, uintptr(newcap))
        capmem = roundupsize(capmem)
        newcap = int(capmem / et.size)
    }
    ...
    var p unsafe.Pointer
    if et.kind&kindNoPointers != 0 {
        p = mallocgc(capmem, nil, false)
        memclrNoHeapPointers(add(p, newlenmem), capmem-newlenmem)
    } else {
        p = mallocgc(capmem, et, true)
        if writeBarrier.enabled {
            bulkBarrierPreWriteSrcOnly(uintptr(p), uintptr(old.array), lenmem)
        }
    }
    memmove(p, old.array, lenmem)
    return slice{p, old.len, newcap}
}
```

如果切片中元素不是指针类型，那么就会调用 `memclrNoHeapPointers` 将超出切片当前长度的位置清空并在最后使用 `memmove` 将原数组内存中的内容拷贝到新申请的内存中。这里的 `memclrNoHeapPointers` 和 `memmove` 都是用目标机器上的汇编指令实现的，在这里就不展开介绍了。

[`runtime.growslice`](https://github.com/golang/go/blob/440f7d64048cd94cba669e16fe92137ce6b84073/src/runtime/slice.go#L76-L191) 函数最终会返回一个新的 `slice` 结构，其中包含了新的数组指针、大小和容量，这个返回的三元组最终会改变原有的切片，帮助 `append` 完成元素追加的功能。



### 3.2.5 拷贝切片

切片的拷贝虽然不是一个常见的操作类型，但是却是我们学习切片实现原理必须要谈及的一个问题，当我们使用 `copy(a, b)` 的形式对切片进行拷贝时，编译期间的 [`cmd/compile/internal/gc.copyany`](https://github.com/golang/go/blob/bf4990522263503a1219372cd8f1ee9422b51324/src/cmd/compile/internal/gc/walk.go#L2980-L3040) 函数也会分两种情况进行处理，如果当前 `copy` 不是在运行时调用的，`copy(a, b)` 会被直接转换成下面的代码：

```
n := len(a)
if n > len(b) {
    n = len(b)
}
if a.ptr != b.ptr {
    memmove(a.ptr, b.ptr, n*sizeof(elem(a))) 
}
```

其中 `memmove` 会负责**对内存进行拷贝**，在其他情况下，编译器会使用 [`runtime.slicecopy`](https://github.com/golang/go/blob/440f7d64048cd94cba669e16fe92137ce6b84073/src/runtime/slice.go#L197-L230) 函数替换运行期间调用的 `copy`，例如：`go copy(a, b)`：

```go
func slicecopy(to, fm slice, width uintptr) int {
    if fm.len == 0 || to.len == 0 {
        return 0
    }
    n := fm.len
    if to.len < n {
        n = to.len
    }
    if width == 0 {
        return n
    }
    ...
    size := uintptr(n) * width
    if size == 1 {
        *(*byte)(to.array) = *(*byte)(fm.array)
    } else {
        memmove(to.array, fm.array, size)
    }
    return n
}
```

上述函数的实现非常直接，两种不同的拷贝方式一般都会通过 `memmove` 将**整块内存**中的内容拷贝到目标的内存区域中：

![golang-slice-copy](go语言设计与实现.assets/c86840ae1b0631fb99c857eaab954d4a.png)

**图 3-5 Go 语言切片的拷贝**

相比于依次对元素进行拷贝，这种方式能够提供更好的性能，但是需要注意的是，哪怕使用 `memmove` 对内存成块进行拷贝，但是这个操作还是会占用非常多的资源，在大切片上执行拷贝操作时一定要注意性能影响。



### 3.2.6 小结

切片的很多功能都是在运行时实现的了，无论是初始化切片，还是对切片进行追加或扩容都需要运行时的支持，需要注意的是在遇到大切片扩容或者复制时可能会发生大规模的内存拷贝，一定要在使用时减少这种情况的发生避免对程序的性能造成影响。

------

- cmd/compile: move slice construction to callers of makeslice https://github.com/golang/go/commit/020a18c545bf49ffc087ca93cd238195d8dcc411#diff-d9238ca551e72b3a80da9e0da10586a4[↩︎](https://www.bookstack.cn/read/draveness-golang/bc59e924b285e5e9.md#fnref:1)



## 3.3 哈希表

这一节会介绍 Go 语言中的另一个集合元素 — 哈希，也就是 Map 的实现原理；哈希表是除了数组之外，最常见的数据结构，几乎所有的语言都会有数组和哈希表这两种集合元素，有的语言将数组实现成列表，有的语言将哈希表称作结构体或者字典，但是它们是两种设计集合元素的思路，数组用于表示元素的序列，而哈希表示的是键值对之间映射关系，只是不同语言的叫法和实现稍微有些不同。

[哈希表](https://en.wikipedia.org/wiki/Hash_table)[1](https://www.bookstack.cn/read/draveness-golang/8a6fa5746b8fbe7e.md#fn:1)是一种古老的数据结构，在 **1953 年**就有人使用拉链法实现了哈希表，它能够根据键（Key）直接访问内存中的存储位置，也就是说我们能够直接通过键找到该键对应的一个值。



### 3.3.1 设计原理

哈希表是计算机科学中的最重要数据结构之一，这不仅因为它 `O(1)` 的读写性能非常优秀，还因为它提供了键值之间的映射。想要实现一个性能优异的哈希表，需要注意两个关键点 —— **哈希函数和冲突解决方法**。

#### 哈希函数

实现哈希表的关键点在于如何选择哈希函数，哈希函数的选择在很大程度上能够决定哈希表的读写性能，在理想情况下，哈希函数应该能够将不同键能够地映射到不同的索引上，这要求**哈希函数输出范围大于输入范围**，但是由于键的数量会远远大于映射的范围，所以在实际使用时，这个理想的结果是不可能实现的。

![perfect-hash-function](go语言设计与实现.assets/7c2327586ef7ffee91a3e179bfbfa306.png)

**图 3-7 完美哈希函数**

比较实际的方式是让哈希函数的结果能够尽可能的均匀分布，然后通过工程上的手段解决哈希碰撞的问题，但是哈希的结果一定要尽可能均匀，结果不均匀的哈希函数会造成更多的冲突并导致更差的读写性能。

![bad-hash-function](go语言设计与实现.assets/d2a41ce657f8c70a0162f0f0ea8ebf6f.png)

**图 3-8 不均匀哈希函数**

在一个使用结果较为均匀的哈希函数中，哈希的增删改查都需要 `O(1)` 的时间复杂度，但是非常不均匀的哈希函数会导致所有的操作都会占用最差 `O(n)` 的复杂度，所以在哈希表中使用好的哈希函数是至关重要的。



#### 冲突解决

就像我们之前所提到的，在通常情况下，哈希函数输入的范围一定会远远大于输出的范围，所以在使用哈希表时一定会遇到冲突，哪怕我们使用了完美的哈希函数，当输入的键足够多最终也会造成冲突。

然而我们的哈希函数往往都是不完美的，输出的范围是有限的，所以一定会发生哈希碰撞，这时就需要一些方法来解决哈希碰撞的问题，常见方法的就是开放寻址法和拉链法。



##### 开放寻址法

[开放寻址法](https://en.wikipedia.org/wiki/Open_addressing)【[2](https://www.bookstack.cn/read/draveness-golang/8a6fa5746b8fbe7e.md#fn:2)】是一种在哈希表中解决哈希碰撞的方法，这种方法的核心思想是**对数组中的元素依次探测和比较以判断目标键值对是否存在于哈希表中**，如果我们使用开放寻址法来实现哈希表，那么在支撑哈希表的数据结构就是**数组**，不过因为数组的长度有限，存储 `(author, draven)` 这个键值对时会从如下的**索引开始遍历**：

```
index := hash("author") % array.len
```

当我们向当前哈希表写入新的数据时发生了冲突，就会将键值对写入到下一个不为空的位置：

![open-addressing-and-set](go语言设计与实现.assets/dd9a2e7e029329af406f14e6a58fa18d.png)

**图 3-9 开放地址法写入数据**



如上图所示，当 Key3 与已经存入哈希表中的两个键值对 Key1 和 Key2 发生冲突时，Key3 会被写入 Key2 后面的空闲内存中；当我们再去读取 Key3 对应的值时就会先对键进行哈希并取模，这会帮助我们找到 Key1，因为 Key1 与我们期望的键 Key3 不匹配，所以会继续查找后面的元素，直到内存为空或者找到目标元素。

![open-addressing-and-get](go语言设计与实现.assets/3e558aa412550d7bfa40b32f561fc0db.png)

**图 3-9 开放地址法读取数据**

当需要查找某个键对应的值时，就会从索引的位置开始对数组进行线性探测，找到目标键值对或者空内存就意味着这一次查询操作的结束。

开放寻址法中对性能影响最大的就是**装载因子**，它是数组中元素的数量与数组大小的比值，随着装载因子的增加，线性探测的平均用时就会逐渐增加，这会同时影响哈希表的读写性能，当装载率超过 70% 之后，哈希表的性能就会急剧下降，而一旦装载率达到 100%，整个哈希表就会完全失效，这时查找任意元素都需要遍历数组中全部的元素，所以在实现哈希表时一定要时刻关注装载因子的变化。



##### 拉链法

与开放地址法相比，拉链法是哈希表中最常见的实现方法，大多数的编程语言都用拉链法实现哈希表，它的实现比较开放地址法稍微复杂一些，但是**平均查找的长度也比较短，各个用于存储节点的内存都是动态申请的，可以节省比较多的存储空间**。

**实现拉链法一般会使用数组加上链表**，不过有一些语言会在拉链法的哈希中引入红黑树以优化性能，拉链法会使用链表数组作为哈希底层的数据结构，我们可以将它看成一个可以扩展的『二维数组』：

![separate-chaing-and-set](go语言设计与实现.assets/a47d5faf4c8efed3cd903d90b8f2fc72.png)

**图 3-10 拉链法写入数据**

如上图所示，当我们需要将一个键值对 `(Key6, Value6)` 写入哈希表时，键值对中的键 `Key6` 都会先经过一个哈希函数，哈希函数返回的哈希会帮助我们选择一个桶，和开放地址法一样，选择桶的方式就是直接对哈希返回的结果取模：

```
index := hash("Key6") % array.len
```

选择了 2 号桶之后就可以遍历当前桶中的链表了，在遍历链表的过程中会遇到以下两种情况：

- 找到键相同的键值对 —— 更新键对应的值；
- 没有找到键相同的键值对 —— 在链表的末尾追加新键值对；将键值对写入哈希之后，要通过某个键在其中获取映射的值，就会经历如下的过程：

![separate-chaing-and-get](go语言设计与实现.assets/796edf3a0694417f6fdf5557241422ba.png)

**图 3-11 拉链法读取数据**

Key11 展示了一个键在哈希表中不存在的例子，当哈希表发现它命中 4 号桶时，它会依次遍历桶中的链表，然而遍历到链表的末尾也没有找到期望的键，所以哈希表中没有该键对应的值。

在一个性能比较好的哈希表中，每一个桶中都应该有 0~1 个元素，有时会有 2~3 个，很少会超过这个数量，计算哈希、定位桶和遍历链表三个过程是哈希表读写操作的主要开销，使用拉链法实现的哈希也有装载因子这一概念：

```
装载因子 := 元素数量 / 桶数量
```

与开放地址法一样，拉链法的**装载因子**越大，哈希的读写性能就越差，在一般情况下使用拉链法的哈希表装载因子都不会超过 1，当哈希表的装载因子较大时就会触发哈希的扩容，创建更多的桶来存储哈希中的元素，保证性能不会出现严重的下降。如果有 1000 个桶的哈希表存储了 10000 个键值对，它的性能是保存 1000 个键值对的 1/10，但是仍然比在链表中直接读写好 1000 倍。



### 3.3.2 数据结构

Go 语言运行时同时使用了多个数据结构组合表示哈希表，其中使用 [`hmap`](https://github.com/golang/go/blob/ed15e82413c7b16e21a493f5a647f68b46e965ee/src/runtime/map.go#L115-L129) 结构体来表示哈希，我们先来看一下这个结构体内部的字段：

```
type hmap struct {
    count     int
    flags     uint8
    B         uint8
    noverflow uint16
    hash0     uint32
    buckets    unsafe.Pointer
    oldbuckets unsafe.Pointer
    nevacuate  uintptr
    extra *mapextra
}
```

- `count` 表示当前哈希表中的元素数量；
- `B` 表示当前哈希表持有的 `buckets` 数量，但是因为哈希表中桶的数量都 2 的倍数，所以该字段会存储对数，也就是 `len(buckets) == 2^B`；
- `hash0` 是哈希的种子，它能为哈希函数的结果引入随机性，这个值在创建哈希表时确定，并在调用哈希函数时作为参数传入；
- `oldbuckets` 是哈希在扩容时用于保存之前 `buckets` 的字段，它的大小是当前 `buckets` 的一半；

- ![hmap-and-buckets](go语言设计与实现.assets/dd9779ba3b8d6758b5ad5f04a01e5ba2.png)

**图 3-12 哈希表的数据结构**

如上图所示哈希表 `hmap` 的桶就是 `bmap`，每一个 `bmap` 都能存储 8 个键值对，当哈希表中存储的数据过多，单个桶无法装满时就会使用 `extra.overflow` 中桶存储溢出的数据。上述两种不同的桶在内存中是连续存储的，我们在这里将它们分别称为**正常桶和溢出桶**，上图中黄色的 `bmap` 就是正常桶，绿色的 `bmap` 是溢出桶，溢出桶是在 Go 语言还使用 C 语言实现时就使用的设计[3](https://www.bookstack.cn/read/draveness-golang/8a6fa5746b8fbe7e.md#fn:3)，由于它能够减少扩容的频率所以一直使用至今。

这个桶的结构体 `bmap` 在 Go 语言源代码中的定义只包含一个简单的 `tophash` 字段，`tophash` 存储了键的哈希的高 8 位，通过比较不同键的哈希的高 8 位可以减少访问键值对次数以提高性能：

```
type bmap struct {
    tophash [bucketCnt]uint8
}
```

`bmap` 结构体其实不止包含 `tophash` 字段，**由于哈希表中可能存储不同类型的键值对并且 Go 语言也不支持泛型，**所以键值对占据的内存空间大小只能在编译时进行推导，这些字段在运行时也都是通过计算内存地址的方式直接访问的，所以它的定义中就没有包含这些字段，但是我们能根据编译期间的 [`cmd/compile/internal/gc.bmap`](https://github.com/golang/go/blob/be64a19d99918c843f8555aad580221207ea35bc/src/cmd/compile/internal/gc/reflect.go#L82-L187) 函数对它的结构重建：

```
type bmap struct {
    topbits  [8]uint8
    keys     [8]keytype
    values   [8]valuetype
    pad      uintptr
    overflow uintptr
}
```

如果哈希表存储的数据逐渐增多，我们会对哈希表进行扩容或者使用额外的桶存储溢出的数据，不会让单个桶中的数据超过 8 个，不过溢出桶只是临时的解决方案，创建过多的溢出桶最终也会导致哈希的扩容。

从 Go 语言哈希的定义中就可以发现，它比前面两节提到的数组和切片复杂得多，结构体中不仅包含大量字段，还使用了较多的复杂结构，在后面的小节中我们会详细介绍不同字段的作用。



### 3.3.3 初始化

既然已经介绍了常见哈希表的基本原理和实现方法，那么可以开始分析 Go 语言中哈希表的实现，首先要分析的就是在 Go 语言中初始化哈希的两种方法 — 通过字面量和运行时。

#### 字面量

目前的现代编程语言基本都支持使用字面量的方式初始化哈希，一般都会使用 `key: value` 的语法来表示键值对，Go 语言中也不例外：

```
hash := map[string]int{
    "1": 2,
    "3": 4,
    "5": 6,
}
```

我们需要在初始化哈希时声明键值对的类型，这种使用字面量初始化的方式最终都会通过 [`cmd/compile/internal/gc.maplit`](https://github.com/golang/go/blob/f07059d949057f414dd0f8303f93ca727d716c62/src/cmd/compile/internal/gc/sinit.go#L768-L873) 函数初始化，我们来分析一下 [`cmd/compile/internal/gc.maplit`](https://github.com/golang/go/blob/f07059d949057f414dd0f8303f93ca727d716c62/src/cmd/compile/internal/gc/sinit.go#L768-L873) 函数初始化哈希的过程：

```
func maplit(n *Node, m *Node, init *Nodes) {
    a := nod(OMAKE, nil, nil)
    a.Esc = n.Esc
    a.List.Set2(typenod(n.Type), nodintconst(int64(n.List.Len())))
    litas(m, a, init)
    var stat, dyn []*Node
    for _, r := range n.List.Slice() {
        stat = append(stat, r)
    }
    if len(stat) > 25 {
        ...
    } else {
        addMapEntries(m, stat, init)
    }
}
```

当哈希表中的元素数量少于或者等于 25 个时，编译器会直接调用 `addMapEntries` 将字面量初始化的结构体转换成以下的代码，将所有的键值对一次加入到哈希表中：

```
hash := make(map[string]int, 3)
hash["1"] = 2
hash["3"] = 4
hash["5"] = 6
```

这种初始化的方式与前面两节分析的[数组](https://www.bookstack.cn/read/draveness-golang/79255565262cc9f6.md)和[切片](https://www.bookstack.cn/read/draveness-golang/bc59e924b285e5e9.md)的几乎完全相同，由此看来集合类型的初始化在 Go 语言中有着相同的处理方式和逻辑。

一旦哈希表中元素的数量超过了 25 个，就会在编译期间创建两个数组分别存储键和值的信息，这些键值对会通过一个如下所示的 for 循环加入目标的哈希：

```
hash := make(map[string]int, 26)
vstatk := []string{"1", "2", "3", ... ， "26"}
vstatv := []int{1, 2, 3, ... , 26}
for i := 0; i < len(vstak); i++ {
    hash[vstatk[i]] = vstatv[i]
}
```

这里展开的两个切片 `vstatk` 和 `vstatv` 还会被编辑器继续展开，具体的展开方式可以阅读上一节了解[切片的初始化](https://www.bookstack.cn/read/draveness-golang/bc59e924b285e5e9.md)，不过无论使用哪种方法，使用字面量初始化的过程都会使用 Go 语言中的关键字 `make` 来创建新的哈希并通过最原始的 `[]` 语法向哈希追加元素。



#### 运行时

无论 `make` 是从哪里来的，只要我们使用 `make` 创建哈希，Go 语言编译器都会在[类型检查](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md)期间将它们转换成对 [`runtime.makemap`](https://github.com/golang/go/blob/dcd3b2c173b77d93be1c391e3b5f932e0779fb1f/src/runtime/map.go#L303-L336) 的调用，使用字面量来初始化哈希也只是语言提供的辅助工具，最后调用的都是 [`runtime.makemap`](https://github.com/golang/go/blob/dcd3b2c173b77d93be1c391e3b5f932e0779fb1f/src/runtime/map.go#L303-L336)：

```
func makemap(t *maptype, hint int, h *hmap) *hmap {
    mem, overflow := math.MulUintptr(uintptr(hint), t.bucket.size)
    if overflow || mem > maxAlloc {
        hint = 0
    }
    if h == nil {
        h = new(hmap)
    }
    h.hash0 = fastrand()
    B := uint8(0)
    for overLoadFactor(hint, B) {
        B++
    }
    h.B = B
    if h.B != 0 {
        var nextOverflow *bmap
        h.buckets, nextOverflow = makeBucketArray(t, h.B, nil)
        if nextOverflow != nil {
            h.extra = new(mapextra)
            h.extra.nextOverflow = nextOverflow
        }
    }
    return h
}
```

这个函数的执行过程会分成以下几个部分：

- 计算哈希占用的内存是否溢出或者超出能分配的最大值；
- 调用 `fastrand` 获取一个随机的哈希种子；
- 根据传入的 `hint` 计算出需要的最小需要的桶的数量；
- 使用 [`runtime.makeBucketArray`](https://github.com/golang/go/blob/dcd3b2c173b77d93be1c391e3b5f932e0779fb1f/src/runtime/map.go#L344-L387) 创建用于保存桶的数组；[`runtime.makeBucketArray`](https://github.com/golang/go/blob/dcd3b2c173b77d93be1c391e3b5f932e0779fb1f/src/runtime/map.go#L344-L387) 函数会根据传入的 `B` 计算出的需要创建的桶数量在内存中分配一片连续的空间用于存储数据：

```
func makeBucketArray(t *maptype, b uint8, dirtyalloc unsafe.Pointer) (buckets unsafe.Pointer, nextOverflow *bmap) {
    base := bucketShift(b)
    nbuckets := base
    if b >= 4 {
        nbuckets += bucketShift(b - 4)
        sz := t.bucket.size * nbuckets
        up := roundupsize(sz)
        if up != sz {
            nbuckets = up / t.bucket.size
        }
    }
    buckets = newarray(t.bucket, int(nbuckets))
    if base != nbuckets {
        nextOverflow = (*bmap)(add(buckets, base*uintptr(t.bucketsize)))
        last := (*bmap)(add(buckets, (nbuckets-1)*uintptr(t.bucketsize)))
        last.setoverflow(t, (*bmap)(buckets))
    }
    return buckets, nextOverflow
}
```

当桶的数量小于 $2^4$ 时，由于数据较少、使用溢出桶的可能性较低，这时就会省略创建的过程以减少额外开销；当桶的数量多于 $2^4$ 时，就会额外创建 $2^{B-4}$ 个溢出桶，根据上述代码，我们能确定正常桶和溢出桶在内存中的存储空间是连续的，只是被 `hmap` 中的不同字段引用。

































# 第四章 语言基础

- [4.1 函数调用](https://www.bookstack.cn/read/draveness-golang/5de3a4330de09d9e.md)
- [4.2 接口](https://www.bookstack.cn/read/draveness-golang/5833e4cb4df6241b.md)
- [4.3 反射](https://www.bookstack.cn/read/draveness-golang/f42669fa0cd10c64.md)
- [4.4 推荐阅读](https://www.bookstack.cn/read/draveness-golang/82497e663992a6f7.md)



# 第五章 常用关键字

- [5.1 for 和 range](https://www.bookstack.cn/read/draveness-golang/816b8b0f4e15285f.md)
- [5.2 select](https://www.bookstack.cn/read/draveness-golang/1a4f7a284cd2b279.md)
- [5.3 defer](https://www.bookstack.cn/read/draveness-golang/f434f07b7b465a9f.md)
- [5.4 panic 和 recover](https://www.bookstack.cn/read/draveness-golang/ada6c076f55fa1aa.md)
- [5.5 make 和 new](https://www.bookstack.cn/read/draveness-golang/bb191ed6f01d91ba.md)
- [5.6 推荐阅读](https://www.bookstack.cn/read/draveness-golang/d9282a6420fd069c.md)





# 第六章 并发编程

- [6.1 上下文 Context](https://www.bookstack.cn/read/draveness-golang/6ee8389651c21ac5.md)
- [6.2 同步原语与锁](https://www.bookstack.cn/read/draveness-golang/b103f6e877e4c9bd.md)
- [6.3 定时器](https://www.bookstack.cn/read/draveness-golang/326bb5c45fbed045.md)
- [6.4 Channel](https://www.bookstack.cn/read/draveness-golang/c666731d5f1a2820.md)
- [6.5 Goroutine](https://www.bookstack.cn/read/draveness-golang/488ee05eea5e58a3.md)



# 第七章 内存管理

- [7.1 内存分配器](https://www.bookstack.cn/read/draveness-golang/3a89cfd15249f9fe.md)
- [7.2 垃圾收集器](https://www.bookstack.cn/read/draveness-golang/30cd28c181a56e61.md)
- [7.3 栈内存管理](https://www.bookstack.cn/read/draveness-golang/4a6101b89b1d41f1.md)



# 第八章 元编程

- [8.1 插件系统](https://www.bookstack.cn/read/draveness-golang/6588a87b3f79f21b.md)
- [8.2 代码生成](https://www.bookstack.cn/read/draveness-golang/19ab6c46e7c9de36.md)

# 第九章 标准库

- [9.1 JSON](https://www.bookstack.cn/read/draveness-golang/f6b06e1693601b3a.md)













































