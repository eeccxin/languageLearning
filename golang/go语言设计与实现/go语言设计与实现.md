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

## 4.1 函数调用

> [4.1 函数调用](https://www.bookstack.cn/read/draveness-golang/5de3a4330de09d9e.md)

函数是 Go 语言中的一等公民，理解和掌握函数的调用过程是我们深入学习 Go 无法跳过的，本节将从函数的调用惯例和参数的传递方法两个方面分别介绍函数的执行过程。

### 4.1.1 调用惯例

无论是系统级编程语言 C 和 Go，还是脚本语言 Ruby 和 Python，这些编程语言在调用函数时往往都使用相同的语法：

```
somefunction(arg0, arg1)
```

虽然它们调用函数的语法很相似，但是它们的调用惯例却可能大不相同。调用惯例是调用方和被调用方对于参数和返回值传递的约定，我们将在这里为各位读者分别介绍 C 和 Go 语言的调用惯例。

#### C 语言

我们先来研究 C 语言的调用惯例，使用 [GCC](https://gcc.gnu.org/)[1](https://www.bookstack.cn/read/draveness-golang/5de3a4330de09d9e.md#fn:1) 和 [Clang](https://clang.llvm.org/)[2](https://www.bookstack.cn/read/draveness-golang/5de3a4330de09d9e.md#fn:2) **编译器将 C 语言编译成汇编代码**是分析它调用惯例的最好方法，从汇编语言中可以一窥函数调用的具体过程。

GCC 和 Clang 编译相同 C 语言代码可能会生成不同的汇编指令，不过生成的代码在结构上不会有太大的区别，所以对只想理解调用惯例的人来说没有太多影响。作者在本节中选择使用 GCC 编译器来编译 C 语言：

```
$ gcc --version
gcc (Ubuntu 4.8.2-19ubuntu1) 4.8.2
Copyright (C) 2013 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

假设我们有以下的 C 语言代码，代码中只包含两个函数，其中一个是主函数 `main`，另一个是我们定义的函数 `my_function`：

```
// ch04/my_function.c
int my_function(int arg1, int arg2) {
    return arg1 + arg2;
}
int main() {
    int i = my_function(1, 2);
}
```

我们可以使用 `cc -S my_function.c` 命令将上述文件编译成如下所示的汇编代码：

```
main:
    pushq    %rbp
    movq    %rsp, %rbp
    subq    $16, %rsp
    movl    $2, %esi  // 设置第二个参数
    movl    $1, %edi  // 设置第一个参数
    call    my_function
    movl    %eax, -4(%rbp)
my_function:
    pushq    %rbp
    movq    %rsp, %rbp
    movl    %edi, -4(%rbp)    // 取出第一个参数，放到栈上
    movl    %esi, -8(%rbp)    // 取出第二个参数，放到栈上
    movl    -8(%rbp), %eax    // eax = edi = 1
    movl    -4(%rbp), %edx    // edx = esi = 2
    addl    %edx, %eax        // eax = eax + edx = 1 + 2 = 3
    popq    %rbp
```

我们按照 `my_function` 函数调用前、调用时以及调用后三个部分分析上述调用过程：

- 在 `my_function` 调用之前，调用方 `main` 函数将 `my_function` 的两个参数分别存到 edi 和 esi 寄存器中；
- 在 `my_function` 执行时，它会将寄存器 edi 和 esi 中的数据存储到 eax 和 edx 两个寄存器中，随后通过汇编指令 `addl` 计算两个入参之和；
- 在 `my_function` 调用之后，使用寄存器 eax 传递返回值，`main` 函数将 `my_function` 的返回值存储到栈上的 `i` 变量中；

当 `my_function` 函数的入参增加至八个，这时重新编译当前的程序可以会得到不同的汇编语言：

```
main:
    pushq    %rbp
    movq    %rsp, %rbp
    subq    $16, %rsp     // 为参数传递申请 16 字节的栈空间
    movl    $8, 8(%rsp)   // 传递第 8 个参数
    movl    $7, (%rsp)    // 传递第 7 个参数
    movl    $6, %r9d
    movl    $5, %r8d
    movl    $4, %ecx
    movl    $3, %edx
    movl    $2, %esi
    movl    $1, %edi
    call    my_function
```

`main` 函数调用 `my_function` 时，前六个参数是使用 edi、esi、edx、ecx、r8d 和 r9d 六个寄存器传递的。寄存器的使用顺序也是调用惯例的一部分，函数的第一个参数一定会使用 edi 寄存器，第二个参数使用 esi 寄存器，以此类推。

最后的两个参数与前面的完全不同，调用方 `main` 函数通过传递这两个参数，图 4-1 展示了 `main` 函数在调用 `my_function` 前的栈信息：

![c-function-call-stack](go语言设计与实现.assets/07d4407ea7e46241245adf7d54421f52.png)

**图 4-1 C 语言 main 函数的调用栈**

```
假设函数 func(7, 8) 被调用，栈帧可能如下（地址从高到低）：

高地址
+-------------------+
| ...               | ← 调用者的栈帧
| prev rbp          | ← 保存的调用者 rbp
| prev rsp          | ← 调用前的 rsp
+-------------------+
| return address    | ← call 指令压入
+-------------------+
| 局部变量/参数 8   | ← [rbp-8]
+-------------------+
| 局部变量/参数 7   | ← [rbp-16]
+-------------------+
| ...               | ← 当前 rsp（栈顶）
低地址
```



上图中 rbp 寄存器的作用是存储函数调用栈的基址指针，即属于 `main` 函数的栈空间的起始位置，而另一个寄存器 rsp 存储的是 `main` 函数调用栈结束的位置，这两个寄存器共同表示了一个函数的栈空间。

在调用 `my_function` 之前，`main` 函数通过 `subq $16, %rsp` 指令分配了 16 个字节的栈地址，随后将第六个以上的参数按照从右到左的顺序存入栈中，即第八个和第七个，余下的六个参数会通过寄存器传递，接下来运行的 `call my_function` 指令会调用 `my_function` 函数：

```
my_function:
    pushq    %rbp
    movq    %rsp, %rbp
    movl    %edi, -4(%rbp)    // rbp-4 = edi = 1
    movl    %esi, -8(%rbp)    // rbp-8 = esi = 2
    ...
    movl    -8(%rbp), %eax    // eax = 2
    movl    -4(%rbp), %edx    // edx = 1
    addl    %eax, %edx        // edx = eax + edx = 3
    ...
    movl    16(%rbp), %eax    // eax = 7
    addl    %eax, %edx        // edx = eax + edx = 28
    movl    24(%rbp), %eax    // eax = 8
    addl    %edx, %eax        // edx = eax + edx = 36
    popq    %rbp
```

`my_function` 会先将寄存器中的全部数据转移到栈上，然后利用 eax 寄存器计算所有入参的和并返回结果。

我们可以将本节的发现和分析简单总结成 —— 当我们在 x86_64 的机器上使用 C 语言中调用函数时，参数都是通过寄存器和栈传递的，其中：

- 六个以及六个以下的参数会按照顺序分别使用 edi、esi、edx、ecx、r8d 和 r9d 六个寄存器传递；
- 六个以上的参数会使用栈传递，函数的参数会以从右到左的顺序依次存入栈中；

而函数的返回值是通过 eax 寄存器进行传递的，由于只使用一个寄存器存储返回值，所以 C 语言的函数不能同时返回多个值。



#### Go 语言

分析了 C 语言函数的调用惯例之后，我们再来剖析一下 Go 语言中函数的调用惯例。我们以下面这个非常简单的代码片段为例简要分析一下：

```go
package main
func myFunction(a, b int) (int, int) {
    return a + b, a - b
}
func main() {
    myFunction(66, 77)
}
```

上述的 `myFunction` 函数接受两个整数并返回两个整数，`main` 函数在调用 `myFunction` 时将 66 和 77 两个参数传递到当前函数中，

使用 `go tool compile -S -N -l main.go` 命令编译上述代码可以得到如下所示的汇编指令：

注：如果编译时不使用 -N -l 参数，编译器会对汇编代码进行优化，编译结果会有较大差别。

```
"".main STEXT size=68 args=0x0 locals=0x28
    0x0000 00000 (main.go:7)    MOVQ    (TLS), CX
    0x0009 00009 (main.go:7)    CMPQ    SP, 16(CX)
    0x000d 00013 (main.go:7)    JLS    61
    0x000f 00015 (main.go:7)    SUBQ    $40, SP      // 分配 40 字节栈空间
    0x0013 00019 (main.go:7)    MOVQ    BP, 32(SP)   // 将基址指针存储到栈上
    0x0018 00024 (main.go:7)    LEAQ    32(SP), BP
    0x001d 00029 (main.go:8)    MOVQ    $66, (SP)    // 第一个参数
    0x0025 00037 (main.go:8)    MOVQ    $77, 8(SP)   // 第二个参数
    0x002e 00046 (main.go:8)    CALL    "".myFunction(SB)
    0x0033 00051 (main.go:9)    MOVQ    32(SP), BP
    0x0038 00056 (main.go:9)    ADDQ    $40, SP
    0x003c 00060 (main.go:9)    RET
```

根据 `main` 函数生成的汇编指令，我们可以分析出 `main` 函数调用 `myFunction` 之前的栈情况：

![golang-function-call-stack-before-calling](go语言设计与实现.assets/b4c819cf0d14966ce7bafaa41a0e3d29.png)

**图 4-2 Go 语言 main 函数的调用栈**

`main` 函数通过 `SUBQ $40, SP` 指令一共在栈上分配了 40 字节的内存空间：

| 空间          | 大小    | 作用                           |
| :------------ | :------ | :----------------------------- |
| SP+32 ~ BP    | 8 字节  | `main` 函数的栈基址指针        |
| SP+16 ~ SP+32 | 16 字节 | 函数 `myFunction` 的两个返回值 |
| SP ~ SP+16    | 16 字节 | 函数 `myFunction` 的两个参数   |

`myFunction` 入参的压栈顺序和 C 语言一样，都是从右到左，即第一个参数 66 在栈顶的 SP ~ SP+8，第二个参数存储在 SP+8 ~ SP+16 的空间中。

当我们准备好函数的入参之后，会调用汇编指令 `CALL "".myFunction(SB)`，这个指令首先会将 `main` 的返回地址存入栈中，然后改变当前的栈指针 SP 并开始执行 `myFunction` 的汇编指令：

```
"".myFunction STEXT nosplit size=49 args=0x20 locals=0x0
    0x0000 00000 (main.go:3)    MOVQ    $0, "".~r2+24(SP) // 初始化第一个返回值
    0x0009 00009 (main.go:3)    MOVQ    $0, "".~r3+32(SP) // 初始化第二个返回值
    0x0012 00018 (main.go:4)    MOVQ    "".a+8(SP), AX    // AX = 66
    0x0017 00023 (main.go:4)    ADDQ    "".b+16(SP), AX   // AX = AX + 77 = 143
    0x001c 00028 (main.go:4)    MOVQ    AX, "".~r2+24(SP) // (24)SP = AX = 143
    0x0021 00033 (main.go:4)    MOVQ    "".a+8(SP), AX    // AX = 66
    0x0026 00038 (main.go:4)    SUBQ    "".b+16(SP), AX   // AX = AX - 77 = -11
    0x002b 00043 (main.go:4)    MOVQ    AX, "".~r3+32(SP) // (32)SP = AX = -11
    0x0030 00048 (main.go:4)    RET
```

从上述的汇编代码中我们可以看出，当前函数在执行时首先会将 `main` 函数中预留的两个返回值地址置成 `int` 类型的默认值 0，然后根据栈的相对位置获取参数并进行加减操作并将值存回栈中，在 `myFunction` 函数返回之间，栈中的数据如图 4-3 所示：

![golang-function-call-stack-before-return](go语言设计与实现.assets/ea21447e7bd40fe5f39b7c27a1a8ad09.png)

**图 4-3 myFunction 函数返回前的栈**

在 `myFunction` 返回之后，`main` 函数会通过以下的指令来恢复栈基址指针并销毁已经失去作用的 40 字节的栈空间：

```
    0x0033 00051 (main.go:9)    MOVQ    32(SP), BP    0x0038 00056 (main.go:9)    ADDQ    $40, SP    0x003c 00060 (main.go:9)    RET
```

通过分析 Go 语言编译后的汇编指令，我们发现 Go 语言使用栈传递参数和接收返回值，所以它只需要在栈上多分配一些内存就可以返回多个值。

> 思考

C 语言和 Go 语言在设计函数的调用惯例时选择也不同的实现。C 语言同时使用寄存器和栈传递参数，使用 eax 寄存器传递返回值；而 Go 语言使用栈传递参数和返回值。我们可以对比一下这两种设计的优点和缺点：

- C 语言的方式能够极大地减少函数调用的额外开销，但是也增加了实现的复杂度；
  - CPU 访问栈的开销比访问寄存器高几十倍[3](https://www.bookstack.cn/read/draveness-golang/5de3a4330de09d9e.md#fn:3)；
  - 需要单独处理函数参数过多的情况；
- Go 语言的方式能够降低实现的复杂度并支持多返回值，但是牺牲了函数调用的性能；
  - 不需要考虑超过寄存器数量的参数应该如何传递；
  - 不需要考虑不同架构上的寄存器差异；
  - 函数入参和出参的内存空间需要在栈上进行分配；

Go 语言使用栈作为参数和返回值传递的方法是综合考虑后的设计，选择这种设计意味着编译器会更加简单、更容易维护。

### 4.1.2 参数传递

除了函数的调用惯例之外，Go 语言在传递参数时是传值还是传引用也是一个有趣的问题，这个问题影响的是当我们在函数中对入参进行修改时会不会影响调用方看到的数据。我们先来介绍一下传值和传引用两者的区别：

- 传值：函数调用时会对参数进行拷贝，被调用方和调用方两者持有不相关的两份数据；
- 传引用：函数调用时会**传递参数的指针**，被调用方和调用方两者持有相同的数据，任意一方做出的修改都会影响另一方。

不同语言会选择不同的方式传递参数，Go 语言选择了传值的方式，**无论是传递基本类型、结构体还是指针，都会对传递的参数进行拷贝**。本节剩下的内容将会验证这个结论的正确性。

#### 整型和数组

我们先来分析 Go 语言是如何传递基本类型和数组的。如下所示的函数 `myFunction` 接收了两个参数，整型变量 `i` 和数组 `arr`，这个函数会将传入的两个参数的地址打印出来，在最外层的主函数也会在 `myFunction` 函数调用前后分别打印两个参数的地址：

```
func myFunction(i int, arr [2]int) {
    fmt.Printf("in my_funciton - i=(%d, %p) arr=(%v, %p)\n", i, &i, arr, &arr)
}
func main() {
    i := 30
    arr := [2]int{66, 77}
    fmt.Printf("before calling - i=(%d, %p) arr=(%v, %p)\n", i, &i, arr, &arr)
    myFunction(i, arr)
    fmt.Printf("after  calling - i=(%d, %p) arr=(%v, %p)\n", i, &i, arr, &arr)
}
$ go run main.go
before calling - i=(30, 0xc00009a000) arr=([66 77], 0xc00009a010)
in my_funciton - i=(30, 0xc00009a008) arr=([66 77], 0xc00009a020)
after  calling - i=(30, 0xc00009a000) arr=([66 77], 0xc00009a010)
```

当我们通过命令运行这段代码我们会发现，`main` 函数和被调用者 `myFunction` 中参数的地址是完全不同的。

不过从 `main` 函数的角度来看，在调用 `myFunction` 前后，整数 `i` 和数组 `arr` 两个参数的地址都没有变化。那么如果我们在 `myFunction` 函数内部对参数进行修改是否会影响 `main` 函数中的变量呢？我们更新 `myFunction` 函数并重新执行这段代码：

```
func myFunction(i int, arr [2]int) {
    i = 29
    arr[1] = 88
    fmt.Printf("in my_funciton - i=(%d, %p) arr=(%v, %p)\n", i, &i, arr, &arr)
}
$ go run main.go
before calling - i=(30, 0xc000072008) arr=([66 77], 0xc000072010)
in my_funciton - i=(29, 0xc000072028) arr=([66 88], 0xc000072040)
after  calling - i=(30, 0xc000072008) arr=([66 77], 0xc000072010)
```



你可以看到在 `myFunction` 中对参数的修改也仅仅影响了当前函数，并没有影响调用方 `main` 函数，所以我们能给出如下的结论 

- **Go 语言中对于整型和数组类型的参数都是值传递的**，也就是在调用函数时会对内容进行拷贝，需要注意的是如果当前数组的大小非常的大，这种传值方式就会对性能造成比较大的影响。



#### 结构体和指针

接下来我们继续分析 Go 语言另外两种常见类型 —— 结构体和指针。在这段代码中定义一个只包含一个成员变量的简单结构体 `MyStruct` 以及接受两个参数的 `myFunction` 方法：

```go
type MyStruct struct {
    i int
}
func myFunction(a MyStruct, b *MyStruct) {
    a.i = 31
    b.i = 41
    fmt.Printf("in my_function - a=(%d, %p) b=(%v, %p)\n", a, &a, b, &b)
}
func main() {
    a := MyStruct{i: 30}
    b := &MyStruct{i: 40}
    fmt.Printf("before calling - a=(%d, %p) b=(%v, %p)\n", a, &a, b, &b)
    myFunction(a, b)
    fmt.Printf("after calling  - a=(%d, %p) b=(%v, %p)\n", a, &a, b, &b)
}
$ go run main.go
before calling - a=({30}, 0xc000018178) b=(&{40}, 0xc00000c028)
in my_function - a=({31}, 0xc000018198) b=(&{41}, 0xc00000c038)
after calling  - a=({30}, 0xc000018178) b=(&{41}, 0xc00000c028)


===========
func main() {
	a := MyStruct{i: 30}
	b := &MyStruct{i: 40}
	fmt.Printf("before calling - a=(%d, %p) b=(%v, %p)\n", a, &a, b, b)
	myFunction(a, b)
	fmt.Printf("after calling  - a=(%d, %p) b=(%v, %p)\n", a, &a, b, b)
}
==》
before calling - a=({30}, 0xc000134000) b=(&{40}, 0xc000134008)
in my_function - a=({31}, 0xc000134018) b=(&{41}, 0xc000134008)
after calling  - a=({30}, 0xc000134000) b=(&{41}, 0xc000134008)
（在 Go 语言中，%p 是格式化字符串中的占位符，用于打印指针的十六进制表示形式，即指向的内存地址。）

```



从运行的结果我们可以得出如下结论：

- 传递结构体时：会对结构体中的全部内容进行拷贝；
- 传递结构体指针时：会对结构体指针进行拷贝；

对结构体指针的修改是改变了指针指向的结构体，`b.i` 可以被理解成 `(*b).i`，也就是我们先获取指针 `b` 背后的结构体，再修改结构体的成员变量。我们简单修改上述代码，分析一下 Go 语言结构体在内存中的布局：

```go
type MyStruct struct {
    i int
    j int
}
func myFunction(ms *MyStruct) {
    ptr := unsafe.Pointer(ms)
    for i := 0; i < 2; i++ {
        c := (*int)(unsafe.Pointer((uintptr(ptr) + uintptr(8*i))))
        *c += i + 1
        fmt.Printf("[%p] %d\n", c, *c)
    }
}
func main() {
    a := &MyStruct{i: 40, j: 50}
    myFunction(a)
    fmt.Printf("[%p] %v\n", a, a)
}
$ go run main.go
[0xc000018180] 41
[0xc000018188] 52
[0xc000018180] &{41 52}
```

在这段代码中，我们通过指针的方式修改结构体中的成员变量，结构体在内存中是一片连续的空间，指向结构体的指针也是指向这个结构体的首地址。将 `MyStruct` 指针修改成 `int` 类型的，那么访问新指针就会返回整型变量 `i`，将指针移动 8 个字节之后就能获取下一个成员变量 `j`。



如果我们将上述代码简化成如下所示的代码片段并使用 `go tool compile` 进行编译会得到如下的结果：

```
type MyStruct struct {
    i int
    j int
}
func myFunction(ms *MyStruct) *MyStruct {
    return ms
}
$ go tool compile -S -N -l main.go
"".myFunction STEXT nosplit size=20 args=0x10 locals=0x0
    0x0000 00000 (main.go:8)    MOVQ    $0, "".~r1+16(SP) // 初始化返回值
    0x0009 00009 (main.go:9)    MOVQ    "".ms+8(SP), AX   // 复制引用
    0x000e 00014 (main.go:9)    MOVQ    AX, "".~r1+16(SP) // 返回引用
    0x0013 00019 (main.go:9)    RET
```

在这段汇编语言中我们发现当参数是指针时，也会使用 `MOVQ "".ms+8(SP), AX` 指令对引用进行复制，然后将复制后的指针作为返回值传递回调用方。

<img src="go语言设计与实现.assets/51cfe2b5b77b52847fc9776814365dd0.png" alt="golang-pointer-as-argument" style="zoom: 80%;" />

**图 4-4 Go 语言指针参数**

所以将指针作为参数传入某一个函数时，在函数内部会对指针进行复制，也就是会同时出现两个指针指向原有的内存空间，所以 Go 语言中『传指针』也是传值。







#### 传值

当我们对 Go 语言中大多数常见的数据结构进行验证之后，其实就能够推测出 Go 语言在传递参数时其实使用的就是传值的方式，接收方收到参数时会对这些参数进行复制；了解到这一点之后，在传递数组或者内存占用非常大的结构体时，我们在一些函数中应该尽量使用指针作为参数类型来避免发生大量数据的拷贝而影响性能。



### 4.1.3 小结

这一节我们详细分析了 Go 语言的调用惯例，包括传递参数和返回值的过程和原理。Go 通过栈传递函数的参数和返回值，在调用函数之前会在栈上为返回值分配合适的内存空间，随后将入参从右到左按顺序压栈并拷贝参数，返回值会被存储到调用方预留好的栈空间上，我们可以简单总结出以下几条规则：

- **通过堆栈传递参数，入栈的顺序是从右到左**；
- 函数返回值通过堆栈传递并由调用者预先分配内存空间；
- 调用函数时都是传值，接收方会对入参进行复制再计算；



















## 4.2 接口

- [4.2 接口](https://www.bookstack.cn/read/draveness-golang/5833e4cb4df6241b.md)

Go 语言中的接口就是一组方法的签名（集合），它是 Go 语言的重要组成部分。使用接口能够让我们更好地组织并写出易于测试的代码，然而很多工程师对 Go的接口了解都非常有限，也不清楚其底层的实现原理，这成为了开发高性能服务的最大阻碍。

本节会介绍使用接口时遇到的一些常见问题以及它的设计与实现，包括接口的类型转换、类型断言以及动态派发机制，帮助各位读者更好地理解接口类型。

### 4.2.1 概述

**在计算机科学中，接口是计算机系统中多个组件共享的边界，不同的组件能够在边界上交换信息**[1](https://www.bookstack.cn/read/draveness-golang/5833e4cb4df6241b.md#fn:1)。如图 4-5，接口的本质就是引入一个新的中间层，调用方可以通过接口与具体实现分离，解除上下游的耦合，上层的模块不再需要依赖下层的具体模块，只需要依赖一个约定好的接口。

![golang-interface](go语言设计与实现.assets/7858ccfb5ae7bc7ea2ca72425bde0339.png)

**图 4-5 上下游通过接口解耦**

这种面向接口的编程方式有着非常强大的生命力，无论是在框架还是操作系统中我们都能够找到接口的身影。可移植操作系统接口（Portable Operating System Interface，POSIX)[2](https://www.bookstack.cn/read/draveness-golang/5833e4cb4df6241b.md#fn:2)就是一个典型的例子，它定义了应用程序接口和命令行等标准，为计算机软件带来了可移植性 — 只要操作系统实现了 POSIX，计算机软件就可以直接在不同操作系统上运行。

除了解耦有依赖关系的上下游，接口还能够帮助我们隐藏底层实现，减少关注点。《计算机程序的构造和解释》中有这么一句话：

> 代码必须能够被人阅读，只是机器恰好可以执行[3](https://www.bookstack.cn/read/draveness-golang/5833e4cb4df6241b.md#fn:3)

人能够同时处理的信息非常有限，定义良好的接口能够隔离底层的实现，让我们将重点放在当前的代码片段中。SQL 就是接口的一个例子，当我们使用 SQL 语句查询数据时，其实不需要关心底层数据库的具体实现，我们只在乎 SQL 返回的结果是否符合预期。

![sql-and-databases](go语言设计与实现.assets/e8633919e287edf2f68b643408ebc902.png)

**图 4-6 SQL 和不同数据库**

计算机科学中的接口是一个比较抽象的概念，但是编程语言中接口的概念就更加具体。

Go 语言中的接口是一种内置的类型，它定义了一组方法的签名，这一小节会先介绍 Go 语言接口的几个基本概念以及常见问题，为之后介绍实现原理做一些铺垫。

#### 隐式接口

很多面向对象语言都有接口这一概念，例如 Java 和 C#。Java 的接口不仅可以定义方法签名，还可以定义变量，这些定义的变量可以直接在实现接口的类中使用：

```java
public interface MyInterface {
    public String hello = "Hello";
    public void sayHello();
}
```

上述代码定义了一个必须实现的方法 `sayHello` 和一个会注入到实现类的变量 `hello`。在下面的代码中，`MyInterfaceImpl` 就实现了 `MyInterface` 接口：

```java
public class MyInterfaceImpl implements MyInterface {
    public void sayHello() {
        System.out.println(MyInterface.hello);
    }
}
```



Java 中的类必须通过上述方式显式地声明实现的接口，但是在 Go 语言中实现接口就不需要使用类似的方式。首先，我们简单了解一下在 Go 语言中如何定义接口。定义接口需要使用 `interface` 关键字，在接口中我们只能定义方法签名，不能包含成员变量，一个常见的 Go 语言接口是这样的：

```
type error interface {
    Error() string
}
```



如果一个类型需要实现 `error` 接口，那么它只需要实现 `Error() string` 方法，下面的 `RPCError` 结构体就是 `error` 接口的一个实现：

```go
type RPCError struct {
    Code    int64
    Message string
}
func (e *RPCError) Error() string {
    return fmt.Sprintf("%s, code=%d", e.Message, e.Code)
}
```

细心的读者可能会发现上述代码根本就没有 `error` 接口的影子，这是为什么呢？Go 语言中**接口的实现都是隐式的**，我们只需要实现 `Error() string` 方法实现了 `error` 接口。Go 语言实现接口的方式与 Java 完全不同：

- 在 Java 中：实现接口需要显式的声明接口并实现所有方法；
- 在 Go 中：实现接口的所有方法就隐式的实现了接口；

我们使用上述 `RPCError` 结构体时并不关心它实现了哪些接口，**Go 语言只会在传递参数、返回参数以及变量赋值时才会对某个类型是否实现接口进行检查**，这里举几个例子来演示发生接口类型检查的时机：



```go
func main() {
    var rpcErr error = NewRPCError(400, "unknown err") // typecheck1
    err := AsErr(rpcErr) // typecheck2
    println(err) 
}
func NewRPCError(code int64, msg string) error {
    return &RPCError{ // typecheck3
        Code:    code,
        Message: msg,
    }
}
func AsErr(err error) error {
    return err
}
```

Go 语言会[编译期间](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md)对代码进行类型检查，上述代码总共触发了三次类型检查：

- 将 `*RPCError` 类型的变量赋值给 `error` 类型的变量 `rpcErr`；
- 将 `*RPCError` 类型的变量 `rpcErr` 传递给签名中参数类型为 `error` 的 `AsErr` 函数；
- 将 `*RPCError` 类型的变量从函数签名的返回值类型为 `error` 的 `NewRPCError` 函数中返回；从类型检查的过程来看，编译器仅在需要时才对类型进行检查，类型实现接口时只需要实现接口中的全部方法，不需要像 Java 等编程语言中一样显式声明。



#### 类型

接口也是 Go 语言中的一种类型，它能够出现在变量的定义、函数的入参和返回值中并对它们进行约束，不过 Go 语言中有两种略微不同的接口，一种是带有一组方法的接口，另一种是不带任何方法的 `interface{}`：

![golang-different-interface](go语言设计与实现.assets/12abffc504eaf4e9411b15a5e8f8c784.png)

**图 4-7 Go 语言中的两种接口**



Go 语言使用 `iface` 结构体表示第一种接口，使用 `eface` 结构体表示第二种空接口，两种接口虽然都使用 `interface` 声明，但是由于后者在 Go 语言中非常常见，所以在实现时使用了特殊的类型。

需要注意的是，与 C 语言中的 `void ` *不同，`interface{}` 类型**不是任意类型**，如果我们将类型转换成了 `interface{}` 类型，这边的变量在运行期间的类型也发生了变化，获取变量类型时就会得到 `interface{}`。

```go
package main
func main() {
    type Test struct{}
    v := Test{}
    Print(v)
}
func Print(v interface{}) {
    println(v)
}
```

上述函数不接受任意类型的参数，只接受 `interface{}` 类型的值，在调用 `Print` 函数时会对参数 `v` 进行类型转换，将原来的 `Test` 类型转换成 `interface{}` 类型，我们会在本节后面介绍类型转换的过程和原理。



#### 指针和接口

在 Go 语言中同时使用指针和接口时会发生一些让人困惑的问题，接口在定义一组方法时没有对实现的接收者做限制，所以我们会看到『一个类型』实现接口的两种方式：

![golang-interface-and-pointer](go语言设计与实现.assets/c2c845570c614588153a36d3be1590e1.png)

**图 4-8 结构体和指针实现接口**

这是因为**结构体类型和指针类型是完全不同的**，就像我们不能向一个接受指针的函数传递结构体，在实现接口时这两种类型也不能划等号。但是上图中的两种实现不可以同时存在，Go 语言的编译器会在结构体类型和指针类型都实现一个方法时报错 —— `method redeclared`。



对 `Cat` 结构体来说，它**在实现接口时可以选择接受者的类型**，即结构体或者结构体指针，在初始化时也可以初始化成结构体或者指针。下面的代码总结了如何使用结构体、结构体指针实现接口，以及如何使用结构体、结构体指针初始化变量。

```
type Cat struct {}
type Duck interface { ... }

type (c  Cat) Quack {}  // 使用结构体实现接口
type (c *Cat) Quack {}  // 使用结构体指针实现接口

var d Duck = Cat{}      // 使用结构体初始化变量
var d Duck = &Cat{}     // 使用结构体指针初始化变量
```



实现接口的类型和初始化返回的类型两个维度组成了四种情况，这四种情况并不都能通过编译器的检查：

|                      | 结构体实现接口 | 结构体指针实现接口 |
| :------------------- | :------------- | :----------------- |
| 结构体初始化变量     | 通过           | 不通过             |
| 结构体指针初始化变量 | 通过           | 通过               |

四种中只有『使用指针实现接口，使用结构体初始化变量』无法通过编译，其他的三种情况都可以正常执行。当实现接口的类型和初始化变量时返回的类型时相同时，代码通过编译是理所应当的：

- 方法接受者和初始化类型都是结构体；
- 方法接受者和初始化类型都是结构体指针；

而剩下的两种方式为什么一种能够通过编译，另一种无法通过编译呢？我们先来看一下能够通过编译的情况，

也就是方法的接受者是结构体，而初始化的变量是结构体指针：

```go
type Cat struct{}
func (c Cat) Quack() {
    fmt.Println("meow")
}
func main() {
    var c Duck = &Cat{}
    c.Quack()
}
```

作为指针的 `&Cat{}` 变量能够**隐式地获取**到指向的结构体，所以能在结构体上调用 `Walk` 和 `Quack` 方法（可以将结构体变量赋值给结构体指针，指针会隐式获取对应结构体的指针）。我们可以将这里的调用理解成 C 语言中的 `d->Walk()` 和 `d->Speak()`，它们都会先获取指向的结构体再执行对应的方法。

但是如果我们将上述代码中方法的接受者和初始化的类型进行交换，代码就无法通过编译了：

```
type Duck interface {
    Quack()
}
type Cat struct{}
func (c *Cat) Quack() {
    fmt.Println("meow")
}
func main() {
    var c Duck = Cat{}
    c.Quack()
}
$ go build interface.go
./interface.go:20:6: cannot use Cat literal (type Cat) as type Duck in assignment:
    Cat does not implement Duck (Quack method has pointer receiver)
```



![image-20241028162550344](go语言设计与实现.assets/image-20241028162550344.png)

编译器会提醒我们：`Cat` 类型没有实现 `Duck` 接口，`Quack` 方法的接受者是指针。这两个报错对于刚刚接触 Go 语言的开发者比较难以理解，如果我们想要搞清楚这个问题，首先要知道 Go 语言在[传递参数](https://www.bookstack.cn/read/draveness-golang/5de3a4330de09d9e.md)时都是传值的。

![golang-interface-method-receive](go语言设计与实现.assets/c129330373c0d63f2760c54b837b586b.png)

**图 4-9 实现接口的接受者类型**

如上图所示，无论上述代码中初始化的变量 `c` 是 `Cat{}` 还是 `&Cat{}`，使用 `c.Quack()` 调用方法时都会发生值拷贝：

- 如图 4-9 左侧，对于 `&Cat{}` 来说，这意味着拷贝一个新的 `&Cat{}` 指针，这个指针与原来的指针指向一个相同并且唯一的结构体，所以编译器可以隐式的对变量解引用（dereference）获取指针指向的结构体；
- 如图 4-9 右侧，对于 `Cat{}` 来说，这意味着 `Quack` 方法会接受一个全新的 `Cat{}`，因为方法的参数是 `*Cat`，编译器不会无中生有创建一个新的指针；即使编译器可以创建新指针，这个指针指向的也不是最初调用该方法的结构体；



上面的分析解释了指针类型的现象，当我们使用指针实现接口时，只有指针类型的变量才会实现该接口；当我们使用结构体实现接口时，指针类型和结构体类型都会实现该接口。当然这并不意味着我们应该一律使用结构体实现接口，这个问题在实际工程中也没那么重要，在这里我们只想解释现象背后的原因。



#### nil 和 non-nil

我们可以通过一个例子理解**『Go 语言的接口类型不是任意类型』**这一句话，下面的代码在 `main` 函数中初始化了一个 `*TestStruct` 结构体指针，由于指针的零值是 `nil`，所以变量 `s` 在初始化之后也是 `nil`：

```go
package main
type TestStruct struct{}
func NilOrNot(v interface{}) bool {
    return v == nil
}
func main() {
    var s *TestStruct
    fmt.Println(s == nil)      // #=> true
    fmt.Println(NilOrNot(s))   // #=> false
}
$ go run main.go
true
false
```

我们简单总结一下上述代码执行的结果：

- 将上述变量与 `nil` 比较会返回 `true`；
- 将上述变量传入 `NilOrNot` 方法并与 `nil` 比较会返回 `false`；

出现上述现象的原因是 —— 调用 `NilOrNot` 函数时发生了**隐式的类型转换**，除了向方法传入参数之外，变量的赋值也会触发隐式类型转换。在类型转换时，`*TestStruct` 类型会转换成 `interface{}` 类型，转换后的变量不仅包含转换前的变量，还包含变量的类型信息 `TestStruct`，所以转换后的变量与 `nil` 不相等。



### 4.2.2 数据结构

相信各位读者已经对 Go 语言的接口有了一些的了解，接下来我们从源代码和汇编指令层面介绍接口的底层数据结构。

Go 语言根据接口类型『是否包含一组方法』对类型做了不同的处理。我们使用 `iface` 结构体表示包含方法的接口；使用 `eface` 结构体表示不包含任何方法的 `interface{}` 类型，`eface` 结构体在 Go 语言的定义是这样的：

```go
type eface struct { // 16 bytes
    _type *_type
    data  unsafe.Pointer
}
```

由于 `interface{}` 类型不包含任何方法，所以它的结构也相对来说比较简单，只包含指向底层数据和类型的两个指针。从上述结构我们也能推断出 — Go 语言中的任意类型都可以转换成 `interface{}` 类型。

另一个用于表示接口的结构体就是 `iface`，这个结构体中有指向原始数据的指针 `data`，不过更重要的是 `itab` 类型中的 `tab` 字段。

```
type iface struct { // 16 bytes
    tab  *itab
    data unsafe.Pointer
}
```

接下来我们将详细分析 Go 语言接口中的这两个类型，即 `_type` 和 `itab`。



#### 类型结构体

`_type` 是 Go 语言类型的运行时表示。下面是运行时包中的结构体，结构体包含了很多元信息，例如：类型的大小、哈希、对齐以及种类等。

```go
type _type struct {
    size       uintptr
    ptrdata    uintptr
    hash       uint32
    tflag      tflag
    align      uint8
    fieldAlign uint8
    kind       uint8
    equal      func(unsafe.Pointer, unsafe.Pointer) bool
    gcdata     *byte
    str        nameOff
    ptrToThis  typeOff
}
```

- `size` 字段存储了类型占用的内存空间，为内存空间的分配提供信息；
- `hash` 字段能够帮助我们快速确定类型是否相等；
- `equal` 字段用于判断当前类型的多个对象是否相等，该字段是为了减少 Go 语言二进制包大小从 `typeAlg` 结构体中迁移过来的[4](https://www.bookstack.cn/read/draveness-golang/5833e4cb4df6241b.md#fn:4)；

我们只需要对 `_type` 结构体中的字段有一个大体的概念，不需要详细理解所有字段的作用和意义。



#### itab 结构体

`itab` 结构体是接口类型的核心组成部分，每一个 `itab` 都占 32 字节的空间，我们可以将其看成接口类型和具体类型的组合，它们分别用 `inter` 和 `_type` 两个字段表示：

```go
type itab struct { // 32 bytes
    inter *interfacetype
    _type *_type
    hash  uint32
    _     [4]byte
    fun   [1]uintptr
}
```

除了 `inter` 和 `_type` 两个用于表示类型的字段之外，上述结构体中的另外两个字段也有自己的作用：

- `hash` 是对 `_type.hash` 的拷贝，当我们想将 `interface` 类型转换成具体类型时，可以使用该字段快速判断目标类型和具体类型 `_type` 是否一致；
- `fun` 是一个动态大小的数组，它是一个用于动态派发的虚函数表，存储了一组函数指针。**虽然该变量被声明成大小固定的数组，但是在使用时会通过原始指针获取其中的数据**，所以 `fun` 数组中保存的元素数量是不确定的；

我们会在类型断言中介绍 `hash` 字段的使用，在动态派发一节中介绍 `fun` 数组中存储的函数指针是如何被使用的。



### 4.2.3 类型转换

既然我们已经了解了接口在运行时的数据结构，接下来会通过几个例子来深入理解接口类型是如何初始化和传递的，这里会介绍在实现接口时使用指针类型和结构体类型的区别。这两种不同的接口实现方式会导致 Go 语言编译器生成不同的汇编代码，带来执行过程上的一些差异。

#### 指针类型

首先我们回到这一节开头提到的 `Duck` 接口的例子，我们使用 `//go:noinline` 指令[5](https://www.bookstack.cn/read/draveness-golang/5833e4cb4df6241b.md#fn:5)禁止 `Quack` 方法的内联编译：

```
package main
type Duck interface {
    Quack()
}
type Cat struct {
    Name string
}
//go:noinline
func (c *Cat) Quack() {
    println(c.Name + " meow")
}
func main() {
    var c Duck = &Cat{Name: "grooming"}
    c.Quack()
}
```

我们使用编译器将上述代码编译成汇编语言，删掉其中一些对理解接口原理无用的指令并保留与赋值语句 `var c Duck = &Cat{Name: "grooming"}` 相关的代码，我们将生成的汇编指令拆分成三部分分析：

- 结构体 `Cat` 的初始化；
- 赋值触发的类型转换过程；
- 调用接口的方法 `Quack()`；我们先来分析结构体 `Cat` 的初始化过程：

```
LEAQ    type."".Cat(SB), AX                ;; AX = &type."".Cat
MOVQ    AX, (SP)                           ;; SP = &type."".Cat
CALL    runtime.newobject(SB)              ;; SP + 8 = &Cat{}
MOVQ    8(SP), DI                          ;; DI = &Cat{}
MOVQ    $8, 8(DI)                          ;; StringHeader(DI.Name).Len = 8
LEAQ    go.string."grooming"(SB), AX       ;; AX = &"grooming"
MOVQ    AX, (DI)                           ;; StringHeader(DI.Name).Data = &"grooming"
```



- 获取 `Cat` 结构体类型指针并将其作为参数放到栈上；
- 通过 `CALL` 指定调用 [`runtime.newobject`](https://github.com/golang/go/blob/5042317d6919d4c84557e04be35130430e8d1dd4/src/runtime/malloc.go#L1162-L1164) 函数，这个函数会以 `Cat` 结构体类型指针作为入参，分配一片新的内存空间并将指向这片内存空间的指针返回到 SP+8 上；
- SP+8 现在存储了一个指向 `Cat` 结构体的指针，我们将栈上的指针拷贝到寄存器 `DI` 上方便操作；
- 由于 `Cat` 中只包含一个字符串类型的 `Name` 变量，所以在这里会分别将字符串地址 `&"grooming"` 和字符串长度 `8` 设置到结构体上，最后三行汇编指令等价于 `cat.Name = "grooming"`；字符串在运行时的表示其实就是指针加上字符串长度，在前面的章节[字符串](https://www.bookstack.cn/read/draveness-golang/0bc3143426c7b7ea.md)已经介绍过它的底层表示和实现原理，但是我们这里要看一下初始化之后的 `Cat` 结构体在内存中的表示是什么样的：

![golang-new-struct-pointe](go语言设计与实现.assets/ae2487ad9dfa69afce60fe283e4e521d.png)

**图 4-10 Cat 结构体指针**

因为 `Cat` 结构体的定义中只包含一个字符串，而字符串在 Go 语言中总共占 16 字节，所以每一个 `Cat` 结构体的大小都是 16 字节。初始化 `Cat` 结构体之后就进入了将 `*Cat` 转换成 `Duck` 类型的过程了：

```
LEAQ    go.itab.*"".Cat,"".Duck(SB), AX    ;; AX = *itab(go.itab.*"".Cat,"".Duck)
MOVQ    DI, (SP)                           ;; SP = AX
```

类型转换的过程比较简单，`Duck` 作为一个包含方法的接口，它在底层使用 `iface` 结构体表示。`iface` 结构体包含两个字段，其中一个是指向数据的指针，另一个是表示接口和结构体关系的 `tab` 字段，我们已经通过上一段代码 SP+8 初始化了 `Cat` 结构体指针，这段代码只是将编译期间生成的 `itab` 结构体指针复制到 SP 上：

![golang-struct-pointer-to-iface](go语言设计与实现.assets/2f3839b315cda204c7ed78a9b6c6788c.png)

**图 4-11 Cat 类型转换**

到这里，我们会发现 SP ~ SP+16 共同组成了 `iface` 结构体，而栈上的这个 `iface` 结构体也是 `Quack` 方法的第一个入参。

```
CALL    "".(*Cat).Quack(SB)                ;; SP.Quack()
```

上述代码会直接通过 `CALL` 指令完成方法的调用，细心的读者可能会发现一个问题 —— 为什么在代码中我们调用的是 `Duck.Quack` 但生成的汇编是 `*Cat.Quack` 呢？Go 语言的编译器会在编译期间将一些需要动态派发的方法调用改写成对目标方法的直接调用，以减少性能的额外开销。如果在这里禁用编译器优化，就会看到动态派发的过程，我们会在后面分析接口的动态派发以及性能上的额外开销。



#### 结构体类型

在这里，我们继续修改上一节中的代码，使用结构体类型实现 `Duck` 接口并初始化结构体类型的变量：

```go
package main
type Duck interface {
    Quack()
}
type Cat struct {
    Name string
}
//go:noinline
func (c Cat) Quack() {
    println(c.Name + " meow")
}
func main() {
    var c Duck = Cat{Name: "grooming"}
    c.Quack()
}
```

如果我们在初始化变量时使用指针类型 `&Cat{Name: "grooming"}` 也能够通过编译，不过生成的汇编代码和上一节中的几乎完全相同，所以这里也就不分析这个情况了。

编译上述的代码会得到如下所示的汇编指令，需要注意的是为了代码更容易理解和分析，这里的汇编指令依然经过了删减，不过不会影响具体的执行过程。与上一节一样，我们将汇编代码的执行过程分成以下几个部分：

- 初始化 `Cat` 结构体；
- 完成从 `Cat` 到 `Duck` 接口的类型转换；
- 调用接口的 `Quack` 方法；我们先来看一下上述汇编代码中用于初始化 `Cat` 结构体的部分：

```
XORPS   X0, X0                          ;; X0 = 0
MOVUPS  X0, ""..autotmp_1+32(SP)        ;; StringHeader(SP+32).Data = 0
LEAQ    go.string."grooming"(SB), AX    ;; AX = &"grooming"
MOVQ    AX, ""..autotmp_1+32(SP)        ;; StringHeader(SP+32).Data = AX
MOVQ    $8, ""..autotmp_1+40(SP)        ;; StringHeader(SP+32).Len =8
```

这段汇编指令会在栈上初始化 `Cat` 结构体，而上一节的代码在堆上申请了 16 字节的内存空间，栈上只有一个指向 `Cat` 的指针。

初始化结构体后就进入类型转换的阶段，编译器会将 `go.itab."".Cat,"".Duck` 的地址和指向 `Cat` 结构体的指针作为参数一并传入 [`runtime.convT2I`](https://github.com/golang/go/blob/0c5d545ccdd01403d6ce865fb03774a6aff6032c/src/runtime/iface.go#L398-L411) 函数：

```
LEAQ    go.itab."".Cat,"".Duck(SB), AX     ;; AX = &(go.itab."".Cat,"".Duck)
MOVQ    AX, (SP)                           ;; SP = AX
LEAQ    ""..autotmp_1+32(SP), AX           ;; AX = &(SP+32) = &Cat{Name: "grooming"}
MOVQ    AX, 8(SP)                          ;; SP + 8 = AX
CALL    runtime.convT2I(SB)                ;; runtime.convT2I(SP, SP+8)
```

这个函数会获取 `itab` 中存储的类型，根据类型的大小申请一片内存空间并将 `elem` 指针中的内容拷贝到目标的内存空间中：

```
func convT2I(tab *itab, elem unsafe.Pointer) (i iface) {
    t := tab._type
    x := mallocgc(t.size, t, true)
    typedmemmove(t, x, elem)
    i.tab = tab
    i.data = x
    return
}
```

[`runtime.convT2I`](https://github.com/golang/go/blob/0c5d545ccdd01403d6ce865fb03774a6aff6032c/src/runtime/iface.go#L398-L411) 会返回一个 `iface` 结构体，其中包含 `itab` 指针和 `Cat` 变量。当前函数返回之后，`main` 函数的栈上会包含以下数据：

![golang-struct-to-iface](go语言设计与实现.assets/6278fa1f42d924af680c7ed03d7ce1e4.png)

**图 4-12 结构体到指针**

SP 和 SP+8 中存储的 `itab` 和 `Cat` 指针就是 [`runtime.convT2I`](https://github.com/golang/go/blob/0c5d545ccdd01403d6ce865fb03774a6aff6032c/src/runtime/iface.go#L398-L411) 函数的入参，这个函数的返回值位于 SP+16，是一个占 16 字节内存空间的 `iface` 结构体，SP+32 存储的就是在栈上的 `Cat` 结构体，它会在 [`runtime.convT2I`](https://github.com/golang/go/blob/0c5d545ccdd01403d6ce865fb03774a6aff6032c/src/runtime/iface.go#L398-L411) 执行的过程中拷贝到堆上。

在最后，我们会通过以下的指令调用 `Cat` 实现的接口方法 `Quack()`：

```
MOVQ    16(SP), AX ;; AX = &(go.itab."".Cat,"".Duck)
MOVQ    24(SP), CX ;; CX = &Cat{Name: "grooming"}
MOVQ    24(AX), AX ;; AX = AX.fun[0] = Cat.Quack
MOVQ    CX, (SP)   ;; SP = CX
CALL    AX         ;; CX.Quack()
```

这几个汇编指令还是非常好理解的，`MOVQ 24(AX), AX` 是最关键的指令，它从 `itab` 结构体中取出 `Cat.Quack` 方法指针作为 `CALL` 指令调用时的参数。接口变量的第 24 字节是 `itab.fun` 数组开始的位置，由于 `Duck` 接口只包含一个方法，所以 `itab.fun[0]` 中存储的就是指向 `Quack` 方法的指针了。



### 4.2.4 类型断言

上一节介绍是如何把具体类型转换成接口类型，而这一节介绍的是如何将一个接口类型转换成具体类型。本节会根据接口中是否存在方法分两种情况介绍类型断言的执行过程。

#### 非空接口

首先分析接口中包含方法的情况，`Duck` 接口一个非空的接口，我们来分析从 `Duck` 转换回 `Cat` 结构体的过程：

```
func main() {
    var c Duck = &Cat{Name: "grooming"}
    switch c.(type) {
    case *Cat:
        cat := c.(*Cat)
        cat.Quack()
    }
}
```

我们将编译得到的汇编指令分成两部分分析，第一部分是变量的初始化，第二部分是类型短信，第一部分的代码如下：

```
00000 TEXT    "".main(SB), ABIInternal, $32-0
...
00029 XORPS    X0, X0
00032 MOVUPS    X0, ""..autotmp_4+8(SP)
00037 LEAQ    go.string."grooming"(SB), AX
00044 MOVQ    AX, ""..autotmp_4+8(SP)
00049 MOVQ    $8, ""..autotmp_4+16(SP)
```

0037 ~ 0049 三个指令初始化了 `Duck` 变量，`Cat` 结构体初始化在 SP+8 ~ SP+24 上。因为 Go 语言的编译器做了一些优化，所以代码中没有`iface` 的构建过程，不过对于这一节要介绍的类型断言和转换没有太多的影响。下面进入类型转换的部分：

```
00058 CMPL  go.itab.*"".Cat,"".Duck+16(SB), $593696792  
                                        ;; if (c.tab.hash != 593696792) {
00068 JEQ   80                          ;;      
00070 MOVQ  24(SP), BP                  ;;      BP = SP+24
00075 ADDQ  $32, SP                     ;;      SP += 32
00079 RET                               ;;      return
                                        ;; } else {
00080 LEAQ  ""..autotmp_4+8(SP), AX     ;;      AX = &Cat{Name: "grooming"}
00085 MOVQ  AX, (SP)                    ;;      SP = AX
00089 CALL  "".(*Cat).Quack(SB)         ;;      SP.Quack()
00094 JMP   70                          ;;      ...
                                        ;;      BP = SP+24
                                        ;;      SP += 32
                                        ;;      return
                                        ;; }
```

switch语句生成的汇编指令会将目标类型的 `hash` 与接口变量中的 `itab.hash` 进行比较：

- 如果两者相等意味着变量的具体类型是Cat，我们会跳转到 0080 所在的分支完成类型转换。
  - 获取 SP+8 存储的 `Cat` 结构体指针；
  - 将结构体指针拷贝到栈顶；
  - 调用 `Quack` 方法；
  - 恢复函数的栈并返回；
- 如果接口中存在的具体类型不是 `Cat`，就会直接恢复栈指针并返回到调用方；

![golang-interface-to-struct](go语言设计与实现.assets/632c104da26faabce818002fa32e905f.png)

**图 4-13 接口转换成结构体**

上图展示了调用 `Quack` 方法时的堆栈情况，其中 `Cat` 结构体存储在 SP+8 ~ SP+24 上，`Cat` 指针存储在栈顶并指向上述结构体。

#### 空接口

当我们使用空接口类型 `interface{}` 进行类型断言时，如果不关闭 Go 语言编译器的优化选项，生成的汇编指令是差不多的。编译器会省略将 `Cat` 结构体转换成 `eface` 的过程：

```go
func main() {
    var c interface{} = &Cat{Name: "grooming"}
    switch c.(type) {
    case *Cat:
        cat := c.(*Cat)
        cat.Quack()
    }
}
```

如果禁用编译器优化，上述代码会在类型断言时就不是直接获取变量中具体类型的 `_type`，而是从 `eface._type` 中获取，汇编指令仍然会使用目标类型的 `hash` 字段与变量的类型比较。



### 4.2.5 动态派发

动态派发（Dynamic dispatch）是在运行期间选择具体多态操作（方法或者函数）执行的过程，它是一种在面向对象语言中常见的特性[6](https://www.bookstack.cn/read/draveness-golang/5833e4cb4df6241b.md#fn:6)。

Go 语言虽然不是严格意义上的面向对象语言，但是接口的引入为它带来了动态派发这一特性，调用接口类型的方法时，如果编译期间不能确认接口的类型，Go 语言会在运行期间决定具体调用该方法的哪个实现。

在如下所示的代码中，`main` 函数调用了两次 `Quack` 方法：

- 第一次以 `Duck` 接口类型的身份调用，调用时需要经过运行时的动态派发；
- 第二次以 `*Cat` 具体类型的身份调用，编译期就会确定调用的函数：

```go
func main() {
    var c Duck = &Cat{Name: "grooming"}
    c.Quack()
    c.(*Cat).Quack()
}
```

因为编译器优化影响了我们对原始汇编指令的理解，所以需要使用编译参数 `-N` 关闭编译器优化。如果不指定这个参数，编译器会对代码进行重写，与最初生成的执行过程有一些偏差，例如：

- 因为接口类型中的 `tab` 参数并没有被使用，所以优化从 `Cat` 转换到 `Duck` 的过程；
- 因为变量的具体类型是确定的，所以删除从 `Duck` 接口类型转换到 `*Cat` 具体类型时可能会发生 `panic` 的分支；
- …

在具体分析调用 `Quack` 方法的两种姿势之前，我们要先了解 `Cat` 结构体究竟是如何初始化的，以及初始化完成后的栈上有哪些数据：

```
LEAQ    type."".Cat(SB), AX                
MOVQ    AX, (SP)
CALL    runtime.newobject(SB)              ;; SP + 8 = new(Cat)
MOVQ    8(SP), DI                          ;; DI = SP + 8
MOVQ    DI, ""..autotmp_2+32(SP)           ;; SP + 32 = DI
MOVQ    $8, 8(DI)                          ;; StringHeader(cat).Len = 8
LEAQ    go.string."grooming"(SB), AX       ;; AX = &"grooming"
MOVQ    AX, (DI)                           ;; StringHeader(cat).Data = AX
MOVQ    ""..autotmp_2+32(SP), AX           ;; AX = &Cat{...}
MOVQ    AX, ""..autotmp_1+40(SP)           ;; SP + 40 = &Cat{...}
LEAQ    go.itab.*"".Cat,"".Duck(SB), CX    ;; CX = &go.itab.*"".Cat,"".Duck
MOVQ    CX, "".c+48(SP)                    ;; iface(c).tab = SP + 48 = CX
MOVQ    AX, "".c+56(SP)                    ;; iface(c).data = SP + 56 = AX
```

这段代码的初始化过程其实和上两节中的过程没有太多的差别，它先初始化了 `Cat` 结构体指针，再将 `Cat` 和 `tab` 打包成了一个 `iface` 类型的结构体，我们直接来看初始化结束后的栈情况：

![stack-after-initialize](go语言设计与实现.assets/dc6d51f3cdcc2ad94464e7f882f75864.png)

**图 4-14 接口类型初始化后的栈**

- SP 是 `Cat` 类型，它也是运行时 [`runtime.newobject`](https://github.com/golang/go/blob/5042317d6919d4c84557e04be35130430e8d1dd4/src/runtime/malloc.go#L1162-L1164) 方法的参数；
- SP+8 是 [`runtime.newobject`](https://github.com/golang/go/blob/5042317d6919d4c84557e04be35130430e8d1dd4/src/runtime/malloc.go#L1162-L1164) 方法的返回值，也就是指向堆上的 `Cat` 结构体的指针；
- SP+32、SP+40 是对 SP+8 的拷贝，这两个指针都会指向栈上的 `Cat` 结构体；
- SP+48 ~ SP+64 是接口变量 `iface` 结构体，其中包含了 `tab` 结构体指针和 `*Cat` 指针；

初始化过程结束之后，我们进入到了动态派发的过程，`c.Quack()` 语句展开的汇编指令会在运行时确定函数指针。

```
MOVQ    "".c+48(SP), AX                    ;; AX = iface(c).tab
MOVQ    24(AX), AX                         ;; AX = iface(c).tab.fun[0] = Cat.Quack
MOVQ    "".c+56(SP), CX                    ;; CX = iface(c).data
MOVQ    CX, (SP)                           ;; SP = CX = &Cat{...}
CALL    AX                                 ;; SP.Quack()
```



这段代码的执行过程可以分成以下三个步骤：

- 从接口变量中获取了保存 `Cat.Quack` 方法指针的 `tab.func[0]`；
- 接口变量在中的数据会被拷贝到栈顶；
- 方法指针会被拷贝到寄存器中并通过汇编指令 `CALL` 触发：另一个调用 `Quack` 方法的语句 `c.(*Cat).Quack()*` *生成的汇编指令看起来会有一些复杂，但是代码前半部分都是在做类型转换，将接口类型转换成 ``*`Cat` 类型，只有最后两行代码才是函数调用相关的指令：

```
MOVQ    "".c+56(SP), AX                    ;; AX = iface(c).data = &Cat{...}
MOVQ    "".c+48(SP), CX                    ;; CX = iface(c).tab
LEAQ    go.itab.*"".Cat,"".Duck(SB), DX    ;; DX = &&go.itab.*"".Cat,"".Duck
CMPQ    CX, DX                             ;; CMP(CX, DX)
JEQ    163
JMP    201
MOVQ    AX, ""..autotmp_3+24(SP)           ;; SP+24 = &Cat{...}
MOVQ    AX, (SP)                           ;; SP = &Cat{...}
CALL    "".(*Cat).Quack(SB)                ;; SP.Quack()
```

下面的几行代码只是将 `Cat` 指针拷贝到了栈顶并调用 `Quack` 方法。这一次调用的函数指针在编译期就已经确定了，所以运行时就不需要动态查找方法的实现：

```
MOVQ    "".c+48(SP), AX                    ;; AX = iface(c).tab
MOVQ    24(AX), AX                         ;; AX = iface(c).tab.fun[0] = Cat.Quack
MOVQ    "".c+56(SP), CX                    ;; CX = iface(c).data
```

#### 基准测试

下面代码中的两个方法 `BenchmarkDirectCall` 和 `BenchmarkDynamicDispatch` 分别会调用结构体方法和接口方法，在接口上调用方法时会使用动态派发机制，我们以直接调用作为基准分析动态派发带来了多少额外开销：

```go
func BenchmarkDirectCall(b *testing.B) {
    c := &Cat{Name: "grooming"}
    for n := 0; n < b.N; n++ {
        // MOVQ    AX, "".c+24(SP)
        // MOVQ    AX, (SP)
        // CALL    "".(*Cat).Quack(SB)
        c.Quack()
    }
}
func BenchmarkDynamicDispatch(b *testing.B) {
    c := Duck(&Cat{Name: "grooming"})
    for n := 0; n < b.N; n++ {
        // MOVQ    "".d+56(SP), AX
        // MOVQ    24(AX), AX
        // MOVQ    "".d+64(SP), CX
        // MOVQ    CX, (SP)
        // CALL    AX
        c.Quack()
    }
}
```

我们直接运行下面的命令，使用 1 个 CPU 运行上述代码，每一个基准测试都会被执行 3 次：

```
$ go test -gcflags=-N -benchmem -test.count=3 -test.cpu=1 -test.benchtime=1s -bench=.
goos: darwin
goarch: amd64
pkg: github.com/golang/playground
BenchmarkDirectCall          500000000             3.11 ns/op           0 B/op           0 allocs/op
BenchmarkDirectCall          500000000             2.94 ns/op           0 B/op           0 allocs/op
BenchmarkDirectCall          500000000             3.04 ns/op           0 B/op           0 allocs/op
BenchmarkDynamicDispatch     500000000             3.40 ns/op           0 B/op           0 allocs/op
BenchmarkDynamicDispatch     500000000             3.79 ns/op           0 B/op           0 allocs/op
BenchmarkDynamicDispatch     500000000             3.55 ns/op           0 B/op           0 allocs/op
```

- 调用结构体方法时，每一次调用需要 `~3.03ns`；
- 使用动态派发时，每一调用需要 `~3.58ns`；

在关闭编译器优化的情况下，从上面的数据来看，动态派发生成的指令会带来 `~18%` 左右的额外性能开销。

这些性能开销在一个复杂的系统中不会带来太多的影响。一个项目不可能只使用动态派发，而且如果我们开启编译器优化后，动态派发的额外开销会降低至 `~5%`，这对应用性能的整体影响就更小了，所以与使用接口带来的好处相比，动态派发的额外开销往往可以忽略。

上面的性能测试建立在实现和调用方法的都是结构体指针上，当我们将结构体指针换成结构体又会有比较大的差异：

```go
func BenchmarkDirectCall(b *testing.B) {
    c := Cat{Name: "grooming"}
    for n := 0; n < b.N; n++ {
        // MOVQ    AX, (SP)
        // MOVQ    $8, 8(SP)
        // CALL    "".Cat.Quack(SB)
        c.Quack()
    }
}
func BenchmarkDynamicDispatch(b *testing.B) {
    c := Duck(Cat{Name: "grooming"})
    for n := 0; n < b.N; n++ {
        // MOVQ    16(SP), AX
        // MOVQ    24(SP), CX
        // MOVQ    AX, "".d+32(SP)
        // MOVQ    CX, "".d+40(SP)
        // MOVQ    "".d+32(SP), AX
        // MOVQ    24(AX), AX
        // MOVQ    "".d+40(SP), CX
        // MOVQ    CX, (SP)
        // CALL    AX
        c.Quack()
    }
}
```

我们重新执行相同的基准测试时，会得到如下所示的结果：

```
$ go test -gcflags=-N -benchmem -test.count=3 -test.cpu=1 -test.benchtime=1s .
goos: darwin
goarch: amd64
pkg: github.com/golang/playground
BenchmarkDirectCall          500000000             3.15 ns/op           0 B/op           0 allocs/op
BenchmarkDirectCall          500000000             3.02 ns/op           0 B/op           0 allocs/op
BenchmarkDirectCall          500000000             3.09 ns/op           0 B/op           0 allocs/op
BenchmarkDynamicDispatch     200000000             6.92 ns/op           0 B/op           0 allocs/op
BenchmarkDynamicDispatch     200000000             6.91 ns/op           0 B/op           0 allocs/op
BenchmarkDynamicDispatch     200000000             7.10 ns/op           0 B/op           0 allocs/op
```

直接调用方法需要消耗时间的平均值和使用指针实现接口时差不多，约为 `~3.09ns`，而使用动态派发调用方法却需要 `~6.98ns` 相比直接调用额外消耗了 `~125%` 的时间，从生成的汇编指令我们也能看出后者的额外开销会高很多。

|        | 直接调用 | 动态派发 |
| :----- | :------- | :------- |
| 指针   | ~3.03ns  | ~3.58ns  |
| 结构体 | ~3.09ns  | ~6.98ns  |

从上述表格我们可以看到使用结构体来实现接口带来的开销会大于使用指针实现，而动态派发在结构体上的表现非常差，这也提醒我们应当尽量避免使用结构体类型实现接口。

使用结构体带来的巨大性能差异不只是接口带来的问题，带来性能问题主要因为 Go 语言在函数调用时是传值的，动态派发的过程只是放大了参数拷贝带来的影响。



### 4.2.6 小结

重新回顾一下本节介绍的内容，我们在开头简单介绍了使用 Go 语言接口的常见问题，例如使用不同类型实现接口带来的差异、函数调用时发生的隐式类型转换；我们还分析了接口的类型转换、类型断言以及动态派发机制。相信这一节中内容能够帮助各位读者深入理解 Go 语言的接口。

------

- Wikepedia: Interface (computing) [https://en.wikipedia.org/wiki/Interface_(computing)](https://en.wikipedia.org/wiki/Interface_(computing))[↩︎](https://www.bookstack.cn/read/draveness-golang/5833e4cb4df6241b.md#fnref:1)
- Wikipedia: POSIX https://en.wikipedia.org/wiki/POSIX[↩︎](https://www.bookstack.cn/read/draveness-golang/5833e4cb4df6241b.md#fnref:2)
- 计算机程序的构造和解释，Harold Abelson / Gerald Jay Sussman / Julie Sussman https://book.douban.com/subject/1148282/[↩︎](https://www.bookstack.cn/read/draveness-golang/5833e4cb4df6241b.md#fnref:3)
- cmd/compile,runtime: generate hash functions only for types which are map keys https://github.com/golang/go/commit/36f30ba289e31df033d100b2adb4eaf557f05a34[↩︎](https://www.bookstack.cn/read/draveness-golang/5833e4cb4df6241b.md#fnref:4)
- Go’s hidden #pragmas https://dave.cheney.net/2018/01/08/gos-hidden-pragmas[↩︎](https://www.bookstack.cn/read/draveness-golang/5833e4cb4df6241b.md#fnref:5)
- Wikipedia: Dynamic Dispatch https://en.wikipedia.org/wiki/Dynamic_dispatch[↩︎](https://www.bookstack.cn/read/draveness-golang/5833e4cb4df6241b.md#fnref:6



## 4.3 反射

> [4.3 反射](https://www.bookstack.cn/read/draveness-golang/f42669fa0cd10c64.md)

反射是 Go 语言比较重要的特性。虽然在大多数的应用和服务中并不常见，

但是**很多框架都依赖 Go 语言的反射机制实现简化代码的逻辑**。

因为 Go 语言的语法元素很少、设计简单，所以它没有特别强的表达能力，但是 Go 语言的 [`reflect`](https://golang.org/pkg/reflect/) 包能够弥补它在语法上的一些劣势。

[`reflect`](https://golang.org/pkg/reflect/) 实现了**运行时的反射能力**，能够让程序操作不同类型的对象[1](https://www.bookstack.cn/read/draveness-golang/f42669fa0cd10c64.md#fn:1)。

反射包中有两对非常重要的函数和类型，[`reflect.TypeOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/type.go#L1365-L1368) 能获取类型信息，[`reflect.ValueOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L2316-L2328) 能获取数据的运行时表示，另外两个类型是 `Type` 和 `Value`，它们与函数是一一对应的关系：

![golang-reflection](go语言设计与实现.assets/4a214c760b4fe93e9686b08d2f816a92.png)

**图 4-15 反射函数和类型**

### 基本类型

#### Type

类型 `Type` 是反射包定义的一个接口，我们可以使用 [`reflect.TypeOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/type.go#L1365-L1368) 函数获取任意变量的的类型，`Type` 接口中定义了一些有趣的方法，`MethodByName` 可以获取当前类型对应方法的引用、`Implements` 可以判断当前类型是否实现了某个接口：

```go
type Type interface {
        Align() int
        FieldAlign() int
        Field(i int) StructField
        Method(int) Method
        MethodByName(string) (Method, bool)
        NumMethod() int
        ...
        Implements(u Type) bool
        ...
}
```



t.Kind()会返回基础类型， **`v.Type()`：完整类型信息**

```
func main() {
	type MyInt int
	var x MyInt = 42

	v := reflect.ValueOf(x)
	fmt.Println(v.Kind()) // 输出: reflect.Int（不是 MyInt）
	fmt.Println(v.Type()) // 输出: main.MyInt（完整类型名）
}

```





#### Value

反射包中 `Value` 的类型与 `Type` 不同，它被声明成了结构体。这个结构体没有对外暴露的字段，但是提供了获取或者写入数据的方法：

```go
type Value struct {
        // contains filtered or unexported fields
}
func (v Value) Addr() Value
func (v Value) Bool() bool
func (v Value) Bytes() []byte
...
```

反射包中的所有方法基本都是围绕着 `Type` 和 `Value` 这两个类型设计的。

我们通过 [`reflect.TypeOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/type.go#L1365-L1368)、[`reflect.ValueOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L2316-L2328) 可以将一个普通的变量转换成『反射』包中提供的 `Type` 和 `Value`，随后就可以使用反射包中的方法对它们进行复杂的操作。

### 4.3.1 三大法则

运行时反射是程序在运行期间检查其自身结构的一种方式。

反射带来的灵活性是一把双刃剑，**反射作为一种元编程方式可以减少重复代码**[2](https://www.bookstack.cn/read/draveness-golang/f42669fa0cd10c64.md#fn:2)，**但是过量的使用反射会使我们的程序逻辑变得难以理解并且运行缓慢**。

我们在这一节中会介绍 Go 语言反射的三大法则[3](https://www.bookstack.cn/read/draveness-golang/f42669fa0cd10c64.md#fn:3)，其中包括：

- 从 `interface{}` 变量可以反射出反射对象；
- 从反射对象可以获取 `interface{}` 变量；
- 要修改反射对象，其值必须可设置；



#### 第一法则

反射的第一法则是我们能将 Go 语言的 `interface{}` 变量转换成反射对象。

很多读者可能会对这以法则产生困惑 —— 为什么是从 `interface{}` 变量到反射对象？

当我们执行 `reflect.ValueOf(1)` 时，虽然看起来是获取了基本类型 `int` 对应的反射类型，但是由于 [`reflect.TypeOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/type.go#L1365-L1368)、[`reflect.ValueOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L2316-L2328) 两个方法的入参都是 `interface{}` 类型，所以在方法执行的过程中发生了**类型转换**。

在[函数调用](http://draveness.me/golang-function-call)一节中曾经介绍过，**Go 语言的函数调用都是值传递的，变量会在函数调用时进行类型转换**。

基本类型 `int` 会转换成 `interface{}` 类型，这也就是为什么第一条法则是『从接口到反射对象』。

上面提到的 [`reflect.TypeOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/type.go#L1365-L1368) 和 [`reflect.ValueOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L2316-L2328) 函数就能完成这里的转换，如果我们认为 Go 语言的类型（Interface Value）和反射类型（Reflection Object）处于两个不同的『世界』，那么这两个函数就是连接这两个世界的桥梁。

![golang-interface-to-reflection](go语言设计与实现.assets/da1f02b0b3444f217cb198667e785dc7.png)

**图 4-16 接口到反射对象**

我们通过以下例子简单介绍这两个函数的作用，[`reflect.TypeOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/type.go#L1365-L1368) 获取了变量 `author` 的类型，[`reflect.ValueOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L2316-L2328) 获取了变量的值 `draven`。

如果我们知道了一个变量的类型和值，那么就意味着知道了这个变量的全部信息。

```go
package main
import (
    "fmt"
    "reflect"
)
func main() {
    author := "draven"
    fmt.Println("TypeOf author:", reflect.TypeOf(author))
    fmt.Println("ValueOf author:", reflect.ValueOf(author))
}
$ go run main.go
TypeOf author: string
ValueOf author: draven
```

有了变量的类型之后，我们可以通过 `Method` 方法获得类型实现的方法，通过 `Field` 获取类型包含的全部字段。

对于不同的类型，我们也可以调用不同的方法获取相关信息：

- 结构体：获取字段的数量并通过下标和字段名获取字段 `StructField`；
- 哈希表：获取哈希表的 `Key` 类型；
- 函数或方法：获取入参和返回值的类型；
- …

总而言之，使用 [`reflect.TypeOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/type.go#L1365-L1368) 和 [`reflect.ValueOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L2316-L2328) 能够获取 Go 语言中的变量对应的反射对象。一旦获取了反射对象，我们就能得到跟当前类型相关数据和操作，并可以使用这些运行时获取的结构执行方法。

示例：

```go
package main

import (
	"fmt"
	"reflect"
)

type Person struct {
	Name string
	Age  int
}

func main() {
	// 结构体示例
	p := Person{Name: "Alice", Age: 30}

	// 获取结构体字段数量
	fmt.Println("Number of fields in Person struct:", reflect.TypeOf(p).NumField())

	// 获取结构体字段信息
	for i := 0; i < reflect.TypeOf(p).NumField(); i++ {
		field := reflect.TypeOf(p).Field(i)
		fmt.Printf("Field %d: Name=%s, Type=%v\n", i, field.Name, field.Type)
	}

	// 哈希表示例
	m := map[string]int{"a": 1, "b": 2}

	// 获取哈希表的键类型
	fmt.Println("Key type of map:", reflect.TypeOf(m).Key())

	// 函数示例
	add := func(a, b int) int {
		return a + b
	}

	// 获取函数的参数和返回值类型
	funcType := reflect.TypeOf(add)
	fmt.Println("Number of input parameters:", funcType.NumIn())
	for i := 0; i < funcType.NumIn(); i++ {
		fmt.Printf("Input parameter %d type: %v\n", i, funcType.In(i))
	}
	fmt.Println("Number of output parameters:", funcType.NumOut())
	for i := 0; i < funcType.NumOut(); i++ {
		fmt.Printf("Output parameter %d type: %v\n", i, funcType.Out(i))
	}
}

==》
Number of fields in Person struct: 2
Field 0: Name=Name, Type=string
Field 1: Name=Age, Type=int
Key type of map: string
Number of input parameters: 2
Input parameter 0 type: int
Input parameter 1 type: int
Number of output parameters: 1
Output parameter 0 type: int
```



#### 第二法则

反射的第二法则是我们可以从反射对象可以获取 `interface{}` 变量。

既然能够将接口类型的变量转换成反射对象，那么一定需要其他方法将反射对象还原成接口类型的变量，[`reflect`](https://golang.org/pkg/reflect/) 中的 [`reflect.Value.Interface`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L992-L994) 方法就能完成这项工作：

![golang-reflection-to-interface](go语言设计与实现.assets/ac99061205fcced9b3a1221a5aca28da.png)

**图 4-17 反射对象到接口**

不过调用 [`reflect.Value.Interface`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L992-L994) 方法只能获得 `interface{}` 类型的变量，如果想要将其还原成最原始的状态还需要经过如下所示的**显式类型转换**：

```go
v := reflect.ValueOf(1)
v.Interface().(int)
```

从反射对象到接口值的过程就是从接口值到反射对象的镜面过程，两个过程都需要经历两次转换：

- 从接口值到反射对象：
  - 从基本类型到接口类型的类型转换；
  - 从接口类型到反射对象的转换；
- 从反射对象到接口值：
  - 反射对象转换成接口类型；
  - 通过显式类型转换变成原始类型；

![golang-bidirectional-reflection](go语言设计与实现.assets/3e5562c7e5cfa4f9a5e737d19a95bd91.png)

**图 4-18 接口和反射对象的双向转换（bidirectional）**

当然不是所有的变量都需要类型转换这一过程。如果变量本身就是 `interface{}` 类型，那么它不需要类型转换，因为类型转换这一过程一般都是隐式的，所以我不太需要关心它，只有在我们需要将反射对象转换回基本类型时才需要显式的转换操作。



#### 第三法则

Go 语言反射的最后一条法则是与值是否可以被更改有关，如果我们想要更新一个 `reflect.Value`，那么它持有的值一定是可以被更新的，假设我们有以下代码：

```go
func main() {
    i := 1
    v := reflect.ValueOf(i)
    v.SetInt(10)
    fmt.Println(i)
}
$ go run reflect.go
panic: reflect: reflect.flag.mustBeAssignable using unaddressable value
goroutine 1 [running]:
reflect.flag.mustBeAssignableSlow(0x82, 0x1014c0)
    /usr/local/go/src/reflect/value.go:247 +0x180
reflect.flag.mustBeAssignable(...)
    /usr/local/go/src/reflect/value.go:234
reflect.Value.SetInt(0x100dc0, 0x414020, 0x82, 0x1840, 0xa, 0x0)
    /usr/local/go/src/reflect/value.go:1606 +0x40
main.main()
    /tmp/sandbox590309925/prog.go:11 +0xe0
```

运行上述代码会导致程序崩溃并报出 `reflect: reflect.flag.mustBeAssignable using unaddressable value` 错误，仔细思考一下就能够发现出错的原因，Go 语言的[函数调用](http://draveness.me/golang-function-call)都是**传值的**，所以我们得到的反射对象跟最开始的变量没有任何关系，所以直接对它修改会导致崩溃。

想要修改原有的变量只能通过如下的方法（地址传值+Elem（））：

```
func main() {
    i := 1
    v := reflect.ValueOf(&i)
    v.Elem().SetInt(10)
    fmt.Println(i)
}
$ go run reflect.go
10
```

- 调用 [`reflect.ValueOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L2316-L2328) 函数获取变量指针；
- 调用 [`reflect.Value.Elem`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L788-L821) 方法获取指针指向的变量；
- 调用 [`reflect.Value.SetInt`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L1600-L1616) 方法更新变量的值：由于 Go 语言的函数调用都是值传递的，所以我们只能先获取指针对应的 `reflect.Value`，再通过 [`reflect.Value.Elem`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L788-L821) 方法迂回的方式得到可以被设置的变量，我们通过如下所示的代码理解这个过程：

```
func main() {
    i := 1
    v := &i
    *v = 10
}
```

如果不能直接操作 `i` 变量修改其持有的值，我们就只能获取 `i` 变量所在地址并使用 `*v` 修改所在地址中存储的整数。



### 4.3.2 类型和值

Go 语言的 `interface{}` 类型在语言内部是通过 `emptyInterface` 这个结体来表示的，其中的 `rtype` 字段用于表示变量的类型，另一个 `word` 字段指向内部封装的数据：

```go
type emptyInterface struct {
    typ  *rtype
    word unsafe.Pointer
}
```

用于获取变量类型的 [`reflect.TypeOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/type.go#L1365-L1368) 函数将传入的变量隐式转换成 `emptyInterface` 类型并获取其中存储的类型信息 `rtype`：

```go
func TypeOf(i interface{}) Type {
    eface := *(*emptyInterface)(unsafe.Pointer(&i))
    return toType(eface.typ)
}
func toType(t *rtype) Type {
    if t == nil {
        return nil
    }
    return t
}
```

[`reflect.TypeOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/type.go#L1365-L1368) 函数的实现原理其实并不复杂，它只是将一个 `interface{}` 变量转换成了内部的 `emptyInterface` 表示，然后从中获取相应的类型信息。

用于获取接口值 `Value` 的函数 [`reflect.ValueOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L2316-L2328) 实现也非常简单，在该函数中我们先调用了 [`reflect.escapes`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/value.go#L2779-L2783) 函数保证当前值逃逸到堆上，然后通过 [`reflect.unpackEface`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/value.go#L140-L152) 方法从接口中获取 `Value` 结构体：

```go
func ValueOf(i interface{}) Value {
    if i == nil {
        return Value{}
    }
    escapes(i)
    return unpackEface(i)
}
func unpackEface(i interface{}) Value {
    e := (*emptyInterface)(unsafe.Pointer(&i))
    t := e.typ
    if t == nil {
        return Value{}
    }
    f := flag(t.Kind())
    if ifaceIndir(t) {
        f |= flagIndir
    }
    return Value{t, e.word, f}
}
```

[`reflect.unpackEface`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/value.go#L140-L152) 函数会将传入的接口转换成 `emptyInterface` 结构体，然后将具体类型和指针包装成 `Value` 结构体并返回。

[`reflect.TypeOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/type.go#L1365-L1368) 和 [`reflect.ValueOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L2316-L2328) 函数的实现都很简单。我们已经分析了这两个函数的实现，现在需要了解**编译器在调用函数之前做了哪些工作**：

```go
package main
import (
    "reflect"
)
func main() {
    i := 20
    _ = reflect.TypeOf(i)
}
$ go build -gcflags="-S -N" main.go
...
MOVQ    $20, ""..autotmp_20+56(SP) // autotmp = 20
LEAQ    type.int(SB), AX           // AX = type.int(SB)
MOVQ    AX, ""..autotmp_19+280(SP) // autotmp_19+280(SP) = type.int(SB)
LEAQ    ""..autotmp_20+56(SP), CX  // CX = 20
MOVQ    CX, ""..autotmp_19+288(SP) // autotmp_19+288(SP) = 20
...
```

从上面这段截取的汇编语言，我们发现在函数调用之前已经发生了类型转换，上述指令将 `int` 类型的变量转换成了占用 16 字节 `autotmp_19+280(SP) ~ autotmp_19+288(SP)` 的接口，两个 `LEAQ` 指令分别获取了类型的指针 `type.int(SB)` 以及变量 `i` 所在的地址。

当我们想要将一个变量转换成反射对象时，Go 语言会在编译期间完成类型转换的工作，将变量的类型和值转换成了 `interface{}` 并等待运行期间使用 [`reflect`](https://golang.org/pkg/reflect/) 包获取接口中存储的信息。





### 4.3.3 更新变量

当我们想要更新一个 `reflect.Value`，就需要调用 [`reflect.Value.Set`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/value.go#L1525-L1538) 方法更新反射对象，该方法会调用 [`reflect.flag.mustBeAssignable`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/value.go#L232-L236) 和 [`reflect.flag.mustBeExported`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/value.go#L214-L218) 分别检查当前反射对象是否是可以被设置的以及字段是否是对外公开的：

```go
func (v Value) Set(x Value) {
    v.mustBeAssignable()
    x.mustBeExported()
    var target unsafe.Pointer
    if v.kind() == Interface {
        target = v.ptr
    }
    x = x.assignTo("reflect.Set", v.typ, target)
    typedmemmove(v.typ, v.ptr, x.ptr)
}
```

[`reflect.Value.Set`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/value.go#L1525-L1538) 方法会调用 [`reflect.Value.assignTo`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/value.go#L2370-L2404) 并返回一个新的反射对象，**这个返回的反射对象指针就会直接覆盖原始的反射变量**。

```go
func (v Value) assignTo(context string, dst *rtype, target unsafe.Pointer) Value {
    ...
    switch {
    case directlyAssignable(dst, v.typ):
        ...
        return Value{dst, v.ptr, fl}
    case implements(dst, v.typ):
        if v.Kind() == Interface && v.IsNil() {
            return Value{dst, nil, flag(Interface)}
        }
        x := valueInterface(v, false)
        if dst.NumMethod() == 0 {
            *(*interface{})(target) = x
        } else {
            ifaceE2I(dst, x, target)
        }
        return Value{dst, target, flagIndir | flag(Interface)}
    }
    panic(context + ": value of type " + v.typ.String() + " is not assignable to type " + dst.String())
}
```

[`reflect.Value.assignTo`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/value.go#L2370-L2404) 会根据当前和被设置的反射对象类型创建一个新的 `Value` 结构体：

- 如果两个反射对象的类型是可以被直接替换，就会直接将目标反射对象返回；
- 如果当前反射对象是接口并且目标对象实现了接口，就会将目标对象简单包装成接口值；

在变量更新的过程中，[`reflect.Value.assignTo`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/value.go#L2370-L2404) 返回的 `reflect.Value` 中的指针会覆盖当前反射对象中的指针实现变量的更新。

示例：

```go
func main() {
    i := 1
    v := reflect.ValueOf(&i)
    v.Elem().SetInt(10)
    fmt.Println(i)
}
$ go run reflect.go
10
```





### 4.3.4 实现协议（implements）

[`reflect`](https://golang.org/pkg/reflect/) 包还为我们提供了 [`reflect.rtypes.Implements`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/type.go#L1430-L1438) 方法可以用于判断某些类型是否遵循特定的接口。在 Go 语言中获取结构体的反射类型 `reflect.Type` 还是比较容易的，但是想要获得接口的类型就需要通过以下方式：

```go
reflect.TypeOf((*<interface>)(nil)).Elem()
```

我们通过一个例子在介绍如何判断一个类型是否实现了某个接口。假设我们需要判断如下代码中的 `CustomError` 是否实现了 Go 语言标准库中的 `error` 接口：

```go
type CustomError struct{}
func (*CustomError) Error() string {
    return ""
}
func main() {
    typeOfError := reflect.TypeOf((*error)(nil)).Elem()
    customErrorPtr := reflect.TypeOf(&CustomError{})
    customError := reflect.TypeOf(CustomError{})
    fmt.Println(customErrorPtr.Implements(typeOfError)) // #=> true
    fmt.Println(customError.Implements(typeOfError)) // #=> false
}
```

上述代码的运行结果正如我们在[接口](https://www.bookstack.cn/read/draveness-golang/5833e4cb4df6241b.md)一节中介绍的：

- `CustomError` 类型并没有实现 `error` 接口；
- `*CustomError` 指针类型实现了 `error` 接口；

抛开上述的执行结果不谈，我们来分析一下 [`reflect.rtypes.Implements`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/type.go#L1430-L1438) 方法的工作原理：

```go
func (t *rtype) Implements(u Type) bool {
    if u == nil {
        panic("reflect: nil type passed to Type.Implements")
    }
    if u.Kind() != Interface {
        panic("reflect: non-interface type passed to Type.Implements")
    }
    return implements(u.(*rtype), t)
}
```



[`reflect.rtypes.Implements`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/type.go#L1430-L1438) 方法会检查传入的类型是不是接口，如果不是接口或者是空值就会直接 panic 中止当前程序。

在参数没有问题的情况下，上述方法会调用私有函数 [`reflect.implements`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/type.go#L1461-L1543) 判断类型之间是否有实现关系：

```go
func implements(T, V *rtype) bool {
    t := (*interfaceType)(unsafe.Pointer(T))
    if len(t.methods) == 0 {
        return true
    }
    ...
    v := V.uncommon()
    i := 0
    vmethods := v.methods()
    for j := 0; j < int(v.mcount); j++ {
        tm := &t.methods[i]
        tmName := t.nameOff(tm.name)
        vm := vmethods[j]
        vmName := V.nameOff(vm.name)
        if vmName.name() == tmName.name() && V.typeOff(vm.mtyp) == t.typeOff(tm.typ) {
            if i++; i >= len(t.methods) {
                return true
            }
        }
    }
    return false
}
```

如果接口中不包含任何方法，就意味着这是一个空的接口，任意类型都自动实现该接口，这时就会直接返回 `true`。

```
typeOfError := reflect.TypeOf((*interface{})(nil)).Elem()
```

![golang-type-implements-interface](go语言设计与实现.assets/2da86d2a51c3e091aa59c2ffd03c87f0.png)

**图 4-19 类型实现接口**

在其他情况下，由于方法都是按照字母序存储的，[`reflect.implements`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/type.go#L1461-L1543) 会维护两个用于遍历接口和类型方法的索引 `i` 和 `j` 判断类型是否实现了接口，因为最多只会进行 `n + m` 次数的比较，所以整个过程的时间复杂度是 `O(n+m)`。

### 4.3.5 方法调用（原理）

作为一门静态语言（编译后执行），如果我们想要通过 [`reflect`](https://golang.org/pkg/reflect/) 包利用反射在运行期间执行方法不是一件容易的事情，下面的十几行代码就使用反射来执行 `Add(0, 1)` 函数：

```go
func Add(a, b int) int { return a + b }
func main() {
    v := reflect.ValueOf(Add)
    if v.Kind() != reflect.Func {
        return
    }
    t := v.Type()
    argv := make([]reflect.Value, t.NumIn())
    for i := range argv {
        if t.In(i).Kind() != reflect.Int {
            return
        }
        argv[i] = reflect.ValueOf(i)
    }
    result := v.Call(argv)
    if len(result) != 1 || result[0].Kind() != reflect.Int {
        return
    }
    fmt.Println(result[0].Int()) // #=> 1
}
```

- 通过 [`reflect.ValueOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L2316-L2328) 获取函数 `Add` 对应的反射对象；
- 根据反射对象 [`reflect.rtype.NumIn`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/type.go#L979-L985) 方法返回的参数个数创建 `argv` 数组；
- 多次调用 [`reflect.ValueOf`](https://github.com/golang/go/blob/52c4488471ed52085a29e173226b3cbd2bf22b20/src/reflect/value.go#L2316-L2328) 函数逐一设置 `argv` 数组中的各个参数；
- 调用反射对象 `Add` 的 [`reflect.Value.Call`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/value.go#L318-L322) 方法并传入参数列表；
- 获取返回值数组、验证数组的长度以及类型并打印其中的数据；使用反射来调用方法非常复杂，原本只需要一行代码就能完成的工作，现在需要十几行代码才能完成，但这也是**在静态语言中使用动态特性需要付出的成本**。

```
func (v Value) Call(in []Value) []Value {
    v.mustBe(Func)
    v.mustBeExported()
    return v.call("Call", in)
}
```

[`reflect.Value.Call`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/value.go#L318-L322) 方法是运行时调用方法的入口，它通过两个 `MustBe` 开头的方法确定了当前反射对象的类型是函数以及可见性，随后调用 [`reflect.Value.call`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/value.go#L339-L501) 完成方法调用，这个私有方法的执行过程会分成以下的几个部分：

- 检查输入参数以及类型的合法性；
- 将传入的 `reflect.Value` 参数数组设置到栈上；
- 通过函数指针和输入参数调用函数；
- 从栈上获取函数的返回值；我们将按照上面的顺序分析使用 [`reflect`](https://golang.org/pkg/reflect/) 进行函数调用的几个过程。



#### 参数检查

参数检查是通过反射调用方法的第一步，在参数检查期间我们会从反射对象中取出当前的函数指针 `unsafe.Pointer`，如果该函数指针是方法，那么我们就会通过 [`reflect.methodReceiver`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/value.go#L612-L645) 函数获取方法的接受者和函数指针。

```go
func (v Value) call(op string, in []Value) []Value {
    t := (*funcType)(unsafe.Pointer(v.typ))
    ...
    if v.flag&flagMethod != 0 {
        rcvr = v
        rcvrtype, t, fn = methodReceiver(op, v, int(v.flag)>>flagMethodShift)
    } else {
        ...
    }
    n := t.NumIn()
    if len(in) < n {
        panic("reflect: Call with too few input arguments")
    }
    if len(in) > n {
        panic("reflect: Call with too many input arguments")
    }
    for i := 0; i < n; i++ {
        if xt, targ := in[i].Type(), t.In(i); !xt.AssignableTo(targ) {
            panic("reflect: " + op + " using " + xt.String() + " as type " + targ.String())
        }
    }
```

在上述方法中，上述方法还会检查传入参数的个数以及参数的类型与函数签名中的类型是否可以匹配，任何参数的不匹配都会导致整个程序的崩溃中止。

#### 准备参数

当我们已经对当前方法的参数完成验证之后，就会进入函数调用的下一个阶段，为函数调用准备参数，在前面的章节[函数调用](https://draveness.me/golang/basic/golang-function-call.html)中我们已经介绍过 Go 语言的函数调用惯例，函数或者方法在调用时，所有的参数都会被依次放置到栈上。

```go
    nout := t.NumOut()
    frametype, _, retOffset, _, framePool := funcLayout(t, rcvrtype)
    var args unsafe.Pointer
    if nout == 0 {
        args = framePool.Get().(unsafe.Pointer)
    } else {
        args = unsafe_New(frametype)
    }
    off := uintptr(0)
    if rcvrtype != nil {
        storeRcvr(rcvr, args)
        off = ptrSize
    }
    for i, v := range in {
        targ := t.In(i).(*rtype)
        a := uintptr(targ.align)
        off = (off + a - 1) &^ (a - 1)
        n := targ.size
        ...
        addr := add(args, off, "n > 0")
        v = v.assignTo("reflect.Value.Call", targ, addr)
        *(*unsafe.Pointer)(addr) = v.ptr
        off += n
    }
```

- 通过 [`reflect.funcLayout`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/type.go#L2975-L3048) 函数计算当前函数需要的参数和返回值的栈布局，也就是每一个参数和返回值所占的空间大小；
- 如果当前函数有返回值，需要为当前函数的参数和返回值分配一片内存空间 `args`；
- 如果当前函数是方法，需要向将方法的接受者拷贝到 `args` 内存中；
- 将所有函数的参数按照顺序依次拷贝到对应args 内存中
  - 使用 [`reflect.funcLayout`](https://github.com/golang/go/blob/4e8d27068df52eb372dc2ba7e929e47850934805/src/reflect/type.go#L2975-L3048) 返回的参数计算参数在内存中的位置；
  - 将参数拷贝到内存空间中；准备参数的过程是计算各个参数和返回值占用的内存空间并将所有的参数都拷贝内存空间对应的位置的过程，该过程会考虑函数和方法、返回值数量以及参数类型带来的差异。

#### 调用函数

准备好调用函数需要的全部参数之后，就会通过以下的代码执行函数指针了。我们会向该函数传入栈类型、函数指针、参数和返回值的内存空间、栈的大小以及返回值的偏移量：

```
    call(frametype, fn, args, uint32(frametype.size), uint32(retOffset))
```

上述函数实际上并不存在，它会在编译期间被链接到 [`runtime.reflectcall`](https://github.com/golang/go/blob/a38a917aee626a9b9d5ce2b93964f586bf759ea0/src/runtime/asm_386.s#L489-L526) 这个用汇编实现的函数上，我们在这里不会分析该函数的具体实现，感兴趣的读者可以自行了解其实现原理。

#### 处理返回值

当函数调用结束之后，就会开始处理函数的返回值：

- 如果函数没有任何返回值，会直接清空 `args` 中的全部内容来释放内存空间；
- 如果当前函数有返回值；
  - 将 `args` 中与输入参数有关的内存空间清空；
  - 创建一个 `nout` 长度的切片用于保存由反射对象构成的返回值数组；
  - 从函数对象中获取返回值的类型和内存大小，将 `args` 内存中的数据转换成 `reflect.Value` 类型并存储到切片中；

```go
    var ret []Value
    if nout == 0 {
        typedmemclr(frametype, args)
        framePool.Put(args)
    } else {
        typedmemclrpartial(frametype, args, 0, retOffset)
        ret = make([]Value, nout)
        off = retOffset
        for i := 0; i < nout; i++ {
            tv := t.Out(i)
            a := uintptr(tv.Align())
            off = (off + a - 1) &^ (a - 1)
            if tv.Size() != 0 {
                fl := flagIndir | flag(tv.Kind())
                ret[i] = Value{tv.common(), add(args, off, "tv.Size() != 0"), fl}
            } else {
                ret[i] = Zero(tv)
            }
            off += tv.Size()
        }
    }
    return ret
}
```

由 `reflect.Value` 构成的 `ret` 数组会被返回到上层，到这里为止使用反射实现函数调用的过程就结束了。

### 4.3.6 小结

Go 语言的 [`reflect`](https://golang.org/pkg/reflect/) 包为我们提供的多种能力，包括如何使用反射来动态修改变量、判断类型是否实现了某些接口以及动态调用方法等功能，通过对反射包中方法原理的分析能帮助我们理解之前看起来比较怪异、令人困惑的现象。

- Package reflect https://golang.org/pkg/reflect/[↩︎](https://www.bookstack.cn/read/draveness-golang/f42669fa0cd10c64.md#fnref:1)
- 谈元编程与表达能力 https://draveness.me/metaprogramming[↩︎](https://www.bookstack.cn/read/draveness-golang/f42669fa0cd10c64.md#fnref:2)
- The Laws of Reflection https://blog.golang.org/laws-of-reflection[↩︎](https://www.bookstack.cn/read/draveness-golang/f42669fa0cd10c64.md#fnref:3)







## 4.5 推荐阅读

> [4.4 推荐阅读](https://www.bookstack.cn/read/draveness-golang/82497e663992a6f7.md)

- [The Function Stack](https://www.tenouk.com/Bufferoverflowc/Bufferoverflow2a.html)
- [Why do byte spills occur and what do they achieve?](https://stackoverflow.com/questions/16453314/why-do-byte-spills-occur-and-what-do-they-achieve)
- [Friday Q&A 2011-12-16: Disassembling the Assembly, Part 1](https://mikeash.com/pyblog/friday-qa-2011-12-16-disassembling-the-assembly-part-1.html)
- [x86 calling conventions](https://en.wikipedia.org/wiki/X86_calling_conventions)
- [Call Stack](https://en.wikipedia.org/wiki/Call_stack)
- [Chapter I: A Primer on Go Assembly](https://github.com/teh-cmc/go-internals/blob/master/chapter1_assembly_primer/README.md)
- [How Interfaces Work in Go](https://www.tapirgames.com/blog/golang-interface-implementation)
- [Interfaces and other types · Effective Go](https://golang.org/doc/effective_go.html#interfaces_and_types)
- [How to use interfaces in Go](https://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go)
- [Go Data Structures: Interfaces](https://research.swtch.com/interfaces)
- [Duck typing · Wikipedia](https://en.wikipedia.org/wiki/Duck_typing)
- [What is POSIX?](http://www.robelle.com/smugbook/posix.html)
- [Chapter II: Interfaces](https://github.com/teh-cmc/go-internals/blob/master/chapter2_interfaces/README.md)
- [The Laws of Reflection](https://blog.golang.org/laws-of-reflection)
- [runtime: new itab lookup table](https://github.com/golang/go/commit/3d1699ea787f38be6088f9a098d6e08dafca9387)
- [runtime: need a better itab table](https://github.com/golang/go/issues/20505)





# 第五章 常用关键字

## 5.1 for 和 range

> [5.1 for 和 range](https://www.bookstack.cn/read/draveness-golang/816b8b0f4e15285f.md)

循环是所有编程语言都有的控制结构，除了使用经典的『三段式』循环之外，Go 语言还引入了另一个关键字 range 帮助我们快速遍历[数组](https://www.bookstack.cn/read/draveness-golang/79255565262cc9f6.md)、[切片](https://www.bookstack.cn/read/draveness-golang/bc59e924b285e5e9.md)、[哈希表](https://www.bookstack.cn/read/draveness-golang/8a6fa5746b8fbe7e.md) 以及 [Channel](https://www.bookstack.cn/read/draveness-golang/c666731d5f1a2820.md) 等集合类型。本节将深入分析 Go 语言的两种不同循环，也就是经典的 for 循环和 for/range 循环，我们会分析这两种循环的运行时结构以及它们的实现原理，

for 循环能够将代码中的数据和逻辑分离，让同一份代码能够多次复用处理同样的逻辑。我们先来看一下 Go 语言 for 循环对应的汇编代码，下面是一段经典的三段式循环的代码，我们将它编译成汇编指令：

```go
package main
func main() {
    for i := 0; i < 10; i++ {
        println(i)
    }
}
"".main STEXT size=98 args=0x0 locals=0x18
    00000 (main.go:3)    TEXT    "".main(SB), $24-0
    ...
    00029 (main.go:3)    XORL    AX, AX                   ;; i := 0
    00031 (main.go:4)    JMP    75
    00033 (main.go:4)    MOVQ    AX, "".i+8(SP)
    00038 (main.go:5)    CALL    runtime.printlock(SB)
    00043 (main.go:5)    MOVQ    "".i+8(SP), AX
    00048 (main.go:5)    MOVQ    AX, (SP)
    00052 (main.go:5)    CALL    runtime.printint(SB)
    00057 (main.go:5)    CALL    runtime.printnl(SB)
    00062 (main.go:5)    CALL    runtime.printunlock(SB)
    00067 (main.go:4)    MOVQ    "".i+8(SP), AX
    00072 (main.go:4)    INCQ    AX                       ;; i++
    00075 (main.go:4)    CMPQ    AX, $10                  ;; 比较变量 i 和 10
    00079 (main.go:4)    JLT    33                           ;; 跳转到 33 行如果 i < 10
    ...
```

我们将上述汇编指令的执行过程分成三个部分进行分析：

- 0029 ~ 0031 行负责循环的初始化；
  - 对寄存器 `AX` 中的变量 `i` 进行初始化并执行 `JMP 75` 指令跳转到 0075 行；
- 0075 ~ 0079 行负责检查循环的终止条件，将寄存器中存储的数据 i与 10 比较；
  - `JLT 33` 命令会在变量的值小于 10 时跳转到 0033 行执行循环主体；
  - `JLT 33` 命令会在变量的值大于 10 时跳出循环体执行下面的代码；
- 0033 ~ 0072 行是循环内部的语句；
  - 通过多个汇编指令打印变量中的内容；
  - `INCQ AX` 指令会将变量加一，然后再与 10 进行比较，回到第二步；for/range 循环经过优化的汇编代码有着完全相同的结构。无论是变量的初始化、循环体的执行还是最后的条件判断都是完全一样的，所以这里也就不展开分析对应的汇编指令了。

```go
package main
func main() {
    arr := []int{1, 2, 3}
    for i, _ := range arr {
        println(i)
    }
}
```

在汇编语言中，无论是经典的 for 循环还是 for/range 循环都会使用 **`JMP`** 以及相关的命令跳回循环体的开始位置来多次执行代码的逻辑。从不同循环具有相同的汇编代码可以猜到，使用 for/range 的控制结构最终也会被 Go 语言编译器转换成普通的 for 循环，后面的分析会印证这一点。



### 5.1.1 现象

在深入语言的源代码中了解两种不同循环的实现之前，我们可以先来看一下使用 `for` 和 `range` 会遇到的一些现象和问题，我们可以带着这些现象和问题去源代码中寻找答案，这样能更高效地理解实现。

#### 循环永动机

如果我们在遍历数组的同时修改数组的元素，能否得到一个永远都不会停止的循环呢？你可以自己尝试运行下面的代码来得到结果：

```go
func main() {
    arr := []int{1, 2, 3}
    for _, v := range arr {
        arr = append(arr, v)
    }
    fmt.Println(arr)
}
$ go run main.go
1 2 3 1 2 3
```



上述代码的输出意味着循环只遍历了原始切片中的三个元素，我们在遍历切片时追加的元素不会增加循环的执行次数，所以循环最终还是停了下来。

#### 神奇的指针

第二个例子是使用 Go 语言经常会犯的错误[1](https://www.bookstack.cn/read/draveness-golang/816b8b0f4e15285f.md#fn:1)。当我们在遍历一个数组时，如果获取 `range` 返回变量的地址并保存到另一个数组或者哈希时，就会遇到令人困惑的现象：

```go
func main() {
    arr := []int{1, 2, 3}
    newArr := []*int{}
    for _, v := range arr {
        newArr = append(newArr, &v)
    }
    for _, v := range newArr {
        fmt.Println(*v)
    }
}
$ go run main.go
3 3 3
```

上述代码最终会输出三个连续的 `3`，这个问题比较常见，一些有经验的开发者不经意也会犯这种错误，正确的做法应该是使用 `&arr[i]` 替代 `&v`，我们会在下面分析这一现象背后的原因。（因为newArr是一个地址数组，而遍历是v的地址是不变的，所以最后其实存的是同个地址，且最后地址上的值是3）



#### 遍历清空数组

当我们想要在 Go 语言中清空一个切片或者哈希表时，我们一般都会使用以下的方法将切片中的元素置零，但是依次去遍历切片和哈希表看起来是非常耗费性能的事情：

```go
func main() {
    arr := []int{1, 2, 3}
    for i, _ := range arr {
        arr[i] = 0
    }
}
```

因为数组、切片和哈希表占用的内存空间都是连续的，所以最快的方法是直接清空这片内存中的内容，当我们编译上述代码时会得到以下的汇编指令：

```
"".main STEXT size=93 args=0x0 locals=0x30
    0x0000 00000 (main.go:3)    TEXT    "".main(SB), $48-0
    ...
    0x001d 00029 (main.go:4)    MOVQ    "".statictmp_0(SB), AX
    0x0024 00036 (main.go:4)    MOVQ    AX, ""..autotmp_3+16(SP)
    0x0029 00041 (main.go:4)    MOVUPS    "".statictmp_0+8(SB), X0
    0x0030 00048 (main.go:4)    MOVUPS    X0, ""..autotmp_3+24(SP)
    0x0035 00053 (main.go:5)    PCDATA    $2, $1
    0x0035 00053 (main.go:5)    LEAQ    ""..autotmp_3+16(SP), AX
    0x003a 00058 (main.go:5)    PCDATA    $2, $0
    0x003a 00058 (main.go:5)    MOVQ    AX, (SP)
    0x003e 00062 (main.go:5)    MOVQ    $24, 8(SP)
    0x0047 00071 (main.go:5)    CALL    runtime.memclrNoHeapPointers(SB)
    ...
```

从生成的汇编代码我们可以看出，编译器会直接使用 [`runtime.memclrNoHeapPointers`](https://github.com/golang/go/blob/05c02444eb2d8b8d3ecd949c4308d8e2323ae087/src/runtime/memclr_386.s#L12-L16) 清空切片中的数据，这也是我们在下面的小节会介绍的内容。



#### 随机遍历

当我们在 Go 语言中使用 `range` 遍历哈希表时，往往都会使用如下的代码结构，但是这段代码在每次运行时都会打印出不同的结果：

```go
func main() {
    hash := map[string]int{
        "1": 1,
        "2": 2,
        "3": 3,
    }
    for k, v := range hash {
        println(k, v)
    }
}
```

两次运行上述代码可能会得到不同的结果，第一次会打印 `2 3 1`，第二次会打印 `1 2 3`，如果我们运行的次数足够多，最后会得到几种不同的遍历顺序。

这是 Go 语言故意的设计，它在运行时为哈希表的遍历引入不确定性，也是告诉所有使用 Go 语言的使用者，程序不要依赖于哈希表的稳定遍历，我们在下面的小节会介绍在**遍历的过程是如何引入不确定性的**。





### 5.1.2 经典循环

Go 语言中的经典循环在编译器看来是一个 `OFOR` 类型的节点，这个节点由以下四个部分组成：

- 初始化循环的 `Ninit`；
- 循环的中止条件 `Left`；
- 循环体结束时执行的 `Right`；
- 循环体 `NBody`：

```
for Ninit; Left; Right {
    NBody
}
```

在生成 SSA 中间代码的阶段，[`cmd/compile/internal/gc.stmt`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/ssa.go#L1023-L1502) 方法在发现传入的节点类型是 `OFOR` 时就会执行以下的代码块，这段代码的会将循环中的代码分成不同的块：

```go
func (s *state) stmt(n *Node) {
    switch n.Op {
    case OFOR, OFORUNTIL:
        bCond, bBody, bIncr, bEnd := ...
        b := s.endBlock()
        b.AddEdgeTo(bCond)
        s.startBlock(bCond)
        s.condBranch(n.Left, bBody, bEnd, 1)
        s.startBlock(bBody)
        s.stmtList(n.Nbody)
        b.AddEdgeTo(bIncr)
        s.startBlock(bIncr)
        s.stmt(n.Right)
        b.AddEdgeTo(bCond)
        s.startBlock(bEnd)
    }
}
```



一个常见的 for 循环代码会被 [`cmd/compile/internal/gc.stmt`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/ssa.go#L1023-L1502) 方法转换成下面的控制结构，该结构中包含了 4 个不同的块，这些代码块之间的连接就表示汇编语言中的跳转关系，与我们理解的 for 循环控制结构其实没有太多的差别。

![golang-for-loop-ssa](go语言设计与实现.assets/f7368d90712554bf0b4fc7cdf6420a03.png)

**图 5-1 Go 语言循环生成的 SSA 代码**

[机器码生成](https://www.bookstack.cn/read/draveness-golang/100aabc3275d17e5.md)阶段会将这些代码块转换成机器码，以及指定 CPU 架构上运行的机器语言，就是我们在前面编译得到的汇编指令。



### 5.1.3 范围循环

与简单的经典循环相比，范围循环在 Go 语言中更常见、实现也更复杂。这种循环同时使用 for 和 range 两个关键字，编译器会在编译期间将所有 for/range 循环变成的经典循环。从编译器的视角来看，就是将 `ORANGE` 类型的节点转换成 `OFOR` 节点:

![Golang-For-Range-Loop](go语言设计与实现.assets/16b73509b943c14f0114f0c6cd7e661f.png)

**图 5-2 范围循环、普通循环和 SSA**

节点类型的转换过程都发生在 SSA 中间代码生成阶段，所有的 for/range 循环都会被 [`cmd/compile/internal/gc.walkrange`](https://github.com/golang/go/blob/440f7d64048cd94cba669e16fe92137ce6b84073/src/cmd/compile/internal/gc/range.go#L155-L456) 函数转换成不包含复杂结构、只包含基本表达式的语句。接下来，我们按照循环遍历的元素类型依次介绍遍历数组和切片、哈希表、字符串以及管道时的过程。

#### 数组和切片

对于数组和切片来说，Go 语言有三种不同的遍历方式，这三种不同的遍历方式分别对应着代码中的不同条件，它们会在 [`cmd/compile/internal/gc.walkrange`](https://github.com/golang/go/blob/440f7d64048cd94cba669e16fe92137ce6b84073/src/cmd/compile/internal/gc/range.go#L155-L456) 函数中转换成不同的控制逻辑，我们将该函数的相关逻辑分成几个部分进行分析：

- 分析遍历数组和切片清空元素的情况；
- 分析使用 `for range a {}` 遍历数组和切片，不关心索引和数据的情况；
- 分析使用 `for i := range a {}` 遍历数组和切片，只关心索引的情况；
- 分析使用 `for i, elem := range a {}` 遍历数组和切片，关心索引和数据的情况；

```go
func walkrange(n *Node) *Node {
    switch t.Etype {
    case TARRAY, TSLICE:
        if arrayClear(n, v1, v2, a) {
            return n
        }
```

[`cmd/compile/internal/gc.arrayClear`](https://github.com/golang/go/blob/440f7d64048cd94cba669e16fe92137ce6b84073/src/cmd/compile/internal/gc/range.go#L532-L610) 是一个非常有趣的优化，这个函数会优化 Go 语言遍历数组或者切片并删除全部元素的逻辑：

```
// original
for i := range a {
    a[i] = zero
}
// optimized
if len(a) != 0 {
    hp = &a[0]
    hn = len(a)*sizeof(elem(a))
    memclrNoHeapPointers(hp, hn)
    i = len(a) - 1
}
```

相比于依次清除数组或者切片中的数据，Go 语言会直接使用 [`runtime.memclrNoHeapPointers`](https://github.com/golang/go/blob/05c02444eb2d8b8d3ecd949c4308d8e2323ae087/src/runtime/memclr_386.s#L12-L16) 或者 [`runtime.memclrHasPointers`](https://github.com/golang/go/blob/db16de920370892b0241d3fa0617dddff2417a4d/src/runtime/mbarrier.go#L345-L348) 函数直接清除目标数组对应内存空间中的数据，并在执行完成后更新用于遍历数组的索引，这也印证了我们在遍历清空数组一节中观察到的现象。

处理了这种特殊的情况之后，我们就可以继续回到 `ORANGE` 节点的处理过程了。这里会设置 for 循环的 `Left` 和 `Right` 字段，也就是终止条件和循环体每次执行结束后运行的代码：

```
ha := a
hv1 := temp(types.Types[TINT])
hn := temp(types.Types[TINT])
init = append(init, nod(OAS, hv1, nil))
init = append(init, nod(OAS, hn, nod(OLEN, ha, nil)))
n.Left = nod(OLT, hv1, hn)
n.Right = nod(OAS, hv1, nod(OADD, hv1, nodintconst(1)))
if v1 == nil {
break
}
```

如果原始的循环是 `for range a {}`，那么就满足 `v1 == nil` 的条件，即循环不关心数组的索引和数据，它会被编译器转换成如下所示的代码：

```
ha := a
hv1 := 0
hn := len(ha)
v1 := hv1
for ; hv1 < hn; hv1++ {
    ...
}
```

这是 `ORANGE` 结构在编译期间被转换的最简单形式，由于原始代码不需要获取数组的索引和元素，只需要使用数组或者切片的数量执行对应次数的循环，所以会生成一个最简单的 for 循环。

如果我们在遍历数组时需要使用索引 `for i := range a {}`，那么编译器会继续会执行下面的代码：

```
        if v2 == nil {
            body = []*Node{nod(OAS, v1, hv1)}
            break
        }
```

`v2 == nil` 意味着调用方不关心数组的元素，只关心遍历数组使用的索引。它会将 `for i := range a {}` 转换成如下所示的逻辑，与第一种循环相比，这种循环在循环体中添加了 `v1 := hv1` 语句，传递遍历数组时的索引：

```
ha := a
hv1 := 0
hn := len(ha)
v1 := hv1
for ; hv1 < hn; hv1++ {
    v1 := hv1
    ...
}
```

上面的两种情况虽然也是使用 range 经常遇到的情况，但是同时去遍历索引和元素也很常见。处理这种情况会使用下面这段的代码：

```
        tmp := nod(OINDEX, ha, hv1)
        tmp.SetBounded(true)
        a := nod(OAS2, nil, nil)
        a.List.Set2(v1, v2)
        a.Rlist.Set2(hv1, tmp)
        body = []*Node{a}
    }
    n.Ninit.Append(init...)
    n.Nbody.Prepend(body...)
    return n
}
```

这段代码处理的就是遍历数组和切片时，同时关心索引和切片的情况。它不仅会在循环体中插入更新索引的语句，还会插入赋值操作让循环体内部的代码能够访问数组中的元素：

```
ha := a
hv1 := 0
hn := len(ha)
v1 := hv1
for ; hv1 < hn; hv1++ {
    tmp := ha[hv1]
    v1, v2 := hv1, tmp
    ...
}
```

对于所有的 range 循环，Go 语言都会在编译期将原切片或者数组赋值给一个新的变量 `ha`，在赋值的过程中就发生了拷贝，所以我们遍历的切片已经不是原始的切片变量了。

而遇到这种同时遍历索引和元素的 range 循环时，Go 语言会额外创建一个新的 `v2` 变量存储切片中的元素，**循环中使用的这个变量 v2 会在每一次迭代被重新赋值，在赋值时也发生了拷贝**。

```
func main() {
    arr := []int{1, 2, 3}
    newArr := []*int{}
    for i, _ := range arr {
        newArr = append(newArr, &arr[i])
    }
    for _, v := range newArr {
        fmt.Println(*v)
    }
}
```

因为在循环中获取返回变量的地址都完全相同，所以会发生神奇的指针一节中的现象。所以如果我们想要访问数组中元素所在的地址，不应该直接获取 range 返回的变量地址 `&v2`，而应该使用 `&a[index]` 这种形式。



#### 哈希表

在遍历哈希表时，编译器会使用 [`runtime.mapiterinit`](https://github.com/golang/go/blob/36f30ba289e31df033d100b2adb4eaf557f05a34/src/runtime/map.go#L797-L844) 和 [`runtime.mapiternext`](https://github.com/golang/go/blob/36f30ba289e31df033d100b2adb4eaf557f05a34/src/runtime/map.go#L846-L970) 两个运行时函数重写原始的 for/range 循环：

```
ha := a
hit := hiter(n.Type)
th := hit.Type
mapiterinit(typename(t), ha, &hit)
for ; hit.key != nil; mapiternext(&hit) {
    key := *hit.key
    val := *hit.val
}
```

上述代码是 `for key, val := range hash {}` 生成的，在 [`cmd/compile/internal/gc.walkrange`](https://github.com/golang/go/blob/440f7d64048cd94cba669e16fe92137ce6b84073/src/cmd/compile/internal/gc/range.go#L155-L456) 函数处理 `TMAP` 节点时会根据接受 range 返回值的数量在循环体中插入需要的赋值语句：

![golang-range-map](go语言设计与实现.assets/dace09cf64b49e7bada876055a3a3578.png)

**图 5-3 不同方式遍历哈希插入的语句**

这三种不同的情况会分别向循环体插入不同的赋值语句。遍历哈希表时会使用 [`runtime.mapiterinit`](https://github.com/golang/go/blob/36f30ba289e31df033d100b2adb4eaf557f05a34/src/runtime/map.go#L797-L844) 函数初始化遍历开始的元素：

```
func mapiterinit(t *maptype, h *hmap, it *hiter) {
    it.t = t
    it.h = h
    it.B = h.B
    it.buckets = h.buckets
    r := uintptr(fastrand())
    it.startBucket = r & bucketMask(h.B)
    it.offset = uint8(r >> h.B & (bucketCnt - 1))
    it.bucket = it.startBucket
    mapiternext(it)
}
```

该函数会初始化 `hiter` 结构体中的字段，并通过 [`runtime.fastrand`](https://github.com/golang/go/blob/383b447e0da5bd1fcdc2439230b5a1d3e3402117/src/runtime/stubs.go#L99-L111) 生成一个随机数帮助我们随机选择一个桶开始遍历。Go 团队在设计哈希表的遍历时就不想让使用者依赖固定的遍历顺序，所以引入了随机数保证遍历的随机性。

遍历哈希会使用 [`runtime.mapiternext`](https://github.com/golang/go/blob/36f30ba289e31df033d100b2adb4eaf557f05a34/src/runtime/map.go#L846-L970) 函数，我们在这里简化了很多逻辑，省去了一些边界条件以及哈希表扩容时的兼容操作，这里只需要关注处理遍历逻辑的核心代码，我们会将该函数分成桶的选择和桶内元素的遍历两部分进行分析，首先是桶的选择过程：

```go
func mapiternext(it *hiter) {
    h := it.h
    t := it.t
    bucket := it.bucket
    b := it.bptr
    i := it.i
    alg := t.key.alg
next:
    if b == nil {
        if bucket == it.startBucket && it.wrapped {
            it.key = nil
            it.value = nil
            return
        }
        b = (*bmap)(add(it.buckets, bucket*uintptr(t.bucketsize)))
        bucket++
        if bucket == bucketShift(it.B) {
            bucket = 0
            it.wrapped = true
        }
        i = 0
    }
```

这段代码主要有两个作用：

- 在待遍历的桶为空时选择需要遍历的新桶；
- 在不存在待遍历的桶时返回 `(nil, nil)` 键值对并中止遍历过程；[`runtime.mapiternext`](https://github.com/golang/go/blob/36f30ba289e31df033d100b2adb4eaf557f05a34/src/runtime/map.go#L846-L970) 函数中第二段代码的主要作用就是从桶中找到下一个遍历的元素，在大多数情况下都会直接操作内存获取目标键值的内存地址，不过如果哈希表处于扩容期间就会调用 [`runtime.mapaccessK`](https://github.com/golang/go/blob/36f30ba289e31df033d100b2adb4eaf557f05a34/src/runtime/map.go#L511-L552) 函数获取键值对：

```
    for ; i < bucketCnt; i++ {
        offi := (i + it.offset) & (bucketCnt - 1)
        k := add(unsafe.Pointer(b), dataOffset+uintptr(offi)*uintptr(t.keysize))
        v := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+uintptr(offi)*uintptr(t.valuesize))
        if (b.tophash[offi] != evacuatedX && b.tophash[offi] != evacuatedY) ||
            !(t.reflexivekey() || alg.equal(k, k)) {
            it.key = k
            it.value = v
        } else {
            rk, rv := mapaccessK(t, h, k)
            it.key = rk
            it.value = rv
        }
        it.bucket = bucket
        it.i = i + 1
        return
    }
    b = b.overflow(t)
    i = 0
    goto next
}
```

当上述函数已经遍历了正常桶，就会通过 [`runtime.bmap.overflow`](https://github.com/golang/go/blob/36f30ba289e31df033d100b2adb4eaf557f05a34/src/runtime/map.go#L207-L209) 获取溢出桶依次进行遍历。

![golang-range-map-and-buckets](go语言设计与实现.assets/153e4cf478dc6c27474a6f29842f9be2.png)

**图 5-4 哈希表的遍历过程**

简单总结一下哈希表遍历的顺序，首先会选出一个绿色的正常桶开始遍历，随后遍历对应的所有黄色溢出桶，最后依次按照索引顺序遍历哈希表中其他的桶，直到所有的桶都被遍历完成。



#### 字符串

遍历字符串的过程与数组、切片和哈希表非常相似，只是在遍历时会获取字符串中索引对应的字节并将字节转换成 `rune`。我们在遍历字符串时拿到的值都是 `rune` 类型的变量，`for i, r := range s {}` 的结构都会被转换成如下所示的形式：

```
ha := s
for hv1 := 0; hv1 < len(ha); {
    hv1t := hv1
    hv2 := rune(ha[hv1])
    if hv2 < utf8.RuneSelf {
        hv1++
    } else {
        hv2, hv1 = decoderune(h1, hv1)
    }
    v1, v2 = hv1t, hv2
}
```

在前面的字符串一节中我们曾经介绍过字符串是一个只读的字节数组切片，所以范围循环在编译期间生成的框架与切片非常类似，只是细节有一些不同。

使用下标访问字符串中的元素时得到的就是字节，但是这段代码会将当前的字节转换成 `rune` 类型。如果当前的 `rune` 是 ASCII 的，那么只会占用一个字节长度，每次循环体运行之后只需要将索引加一，但是如果当前 `rune` 占用了多个字节就会使用 [`runtime.decoderune`](https://github.com/golang/go/blob/c6e84263865fa418b4d4a60f077d02c10a0fff23/src/runtime/utf8.go#L60-L100) 函数解码，具体的过程就不在这里详细介绍了。

#### 通道

使用 range 遍历 Channel 也是比较常见的做法，一个形如 `for v := range ch {}` 的语句最终会被转换成如下的格式：

```
ha := a
hv1, hb := <-ha
for ; hb != false; hv1, hb = <-ha {
    v1 := hv1
    hv1 = nil
    ...
}
```

这里的代码可能与编译器生成的稍微有一些出入，但是结构和效果是完全相同的。该循环会使用 `<-ch` 从管道中取出等待处理的值，这个操作会调用 [`runtime.chanrecv2`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L437-L440) 并阻塞当前的协程，当 [`runtime.chanrecv2`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L437-L440) 返回时会根据布尔值 `hb` 判断当前的值是否存在，如果不存在就意味着当前的管道已经被关闭了，如果存在就会为 `v1` 赋值并清除 `hv1` 变量中的数据，然后会重新陷入阻塞等待新数据。



### 5.1.4 小结

这一节介绍的两个关键字 for 和 range 都是我们在学习和使用 Go 语言中无法绕开的，通过分析和研究它们的底层原理，让我们对实现细节有了更清楚的认识，包括 Go 语言遍历数组和切片时会复用变量、哈希表的随机遍历原理以及底层的一些优化，这都能帮助我们理解和使用 Go 语言。

- CommonMistakes · Go https://github.com/golang/go/wiki/CommonMistakes[↩︎](https://www.bookstack.cn/read/draveness-golang/816b8b0f4e15285f.md#fnref:1)



## 5.2 select

> [5.2 select](https://www.bookstack.cn/read/draveness-golang/1a4f7a284cd2b279.md)

很多 C 语言或者 Unix 开发者听到 `select` 想到的都是系统调用，而谈到 I/O 模型时最终大都会提到基于 `select`、`poll` 和 `epoll` 等函数构建的 IO 多路复用模型。

Go 语言的 `select` 与 C 语言中的 `select` 有着比较相似的功能。本节会介绍 Go 语言 `select` 常见的现象、数据结构以及四种不同情况下的实现原理。

C 语言中的 `select` 关键字可以同时监听多个文件描述符的可读或者可写的状态，**Go 语言中的 `select` 关键字也能够让 Goroutine 同时等待多个 Channel 的可读或者可写，在多个文件或者 Channel 发生状态改变之前，`select` 会一直阻塞当前线程或者 Goroutine**。

![Golang-Select-Channels](go语言设计与实现.assets/d5f86fa94ffc8ee348eda427e3344f84.png)

**图 5-5 Select 和 Channels**

`select` 是一种与 `switch` 相似的控制结构，与 `switch` 不同的是，`select` 中虽然也有多个 `case`，但是**这些 `case` 中的表达式必须都是 [Channel](https://www.bookstack.cn/read/draveness-golang/c666731d5f1a2820.md) 的收发操作**。下面的代码就展示了一个包含 Channel 收发操作的 `select` 结构：

```go
func fibonacci(c, quit chan int) {
    x, y := 0, 1
    for {
        select {
        case c <- x:
            x, y = y, x+y
        case <-quit:
            fmt.Println("quit")
            return
        }
    }
}

func main() {
	c := make(chan int)
	quit := make(chan int)

	// 启动 Fibonacci 协程
	go fibonacci(c, quit)

	// 从通道 c 中接收 Fibonacci 数列并打印
	for i := 0; i < 10; i++ {
		fmt.Println(<-c)
	}
	// 发送信号给 quit 通道，结束 Fibonacci 协程
	quit <- 0

}
```

上述控制结构会等待 `c <- x` 或者 `<-quit` 两个表达式中任意一个的返回。无论哪一个表达式返回都会立刻执行 `case` 中的代码，当 `select` 中的两个 `case` 同时被触发时，就会随机选择一个 `case` 执行。



### 5.2.1 现象

当我们在 Go 语言中使用 `select` 控制结构时，会遇到两个有趣的现象：

- `select` 能在 Channel 上进行非阻塞的收发操作；
- `select` 在**遇到多个 Channel 同时响应时会随机挑选 `case` 执行**；这两个现象是学习 `select` 时经常会遇到的，我们来深入了解具体的场景并分析这两个现象背后的设计原理。

#### 非阻塞的收发（default）

在通常情况下，`select` 语句会阻塞当前 Goroutine 并等待多个 Channel 中的一个达到可以收发的状态。但是如果 `select` 控制结构中**包含 `default` 语句**，那么这个 `select` 语句在执行时会遇到以下两种情况：

- 当存在可以收发的 Channel 时，直接处理该 Channel 对应的 `case`；
- 当不存在可以收发的 Channel 是，执行 `default` 中的语句；当我们运行下面的代码时就不会阻塞当前的 Goroutine，它会直接执行 `default` 中的代码并返回。

```go
func main() {
    ch := make(chan int)
    select {
    case i := <-ch:
        println(i)
    default:
        println("default")
    }
}
$ go run main.go
default
```

只要我们稍微想一下，就会发现 Go 语言设计的这个现象就非常合理。`select` 的作用就是同时监听多个 `case` 是否可以执行，如果多个 Channel 都不能执行，那么运行 `default` 中的代码也是理所当然的。

非阻塞的 Channel 发送和接收操作还是很有必要的，

在很多场景下我们不希望向 Channel 发送消息或者从 Channel 中接收消息会阻塞当前 Goroutine，**我们只是想看看 Channel 的可读或者可写状态**。下面就是一个常见的例子：

```go
errCh := make(chan error, len(tasks)
wg := sync.WaitGroup{}
wg.Add(len(tasks))
for i := range tasks {
    go func() {
        defer wg.Done()
        if err := tasks[i].Run(); err != nil {
            errCh <- err
        }
    }()
}
wg.Wait()
select {
case err := <-errCh:
    return err
default:
    return nil
}
```

> 有错误时才返回

在上面这段代码中，我们不关心到底多少个任务执行失败了，只关心是否存在返回错误的任务，最后的 `select` 语句就能很好地完成这个任务。然而使用 `select` 的语法不是最原始的设计，它在最初版本使用 `x, ok := <-c` 的语法实现非阻塞的收发，以下是与非阻塞收发的相关提交：

- [select default](https://github.com/golang/go/commit/79fbbe37a76502e6f5f9647d2d82bab953ab1546#diff-fb0a5ae9dd70f0a43038d55c0204fdff) 提交支持了 `select` 语句中的 `default` 情况[1](https://www.bookstack.cn/read/draveness-golang/1a4f7a284cd2b279.md#fn:1)；
- [gc: special case code for single-op blocking and non-blocking selects](https://github.com/golang/go/commit/5038792837355abde32f2e9549ef132fc5ffbd16) 提交引入了基于 `select` 的非阻塞收发的特性[2](https://www.bookstack.cn/read/draveness-golang/1a4f7a284cd2b279.md#fn:2)。
- [gc: remove non-blocking send, receive syntax](https://github.com/golang/go/commit/cb584707af2d8803adba88fd9692e665ecd2f059) 提交将 `x, ok := <-c` 语法删除删除[3](https://www.bookstack.cn/read/draveness-golang/1a4f7a284cd2b279.md#fn:3)；
- [gc, runtime: replace closed(c) with x, ok := <-c](https://github.com/golang/go/commit/8bf34e335686816f7fe7e28614b2c7a3e04e9e7c) 提交使用 `x, ok := <-c` 语法替代 `closed(c)` 语法判断 Channel 的关闭状态[4](https://www.bookstack.cn/read/draveness-golang/1a4f7a284cd2b279.md#fn:4)；我们可以从上面的几个提交中看到非阻塞收发从最初到现在的演变



#### 随机执行

另一个使用 `select` 遇到的情况是同时有多个 `case` 就绪时，`select` 会选择那个 `case` 执行的问题，我们通过下面的代码可以简单了解一下：

```go
func main() {
    ch := make(chan int)
    go func() {
        for range time.Tick(1 * time.Second) {
            ch <- 0
        }
    }()
    for {
        select {
        case <-ch:
            println("case1")
        case <-ch:
            println("case2")
        }
    }
}
$ go run main.go
case1
case2
case1
...
```

从上述代码输出的结果中我们可以看到，`select` 在遇到多个 `<-ch` 同时满足可读或者可写条件时会随机选择一个 `case` 执行其中的代码。

这个设计是在十多年前被 [select](https://github.com/golang/go/commit/cb9b1038db77198c2b0961634cf161258af2374d) 提交[5](https://www.bookstack.cn/read/draveness-golang/1a4f7a284cd2b279.md#fn:5)引入并一直保留到现在的，虽然中间经历过一些修改[6](https://www.bookstack.cn/read/draveness-golang/1a4f7a284cd2b279.md#fn:6)，但是语义一直都没有改变。在上面的代码中，两个 `case` 都是同时满足执行条件的，如果我们按照顺序依次判断，那么后面的条件永远都会得不到执行，而随机的引入就是**为了避免饥饿问题的发生**。



### 5.2.2 数据结构

`select` 在 Go 语言的源代码中不存在对应的结构体，但是 `select` 控制结构中的 `case` 却使用 [`runtime.scase`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/select.go#L28-L34) 结构体来表示：

```go
type scase struct {
    c           *hchan
    elem        unsafe.Pointer
    kind        uint16
    pc          uintptr
    releasetime int64
}
```

因为非默认的 `case` 中都与 Channel 的发送和接收有关，所以 [`runtime.scase`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/select.go#L28-L34) 结构体中也包含一个 [`runtime.hchan`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L32-L51) 类型的字段存储 `case` 中使用的 Channel；除此之外，`elem` 是接收或者发送数据的变量地址、`kind` 表示 [`runtime.scase`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/select.go#L28-L34) 的种类，总共包含以下四种：

```go
const (
    caseNil = iota
    caseRecv
    caseSend
    caseDefault
)
```

这四种常量分别表示不同类型的 `case`，相信它们的命名已经能够充分帮助我们理解它们的作用了，所以这里也不一一介绍了。



### 5.2.3 实现原理（代码转换）

`select` 语句在编译期间会被转换成 `OSELECT` 节点。每一个 `OSELECT` 节点都会持有一组 `OCASE` 节点，如果 `OCASE` 的执行条件是空，那就意味着这是一个 `default` 节点:

![golang-oselect-and-ocases](go语言设计与实现.assets/8d58644e2f05754b5a538fa21436d2e0.png)

**图 5-7 OSELECT 和多个 OCASE**



上图展示的就是 `select` 语句在编译期间的结构，每一个 `OCASE` 既包含执行条件也包含满足条件后执行的代码。

编译器在中间代码生成期间会根据 `select` 中 `case` 的不同对控制语句进行优化，这一过程都发生在 [`cmd/compile/internal/gc.walkselectcases`](https://github.com/golang/go/blob/c729116332ffb66a21dd587e3ee003cb8d0b16fe/src/cmd/compile/internal/gc/select.go#L108-L370) 函数中，我们在这里会分四种情况介绍处理的过程和结果：

- `select` 不存在任何的 `case`；
- `select` 只存在一个 `case`；
- `select` 存在量个 `case`，其中一个 `case` 是 `default`；
- `select` 存在多个 `case`；上述的四种情况不仅会涉及编译器的重写和优化，还会涉及 Go 语言的运行时机制，我们会从编译期间和运行时两方面分析上述情况。



#### 直接阻塞(空case)

首先介绍的是最简单的情况，也就是当 `select` 结构中不包含任何 `case` 时编译器是如何进行处理的，

```
select {

}
```

我们截取 [`cmd/compile/internal/gc.walkselectcases`](https://github.com/golang/go/blob/c729116332ffb66a21dd587e3ee003cb8d0b16fe/src/cmd/compile/internal/gc/select.go#L108-L370) 函数的前几行代码：

```go
func walkselectcases(cases *Nodes) []*Node {
    n := cases.Len()
    if n == 0 {
        return []*Node{mkcall("block", nil, nil)}
    }
    ...
}
```

这段代码非常简单并且容易理解，它直接将类似 `select {}` 的空语句转换成调用 [`runtime.block`](https://github.com/golang/go/blob/c112289ee4141ebc31db50328c355b01278b987b/src/runtime/select.go#L104-L106) 函数：

```
func block() {
    gopark(nil, nil, waitReasonSelectNoCases, traceEvGoStop, 1)
}
```

[`runtime.block`](https://github.com/golang/go/blob/c112289ee4141ebc31db50328c355b01278b987b/src/runtime/select.go#L104-L106) 函数的实现非常简单，它会调用 [`runtime.gopark`](https://github.com/golang/go/blob/c112289ee4141ebc31db50328c355b01278b987b/src/runtime/proc.go#L287-L305) 让出当前 Goroutine 对处理器的使用权，传入的等待原因是 `waitReasonSelectNoCases`。

简单总结一下，**空的 `select` 语句会直接阻塞当前的 Goroutine，导致 Goroutine 进入无法被唤醒的永久休眠状态**。

```go
func main() {
	select {}
}

fatal error: all goroutines are asleep - deadlock!
goroutine 1 [select (no cases)]:
main.main()
	go_test/go_context/test.go:22 +0x17
```



#### 单一管道

如果当前的 `select` 条件只包含一个 `case`，那么就会将 `select` 改写成 `if` 条件语句。下面展示了原始的 `select` 语句和被改写、优化后的代码：

```go
// 改写前
select {
case v, ok <-ch: // case ch <- v
    ...    
}
// 改写后
if ch == nil {
    block()
}
v, ok := <-ch // case ch <- v
...
```

[`cmd/compile/internal/gc.walkselectcases`](https://github.com/golang/go/blob/c729116332ffb66a21dd587e3ee003cb8d0b16fe/src/cmd/compile/internal/gc/select.go#L108-L370) 在处理单操作 `select` 语句时，会根据 Channel 的收发情况生成不同的语句。当 `case` 中的 Channel 是空指针时，就会直接挂起当前 Goroutine 并永久休眠。



#### 非阻塞操作

当 `select` 中仅包含两个 `case`，并且其中一个是 `default` 时，Go 语言的编译器就会认为这是一次非阻塞的收发操作。[`cmd/compile/internal/gc.walkselectcases`](https://github.com/golang/go/blob/c729116332ffb66a21dd587e3ee003cb8d0b16fe/src/cmd/compile/internal/gc/select.go#L108-L370) 函数会对这种情况单独处理，不过在正式优化之前，该函数会将 `case` 中的所有 Channel 都转换成指向 Channel 的地址。我们会分别介绍非阻塞发送和非阻塞接收时，编译器进行的不同优化。



##### 发送

首先是 Channel 的发送过程，当 `case` 中表达式的类型是 `OSEND` 时，编译器会使用 `if/else` 语句和 [`runtime.selectnbsend`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L662-L664) 函数改写代码：

```go
select {
case ch <- i:
    ...
default:
    ...
}

==》
if selectnbsend(ch, i) {
    ...
} else {
    ...
}
```

这段代码中最重要的就是 [`runtime.selectnbsend`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L662-L664) 函数，它为我们提供了向 Channel 非阻塞地发送数据的能力。我们在 [Channel](https://www.bookstack.cn/read/draveness-golang/c666731d5f1a2820.md) 一节介绍了向 Channel 发送数据的 [`runtime.chansend`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L157-L278) 函数包含一个 `block` 参数，该参数会决定这一次的发送是不是阻塞的：

```
func selectnbsend(c *hchan, elem unsafe.Pointer) (selected bool) {
    return chansend(c, elem, false, getcallerpc())
}
```



由于我们向 [`runtime.chansend`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L157-L278) 函数传入了 `false`，所以哪怕是不存在接收方或者缓冲区空间不足都不会阻塞当前 Goroutine 而是会直接返回。

##### 接收

由于从 Channel 中接收数据可能会返回一个或者两个值，所以接受数据的情况会比发送稍显复杂，不过改写的套路是差不多的：

```
// 改写前
select {
case v <- ch: // case v, ok <- ch:
    ......
default:
    ......
}
// 改写后
if selectnbrecv(&v, ch) { // if selectnbrecv2(&v, &ok, ch) {
    ...
} else {
    ...
}
```

返回值数量不同会导致使用函数的不同，两个用于非阻塞接收消息的函数 [`runtime.selectnbrecv`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L683-L686) 和 [`runtime.selectnbrecv2`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L705-L709) 只是对 [`runtime.chanrecv`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L448-L579) 返回值的处理稍有不同：

```go
func selectnbrecv(elem unsafe.Pointer, c *hchan) (selected bool) {
    selected, _ = chanrecv(c, elem, false)
    return
}
func selectnbrecv2(elem unsafe.Pointer, received *bool, c *hchan) (selected bool) {
    selected, *received = chanrecv(c, elem, false)
    return
}
```

因为接收方不需要，所以 [`runtime.selectnbrecv`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L683-L686) 会直接忽略返回的布尔值，而 [`runtime.selectnbrecv2`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L705-L709) 会将布尔值回传给调用方。与 [`runtime.chansend`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L157-L278) 一样，[`runtime.chanrecv`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L448-L579) 也提供了一个 `block` 参数用于控制这一次接收是否阻塞。



#### 常见流程

在默认的情况下，编译器会使用如下的流程处理 `select` 语句：

- 将所有的 `case` 转换成包含 Channel 以及类型等信息的 [`runtime.scase`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/select.go#L28-L34) 结构体；
- 调用运行时函数 [`runtime.selectgo`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/select.go#L118-L497) 从多个准备就绪的 Channel 中选择一个可执行的 [`runtime.scase`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/select.go#L28-L34) 结构体；
- 通过 `for` 循环生成一组 `if` 语句，在语句中判断自己是不是被选中的 `case`一个包含三个 `case` 的正常 `select` 语句其实会被展开成如下所示的逻辑，我们可以看到其中处理的三个部分：

```
selv := [3]scase{}
order := [6]uint16
for i, cas := range cases {
    c := scase{}
    c.kind = ...
    c.elem = ...
    c.c = ...
}
chosen, revcOK := selectgo(selv, order, 3)
if chosen == 0 {
    ...
    break
}
if chosen == 1 {
    ...
    break
}
if chosen == 2 {
    ...
    break
}
```



展开后的代码片段中最重要的就是用于选择待执行 `case` 的运行时函数 [`runtime.selectgo`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/select.go#L118-L497)，这也是我们要关注的重点。因为这个函数的实现比较复杂， 所以这里分两部分分析它的执行过程：

- 执行一些必要的初始化操作并确定 `case` 的处理顺序；
- 在循环中根据 `case` 的类型做出不同的处理；



##### 初始化

[`runtime.selectgo`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/select.go#L118-L497) 函数首先会进行执行必要的初始化操作并决定处理 `case` 的两个顺序 — 轮训顺序 `pollOrder` 和加锁顺序 `lockOrder`：

```go
func selectgo(cas0 *scase, order0 *uint16, ncases int) (int, bool) {
    cas1 := (*[1 << 16]scase)(unsafe.Pointer(cas0))
    order1 := (*[1 << 17]uint16)(unsafe.Pointer(order0))
    scases := cas1[:ncases:ncases]
    pollorder := order1[:ncases:ncases]
    lockorder := order1[ncases:][:ncases:ncases]
    for i := range scases {
        cas := &scases[i]
    }
    for i := 1; i < ncases; i++ {
        j := fastrandn(uint32(i + 1))
        pollorder[i] = pollorder[j]
        pollorder[j] = uint16(i)
    }
    // 根据 Channel 的地址排序确定加锁顺序
    ...
    sellock(scases, lockorder)
    ...
}
```

轮训顺序 `pollOrder` 和加锁顺序 `lockOrder` 分别是通过以下的方式确认的：

- 轮训顺序：通过 [`runtime.fastrandn`](https://github.com/golang/go/blob/383b447e0da5bd1fcdc2439230b5a1d3e3402117/src/runtime/stubs.go#L114-L118) 函数引入随机性；
- 加锁顺序：按照 Channel 的地址排序后确定加锁顺序；

随机的轮训顺序可以避免 Channel 的饥饿问题，保证公平性；而根据 Channel 的地址顺序确定加锁顺序能够避免死锁的发生。这段代码最后调用的 [`runtime.sellock`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/select.go#L45-L54) 函数会按照之前生成的加锁顺序锁定 `select` 语句中包含所有的 Channel。



##### 循环

当我们为 `select` 语句锁定了所有 Channel 之后就会进入 [`runtime.selectgo`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/select.go#L118-L497) 函数的主循环，它会分三个阶段查找或者等待某个 Channel 准备就绪：

- 查找是否已经存在准备就绪的 Channel，即可以执行收发操作；
- 将当前 Goroutine 加入 Channel 对应的收发队列上并等待其他 Goroutine 的唤醒；
- 当前 Goroutine 被唤醒之后找到满足条件的 Channel 并进行处理；[`runtime.selectgo`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/select.go#L118-L497) 函数会根据不同情况通过 `goto` 跳转到函数内部的不同标签执行相应的逻辑，其中包括：

- `bufrecv`：可以从缓冲区读取数据；
- `bufsend`：可以向缓冲区写入数据；
- `recv`：可以从休眠的发送方获取数据；
- `send`：可以向休眠的接收方发送数据；
- `rclose`：可以从关闭的 Channel 读取 EOF；
- `sclose`：向关闭的 Channel 发送数据；
- `retc`：结束调用并返回；

我们先来分析循环执行的第一个阶段，查找已经准备就绪的 Channel。循环会遍历所有的 `case` 并找到需要被唤起的 [`runtime.sudog`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L342-L368) 结构，在这个阶段，我们会根据 `case` 的四种类型分别处理：

- caseNil：当前case不包含 Channel；
  - 这种 `case` 会被跳过；
- caseRecv：当前case会从 Channel 中接收数据；
  - 如果当前 Channel 的 `sendq` 上有等待的 Goroutine，就会跳到 `recv` 标签从 Goroutine 中读取数据；
  - 如果当前 Channel 的缓冲区不为空就会跳到 `bufrecv` 标签处从缓冲区获取数据；
  - 如果当前 Channel 已经被关闭就会跳到 `rclose` 做一些清除的收尾工作；
- caseSend：当前case会向 Channel 发送数据；
  - 如果当前 Channel 已经被关闭就会直接跳到 `sclose` 标签，触发 `panic` 尝试中止程序；
  - 如果当前 Channel 的 `recvq` 上有等待的 Goroutine 就会跳到 `send` 标签向 Channel 发送数据；
- caseDefault：当前case为default语句；
  - 表示前面的所有 `case` 都没有被执行，这里会解锁所有 Channel 并返回，意味着当前 `select` 结构中的收发都是非阻塞的；![golang-runtime-selectgo](go语言设计与实现.assets/b1260b3496e78a79f57f45419239946c.png)

**图 5-8 运行时 selectgo 函数**

第一阶段的主要职责是查找所有 `case` 中 Channel 是否有可以立刻被处理的情况。无论是在包含等待的 Goroutine 还是缓冲区中存在数据，只要满足条件就会立刻处理，如果不能立刻找到活跃的 Channel 就会进入循环的下一阶段，按照需要将当前的 Goroutine 加入到 Channel 的 `sendq` 或者 `recvq` 队列中：

```
func selectgo(cas0 *scase, order0 *uint16, ncases int) (int, bool) {
    ...
    gp = getg()
    nextp = &gp.waiting
    for _, casei := range lockorder {
        casi = int(casei)
        cas = &scases[casi]
        c = cas.c
        sg := acquireSudog()
        sg.g = gp
        sg.c = c
        switch cas.kind {
        case caseRecv:
            c.recvq.enqueue(sg)
        case caseSend:
            c.sendq.enqueue(sg)
        }
    }
    gopark(selparkcommit, nil, waitReasonSelect, traceEvGoBlockSelect, 1)
    ...
}
```



除了将当前 Goroutine 对应的 [`runtime.sudog`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L342-L368) 结构体加入队列之外，这些 [`runtime.sudog`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L342-L368) 结构体都会被串成链表附着在 Goroutine 上。在入队之后会调用 [`runtime.gopark`](https://github.com/golang/go/blob/c112289ee4141ebc31db50328c355b01278b987b/src/runtime/proc.go#L287-L305) 函数挂起当前 Goroutine 等待调度器的唤醒。

![Golang-Select-Waiting](go语言设计与实现.assets/bf0f50fda03faa4ce46b9980b4fb3c17.png)

**图 5-9 Goroutine 上等待收发的 sudog 链表**

等到 `select` 中的一些 Channel 准备就绪之后，当前 Goroutine 就会被调度器唤醒。这时会继续执行 [`runtime.selectgo`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/select.go#L118-L497) 函数的第三阶段，从 [`runtime.sudog`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L342-L368) 结构体中获取数据：

```
func selectgo(cas0 *scase, order0 *uint16, ncases int) (int, bool) {
    ...
    sg = (*sudog)(gp.param)
    gp.param = nil
    casi = -1
    cas = nil
    sglist = gp.waiting
    for _, casei := range lockorder {
        k = &scases[casei]
        if sg == sglist {
            casi = int(casei)
            cas = k
        } else {
            if k.kind == caseSend {
                c.sendq.dequeueSudoG(sglist)
            } else {
                c.recvq.dequeueSudoG(sglist)
            }
        }
        sgnext = sglist.waitlink
        sglist.waitlink = nil
        releaseSudog(sglist)
        sglist = sgnext
    }
    c = cas.c
    goto retc
    ...
}
```

第三次遍历全部 `case` 时，我们会先获取当前 Goroutine 接收到的参数 `sudog` 结构，我们会依次对比所有 `case` 对应的 `sudog` 结构找到被唤醒的 `case`，获取该 `case` 对应的索引并返回。

由于当前的 `select` 结构找到了一个 `case` 执行，那么剩下 `case` 中没有被用到的 `sudog` 就会被忽略并且释放掉。为了不影响 Channel 的正常使用，我们还是需要将这些废弃的 `sudog` 从 Channel 中出队。

当我们在循环中发现缓冲区中有元素或者缓冲区未满时就会通过 `goto` 关键字跳转到 `bufrecv` 和 `bufsend` 两个代码段，这两段代码的执行过程都很简单，它们只是向 Channel 中发送数据或者从缓冲区中获取新数据：

```
bufrecv:
    recvOK = true
    qp = chanbuf(c, c.recvx)
    if cas.elem != nil {
        typedmemmove(c.elemtype, cas.elem, qp)
    }
    typedmemclr(c.elemtype, qp)
    c.recvx++
    if c.recvx == c.dataqsiz {
        c.recvx = 0
    }
    c.qcount--
    selunlock(scases, lockorder)
    goto retc
bufsend:
    typedmemmove(c.elemtype, chanbuf(c, c.sendx), cas.elem)
    c.sendx++
    if c.sendx == c.dataqsiz {
        c.sendx = 0
    }
    c.qcount++
    selunlock(scases, lockorder)
    goto retc
```

这里在缓冲区进行的操作和直接调用 [`runtime.chansend`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L157-L278) 和 [`runtime.chanrecv`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L448-L579) 函数差不多，上述两个过程在执行结束之后都会直接跳到 `retc` 字段。

两个直接对 Channel 收发的情况会调用 Channel 运行时函数 [`runtime.send`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L286-L317) 和 [`runtime.recv`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L594-L635)，这两个函数会直接与处于休眠状态的 Goroutine 打交道：

```
recv:
    recv(c, sg, cas.elem, func() { selunlock(scases, lockorder) }, 2)
    recvOK = true
    goto retc
send:
    send(c, sg, cas.elem, func() { selunlock(scases, lockorder) }, 2)
    goto retc
```

不过如果向关闭的 Channel 发送数据或者从关闭的 Channel 中接收数据，情况就稍微有一点复杂了：

- 从一个关闭 Channel 中接收数据会直接清除 Channel 中的相关内容；
- 向一个关闭的 Channel 发送数据就会直接 `panic` 造成程序崩溃：

```go
rclose:
    selunlock(scases, lockorder)
    recvOK = false
    if cas.elem != nil {
        typedmemclr(c.elemtype, cas.elem)
    }
    goto retc
sclose:
    selunlock(scases, lockorder)
    panic(plainError("send on closed channel"))
```

总体来看，`select` 语句中的 Channel 收发操作和直接操作 Channel 没有太多出入，只是由于 `select` 多出了 `default` 关键字所以会支持非阻塞的收发。

### 5.2.4 小结

我们简单总结一下 `select` 结构的执行过程与实现原理，首先在编译期间，Go 语言会对 `select` 语句进行优化，它会根据 `select` 中 `case` 的不同选择不同的优化路径：

- 空的 `select` 语句会被转换成 [`runtime.block`](https://github.com/golang/go/blob/c112289ee4141ebc31db50328c355b01278b987b/src/runtime/select.go#L104-L106) 函数的调用，直接挂起当前 Goroutine；

- 如果select语句中只包含一个case，就会被转换成

  ```
  if ch == nil { block }; n;
  ```

  表达式；

  - 首先判断操作的 Channel 是不是空的；
  - 然后执行 `case` 结构中的内容；

- 如果 `select` 语句中只包含两个 `case` 并且其中一个是 `default`，那么会使用 [`runtime.selectnbrecv`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L683-L686) 和 [`runtime.selectnbsend`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L662-L664) 非阻塞地执行收发操作；

- 在默认情况下会通过 [`runtime.selectgo`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/select.go#L118-L497) 函数获取执行 `case` 的索引，并通过多个 `if` 语句执行对应 `case` 中的代码；在编译器已经对 `select` 语句进行优化之后，Go 语言会在运行时执行编译期间展开的 [`runtime.selectgo`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/select.go#L118-L497) 函数，该函数会按照以下的流程执行：

- 随机生成一个遍历的轮询顺序 `pollOrder` 并根据 Channel 地址生成锁定顺序 `lockOrder`；

- 根据pollOrder遍历所有的case查看是否有可以立刻处理的 Channel；

  - 如果存在就直接获取 `case` 对应的索引并返回；
  - 如果不存在就会创建 [`runtime.sudog`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L342-L368) 结构体，将当前 Goroutine 加入到所有相关 Channel 的收发队列，并调用 [`runtime.gopark`](https://github.com/golang/go/blob/c112289ee4141ebc31db50328c355b01278b987b/src/runtime/proc.go#L287-L305) 挂起当前 Goroutine 等待调度器的唤醒；

- 当调度器唤醒当前 Goroutine 时就会再次按照 `lockOrder` 遍历所有的 `case`，从中查找需要被处理的 [`runtime.sudog`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L342-L368) 结构对应的索引；`select` 关键字是 Go 语言特有的控制结构，它的实现原理比较复杂，需要编译器和运行时函数的通力合作。





- Ken Thompson. Nov 6, 2008. select default. https://github.com/golang/go/commit/79fbbe37a76502e6f5f9647d2d82bab953ab1546#diff-fb0a5ae9dd70f0a43038d55c0204fdff[↩︎](https://www.bookstack.cn/read/draveness-golang/1a4f7a284cd2b279.md#fnref:1)
- Russ Cox. Jan 31, 2011. gc: special case code for single-op blocking and non-blocking selects. https://github.com/golang/go/commit/5038792837355abde32f2e9549ef132fc5ffbd16[↩︎](https://www.bookstack.cn/read/draveness-golang/1a4f7a284cd2b279.md#fnref:2)
- Russ Cox. Feb 1, 2011. gc: remove non-blocking send, receive syntax. https://github.com/golang/go/commit/cb584707af2d8803adba88fd9692e665ecd2f059[↩︎](https://www.bookstack.cn/read/draveness-golang/1a4f7a284cd2b279.md#fnref:3)
- Russ Cox. Mar 12, 2011. gc, runtime: replace closed(c) with x, ok := <-c. https://github.com/golang/go/commit/8bf34e335686816f7fe7e28614b2c7a3e04e9e7c[↩︎](https://www.bookstack.cn/read/draveness-golang/1a4f7a284cd2b279.md#fnref:4)
- Ken Thompson. Jul 25, 2008. select. https://github.com/golang/go/commit/cb9b1038db77198c2b0961634cf161258af2374d[↩︎](https://www.bookstack.cn/read/draveness-golang/1a4f7a284cd2b279.md#fnref:5)
- Gustavo Niemeyer. Aug 15, 2011. runtime: fix pseudo-randomness on some selects. https://github.com/golang/go/commit/175849295ce632c2ddeca7024f7c783327b5e571[↩︎](https://www.bookstack.cn/read/draveness-golang/1a4f7a284cd2b279.md#fnref:6)



## 5.3 defer

> [5.3 defer](https://www.bookstack.cn/read/draveness-golang/f434f07b7b465a9f.md)

很多现代的编程语言中都有 `defer` 关键字，Go 语言的 `defer` 会在当前函数或者方法返回之前执行传入的函数。它会经常被用于关闭文件描述符、关闭数据库连接以及解锁资源。

在这一节中我们就会深入 Go 语言的源代码介绍 `defer` 关键字的实现原理，相信读者读完这一节会对 `defer` 的数据结构、实现以及调用过程有着更清晰的理解。

作为一个编程语言中的关键字，`defer` 的实现一定是由编译器和运行时共同完成的，不过在深入源码分析它的实现之前我们还是需要了解 `defer` 关键字的常见使用场景以及使用时的注意事项。

使用 `defer` 的最常见场景就是在函数调用结束后完成一些收尾工作，例如在 `defer` 中回滚数据库的事务：

```
func createPost(db *gorm.DB) error {
    tx := db.Begin()
    defer tx.Rollback()
    if err := tx.Create(&Post{Author: "Draveness"}).Error; err != nil {
        return err
    }
    return tx.Commit().Error
}
```

在使用数据库事务时，我们可以使用如上所示的代码在创建事务之后就立刻调用 `Rollback` 保证事务一定会回滚。哪怕事务真的执行成功了，那么调用 `tx.Commit()` 之后再执行 `tx.Rollback()` 也不会影响已经提交的事务。



### 5.3.1 现象

我们在 Go 语言中使用 `defer` 时会遇到两个比较常见的问题，这里会介绍具体的场景并分析这两个现象背后的设计原理：

- `defer` 关键字的调用时机以及多次调用 `defer` 时执行顺序是如何确定的；（先进后出，类似栈）
- `defer` 关键字使用传值的方式传递参数时会进行**预计算**，导致不符合预期的结果；

#### 作用域

向 `defer` 关键字传入的函数会在函数返回之前运行。假设我们在 `for` 循环中多次调用 `defer` 关键字：

```go
func main() {
    for i := 0; i < 5; i++ {
        defer fmt.Println(i)
    }
}
$ go run main.go
4
3
2
1
0
```

运行上述代码会倒序执行所有向 `defer` 关键字中传入的表达式，最后一次 `defer` 调用传入了 `fmt.Println(4)`，所以会这段代码会优先打印 4。我们可以通过下面这个简单例子强化对 `defer` 执行时机的理解：

```
func main() {
    {
        defer fmt.Println("defer runs")
        fmt.Println("block ends")
    }
    fmt.Println("main ends")
}
$ go run main.go
block ends
main ends
defer runs
```

从上述代码的输出我们会发现，`defer` 传入的函数不是在退出代码块的作用域时执行的，它只会在当前函数和方法返回之前被调用。



#### 预计算参数

Go 语言中所有的函数调用都是传值的，`defer` 虽然是关键字，但是也继承了这个特性。假设我们想要计算 `main` 函数运行的时间，可能会写出以下的代码：

```
func main() {
    startedAt := time.Now()
    defer fmt.Println(time.Since(startedAt))
    time.Sleep(time.Second)
}
$ go run main.go
0s
```

然而上述代码的运行结果并不符合我们的预期，这个现象背后的原因是什么呢？经过分析，**我们会发现调用 `defer` 关键字会立刻对函数中引用的外部参数进行拷贝**，所以 `time.Since(startedAt)` 的结果不是在 `main` 函数退出之前计算的，而是在 `defer` 关键字调用时计算的，最终导致上述代码输出 0s。



想要解决这个问题的方法非常简单，我们只需要向 `defer` 关键字传入匿名函数：

```go
func main() {
    startedAt := time.Now()
    defer func() { fmt.Println(time.Since(startedAt)) }()
    time.Sleep(time.Second)
}
$ go run main.go
1s
```

虽然调用 `defer` 关键字时也使用值传递，但是因为拷贝的是函数指针，所以 `time.Since(startedAt)` 会在 `main` 函数执行前被调用并打印出符合预期的结果。



### 5.3.2 数据结构

在介绍 `defer` 函数的执行过程与实现原理之前，我们首先来了解一下 `defer` 关键字在 Go 语言源代码中对应的数据结构：

```go
type _defer struct {
    siz     int32
    started bool
    sp      uintptr
    pc      uintptr
    fn      *funcval
    _panic  *_panic
    link    *_defer
}
```

[`runtime._defer`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L853-L878) 结构体是延迟调用链表上的一个元素，所有的结构体都会通过 `link` 字段串联成链表。

![golang-defer-link](go语言设计与实现.assets/9aec08defae33c08897ca9b3a4f4c0d3.png)

**图 5-10 延迟调用链表**

我们简单介绍一下 [`runtime._defer`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L853-L878) 结构体中的几个字段：

- `siz` 是参数和结果的内存大小；
- `sp` 和 `pc` 分别代表栈指针和调用方的程序计数器；
- `fn` 是 `defer` 关键字中传入的函数；
- `_panic` 是触发延迟调用的结构体，可能为空；

除了上述的这些字段之外，[`runtime._defer`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L853-L878) 中还包含一些垃圾回收机制使用的字段，这里为了减少理解的成本就都省去了。



## 5.3.3 编译过程

中间代码生成阶段执行的被 [`cmd/compile/internal/gc.state.stmt`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/ssa.go#L1023-L1502) 函数会处理 `defer` 关键字。从下面截取的这段代码中，我们会发现编译器调用了 [`cmd/compile/internal/gc.state.call`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/ssa.go#L4324-L4517) 函数，这表示 `defer` 在编译器看来也是函数调用：

```go
func (s *state) stmt(n *Node) {
    switch n.Op {
    case ODEFER:
        s.call(n.Left, callDefer)
    }
}
```

[`cmd/compile/internal/gc.state.call`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/ssa.go#L4324-L4517) 函数会负责了为所有函数和方法调用生成中间代码，它的工作包括以下内容：

- 获取需要执行的函数名、闭包指针、代码指针和函数调用的接收方；
- 获取栈地址并将函数或者方法的参数写入栈中；
- 使用 [`cmd/compile/internal/gc.state.newValue1A`](https://github.com/golang/go/blob/4d5bb9c60905b162da8b767a8a133f6b4edcaa65/src/cmd/compile/internal/gc/ssa.go#L764-L766) 以及相关函数生成函数调用的中间代码；
- 如果当前调用的函数是 `defer`，那么就会单独生成相关的结束代码块；
- 获取函数的返回值地址并结束当前调用；

```
func (s *state) call(n *Node, k callKind) *ssa.Value {
    ...
    var call *ssa.Value
    switch {
    case k == callDefer:
        call = s.newValue1A(ssa.OpStaticCall, types.TypeMem, deferproc, s.mem())
    ...
    }
    call.AuxInt = stksize
    s.vars[&memVar] = call
    ...
}
```

从上述代码中我们能看到，`defer` 关键字在运行期间会调用 [`runtime.deferproc`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L218-L258) 函数，这个函数接收了参数的大小和闭包所在的地址两个参数。

编译器不仅将 `defer` 关键字都转换成 [`runtime.deferproc`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L218-L258) 函数，它还会通过以下三个步骤为所有调用 `defer` 的函数末尾插入 [`runtime.deferreturn`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L526-L571) 的函数调用：

- [`cmd/compile/internal/gc.walkstmt`](https://github.com/golang/go/blob/316fd8cc4a7fab2e1bb45848bc30ea8b8a0b231a/src/cmd/compile/internal/gc/walk.go#L115-L351) 在遇到 `ODEFER` 节点时会执行 `Curfn.Func.SetHasDefer(true)` 设置当前函数的 `hasdefer`；
- [`cmd/compile/internal/gc.buildssa`](https://github.com/golang/go/blob/98d2717499575afe13d9f815d46fcd6e384efb0c/src/cmd/compile/internal/gc/ssa.go#L281-L451) 会执行 `s.hasdefer = fn.Func.HasDefer()` 更新 `state` 的 `hasdefer`；
- [`cmd/compile/internal/gc.state.exit`](https://github.com/golang/go/blob/98d2717499575afe13d9f815d46fcd6e384efb0c/src/cmd/compile/internal/gc/ssa.go#L1511-L1552) 会根据 `state` 的 `hasdefer` 在函数返回之前插入 [`runtime.deferreturn`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L526-L571) 的函数调用；

```
func (s *state) exit() *ssa.Block {
    if s.hasdefer {
        s.rtcall(Deferreturn, true, nil)
    }
    ...
}
```

Go 语言的编译器不仅将 `defer` 转换成了 [`runtime.deferproc`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L218-L258) 的函数调用，还在所有调用 `defer` 的函数结尾插入了 [`runtime.deferreturn`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L526-L571)，接下来我们就需要了解这两个运行时方法的实现原理了。

### 5.3.4 运行过程

`defer` 关键字的运行时实现分成两个部分：

- [`runtime.deferproc`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L218-L258) 函数负责创建新的延迟调用；
- [`runtime.deferreturn`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L526-L571) 函数负责在函数调用结束时执行所有的延迟调用；

这两个函数是 `defer` 关键字运行时机制的入口，我们从它们开始分别介绍这两个函数的执行过程。

#### 创建延迟调用

[`runtime.deferproc`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L218-L258) 会为 `defer` 创建一个新的 [`runtime._defer`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L853-L878) 结构体、设置它的函数指针 `fn`、程序计数器 `sp` 和栈指针 `sp` 并将相关的参数拷贝到相邻的内存空间中：

```
func deferproc(siz int32, fn *funcval) {
    sp := getcallersp()
    argp := uintptr(unsafe.Pointer(&fn)) + unsafe.Sizeof(fn)
    callerpc := getcallerpc()
    d := newdefer(siz)
    if d._panic != nil {
        throw("deferproc: d.panic != nil after newdefer")
    }
    d.fn = fn
    d.pc = callerpc
    d.sp = sp
    switch siz {
    case 0:
    case sys.PtrSize:
        *(*uintptr)(deferArgs(d)) = *(*uintptr)(unsafe.Pointer(argp))
    default:
        memmove(deferArgs(d), unsafe.Pointer(argp), uintptr(siz))
    }
    return0()
}
```

最后调用的 [`runtime.return0`](https://github.com/golang/go/blob/a38a917aee626a9b9d5ce2b93964f586bf759ea0/src/runtime/asm_386.s#L1320-L1322) 函数的作用是避免无限递归调用 [`runtime.deferreturn`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L526-L571)，它是唯一一个不会触发由延迟调用的函数了。

[`runtime.deferproc`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L218-L258) 中 [`runtime.newdefer`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L383-L430) 的作用就是想尽办法获得一个 [`runtime._defer`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L853-L878) 结构体，办法总共有三个：

- 从调度器的延迟调用缓存池 `sched.deferpool` 中取出结构体并将该结构体追加到当前 Goroutine 的缓存池中；
- 从 Goroutine 的延迟调用缓存池 `pp.deferpool` 中取出结构体；
- 通过 [`runtime.mallocgc`](https://github.com/golang/go/blob/5042317d6919d4c84557e04be35130430e8d1dd4/src/runtime/malloc.go#L889-L1132) 创建一个新的结构体；

```
func newdefer(siz int32) *_defer {
    var d *_defer
    sc := deferclass(uintptr(siz))
    gp := getg()
    if sc < uintptr(len(p{}.deferpool)) {
        pp := gp.m.p.ptr()
        if len(pp.deferpool[sc]) == 0 && sched.deferpool[sc] != nil {
            for len(pp.deferpool[sc]) < cap(pp.deferpool[sc])/2 && sched.deferpool[sc] != nil {
                d := sched.deferpool[sc]
                sched.deferpool[sc] = d.link
                pp.deferpool[sc] = append(pp.deferpool[sc], d)
            }
        }
        if n := len(pp.deferpool[sc]); n > 0 {
            d = pp.deferpool[sc][n-1]
            pp.deferpool[sc][n-1] = nil
            pp.deferpool[sc] = pp.deferpool[sc][:n-1]
        }
    }
    if d == nil {
        total := roundupsize(totaldefersize(uintptr(siz)))
        d = (*_defer)(mallocgc(total, deferType, true))
    }
    d.siz = siz
    d.link = gp._defer
    gp._defer = d
    return d
}
```

无论使用哪种方式获取 [`runtime._defer`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L853-L878)，它都会被追加到所在的 Goroutine `_defer` 链表的**最前面**。

![golang-new-defe](go语言设计与实现.assets/83655588e61028c6c0f7c8b931101ce0.png)

**图 5-11 追加新的延迟调用**

`defer` 关键字插入时是从后向前的，而 `defer` 关键字执行是从前向后的，而这就是后调用的 `defer` 会优先执行的原因。

#### 执行延迟调用

[`runtime.deferreturn`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L526-L571) 会从 Goroutine 的 `_defer` 链表中取出最前面的 [`runtime._defer`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L853-L878) 结构体并调用 [`runtime.jmpdefer`](https://github.com/golang/go/blob/a38a917aee626a9b9d5ce2b93964f586bf759ea0/src/runtime/asm_386.s#L614-L624) 函数传入需要执行的函数和参数：

```
func deferreturn(arg0 uintptr) {
    gp := getg()
    d := gp._defer
    if d == nil {
        return
    }
    sp := getcallersp()
    switch d.siz {
    case 0:
    case sys.PtrSize:
        *(*uintptr)(unsafe.Pointer(&arg0)) = *(*uintptr)(deferArgs(d))
    default:
        memmove(unsafe.Pointer(&arg0), deferArgs(d), uintptr(d.siz))
    }
    fn := d.fn
    gp._defer = d.link
    freedefer(d)
    jmpdefer(fn, uintptr(unsafe.Pointer(&arg0)))
}
```

[`runtime.jmpdefer`](https://github.com/golang/go/blob/a38a917aee626a9b9d5ce2b93964f586bf759ea0/src/runtime/asm_386.s#L614-L624) 是一个用汇编语言实现的运行时函数，它的工作就是跳转 `defer` 所在的代码段并在执行结束之后跳转回 [`runtime.deferreturn`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L526-L571)。

```
TEXT runtime·jmpdefer(SB), NOSPLIT, $0-8
    MOVL    fv+0(FP), DX    // fn
    MOVL    argp+4(FP), BX    // caller sp
    LEAL    -4(BX), SP    // caller sp after CALL
#ifdef GOBUILDMODE_shared
    SUBL    $16, (SP)    // return to CALL again
#else
    SUBL    $5, (SP)    // return to CALL again
#endif
    MOVL    0(DX), BX
    JMP    BX    // but first run the deferred function
```

[`runtime.deferreturn`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L526-L571) 函数会多次判断当前 Goroutine 的 `_defer` 链表中是否有未执行的剩余结构，在所有的延迟函数调用都执行完成之后，该函数才会返回。

### 5.3.5 小结

`defer` 关键字的实现主要依靠编译器和运行时的协作，我们总结一下本节提到的内容：

- 编译期；
  - 将 `defer` 关键字被转换 [`runtime.deferproc`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L218-L258)；
  - 在调用 `defer` 关键字的函数返回之前插入 [`runtime.deferreturn`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L526-L571)；
- 运行时：
  - [`runtime.deferproc`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L218-L258) 会将一个新的 [`runtime._defer`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L853-L878) 结构体追加到当前 Goroutine 的链表头；
  - [`runtime.deferreturn`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L526-L571) 会从 Goroutine 的链表中取出 [`runtime._defer`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L853-L878) 结构并依次执行；

我们在本节前面提到的两个现象在这里也可以解释清楚了：

- 后调用的defer函数会先执行：
  - 后调用的 `defer` 函数会被追加到 Goroutine `_defer` 链表的最前面；
  - 运行 [`runtime._defer`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L853-L878) 时是从前到后依次执行；
- 函数的参数会被预先计算；
  - 调用 [`runtime.deferproc`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L218-L258) 函数创建新的延迟调用时就会立刻拷贝函数的参数，函数的参数不会等到真正执行时计算；



## 5.4 panic 和 recover

> [5.4 panic 和 recover](https://www.bookstack.cn/read/draveness-golang/ada6c076f55fa1aa.md)

本节将分析两个经常成对出现的关键字 `panic` 和 `recover`。这两个关键字都与 `defer` 有千丝万缕的联系，也都是 Go 语言中的内置函数，但是提供的功能却是互补的：

- `panic` 能够改变程序的控制流，函数调用`panic` 时会立刻停止执行函数的其他代码，并在执行结束后在当前 Goroutine 中递归执行调用方的延迟函数调用 `defer`；
- `recover` 可以中止 `panic` 造成的程序崩溃。它是一个只能在 `defer` 中发挥作用的函数，在其他作用域中调用不会发挥任何作用；

![golang-panic](go语言设计与实现.assets/5d892e850800f100fa788c91b70a2f40.png)

**图 5-12 panic 触发的递归延迟调用**

Andrew Gerrand 写过的一篇名为 [Defer, Panic, and Recover](https://blog.golang.org/defer-panic-and-recover) 的博客很好地介绍了这三个关键字的不同作用以及它们的关系[1](https://www.bookstack.cn/read/draveness-golang/ada6c076f55fa1aa.md#fn:1)。

### 5.4.1 现象

我们先通过几个例子了解一下使用 `panic` 和 `recover` 关键字时遇到的一些现象，部分现象也与上一节分析的 `defer` 关键字有关：

- `panic` 只会触发当前 Goroutine 的延迟函数调用；
- `recover` 只有在 `defer` 函数中调用才会生效；
- `panic` 允许在 `defer` 中嵌套多次调用；

#### 跨协程失效

首先要展示的例子就是 `panic` 只会触发当前 Goroutine 的延迟函数调用。这里有一段简单的代码：

```go
func main() {
    defer println("in main")
    go func() {
        defer println("in goroutine")
        panic("")
    }()
    time.Sleep(1 * time.Second)
}
$ go run main.go
in goroutine
panic:

goroutine 17 [running]:
main.main.func1()
	/xxxxx/go_test/test.go:39 +0x45
created by main.main
	/xxxx/go_test/test.go:37 +0x3c
```

当我们运行这段代码时会发现 `main` 函数中的 `defer` 语句并没有执行，执行的只有当前 Goroutine 中的 `defer`。

上一节我们曾经介绍过 `defer` 关键字对应的 [`runtime.deferproc`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L218-L258) 会将延迟调用函数与调用方所在 Goroutine 进行关联。所以当程序发生崩溃时只会调用当前 Goroutine 的延迟调用函数也是非常合理的。

![golang-panic-and-defers](go语言设计与实现.assets/948d3fa4c62cf6c9b1c369375bc18221.png)

**图 5-13 panic 触发当前 Goroutine 的延迟调用**

如上图所示，多个 Goroutine 之间没有太多的关联，一个 Goroutine 在 `panic` 时也不应该执行其他 Goroutine 的延迟函数。



#### 失效的崩溃恢复

初学 Go 语言的读者可能会写出下面的代码，在主程序中调用 `recover` 试图中止程序的崩溃，但是从运行的结果中我们也能看出，如下所示的程序依然没有正常退出。

```go
func main() {
    defer fmt.Println("in main")
    if err := recover(); err != nil {
        fmt.Println(err)
    }
    panic("unknown err")
}
$ go run main.go
in main
panic: unknown err
goroutine 1 [running]:
main.main()
    ...
exit status 2
```

仔细分析一下这个过程就能理解这种现象背后的原因，`recover` 只有在发生 `panic` 之后调用才会生效。然而在上面的控制流中，`recover` 是在 `panic` 之前调用的，并不满足生效的条件，所以我们需要在 `defer` 中使用 `recover` 关键字。

```go
func main() {
	defer func() {
		fmt.Println("in main")
		if err := recover(); err != nil {
			fmt.Println("here", err)
		}
	}()
	panic("unknown err")
}

==>
in main
here unknown err

```

#### 嵌套崩溃

Go 语言中的 `panic` 是可以多次嵌套调用的。一些熟悉 Go 语言的读者很可能也不知道这个知识点，如下所示的代码就展示了如何在 `defer` 函数中多次调用 `panic`：

```go
func main() {
    defer fmt.Println("in main")
    defer func() {
        defer func() {
            panic("panic again and again")
        }()
        panic("panic again")
    }()
    panic("panic once")
}
$ go run main.go
in main
panic: panic once
    panic: panic again
    panic: panic again and again
goroutine 1 [running]:
...
exit status 2
```

从上述程序的输出，**我们可以确定程序多次调用 `panic` 也不会影响 `defer` 函数的正常执行。所以使用 `defer` 进行收尾的工作一般来说都是安全的**。

### 5.4.2 数据结构

`panic` 关键字在 Go 语言的源代码是由数据结构 [`runtime._panic`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L891-L900) 表示的。每当我们调用 `panic` 都会创建一个如下所示的数据结构存储相关信息：

```
type _panic struct {
    argp      unsafe.Pointer
    arg       interface{}
    link      *_panic
    recovered bool
    aborted   bool
    pc        uintptr
    sp        unsafe.Pointer
    goexit    bool
}
```

- `argp` 是指向 `defer` 调用时参数的指针；
- `arg` 是调用 `panic` 时传入的参数；
- `link` 指向了更早调用的 [`runtime._panic`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L891-L900) 结构；
- `recovered` 表示当前 [`runtime._panic`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L891-L900) 是否被 `recover` 恢复；
- `aborted` 表示当前的 `panic` 是否被强行终止；从数据结构中的 `link` 字段我们就可以推测出以下的结论 — `panic` 函数可以被连续多次调用，它们之间通过 `link` 的关联形成一个链表。

结构体中的 `pc`、`sp` 和 `goexit` 三个字段都是为了修复 [`runtime.Goexit`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L581-L656) 的问题引入的[2](https://www.bookstack.cn/read/draveness-golang/ada6c076f55fa1aa.md#fn:2)。该函数能够只结束调用该函数的 Goroutine 而不影响其他的 Goroutine，但是该函数会被 `defer` 中的 `panic` 和 `recover` 取消[3](https://www.bookstack.cn/read/draveness-golang/ada6c076f55fa1aa.md#fn:3)，引入这三个字段的目的就是为了解决这个问题。

### 5.4.3 程序崩溃

首先了解一下 `panic` 函数是如何终止程序的。编译器会将关键字 `panic` 转换成 [`runtime.gopanic`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L887-L1062)，该函数的执行过程包含以下几个步骤：

- 创建新的 [`runtime._panic`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L891-L900) 结构并添加到所在 Goroutine `_panic` 链表的最前面；
- 在循环中不断从当前 Goroutine 的 `_defer` 中链表获取 [`runtime._defer`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L853-L878) 并调用 [`runtime.reflectcall`](https://github.com/golang/go/blob/a38a917aee626a9b9d5ce2b93964f586bf759ea0/src/runtime/asm_386.s#L496-L526) 运行延迟调用函数；
- 调用 [`runtime.fatalpanic`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L1185-L1220) 中止整个程序；

```
func gopanic(e interface{}) {
    gp := getg()
    ...
    var p _panic
    p.arg = e
    p.link = gp._panic
    gp._panic = (*_panic)(noescape(unsafe.Pointer(&p)))
    for {
        d := gp._defer
        if d == nil {
            break
        }
        d._panic = (*_panic)(noescape(unsafe.Pointer(&p)))
        reflectcall(nil, unsafe.Pointer(d.fn), deferArgs(d), uint32(d.siz), uint32(d.siz))
        d._panic = nil
        d.fn = nil
        gp._defer = d.link
        freedefer(d)
        if p.recovered {
            ...
        }
    }
    fatalpanic(gp._panic)
    *(*int)(nil) = 0
}
```

需要注意的是，我们在上述函数中省略了三部分比较重要的代码：

- 恢复程序的 `recover` 分支中的代码；
- 通过内联优化defer调用性能的代码4；
  - [runtime: make defers low-cost through inline code and extra funcdata](https://github.com/golang/go/commit/be64a19d99918c843f8555aad580221207ea35bc)
- 修复`runtime.Goexit`异常情况的代码；
  - [runtime: ensure that Goexit cannot be aborted by a recursive panic/recover](https://github.com/golang/go/commit/7dcd343ed641d3b70c09153d3b041ca3fe83b25e)[`runtime.fatalpanic`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L1185-L1220) 实现了无法被恢复的程序崩溃，它在中止程序之前会通过 [`runtime.printpanics`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L679-L695) 打印出全部的 `panic` 消息以及调用时传入的参数：

```
func fatalpanic(msgs *_panic) {
    pc := getcallerpc()
    sp := getcallersp()
    gp := getg()
    if startpanic_m() && msgs != nil {
        atomic.Xadd(&runningPanicDefers, -1)
        printpanics(msgs)
    }
    if dopanic_m(gp, pc, sp) {
        crash()
    }
    exit(2)
}
```

打印 `panic` 消息之后会通过 [`runtime.exit`](https://github.com/golang/go/blob/cbaa666682386fe5350bf87d7d70171704c90fe4/src/runtime/sys_darwin.go#L231-L233) 退出当前程序并返回错误码 2，而程序的正常退出也是通过 [`runtime.exit`](https://github.com/golang/go/blob/cbaa666682386fe5350bf87d7d70171704c90fe4/src/runtime/sys_darwin.go#L231-L233) 函数实现的。

### 5.4.4 崩溃恢复

到这里我们已经掌握了 `panic` 退出程序的过程，接下来将分析 `defer` 中的 `recover` 是如何中止程序崩溃的。编译器会将关键字 `recover` 转换成 [`runtime.gorecover`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L1080-L1094)：

```go
func gorecover(argp uintptr) interface{} {
    p := gp._panic
    if p != nil && !p.recovered && argp == uintptr(p.argp) {
        p.recovered = true
        return p.arg
    }
    return nil
}
```

这个函数的实现非常简单，如果当前 Goroutine 没有调用 `panic`，那么该函数会直接返回 `nil`，这也是崩溃恢复在非 `defer` 中调用会失效的原因。

在正常情况下，它会修改 [`runtime._panic`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L891-L900) 结构体的 `recovered` 字段，[`runtime.gorecover`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L1080-L1094) 函数本身不包含恢复程序的逻辑，程序的恢复也是由 [`runtime.gopanic`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L887-L1062) 函数负责的：

```go
func gopanic(e interface{}) {
    ...
    for {
        // 执行延迟调用函数，可能会设置 p.recovered = true
        ...
        pc := d.pc
        sp := unsafe.Pointer(d.sp)
        ...
        if p.recovered {
            gp._panic = p.link
            for gp._panic != nil && gp._panic.aborted {
                gp._panic = gp._panic.link
            }
            if gp._panic == nil {
                gp.sig = 0
            }
            gp.sigcode0 = uintptr(sp)
            gp.sigcode1 = pc
            mcall(recovery)
            throw("recovery failed")
        }
    }
    ...
}
```

上述这段代码也省略了 `defer` 的内联优化，它从 [`runtime._defer`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L853-L878) 结构体中取出了程序计数器 `pc` 和栈指针 `sp` 并调用 [`runtime.recovery`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L1132-L1151) 函数触发 Goroutine 的调度，调度之前会准备好 `sp`、`pc` 以及函数的返回值：

```
func recovery(gp *g) {
    sp := gp.sigcode0
    pc := gp.sigcode1
    gp.sched.sp = sp
    gp.sched.pc = pc
    gp.sched.lr = 0
    gp.sched.ret = 1
    gogo(&gp.sched)
}
```

当我们在调用 `defer` 关键字时，调用时的栈指针 `sp` 和程序计数器 `pc` 就已经存储到了 [`runtime._defer`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L853-L878) 结构体中，这里的 [`runtime.gogo`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/asm_386.s#L301-L314) 函数会跳回 `defer` 关键字调用的位置。

[`runtime.recovery`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L1132-L1151) 在调度过程中会将函数的返回值设置成 1。从 [`runtime.deferproc`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L218-L258) 的注释中我们会发现，当 [`runtime.deferproc`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L218-L258) 函数的返回值是 1 时，编译器生成的代码会直接跳转到调用方函数返回之前并执行 [`runtime.deferreturn`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L526-L571)：

```
func deferproc(siz int32, fn *funcval) {
    ...
    // deferproc returns 0 normally.
    // a deferred func that stops a panic
    // makes the deferproc return 1.
    // the code the compiler generates always
    // checks the return value and jumps to the
    // end of the function if deferproc returns != 0.
    return0()
}
```

跳转到 [`runtime.deferreturn`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L526-L571) 函数之后，程序就已经从 `panic` 中恢复了并执行正常的逻辑，而 [`runtime.gorecover`](https://github.com/golang/go/blob/22d28a24c8b0d99f2ad6da5fe680fa3cfa216651/src/runtime/panic.go#L1080-L1094) 函数也能从 [`runtime._panic`](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L891-L900) 结构体中取出了调用 `panic` 时传入的 `arg` 参数并返回给调用方。



## 5.4.5 小结

分析程序的崩溃和恢复过程比较棘手，代码不是特别容易理解。我们在本节的最后还是简单总结一下程序崩溃和恢复的过程：

- 编译器会负责做转换关键字的工作；
  - 将 `panic` 和 `recover` 分别转换成 `runtime.gopanic` 和 `runtime.gorecover`；
  - 将 `defer` 转换成 `deferproc` 函数；
  - 在调用 `defer` 的函数末尾调用 `deferreturn` 函数；
- 在运行过程中遇到 `gopanic` 方法时，会从 Goroutine 的链表依次取出 `_defer` 结构体并执行；
- 如果调用延迟执行函数时遇到了`gorecover`就会将`_panic.recovered`标记成true并返回panic的参数；
  - 在这次调用结束之后，`gopanic` 会从 `_defer` 结构体中取出程序计数器 `pc` 和栈指针 `sp` 并调用 `recovery` 函数进行恢复程序；
  - `recovery` 会根据传入的 `pc` 和 `sp` 跳转回 `deferproc`；
  - 编译器自动生成的代码会发现 `deferproc` 的返回值不为 0，这时会跳回 `deferreturn` 并恢复到正常的执行流程；
- 如果没有遇到 `gorecover` 就会依次遍历所有的 `_defer` 结构，并在最后调用 `fatalpanic` 中止程序、打印 `panic` 的参数并返回错误码 `2`；分析的过程涉及了很多语言底层的知识，源代码阅读起来也比较晦涩，其中充斥着反常规的控制流程，通过程序计数器来回跳转，不过对于我们理解程序的执行流程还是很有帮助。

------

- Andrew Gerrand. 4 August 2010. Defer, Panic, and Recover - The Go Blog. https://blog.golang.org/defer-panic-and-recover[↩︎](https://www.bookstack.cn/read/draveness-golang/ada6c076f55fa1aa.md#fnref:1)
- Dan Scales. Oct 10, 2019. runtime: ensure that Goexit cannot be aborted by a recursive panic/recover https://github.com/golang/go/commit/7dcd343ed641d3b70c09153d3b041ca3fe83b25e[↩︎](https://www.bookstack.cn/read/draveness-golang/ada6c076f55fa1aa.md#fnref:2)
- runtime: panic + recover can cancel a call to Goexit #29226 https://github.com/golang/go/issues/29226[↩︎](https://www.bookstack.cn/read/draveness-golang/ada6c076f55fa1aa.md#fnref:3)
- Dan Scales. Jun 25, 2019. cmd/compile, cmd/link, runtime: make defers low-cost through inline code and extra funcdata https://github.com/golang/go/commit/be64a19d99918c843f8555aad580221207ea35bc[↩︎](https://www.bookstack.cn/read/draveness-golang/ada6c076f55fa1aa.md#fnref:4)



## 5.5 make 和 new

> [5.5 make 和 new](https://www.bookstack.cn/read/draveness-golang/bb191ed6f01d91ba.md)

当我们想要在 Go 语言中初始化一个结构时，可能会用到两个不同的关键字 — `make` 和 `new`。因为它们的功能相似，所以初学者可能会对这两个关键字的作用感到困惑[1](https://www.bookstack.cn/read/draveness-golang/bb191ed6f01d91ba.md#fn:1)，但是它们两者能够初始化的却有较大的不同。

- `make` 的作用是初始化内置的数据结构，也就是我们在前面提到的切片、哈希表和 Channel[2](https://www.bookstack.cn/read/draveness-golang/bb191ed6f01d91ba.md#fn:2)；

- `new` 的作用是根据传入的类型在堆上分配一片内存空间并返回指向这片内存空间的指针[3](https://www.bookstack.cn/read/draveness-golang/bb191ed6f01d91ba.md#fn:3)；

我们在代码中往往都会使用如下所示的语句初始化这三类基本类型，这三个语句分别返回了不同类型的数据结构：

```
slice := make([]int, 0, 100)
hash := make(map[int]bool, 10)
ch := make(chan int, 5)
```

- `slice` 是一个包含 `data`、`cap` 和 `len` 的私有结构体 [`internal/reflectlite.sliceHeader`](https://github.com/golang/go/blob/a5026af57c7934f0856cfd4b539a7859d85a0474/src/internal/reflectlite/value.go#L389-L393)；
- `hash` 是一个指向 [`runtime.hmap`](https://github.com/golang/go/blob/36f30ba289e31df033d100b2adb4eaf557f05a34/src/runtime/map.go#L115-L129) 结构体的指针；
- `ch` 是一个指向 [`runtime.hchan`](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/chan.go#L32-L51) 结构体的指针；相比与复杂的 `make` 关键字，`new` 的功能就很简单了，它只能接收一个类型作为参数然后返回一个指向该类型的指针：

```
i := new(int)


var v int
i := &v
```



上述代码片段中的两种不同初始化方法是等价的，它们都会创建一个指向 `int` 零值的指针。

![golang-make-and-ne](go语言设计与实现.assets/07429a08ed9d25de704618353b118176.png)

**图 5-14 make 和 new 初始化的类型**

接下来我们将分别介绍 `make` 和 `new` 在初始化不同数据结构时的过程，我们会从编译期间和运行时两个不同阶段理解这两个关键字的原理，不过由于前面的章节已经详细地分析过 `make` 的原理，所以这里会将重点放在另一个关键字 `new` 上。



### 5.5.1 make

在前面的章节中我们已经谈到过 `make` 在创建切片、哈希表和 Channel 的具体过程，所以在这一小节，我们只是会简单提及 `make` 相关的数据结构的初始化原理。

![golang-make-typecheck](go语言设计与实现.assets/d2900eb45fe1e3c2dbfa0d67ca510c7c.png)

**图 5-15 make 关键字的类型检查**

在编译期间的[类型检查](https://www.bookstack.cn/read/draveness-golang/7ab240185c175c73.md)阶段，Go 语言就将代表 `make` 关键字的 `OMAKE` 节点根据参数类型的不同转换成了 `OMAKESLICE`、`OMAKEMAP` 和 `OMAKECHAN` 三种不同类型的节点，这些节点会调用不同的运行时函数来初始化相应的数据结构。



### 5.5.2 new

编译器会在[中间代码生成](https://www.bookstack.cn/read/draveness-golang/5a89e04c79706261.md)阶段通过以下两个函数处理该关键字：

- [`cmd/compile/internal/gc.callnew`](https://github.com/golang/go/blob/316fd8cc4a7fab2e1bb45848bc30ea8b8a0b231a/src/cmd/compile/internal/gc/walk.go#L1930-L1940) 函数会将关键字转换成 `ONEWOBJ` 类型的节点[2](https://www.bookstack.cn/read/draveness-golang/bb191ed6f01d91ba.md#fn:2)；

- `cmd/compile/internal/gc.state.expr`

   

  函数会根据申请空间的大小分两种情况处理：

  - 如果申请的空间为 0，就会返回一个表示空指针的 `zerobase` 变量；
  - 在遇到其他情况时会将关键字转换成 [`runtime.newobject`](https://github.com/golang/go/blob/5042317d6919d4c84557e04be35130430e8d1dd4/src/runtime/malloc.go#L1162-L1164) 函数：

```
func callnew(t *types.Type) *Node {
    ...
    n := nod(ONEWOBJ, typename(t), nil)
    ...
    return n
}
func (s *state) expr(n *Node) *ssa.Value {
    switch n.Op {
    case ONEWOBJ:
        if n.Type.Elem().Size() == 0 {
            return s.newValue1A(ssa.OpAddr, n.Type, zerobaseSym, s.sb)
        }
        typ := s.expr(n.Left)
        vv := s.rtcall(newobject, true, []*types.Type{n.Type}, typ)
        return vv[0]
    }
}
```

需要注意的是，无论是直接使用 `new`，还是使用 `var` 初始化变量，它们在编译器看来就是 `ONEW` 和 `ODCL` 节点。这些节点在这一阶段都会被 [`cmd/compile/internal/gc.walkstmt`](https://github.com/golang/go/blob/316fd8cc4a7fab2e1bb45848bc30ea8b8a0b231a/src/cmd/compile/internal/gc/walk.go#L115-L1532) 转换成通过 [`runtime.newobject`](https://github.com/golang/go/blob/5042317d6919d4c84557e04be35130430e8d1dd4/src/runtime/malloc.go#L1162-L1164) 函数在堆上申请内存：

```
func walkstmt(n *Node) *Node {
    switch n.Op {
    case ODCL:
        v := n.Left
        if v.Class() == PAUTOHEAP {
            if prealloc[v] == nil {
                prealloc[v] = callnew(v.Type)
            }
            nn := nod(OAS, v.Name.Param.Heapaddr, prealloc[v])
            nn.SetColas(true)
            nn = typecheck(nn, ctxStmt)
            return walkstmt(nn)
        }
    case ONEW:
        if n.Esc == EscNone {
            r := temp(n.Type.Elem())
            r = nod(OAS, r, nil)
            r = typecheck(r, ctxStmt)
            init.Append(r)
            r = nod(OADDR, r.Left, nil)
            r = typecheck(r, ctxExpr)
            n = r
        } else {
            n = callnew(n.Type.Elem())
        }
    }
}
```

不过这也不是绝对的，如果通过 `var` 或者 `new` 创建的变量不需要在当前作用域外生存，例如不用作为返回值返回给调用方，那么就不需要初始化在堆上。

[`runtime.newobject`](https://github.com/golang/go/blob/5042317d6919d4c84557e04be35130430e8d1dd4/src/runtime/malloc.go#L1162-L1164) 函数会是获取传入类型占用空间的大小，调用 [`runtime.mallocgc`](https://github.com/golang/go/blob/5042317d6919d4c84557e04be35130430e8d1dd4/src/runtime/malloc.go#L889-L1132) 在堆上申请一片内存空间并返回指向这片内存空间的指针：

```
func newobject(typ *_type) unsafe.Pointer {
    return mallocgc(typ.size, typ, true)
}
```



[`runtime.mallocgc`](https://github.com/golang/go/blob/5042317d6919d4c84557e04be35130430e8d1dd4/src/runtime/malloc.go#L889-L1132) 函数的实现大概有 200 多行代码，我们会在后面的章节中详细分析 Go 语言的内存管理机制。

### 5.5.3 小结

到了最后，简单总结一下 Go 语言中 `make` 和 `new` 关键字的实现原理，`make` 关键字的作用是创建切片、哈希表和 Channel 等内置的数据结构，而 `new` 的作用是为类型申请一片内存空间，并返回指向这片内存的指针。

- Make and new [https://groups.google.com/forum/#!topic/golang-nuts/kWXYU95XN04/discussion%5B1-25%5D](https://groups.google.com/forum/#!topic/golang-nuts/kWXYU95XN04/discussion[1-25])[↩︎](https://www.bookstack.cn/read/draveness-golang/bb191ed6f01d91ba.md#fnref:1)
- Allocation with make https://golang.org/doc/effective_go.html#allocation_make[↩︎](https://www.bookstack.cn/read/draveness-golang/bb191ed6f01d91ba.md#fnref:2)
- Allocation with new https://golang.org/doc/effective_go.html#allocation_new[↩︎](https://www.bookstack.cn/read/draveness-golang/bb191ed6f01d91ba.md#fnref:3)



## 5.6 推荐阅读

- [Go maps in action](https://blog.golang.org/go-maps-in-action)
- [SELECT(2) · Linux](http://man7.org/linux/man-pages/man2/select.2.html)





# 第六章 并发编程

## 6.1 上下文 Context

> [6.1 上下文 Context](https://www.bookstack.cn/read/draveness-golang/6ee8389651c21ac5.md)

`Context` 是 Golang 中非常有趣的设计，它与 Go 语言中的并发编程有着比较密切的关系，在其他语言中我们很难见到类似 `Context` 的东西，它不仅能够用来**设置截止日期**、**同步『信号』**还能用来**传递请求相关的值**。

这一节就会介绍 Go 语言中这个非常常见的 `Context` 接口，我们将从这里开始了解 Go 语言并发编程的设计理念以及实现原理。

### 概述

Go 语言中的每一个请求的都是通过一个单独的 Goroutine 进行处理的，HTTP/RPC 请求的处理器往往都会启动新的 Goroutine 访问数据库和 RPC 服务，我们可能会创建多个 Goroutine 来处理一次请求，而 `Context` 的主**要作用就是在不同的 Goroutine 之间同步请求特定的数据、取消信号以及处理请求的截止日期**。

![golang-context-usage](go语言设计与实现.assets/ee084778f956d8202c94c881926806bb.png)

每一个 `Context` 都会从最顶层的 Goroutine 一层一层传递到最下层，这也是 Golang 中上下文最常见的使用方式，如果没有 `Context`，当上层执行的操作出现错误时，下层其实不会收到错误而是会继续执行下去。


![golang-without-context](go语言设计与实现.assets/9c27d2f050f806c88e0808e8cceaf679.png)

当最上层的 Goroutine 因为某些原因执行失败时，下两层的 Goroutine 由于没有接收到这个信号所以会继续工作；但是当我们正确地使用 `Context` 时，就可以在下层及时停掉无用的工作减少额外资源的消耗：

![golang-with-context](go语言设计与实现.assets/16668310ee366c684edf21dc7fae3924.png)

这其实就是 Golang 中上下文的**最大作用，在不同 Goroutine 之间对信号进行同步避免对计算资源的浪费**，与此同时 `Context` 还能携带以请求为作用域的键值对信息。



#### 接口

`Context` 其实是 Go 语言 `context` 包对外暴露的接口，该接口定义了四个需要实现的方法，其中包括：

- `Deadline` 方法需要返回当前 `Context` 被取消的时间，也就是**完成工作的截止日期**；
- `Done` 方法需要**返回一个 Channel**，这个 Channel 会在当前工作完成或者上下文被取消之后关闭，多次调用 `Done` 方法会返回同一个 Channel；
- Err 方法会返回当前Context结束的原因，它只会在Done返回的 Channel 被关闭时才会返回非空的值；
  - 如果当前 `Context` 被取消就会返回 `Canceled` 错误；
  - 如果当前 `Context` 超时就会返回 `DeadlineExceeded` 错误；
- `Value` 方法会从 `Context` 中返回键对应的值，对于同一个上下文来说，多次调用 `Value` 并传入相同的 `Key` 会返回相同的结果，这个功能可以用来传递请求特定的数据；

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

`context` 包中提供的 `Background`、`TODO`、`WithDeadline` 等方法就会返回实现该接口的私有结构体的，我们会在后面的小节中详细介绍它们的工作原理。

#### 示例

我们可以通过一个例子简单了解一下 `Context` 是**如何对信号进行同步的**，在这段代码中我们创建了一个过期时间为 `1s` 的上下文，并将上下文传入 `handle` 方法，该方法会使用 `500ms` 的时间处理该『请求』：

```go
func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel()
    go handle(ctx, 500*time.Millisecond)
    select {
    case <-ctx.Done():
        fmt.Println("main", ctx.Err())
    }
}
func handle(ctx context.Context, duration time.Duration) {
    select {
    case <-ctx.Done():
        fmt.Println("handle", ctx.Err())
    case <-time.After(duration):
        fmt.Println("process request with", duration)
    }
}
```

所以我们有足够的时间处理该『请求』，而运行上述代码时会打印出如下所示的内容：

```bash
$ go run context.go
process request with 500ms
main context deadline exceeded
```

『请求』被 Goroutine 正常处理没有进入超时的 `select` 分支，但是在 `main` 函数中的 `select` 却会等待 `Context` 的超时最终打印出 `main context deadline exceeded`，如果我们将处理『请求』的时间改成 `1500ms`，当前处理的过程就会因为 `Context` 到截止日期而被中止：

```
$ go run context.go
handle context deadline exceeded
main context deadline exceeded

（无法确定子协程和main协程谁先接收到done信号）
```

两个函数都会因为 `ctx.Done()` 返回的管道被关闭而中止，也就是上下文超时。

相信这两个例子能够帮助各位读者了解 `Context` 的使用方法以及基本的工作原理 --------- 多个 Goroutine 同时订阅 `ctx.Done()` 管道中的消息，一旦接收到取消信号就停止当前正在执行的工作并提前返回。



### 实现原理

`Context` 相关的源代码都在 [context.go](https://golang.org/src/context/context.go) 这个文件中，在这一节中我们就会从 Go 语言的源代码出发介绍 `Context` 的实现原理，包括如何在多个 Goroutine 之间同步信号、为请求设置截止日期并传递参数和信息。

#### 默认上下文（Background）

在 `context` 包中，最常使用其实还是 `context.Background` 和 `context.TODO` 两个方法，这两个方法最终都会返回一个预先初始化好的私有变量 `background` 和 `todo`：

```
func Background() Context {
    return background
}
func TODO() Context {
    return todo
}
```

这两个变量是在包初始化时就被创建好的，它们都是通过 `new(emptyCtx)` 表达式初始化的指向私有结构体 `emptyCtx` 的指针，这是包中最简单也是最常用的类型：

```go
type emptyCtx int
func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
    return
}
func (*emptyCtx) Done() <-chan struct{} {
    return nil
}
func (*emptyCtx) Err() error {
    return nil
}
func (*emptyCtx) Value(key interface{}) interface{} {
    return nil
}
```

它对 `Context` 接口方法的实现也都非常简单，无论何时调用都会返回 `nil` 或者空值，并没有任何特殊的功能，`Background` 和 `TODO` 方法在某种层面上看其实也只是互为别名，两者没有太大的差别，不过 `context.Background()` 是上下文中最顶层的默认值，所有其他的上下文都应该从 `context.Background()` 演化出来。

![golang-context-hierarchy](go语言设计与实现.assets/996763ae96329802badedc359870af2a.png)

我们应该只在不确定时使用 `context.TODO()`，在多数情况下如果函数没有上下文作为入参，我们往往都会使用 `context.Background()` 作为起始的 `Context` 向下传递。



#### 取消信号（WithCancel）

`WithCancel` 方法能够从 `Context` 中创建出一个新的子上下文，同时还会返回用于取消该上下文的函数，也就是 `CancelFunc`，我们直接从 `WithCancel` 函数的实现来看它到底做了什么：

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
    c := newCancelCtx(parent)
    propagateCancel(parent, &c)
    return &c, func() { c.cancel(true, Canceled) }
}
```

`newCancelCtx` 是包中的私有方法，它将传入的父上下文包到私有结构体 `cancelCtx{Context: parent}` 中，`cancelCtx` 就是当前函数最终会返回的结构体类型，我们在详细了解它是如何实现接口之前，先来了解一下用于传递取消信号的 `propagateCancel` 函数：

```go
func propagateCancel(parent Context, child canceler) {
    if parent.Done() == nil {
        return // parent is never canceled
    }
    if p, ok := parentCancelCtx(parent); ok {
        p.mu.Lock()
        if p.err != nil {
            child.cancel(false, p.err)
        } else {
            if p.children == nil {
                p.children = make(map[canceler]struct{})
            }
            p.children[child] = struct{}{}
        }
        p.mu.Unlock()
    } else {
        go func() {
            select {
            case <-parent.Done():
                child.cancel(false, parent.Err())
            case <-child.Done():
            }
        }()
    }
}
```

该函数总共会处理与父上下文相关的三种不同的情况：

- 当 `parent.Done() == nil`，也就是 `parent` 不会触发取消事件时，当前函数直接返回；
- 当child的继承链上有parent是可以取消的上下文时，就会判断parent是否已经触发了取消信号；
  - 如果已经被取消，当前 `child` 就会立刻被取消；
  - 如果没有被取消，当前 `child` 就会被加入 `parent` 的 `children` 列表中，等待 `parent` 释放取消信号；
- 遇到其他情况就会开启一个新的 Goroutine，同时监听 `parent.Done()` 和 `child.Done()` 两个管道并在前者结束后立刻调用 `child.cancel` 取消子上下文；这个函数的主要作用就是在 `parent` 和 `child` 之间同步取消和结束的信号，保证在 `parent` 被取消时，`child` 也会收到对应的信号，不会发生状态不一致的问题。

`cancelCtx` 实现的几个接口方法其实没有太多值得介绍的地方，**该结构体最重要的方法其实是 `cancel` 方法，这个方法会关闭上下文的管道并向所有的子上下文发送取消信号**：

```go
func (c *cancelCtx) cancel(removeFromParent bool, err error) {
    c.mu.Lock()
    if c.err != nil {
        c.mu.Unlock()
        return
    }
    c.err = err
    if c.done == nil {
        c.done = closedchan
    } else {
        close(c.done)
    }
    for child := range c.children {
        child.cancel(false, err)
    }
    c.children = nil
    c.mu.Unlock()
    if removeFromParent {
        removeChild(c.Context, c)
    }
}
```

除了 `WithCancel` 之外，`context` 包中的另外两个函数 `WithDeadline` 和 `WithTimeout` 也都能创建可以被取消的上下文，`WithTimeout` 只是 `context` 包为我们提供的便利方法，能让我们更方便地创建 `timerCtx`

```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
    return WithDeadline(parent, time.Now().Add(timeout))
}
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
    if cur, ok := parent.Deadline(); ok && cur.Before(d) {
        return WithCancel(parent)
    }
    c := &timerCtx{
        cancelCtx: newCancelCtx(parent),
        deadline:  d,
    }
    propagateCancel(parent, c)
    dur := time.Until(d)
    if dur <= 0 {
        c.cancel(true, DeadlineExceeded) // deadline has already passed
        return c, func() { c.cancel(false, Canceled) }
    }
    c.mu.Lock()
    defer c.mu.Unlock()
    if c.err == nil {
        c.timer = time.AfterFunc(dur, func() {
            c.cancel(true, DeadlineExceeded)
        })
    }
    return c, func() { c.cancel(true, Canceled) }
}
```

`WithDeadline` 方法在创建 `timerCtx` 上下文的过程中，判断了上下文的截止日期与当前日期，并通过 `time.AfterFunc` 方法创建了定时器，当时间超过了截止日期之后就会调用 `cancel` 方法同步取消信号。

使用示例：

```go
func main() {
	// 创建一个截止时间为 5 秒后的上下文
	ctx, cancel := context.WithDeadline(context.Background(), time.Now().Add(5*time.Second))
	defer cancel()

	// 模拟一个长时间运行的任务
	go longRunningTask(ctx)

	// 等待一段时间，模拟主程序运行
	time.Sleep(10 * time.Second)
}
func longRunningTask(ctx context.Context) {
	select {
	case <-ctx.Done():
		fmt.Println("Task canceled due to deadline exceeded")
		return
	case <-time.After(7 * time.Second):
		fmt.Println("Task completed successfully")
	}
}

```



`timerCtx` 结构体内部嵌入了一个 `cancelCtx` 结构体，也『继承』了相关的变量和方法，除此之外，持有的定时器和 `timer` 和截止时间 `deadline` 也实现了定时取消这一功能：

```go
type timerCtx struct {
    cancelCtx
    timer *time.Timer // Under cancelCtx.mu.
    deadline time.Time
}
func (c *timerCtx) Deadline() (deadline time.Time, ok bool) {
    return c.deadline, true
}
func (c *timerCtx) cancel(removeFromParent bool, err error) {
    c.cancelCtx.cancel(false, err)
    if removeFromParent {
        removeChild(c.cancelCtx.Context, c)
    }
    c.mu.Lock()
    if c.timer != nil {
        c.timer.Stop()
        c.timer = nil
    }
    c.mu.Unlock()
}
```

`cancel` 方法不仅调用了内部嵌入的 `cancelCtx.cancel`，还会停止持有的定时器减少不必要的资源浪费。



#### 传值方法 withValue

在最后我们需要了解一下如何使用上下文传值，`context` 包中的 `WithValue` 函数能从父上下文中创建一个子上下文，传值的子上下文使用私有结构体 `valueCtx` 类型：

```go
func WithValue(parent Context, key, val interface{}) Context {
    if key == nil {
        panic("nil key")
    }
    if !reflectlite.TypeOf(key).Comparable() {
        panic("key is not comparable")
    }
    return &valueCtx{parent, key, val}
}
```

`valueCtx` 函数会将除了 `Value` 之外的 `Err`、`Deadline` 等方法代理到父上下文中，只会处理 `Value` 方法的调用，然而每一个 `valueCtx` 内部也并没有存储一个键值对的哈希，而是只包含一个键值对：

```go
type valueCtx struct {
    Context
    key, val interface{}
}
func (c *valueCtx) Value(key interface{}) interface{} {
    if c.key == key {
        return c.val
    }
    return c.Context.Value(key)
}
```

如果当前 `valueCtx` 中存储的键与 `Value` 方法中传入的不匹配，就会从父上下文中查找该键对应的值直到在某个父上下文中返回 `nil` 或者查找到对应的值。

使用场景与例子：

假设你有一个 HTTP 服务器，每个请求都需要一个唯一的请求 ID。你可以使用 `context.WithValue` 将请求 ID 存储在上下文中，以便在整个请求处理过程中访问和使用它。

```go
type key int

const requestIDKey key = 0

func main() {
	http.HandleFunc("/hello", helloHandler)
	http.ListenAndServe(":18080", nil)
}

func helloHandler(w http.ResponseWriter, r *http.Request) {
	// 生成一个唯一的请求 ID
	requestID := generateRequestID()

	// 将请求 ID 存储在上下文中
	ctx := context.WithValue(r.Context(), requestIDKey, requestID)

	// 在处理函数中可以通过上下文获取请求 ID
	greetUser(ctx, w)
}

func greetUser(ctx context.Context, w http.ResponseWriter) {
	// 从上下文中获取请求 ID
	requestID := ctx.Value(requestIDKey).(string)

	// 使用请求 ID 打印欢迎消息
	fmt.Fprintf(w, "Hello! Your request ID is: %s\n", requestID)
}

func generateRequestID() string {
	// 这里可以实现生成唯一请求 ID 的逻辑
	return "ABC123"
}

```



### 总结

Go 语言中的 `Context` 的**主要作用还是在多个 Goroutine 或者模块之间同步取消信号或者截止日期，用于减少对资源的消耗和长时间占用，避免资源浪费**，虽然**传值也是它的功能之一，但是这个功能我们还是很少用到**。

在真正使用传值的功能时我们也应该非常谨慎，**不能将请求的所有参数都使用 `Context` 进行传递，这是一种非常差的设计，比较常见的使用场景是传递请求对应用户的认证令牌(token)以及用于进行分布式追踪的请求 ID(request-id)**。

### Reference

- [Package context · Golang](https://golang.org/pkg/context/)
- [Go Concurrency Patterns: Context](https://blog.golang.org/context)
- [Using context cancellation in Go](https://www.sohamkamani.com/blog/golang/2018-06-17-golang-using-context-cancellation/)



## 6.2 同步原语与锁

> [6.2 同步原语与锁](https://www.bookstack.cn/read/draveness-golang/b103f6e877e4c9bd.md)

当提到并发编程、多线程编程时，我们往往都离不开『锁』这一概念，Go 语言作为一个原生支持用户态进程 Goroutine 的语言，也一定会为开发者提供这一功能，锁的主要作用就是保证多个线程或者 Goroutine 在访问同一片内存时不会出现混乱的问题，锁其实是一种并发编程中的同步原语（**Synchronization Primitives**）。

在这一节中我们就会介绍 Go 语言中常见的同步原语 `Mutex`、`RWMutex`、`WaitGroup`、`Once` 和 `Cond` 以及扩展原语 `ErrGroup`、`Semaphore`和 `SingleFlight` 的实现原理，同时也会涉及互斥锁、信号量等并发编程中的常见概念。

### 基本原语

Go 语言在 [sync](https://golang.org/pkg/sync/) 包中提供了用于同步的一些基本原语，包括常见的互斥锁 `Mutex` 与读写互斥锁 `RWMutex` 以及 `Once`、`WaitGroup`。

![golang-basic-sync-primitives](go语言设计与实现.assets/b468a5c547306515f9eaf4d27a18e962.png)

这些基本原语的主要作用是提供较为基础的同步功能，我们应该使用 Channel 和通信来实现更加高级的同步机制，我们在这一节中并不会介绍标准库中全部的原语，而是会介绍其中比较常见的 `Mutex`、`RWMutex`、`Once`、`WaitGroup` 和 `Cond`，我们并不会涉及剩下两个用于存取数据的结构体 `Map` 和 `Pool`。

#### Mutex

Go 语言中的互斥锁在 `sync` 包中，它由两个字段 `state` 和 `sema` 组成，`state` 表示当前互斥锁的状态，而 `sema` 真正用于控制锁状态的信号量，这两个加起来只占 8 个字节空间的结构体就表示了 Go 语言中的互斥锁。

```go
type Mutex struct {
    state int32
    sema  uint32
}
```



##### 状态

互斥锁的状态是用 `int32` 来表示的，但是锁的状态并不是互斥的，它的最低三位分别表示 `mutexLocked`、`mutexWoken` 和 `mutexStarving`，剩下的位置都用来表示当前有多少个 Goroutine 等待互斥锁被释放：

![golang-mutex-state](go语言设计与实现.assets/fc6f0b153ab9415ccaa70172f38acb57.png)

互斥锁在被创建出来时，所有的状态位的默认值都是 `0`，当互斥锁被锁定时 `mutexLocked` 就会被置成 `1`、当互斥锁被在正常模式下被唤醒时 `mutexWoken` 就会被被置成 `1`、`mutexStarving` 用于表示当前的互斥

锁进入了状态，最后的几位是在当前互斥锁上等待的 Goroutine 个数。



##### 饥饿模式

在了解具体的加锁和解锁过程之前，我们需要先简单了解一下 `Mutex` 在使用过程中可能会进入的饥饿模式，**饥饿模式是在 Go 语言 [1.9](https://go-review.googlesource.com/c/go/+/34310/8/src/sync/mutex.go#65) 版本引入的特性**，它的主要功能就是保证互斥锁的获取的『公平性』（Fairness）。

互斥锁可以同时处于两种不同的模式，也就是正常模式和饥饿模式，在正常模式下，所有锁的等待者都会按照先进先出的顺序获取锁，但是如果一个刚刚被唤起的 Goroutine 遇到了新的 Goroutine 进程也调用了 `Lock` 方法时，大概率会获取不到锁，为了减少这种情况的出现，防止 Goroutine 被『饿死』，一旦 Goroutine 超过 1ms 没有获取到锁，它就会将当前互斥锁切换饥饿模式。

![golang-mutex-mode](go语言设计与实现.assets/7b8ea4a921b415c0a842628e4ea67481.png)

在饥饿模式中，互斥锁会被直接交给等待队列最前面的 Goroutine，新的 Goroutine 在这时不能获取锁、也不会进入自旋的状态，它们只会在队列的末尾等待，如果一个 Goroutine 获得了互斥锁并且它是队列中最末尾

的协程或者它等待的时间少于 1ms，那么当前的互斥锁就会被切换回正常模式。

相比于饥饿模式，正常模式下的互斥锁能够提供更好地性能，饥饿模式的主要作用就是避免一些 Goroutine 由于陷入等待无法获取锁而造成较高的尾延时，这也是对 `Mutex` 的一个优化。



##### 加锁

互斥锁 `Mutex` 的加锁是靠 `Lock` 方法完成的，最新的 Go 语言源代码中已经将 `Lock` 方法进行了简化，方法的主干只保留了最常见、简单并且快速的情况；当锁的状态是 `0` 时直接将 `mutexLocked` 位置成 `1`：

```
func (m *Mutex) Lock() {
    if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
        return
    }
    m.lockSlow()
}
```

但是当 `Lock` 方法被调用时 `Mutex` 的状态不是 `0` 时就会进入 `lockSlow` 方法尝试通过自旋或者其他的方法等待锁的释放并获取互斥锁，该方法的主体是一个非常大 `for` 循环，我们会将该方法分成几个部分介绍获取锁的过程：

```
func (m *Mutex) lockSlow() {
    var waitStartTime int64
    starving := false
    awoke := false
    iter := 0
    old := m.state
    for {
        if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {
            if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
                atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
                awoke = true
            }
            runtime_doSpin()
            iter++
            old = m.state
            continue
        }    
```

在这段方法的第一部分会判断当前方法能否进入自旋来等待锁的释放，自旋（Spinnig）其实是在多线程同步的过程中使用的一种机制，当前的进程在进入自旋的过程中会一直保持 CPU 的占用，持续检查某个条件是否为真，**在多核的 CPU 上，自旋的优点是避免了 Goroutine 的切换**，所以如果使用恰当会对性能带来非常大的增益。

在 Go 语言的 `Mutex` 互斥锁中，只有在普通模式下才可能进入自旋，除了模式的限制之外，`runtime_canSpin` 方法中会判断当前方法是否可以进入自旋，进入自旋的条件非常苛刻：

- 运行在多 CPU 的机器上；
- 当前 Goroutine 为了获取该锁进入自旋的次数小于四次；
- 当前机器上至少存在一个正在运行的处理器 `P` 并且处理的运行队列是空的；一旦当前 Goroutine 能够进入自旋就会调用 `runtime_doSpin`，它最终调用汇编语言编写的方法 `procyield` 并执行指定次数的 `PAUSE` 指令，`PAUSE` 指令什么都不会做，但是会消耗 CPU 时间，每次自旋都会调用 `30` 次 `PAUSE`，下面是该方法在 386 架构的机器上的实现：

```
TEXT runtime·procyield(SB),NOSPLIT,$0-0
    MOVL    cycles+0(FP), AX
again:
    PAUSE
    SUBL    $1, AX
    JNZ    again
    RET
```

处理了自旋相关的特殊逻辑之后，互斥锁接下来就根据上下文计算当前互斥锁最新的状态了，几个不同的条件分别会更新 `state` 中存储的不同信息 `mutexLocked`、`mutexStarving`、`mutexWoken` 和 `mutexWaiterShift`：

```go
        new := old
        if old&mutexStarving == 0 {
            new |= mutexLocked
        }
        if old&(mutexLocked|mutexStarving) != 0 {
            new += 1 << mutexWaiterShift
        }
        if starving && old&mutexLocked != 0 {
            new |= mutexStarving
        }
        if awoke {
            new &^= mutexWoken
        }
```

计算了新的互斥锁状态之后，我们就会使用 `atomic` 包提供的 CAS 函数修改互斥锁的状态，如果当前的互斥锁已经处于饥饿和锁定的状态，就会跳过当前步骤，调用 `runtime_SemacquireMutex` 方法：

```
        if atomic.CompareAndSwapInt32(&m.state, old, new) {
            if old&(mutexLocked|mutexStarving) == 0 {
                break // locked the mutex with CAS
            }
            queueLifo := waitStartTime != 0
            if waitStartTime == 0 {
                waitStartTime = runtime_nanotime()
            }
            runtime_SemacquireMutex(&m.sema, queueLifo, 1)
            starving = starving || runtime_nanotime()-waitStartTime > starvationThresholdNs
            old = m.state
            if old&mutexStarving != 0 {
                delta := int32(mutexLocked - 1<<mutexWaiterShift)
                if !starving || old>>mutexWaiterShift == 1 {
                    delta -= mutexStarving
                }
                atomic.AddInt32(&m.state, delta)
                break
            }
            awoke = true
            iter = 0
        } else {
            old = m.state
        }
    }
}
```

`runtime_SemacquireMutex` 方法的主要作用就是通过 `Mutex` 的使用互斥锁中的信号量保证资源不会被两个 Goroutine 获取，从这里我们就能看出 `Mutex` 其实就是对更底层的信号量进行封装，对外提供更加易用的 API，`runtime_SemacquireMutex` 会在方法中不断调用 `goparkunlock` 将当前 Goroutine 陷入休眠等待信号量可以被获取。

一旦当前 Goroutine 可以获取信号量，就证明互斥锁已经被解锁，该方法就会立刻返回，`Lock` 方法的剩余代码也会继续执行下去了，当前互斥锁处于饥饿模式时，如果该 Goroutine 是队列中最后的一个 Goroutine 或者等待锁的时间小于 `starvationThresholdNs(1ms)`，当前 Goroutine 就会直接获得互斥锁并且从饥饿模式中退出并获得锁。



##### 解锁

互斥锁的解锁过程相比之下就非常简单，`Unlock` 方法会直接使用 `atomic` 包提供的 `AddInt32`，如果返回的新状态不等于 `0` 就会进入 `unlockSlow` 方法：

```
func (m *Mutex) Unlock() {
    new := atomic.AddInt32(&m.state, -mutexLocked)
    if new != 0 {
        m.unlockSlow(new)
    }
}
```



`unlockSlow` 方法首先会对锁的状态进行校验，如果当前互斥锁已经被解锁过了就会直接抛出异常 `sync: unlock of unlocked mutex` 中止当前程序，在正常情况下会根据当前互斥锁的状态是正常模式还是饥饿模式进入不同的分支：

```go
func (m *Mutex) unlockSlow(new int32) {
    if (new+mutexLocked)&mutexLocked == 0 {
        throw("sync: unlock of unlocked mutex")
    }
    if new&mutexStarving == 0 {
        old := new
        for {
            if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken|mutexStarving) != 0 {
                return
            }
            new = (old - 1<<mutexWaiterShift) | mutexWoken
            if atomic.CompareAndSwapInt32(&m.state, old, new) {
                runtime_Semrelease(&m.sema, false, 1)
                return
            }
            old = m.state
        }
    } else {
        runtime_Semrelease(&m.sema, true, 1)
    }
}
```

如果当前互斥锁的状态是饥饿模式就会直接调用 `runtime_Semrelease` 方法直接将当前锁交给下一个正在尝试获取锁的等待者，等待者会在被唤醒之后设置 `mutexLocked` 状态，由于此时仍然处于 `mutexStarving`，所以新的 Goroutine 也无法获得锁。

在正常模式下，如果当前互斥锁不存在等待者或者最低三位表示的状态都为 `0`，那么当前方法就不需要唤醒其他 Goroutine 可以直接返回，当有 Goroutine 正在处于等待状态时，还是会通过 `runtime_Semrelease` 唤醒对应的 Goroutine 并移交锁的所有权。



##### 小结

通过对互斥锁 `Mutex` 加锁和解锁过程的分析，我们能够得出以下的一些结论，它们能够帮助我们更好地理解互斥锁的工作原理，互斥锁的加锁的过程比较复杂，涉及自旋、信号量以及 Goroutine 调度等概念：

- 如果互斥锁处于初始化状态，就会直接通过置位 `mutexLocked` 加锁；
- 如果互斥锁处于 `mutexLocked` 并且在普通模式下工作，就会进入自旋，执行 30 次 `PAUSE` 指令消耗 CPU 时间等待锁的释放；
- 如果当前 Goroutine 等待锁的时间超过了 `1ms`，互斥锁就会被切换到饥饿模式；
- 互斥锁在正常情况下会通过 `runtime_SemacquireMutex` 方法将调用 `Lock` 的 Goroutine 切换至休眠状态，等待持有信号量的 Goroutine 唤醒当前协程；
- 如果当前 Goroutine 是互斥锁上的最后一个等待的协程或者等待的时间小于 `1ms`，当前 Goroutine 会将互斥锁切换回正常模式；

互斥锁的解锁过程相对来说就比较简单，虽然对于普通模式和饥饿模式的处理有一些不同，但是由于代码行数不多，所以逻辑清晰，也非常容易理解：

- 如果互斥锁已经被解锁，那么调用 `Unlock` 会直接抛出异常；
- 如果互斥锁处于饥饿模式，会直接将锁的所有权交给队列中的下一个等待者，等待者会负责设置 `mutexLocked` 标志位；
- 如果互斥锁处于普通模式，并且没有 Goroutine 等待锁的释放或者已经有被唤醒的 Goroutine 获得了锁就会直接返回，在其他情况下回通过 `runtime_Semrelease` 唤醒对应的 Goroutine；



#### RWMutex

读写互斥锁也是 Go 语言 `sync` 包为我们提供的接口之一，一个常见的服务对资源的读写比例会非常高，如果大多数的请求都是读请求，它们之间不会相互影响，那么我们为什么不能将对资源读和写操作分离呢？这也就是 `RWMutex` 读写互斥锁解决的问题，不限制对资源的并发读，但是读写、写写操作无法并行执行。

|      | 读   | 写   |
| :--- | :--- | :--- |
| 读   | Y    | N    |
| 写   | N    | N    |

读写互斥锁在 Go 语言中的实现是 `RWMutex`，**其中不仅包含一个互斥锁，还持有两个信号量，分别用于写等待读和读等待写**：

```
type RWMutex struct {
    w           Mutex
    writerSem   uint32
    readerSem   uint32
    readerCount int32
    readerWait  int32
}
```

`readerCount` 存储了当前正在执行的读操作的数量，最后的 `readerWait` 表示当写操作被阻塞时等待的读操作个数。



##### 读锁

读锁的加锁非常简单，我们通过 `atomic.AddInt32` 方法为 `readerCount` 加一，如果该方法返回了负数说明当前有 Goroutine 获得了写锁，当前 Goroutine 就会调用 `runtime_SemacquireMutex` 陷入休眠等待唤醒：

```
func (rw *RWMutex) RLock() {
    if atomic.AddInt32(&rw.readerCount, 1) < 0 {
        runtime_SemacquireMutex(&rw.readerSem, false, 0)
    }
}
```



如果没有写操作获取当前互斥锁，当前方法就会在 `readerCount` 加一后返回；当 Goroutine 想要释放读锁时会调用 `RUnlock` 方法：

```
func (rw *RWMutex) RUnlock() {
    if r := atomic.AddInt32(&rw.readerCount, -1); r < 0 {
        rw.rUnlockSlow(r)
    }
}
```

该方法会在减少正在读资源的 `readerCount`，当前方法如果遇到了返回值小于零的情况，说明有一个正在进行的写操作，在这时就应该通过 `rUnlockSlow` 方法减少当前写操作等待的读操作数 `readerWait` 并在所有读操作都被释放之后触发写操作的信号量 `writerSem`：

```
func (rw *RWMutex) rUnlockSlow(r int32) {
    if r+1 == 0 || r+1 == -rwmutexMaxReaders {
        throw("sync: RUnlock of unlocked RWMutex")
    }
    if atomic.AddInt32(&rw.readerWait, -1) == 0 {
        runtime_Semrelease(&rw.writerSem, false, 1)
    }
}
```

`writerSem` 在被触发之后，尝试获取读写锁的进程就会被唤醒并获得锁。



##### 读写锁

当资源的使用者想要获取读写锁时，就需要通过 `Lock` 方法了，在 `Lock` 方法中首先调用了读写互斥锁持有的 `Mutex` 的 `Lock` 方法保证其他获取读写锁的 Goroutine 进入等待状态，随后的 `atomic.AddInt32(&rw.readerCount, -rwmutexMaxReaders)` 其实是为了阻塞后续的读操作：

```
func (rw *RWMutex) Lock() {
    rw.w.Lock()
    r := atomic.AddInt32(&rw.readerCount, -rwmutexMaxReaders) + rwmutexMaxReaders
    if r != 0 && atomic.AddInt32(&rw.readerWait, r) != 0 {
        runtime_SemacquireMutex(&rw.writerSem, false, 0)
    }
}
```

如果当前仍然有其他 Goroutine 持有互斥锁的读锁，该 Goroutine 就会调用 `runtime_SemacquireMutex` 进入休眠状态，等待读锁释放时触发 `writerSem` 信号量将当前协程唤醒。

对资源的读写操作完成之后就会将通过 `atomic.AddInt32(&rw.readerCount, rwmutexMaxReaders)` 变回正数并通过 for 循环触发所有由于获取读锁而陷入等待的 Goroutine：

```
func (rw *RWMutex) Unlock() {
    r := atomic.AddInt32(&rw.readerCount, rwmutexMaxReaders)
    if r >= rwmutexMaxReaders {
        throw("sync: Unlock of unlocked RWMutex")
    }
    for i := 0; i < int(r); i++ {
        runtime_Semrelease(&rw.readerSem, false, 0)
    }
    rw.w.Unlock()
}
```

在方法的最后，`RWMutex` 会释放持有的互斥锁让其他的协程能够重新获取读写锁。

##### 小结

相比状态复杂的互斥锁 `Mutex` 来说，读写互斥锁 `RWMutex` 虽然提供的功能非常复杂，但是由于站在了 `Mutex` 的『肩膀』上，所以整体的实现上会简单很多。

- `readerSem` — 读写锁释放时通知由于获取读锁等待的 Goroutine；
- `writerSem` — 读锁释放时通知由于获取读写锁等待的 Goroutine；
- `w` 互斥锁 — 保证写操作之间的互斥；
- `readerCount` — 统计当前进行读操作的协程数，触发写锁时会将其减少 `rwmutexMaxReaders` 阻塞后续的读操作；
- `readerWait` — 当前读写锁等待的进行读操作的协程数，在触发 `Lock` 之后的每次 `RUnlock` 都会将其减一，当它归零时该 Goroutine 就会获得读写锁；
- 当读写锁被释放 `Unlock` 时首先会通知所有的读操作，然后才会释放持有的互斥锁，这样能够保证读操作不会被连续的写操作『饿死』；`RWMutex` 在 `Mutex` 之上提供了额外的读写分离功能，能够在读请求远远多于写请求时提供性能上的提升，我们也可以在场景合适时选择读写互斥锁。



#### WaitGroup

`WaitGroup` 是 Go 语言 `sync` 包中比较常见的同步机制，它可以**用于等待一系列的 Goroutine 的返回**，一个比较常见的使用场景是批量执行 RPC 或者调用外部服务：

```go
requests := []*Request{...}
wg := &sync.WaitGroup{}
wg.Add(len(requests))
for _, request := range requests {
    go func(r *Request) {
        defer wg.Done()
        // res, err := service.call(r)
    }(request)
}
wg.Wait()
```

通过 `WaitGroup` 我们可以在多个 Goroutine 之间非常轻松地同步信息，原本顺序执行的代码也可以在多个 Goroutine 中并发执行，加快了程序处理的速度，在上述代码中只有在所有的 Goroutine 都执行完毕之后 `Wait` 方法才会返回，程序可以继续执行其他的逻辑。

![golang-syncgroup](go语言设计与实现.assets/af2190d45653d6fd8c983873df41a53b.png)

总而言之，它的作用就像它的名字一样，，通过 `Done` 来传递任务完成的信号，比较常用于等待一组 Goroutine 中并发执行的任务全部结束。



##### 结构体

`WaitGroup` 结构体中的成员变量非常简单，其中的 `noCopy` 的主要作用就是保证 `WaitGroup` 不会被开发者通过再赋值的方式进行拷贝，进而导致一些诡异的行为：

```
type WaitGroup struct {
    noCopy noCopy
    state1 [3]uint32
}
```

[copylock](http://golang.so/pkg/cmd/vendor/golang.org/x/tools/go/analysis/passes/copylock/) 包就是一个用于检查类似错误的分析器，它的原理就是在 [编译期间](https://www.bookstack.cn/read/draveness-golang/9e405d643b49d00e.md) 检查被拷贝的变量中是否包含 `noCopy` 或者 `sync` 关键字，如果包含当前关键字就会报出以下的错误：

```go
package main
import (
    "fmt"
    "sync"
)
func main() {
    wg := sync.Mutex{}
    yawg := wg
    fmt.Println(wg, yawg)
}
$ go run proc.go
./prog.go:10:10: assignment copies lock value to yawg: sync.Mutex
./prog.go:11:14: call of fmt.Println copies lock value: sync.Mutex
./prog.go:11:18: call of fmt.Println copies lock value: sync.Mutex
```

这段代码会在赋值和调用 `fmt.Println` 时发生值拷贝导致分析器报错，你可以通过访问 [链接](https://play.golang.org/p/l12ZpEeA9yE) 尝试运行这段代码。

除了 `noCopy` 之外，`WaitGroup` 结构体中还包含一个总共占用 12 字节大小的数组，这个数组中会存储当前结构体持有的状态和信号量，在 64 位与 32 位的机器上表现也非常不同。

![golang-waitgroup-state](go语言设计与实现.assets/b02558f1bdf14f50395b75ba0aed09c7.png)

`WaitGroup` 提供了私有方法 `state` 能够帮助我们从 `state1` 字段中取出它的状态和信号量。

##### 操作

`WaitGroup` 对外暴露的接口只有三个 `Add`、`Wait` 和 `Done`，其中 `Done` 方法只是调用了 `wg.Add(-1)` 本身并没有什么特殊的逻辑，我们来了解一下剩余的两个方法：

```
func (wg *WaitGroup) Add(delta int) {
    statep, semap := wg.state()
    state := atomic.AddUint64(statep, uint64(delta)<<32)
    v := int32(state >> 32)
    w := uint32(state)
    if v < 0 {
        panic("sync: negative WaitGroup counter")
    }
    if v > 0 || w == 0 {
        return
    }
    *statep = 0
    for ; w != 0; w-- {
        runtime_Semrelease(semap, false, 0)
    }
}
```

`Add` 方法的主要作用就是更新 `WaitGroup` 中持有的计数器 `counter`，64 位状态的高 32 位，虽然 `Add` 方法传入的参数可以为负数，但是一个 `WaitGroup` 的计数器只能是非负数，当调用 `Add` 方法导致计数器归零并且还有等待的 Goroutine 时，就会通过 `runtime_Semrelease` 唤醒处于等待状态的所有 Goroutine。

另一个 `WaitGroup` 的方法 `Wait` 就会在当前计数器中保存的数据大于 `0` 时修改等待 Goroutine 的个数 `waiter` 并调用 `runtime_Semacquire` 陷入睡眠状态。

```
func (wg *WaitGroup) Wait() {
    statep, semap := wg.state()
    for {
        state := atomic.LoadUint64(statep)
        v := int32(state >> 32)
        if v == 0 {
            return
        }
        if atomic.CompareAndSwapUint64(statep, state, state+1) {
            runtime_Semacquire(semap)
            if +statep != 0 {
                panic("sync: WaitGroup is reused before previous Wait has returned")
            }
            return
        }
    }
}
```

陷入睡眠的 Goroutine 就会等待 `Add` 方法在计数器为 `0` 时唤醒。

##### 小结

通过对 `WaitGroup` 的分析和研究，我们能够得出以下的一些结论：

- `Add` 不能在和 `Wait` 方法在 Goroutine 中并发调用，一旦出现就会造成程序崩溃；
- `WaitGroup` 必须在 `Wait` 方法返回之后才能被重新使用；
- `Done` 只是对 `Add` 方法的简单封装，我们可以向 `Add` 方法传入任意负数（需要保证计数器非负）快速将计数器归零以唤醒其他等待的 Goroutine；
- 可以同时有多个 Goroutine 等待当前 `WaitGroup` 计数器的归零，这些 Goroutine 也会被『同时』唤醒；





#### Once

Go 语言在标准库的 `sync` 同步包中还提供了 `Once` 语义，它的主要功能其实也很好理解，**保证在 Go 程序运行期间 `Once` 对应的某段代码只会执行一次**。

在如下所示的代码中，`Do` 方法中传入的函数只会被执行一次，也就是我们在运行如下所示的代码时只会看见一次 `only once` 的输出结果：

```go
func main() {
    o := &sync.Once{}
    for i := 0; i < 10; i++ {
        o.Do(func() {
            fmt.Println("only once")
        })
    }
}
$ go run main.go
only once
```

作为 `sync` 包中的结构体，`Once` 有着非常简单的数据结构，每一个 `Once` 结构体中都只包含一个用于标识代码块是否被执行过的 `done` 以及一个互斥锁 `Mutex`：

```
type Once struct {
    done uint32
    m    Mutex
}
```

`Once` 结构体对外唯一暴露的方法就是 `Do`，该方法会接受一个入参为空的函数，如果使用 `atomic.LoadUint32` 检查到已经执行过函数了，就会直接返回，否则就会进入 `doSlow` 运行传入的函数：

```
func (o *Once) Do(f func()) {
    if atomic.LoadUint32(&o.done) == 0 {
        o.doSlow(f)
    }
}
func (o *Once) doSlow(f func()) {
    o.m.Lock()
    defer o.m.Unlock()
    if o.done == 0 {
        defer atomic.StoreUint32(&o.done, 1)
        f()
    }
}
```

`doSlow` 的实现也非常简单，我们先为当前的 Goroutine 获取互斥锁，然后通过 `defer` 关键字将 `done` 成员变量设置成 `1` 并运行传入的函数，无论当前函数是正常运行还是抛出 `panic`，当前方法都会将 `done` 设置成 `1` 保证函数不会执行第二次。



##### 小结

作为用于保证函数执行次数的 `Once` 结构体，它使用互斥锁和 `atomic` 提供的方法实现了某个函数在程序运行期间只能执行一次的语义，在使用的过程中我们也需要注意以下的内容：

- `Do` 方法中传入的函数只会被执行一次，哪怕函数中发生了 `panic`；
- 两次调用 `Do` 方法传入不同的函数时只会执行第一次调用的函数；



#### Cond

Go 语言在标准库中提供的 `Cond` 其实是一个条件变量，通过 `Cond` 我们**可以让一系列的 Goroutine 都在触发某个事件或者条件时才被唤醒**，每一个 `Cond` 结构体都包含一个互斥锁 `L`，我们先来看一下 `Cond` 是如何使用的：

```go
func main() {
    c := sync.NewCond(&sync.Mutex{})
    for i := 0; i < 10; i++ {
        go listen(c)
    }
    time.Sleep(1*time.Second)
    go broadcast(c)
    ch := make(chan os.Signal, 1)
    signal.Notify(ch, os.Interrupt)
    <-ch
}
func broadcast(c *sync.Cond) {
    c.L.Lock()
    c.Broadcast()
    c.L.Unlock()
}
func listen(c *sync.Cond) {
    c.L.Lock()
    c.Wait()
    fmt.Println("listen")
    c.L.Unlock()
}
$ go run main.go
listen
listen
...
listen
```

在上述代码中我们同时运行了 11 个 Goroutine，其中的 10 个 Goroutine 会通过 `Wait` 等待期望的信号或者事件，而剩下的一个 Goroutine 会调用 `Broadcast` 方法通知所有陷入等待的 Goroutine，当调用 `Boardcast` 方法之后，就会打印出 10 次 `"listen"` 并结束调用。



![golang-cond-broadcast](go语言设计与实现.assets/8332488ca73a16a1941471895d2c5025.png)

##### 结构体

`Cond` 的结构体中包含 `noCopy` 和 `copyChecker` 两个字段，前者用于保证 `Cond` 不会再编译期间拷贝，后者保证在运行期间发生拷贝会直接 `panic`，持有的另一个锁 `L` 其实是一个接口 `Locker`，任意实现 `Lock` 和 `Unlock` 方法的结构体都可以作为 `NewCond` 方法的参数：

```go
type Cond struct {
    noCopy noCopy
    L Locker
    notify  notifyList
    checker copyChecker
}
```

结构体中最后的变量 `notifyList` 其实也就是为了实现 `Cond` 同步机制，该结构体其实就是一个 `Goroutine` 的链表：

```
type notifyList struct {
    wait uint32
    notify uint32
    lock mutex
    head *sudog
    tail *sudog
}
```

在这个结构体中，`head` 和 `tail` 分别指向的就是整个链表的头和尾，而 `wait` 和 `notify` 分别表示当前正在等待的 Goroutine 和已经通知到的 Goroutine，我们通过这两个变量就能确认当前待通知和已通知的 Goroutine。



##### 操作

`Cond` 对外暴露的 `Wait` 方法会将当前 Goroutine 陷入休眠状态，它会先调用 `runtime_notifyListAdd` 将等待计数器 `+1`，然后解锁并调用 `runtime_notifyListWait` 等待其他 Goroutine 的唤醒：

```
func (c *Cond) Wait() {
    c.checker.check()
    t := runtime_notifyListAdd(&c.notify)
    c.L.Unlock()
    runtime_notifyListWait(&c.notify, t)
    c.L.Lock()
}
func notifyListAdd(l *notifyList) uint32 {
    return atomic.Xadd(&l.wait, 1) - 1
}
```

`notifyListWait` 方法的主要作用就是获取当前的 Goroutine 并将它追加到 `notifyList` 链表的最末端：

```

func notifyListWait(l *notifyList, t uint32) {
    lock(&l.lock)
    if less(t, l.notify) {
        unlock(&l.lock)
        return
    }
    s := acquireSudog()
    s.g = getg()
    s.ticket = t
    if l.tail == nil {
        l.head = s
    } else {
        l.tail.next = s
    }
    l.tail = s
    goparkunlock(&l.lock, waitReasonSyncCondWait, traceEvGoBlockCond, 3)
    releaseSudog(s)
}
```

除了将当前 Goroutine 追加到链表的末端之外，我们还会调用 `goparkunlock` 陷入休眠状态，该函数也是在 Go 语言切换 Goroutine 时经常会使用的方法，它会直接让出当前处理器的使用权并等待调度器的唤醒。

![golang-cond-notifylist](go语言设计与实现.assets/57d27d84e3f03689b5b3ca181a697d76.png)

`Cond` 对外提供的 `Signal` 和 `Broadcast` 方法就是用来唤醒调用 `Wait` 陷入休眠的 Goroutine，从两个方法的名字来看，**前者会唤醒队列最前面的 Goroutine**，后者会唤醒队列中全部的 Goroutine：

```
func (c *Cond) Signal() {
    c.checker.check()
    runtime_notifyListNotifyOne(&c.notify)
}
func (c *Cond) Broadcast() {
    c.checker.check()
    runtime_notifyListNotifyAll(&c.notify)
}
```

`notifyListNotifyAll` 方法会从链表中取出全部的 Goroutine 并为它们依次调用 `readyWithTime`，该方法会通过 `goready` 将目标的 Goroutine 唤醒：

```
func notifyListNotifyAll(l *notifyList) {
    s := l.head
    l.head = nil
    l.tail = nil
    atomic.Store(&l.notify, atomic.Load(&l.wait))
    for s != nil {
        next := s.next
        s.next = nil
        readyWithTime(s, 4)
        s = next
    }
}
```

虽然它会依次唤醒全部的 Goroutine，但是这里唤醒的顺序其实也是按照加入队列的先后顺序，先加入的会先被 `goready` 唤醒，后加入的 Goroutine 可能就需要等待调度器的调度。

而 `notifyListNotifyOne` 函数就只会从 `sudog` 构成的链表中满足 `sudog.ticket == l.notify` 的 Goroutine 并通过 `readyWithTime` 唤醒：

```
func notifyListNotifyOne(l *notifyList) {
    t := l.notify
    atomic.Store(&l.notify, t+1)
    for p, s := (*sudog)(nil), l.head; s != nil; p, s = s, s.next {
        if s.ticket == t {
            n := s.next
            if p != nil {
                p.next = n
            } else {
                l.head = n
            }
            if n == nil {
                l.tail = p
            }
            s.next = nil
            readyWithTime(s, 4)
            return
        }
    }
}
```

在一般情况下我们都会选择在不满足特定条件时调用 `Wait` 陷入休眠，当某些 Goroutine 检测到当前满足了唤醒的条件，就可以选择使用 `Signal` 通知一个或者 `Broadcast` 通知全部的 Goroutine 当前条件已经满足，可以继续完成工作了。

##### 小结

与 `Mutex` 相比，`Cond` 还是一个不被所有人都清楚和理解的同步机制，它提供了类似队列的 FIFO 的等待机制，同时也提供了 `Signal` 和 `Broadcast` 两种不同的唤醒方法，相比于使用 `for {}` 忙碌等待，使用 `Cond` 能够在遇到长时间条件无法满足时将当前处理器让出的功能，如果我们合理使用还是能够在一些情况下提升性能，在使用的过程中我们需要注意：

- `Wait` 方法在调用之前一定要使用 `L.Lock` 持有该资源，否则会发生 `panic` 导致程序崩溃；
- `Signal` 方法唤醒的 Goroutine 都是队列最前面、等待最久的 Goroutine；
- `Broadcast` 虽然是广播通知全部等待的 Goroutine，但是真正被唤醒时也是按照一定顺序的；

### 扩展原语

除了这些标准库中提供的同步原语之外，Go 语言还在子仓库 `x/sync` 中提供了额外的四种同步原语，`ErrGroup`、`Semaphore`、`SingleFlight` 和 `SyncMap`，其中的 `SyncMap` 其实就是 `sync` 包中的 `sync.Map`，它在 1.9 版本的 Go 语言中被引入了 `x/sync` 包，随着 API 的成熟和稳定最后被移到了标准库 `sync` 包中。

![golang-extension-sync-primitives](go语言设计与实现.assets/41e4c0da7fbd58caba17f9984c85fd98.png)

我们在这一节中就会介绍 Go 语言目前在扩展包中提供的三种原语，也就是 `ErrGroup`、`Semaphore` 和 `SingleFlight`。



#### ErrGroup

子仓库 `x/sync` 中的包 [errgroup](https://godoc.org/golang.org/x/sync/errgroup) 其实就为我们在一组 Goroutine 中提供了同步、错误传播以及上下文取消的功能，我们可以使用如下所示的方式并行获取网页的数据：

```
var g errgroup.Group
var urls = []string{
    "http://www.golang.org/",
    "http://www.google.com/",
    "http://www.somestupidname.com/",
}
for i := range urls {
    url := urls[i]
    g.Go(func() error {
        resp, err := http.Get(url)
        if err == nil {
            resp.Body.Close()
        }
        return err
    })
}
if err := g.Wait(); err == nil {
    fmt.Println("Successfully fetched all URLs.")
}
```

`Go` 方法能够创建一个 Goroutine 并在其中执行传入的函数，而 `Wait` 方法会等待 `Go` 方法创建的 Goroutine 全部返回后返回第一个非空的错误，如果所有的 Goroutine 都没有返回错误，该函数就会返回 `nil`。



##### 结构体

`errgroup` 包中的 `Group` 结构体同时由三个比较重要的部分组成：

- 创建 `Context` 时返回的 `cancel` 函数，主要用于通知使用 `context` 的 Goroutine 由于某些子任务出错，可以停止工作让出资源了；
- 用于等待一组 Goroutine 完成子任务的 `WaitGroup` 同步原语；
- 用于接受子任务返回错误的 `err` 和保证 `err` 只会被赋值一次的 `errOnce`；

这些字段共同组成了 `Group` 结构体并为我们提供同步、错误传播以及上下文取消等功能。



##### 操作

`errgroup` 对外唯一暴露的构造器就是 `WithContext` 方法，我们只能从一个 `Context` 中创建一个新的 `Group` 变量，`WithCancel` 返回的取消函数也仅会在 `Group` 结构体内部使用：

```
func WithContext(ctx context.Context) (*Group, context.Context) {
    ctx, cancel := context.WithCancel(ctx)
    return &Group{cancel: cancel}, ctx
}
```



创建新的并行子任务需要使用 `Go` 方法，这个方法内部会对 `WaitGroup` 加一并创建一个新的 Goroutine，在 Goroutine 内部运行子任务并在返回错误时及时调用 `cancel` 并对 `err` 赋值，只有最早返回的错误才会被上游感知到，后续的错误都会被舍弃：

```go
func (g *Group) Go(f func() error) {
    g.wg.Add(1)
    go func() {
        defer g.wg.Done()
        if err := f(); err != nil {
            g.errOnce.Do(func() {
                g.err = err
                if g.cancel != nil {
                    g.cancel()
                }
            })
        }
    }()
}
func (g *Group) Wait() error {
    g.wg.Wait()
    if g.cancel != nil {
        g.cancel()
    }
    return g.err
}
```

`Wait` 方法其实就只是调用了 `WaitGroup` 的同步方法，在子任务全部完成时取消 `Context` 并返回可能出现的错误。



##### 小结

`errgroup` 包中的 `Group` 同步原语的实现原理还是非常简单的，它没有涉及非常底层和运行时包中的 API，只是对基本同步语义进行了简单的封装提供了更加复杂的功能，在使用时我们也需要注意以下的几个问题：

- 出现错误或者等待结束后都会调用 `Context` 的 `cancel` 方法取消上下文；
- 只有第一个出现的错误才会被返回，剩余的错误都会被直接抛弃；



#### Semaphore

信号量是在并发编程中比较常见的一种同步机制，它会保证持有的计数器在 `0` 到初始化的权重之间，每次获取资源时都会将信号量中的计数器减去对应的数值，在释放时重新加回来，当遇到计数器大于信号量大小时就会进入休眠等待其他进程释放信号，我们常常会在控制访问资源的进程数量时用到。

Golang 的扩展包中就提供了带权重的信号量，我们可以按照不同的权重对资源的访问进行管理，这个包对外也只提供了四个方法：

- `NewWeighted` 用于创建新的信号量；
- `Acquire` 获取了指定权重的资源，如果当前没有『空闲资源』，就会陷入休眠等待；
- `TryAcquire` 也用于获取指定权重的资源，但是如果当前没有『空闲资源』，就会直接返回 `false`；
- `Release` 用于释放指定权重的资源；



##### 结构体

`NewWeighted` 方法的主要作用创建一个新的权重信号量，传入信号量最大的权重就会返回一个新的 `Weighted` 结构体指针：

```
func NewWeighted(n int64) *Weighted {
    w := &Weighted{size: n}
    return w
}
type Weighted struct {
    size    int64
    cur     int64
    mu      sync.Mutex
    waiters list.List
}
```

`Weighted` 结构体中包含一个 `waiters` 列表其中存储着等待获取资源的『用户』，除此之外它还包含当前信号量的上限以及一个计数器 `cur`，这个计数器的范围就是 `[0, size]`：

![golang-semaphore](go语言设计与实现.assets/b0e874d9d0e29fb21400504877371cbe.png)

信号量中的计数器会随着用户对资源的访问和释放进行改变，引入的权重概念能够帮助我们更好地对资源的访问粒度进行控制，尽可能满足所有常见的用例。



##### 获取

在上面我们已经提到过 `Acquire` 方法就是用于获取指定权重资源的方法，这个方法总共由三个不同的情况组成：

- 当信号量中剩余的资源大于获取的资源并且没有等待的 Goroutine 时就会直接获取信号量；
- 当需要获取的信号量大于 `Weighted` 的大小时，由于不可能满足条件就会直接返回；
- 遇到其他情况时会将当前 Goroutine 加入到等待列表并通过 `select` 等待当前 Goroutine 被唤醒，被唤醒后就会获取信号量；

```
func (s *Weighted) Acquire(ctx context.Context, n int64) error {
    s.mu.Lock()
    if s.size-s.cur >= n && s.waiters.Len() == 0 {
        s.cur += n
        s.mu.Unlock()
        return nil
    }
    if n > s.size {
        s.mu.Unlock()
        <-ctx.Done()
        return ctx.Err()
    }
    ready := make(chan struct{})
    w := waiter{n: n, ready: ready}
    elem := s.waiters.PushBack(w)
    s.mu.Unlock()
    select {
    case <-ctx.Done():
        err := ctx.Err()
        s.mu.Lock()
        select {
        case <-ready:
            err = nil
        default:
            s.waiters.Remove(elem)
        }
        s.mu.Unlock()
        return err
    case <-ready:
        return nil
    }
}
```

另一个用于获取信号量的方法 `TryAcquire` 相比之下就非常简单，它只会判断当前信号量是否有充足的资源获取，如果有充足的资源就会直接立刻返回 `true` 否则就会返回 `false`：

```
func (s *Weighted) TryAcquire(n int64) bool {
    s.mu.Lock()
    success := s.size-s.cur >= n && s.waiters.Len() == 0
    if success {
        s.cur += n
    }
    s.mu.Unlock()
    return success
}
```

与 `Acquire` 相比，`TryAcquire` 由于不会等待资源的释放所以可能更适用于一些延时敏感、用户需要立刻感知结果的场景。



##### 释放

最后要介绍的 `Release` 方法其实也非常简单，当我们对信号量进行释放时，`Release` 方法会从头到尾遍历 `waiters` 列表中全部的等待者，如果释放资源后的信号量有充足的剩余资源就会通过 Channel 唤起指定的 Goroutine：

```
func (s *Weighted) Release(n int64) {
    s.mu.Lock()
    s.cur -= n
    for {
        next := s.waiters.Front()
        if next == nil {
            break
        }
        w := next.Value.(waiter)
        if s.size-s.cur < w.n {
            break
        }
        s.cur += w.n
        s.waiters.Remove(next)
        close(w.ready)
    }
    s.mu.Unlock()
}
```

当然也可能会出现剩余资源无法唤起 Goroutine 的情况，在这时当前方法就会释放锁后直接返回，通过对这段代码的分析我们也能发现，如果一个信号量需要的占用的资源非常多，他可能会长时间无法获取锁，这可能也是 `Acquire` 方法引入另一个参数 `Context` 的原因，为信号量的获取设置一个超时时间。

##### 小结

带权重的信号量确实有着更多的应用场景，这也是 Go 语言对外提供的唯一一种信号量实现，在使用的过程中我们需要注意以下的几个问题：

- `Acquire` 和 `TryAcquire` 方法都可以用于获取资源，前者用于同步获取会等待锁的释放，后者会在无法获取锁时直接返回；
- `Release` 方法会按照 FIFO 的顺序唤醒可以被唤醒的 Goroutine；
- 如果一个 Goroutine 获取了较多地资源，由于 `Release` 的释放策略可能会等待比较长的时间；

#### SingleFlight

[singleflight](https://godoc.org/golang.org/x/sync/singleflight) 是 Go 语言扩展包中提供了另一种同步原语，这其实也是作者最喜欢的一种同步扩展机制，它**能够在一个服务中抑制对下游的多次重复请求**，一个比较常见的使用场景是 — 我们在使用 Redis 对数据库中的一些热门数据进行了缓存并设置了超时时间，缓存超时的一瞬间可能有非常多的并行请求发现了 Redis 中已经不包含任何缓存所以大量的流量会打到数据库上影响服务的延时和质量。

![golang-query-without-single-flight](go语言设计与实现.assets/d32413402bbf0890edb987a0f2f32caa.png)

但是 `singleflight` 就能有效地解决这个问题，它的主要作用就是对于同一个 `Key` 最终只会进行一次函数调用，在这个上下文中就是只会进行一次数据库查询，查询的结果会写回 Redis 并同步给所有请求对应 `Key` 的用户：

![golang-extension-single-flight](go语言设计与实现.assets/3720534bfdbdf291f169a092d4cfbcc4.png)

这其实就减少了对下游的瞬时流量，在获取下游资源非常耗时，例如：访问缓存、数据库等场景下就非常适合使用 `singleflight` 对服务进行优化，在上述的这个例子中我们就可以在想 Redis 和数据库中获取数据时都使用 `singleflight` 提供的这一功能减少下游的压力；它的使用其实也非常简单，我们可以直接使用 `singleflight.Group{}` 创建一个新的 `Group` 结构体，然后通过调用 `Do` 方法就能对相同的请求进行抑制：

```
type service struct {
    requestGroup singleflight.Group
}
func (s *service) handleRequest(ctx context.Context, request Request) (Response, error) {
    v, err, _ := requestGroup.Do(request.Hash(), func() (interface{}, error) {
        rows, err := // select * from tables
        if err != nil {
            return nil, err
        }
        return rows, nil
    })
    if err != nil {
        return nil, err
    }
    return Response{
        rows: rows,
    }, nil
}
```

上述代码使用请求的哈希作为抑制相同请求的键，我们也可以选择一些比较关键或者重要的字段作为 `Do` 方法的第一个参数避免对下游的瞬时大量请求。

##### 结构体

`Group` 结构体本身由一个互斥锁 `Mutex` 和一个从 `Key` 到 `call` 结构体指针的映射表组成，每一个 `call` 结构体都保存了当前这次调用对应的信息：

```
type Group struct {
    mu sync.Mutex
    m  map[string]*call
}
type call struct {
    wg sync.WaitGroup
    val interface{}
    err error
    dups  int
    chans []chan<- Result
}
```

`call` 结构体中的 `val` 和 `err` 字段都是在执行传入的函数时只会被赋值一次，它们也只会在 `WaitGroup` 等待结束都被读取，而 `dups` 和 `chans` 字段分别用于存储当前 `singleflight` 抑制的请求数量以及在结果返回时将信息传递给调用方。

##### 操作

`singleflight` 包提供了两个用于抑制相同请求的方法，其中一个是同步等待的方法 `Do`，另一个是返回 Channel 的 `DoChan`，这两个方法在功能上没有太多的区别，只是在接口的表现上稍有不同。

每次 `Do` 方法的调用时都会获取互斥锁并尝试对 `Group` 持有的映射表进行懒加载，随后判断是否已经存在 `key` 对应的函数调用：

- 当不存在对应的call 

  结构体时：

  - 初始化一个新的 `call` 结构体指针；
  - 增加 `WaitGroup` 持有的计数器；
  - 将 `call` 结构体指针添加到映射表；
  - 释放持有的互斥锁 `Mutex`；
  - 阻塞地调用 `doCall` 方法等待结果的返回；

- 当已经存在对应的call结构体时；

  - 增加 `dups` 计数器，它表示当前重复的调用次数；
  - 释放持有的互斥锁 `Mutex`；
  - 通过 `WaitGroup.Wait` 等待请求的返回；

```
func (g *Group) Do(key string, fn func() (interface{}, error)) (v interface{}, err error, shared bool) {
    g.mu.Lock()
    if g.m == nil {
        g.m = make(map[string]*call)
    }
    if c, ok := g.m[key]; ok {
        c.dups++
        g.mu.Unlock()
        c.wg.Wait()
        return c.val, c.err, true
    }
    c := new(call)
    c.wg.Add(1)
    g.m[key] = c
    g.mu.Unlock()
    g.doCall(c, key, fn)
    return c.val, c.err, c.dups > 0
}
```

因为 `val` 和 `err` 两个字段都只会在 `doCall` 方法中被赋值，所以当 `doCall` 方法和 `WaitGroup.Wait` 方法返回时，这两个值就会返回给 `Do` 函数的调用者。

```
func (g *Group) doCall(c *call, key string, fn func() (interface{}, error)) {
    c.val, c.err = fn()
    c.wg.Done()
    g.mu.Lock()
    delete(g.m, key)
    for _, ch := range c.chans {
        ch <- Result{c.val, c.err, c.dups > 0}
    }
    g.mu.Unlock()
}
```

`doCall` 中会运行传入的函数 `fn`，该函数的返回值就会赋值给 `c.val` 和 `c.err`，函数执行结束后就会调用 `WaitGroup.Done` 方法通知所有被抑制的请求，当前函数已经执行完成，可以从 `call` 结构体中取出返回值并返回了；在这之后，`doCall` 方法会获取持有的互斥锁并通过管道将信息同步给使用 `DoChan` 方法的调用方。

```
func (g *Group) DoChan(key string, fn func() (interface{}, error)) <-chan Result {
    ch := make(chan Result, 1)
    g.mu.Lock()
    if g.m == nil {
        g.m = make(map[string]*call)
    }
    if c, ok := g.m[key]; ok {
        c.dups++
        c.chans = append(c.chans, ch)
        g.mu.Unlock()
        return ch
    }
    c := &call{chans: []chan<- Result{ch}}
    c.wg.Add(1)
    g.m[key] = c
    g.mu.Unlock()
    go g.doCall(c, key, fn)
    return ch
}
```

`DoChan` 方法和 `Do`的区别就是，它使用 Goroutine 异步执行 `doCall` 并向 `call` 持有的 `chans` 切片中追加 `chan Result` 变量，这也是它能够提供异步传值的原因。



##### 小结

`singleflight` 包提供的 `Group` 接口确实非常好用，当我们需要这种抑制对下游的相同请求时就可以通过这个方法来增加吞吐量和服务质量，在使用的过程中我们也需要注意以下的几个问题：

- `Do` 和 `DoChan` 一个用于同步阻塞调用传入的函数，一个用于异步调用传入的参数并通过 Channel 接受函数的返回值；
- `Forget` 方法可以通知 `singleflight` 在持有的映射表中删除某个键，接下来对该键的调用就会直接执行方法而不是等待前面的函数返回；
- 一旦调用的函数返回了错误，所有在等待的 Goroutine 也都会接收到同样的错误；



### 总结

我们在这一节中介绍了 Go 语言标准库中提供的基本原语以及扩展包中的扩展原语，这些并发编程的原语能够帮助我们更好地利用 Go 语言的特性构建高吞吐量、低延时的服务，并解决由于并发带来的错误，到这里我们再重新回顾一下这一节介绍的内容：

- ```
  Mutex
  ```

  互斥锁

  - 如果互斥锁处于初始化状态，就会直接通过置位 `mutexLocked` 加锁；
  - 如果互斥锁处于 `mutexLocked` 并且在普通模式下工作，就会进入自旋，执行 30 次 `PAUSE` 指令消耗 CPU 时间等待锁的释放；
  - 如果当前 Goroutine 等待锁的时间超过了 `1ms`，互斥锁就会被切换到饥饿模式；
  - 互斥锁在正常情况下会通过 `runtime_SemacquireMutex` 方法将调用 `Lock` 的 Goroutine 切换至休眠状态，等待持有信号量的 Goroutine 唤醒当前协程；
  - 如果当前 Goroutine 是互斥锁上的最后一个等待的协程或者等待的时间小于 `1ms`，当前 Goroutine 会将互斥锁切换回正常模式；
  - 如果互斥锁已经被解锁，那么调用 `Unlock` 会直接抛出异常；
  - 如果互斥锁处于饥饿模式，会直接将锁的所有权交给队列中的下一个等待者，等待者会负责设置 `mutexLocked` 标志位；
  - 如果互斥锁处于普通模式，并且没有 Goroutine 等待锁的释放或者已经有被唤醒的 Goroutine 获得了锁就会直接返回，在其他情况下回通过 `runtime_Semrelease` 唤醒对应的 Goroutine；

- ```
  RWMutex
  ```

   读写互斥锁

  - `readerSem` — 读写锁释放时通知由于获取读锁等待的 Goroutine；
  - `writerSem` — 读锁释放时通知由于获取读写锁等待的 Goroutine；
  - `w` 互斥锁 — 保证写操作之间的互斥；
  - `readerCount` — 统计当前进行读操作的协程数，触发写锁时会将其减少 `rwmutexMaxReaders` 阻塞后续的读操作；
  - `readerWait` — 当前读写锁等待的进行读操作的协程数，在触发 `Lock` 之后的每次 `RUnlock` 都会将其减一，当它归零时该 Goroutine 就会获得读写锁；
  - 当读写锁被释放 `Unlock` 时首先会通知所有的读操作，然后才会释放持有的互斥锁，这样能够保证读操作不会被连续的写操作『饿死』；

- ```
  WaitGroup
  ```

   等待一组 Goroutine 结束

  - `Add` 不能在和 `Wait` 方法在 Goroutine 中并发调用，一旦出现就会造成程序崩溃；
  - `WaitGroup` 必须在 `Wait` 方法返回之后才能被重新使用；
  - `Done` 只是对 `Add` 方法的简单封装，我们可以向 `Add` 方法传入任意负数（需要保证计数器非负）快速将计数器归零以唤醒其他等待的 Goroutine；
  - 可以同时有多个 Goroutine 等待当前 `WaitGroup` 计数器的归零，这些 Goroutine 也会被『同时』唤醒；

- ```
  Once
  ```

   程序运行期间仅执行一次

  - `Do` 方法中传入的函数只会被执行一次，哪怕函数中发生了 `panic`；
  - 两次调用 `Do` 方法传入不同的函数时只会执行第一次调用的函数；

- ```
  Cond
  ```

   发生指定事件时唤醒

  - `Wait` 方法在调用之前一定要使用 `L.Lock` 持有该资源，否则会发生 `panic` 导致程序崩溃；
  - `Signal` 方法唤醒的 Goroutine 都是队列最前面、等待最久的 Goroutine；
  - `Broadcast` 虽然是广播通知全部等待的 Goroutine，但是真正被唤醒时也是按照一定顺序的；

- ```
  ErrGroup
  ```

   为一组 Goroutine 提供同步、错误传播以及上下文取消的功能

  - 出现错误或者等待结束后都会调用 `Context` 的 `cancel` 方法取消上下文；
  - 只有第一个出现的错误才会被返回，剩余的错误都会被直接抛弃；

- ```
  Semaphore
  ```

   **带权重的信号量**

  - `Acquire` 和 `TryAcquire` 方法都可以用于获取资源，前者用于同步获取会等待锁的释放，后者会在无法获取锁时直接返回；
  - `Release` 方法会按照 FIFO 的顺序唤醒可以被唤醒的 Goroutine；
  - 如果一个 Goroutine 获取了较多地资源，由于 `Release` 的释放策略可能会等待比较长的时间；

- ```
  SingleFlight
  ```

   用于抑制对下游的重复请求

  - `Do` 和 `DoChan` 一个用于同步阻塞调用传入的函数，一个用于异步调用传入的参数并通过 Channel 接受函数的返回值；
  - `Forget` 方法可以通知 `singleflight` 在持有的映射表中删除某个键，接下来对该键的调用就会直接执行方法而不是等待前面的函数返回；
  - 一旦调用的函数返回了错误，所有在等待的 Goroutine 也都会接收到同样的错误；

这些同步原语的实现不仅要考虑 API 接口的易用、解决并发编程中可能遇到的线程竞争问题，还需要对尾延时进行优化避免某些 Goroutine 无法获取锁或者资源而被饿死，对同步原语的学习也能够增强我们队并发编程的理解和认识，也是了解并发编程无法跨越的一个步骤。

### Reference

- [sync: make Mutex more fair](https://go-review.googlesource.com/c/go/+/34310)
- [runtime: fall back to fair locks after repeated sleep-acquire failures #13086](https://github.com/golang/go/issues/13086)
- [x/sync · Golang](https://godoc.org/golang.org/x/sync)
- [The Go Memory Model](https://golang.org/ref/mem)
- [The X-Files: Exploring the Golang Standard Library Sub-Repositories](https://rodaine.com/2017/05/x-files-intro/)
- [Go: Avoid duplicate requests with sync/singleflight](https://medium.com/@vCabbage/go-avoid-duplicate-requests-with-sync-singleflight-311601b3068b)









## 6.3 定时器

- [6.3 定时器](https://www.bookstack.cn/read/draveness-golang/326bb5c45fbed045.md)























































## 6.4 Channel

- [6.4 Channel](https://www.bookstack.cn/read/draveness-golang/c666731d5f1a2820.md)

这一节中的内容总共包含两个部分，我们会先介绍 Channel 的设计原理以及它在 Go 语言中的数据结构，接下来我们会分析常见的 Channel 操作，例如创建、发送、接收和关闭的实现原理，由于在 [Range](https://www.bookstack.cn/read/draveness-golang/816b8b0f4e15285f.md) 和 [Select](https://www.bookstack.cn/read/draveness-golang/1a4f7a284cd2b279.md) 两节中我们会提到 Channel 在不同的控制结构中组合使用时的现象，所以这一节还是会将重点放到 Channel 的常见操作上。



### 概述

作为 Go 语言中核心的数据结构和 Goroutine 之间的通信方式，Channel 是支撑 Go 语言高性能并发编程模型的结构之一，我们首先需要了解 Channel 背后的设计原理以及它的底层数据结构。

#### 设计原理

在 Go 语言中，一个最常见的也是经常被人提及的设计模式就是**不要通过共享内存的方式进行通信，而是应该通过通信的方式共享内存**，在很多主流的编程语言中，当我们想要并发执行一些代码时，我们往往都会在多个线程之间共享变量，同时为了解决线程冲突的问题，我们又需要在读写这些变量时加锁。

![Golang-Channel-Share-Memory](go语言设计与实现.assets/f9ed27afad45d2cb4ce29b113b40039a.png)

Go 语言对于并发编程的设计与上述这种共享内存的方式完全不同，虽然我们在 Golang 中也能使用共享内存加互斥锁来实现并发编程，但是与此同时，Go 语言也提供了一种不同的并发模型，也就是 CSP，即通信顺序进程（Communicating sequential processes），Goroutine 其实就是 CSP 中的实体，Channel 就是用于传递信息的通道，使用 CSP 并发模型的 Goroutine 就会通过 Channel 来传递消息。











- [6.5 Goroutine](https://www.bookstack.cn/read/draveness-golang/488ee05eea5e58a3.md)



# 第七章 内存管理

## 7.1 内存分配器

> [7.1 内存分配器](https://www.bookstack.cn/read/draveness-golang/3a89cfd15249f9fe.md)

程序中的数据和变量都会被分配到程序所在的虚拟内存中，内存空间包含两个重要区域 — 栈区（Stack）和堆区（Heap）。函数调用的参数、返回值以及局部变量大都会被分配到栈上，这部分内存会由编译器进行管理；不同编程语言使用不同的方法管理堆区的内存，C++ 等编程语言会由工程师主动申请和释放内存，Go 以及 Java 等编程语言会由工程师和编译器共同管理，堆中的对象由内存分配器分配并由垃圾收集器回收。

不同的编程语言会选择不同的方式管理内存，本节会介绍 Go 语言内存分配器，详细分析内存分配的过程以及其背后的设计与实现原理。



- [7.2 垃圾收集器](https://www.bookstack.cn/read/draveness-golang/30cd28c181a56e61.md)
- [7.3 栈内存管理](https://www.bookstack.cn/read/draveness-golang/4a6101b89b1d41f1.md)



# 第八章 元编程

- [8.1 插件系统](https://www.bookstack.cn/read/draveness-golang/6588a87b3f79f21b.md)
- [8.2 代码生成](https://www.bookstack.cn/read/draveness-golang/19ab6c46e7c9de36.md)
- 

# 第九章 标准库

## 9.1 JSON

> [9.1 JSON](https://www.bookstack.cn/read/draveness-golang/f6b06e1693601b3a.md)

JSON（JavaScript 对象表示，JavaScript Object Notation）作为一种轻量级的数据交换格式[1](https://www.bookstack.cn/read/draveness-golang/f6b06e1693601b3a.md#fn:1)，在今天几乎占据了绝大多数的市场份额。虽然与更紧凑的数据交换格式相比，它的序列化和反序列化性能不足，但是它也提供了良好的可读性与易用性，在不追求机制性能的情况下，JSON 是一种非常好的选择。

### 9.1.1 设计原理

几乎所有的现代编程语言都会将处理 JSON 的函数直接纳入标准库，Go 语言也不例外，它通过 [`encoding/json`](https://golang.org/pkg/encoding/json/) 对外提供标准的 JSON 序列化和反序列化方法，即 [`encoding/json.Marshal`](https://github.com/golang/go/blob/e43239aabe2fbbd8ad08b46fb2a7c3d9a4d36589/src/encoding/json/encode.go#L158) 和 [`encoding/json.Unmarshal`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L96)，它们也是包中最常用的两个方法。

![json-marshal-and-unmarshal](go语言设计与实现.assets/9fb42af8026af691f73e5132bc392d41.png)

**图 9-1 序列化和反序列化**

序列化和反序列化的开销完全不同，JSON 反序列化的开销是序列化开销的好几倍，相信这背后的原因也非常好理解。

Go 语言中的 JSON 序列化过程不需要被序列化的对象预先实现任何接口，它会通过反射获取结构体或者数组中的值并以树形的结构递归地进行编码，标准库也会根据 [`encoding/json.Unmarshal`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L96) 中传入的值对 JSON 进行解码。

**Go 语言 JSON 标准库编码和解码的过程大量地运用了反射这一特性**，你会在本节的后半部分看到大量的反射代码，这里就不过多介绍了。我们在这里会简单介绍 JSON 标准库中的接口和标签，这是它为开发者提供的为数不多的影响编解码过程的接口。



#### 接口

JSON 标准库中提供了 [`encoding/json.Marshaler`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/encode.go#L225) 和 [`encoding/json.Unmarshaler`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L118) 两个接口分别可以影响 JSON 的序列化和反序列化结果：

```go
type Marshaler interface {
    MarshalJSON() ([]byte, error)
}
type Unmarshaler interface {
    UnmarshalJSON([]byte) error
}
```

> go marshaler序列化结果是[]byte,可以后续格式化为string

在 JSON 序列化和反序列化的过程中，它们会使用反射判断结构体类型是否实现了上述接口，如果实现了上述接口就会优先使用对应的方法进行编码和解码操作，除了这两个方法之外，Go 语言其实还提供了另外两个用于控制编解码结果的方法，即 [`encoding.TextMarshaler`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/encoding.go#L36) 和 [`encoding.TextUnmarshaler`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/encoding.go#L45)：

```go
type TextMarshaler interface {
    MarshalText() (text []byte, err error)
}
type TextUnmarshaler interface {
    UnmarshalText(text []byte) error
}
```

一旦发现 JSON 相关的序列化方法没有被实现，上述两个方法会作为候选方法被 JSON 标准库调用，参与编解码的过程。总得来说，我们可以在任意类型上实现上述这四个方法自定义最终的结果，后面的两个方法的适用范围更广，但是不会被 JSON 标准库优先调用。



#### 标签

Go 语言的结构体标签也是一个比较有趣的功能，在默认情况下，当我们在序列化和反序列化结构体时，标准库都会认为字段名和 JSON 中的键具有一一对应的关系，然而 Go 语言的字段一般都是驼峰命名法，JSON 中下划线的命名方式相对比较常见，所以使用标签这一特性直接建立键与字段之间的映射关系是一个非常方便的设计。

![struct-and-json](go语言设计与实现.assets/07ab90e9ad7c487edd562b089d5a736b.png)

**图 9-2 结构体与 JSON 的映射**

JSON 中的标签由两部分组成，如下所示的 `name` 和 `age` 都是标签名，后面的所有的字符串是标签选项，即 [`encoding/json.tagOptions`](https://github.com/golang/go/blob/50bd1c4d4eb4fac8ddeb5f063c099daccfb71b26/src/encoding/json/tags.go#L13)，标签名和字段名会建立一一对应的关系，后面的标签选项也会影响编解码的过程：

```go
type Author struct {
    Name string `json:"name,omitempty"`
    Age  int32  `json:"age,string,omitempty"`
}
```

常见的两个标签是 `string` 和 `omitempty`，前者表示当前的整数或者浮点数是由 JSON 中的字符串表示的，而另一个字段 `omitempty` 会在字段为零值时，直接在生成的 JSON 中忽略对应的键值对，例如：`"age": 0`、`"author": ""` 等。标准库会使用 [`encoding/json.parseTag`](https://github.com/golang/go/blob/e43239aabe2fbbd8ad08b46fb2a7c3d9a4d36589/src/encoding/json/tags.go#L17) 函数来解析标签：

```go
func parseTag(tag string) (string, tagOptions) {
    if idx := strings.Index(tag, ","); idx != -1 {
        return tag[:idx], tagOptions(tag[idx+1:])
    }
    return tag, tagOptions("")
}
```

从该方法的实现中，我们能分析出 JSON 标准库中的合法标签是什么形式的 — **标签名和标签选项**都以 `,` 连接，最前面的字符串为标签名，后面的都是标签选项。

> 每个标签会自己定义标签选项的格式，比如gorm的标签选项是以；分隔
>
> ```go
> type UserPointFlowDbRecord struct {
> 	Id         uint64               `gorm:"column:id;primary_key;AUTO_INCREMENT;NOT NULL" json:"id,omitempty"`
> }
> ```



### 9.1.2 序列化

[`encoding/json.Marshal`](https://github.com/golang/go/blob/e43239aabe2fbbd8ad08b46fb2a7c3d9a4d36589/src/encoding/json/encode.go#L158) 是 JSON 标准库中提供的最简单的序列化函数，它会接收一个 `interface{}` 类型的值作为参数，这也意味着几乎全部的 Go 语言变量都可以被 JSON 标准库序列化，为了提供如此复杂和通用的功能，在静态语言中使用反射是常见的选项，我们来深入了解一下该方法的实现：

```
func Marshal(v interface{}) ([]byte, error) {
    e := newEncodeState()
    err := e.marshal(v, encOpts{escapeHTML: true})
    if err != nil {
        return nil, err
    }
    buf := append([]byte(nil), e.Bytes()...)
    encodeStatePool.Put(e)
    return buf, nil
}
```

上述方法会调用 [`encoding/json.newEncodeState`](https://github.com/golang/go/blob/e43239aabe2fbbd8ad08b46fb2a7c3d9a4d36589/src/encoding/json/encode.go#L302) 从全局的编码状态池中获取 [`encoding/json.encodeState`](https://github.com/golang/go/blob/e43239aabe2fbbd8ad08b46fb2a7c3d9a4d36589/src/encoding/json/encode.go#L285)，随后的序列化过程都会使用这个编码状态，该结构体也会在编码结束后被重新放回池中以便重复利用。

![json-marshal-call-stack](go语言设计与实现.assets/fd75e905c515a8f292e3b327c16bbac1.png)

**图 9-3 序列化调用栈**



按照如上所示的复杂调用栈，一系列的序列化方法在最后获取了对象的反射类型并调用了 [`encoding/json.newTypeEncoder`](https://github.com/golang/go/blob/e43239aabe2fbbd8ad08b46fb2a7c3d9a4d36589/src/encoding/json/encode.go#L415) 这个核心的编码方法，该方法会**递归**地为所有的类型找到对应的编码方法，不过它的执行过程可以分成以下两个步骤：

1. 获取用户自定义的 [`encoding/json.Marshaler`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/encode.go#L225) 或者 [`encoding.TextMarshaler`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/encoding.go#L36) 编码器；
2. 获取标准库中为基本类型内置的 JSON 编码器；

在该方法的第一部分，我们会检查当前值的类型是否可以使用用户自定义的编码器，这里有两种不同的判断方法：

```go
func newTypeEncoder(t reflect.Type, allowAddr bool) encoderFunc {
    if t.Kind() != reflect.Ptr && allowAddr && reflect.PtrTo(t).Implements(marshalerType) {
        return newCondAddrEncoder(addrMarshalerEncoder, newTypeEncoder(t, false))
    }
    if t.Implements(marshalerType) {
        return marshalerEncoder
    }
    if t.Kind() != reflect.Ptr && allowAddr && reflect.PtrTo(t).Implements(textMarshalerType) {
        return newCondAddrEncoder(addrTextMarshalerEncoder, newTypeEncoder(t, false))
    }
    if t.Implements(textMarshalerType) {
        return textMarshalerEncoder
    }
    ...
}
```



[`encoding/json.newTypeEncoder`](https://github.com/golang/go/blob/e43239aabe2fbbd8ad08b46fb2a7c3d9a4d36589/src/encoding/json/encode.go#L415) 方法随后会根据传入值的反射类型获取对应的编码器，其中包括 `bool`、`int`、`float` 等基本类型编码器等和数组、结构体、切片等复杂类型的编码器：

```go
func newTypeEncoder(t reflect.Type, allowAddr bool) encoderFunc {
    ...
    switch t.Kind() {
    case reflect.Bool:
        return boolEncoder
    case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
        return intEncoder
    case reflect.Uint, reflect.Uint8, reflect.Uint16, reflect.Uint32, reflect.Uint64, reflect.Uintptr:
        return uintEncoder
    case reflect.Float32:
        return float32Encoder
    case reflect.Float64:
        return float64Encoder
    case reflect.String:
        return stringEncoder
    case reflect.Interface:
        return interfaceEncoder
    case reflect.Struct:
        return newStructEncoder(t)
    case reflect.Map:
        return newMapEncoder(t)
    case reflect.Slice:
        return newSliceEncoder(t)
    case reflect.Array:
        return newArrayEncoder(t)
    case reflect.Ptr:
        return newPtrEncoder(t)
    default:
        return unsupportedTypeEncoder
    }
}
```



我们在这里就不一一介绍全部的内置类型编码器了，只挑选其中的几个帮助各位读者了解整体的设计。首先我们来看布尔值的 JSON 编码器，它的实现很简单，甚至没有太多值得介绍的地方：

```go
func boolEncoder(e *encodeState, v reflect.Value, opts encOpts) {
    if opts.quoted {
        e.WriteByte('"')
    }
    if v.Bool() {
        e.WriteString("true")
    } else {
        e.WriteString("false")
    }
    if opts.quoted {
        e.WriteByte('"')
    }
}
```



它会根据当前值向编码状态中写入不同的字符串，也就是 `true` 或者 `false`，除此之外还会根据编码配置决定是否要在布尔值周围写入双引号 `"`，而其他的基本类型编码器也都大同小异。

复杂类型的编码器有着相对复杂的控制结构，我们在这里以结构体的编码器 [`encoding/json.structEncoder`](https://github.com/golang/go/blob/e43239aabe2fbbd8ad08b46fb2a7c3d9a4d36589/src/encoding/json/encode.go#L720) 为例介绍它们的原理，[`encoding/json.newStructEncoder`](https://github.com/golang/go/blob/e43239aabe2fbbd8ad08b46fb2a7c3d9a4d36589/src/encoding/json/encode.go#L767) 会为当前结构体的所有字段调用 [`encoding/json.typeEncoder`](https://github.com/golang/go/blob/e43239aabe2fbbd8ad08b46fb2a7c3d9a4d36589/src/encoding/json/encode.go#L379) 获取类型编码器并返回 [`encoding/json.structEncoder.encode`](https://github.com/golang/go/blob/e43239aabe2fbbd8ad08b46fb2a7c3d9a4d36589/src/encoding/json/encode.go#L729) 方法：

```go
func newStructEncoder(t reflect.Type) encoderFunc {
    se := structEncoder{fields: cachedTypeFields(t)}
    return se.encode
}
```

从 [`encoding/json.structEncoder.encode`](https://github.com/golang/go/blob/e43239aabe2fbbd8ad08b46fb2a7c3d9a4d36589/src/encoding/json/encode.go#L729) 的实现我们能看出结构体序列的结果，该方法会遍历结构体中的全部字段，在写入了字段名后，它会调用字段对应类型的编码方法将该字段对应的 JSON 写入缓冲区：

```go
func (se structEncoder) encode(e *encodeState, v reflect.Value, opts encOpts) {
    next := byte('{')
FieldLoop:
    for i := range se.fields.list {
        f := &se.fields.list[i]
        fv := v
        for _, i := range f.index {
            if fv.Kind() == reflect.Ptr {
                if fv.IsNil() {
                    continue FieldLoop
                }
                fv = fv.Elem()
            }
            fv = fv.Field(i)
        }
        if f.omitEmpty && isEmptyValue(fv) {
            continue
        }
        e.WriteByte(next)
        next = ','
        e.WriteString(f.nameNonEsc)
        opts.quoted = f.quoted
        f.encoder(e, fv, opts)
    }
    if next == '{' {
        e.WriteString("{}")
    } else {
        e.WriteByte('}')
    }
}
```

数组以及指针等编码器的实现原理与该方法也没有太多的区别，它们都会使用类似的策略递归地调用持有字段的编码方法，这也就能形成一个如下图所示的树形结构：

![struct-encoder](go语言设计与实现.assets/3f472931598a7ba44858ed85c1569771.png)

**图 9-4 序列化与树形结构体**

树形结构的所有叶节点都是基础类型编码器或者开发者自定义的编码器，得到了整棵树的编码器之后会调用 [`encoding/json.encodeState.reflectValue`](https://github.com/golang/go/blob/e43239aabe2fbbd8ad08b46fb2a7c3d9a4d36589/src/encoding/json/encode.go#L357) 从根节点依次调用整棵树的序列化函数，整个 JSON 序列化的过程其实是查找类型和子类型的编码方法并调用的过程，它利用了大量反射的特性做到了足够的通用。



### 9.1.3 反序列化

标准库会使用 [`encoding/json.Unmarshal`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L96) 函数处理 JSON 的反序列化，**与执行过程确定的序列化相比，反序列化的过程比较像一个逐渐探索的过程**，所以会复杂很多，开销也会高出几倍。因为 Go 语言的表达能力比较有限，反序列化的使用相对比较繁琐，需要传入一个变量帮助标准库进行反序列化：

```go
func Unmarshal(data []byte, v interface{}) error {
    var d decodeState
    err := checkValid(data, &d.scan)
    if err != nil {
        return err
    }
    d.init(data)
    return d.unmarshal(v)
}
```

在真正执行反序列化之前，我们会先调用 [`encoding/json.checkValid`](https://github.com/golang/go/blob/a1103dcc27b9c85800624367ebb89ef46d4307af/src/encoding/json/scanner.go#L30) 验证传入 JSON 的合法性保证在反序列化的过程中不会遇到语法错误的问题，在通过合法性的验证之后，标准库就会初始化数据并调用 [`encoding/json.decodeState.unmarshal`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L170) 开始反序列化了：

```go
func (d *decodeState) unmarshal(v interface{}) error {
    rv := reflect.ValueOf(v)
    if rv.Kind() != reflect.Ptr || rv.IsNil() {
        return &InvalidUnmarshalError{reflect.TypeOf(v)}
    }
    d.scan.reset()
    d.scanWhile(scanSkipSpace)
    err := d.value(rv)
    if err != nil {
        return d.addErrorContext(err)
    }
    return d.savedError
}
```

如果传入的值不是指针或者是空指针，当前方法就会返回我们经常会见到的错误 [`encoding/json.InvalidUnmarshalError`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L155)，使用格式化输出可以将该错误转换成 `json: Unmarshal(non-pointer xxx)`。该方法调用的 [`encoding/json.decodeState.value`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L370) 是所有反序列化过程的执行入口：

```go
func (d *decodeState) value(v reflect.Value) error {
    switch d.opcode {
    default:
        panic(phasePanicMsg)
    case scanBeginArray:
        ...
    case scanBeginLiteral:
        ...
    case scanBeginObject:
        if v.IsValid() {
            if err := d.object(v); err != nil {
                return err
            }
        } else {
            d.skip()
        }
        d.scanNext()
    }
    return nil
}
```

该方法作为最顶层的反序列化方法可以接收三种不同类型的值，也就是数组、字面量和对象，这三种类型都可以作为 JSON 的顶层对象，我们首先来了解一下标准库是如何解析 JSON 中对象的，该过程会使用 [`encoding/json.decodeState.object`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L620) 函数进行反序列化，它会先调用 [`encoding/json.indirect`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L439) 函数查找当前类型对应的非指针类型：

```go
func (d *decodeState) object(v reflect.Value) error {
    u, ut, pv := indirect(v, false)
    if u != nil {
        start := d.readIndex()
        d.skip()
        return u.UnmarshalJSON(d.data[start:d.off])
    }
    ...
}
```

在调用 [`encoding/json.indirect`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L439) 的过程中，如果当前值的类型是 `**Type`，那么它会依次检查形如 `**Type`、`*Type` 和 `Type` 类型是否实现了 [`encoding/json.Unmarshal`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L96) 或者 [`encoding.TextUnmarshaler`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/encoding.go#L45) 接口；如果实现了该接口，标准库会直接调用 `UnmarshalJSON` 方法使用开发者定义的方法完成反序列化。

在其他情况下，我们仍然会回到默认的逻辑中处理对象中的键值对，如下所示的代码会调用 [`encoding/json.decodeState.rescanLiteral`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L315) 方法扫描 JSON 中的键并在结构体中找到对应字段的反射值，接下来继续扫描符号 `:` 并调用 [`encoding/json.decodeState.value`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L370) 解析对应的值：

```go
func (d *decodeState) object(v reflect.Value) error {
    ...
    v = pv
    t := v.Type()
    fields = cachedTypeFields(t)
    for {
        start := d.readIndex()
        d.rescanLiteral()
        item := d.data[start:d.readIndex()]
        key, _ := d.unquoteBytes(item)
        var subv reflect.Value
        var f *field
        if i, ok := fields.nameIndex[string(key)]; ok {
            f = &fields.list[i]
        }
        if f != nil {
            subv = v
            for _, i := range f.index {
                subv = subv.Field(i)
            }
        }
        if d.opcode != scanObjectKey {
            panic(phasePanicMsg)
        }
        d.scanWhile(scanSkipSpace)
        if err := d.value(subv); err != nil {
            return err
        }
        if d.opcode == scanEndObject {
            break
        }
    }
    return nil
}
```

当上述方法调用 [`encoding/json.decodeState.value`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L370) 时，该方法会重新判断键对应的值是否是对象、数组或者字面量，因为数组和对象都是集合类型，所以该方法会递归地进行扫描，在这里也就不再继续介绍这些集合类型的解析过程了，我们来简单分析一下字面量是如何被处理的：

```go
func (d *decodeState) value(v reflect.Value) error {
    switch d.opcode {
    default:
        panic(phasePanicMsg)
    case scanBeginArray:
        ...
    case scanBeginObject:
        ...
    case scanBeginLiteral:
        start := d.readIndex()
        d.rescanLiteral()
        if v.IsValid() {
            if err := d.literalStore(d.data[start:d.readIndex()], v, false); err != nil {
                return err
            }
        }
    }
    return nil
}
```

字面量的扫描会通过 [`encoding/json.decodeState.rescanLiteral`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L315)，该方法会依次扫描缓冲区中的字符并根据字符的不同对字符串进行切片，整个过程有点像编译器的词法分析：

```go
func (d *decodeState) rescanLiteral() {
    data, i := d.data, d.off
Switch:
    switch data[i-1] {
    case '"': // string
        ...
    case '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '-': // number
        ...
    case 't': // true
        i += len("rue")
    case 'f': // false
        i += len("alse")
    case 'n': // null
        i += len("ull")
    }
    if i < len(data) {
        d.opcode = stateEndValue(&d.scan, data[i])
    } else {
        d.opcode = scanEnd
    }
    d.off = i + 1
}
```

因为 JSON 中的字面量其实也只包含字符串、数字、布尔值和空值几种，所以该方法的实现也不会特别复杂，当该方法扫描了对应的字面量之后，就会调用 [`encoding/json.decodeState.literalStore`](https://github.com/golang/go/blob/ee8972cd126f0d575be90c0ffbb08fc09d6ede19/src/encoding/json/decode.go#L867) 字面量存储到反射类型变量所在的地址中，在这个过程中会调用反射的 [`reflect.Value.SetInt`](https://github.com/golang/go/blob/a9f8f02f3cfba18501669cdf58ae75ca36a4cff0/src/reflect/value.go#L1611)、[`reflect.Value.SetFloat`](https://github.com/golang/go/blob/a9f8f02f3cfba18501669cdf58ae75ca36a4cff0/src/reflect/value.go#L1597) 和 [`reflect.Value.SetBool`](https://github.com/golang/go/blob/a9f8f02f3cfba18501669cdf58ae75ca36a4cff0/src/reflect/value.go#L1553) 等方法。



### 9.1.4 小结

JSON 本身就是一种树形的数据结构，无论是序列化还是反序列化，都会遵循自顶向下的编码和解码过程，使用递归的方式处理 JSON 对象。作为标准库的 JSON 提供的接口非常简洁，虽然它的性能一直被开发者所诟病，但是作为框架它提供了很好的通用性，通过分析 JSON 库的实现，我们也可以从中学习到使用反射的各种方法。

1. Introducing JSON https://www.json.org/json-en.html [↩︎](https://www.bookstack.cn/read/draveness-golang/f6b06e1693601b3a.md#fnref:1)

