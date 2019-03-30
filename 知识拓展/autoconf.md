> [autoconf][1]是M4宏的扩展包。可生成用于自动配置软件源代码包的shell脚本。这些脚本可以使软件包适应多种类UNIX（遵循Posix）系统，无需用户手动干预。Autoconf从模板文件为包创建配置脚本，该模板文件以M4宏调用的形式列出了包可以使用的操作系统功能。

#### 下载

```bash
#源码
git clone git：//git.sv.gnu.org/autoconf
git clone http://git.sv.gnu.org/r/autoconf.git
#安装
sudo apt install autoconf
```

#### 使用

简单使用：

```bash
#1.编写C的程序
vim hello.c
vim hello.h
#2.扫描程序，生成configure.scan
autoscan
#3.编辑configure.ac
mv configure.scan configure.ac
vim configure.ac
#3.1编辑包、版本、报告bug通知邮箱
AC_INIT(hello, 1.0, whoam163@163.com)
#3.2新版本增加automake
AM_INIT_AUTOMAKE
#3.3输出文件
AC_OUTPUT(Makefile)
#4.生成aclocal.m4
aclocal
#5.生成configure
autoconf
#6.创建并编辑Makefile.am
vim Makefile.am
#6.1
AUTOMAKE_OPTIONS=foreign
bin_PROGRAMS=hello
hello_SOURCES=hello.c hello.h
#7.生成Makefile.in
automake --add-missing
#8.编译、运行
make
./hello
```



#### 资料

[1]: https://www.gnu.org/software/autoconf/	"GNU autoconf官网"
[2]: https://www.gnu.org/savannah-checkouts/gnu/autoconf/manual/autoconf-2.69/html_node/index.html	"GNU autoconf手册"

