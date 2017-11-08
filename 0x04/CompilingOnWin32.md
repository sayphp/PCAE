# 在Win32上编译

与UNIX构建一样，准备Windows构建的第一步是解压缩源代码包。默认情况下，Windows不知道如何处理.tar.gz文件。

首先将文件重命名为PHP-5.1.0.tar.gz。如果你安装了能够读取.tar.gz文件的程序，你会注意到该图标立即更改。你现在可以双击文件夹来打开解压缩程序。如果图标没有改变，或者双击图标时没有任何反应，则意味着没有安装与tar/gzip兼容的解压缩程序。检查你最新换得搜索引擎WinZIP、WinRAR或任何其他适合提取.tar.gz档案的应用程序。

无论你使用哪种解压缩程序，都要将php-5.1.0.tar.gz解压到你以前创建的root开发文件夹中。本节将假定你已将其压缩到C:\PHPDEV\，因为zip文件包含文件夹结构，将导致源树驻留在C：\PHPDEV\php-5.1.0中。

解压缩后，通过选择开始，所有程序，Windows Server 2003 SP1的Microsoft平台SDK，打开构建环境窗口，Windows 2000构建环境，设置Windows 2000构建环境（调试）打开构建环境窗口。此快捷方式的具体路径可能略有不同，具体取决于你安装的平台SDK的版本以及你要构建的目标平台（2000、xp、2003）。

一个简单的命令提示符窗口将打开说明目标构建平台。这个命令提示符拥有大部分但并非全部必要的环境变量。你需要运行一个额外的批处理文件，以便让PHP构建系统知道Visual C++ Express的位置。如果你接收默认安装位置，则此批处理文件将位于C:\ProgramFiles\MicrosoftVisualStudio8\VC\bin\vcvars32.bat中。如果找不到vcvars32.bat，请检查vcvarsall.bat的相同目录或其父项。只要确保在刚打开的命令提示符窗口中运行它。它将设置构建过程将需要的其他环境变量。

现在，将目录更改为解压PHP的位置：

```shell
C:\PHPDEV\php-5.1.0 and run buildconf.bat
C:\Program Files\Microsoft Platform SDK> cd \PHPDEV\php-5.1.0
C:\PHPDEV\php-5.1.0 > buildconf.bat
```

如果一切进展顺利，你将看到一下两行输出

```shell
Rebuilding configure.js
Now run 'cscript /nologo configure.s help'
```

在这一点上，你可以做消息说，看看有什么选项可用。enable-maintainer-zts选项在这里不是必要的，因为Win32版本会自动假定任何SAPI都需要ZTS。如果你想关闭它，你可以发出disable-zts,但这不是这种情况，因为你正在为开发环境而建造。

在这个例子中，为了简单起见，我闪除了一些与扩展和嵌入开发无关的扩展。如果你想使用其他扩展来重建PHP，则需要搜索它们所依赖的库。

> **注意**
>
> 对于PHP来说，Linux可能是更好的选择，建议在Linux下去编译PHP。