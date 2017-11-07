# 配置PHP开发环境

正如第一章所讨论的那样，当构建一个开发友好的PHP时，有两个特殊的./configure开关可用，无论你是打算像PHP编写扩展还是将PHP嵌入到其他应用程序中。除了构建PHP时通常使用的其他开关之外，还应该使用这两个开关。

```shell
enable-debug
```

启用调试开关打开PHP和Zend源代码树中一些重要功能。首先，它能够在每个请求结束时报告泄漏的内存。

回想一下第三章”内存管理“，Zend内存管理器将隐式释放每个请求的内存，这个内存是在脚本结束之前分配的，但没有明确的释放。通过对新开发的代码进行一系列积极的回归测试，泄漏点可以以在任何公开发布之前轻易被发现和插入。看看下面的代码片段：

```c
void show_value(int n){
  char *message = emalloc(1024);
  sprintf(message, "The value of n is %d\n", n);
  php_printf("%s", message);
}
```

如果在PHP请求的过程中执行这段愚蠢的代码，将会泄漏1024字节的内存。在一般情况下，Zend内存管理器会在脚本执行结束时悄悄释放。

然而，在启用调式开启的情况下，开发人员会看到一条错误信息，告诉他们需要解决的问题。

```shell
/cvs/php5/ext/sample/sample.c(33) : Freeing 0x084504B8 (1024 bytes), script=-
=== Total 1 memory leaks detected ===
```

这个简短单内容丰富的消息告诉你，Zend内存管理器不得不在乱七八糟的时候清理掉，并且准确的从丢失的内存块的分配中识别出来。使用这些信息，打开文件，向下滚动到有问题的行，并在函数结尾添加一个适当的调用efree(message)是一件简单的事情。

内存泄漏并不是你遇到的唯一难题，当然很难追查到。有时候这些问题更加阴险，而且不那么有说服力。假设你一整晚都在打一个打补丁，需要设计十几个文件，并且需要更改一大堆代码。当一切都到位时，你可以自信的敲出*make*，尝试一个实例脚本，并将其视为一下的输出：

```shell
$ sapi/cli/php -r 'myext_samplefunc();'
Segmentation Fault
```

好吧，有些尴尬了，但问题在哪里？看看你的myext_samplefunc()的实现没有显示任何明显的线索，通过gdb运行它只显示一堆未知的符号。

再次，启用调式帮助。通过将此开关添加到./configure，生成的PHP二进制文件将包含gdb或另一个核心文件检查程序所需的所有调式符号，以向你显示问题发生的位置。

用这个选项来重建，并通过gdb触发崩溃，现在你可以想象下面这样的处理：

```shell
#0 0x1234567 php_myext_find_delimiter(str=0x1234567 "foo@#(FHVN)@\x98\xE0...", strlen=3, tsrm_ls=0x1234567)
p = strchr(str, ',');
```

突然，原因很明显。str字符串不是NULL结尾的字符串，最后由垃圾证明，但是使用非二进制安全函数。下面的strchr()实现尝试扫描str的分配内存的末尾，并进入它没有的区域，导致段错误。使用memchr()和strlen参数的快速替换将防止崩溃。

```shell
enable-maintainer-zts
```

第二个 ./configure选项强制PHP使用线程安全资源管理器（TSRM）/Zend线程安全（ZTS）层启用。这个开关会增加 复杂性和处理时间，但是为了开发的目的，你会发现这是一件好事。对于为什么是ZTS的详细描述，以及为什么要打开它，请参阅第一章。

```shell
enable-embed
```

最后一个重要的./configure开关只有在你将PHP嵌入 到另一个应用程序时才是必需的。这个选项表示libphp5.so应该被构建为选定的SAPI，就像使用-apxs构建mod_php5.so将PHP嵌入到Apache一样。

