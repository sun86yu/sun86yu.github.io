---
layout: post
book: true
background-image: http://ot1cc1u9t.bkt.clouddn.com/17-7-16/91630214.jpg
category: 工具
title: CGI 和 FastCGI php-fpm
tags:
- CGI
- FastCGI
- php-fpm
---
CGI
==
CGI (Common Gateway Interface) 通用网关接口。

CGI 是WWW技术中最重要的技术之一，有着不可替代的重要地位。CGI是外部应用程序（CGI程序）与Web服务器之间的接口标准，是在CGI程序和Web服务器之间传递信息的规程。CGI规范允许Web服务器执行外部程序，并将它们的输出发送给Web浏览器。

CGI可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量。

最好选用易于归档和能有效表示大量数据结构的语言，例如UNIX环境中：

Perl (Practical Extraction and Report Language)

Bourne Shell或者Tcl (Tool Command Language)

PHP


CGI 程序工作原理
---

web server（如: Nginx）只是内容的分发者。比如，如果请求/index.html，那么web server会去文件系统中找到这个文件，发送给浏览器，这里分发的是静态资源。

如果现在请求的是/index.php，根据配置文件，Nginx知道这个不是静态文件，需要去找PHP解析器来处理，那么他会把这个请求简单处理后交给PHP解析器。

此时CGI便是规定了要传什么数据，以什么格式传输给php解析器的协议。

当 web server收到 /index.php这个请求后，会启动对应的CGI程序，这里就是PHP的解析器。接下来***PHP解析器会解析php.ini文件，初始化执行环境，然后处理请求***，再以CGI规定的格式返回处理后的结果，退出进程。web server再把结果返回给浏览器。

CGI针对每个http请求都是fork一个新进程来进行处理，***每一个Web请求PHP都必须重新解析php.ini、重新载入全部扩展并重初始化全部数据结构***。然后这个进程会把处理完的数据返回给web服务器，最后web服务器把内容发送给用户，刚才fork的进程也随之退出。 如果下次用户还请求动态资源，那么web服务器又再次fork一个新进程，周而复始的进行。

从这个过程就可以看出有许多重复的步骤，浪费了许多时间和资源。这就是 FastCGI改进的地方。

FastCGI 工作原理
---
Fastcgi则会先fork一个***master***，解析配置文件，初始化执行环境，然后再***fork多个worker***。

当请求过来时，master会传递给一个worker，然后立即可以接受下一个请求。这样就避免了重复的劳动，效率自然是高。而且当worker不够用时，master可以根据配置预先启动几个worker等着；当空闲worker太多时，也会停掉一些，这样就提高了性能，也节约了资源。

FastCGI生命周期可能如下：

1. Web Server启动时载入FastCGI进程管理器（IIS ISAPI, Apache Module, php-fpm)
2. FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。
3. 当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。
4. FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。
5. FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出了。

FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一次(这是CGI最为人诟病的fork-and-execute 模式)。

FastCGI是语言无关的、可伸缩架构的CGI开放扩展，其主要行为是将CGI解释器进程保持在内存中并因此获得较高的性能。CGI解释器的反复加载是CGI性能低下的主要原因，如果CGI解释器保持在内存中并接受FastCGI进程管理器调度，则可以提供良好的性能、伸缩性等等。

>大多数Fastcgi实现都会维护一个进程池。php-fpm 就是这样一种方式。另外， swoole 也可以替代 php-fpm 来实现这一功能。