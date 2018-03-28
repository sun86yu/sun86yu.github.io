---
layout: post
book: true
background-image: http://ot1cc1u9t.bkt.clouddn.com/17-7-16/91630214.jpg
category: 工具
title: Composer
tags:
- Composer
- PHP
---

用于 PHP 依赖关系处理的 Composer
===
>利用强大的开源工具从第三方库组装 PHP 项目

现在 PHP 开发人员往往依靠第三方库来帮助自己更快地构建项目。但是，软件重用的好处是有代价的：我们不仅必须管理每个应用程序安装所需的库的列表，还必须管理所创建的依赖关系树，因为所使用的库均构建于其他库之上。

其中一个解决方案是将自己所需的所有库与自己的代码放在一起。这种方式在某程度上是可行的，但所造成的麻烦往往比它解决的问题更大。如果我们需要维护自己的本地库代码，就必须通过下载新的版本，并将其加入代码中，再手动执行并进行错误修复。

最终，我们会在这些内容上做许多工作。由于这些原因，我们通常不对三方库进行更新。

PHP Extension and Application Repository (PEAR) 项目的部分设计目的是为了解决这个问题。PEAR 提供了一组配合工作的库，程序员可以为该库做出贡献。PEAR 还包括命令行工具，用于安装所需的库及其依赖关系（如果有的话）。PEAR 在很长一段时间中曾是最好的办法，并且有很多人使用它，但这个系统也有其不足之处:

PEAR 库是一些扩展，它装在操作系统里。虽然这种设计可以避免将库引入到您自己的代码中，增加维护成本。但它导致的问题也不少，我们永远不知道需要运行哪个版本的库。

2011 年 4 月，两位 PHP 开发人员（Nils Adermann 和 Jordi Boggiano）认为 PHP 的依赖关系处理问题需要有一个新的解决方案，并开始进行开发。他们在 2012 年 3 月 1 日发布了 Composer。在 Composer 中，您可以创建一个配置文件，指定应用程序所需的第三方库（无论它们被托管在哪里）。然后运行 Composer，编写 完整的应用程序：Composer 下载您指定所有的库及其所有依赖关系。

安装 Composer
---
Composer 是一个多平台工具。在任何基于 UNIX 或 Linux 的计算机上都可以安装:

```
curl -sS https://getcomposer.org/installer | php
```
CentOS 可以直接通过 yum 安装:

```
yum install composer
```

而 mac 也可以直接 ``` brew install composer```

如果是自己下载运行安装的，可以将运行后得到的执行文件放到 /usr/local/bin 下:

```
mv composer.phar /usr/local/bin/composer
```
这样我们就可以全局使用该命令了

基本用法
---
最常见的用法是，根据第三方提供的配置，为第三方 PHP 应用程序或框架创建/下载/安装一个代码库。

例如，可以 使用 Composer 安装 phpunit及其所有依赖关系。

```
{
   "require":{
      "phpunit/phpunit":"4.8.*@dev"
   }
}
```
运行 Composer 的 install 命令，Composer 就会完成余下的工作。

```
composer install
```

Composer 将所有的库安装到一个命名为 vendor 的文件夹，将它们与我们自己的项目代码区分开来。

在 vendor 文件夹中，它创建了一个名为 autoload.php 的文件。我们可以在项目中包括该文件，这等于为 Composer 所下载的所有库都安装了一个自动加载程序：

```
require 'vendor/autoload.php';
```

