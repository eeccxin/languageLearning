# **资料**

[Swoole_ZyBlog码农老张](http://www.zyblog.com.cn/t/Swoole/p/2)

wiki：[Swoole 文档](https://wiki.swoole.com/#/server/setting?id=dispatch_mode)



# Swoole文档

> wiki：[Swoole 文档](https://wiki.swoole.com/#/server/setting?id=dispatch_mode)





# Swoole系列博客

> **微信文章地址：**https://mp.weixin.qq.com/s/ijBA7cXjjfxQ_oNazeRk1A
>
> **B站视频地址：**https://www.bilibili.com/video/BV1eD4y167Um
>
> **微信视频地址：**https://mp.weixin.qq.com/s/_pDmdBRd48oPdZMO2UtrDg





## **【Swoole系列1】在Swoole的世界中，你将学习到什么？**

> https://www.zyblog.com.cn/article/264

在接下来的学习中，我们将要接触到的，将是 **PHP 扩展**中非常出名的一个高大上的框架，那就是 Swoole 。或许你已经在生产环境中使用过了，或许你只是看过官方文档写过几个例子，当然，更有可能你只是听过它的名字。

不用太担心，通过我们的学习，你将会掌握到基本的 Swoole 开发知识，一些计算机操作系统以及网络的简单知识，以及**一个非常类似于 Laravel 的 Swoole 框架**。相信通过这些，你就可以尝试在自己的真实项目中使用 Swoole 来做一些项目，体验 PHP 最为人诟病的效率问题的超强解决方案。





### Swoole

我们先来看看 Swoole 是什么。

> Swoole 使 PHP 开发人员可以编写**高性能高并发**的 TCP、UDP（传输层协议）、Unix Socket（套接字类型）、HTTP、 WebSocket（应用层协议） 等服务，让 PHP 不再局限于 Web 领域。Swoole4 协程的成熟将 PHP 带入了前所未有的时期， 为性能的提升提供了独一无二的可能性。Swoole 可以广泛应用于互联网、移动通信、云计算、 网络游戏、物联网（IOT）、车联网、智能家居等领域。使用 PHP + Swoole 可以使企业 IT 研发团队的效率大大提升，更加专注于开发创新产品。

上面这段是官方网站首页上对于 Swoole 的说明。看着就感觉高大上吧？其实，TCP、UDP、Unix Socket、HTTP、 WebSocket 这些，我们普通 PHP 也能做到，但是，通常我们在进行普通的 Web 开发时，都会借助一个服务器应用，比如说 Apache 或者 Nginx 来配合 fastcgi 进行实现。而在 Swoole 中，只需要运行起 Swoole 服务就可以实现这些服务的挂载了。当然，我们还可以在外面套上 Nginx ，这样可以更方便地管理应用地址（域名）。

此外，在官方描述中，**高性能**是一个关键词，究竟性能能提升到什么程度呢？我们后面将会有例子演示。

<img src="swoole课程.assets/image-20250708120230211.png" alt="image-20250708120230211" style="zoom: 67%;" />



<img src="swoole课程.assets/image-20250708120251644.png" alt="image-20250708120251644" style="zoom:67%;" />



### 和传统 PHP 概念上的不同

即使你没有做过 Java 或者 .NET（微软开发的**跨平台**、**开源**的开发者平台）、C/C++ 之类的开发，应该也多少听说过它们是需要编译之后生成一个运行文件后才能正式部署上线的。而我们传统的 PHP 貌似并没有这种情况，随时更新一个文件，丢到服务器上就可以运行。这个问题就要说到**静态和动态语言**的问题了。



一般情况下，Java 这类的语言可以归结为静态语言，它们**有固定的变量类型**，必须编译后才能运行，特点是**一次加载会直接将代码加载到内存中**。

典型的就像是我们电脑上的各种应用程序，直接执行一个程序的 .exe 文件，这个程序就在你的电脑上运行起来了。如果你用文本工具打开这种 exe 或者 Java 的 Jar 文件的话，看到的将是一堆乱码似的二进制内容。



**而 PHP 这一类的，则可以归为动态语言，特点是变量不用指定类型，随便一个文件就可以直接运行**。

相信你一定想到了，Python、JavaScript 都是这样的运行方式。即使 JS 的 npm 编译，实际上也只是对代码进行了混淆和格式化，并没有完全编译成一个类似于 jar 包那样的中间代码执行文件。



对于这两种语言编译运行方式来说，静态语言将代码一次加载到内存，效率明显会提升不少，毕竟内存和硬盘的速度差距还是蛮大的。而且，静态语言会一次性将很多初始对象，类模板文件加载，调用的时候不用重新再加载实例化，性能就会有更进一步的上升空间。但是，静态语言通常都是需要编译成一个可执行的中间文件的，如果有代码的更新，则必须重启整个程序。



好吧，动态语言的优缺点很明显就和静态语言是反过来的了。动态语言每一次运行一个脚本，就需要将所有相关的文件全部加载一次，而且如果没别的优化的话（比如 OPcache），所有的相关文件都要从硬盘读取、加载内存、实例化这些步骤中从头走一遍。

可想而知，他的效率和性能是完全无法与静态语言相比的。但是，优点也很明确，随时修改一个文件就可以随时上线，线上业务不用中断。



因此，PHP 通常会是创业公司的首选，因为它方便，更新迭代速度快，对线上业务影响小。

但当公司发展到一定规模之后，却会因为效率性能的问题而容易被 Java、Golang 等语言代替。毕竟，一台服务器能够抗 5 台服务器的压力，成本还是能节省不少的，更主要的是，公司到一定规模之后，对于热更新、规范化上线等等相关的操作，也会让静态语言需要编译或重启服务这类问题成为边缘化的小问题。

**性能效率往往才是中大型公司更重要的考虑**。



上述内容只是基于我自己的理解，不代表完全正确，但是大方向应该是没有问题的。想必说到这里，**你也能猜到 Swoole 是如何来解决效率性能问题的。它就是通过直接将代码加载到内存的方式**，就像 Java 他们一样来启动一个进程，实现 PHP 代码的高性能执行。同时，尽量保持代码还是可以按照传统的方式来写，为我们 PHP 程序员提供了一个高性能的解决方案。



## 教程框架

这一次的系列教程同样是文章和视频形式，我们会分两个大的模块。



第一个模块是以官方文档为基础，简单地学习了解 Swoole 框架中的各项内容，同时尽已所能的解释一些相关的计算机和网络知识。



第二个模块就是一个我使用在生产环境中的 Swoole 框架 Hyperf 的相关配置使用。这个框架与 Laravel 非常类似，很好入手。如果你已经追过之前我们的 Laravel 系列，那么应该不会有太大难度。



同样的，不会有太多的项目实战，毕竟这些东西讲得实在是太多了，随便一搜一大把。



参考资料同样是以 Swoole 官方文档以及 Hyperf 官方文档为基础。官方文档永远是你学习的最重要参考资料，没有之一，包括我写的也只是对官方文档的扩展。



最后，还有说明一下的是，Swoole 是我们国人开发的：**韩天峰** 大佬。文档全中文无压力，同样地，Hyperf 也是我们国人大佬开发的，一样的纯中文文档。感谢各位大佬们！



## **【Swoole系列2.1】先把Swoole跑起来**

> https://www.zyblog.com.cn/article/265

在对 Swoole 有一个初步的印象之后，今天我们就来简单地搭建起 Swoole 环境并且运行起一个简单的 Http 服务。大家不用太有压力，今天的内容还没有太多理论方面的东西，一步步地一起把运行环境先准备好，能看到 Swoole 运行起来的效果就可以了。



### 环境准备

其实安装 Swoole 扩展并不麻烦，和其它的 PHP 扩展一样的安装过程就可以了。不过需要注意的是，Swoole 会和一些扩展产生冲突，比如说 XDebug、phptrace、aop、molten、xhprof、phalcon（协程无法运行在 phalcon 框架中）。



大家一定会担心了，不能使用 XDebug ，我们的调试会很麻烦呀！没关系，**Swoole 也有它自己推荐的调试工具**，有兴趣的小伙伴可以自己查阅下相关资料。



如果你是 Windows 环境，由于操作系统差异的问题，并没有直接的 dll 扩展包可以使用。所以在 Windows 环境下最好是建一个虚拟机，然后设置 PHPStrom 的同步就可以方便地进行开发了。这个配置我就不多说了，百度一搜一大堆。



在这里，我也是通过虚拟机安装的，系统环境为 CentOS8、PHP8.1、Swoole4.8.3、MySQL8.0.27 都是最新的，如果你是 PHP7 以及 Swoole4.4 的话也是没问题的，不会有很大的区别。其它的工具，比如 Nginx、Redis 等，大家可以根据情况安装，我们在后续的课程中也会用到。



扩展安装成功后，在 php.ini 文件中加上扩展信息，然后就可以通过下面的命令行查看 Swoole 的版本信息以及扩展配置情况。

```bash
[root]# php --ri swoole

swoole

Swoole => enabled
Author => Swoole Team <team@swoole.com>
Version => 4.8.10
Built => Jul  8 2022 12:54:21
coroutine => enabled with boost asm context
epoll => enabled
eventfd => enabled
signalfd => enabled
cpu_affinity => enabled
spinlock => enabled
rwlock => enabled
pcre => enabled
zlib => 1.2.11
mutex_timedlock => enabled
pthread_barrier => enabled
futex => enabled
async_redis => enabled

Directive => Local Value => Master Value
swoole.enable_coroutine => On => On
swoole.enable_library => On => On
swoole.enable_preemptive_scheduler => Off => Off
swoole.display_errors => On => On
swoole.use_shortname => On => On
swoole.unixsock_buffer_size => 8388608 => 8388608


[root]# php -v
PHP 7.4.30 (cli) (built: Jul  8 2022 11:13:44) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Xdebug v3.1.6, Copyright (c) 2002-2022, by Derick Rethans


```



### 启动一个服务

相信各位大佬对环境和扩展的安装都是没问题的，如果有问题的话，问我也没用，编译安装这些东西有很多玄学（懵逼）问题存在的，只能是多拿虚拟机练手玩了。环境安装成功后，我们就先来简单地搭建一个 Http 服务吧。



什么？直接起一个 Http 服务？不是要用 Nginx 或者 Apache 吗？忘了上篇文章中我们说过的东西了吧，Swoole 是直接启动服务的，而不是像传统的 PHP 使用 FastCGI 来启动 php-fpm 这种形式的。或者，你也可以理解为直接使用 Swoole 启动的服务就是 php-fpm 使用连接形式启动的那个 9000 端口的服务。



关于 php-fpm 相关的知识，我们之前在 **了解PHP-FPM** https://mp.weixin.qq.com/s/NUpDnfYfbPuWmal4Am3lsg 中有过详细的说明，大家可以回去看看哦。



好了，让我们进入正题，先新一个文件，然后输入下面的代码。

```php
<?php
$http = new Swoole\Http\Server('0.0.0.0', 9501);

$http->on('Request', function ($request, $response) {
    echo "接收到了请求";
    $response->header('Content-Type', 'text/html; charset=utf-8');
    $response->end('<h1>Hello Swoole. #' . rand(1000, 9999) . '</h1>');
});

echo "服务启动";
$http->start();
```

将它放到开启了 Swoole 环境的系统中，然后去执行命令行脚本运行它。

```bash
[root@localhost source]# php 2.1先把Swoole跑起来.php
服务启动
```



一运行起来命令行就停住了，很明显，现在程序已经被挂载起来了。然后我们去浏览器访问页面，如果是本机的话，直接 http://localhost:9501 就可以了，如果是虚拟机，访问虚拟机的 IP 地址并加上端口号。注意，访问失败的话可以检查下有没有关掉防火墙。

访问页面的结果可以看到如下截图。



<img src="swoole课程.assets/image-20250708171354853.png" alt="image-20250708171354853" style="zoom:67%;" />

同时，命令行中也会输出下面的内容。

```
[root@localhost source]# php 2.1先把Swoole跑起来.php
服务启动接收到了请求
```

恭喜你，你的 Swoole 环境没有任何问题哦，现在一个最简单的 Http 服务器已经搭建成功啦！

接下来，我们尝试修改一下文件内容，像上面的输出信息并没有换行，我们把换行符加上吧。

```
// .....
    echo "接收到了请求", PHP_EOL;
// .....

echo "服务启动", PHP_EOL;
```

再次请求页面，你会发现命令行中输出的内容还是没有换行，这是为什么呢？



还记得我们在上篇说过的动态语言与静态语言的问题吧，现在的 Swoole 其实就已经是类似静态语言的运行方式了。它已经将程序挂载起来成为一个独立的进程，其实这个时候就相当于已经编译成了一个类似于 jar 或者 exe 的文件，并且直接运行起来了。因此，我们修改文件是不会对当前进程中的程序产生任何影响的。如果要更新修改之后的内容，就需要重新启动服务。现在就 Ctrl+C 先关闭应用，然后再命令行执行一下吧。你会看到输出就会有换行了。

```
[root@localhost source]# php 2.1先把Swoole跑起来.php
服务启动
接收到了请求
接收到了请求
接收到了请求
接收到了请求
```



### echo 为什么打印在命令行了

相信不少同学又会有一个疑问，为什么 echo 被输出到命令行了？传统的 PHP 开发中，我们的 echo 是直接输出到页面了呀？



这就是另一个 Swoole 与传统开发的不同。在 Swoole 中，我们的服务程序是使用命令行挂起的，我们上面的代码其实在内部是实现了一个 Http 服务功能，而不是通过 php-fpm 去输出给 Nginx 这种服务器的

。转换一个角度来想，php-fpm 也只是把这些输出交给了服务器程序，然后由服务器程序原样给输出到页面上。但是在 Swoole 中，echo 之类的输出流是直接将结果发送到操作系统对应的 stdout 上了。

相对应地，我们的**服务输出则是直接通过 Swoole 代码中的服务回调参数上的 response 对象来进行服务流输出过程（输出到网页上）**。



这个地方的思维是需要大家转变一下的。如果你学习过 Java 开发的话对这里应该不会陌生，在 Swoole 环境下，echo（print、var_dump()等等）之类的传统输出就变成了 Java 中的 System.out.println() 。

同样，在 Java 中，不管什么框架，你要输出页面上的结果值，也是通过类似于一个 Response 对象来实现的。其实，就只是将我们打印的内容输出到了不同的流上，普通的打印流到了 stdout 上，而 Response 对象则是通过 TCP 将结果输出到了响应流上。这一块的原理就很深了，至少得要更深入的学习网络相关的知识，可惜目前的我水平还达不到，所以有兴趣的小伙伴还是自己去查阅相关的资料吧！



### 总结

今天的学习内容不多，最主要的是我们先了解一下最简单的 Http 服务是如何跑起来的，以及在输出方面与传统开发模式的异同。当然，更重要的是，你现在得要把开发环境准备好。否则后续的内容就没法跟着一起实践了哦。



## **【Swoole系列2.2】Http、TCP、UDP服务端**

> https://www.zyblog.com.cn/article/266

其实在上篇文章中，我们就已经运行起来了一个 Http 服务，也简单地说明了一下使用 Swoole 运行起来的服务与普通的 PHP 开发有什么区别。想必你现在会说这没什么大不了的呀，这些我们的传统开发又不是做不到，而且还更方便一些。在基础篇章中，我们还不会看到 Swoole 在性能上的优势，毕竟最基础的一些服务搭建还是要先了解清楚的。因此，今天我们将继续再深入的讲一下 Http 相关的内容以及了解一下 TCP、UDP 服务在 Swoole 中如何运行。



### **http服务**

我们还是看看上次的 Http 服务的代码。

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);
$http->on('Request', function ($request, $response) {
    echo "接收到了请求", PHP_EOL;
    var_dump($request);
    $response->header('Content-Type', 'text/html; charset=utf-8');
    $response->end('<h1>Hello Swoole. #' . rand(1000, 9999) . '</h1>');
});
echo "服务启动", PHP_EOL;
$http->start();
```

首先，我们实例化了一个 Server 对象，在这里我们传递了两个构造函数，一个是监听的 IP 地址，一个是端口号。一般情况下，如果是生产环境内网，我们建议使用内网的本机 IP ，或者直接就是 127.0.0.1 只允许本机访问。但是在我们的测试过程中，需要在虚拟机外访问的话，就需要 0.0.0.0 这样的监听全部 IP 地址。这一块相信不用我过多解释了，Linux 服务的基本知识，数据库、Redis、PHP-FPM 什么的都有这样的配置。



接下来，使用 on() 函数，它是一个监听函数，用于监听指定的事件。在这里，我们监听的就是 Request 事件，监听到的内容将通过回调函数的参数返回，也就是第一个参数 $request ，然后它还会带一个 $response 参数用于返回响应事件。当使用 $response 参数的 end() 方法时，将响应输出指定的内容并结束当前的请求。

上述步骤就完成了一次普通的 Http 请求响应。



#### **查看request参数**

接下来，我们尝试打印一下 $request 参数，看看里面都有什么。

```php
$http->on('Request', function ($request, $response) {
    // .....

    var_dump($request);

    // ....
});
```

在命令行的输出中，你会看到打印的结果，内容非常多

```bash
object(Swoole\Http\Request)#6 (8) {
  ["fd"]=>
  int(1)
  ["header"]=>
  array(7) {
    ["host"]=>
    string(19) "192.168.56.133:9501"
    ["connection"]=>
    string(10) "keep-alive"
    ["upgrade-insecure-requests"]=>
    string(1) "1"
    ["user-agent"]=>
    string(120) "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.55 Safari/537.36"
    ["accept"]=>
    string(135) "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9"
    ["accept-encoding"]=>
    string(13) "gzip, deflate"
    ["accept-language"]=>
    string(23) "zh-CN,zh;q=0.9,en;q=0.8"
  }
  ["server"]=>
  array(10) {
    ["request_method"]=>
    string(3) "GET"
    ["request_uri"]=>
    string(1) "/"
    ["path_info"]=>
    string(1) "/"
    ["request_time"]=>
    int(1639098961)
    ["request_time_float"]=>
    float(1639098961.89757)
    ["server_protocol"]=>
    string(8) "HTTP/1.1"
    ["server_port"]=>
    int(9501)
    ["remote_port"]=>
    int(54527)
    ["remote_addr"]=>
    string(12) "192.168.56.1"
    ["master_time"]=>
    int(1639098961)
  }
  ["cookie"]=>
  NULL
  ["get"]=>
  NULL
  ["files"]=>
  NULL
  ["post"]=>
  NULL
  ["tmpfiles"]=>
  NULL
}
```

发现什么了吗？有 header、server、cookie、get、post 等内容。这些是做什么用的呢？别急，再来测试一下，你可以尝试打印一下 `$_SERVER、$_REQUEST` 等相关的内容。同时为了方便查看，可以给请求链接增加一个 GET 参数，比如说这样请求：http://192.168.56.133:9501/?a=1 。

```
$http->on('Request', function ($request, $response) {
    // .....

    var_dump($request);
    var_dump($_REQUEST);
    var_dump($_SERVER);
    // ....
});
```

在你的命令行中，输出的结果应该是这样的。

```bash
// $request 输出
object(Swoole\Http\Request)#6 (8) {
  ["fd"]=>
  int(1)
  ["header"]=>
  array(8) {
    ["host"]=>
    string(19) "192.168.56.133:9501"
    ["connection"]=>
    string(10) "keep-alive"
    ["cache-control"]=>
    string(9) "max-age=0"
    ["upgrade-insecure-requests"]=>
    string(1) "1"
    ["user-agent"]=>
    string(120) "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.55 Safari/537.36"
    ["accept"]=>
    string(135) "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9"
    ["accept-encoding"]=>
    string(13) "gzip, deflate"
    ["accept-language"]=>
    string(23) "zh-CN,zh;q=0.9,en;q=0.8"
  }
  ["server"]=>
  array(11) {
    ["query_string"]=>
    string(3) "a=1"
    ["request_method"]=>
    string(3) "GET"
    ["request_uri"]=>
    string(1) "/"
    ["path_info"]=>
    string(1) "/"
    ["request_time"]=>
    int(1639099269)
    ["request_time_float"]=>
    float(1639099269.109327)
    ["server_protocol"]=>
    string(8) "HTTP/1.1"
    ["server_port"]=>
    int(9501)
    ["remote_port"]=>
    int(54864)
    ["remote_addr"]=>
    string(12) "192.168.56.1"
    ["master_time"]=>
    int(1639099269)
  }
  ["cookie"]=>
  NULL
  ["get"]=>
  array(1) {
    ["a"]=>
    string(1) "1"
  }
  ["files"]=>
  NULL
  ["post"]=>
  NULL
  ["tmpfiles"]=>
  NULL
}

