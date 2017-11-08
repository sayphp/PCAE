# 在UNIX上编译

现在你已经将所有必要的工具组合在一起，你已经下载了PHP源代码压缩包，并且已经确定了所有必需的 ./configure开关，现在是编译PHP的时候了。

假设你已经php-5.1.0.tar.gz下载到你的主目录，你将输入一下一系列命令解压tar包并切换到PHP源目录：

```shell
cd /say/
tar -zxf php-5.1.0.tar.gz
cd php-5.1.0
```

如果你使用的不是GNU tar的工具，则可能需要使用稍微不同的命令：

```shell
gzip -d php-5.1.0.tar.gz | tar -xf -
```

现在，使用*./configure*命令启用或禁用你的任何其他选项。

```shell
./configure enable-debug enable-maintainer-zts disable-cg enable-cli disable-pear disable-xml disable-sqlite without-mysql enable-embed
```

在经过漫长的等待过程后，你将准备开始编译过程：

```shell
make
make test
make install
```