Composer 如何查找库
---
要下载库包，Composer 首先需要知道在哪里可以找到这些软件包。信息由 Composer 存储库 提供：在线来源列出了 Internet 上提供的软件包、如何检索它们，以及它们自己的依赖关系。虽然任何人都可以维护自己的存储库，以提供对内部库的访问权限（Composer 网站为此提供了 说明），但我们通常使用的主要存储库是 Packagist(https://packagist.org/)。Packagist 提供为 PHP 中的大部分开源项目提供软件包。

要维护好这一个三方库的大集合，主要靠 PHP Framework Interop Group （原名 PHP Standards Group）。该组织是在 php[tek] 2009 会议 上成立的。成立 PHP-FIG（代表着众多流行的 PHP 应用程序和框架的一组人）就是为了看看人们的项目如何能够更好地协同工作。该合作的高潮是 PHP Standards Recommendations (PSR) 的创建，它描述库的可选标准。实现这些共同标准的库能够在一组共同的期望下互操作。

在此，感谢开源，感谢组织！

最重要的 PSR 是 PSR-0 和 PSR-4，它们促进了 Composer 的创建。这些 PSR 为类和命名空间声明一个共同的命名方法，以及它们应该如何将文件映射到文件系统上。然后，一个共同的自动加载程序接口可以从它需要的任何库加载类。创建一个通用的标准方式，让库不需要覆盖彼此就可以共享它们的类，这使得 Composer 变得非常有效。

为自己的项目配置 Composer
---
我们开始编写一个新的 PHP 小程序，能够将 Markdown 文件转换为 HTML 输出。在 Packagist 搜索 markdown，显示 michaelf/php-markdown 库作为一个不错的选择。


为了告诉 Composer 在项目中要包括哪些文件，创建一个名为 ***composer.json*** 的配置文件。这个 JSON 格式的文件可以包含各种命令，但最常用的（而且往往是唯一的）命令是 require 键。将想要的软件包名称以及将要支持的版本传递给这个键：

```
{
   "require":{
      "michelf/php-markdown":"1.4.*"
   }
}
```

现在，在我们的应用程序目录中运行 ```composer install```。 Composer 需要几分钟来下载指定的库到一个 vendor 目录中，并创建一个包括此目录的自动加载程序。

执行完后可以看到当前目录下有了 vendor 目录,它的结构如下:
![](/images/composer_1.png)

现在我们可以编写依赖于 michelf/php-markdown 软件包的代码。如下:

test.php
```
<?php
require 'vendor/autoload.php';
 
use \Michelf\Markdown;
echo Markdown::defaultTransform(file_get_contents("php://stdin"));
```

我们运行上面的代码:

```
php test.php
```

这时候我们可以在终端上输入文字。我们要输入的是 markdown 格式的内容，输入完后按 ctrl+d 终止。程序会输出相应的 html 代码,如:

```
hello
===
```yum install composer```

下面的是输出的内容:

<h1>hello</h1>

<p><code>yum install composer</code></p>
```

项目运行成功! 我们选择的 Markdown 包碰巧没有额外的依赖关系。如果我们选择了一个有依赖关系的软件包，Composer 将在同一时间自动下载所有这些依赖关系并配置它们。

指定版本
---
我们可以根据自己的需求，将任意数量的库添加到 composer.json 文件。此外，对于每一个库，都可以指定想接受的版本。指定版本是一个很重要的部分，可以确保代码始终可用。通过在版本号中使用通配符，甚至可以允许 Composer 以我们的名义升级库。

在前面的示例中，我指定的版本为 "1.4.*"。现在，每当运行 composer install 时，Composer 将查找 1.4 版库的最新版本，但不会接受 1.5、2.0 或其他任何更高版本。如果我希望总是获得库的最新版本，那么我可以指定 "*"（但如果库底层 API 被更改，这可能会引起问题，又要我们自己适配）。

指定版本的方式还有几种，分别是：

<table>
<tr>
<td style="width:80px">确切版本</td>
<td>1.0.2</td>
<td>软件包的确切版本。</td>
</tr>
<tr>
<td>范围</td>
<td> >1.0  >=1.0  <2.0   >=1.0  <1.1 || >=1.2</td>
<td>比较运算符可以指定有效版本的范围。有效的运算符是 >、>=、<、<= 和 !=。可以定义多个范围，而且默认情况下按照 AND 处理，或者用双竖线 (||) 分开它们，则作为一个 OR 运算符。</td>
</tr>
<tr>
<td>连字符范围</td>
<td>1.0 - 2.0</td>
<td>创建一个包容性的版本集。</td>
</tr>
<tr>
<td>通配符</td>
<td>1.0.*</td>
<td>带有 * 通配符的模式。1.0.* 相当于 >=1.0 <1.1。</td>
</tr>
<tr>
<td>波浪运算符</td>
<td>~1.2.3</td>
<td>“下一个重要版本”：允许最后一位数字增加，因此变得和 >=1.2.3 <1.3.0 一样。允许最后一位数字增加。</td>
</tr>
<tr>
<td>^运算符</td>
<td>^1.2.3</td>
<td>“下一个重要版本”：类似于波浪线运算符，但假设语义版本和直到下一个主要版本的所有变更都应该被允许，因此变得和 >=1.2.3 <2.0 一样。</td>
</tr>
</table>

软件包稳定性
---
在配置 Composer 获取项目所需要的准确的库时，另一个要考虑的因素是想要的库版本有多稳定。如果需要最新的版本或正在帮助测试软件，那么可以请求软件包的测试版或开发分支。我们可以指定稳定性标志，将它添加到 require 字符串的末尾，并使用 @。例如，为了请求 PHPUnit 的最新开发版本，可以指定：

```
{
   "require":{
      "phpunit/phpunit":"4.8.*@dev"
   }
}
```

对于引用，Packagist 显示存在哪些分支，以及引用它们需要使用的字符串。packagist 的库详情页面里有标识。如：

![](/images/composer_2.png)

版本锁定
---
使用 Composer 的巨大好处是，我们的代码中不需要包括第三方库。在发布软件时，让 Composer 为我们下载并配置所有库，从而保持代码精简。但是，可能会遇到问题: 代码要部署到 20 个服务器，代码的部署和 composer install 的运行分别在不同的时间发生，可能导致不同服务器上下载不同版本的库，其结果可能是灾难性的。

这就要求我们在配置库的版本的时候要十分注意。当然，我们也可以把该任务交给 composer.lock 文件。初次运行 composer install 时，会自动创建了一个 composer.lock 文件。该文件指定到底要安装哪些库，以及过程中下载哪些特定的版本。当我们将项目提交到版本控制软件（Git, SVN等）中的时候，提交 composer.lock 文件，而不是 vendor 目录。当准备好一个代码的新部署，并且运行了 composer install 的时候，Composer 会首先查找一个锁定文件。如果找到该文件，则会安装一个与原始安装完全相同的副本，以确保所有安装的一致性。

如果我们要升级三方库，当然可以删除整个 vendor目录以及 composer.lock 文件。也可以直接运行``` composer update ```。Composer 会比较已安装的软件版本与锁定文件。如果在配置允许的版本范围内有新的版本可用，那就会自动为我们更新。

为自己的类创建一个自动加载程序
---
Composer 还有许多内置的功能。其中一个最好的功能是，我们可以在配置中指定自己的类，使得 Composer 可以自动在自动加载程序中生成相应的代码。更改一下 composer.json:

```
{
  "require":{
    "michelf/php-markdown":"1.4.*"
  },
  "autoload":{
    "psr-4":{"Converter\\":"src/"}
  }
}
```

我指定了一个名为 Converter 的命名空间，并说明该命名空间的所有类文件均存在于名为 src 的相对目录中。

将之前生成的 vendor 目录删除。然后重新执行 ```composer install```。这时候会重新下载一份库，并且生成自动加载的代码(在 vendor/composer 目录中)。

其中，目录中有一个 autoload_psr4.php 文件，它里面有如下代码:

```
<?php

// autoload_psr4.php @generated by Composer

$vendorDir = dirname(dirname(__FILE__));
$baseDir = dirname($vendorDir);

return array(
    'Converter\\' => array($baseDir . '/src'),
);
```
可以理解为：已经帮我们做好了 Converter 全名空间里各类自动加载的定义。

所以，如果我有一个名为 Converter\CommandLine 的类，自动加载程序将会查找这个类，它在文件系统中的地址是 src/CommandLine.php。

提供自己的软件包
---
此时，我可以将 Markdown 到 HTML 的转换器应用程序提供为一个 Packagist 上的软件包。（由于转换器是一个应用程序，而不是一个可重用的库，将它作为一个软件包提供并没有实际意义，但是，为了本练习的需要，我们假装它是一个库）。

从本质上讲，通过创建我的 composer.json 文件，该应用程序本身是一个软件包，可以安装它。

包名称的格式必须是 vendor/package。所以，在我的例子中，我将下面的代码添加到配置文件：

```
"name":"EliW/Converter",
```

其他一些配置也值得补充。可以将所需的 PHP 版本指定为 Composer 将执行的虚拟软件包名称。为了指定 PHP 版本号，并将我自己的转换器打包为完整的软件包:

```
{
  "name":"EliW/Converter",
  "require":{
    "michelf/php-markdown":"1.4.*",
    "php":">=5.3.0"
  },
}
```

配置现在是完整的。可以将自己的应用程序提交给一个在线版本控制系统（如 GitHub），然后登录到我的 Packagist 帐户，并提交我的软件包信息，让其他人可以将我的应用程序包含在他们的项目中。