// $_REQUEST 输出
array(0) {
}

// $_SERVER 输出
array(38) {
  ["LC_ALL"]=>
  string(11) "en_US.UTF-8"
  ["LS_COLORS"]=>
  string(1779) "rs=0:di=38;5;33:ln=38;5;51:mh=00:pi=40;38;5;11:so=38;5;13:do=38;5;5:bd=48;5;232;38;5;11:cd=48;5;232;38;5;3:or=48;5;232;38;5;9:mi=01;05;37;41:su=48;5;196;38;5;15:sg=48;5;11;38;5;16:ca=48;5;196;38;5;226:tw=48;5;10;38;5;16:ow=48;5;10;38;5;21:st=48;5;21;38;5;15:ex=38;5;40:*.tar=38;5;9:*.tgz=38;5;9:*.arc=38;5;9:*.arj=38;5;9:*.taz=38;5;9:*.lha=38;5;9:*.lz4=38;5;9:*.lzh=38;5;9:*.lzma=38;5;9:*.tlz=38;5;9:*.txz=38;5;9:*.tzo=38;5;9:*.t7z=38;5;9:*.zip=38;5;9:*.z=38;5;9:*.dz=38;5;9:*.gz=38;5;9:*.lrz=38;5;9:*.lz=38;5;9:*.lzo=38;5;9:*.xz=38;5;9:*.zst=38;5;9:*.tzst=38;5;9:*.bz2=38;5;9:*.bz=38;5;9:*.tbz=38;5;9:*.tbz2=38;5;9:*.tz=38;5;9:*.deb=38;5;9:*.rpm=38;5;9:*.jar=38;5;9:*.war=38;5;9:*.ear=38;5;9:*.sar=38;5;9:*.rar=38;5;9:*.alz=38;5;9:*.ace=38;5;9:*.zoo=38;5;9:*.cpio=38;5;9:*.7z=38;5;9:*.rz=38;5;9:*.cab=38;5;9:*.wim=38;5;9:*.swm=38;5;9:*.dwm=38;5;9:*.esd=38;5;9:*.jpg=38;5;13:*.jpeg=38;5;13:*.mjpg=38;5;13:*.mjpeg=38;5;13:*.gif=38;5;13:*.bmp=38;5;13:*.pbm=38;5;13:*.pgm=38;5;13:*.ppm=38;5;13:*.tga=38;5;13:*.xbm=38;5;13:*.xpm=38;5;13:*.tif=38;5;13:*.tiff=38;5;13:*.png=38;5;13:*.svg=38;5;13:*.svgz=38;5;13:*.mng=38;5;13:*.pcx=38;5;13:*.mov=38;5;13:*.mpg=38;5;13:*.mpeg=38;5;13:*.m2v=38;5;13:*.mkv=38;5;13:*.webm=38;5;13:*.ogm=38;5;13:*.mp4=38;5;13:*.m4v=38;5;13:*.mp4v=38;5;13:*.vob=38;5;13:*.qt=38;5;13:*.nuv=38;5;13:*.wmv=38;5;13:*.asf=38;5;13:*.rm=38;5;13:*.rmvb=38;5;13:*.flc=38;5;13:*.avi=38;5;13:*.fli=38;5;13:*.flv=38;5;13:*.gl=38;5;13:*.dl=38;5;13:*.xcf=38;5;13:*.xwd=38;5;13:*.yuv=38;5;13:*.cgm=38;5;13:*.emf=38;5;13:*.ogv=38;5;13:*.ogx=38;5;13:*.aac=38;5;45:*.au=38;5;45:*.flac=38;5;45:*.m4a=38;5;45:*.mid=38;5;45:*.midi=38;5;45:*.mka=38;5;45:*.mp3=38;5;45:*.mpc=38;5;45:*.ogg=38;5;45:*.ra=38;5;45:*.wav=38;5;45:*.oga=38;5;45:*.opus=38;5;45:*.spx=38;5;45:*.xspf=38;5;45:"
  ["SSH_CONNECTION"]=>
  string(36) "192.168.56.1 54331 192.168.56.133 22"
  ["LANG"]=>
  string(11) "zh_CN.UTF-8"
  ["HISTCONTROL"]=>
  string(10) "ignoredups"
  ["HOSTNAME"]=>
  string(21) "localhost.localdomain"
  ["XDG_SESSION_ID"]=>
  string(1) "1"
  ["USER"]=>
  string(4) "root"
  ["SELINUX_ROLE_REQUESTED"]=>
  string(0) ""
  ["PWD"]=>
  string(25) "/home/www/2.基础/source"
  ["HOME"]=>
  string(5) "/root"
  ["SSH_CLIENT"]=>
  string(21) "192.168.56.1 54331 22"
  ["SELINUX_LEVEL_REQUESTED"]=>
  string(0) ""
  ["PHP_HOME"]=>
  string(14) "/usr/local/php"
  ["SSH_TTY"]=>
  string(10) "/dev/pts/0"
  ["MAIL"]=>
  string(20) "/var/spool/mail/root"
  ["TERM"]=>
  string(14) "xterm-256color"
  ["SHELL"]=>
  string(9) "/bin/bash"
  ["SELINUX_USE_CURRENT_RANGE"]=>
  string(0) ""
  ["SHLVL"]=>
  string(1) "1"
  ["LANGUAGE"]=>
  string(11) "en_US.UTF-8"
  ["LOGNAME"]=>
  string(4) "root"
  ["DBUS_SESSION_BUS_ADDRESS"]=>
  string(25) "unix:path=/run/user/0/bus"
  ["XDG_RUNTIME_DIR"]=>
  string(11) "/run/user/0"
  ["PATH"]=>
  string(78) "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/php/bin:/root/bin"
  ["HISTSIZE"]=>
  string(4) "1000"
  ["LESSOPEN"]=>
  string(25) "||/usr/bin/lesspipe.sh %s"
  ["OLDPWD"]=>
  string(9) "/home/www"
  ["_"]=>
  string(22) "/usr/local/php/bin/php"
  ["PHP_SELF"]=>
  string(29) "2.2Http、TCP、UDP服务.php"
  ["SCRIPT_NAME"]=>
  string(29) "2.2Http、TCP、UDP服务.php"
  ["SCRIPT_FILENAME"]=>
  string(29) "2.2Http、TCP、UDP服务.php"
  ["PATH_TRANSLATED"]=>
  string(29) "2.2Http、TCP、UDP服务.php"
  ["DOCUMENT_ROOT"]=>
  string(0) ""
  ["REQUEST_TIME_FLOAT"]=>
  float(1639099267.431116)
  ["REQUEST_TIME"]=>
  int(1639099267)
  ["argv"]=>
  array(1) {
    [0]=>
    string(29) "2.2Http、TCP、UDP服务.php"
  }
  ["argc"]=>
  int(1)
}
```

这一下看出问题所在了吗？你会发现 **`$_REQUEST、$_SERVER` 这些之前传统 PHP 中的全局常量都无效了**。虽然 \$_SERVER 也输出了内容，但是请仔细看，这里 \$_SERVER 输出的是我们的命令行信息，不是我们请求过来的信息。除了这两个之外，`$_COOKIE、$_GET、$_POST、$_FILES、$_SESSION` 等等都是这种情况。那么这些内容要获取的话从哪里获取呢？相信大家也都看到了，**直接在 $request 参数中就有我们需要的内容**。

<img src="swoole课程.assets/image-20250708174015849.png" alt="image-20250708174015849" style="zoom:50%;" />



这一块又是一个需要我们转变思维的地方。**为什么这些全局变量不能使用了呢？最主要的原因一是进程隔离问题，二是常驻进程可能会导致的内存泄漏问题**。

> #### 1. **进程隔离**
>
> - **传统 PHP-FPM**：每个请求都是独立的进程，超全局变量会随请求结束自动销毁。
> - **Swoole**：Worker 进程常驻内存，多个请求复用同一个进程，如果使用全局变量会导致**数据污染**（前一个请求的数据泄漏到下一个请求）。
>
> #### 2. **内存泄漏风险**
>
> - 常驻进程中，全局变量不会自动重置，如果不手动清理，会持续占用内存，最终导致内存溢出。
>
> #### 3. **协程安全问题**
>
> - Swoole 的协程是轻量级线程，全局变量在协程切换时可能被覆盖，引发逻辑错误。





关于**进程隔离**问题，我们可以这样来测。

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);

$i = 1;

$http->set([
    'worker_num'=>2,
]);

$http->on('Request', function ($request, $response) {
    global $i;
    $response->end($i++);
});

$http->start();
```

