# 构建PHP

除非你从php.net以tar压缩包的形式下载源代码并自己编译，否则你很可能只少缺少一个组件。

> **注意**
>
> 目前使用Git通过https://github.com/php/php-src/进行下载编译安装

## *nix工具

任何C语言开发者工具箱中的第一件工具都是一个C的编译器。有一个很好的机会，你的Linux发行版默认包含一个，而且很有可能包含gcc（GNU Compiler Collection）。你可以通过发布gcc -v或cc -v来轻松地检查是否安装了编译器，其中之一将有望成功运行，并以安装的编译器的版本信息作为响应。

如果你还没有编译器，请查看你的Linux版本，以获取有关下载和安装gcc的说明。通常这将相当于下载一个.rpm或.deb文件并发出一个命令来安装它。根据具体的分布情况，下面的一个命令可以直接使用而不需要进一步研究：

```shell
urpmi gcc
apt-get install gcc
yum install gcc
pkg-add -r gcc
emerge gcc
```

> **注意**
>
> 因为本书成书时间较早，那个时候，Unix、FreeBSD、Linux都属于这个范畴。现在，我们可以在一定程度上直接视为Linux的各种版本：redhat、centos、debain、ubuntu、kali等。即便是BSD的衍生MacOS，在使用上也可以视为Linux，当然它们都属于*nix。

除了编译器之外，还需要以下程序：make、autoconf、automake和libtool。这些程序可以让你进行编译安装。

为了获得最佳效果，推荐使用libtool 1.4.3和autoconf 2.13以及automake 1.4或1.5版本。使用这些软件包的更新版本很可能会工作，但只有这些版本是 认证的。

> **注意**
>
> 这里文章介绍了使用cvs检查更新，并需要bison和flex来构建语言解析器。但是时代已经变了，目前更推荐使用Git：
>
> bison的git：http://git.savannah.gnu.org/cgit/bison.git
>
> flex的git：https://github.com/westes/flex



## win32工具

Win32/PHP5构建系统是一个完整的重写，是从PHP4构建系统的重大飞跃。在Windows下编译PHP4的说明可以在php.net上找到，只需要在这里讨论需要Windows 2000、Windows 2003或Windows XP的PHP5编译系统。

首先你需要获取许多核心PHP扩展使用的库和开发头。幸运的是，这些文件中的许多文件都是从php.net重新分发的。

> **注意**
>
> 现在windows已经从2008到win10,中间有无数个新的操作系统版本了，更多关于Windows下PHP安装环境，这里可以直接从PHP的Windows网站中下载。
>
> php4windows：http://windows.php.net/
>
> php4windows下载：http://windows.php.net/download

## 获取PHP源代码

下载PHP源码时，我们可以通过Git直接获取

```shell
git clone https://github.com/php/php-src.git
```