注意我们的 $i 变量是放在监听函数外部的，它是一个针对当前 PHP 文件的全局变量。之后我们设置当前服务的 worker_num ，它的意思是启用两个 Worker 进程，其实也就是我们的工作进程。

启动服务后可以查看当前的进程信息，可以看到有四条 php 进程，其中第一个是主进程，剩下三个是子进程，在子进程中，还有一个管理进程，最后两个就是我们创建的两个 Worker 进程。

```
[root@localhost ~]# ps -ef | grep php
root      1675  1400  0 22:19 pts/0    00:00:00 php 2.2Http、TCP、UDP服务.php
root      1676  1675  0 22:19 pts/0    00:00:00 php 2.2Http、TCP、UDP服务.php
root      1678  1676  0 22:19 pts/0    00:00:00 php 2.2Http、TCP、UDP服务.php
root      1679  1676  0 22:19 pts/0    00:00:00 php 2.2Http、TCP、UDP服务.php
```

接下来，开两个不同的浏览器访问吧，看看 $i 的输出会怎么样。是不是两个浏览器刷新的时候 $i 没有同步地增加呀（因为是进程隔离的，每个worker进程维护自己的内存），体会一下多进程的效果吧。

另一方面运行起来的程序是完全一次性加载到内存当中的，所以这些**全局变量不会自动销毁**，我们的程序毕竟是在一直运行的。因此，如果**稍加不注意，就会出现内存泄露的问题**。



综上所述，global 声明的变量、static 声明的静态变量、静态函数、PHP 原生的超全局变量都有非常大的风险，Swoole 直接干掉了默认的超全局变量，而我们如果要使用全局变量的话也有其它的处理方式。这个我们以后再说。





x-www-form-urlencoded的post请求【不加content请求头时的post默认方式】：

```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "name=John+Doe&email=john.doe@example.com" http://9.135.32.165:9501/
```

输出：

```bash
/data/projects/swoole_test/swoole/2.基础/source/2.2Http、TCP、UDP服务.php:8:
class Swoole\Http\Request#6 (8) {
  public $fd =>
  int(3)
  public $header =>
  array(5) {
    'host' =>
    string(17) "9.135.32.165:9501"
    'user-agent' =>
    string(11) "curl/7.61.1"
    'accept' =>
    string(3) "*/*"
    'content-type' =>
    string(33) "application/x-www-form-urlencoded"
    'content-length' =>
    string(2) "40"
  }
  public $server =>
  array(10) {
    'request_method' =>
    string(4) "POST"
    'request_uri' =>
    string(1) "/"
    'path_info' =>
    string(1) "/"
    'request_time' =>
    int(1690945424)
    'request_time_float' =>
    double(1690945424.8642)
    'server_protocol' =>
    string(8) "HTTP/1.1"
    'server_port' =>
    int(9501)
    'remote_port' =>
    int(45564)
    'remote_addr' =>
    string(12) "9.135.32.165"
    'master_time' =>
    int(1690945424)
  }
  public $cookie =>
  NULL
  public $get =>
  NULL
  public $files =>
  NULL
  public $post =>
  array(2) {
    'name' =>
    string(8) "John Doe"
    'email' =>
    string(20) "john.doe@example.com"
  }
  public $tmpfiles =>
  NULL
}
```

注：

在 Swoole 中，$request->post 属性只能获取 Content-Type 为 application/x-www-form-urlencoded 或 multipart/form-data 的 POST 数据，无法获取 JSON 格式的 POST 数据，即请求头设置为 "Content-Type: application/json"。

json格式的post请求：

```
curl -X POST -H "Content-Type: application/json" -d '{"name": "John Doe", "email": "john.doe@example.com"}' http://9.135.32.165:9501/
```



如果需要获取 JSON 格式的 POST 数据，可以使用 **$request->rawContent() 方法**获取原始的请求内容，然后使用 json_decode() 函数将 JSON 字符串解码为 PHP 数组。

例如：

```php
$http->on('Request', function ($request, $response) {
    // 检查 Content-Type
    if (strpos($request->header['content-type'], 'application/json') !== false) {
        $jsonData = json_decode($request->rawContent(), true);
        
        if (json_last_error() !== JSON_ERROR_NONE) {
            $response->end(json_encode(['error' => 'Invalid JSON']));
            return;
        }
        
        // 使用 $jsonData['name'] 访问数据
        $response->end("Hello, " . $jsonData['name']);
    } else {
        // 处理普通表单数据
        $name = $request->post['name'] ?? 'Guest';
        $response->end("Hello, " . $name);
    }
});
```



```
# JSON 请求
curl -X POST -H "Content-Type: application/json" \
     -d '{"name":"John","age":30}' \
     http://localhost:9501/

# 表单请求（对比）
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" \
     -d 'name=John&age=30' \
     http://localhost:9501/
```





#### **$server服务器对象**

```
$server = new Swoole\Http\Server('0.0.0.0', 9501);
$server->on('Request', function ($request, $response) use ($server) {
  echo json_encode($server) . "\n";
});
```



```
{
    "setting": {
        "worker_num": 8,
        "task_worker_num": 0,
        "output_buffer_size": 4294967295,
        "max_connection": 100000,
        "open_http_protocol": true,
        "open_mqtt_protocol": false,
        "open_eof_check": false,
        "open_length_check": false
    },
    "connections": {},
    "host": "0.0.0.0",
    "port": 9501,
    "type": 1,
    "mode": 2,
    "ports": [
        {
            "host": "0.0.0.0",
            "port": 9501,
            "type": 1,
            "sock": 3,
            "setting": {
                "worker_num": 8
            },
            "connections": {}
        }
    ],
    "master_pid": 1013833,
    "manager_pid": 1013834,
    "worker_id": 5,
    "taskworker": false,
    "worker_pid": 1013844,
    "stats_timer": null,
    "admin_server": null
}
```



#### **php全局变量不可用**

$_REQUEST、$_SERVER 这些之前传统 PHP 中的全局常量都无效了，最主要的原因一是进程隔离问题，二是常驻进程可能会导致的内存泄漏问题。



#### **测试task异步任务**

```php
//测试task异步任务
$server = new Swoole\Http\Server('0.0.0.0', 9501); 
$server->set(
   [
      'task_worker_num' => 1,
       'worker_num' => 1
       ]
);
$server->on('Request', function ($request, $response) use ($server) {
   $task_id = $server->task('task data');
   echo "Task $task_id has been dispatched\n";
   $response->end('task ok');
   // 处理请求
});

$server->on('Task', function ($server, $task_id, $from_id, $data) {
   echo "Task $task_id is processing\n";
   // 处理任务
   $server->finish('task result');
});

$server->on('Finish', function ($server, $task_id, $data) {
   echo "Task $task_id has been finished\n";
   // 处理任务结果
});
$server->start();
```



```
Task 0 has been dispatched
Task 0 is processing
Task 0 has been finished
```



#### **请求分配worker的方式**

dispatch_mode

默认 2 根据文件描述符分配 



### **tcp服务**

对于 Http 服务我们又进行了一次复习，并且通过这个 Http 服务我们还看到了多进程程序的特点以及在开发时需要转变的一个重大的思维。

当然，这些东西我们在后面会经常接触到。接下来，大家一起继续学习了解一下使用 Swoole 来搭建一个 TCP 服务端。



只要是学习过一点网络相关知识的同学肯定都知道，我们的 Http 服务本身就是建立在 TCP 的基础之上的。因此，其实要建立 TCP 服务的基本步骤和 Http 服务是没啥差别的。最主要的就是监听的内容不同。



```php
//创建Server对象，监听 9501 端口
 $server = new Swoole\Server('0.0.0.0', 9501);

 //监听连接进入事件
 $server->on('Connect', function ($server, $fd) {
    echo "Client: Connect.\n";
 });

 //监听数据接收事件
 $server->on('Receive', function ($server, $fd, $reactor_id, $data) {
    $server->send($fd, "Server TCP: {$data}");
 });

 //监听连接关闭事件
 $server->on('Close', function ($server, $fd) {
    echo "Client: Close.\n";
 });

 //启动服务器
 $server->start();
```

相比原生的 PHP 的 socket 函数来说，Swoole 是不是清晰方便很多。我们启动服务之后，使用 telnet 命令行就可以对这个 TCP 服务器进行测试了。

```bash
# telnet 9.135.32.165 9501
Trying 9.135.32.165...
Connected to 9.135.32.165.
Escape character is '^]'.
hello
Server TCP: hello
world
Server TCP: world
```

Swoole 中还有对应的 TCP 和 UDP 客户端，这个我们后面再说。



### **udp服务**

UDP 和 TCP 的区别相信不用我多说了吧，这玩意不建立可靠连接的，但是速度快，所以现在的各种视频直播之类的应用全是建立在 UDP 之上的。可以说是支撑当前短视频和直播时代的基石了。

```php
$server = new Swoole\Server('0.0.0.0', 9501, SWOOLE_PROCESS, SWOOLE_SOCK_UDP);

//监听数据接收事件
$server->on('Packet', function ($server, $data, $clientInfo) {
    var_dump($clientInfo);
    $server->sendto($clientInfo['address'], $clientInfo['port'], "Server UDP：{$data}");
});

//启动服务器
$server->start();
```

写法也基本都是类似的，不同的还是监听的内容不同。由于它不建立连接，所以我们只需要监听接收到的数据包信息就可以了。



```bash
nc -vu 9.135.32.165 9501
Ncat: Version 7.70 ( https://nmap.org/ncat )
Ncat: Connected to 9.135.32.165:9501.
hello world
Server UDP：hello world
```

补充：-u使用udp协议。。

对于命令行的测试来说，我们也不能使用 telnet 了，在这里，我使用的也是 Linux 环境中比较常见的 nc 命令来进行测试的。



客户端例子：

```bash
//server.php
<?php
$server = new Swoole\Server('0.0.0.0', 9502, SWOOLE_PROCESS, SWOOLE_SOCK_UDP);

// 监听 Packet 事件（UDP 数据接收）
$server->on('Packet', function ($serv, $data, $clientInfo) {
    // 打印客户端信息
    echo "Received data from {$clientInfo['address']}:{$clientInfo['port']}\n";
    echo "Data: " . $data . "\n";

    echo json_encode(compact('serv', 'data', 'clientInfo'), JSON_PRETTY_PRINT);


    // 模拟处理逻辑（这里简单反转字符串）
    $processedData = strrev($data);

    // 返回响应给客户端
    $serv->sendto($clientInfo['address'], $clientInfo['port'], "Server Response: " . $processedData);
});

// 启动服务器
$server->start();


//client.php
<?php
$client = new Swoole\Client(SWOOLE_SOCK_UDP);

// 发送数据到服务器
$data = "Hello Swoole UDP!";
if (!$client->sendto('127.0.0.1', 9502, $data)) {
    die("Send failed.\n");
}

// 接收服务器响应
$response = $client->recv();
echo "Server Response: " . $response . "\n";
```



==>打印相关内容

```
{
    "serv": {
        "setting": {
            "worker_num": 32,
            "task_worker_num": 0,
            "output_buffer_size": 4294967295,
            "max_connection": 100000
        },
        "connections": {},
        "host": "0.0.0.0",
        "port": 9502,
        "type": 2,
        "mode": 2,
        "ports": [
            {
                "host": "0.0.0.0",
                "port": 9502,
                "type": 2,
                "sock": 3,
                "setting": null,
                "connections": {}
            }
        ],
        "master_pid": 566828,
        "manager_pid": 566829,
        "worker_id": 31,
        "taskworker": false,
        "worker_pid": 566869,
        "stats_timer": null,
        "admin_server": null
    },
    "data": "Hello Swoole UDP!",
    "clientInfo": {
        "server_socket": 3,
        "dispatch_time": 1750995674.142039,
        "server_port": 9502,
        "address": "127.0.0.1",
        "port": 60957
    }
}
```



### 总结

今天我们就是简单地先看一下在整个 Swoole 中，Http、TCP、UDP 服务是如何跑起来的，另外也尝试了一下多进程对于全局变量的影响。

其实要学习 Swoole ，就不可避免地要学习到很多计算机相关的基础知识，如果你还没有这方面的准备的话，可以先看看操作系统、计算机组成原理相关的内容。毕竟我也不会讲得太详细，也达不到来讲这些基础理论知识的水平。所以，有相关的内容我也只能尽已所能地去稍带地提出，毕竟我自己也还是在不断学习这些基础的过程之中的。





## **【Swoole系列2.3】TCP、UDP客户端**

> https://www.zyblog.com.cn/article/769

上一节，我们学习了如何搭起简单的 Http、TCP 以及 UDP 服务。是不是发现在 Swoole 中搭建这三种服务非常地简单方便。对于 Http 客户端来说，我们可以直接使用浏览器来进行测试，或者普通的 Curl、Guzzle 也可以方便地从代码中进行 Http 的测试。因此，我们也就不会过多地说 Http 客户端的问题。等到进阶相关的文章时，我们会再看看在 协程 中的 Http 客户端如何使用。

今天的内容主要是针对于 TCP 和 UDP 的客户端。上篇文章中，我们使用的是命令行的 telnet 和 nc 工具来测试这两种服务的运行情况，今天我们直接通过 Swoole 的客户端对象来进行测试。





### **tcp客户端**

在 Swoole 中，有同步阻塞客户端和协程客户端两种类型的客户端，今天我们就行来简单地学习一下同步阻塞客户端。

什么叫 同步阻塞 ？其实就是我们正常的那种按照前后关系顺序执行的代码，也就是我们在传统开发中写的那种代码。代码是按照顺序从上往下执行的，前面的代码没有执行完，后面的代码也不会运行。如果中间遇到函数，则会通过类似栈的处理方式进入函数中进行处理。从本质上来说，其实 面向对象 这种编程方式是有部分跳出这种线性执行代码的模式的，但是，它还是同步执行的。

而多线程、协程这种东西，其实就是脱离了同步阻塞问题的，关于进程、线程、协程相关的问题，我们后面有专门的文章来说明。今天大家就只需要大概了解一下就可以了。或者，你把我们今天实现的代码就当做是一个 Swoole 中自带的 Guzzle TCP/UDP 版本客户端就好了。



#### **同步阻塞客户端**

同步阻塞客户端是一种基于阻塞 I/O 的客户端实现方式，它的主要特点是在进行网络通信时会阻塞当前线程，直到网络通信完成或者发生错误。同步阻塞客户端的实现方式非常简单，可以使用 Swoole\Client 类来实现。

```php
$client = new Swoole\Client(SWOOLE_SOCK_TCP);
if (!$client->connect('127.0.0.1', 9501, -1)) {
   exit("connect failed. Error: {$client->errCode}\n");
}

$client->send("hello world\n");
echo $client->recv(); //
$client->close();
```

实现一个 TCP 客户端非常简单，实例化一个 Swoole\Client 对象。它的构造参数可以传递 SWOOLE_SOCK_TCP 或者 SWOOLE_SOCK_UDP 等内容。从名字就可以看出，一个是 TCP 客户端，一个是 UDP 客户端。

接着，我们通过 connect() 方法进行连接，连接的就是本机的 TCP 端口。这里我们直接将上篇文章中的 TCP 服务启动起来就可以了。

接着 send() 方法用于发送数据到 服务端 ，recv() 方法用于接收服务端返回的信息，最后的 close() 用于关闭客户端句柄。

```
# php 2.3Http、TCP、UDP服务客户端.php
Server TCP：hello world
```

这个打印出来的内容，就是我们在服务端输出的数据。相信这一块的内容大家应该是没有什么难度的。我们直接再看看 UDP 客户端。



#### **协程客户端**

协程客户端是一种基于协程的客户端实现方式，它的主要特点是在进行网络通信时不会阻塞当前线程，而是通过协程调度器来实现异步 I/O。协程客户端的实现方式相对复杂，需要使用 **Swoole\Coroutine\Client** 类来实现。例如：



```
//TCP协程客户端
go(function () {
   $client = new Swoole\Coroutine\Client(SWOOLE_SOCK_TCP);

   if (!$client->connect('127.0.0.1', 9501, 0.5)) {
       echo "Error: {$client->errMsg}\n";
       return;
   }

   $client->send("Hello World\n");
   echo $client->recv();
   $client->close();
});
```

ps:Swoole\Coroutine\Client 必须在协程中调用，否则报错 Fatal error: Uncaught Swoole\Error: API must be called in the coroutine



### **udp客户端**

对于 UDP 来说，其实它的实现代码和上面的 TCP 差不多，而且更加简洁。为什么呢？我们都知道，TCP 是要建立稳定连接的，有三次握手四次挥手的过程，这也是 TCP 的基础知识。而 UDP 不需要，它不用建立稳定的连接，所以，我们可以在 UDP 中省略掉 connect() 的步骤。

```php
$client = new Swoole\Client(SWOOLE_SOCK_UDP);
$client->sendto('127.0.0.1', 9501, "hello world\n");
echo $client->recv();
$lient->close();
```

这里使用的是 sendto() 方法，它的作用是向任意的地址和端口发送 UDP 数据包.

当然，你在这里使用 connect() 并且通过 send() 发送 UDP 数据也是没问题的，大家可以自己尝试一下。

```bash
[root@localhost source]# php 2.3Http、TCP、UDP服务客户端.php
Server UDP：hello world
```



### 其他方法

最后，我们再来看几个客户端对象的其它方法。

```php
var_dump($client->isConnected()); // bool(true)
// var_dump($client->getSocket());
var_dump($client->getsockname());
//array(2) {
//    ["port"]=>
//  int(47998)
//  ["host"]=>
//  string(9) "127.0.0.1"
//}
```

第一个 isConnected() 用于返回客户端是否连接的布尔值。前提当然是要调用了 connect() 并成功建立连接之后才会返回 true 。

getSocket() 用于返回一个 socket 扩展的句柄资源符，目前我们的系统环境中暂时没有安装 socket 扩展，所以这个函数还用不了。

getsockname() 用于获取客户端的 socket 在本地的 host 和 port 端口。可以看到注释中我们程序自动在本地开了 47998 这个端口用于和服务端的 TCP 进行通信使用。



另外在 UDP 中，我们可以使用 getpeername() 获得对端 socket 的 IP 地址和端口。

```php
var_dump($client->getpeername());
//array(2) {
//    ["port"]=>
//  int(0)
//  ["host"]=>
//  string(7) "0.0.0.0"
//}
```

这个方法仅支持 UDP 连接，因为 UDP 协议通信客户端向一台服务器发送数据包后，可能并非由此服务器向客户端发送响应。可以使用 getpeername() 方法获取实际响应的服务器 IP 和 PORT。当然，我们目前在本机没有这种情况，直接返回的全是零。



### 总结

除了上述内容之外，还有证书相关的方法函数，另外也有建立长连接的常量参数，这些内容大家可以自己在下面的官方文档链接中找到，在这里我就不做过多的演示了。毕竟只是带大家入个门，直接搬文档可不是我的风格。



好了，最重要的三个网络服务及相关的客户端的入门展示我们就学习完成了，下一篇文章我们将再学习一个现在比较流行的服务应用，那就是 WebSocket 的使用。





## **【Swoole系列2.4】WebSocket服务**

关于 WebSocket 的好处最主要的是，它建立起来的是一个持久的长链接，不需要像轮询一样不停地发送 Http 请求，能够非常有效地节省服务器资源。



### **websocket vs http**

[WebSocket 教程 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2017/05/websocket.html)

![img](data:image/png;base64,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)

解决痛点：HTTP通信只能由客户端发起，如果服务器有连续的状态变化，客户端要获知就非常麻烦。我们只能使用["轮询"](https://www.pubnub.com/blog/2014-12-01-http-long-polling/)。



websocket的特点：

它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话。

其他特点包括：

（1）建立在 TCP 协议之上，服务器端的实现比较容易。

（2）与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。

（3）数据格式比较轻量，性能开销小，通信高效。

（4）可以发送文本，也可以发送二进制数据。

（5）没有同源限制，客户端可以与任意服务器通信。

（6）协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。



```
ws://example.com:80/some/path
```



### **ws服务端**

整个 WebSocket 最核心的内容就是监听三个事件：Open、Message、Close。



```
//创建WebSocket Server对象，监听0.0.0.0:9501端口
$ws = new Swoole\WebSocket\Server('0.0.0.0', 9501);

//监听WebSocket连接打开(建立)事件
$ws->on('Open', function ($ws, $request) {
    while(1){
        $time = date("Y-m-d H:i:s");
        $ws->push($request->fd, "hello, welcome {$time}\n");
        Swoole\Coroutine::sleep(10);
    }
});

//监听WebSocket消息事件
$ws->on('Message', function ($ws, $frame) {
    echo "Message: {$frame->data}\n";
    $ws->push($frame->fd, "server: {$frame->data}");
});

//监听WebSocket连接关闭事件
$ws->on('Close', function ($ws, $fd) {
    echo "client-{$fd} is closed\n";
});


$ws->start();
```





当我们的客户端连接到服务时，就会触发 Open 监听，其中在 $request 中会返回连接的 fd 信息，这是一个句柄，或者说是标识我们的客户端的一个标志。然后我们在 Open 监听中每隔十秒去发送一条消息，假装是一个后台的通知信息。

Coroutine::sleep() 这个 Swoole 提供的休眠函数，它会只针对当前的协程进行休眠。



### **wb客户端（http）**



```
<?php
use Swoole\Coroutine\Http\Client;

$websocketUrl = 'ws://9.135.32.165:9501';

go(function () use ($websocketUrl) {
    $client = new Client(parse_url($websocketUrl)['host'], parse_url($websocketUrl)['port']);

    if ($client->upgrade($websocketUrl)) {
        echo "Connected to WebSocket server.\n";

        // 发送消息到服务器
        $message = 'Hello, server!';
        $client->push($message);

        // 接收服务器发送的消息
        while (true) {
            $frame = $client->recv();
            if ($frame === false) {
                break;
            }
            echo 'Retrieved data from server: ' . $frame->data . "\n";
        }

        // 关闭连接
        $client->close();
    } else {
        echo "Failed to connect to WebSocket server.\n";
    }
});
?>
```





$client->upgrade($websocketUrl) 方法是用于建立 WebSocket 连接的。



### **wb客户端（前端js代码）**



```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>

<body>

<input type="text" id="txt"/><br/>
<button onclick="send()">发送</button>


<p id="response">

</p>
<script type="text/javascript">
  var wsServer = 'ws://9.135.32.165:9501/ws';
  var websocket = new WebSocket(wsServer);
  websocket.onopen = function (evt) {
    console.log("Connected to WebSocket server.");
  };

  websocket.onclose = function (evt) {
    console.log("Disconnected");
  };

  websocket.onmessage = function (evt) {
    console.log('Retrieved data from server: ' + evt.data);
    document.getElementById("response").innerHTML = document.getElementById("response").innerHTML + 'Retrieved data from server: ' + evt.data + '<br/>';
  };

  websocket.onerror = function (evt, e) {
    console.log('Error occured: ' + evt.data);
  };


  function send(){
      websocket.send(document.getElementById('txt').value);
  }

</script>
</body>
</html>
```





在 JS 代码中，我们直接使用的就是原生的 WebSocket 对象。它同样是需要监听三个主要的方法，和服务端也是一一对应的，分别就是 onopen()、onclose() 和 onmessage() 方法。另外还有一个 send() 方法是上面的按扭调用的，当点击按扭后，将文本输入框中的内容通过 WebSockent 的 send() 方法发送给服务端。



ps:[vscode通过服务器打开html文件](https://blog.51cto.com/u_14524868/3171492)

使用插件:Live Server

## **【Swoole系列2.5】异步任务**

### **使用异步任务**

Task 进程:

​                ● Task 任务是需要监听两个事件的，Task 事件是用于处理任务的，可以根据传递过来的 $data 内容进行处理

​                ● Finish 事件是监听任务结束，当执行的任务结束后，就会调用这个事件回调，可以进行后续的处理。

​                ● $http->task("发送邮件"); 投递task任务



```
<?php

$http = new Swoole\Http\Server('0.0.0.0', 9501);

$http->set([
   'worker_num' => 1,
   'task_worker_num'=>1,
]);

$http->on('Request', function ($request, $response) use($http) {
   echo "接收到了请求", PHP_EOL;
   $response->header('Content-Type', 'text/html; charset=utf-8');

   $http->task("发送邮件");
   $http->task("发送广播");
   $http->task("执行队列");

   // $http->task("发送邮件2");
   // $http->task("发送广播2");
   // $http->task("执行队列2");

   $response->end('<h1>Hello Swoole. #' . rand(1000, 9999) . '</h1>');

});

//处理异步任务(此回调函数在task进程中执行)
$http->on('Task', function ($serv, $task_id, $reactor_id, $data) {
   $sec = rand(1,5);
   echo "New AsyncTask[id={$task_id}] sleep sec: {$sec}".PHP_EOL;
   sleep($sec);
   // sleep(rand(1,5));
   //返回任务执行的结果
   $serv->finish("{$data} -> OK");
});

//处理异步任务的结果(此回调函数在worker进程中执行)
$http->on('Finish', function ($serv, $task_id, $data) {
   echo "AsyncTask[{$task_id}] Finish: {$data}".PHP_EOL;
});

echo "服务启动", PHP_EOL;
$http->start();
```





![img](data:image/png;base64,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)



### **多于指定任务数执行**



6个 task() 任务，但 task_worker_num 还是 4 个 ==> 排队



```
$http->on('Request', function ($request, $response) use($http) {
    echo "接收到了请求", PHP_EOL;
    $response->header('Content-Type', 'text/html; charset=utf-8');

    $http->task("发送邮件");
    $http->task("发送广播");
    $http->task("执行队列");

    $http->task("发送邮件2");
    $http->task("发送广播2");
    $http->task("执行队列2");

    $response->end('<h1>Hello Swoole. #' . rand(1000, 9999) . '</h1>');

});
```





![img](data:image/png;base64,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)

首先先开始了 4 个任务进程，然后执行完一个才又创建了一个新的 Task 。Task 2 和 1 都是 1 秒的阻塞，它们完成之后又创建了 Task 4 和 5 。仔细看截图，5 还比 4 先创建出来呢。这就是多进程（线程）编程的最大特点，执行顺序完全不是你能确定的，但它们真的是在并行执行的！



## **【Swoole系列2.6】Redis服务器**

[【Swoole系列2.6】Redis服务器](http://www.zyblog.com.cn/article/269)

Redis 服务端是个什么东西呢？其实它是一个基于 Redis 协议的服务器程序，可以让我们使用 Redis 的客户端来连接这个服务。

就是自己写了个redis server 模拟redis的功能。

不是平时连接redis用的client。



### **服务器**



```
<?php
use Swoole\Redis\Server;

define('DB_FILE', __DIR__ . '/db');

$server = new Server("0.0.0.0", 9501, SWOOLE_BASE);

if (is_file(DB_FILE)) {
    $server->data = unserialize(file_get_contents(DB_FILE));
} else {
    $server->data = array();
}

$server->setHandler('GET', function ($fd, $data) use ($server) {
    if (count($data) == 0) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'GET' command"));
    }

    $key = $data[0];
    if (empty($server->data[$key])) {
        return $server->send($fd, Server::format(Server::NIL));
    } else {
        return $server->send($fd, Server::format(Server::STRING, $server->data[$key]));
    }
});

$server->setHandler('SET', function ($fd, $data) use ($server) {
    if (count($data) < 2) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'SET' command"));
    }

    $key = $data[0];
    $server->data[$key] = $data[1];
    return $server->send($fd, Server::format(Server::STATUS, "OK"));
});

$server->setHandler('sAdd', function ($fd, $data) use ($server) {
    if (count($data) < 2) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'sAdd' command"));
    }

    $key = $data[0];
    if (!isset($server->data[$key])) {
        $array[$key] = array();
    }

    $count = 0;
    for ($i = 1; $i < count($data); $i++) {
        $value = $data[$i];
        if (!isset($server->data[$key][$value])) {
            $server->data[$key][$value] = 1;
            $count++;
        }
    }

    return $server->send($fd, Server::format(Server::INT, $count));
});

$server->setHandler('sMembers', function ($fd, $data) use ($server) {
    if (count($data) < 1) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'sMembers' command"));
    }
    $key = $data[0];
    if (!isset($server->data[$key])) {
        return $server->send($fd, Server::format(Server::NIL));
    }
    return $server->send($fd, Server::format(Server::SET, array_keys($server->data[$key])));
});

$server->setHandler('hSet', function ($fd, $data) use ($server) {
    if (count($data) < 3) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'hSet' command"));
    }

    $key = $data[0];
    if (!isset($server->data[$key])) {
        $array[$key] = array();
    }
    $field = $data[1];
    $value = $data[2];
    $count = !isset($server->data[$key][$field]) ? 1 : 0;
    $server->data[$key][$field] = $value;
    return $server->send($fd, Server::format(Server::INT, $count));
});

$server->setHandler('hGetAll', function ($fd, $data) use ($server) {
    if (count($data) < 1) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'hGetAll' command"));
    }
    $key = $data[0];
    if (!isset($server->data[$key])) {
        return $server->send($fd, Server::format(Server::NIL));
    }
    return $server->send($fd, Server::format(Server::MAP, $server->data[$key]));
});

$server->on('WorkerStart', function ($server) {
    $server->tick(10000, function () use ($server) {
        file_put_contents(DB_FILE, serialize($server->data));
    });
});

$server->setHandler('KEYS', function ($fd, $data) use ($server) {
   if (count($data) == 0) {
       return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'GET' command"));
   }

   if(!$server->data){
       return $server->send($fd, Server::format(Server::NIL));
   }

   $key = $data[0];
   if($key == "*"){
       return $server->send($fd, Server::format(Server::SET, array_keys($server->data)));
   }else{
       $dataKeys = array_keys($server->data);
       $key = str_replace(["*", "?"], [".*", ".?"], $key);
       $values = array_filter($dataKeys, function($value) use ($key){
           return preg_match('/'.$key.'/i', $value, $mchs);
       });
       if($values){
           return $server->send($fd, Server::format(Server::SET, $values));
       }
   }
   return $server->send($fd, Server::format(Server::NIL));
});



$server->start();
```





查看上面的代码，我们会发现这也是一个 Server 对象。然后主要是**使用 setHandler() 方法**来监听 Reids 命令，在这里我们看到了熟悉的 get、set 等命令的定义。然后我们指定了 $server->data ，可以将它看成是一个数据源，直接使用的就是一个文件，直接在当前测试环境目录下创建一个叫做 db 的空文件就可以了。



在 setHandler() 方法中，我们使用 send() 方法来返回响应的命令信息，并通过 format() 方法格式化返回的响应数据。



最后，通过一个监听一个 WorkerStart 事件，设置了一个每隔一秒将数据写入到 db 文件的操作。



### **客户端测试**

现在，运行起来这个 Swoole 写的 Redis 服务端文件吧，然后使用你的 redis-cli 来连接并测试它。



```
# redis-cli -h 9.135.32.165 -p 9501
9.135.32.165:9501> set name ccx
OK
9.135.32.165:9501> get name
"ccx"
```





神奇吗？惊喜吗？我们竟然实现了一个 Redis 服务器，再看看 db 文件中是什么内容。

obj序列化的结果：



```
a:1:{s:4:"name";s:3:"ccx";}
```



## **【Swoole系列3.1】进程、线程、协程，面试你被问了吗？**

进程相关的问题，在计算机专业中一般是操作系统中来进行讲解的。

不过之前包括在数据结构相关的课程中我也说过，我并不是计算机专业的，所以说，这个问题对于之前的我来说还真是挺懵圈的。

通过这些年来学习了一些操作系统相关的知识，有了一些了解，并且确实真的是仅限于了解的水平，也没有特别深入的学习和实验，所以今天的内容中的各种纰漏也请大家多多指正。



### 进程

进程，最简单的理解就是我们运行起来的软件。它是一段程序的执行过程。比如说，你在 Windows 上打开了 QQ ，然后去任务管理器就可以看到一个 QQ.exe 的程序正在执行。这个东西就叫做**进程**。



进程有开始有结束，我们关了 QQ ，自然 QQ 对应的进程也就结束了。而对于服务器操作系统来说，我们的服务应用程序一般是不会进行重启的，但它确实也有结束的可能。比如说升级服务器，修改配置文件等等。



多开几个 QQ ，或者就像当年我们玩游戏时多开游戏，其实就是**多进程的应用**。

另外，进程还有**子进程**这一概念，它是调用操作系统的 fork() 函数创建的，一般这样的子进程是父进程的一个复制品。

**Swoole 的核心高并发实现其实正是基于这种多子进程的形式的**，后面我们会详说。



对于 CPU 来说，它会使用轮询或者其它的调度算法不停地以时间片的方式执行不同的进程，所以不同的进程会互相切换。假如说你听着歌，看着网页，写着代码，看似同时在进行，其实 CPU 在底层是在不断地切换执行这些进程的，只是速度太快你感觉不到而已。操作系统则主要是负责这些进程的管理、调度、切换。



当你使用 PHP 命令行执行某个 php 文件时，其实也和打开 QQ 的操作一样启动了一个 PHP 进程，或者说就是运行起了一个 PHP 程序。这时在 ps -ef 或者 top 命令中，就可以看到你运行起来的程序信息。

进程会在执行过程中会占用 CPU 和 内存 等资源，根据程序的操作不同，也可能会产生网络IO、硬盘IO，这一切，都是通过操作系统来进行调度的。**传说中的优化也是针对这些资源的利用率进行优化**。



传统的 PHP-FPM(FastCGI Process Manager)是多进程形式的，它本身的意思就是 FastCGI (Common Gateway Interface，通用网关接口）进程管理器。

Nginx 其实也是多进程的，当同一个进程中的程序如果有耗时操作，会产生阻塞，Nginx 就会进行等待，因此，当请求量非常大的时候，PHP-FPM 就会非常累。同时，**PHP 是我们之前说过的动态语言，加载过程是完全的整体加载，这样也会带来性能的损耗**。传统 PHP 抗不了高并发，或者说高并发性能很差的原因也就在这里了。



虽说有问题，但是进程是一个程序的开始，如果你做过 C 或 Java 的开始，一定知道有个 main() 方法很重要。

其实，在 PHP 中，一样存在这个 main() 方法，只是是在底层的 C 中，我们看不到而已。没有进程，程序就不存在，这是基础，是后面线程和协程的根本。



多进程操作其实就是充分利用多核 CPU 的优势，一个 CPU 去对应一个进程的操作，从而提高进程的执行效率。

比如我们在 Swoole 中可以设置 worker_num ，它的建议设置值就是根据 **CPU 的核数**。同样地，Nginx 配置中的 worker_processes 也是建议配置成 CPU 的核数。



### 线程

线程，是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。



这么说吧，进程是个包工头，线程是它手下的工人。如果只有一个包工头，又要铺地板，又要刷墙，只能一步一步地顺序地来。如果其中一个出现问题了，后面的工作就要等着前面的工作完成，这种情况的操作过程就叫做 串行 操作。这个时候，包工头雇佣了两个工人，让他们同时干活，一个铺地板，一个刷墙，是不是一我们的工程速度一下就提高了很多，这种情况就叫做 并行 操作。



再来概括一下，线程就是进程的小弟，并且可以在一个进程中同步地执行一些并行操作，一些复杂的 IO 操作、容易被阻塞的业务都可以放到线程中进行。想想也知道，我们的程序执行效率将会有一个质的飞跃。但是，线程也不是相像中的那么美好，**线程之间会竞争资源**，特别是对于同一个资源。比如说我们要操作一个文件，两个线程并行同时访问修改，那就很容易出问题。这个问题其实就是非常类似于我们 MySQL 中的事务问题。MySQL 是怎么解决的？用锁，用事务隔离级别。同样地，在应用程序中，我们也可以加锁。就像 Java 中的 synchronized 同步锁。当加了同步锁之后，访问这个锁相关内容的线程将被阻塞，前面的释放了后面的才能进来执行。



**线程之间是互相独立的，但是它们可以共享进程的资源，而多个进程之间是没办法做到资源共享的**，只能借助外部力量或进程间通信。其实这个也很好理解，线程是进程的小弟嘛，同一个进程里面的线程都可以获得大哥的信息。同时，线程也是会消耗系统资源的，线程开得太多就和进程开得太多一样，让 CPU 、内存 等资源非常嗨。



线程的英文名是 Thread ，在 Java 中，直接就可以实例化这样一个名字的对象来执行线程操作。

但在 PHP 中，估计你就真没见过了。其实，在 PHP 中，本身也有一个 pthreads 扩展，在这个扩展里面就有一个 Thread 类可以使用。但是，pthreads 扩展已经停止维护了。在 PHP7.2 之后有一个 **parallel 扩展**可以替代 pthreads 扩展。关于这个扩展我们以后有机会再详细说吧，毕竟没啥现成的资料，而且文档还全是英文的。



在 Swoole 中，其实我们是创建不了线程的。Swoole 中默认会有一系列的线程能力，但是是用在底层处理网络 IO 等相关操作。包括我们之前讲过的异步任务，其实它也只是子进程和异步IO进程的应用，不是线程。在 Swoole 中，我们使用的更多的其实是下面要讲到的 协程 。



### 协程

协程，从官方意义上来说，不是进程也不是线程，它更类似于一个不带返回值的函数调用。但是，如果通俗一点说，你把它想象成线程也问题不大，只不过这个**线程是用户态的**。什么意思呢？进程和线程是系统来控制的，包括切换调度，而协程是可以自己操作的，不需要操作系统参与，创建销毁的成本非常低。但同时，它又不像多线程可以充分利用 CPU 。**在 Swoole 中，我们在协程中需要借助于多进程模型来利用到多核 CPU 的优势**。



话说回来，协程就是个更轻量级的线程，或者再进一步说协程是线程的小弟。线程进程开得越多，资源占用得越多，操作系统为之带来的切换消耗也越多。而协程则是运行在线程之上，当一个协程运行完成之后，主动让出，让另一个协程运行在当前线程之上，减少了线程的切换。同理，我们也就不需要再开那么多的线程了。注意，这里的协程是 并发 ，不是上面线程中说的 并行 。并行是真的可以在同一时间一起运行，而并发则是一起启动，但还是依靠 CPU 快速切换来执行，本质上还是串行，只是看起来像是同时在执行。对于 **并行 和 并发** 不太了解的小伙伴可以再详细查阅下相关资料哦。



打个比方，原来我们需要开 1 个进程，100 个线程，现在使用协程，我们可以开 1 个进程，100 个线程，然后每个线程之上再运行 100 个协程。现在，你相当于有 10000 个处理事务的小弟在工作，过不过瘾。最主要的是，机器配置还不用变，并发同步处理从 100 变成了 10000 。



当然，并不是说协程就无敌了。**协程只有在等待 IO 的过程中才能重复利用线程**。一条线程在等待耗时的 IO 操作时会阻塞，其实操作系统这个时候认识的只有线程，当它阻塞了，这个线程里面的协程也是在阻塞态的。如果没有异步调用的支持，其实协程也发挥不了什么作用。**异步调用**又是个啥？就是 **异步IO** ，epoll 有了解过不？或者说，JavaScript 中的事件循环。



既然是循环，其实想必你也清楚了，我们来总结一下。协程只是说在一个线程中，通过自己控制自己，来让一个线程中可以执行多个协程程序，当前面的协程阻塞时，通过异步 IO 放在那里去执行，然后切换到其它的协程运行。在这个时候，**操作系统并不需要切换或者创新新的线程，减少了线程的切换时间**。注意，真正并行的只有线程，或者两个不相干的进程，而协程，并不是并行处理的，在线程中，它也是在 CPU 的时间分片机制下的切换执行。

**一个线程中的一个协程运行时，其它的协程是挂起状态的。它的切换成本非常低，而且是用户态可控的**。协程和进程、线程完全不是一个维度的概念，就像上面说的，它就是个函数。



现在协程是比较流行的开发趋势，特别是 Go 语言这种天生支持协程的语言大火也是有它的原因的。Swoole 其实也从很早就开始支持协程了，并且最主要是的，使用协程开发我们的代码还是可以使用原来的传统方式来写，极大地方便了我们的业务迁移。这些我们在后面都会接触到。

### 三程特点

我们来总结一下这三个程的特点。一个一个来说。

> 进程

- 就是我们执行的程序。程序和程序之间没有共享内容，都是自己独立的内存空间。
- 调度切换由操作系统完成，用户无感知，切换内容非常多，开销大，效率低。
- 需要通信的话一般是通过信号传递，或者外部工具。

> 线程

- 进程下面的小弟，同一个进程间的多个线程共享内存。
- **真正的并行执行，可以利用 CPU 的核数**。
- 调度切换比进程小，但一样是操作系统完成，开销和效率中等。

> 协程

- 线程的小弟，但其实更像是一个函数。
- 在线程之上的串行运行，**依赖于线程，而且依赖于异步 IO** ，如果没有 异步IO 的支持，它和线程没什么区别。
- 更加轻量级，用户态切换，开销成本低，效率非常高。

<img src="swoole课程.assets/image-20250627193515749.png" alt="image-20250627193515749" style="zoom: 33%;" />



### 总结

今天全是概念性的内容，参考的资料也非常多，这里就不一一列举了，反正就是百度来的，另外还包含一些我之前学过的 哈工大 操作系统 李治军 教授的这门慕课上的一些笔记。大家可以去学习一下哦，中国大学慕课上很多好的资源。之前我也一直说过，操作系统、数据结构、网络、设计模式 四大件是我们程序员的四大法宝，不说成为大神，但至少要都了解到。否则，你的进步会很慢，也很有限。





## **【Swoole系列3.2】Swoole 异步进程服务系统**

> https://www.zyblog.com.cn/article/768



在了解了整个进程、线程、协程相关的知识后，我们再来看看在 Swoole 中是如何通过异步方式处理进程问题的，并且了解一下线程在 Swoole 中的作用。

### Server两种运行模式

其实在之前的测试代码中，我们就已经见到过这两种模式了，只是当时没说而已。

不管是 Http 还是 TCP 等服务中，我们都有第三个参数的存在，默认情况下，它会赋值为一个 SWOOLE_PROCESS 参数，因此，如果是默认情况下我们一般不会写这个参数。而另外一种模式就是 SWOOLE_BASE 。

> SWOOLE_BASE模式

这种模式就是传统的**异步非阻塞模式**，它的效果和 Nginx 以及 Node.js 是完全一样的。

在 Node.js 中，是通过一个主线线程来处理所有的请求，然后对 I/O 操作进行异步线程处理，避免创建、销毁线程以及线程切换的消耗。

当 I/O 任务完成后，通过观察者执行指定的回调函数，并把这个完成的事件放到事件队列的尾部，等待事件循环。



这个东西吧，要讲清楚，开一个大系列都不为过。但是如果你之前学习过一点 Node 的话，那么其实就很好理解。因为我们之前写的各种 Server 服务代码，其实和 Node 中写得基本完全一样。

**都是一个事件，然后监听成功后在回调函数中写业务代码**。这就是**通过回调机制来实现的异步非阻塞模式**，将耗时操作放在回调函数中。有兴趣的同学可以去简单地学习一下 Node.js ，只要有 JS 基础，一两看看完一套入门教程就可以了。



在 Swoole 的 SWOOLE_BASE 模式下，原理也是完全一样的。当一个请求进来的时候，所有的 Worker 都会争抢这一个连接，并最终会有一个 Worker 进程成功直接和客户端建立连接，之后这个连接中所有的数据收发直接和这个 Worker 通讯，**不再经过主进程的 Reactor 线程转发**。





> SWOOLE_PROCESS模式

SWOOLE_PROCESS 的所有客户端都是和主进程建立的，内部实现比较复杂，用了大量的进程间通信、进程管理机制。适合业务逻辑非常复杂的场景，因为它可以方便地进行进程间的互相通信。



在 SWOOLE_PROCESS 中，所有的 Worker 不会去争抢连接，也不会让某一个连接与某个固定的 Worker 通讯，而是通过一个主进程进行连接。剩下的事件处理则交给不同的 Worker 去执行，当到达 Worker 之后，同样地也是使用回调方式进行处理了，后续内容基本就和 BASE 差不多了。也就是说，Worker 功能的不同是它和 SWOOLE_BASE 最大的差异，它实现了连接与数据请求的分离，不会因为某些连接数据量大某些量小导致 Worker 进程不均衡。**具体的方式就是在这种模式下，会多出来一个主管线程的进程**，其中还会有一个非常重要的 Reactor 线程，下面我们再详细说明。



> 两种模式的异同与优缺点

如果客户端之间不需要交互，也就是我们普通的 HTTP 服务的话，SWOOLE_BASE 模式是一个很好的选择，但是它除了 send() 和 close() 方法之外，不支持跨进程执行。但其实，**这两种模式在底层的处理上没什么太大的区别，都是走的异步IO机制。只是说它们的连接方式不同**。SWOOLE_BASE 的每个 Worker 都可以看成是 SWOOLE_PROCESS 的 Reactor 线程和 Worker 进程两部分的组合。



我们可以来测试一下。

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501, SWOOLE_BASE);

//$http = new Swoole\Http\Server('0.0.0.0', 9501, SWOOLE_PROCESS);

$http->set([
    'worker_num'=>2
]);

$http->on('Request', function ($request, $response) {
    var_dump(func_get_args());
    $response->end('开始测试');
});

$http->start();
```

通过切换上面两个注释，我们就可以查看两种服务运行模式的情况，可以通过 pstree -p 命令。

在 SWOOLE_BASE 模式下，输出的内容是这样的。

![image-20250707193608399](swoole课程.assets/image-20250707193608399.png)

可以看到，在 1629 这个进程下面有两个子进程 1630 和 1631 。然后切换成 SWOOLE_PROCESS 模式，再查看进程情况。

![image-20250707193704618](swoole课程.assets/image-20250707193704618.png)

很明显，这里不一样了，在 1577 这个 Master 主进程下，有两个进程，一个是 1578 ，一个是 1579 它表示的是线程组，然后在 1578 Manager 管理进程下面，又有 1580 和 1581 两个 Worker 进程。

同样地，我们使用之前在 **【Swoole教程2.5】异步任务**https://mp.weixin.qq.com/s/bQt9Ul-H34eUYw2-Qu-N0g 中的代码来测试，可以看到 Task 异步任务也是起的进程。（注意，我们在测试代码中设置的是 task_worker_num，没有设置 worker_num ，所以是 1个Worker + 4个 TaskWorker 进程，最后再加一个 PROCESS 模式的 线程组 ）如果如图所示。

![image-20250707193902948](swoole课程.assets/image-20250707193902948.png)

到这里，相信你也看出了，SWOOLE_BASE 比 SWOOLE_PROCESS 少了一层进程的递进，也就是少了一个层级。在 SWOOLE_BASE 模式下，没有 Master 进程，只有一个 Manager 进程，另外就是没有从 Master 中分出来的线程组。关于 Master/Manager/Reactor/TaskWorker 这些东西我们下一小节就会说到。



BASE 模式因为更简单，所以不容易出错，它也没有 IPC 开销，而 PROCESS 模式有 2 次 IPC 开销，master 进程与 worker 进程需要 Unix Socket 进行通信。IPC 这东西就是同一台主机上两个进程间通信的简称。它一般有两种形式，一个是通过 Unix Socket 的方式，就是我们最常见的类似于 php-fcgi.sock 或者 mysql.sock 那种东西。另一种就是 sysvmsg 形式，这是 Linux 提供的一种消息队列，一般使用的比较少。



当然，BASE 模式也有它自身存在的问题，主要也是因为上面讲过的特性。由于 Worker 是和连接绑定的，因此，某个 Worker 挂掉的话，这个 Worker 里面的所有连接都会被关闭。另外由于争抢的情况，Worker 进程无法实现均衡，有可能有些连接数据量小，负载也会非常低。最后，如果回调函数中阻塞操作，会导致 Server 退化为同步模式，容易导致 TCP 的 backlog 队列塞满。不过就像上面说过的，Http 这种无状态的不需要交互的连接，使用 BASE 没什么问题，而且效率也是非常 OK 的。当然，**既然默认情况下 Swoole 已经为我们提供的是 SWOOLE_PROCESS 进程了，那么也就说明 SWOOLE_PROCESS 模式是更加推荐的一种模式**。



### 各种进程问题

接下来，我们继续学习上面经常会提到的各种进程和线程问题。

#### Master 进程

它是一个多线程进程。用于管理线程，它会创建 Master 线程以及 Reactor 线程，并且还有心跳检测线程、UDP 收包线程等等。



#### Reactor 线程

这个线程我们不止一次提到了，它是在 Master 进程中创建的，负责客户端 TCP 连接、处理网络 IO 、处理协议、收发数据，它不执行任何 PHP 代码，用于将 TCP 客户端发来的数据缓冲、拼接、拆分成完整的一个请求数据包。我们在 Swoole 代码中操作不了线程，为什么呢？其实 PHP 本身就是不支持多线程的，Swoole 是一种多进程应用框架。在这里的线程是在底层用 C/C++ 封装好的。因此，也并没有为我们提供可以直接操作线程的接口。但我们已经学习过了，协程本身就是工作在线程之上的，而且，协程也已经是现在的主流方向了，所以在 Swoole 中，进程管理和协程，才是我们学习的重点。



#### TaskWorker 进程

它是接受收 Worker 进程投递过来的任务，处理任务后将结果返回给 Worker 进程，这种模式是同步阻塞的模式，同样它也是以多进程的方式运行的。

#### Manager 进程

这个进程主要负责创建、回收 Worker/TaskWorkder 进程。其实就是一个进程管理器。

#### 它们的关系

首先，我们先来看两张图，也是官网给出的图，并根据这两张图再来看看官网给出的例子。

<img src="swoole课程.assets/image-20250707194256052.png" alt="image-20250707194256052" style="zoom:50%;" />

第一张图主要是 Manager 和 Master 的功能。我们主要看第二张图。

<img src="swoole课程.assets/image-20250707194325992.png" alt="image-20250707194325992" style="zoom: 80%;" />

在这张图中，我们可以看到，Manager 进程创建、回收、管理最下面的 Worker 进程和 Task 进程。并且是通过操作系统的 fork() 函数创建的，这个东西如果学过操作系统的同学应该不会陌生，fork() 就是创建子进程的函数。子进程间通过 Unix Socket 或者 MQ 队列进行通信。如果你是 BASE 模式，那么就不会有 Master 进程，这个时候，每一个 Worker 进程自己会承担起 Reactor 的功能，接收、响应请求数据。



如果你是使用的 PROCESS 模式，那么上面的 Master 进程就会创建各种线程，还记得那个大括号的线程组吧，这个可是 BASE 模式没有的。它用于处理请求响应问题，不用想，多线程方式对于连接请求来说效率会更高。这也是前面说过的两种模式的优缺点的具体体现。然后 Reactor 线程通过 Unix Socket 与 Worker 进行通讯，完成数据向 Worker 的转发与接收。



我们用官网给出的例子来再说明一下它们之间的关系。Reactor 就是 nginx，Worker 就是 PHP-FPM 。Reactor 线程异步并行地处理网络请求，然后再转发给 Worker 进程中去处理。Reactor 和 Worker 间通过 Unix Socket 进行通信。



在 PHP-FPM 的应用中，经常会将一个任务异步投递到 Redis 等队列中，并在后台启动一些 PHP 进程异步地处理这些任务。这个场景相信大家都不会陌生吧，比如说我们下了订单之后，在原生 PHP 环境下进行消息通知、邮件发送，我们都会直接将这种问题放到一个队列中，然后让后台跑起一个脚本去消费这些队列从而进行信息发送。而 Swoole 提供的 TaskWorker 则是一套更完整的方案，将任务的投递、队列、PHP 任务处理进程管理合为一体。通过底层提供的 API 可以非常简单地实现异步任务的处理。另外 TaskWorker 还可以在任务执行完成后，再返回一个结果反馈到 Worker。



一个更通俗的比喻，假设 Server 就是一个工厂，那 Reactor 就是销售，接受客户订单。而 Worker 就是工人，当销售接到订单后，Worker 去工作生产出客户要的东西。而 TaskWorker 可以理解为行政人员，可以帮助 Worker 干些杂事，让 Worker 专心工作。



上述内容需要好好理解，特别是对于我们些长年接触传统的 PHP-FPM 模式开发的同学来说，要转换思维很不容易。不过根据官方提供的例子，相信大家也能很快把这个弯转过来。普通的请求就是把我们的 Nginx+PHP-FPM 给结合起来了，而 Task 则是可以处理一些类似于消息队列的异步操作。



### Swoole 服务运行流程

最后，我们再来了解一下整体 Swoole 服务的运行流程，同样也是来自官网的图片。

<img src="swoole课程.assets/image-20250707194411742.png" alt="image-20250707194411742" style="zoom: 67%;" />

其实这个流程图和我们的代码流程非常类似。定义一个 Server 对象，使用 set() 方法设置参数，然后使用 on() 方法开始监听各种回调，最后 start() 方法启动服务。在服务启动之后，创建了 Manager 进程，如果是 PROCESS 模式的话，则是先创建一个 Master 进程，然后在 Master 之下创建 Manager 。接着，Manager 根据 worker_num 数量创建并管理相对应数量的 Worker 进程。其中，可以在 Worker 中创建异步的 task 进程。



Reactor 线程在最外面处理请求响应问题，监听相对应的事件，并与 Worker 进行通信。如果是 BASE 模式，不存在 Reactor 线程，则是全部由 Worker 来解决，而且它与连接的关系是一对一的。

### 总结

又是让人晕头转向的一篇文章吧。在今天的学习中，最主要的其实还是一种思维的转变，那就是我们要通过多进程的方式来提供服务应用。而且这种模式其实并不陌生，Nginx+PHP-FPM 就是这种模式，只不过，PHP-FPM 本身就是一个进程管理工具，但它的效率以及实现方式都与 Swoole 略有差别。包括在 PHP8 之后的 JIT ，它就是通过 OPCahce 来实现的，也是在将大部分代码全部一次加载到内存中，就像 Swoole 一样节约每次 PHP-FPM 的全量加载问题从而提升性能。



测试代码：[https://github.com/zhangyue0503/swoole/blob/main/3.Swoole%E8%BF%9B%E7%A8%8B/source/3.2Swoole%E5%BC%82%E6%AD%A5%E8%BF%9B%E7%A8%8B%E7%B3%BB%E7%BB%9F.php](https://github.com/zhangyue0503/swoole/blob/main/3.Swoole进程/source/3.2Swoole异步进程系统.php)

参考文档：

https://wiki.swoole.com/#/server/init

https://wiki.swoole.com/#/learn?id=process-diff



## 补充

### 共享内存

在 Swoole 中，**默认情况下 Worker 进程是隔离的**，每个 Worker 有独立的内存空间。如果想让所有 Worker 共享数据，需要使用 **跨进程共享机制**有以下几种方式：

#### **1. 使用 `Swoole\Atomic`（原子计数器）**

**适用场景**：简单的计数器、标志位等
​**​特点​**​：

- 原子操作（线程/进程安全）
- 仅支持整数
- 高性能（无锁）

**示例代码**

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);

// 定义一个全局 Atomic 计数器
$counter = new Swoole\Atomic(0);

$http->on('Request', function ($request, $response) use ($counter) {
    $count = $counter->add(1); // 原子递增
    $response->end("Global Count: " . $count);
});

$http->start();
```

**输出**：

```
Global Count: 1
Global Count: 2
Global Count: 3
...
```

所有 Worker 共享 `$counter`，计数会连续递增。



#### **2. 使用 `Swoole\Table`（共享内存表）**

**适用场景**：需要共享复杂数据（数组、KV 存储）
​**​特点​**​：

- 支持 `int`、`float`、`string` 类型
- 类似 PHP 数组的操作方式
- 适用于多进程共享配置、Session 等

**示例代码**

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);

// 创建共享内存表
$table = new Swoole\Table(1024); // 1024 行
$table->column('count', Swoole\Table::TYPE_INT); // 整数字段
$table->column('name', Swoole\Table::TYPE_STRING, 64); // 字符串字段
$table->create();

// 初始化数据
$table->set('global', ['count' => 0, 'name' => 'Swoole']);

$http->on('Request', function ($request, $response) use ($table) {
    // 原子递增
    $table->incr('global', 'count');
    
    // 获取当前值
    $data = $table->get('global');
    $response->end("Count: {$data['count']}, Name: {$data['name']}");
});

$http->start();
```

**输出**：

```
Count: 1, Name: Swoole
Count: 2, Name: Swoole
Count: 3, Name: Swoole
...
```

所有 Worker 共享 `$table` 数据。

------

#### **3. 使用 `Redis` / `APCu`（外部存储）**

**适用场景**：

- 需要持久化存储
- 多机共享数据
- 复杂数据结构（如哈希、集合）

**示例（Redis）**

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

$http->on('Request', function ($request, $response) use ($redis) {
    $count = $redis->incr('global_counter');
    $response->end("Redis Count: " . $count);
});

$http->start();
```

**输出**：

```
Redis Count: 1
Redis Count: 2
Redis Count: 3
...
```

所有 Worker 共享 Redis 数据。

------

#### **4. 使用 `shmop`（共享内存扩展）**

**适用场景**：

- 需要更底层的共享内存操作
- 高性能 IPC（进程间通信）

**示例**

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);

// 创建共享内存块
$shmKey = ftok(__FILE__, 't');
$shmId = shmop_open($shmKey, "c", 0644, 128);

$http->on('Request', function ($request, $response) use ($shmId) {
    // 读取共享内存
    $count = (int) shmop_read($shmId, 0, 10);
    $count++;
    
    // 写入共享内存
    shmop_write($shmId, (string) $count, 0);
    
    $response->end("Shared Memory Count: " . $count);
});

$http->start();
```

**注意**：`shmop` 需要手动管理内存，容易出错，建议优先使用 `Swoole\Table` 或 `Redis`。

------

#### **5. 使用 `Swoole\Process` + `msgpack`（进程通信）**

**适用场景**：

- 需要自定义进程间通信
- 复杂数据交换

### **示例**

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);

// 创建 Unix Socket 通信
$process = new Swoole\Process(function ($worker) {
    $count = 0;
    while (true) {
        $data = $worker->read();
        $count++;
        $worker->write($count);
    }
}, false, 2); // 2 = SOCK_DGRAM

$process->start();

$http->on('Request', function ($request, $response) use ($process) {
    $process->write("increment");
    $count = $process->read();
    $response->end("Process IPC Count: " . $count);
});

$http->start();
```

**说明**：

- 这种方式适用于 **复杂 IPC 场景**，但性能低于 `Swoole\Table` 和 `Redis`。

<img src="swoole课程.assets/image-20250708183101678.png" alt="image-20250708183101678" style="zoom: 50%;" />

# 其他

## **swoole进程**

进程并发



```
$workerNum = count($subGameIds);

for ($i = 0; $i < $workerNum; $i++) {
    $process = new swoole_process('callback_function', false);
    $pid = $process->start();
    $workers[$pid] = $process;
}
```





## PHP-FPM（FastCGI Process Manager）





### FastCGI

FastCGI（Fast Common Gateway Interface）是一种 **高性能的 CGI（通用网关接口）协议改进版**，旨在解决传统 CGI 的性能瓶颈，专为高并发 Web 应用设计。以下是深度解析：

------

**1. 核心作用**

- **持久化进程**：与 CGI 的“每次请求新建进程”不同，FastCGI 保持**进程常驻**，重复处理多个请求。
- **协议标准化**：定义 Web 服务器（如 Nginx）与后端程序（如 PHP/Python）的通信规则。
- **性能优化**：减少进程创建/销毁的开销，提升吞吐量。



**2. 工作原理**

<img src="swoole课程.assets/下载.png" alt="下载" style="zoom:50%;" />

**3. 关键特性**

|       特性        |                       说明                        |
| :---------------: | :-----------------------------------------------: |
|   **进程复用**    | 一个进程处理多个请求（vs CGI 的“单次请求单进程”） |
|  **协议无关性**   |        支持任何语言（PHP/Python/Perl 等）         |
| **网络/本地通信** |  可通过 TCP/IP 或 Unix Socket 与 Web 服务器交互   |
|   **负载均衡**    |            支持多后端进程并行处理请求             |



**4. 与传统 CGI 的对比**

|              |          FastCGI           |         传统 CGI         |
| :----------: | :------------------------: | :----------------------: |
| **进程模型** |    ✅ 持久化进程（复用）    |    ❌ 每次请求新建进程    |
|   **性能**   | ⚡️ 高并发下延迟低、吞吐量高 | 🐢 频繁创建进程导致高延迟 |
| **资源占用** |        内存占用稳定        |      内存碎片化严重      |
| **适用场景** |   生产环境（如 PHP-FPM）   | 仅历史遗留系统或开发测试 |

**5. 通信流程（以 Nginx + PHP-FPM 为例）**

1. **请求到达**
   Nginx 接收到用户请求（如 `index.php`）。
2. **协议转换**
   Nginx 通过 FastCGI 协议将请求转发给 PHP-FPM **进程池**。
3. **进程分配**
   PHP-FPM 分配一个空闲的 PHP 进程处理脚本。
4. **结果返回**
   PHP 进程执行完毕后，将结果通过 FastCGI 协议返回给 Nginx。



**6. 配置示例**

> **Nginx 配置片段**

```nginx
location ~ \.php$ {
    fastcgi_pass   127.0.0.1:9000;  # 转发到 PHP-FPM
    fastcgi_index  index.php;
    include        fastcgi_params;  # 包含 FastCGI 标准参数
    fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```

> **PHP-FPM 配置片段**

```ini
[www]
listen = 127.0.0.1:9000  # 监听 FastCGI 请求
pm = dynamic             # 进程管理方式
```

**7. 性能优化建议**

- **进程池调优**：根据服务器内存调整 PHP-FPM 的 `pm.max_children`。
- **缓存加速**：启用 OPcache + APCu 减少重复计算。
- **协议选择**：Unix Socket 比 TCP/IP 更快（本地通信时）。

**8. 常见实现**

|  语言  | FastCGI 实现  |       典型用途       |
| :----: | :-----------: | :------------------: |
|  PHP   |    PHP-FPM    | 与 Nginx/Apache 协作 |
| Python | flup/fastcgi  |  Django/Flask 部署   |
|  Ruby  | FCGI (legacy) |    旧版 Ruby 应用    |

**9. 为什么现代 Web 需要 FastCGI？**

- **解决 C10K 问题**：通过进程复用支撑高并发。
- **安全性**：隔离 Web 服务器与后端程序（避免直接执行 CGI 脚本）。
- **灵活性**：支持分布式部署（Web 服务器与后端程序可分离）。





