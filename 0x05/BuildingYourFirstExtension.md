# 构建你的第一个扩展

现在，你已经知道了构建扩展所需要的所有基础文件了，是时候展现真正的技术了。与构建主要的PHP二进制文件一样，根据你是针对*nix还是针对Windows进行编译，可以采取不同的步骤。

## 在*nix下构建

第一步是使用config.m4中的信息作为模板生成一个./configure脚本。这可以通过在编译主PHP二进制文件时运行安装的phpize程序来完成。

```shell
$ phpize -v
PHP Api Version：20041225
Zend Module Api No：20050617
Zend Extension Api No: 220050617
```

> **注意**
>
> Zend Exntesion Api No开头的额外2不是拼写错误；它对应于Zend Engine 2的版本，并且意味着保持这个API号码大于它的ZE1对应的号码。
>
> 目前PHP 7的Zend  Extension Api No：320151012

如果你在当前目录中查看，你会注意到把以前更多的文件。phpize程序将扩展的config.m4文件中的信息与从PHP构建中收集的数据相结合，并列出了进行编译所需的所有部分。这意味着你不必与makefile和找到你要编译的PHP头文件争执。PHP已经为你完成了这项工作。

下一步是一个简单的./configure，你可以用任何其他的OSS包来执行。你不是在这里配置整个PHP包，只是你的一个扩展，所以你只需要输入以下内容：

```shell
$ ./configure enable-sample
```

注意，这里也没有使用enable-debug和enable-maintainer-zts。这是因为phpize已经从主要的PHP构建中取得这些值，并将它们应用到了扩展的./configure脚本中。

现在建立扩展，就像其他任何软件包一样。你只需要输入make，生成的脚本文件就可以处理其余的内容。

当构建过程完成后，你将看到一条消息，指出sample.so已被编译并放置在当前构建目录中名为”modules“的目录中。

## 在Windows下构建

先前创建的config.m4文件实际上是特定于\*nix版本。为了使你的扩展在Windows下编译，你需要为它创建一个单独但相似的配置文件。

将以下内容的config.w32添加到你的ext/sample目录中：

```autoconf
ARG_ENABLE("sample", "enable sample extension", "no");
if(PHP_sAMPLE != "no"){
  EXTENSION("sample", "sample.c");
}
```

正如你所看到的，这个文件在config.m4上有着很高的相似性。该选项被声明，测试并有条件地用于启用扩展的构建。

现在，当你构建PHP内核时，你将重复第4章”设置构建环境“中执行的一些步骤。首先，从”开始“菜单中选择所有程序，Windows Server 2003 SP1的Microsoft Platform SDK，打开生成环境窗口，Windows 2000生成环境，设置Windows 2000生成环境（调试），然后运行C:\Program Files\Microsoft Visual Studio 8\VC\bin\vcvars32.bat批处理文件。

请记住，你的安装可能需要你选择不同的构建目标或运行稍有不同的批处理文件。请参阅第4章中的注意事项来刷新记忆。

此外，你要想到你的编译目录的根目录并重建配置脚本。

```shell
C:\Program Files\Microsoft Platform SDK > cd \PHPDEV\php-5.1.0
C:\PHPDEV\php-5.1.0 > buildconf.bat
Rebuilding configure.js
Now run 'cscript/nologo configure.js help'
```

这一次，你将使用一组缩减的选项运行配置脚本。因为你只关注你的扩展，而不是整个PHP，你可以省略其他扩展的选项；然而，与Unix版本不同的是，即使核心版本已经包含enable-debug开关，你也许要明确地包含enable-debug开关。

这里唯一的关键的开关是enable-sample=shared。共享选项在这里是必需的，因为configure.js不知道你打算把样本作为一个可载入的扩展。因此你的配置应该是这样的：

```shell
C:\PHPDEV\php-5.1.0 > cscript /nologo configure.js enable-debug enable-sample=shared
```

> **注意**
>
> 回想一下，在这里不需要enable-maintainer-zts，因为所有的Win32版本都假定必须启用ZTS。由于SAPI层独立于扩展层，所以在此不需要涉及SAPI的嵌入式选项。

最后，你准备建立扩展。因为这个版本是基于核心的，而不是基于扩展名的Unix扩展版本，所以你需要在你的build line中指定目标名称。

```shell
C:\PHPDEV\php-5.1.0 > nmake php_sample.dll
```

一旦编译完成，你应该有一个工作php_sample.dll二进制文件准备在下一步使用。

请记住，因为本书要关注*nix开发，所以下面的所有文本中的扩展名将被称为sample.so而不是php_sample.dll。

## 加载构建为共享模块的扩展

为了让PHP在请求时找到这个模块，它需要位于php.ini设置中指定的目录下：exntension_dir。默认情况下，php.ini位于/usr/local/lib/php.ini；然而，这个默认值可以改变，而且通常是配送包装系统。检查php -i的输出以查看PHP的位置位置寻找你的配置文件。

这个设置，在一个未经修改的php.ini中，是一个无益的./。如果你还没有加载扩展，或者只是没有任何扩展，除了sample.so之外，你可以改变这个值到放置你的模块的位置。否则，只需将sample.so复制到此设置指向的目录即可。

extension_dir指向正确的地方后，有两种方法可以告诉PHP加载你的模块。首先是在脚本中使用dl()函数：

```php
<?php
	dl('sample.so');
	var_dump(get_loaded_modules());
```

如果这个脚本没有显示样本作为一个加载模块，出了什么问题。在输出的上方寻找错误消息，或者在你的php.ini文件中定义error_log。第二种方法是使用扩展指令在php.ini中指定模块。

扩展设置在php.ini设置中是相对唯一的，因为它可以用不同的值指定多次。所以如果你的php.ini中已经有了扩展设置，不要把它像分割列表一样添加到同一行；而是插入一个只包含sample.so的附加行。在这一点上你的php.ini应该是这样的：

```ini
extension_dir=/usr/local/lib/php/modules/
extension=sample.so
```

现在你可以运行相同的脚本，而不需要dl()行，或者只需发出php -m命令，仍然可以在家在的模块列表中看到“sample”。

> **注意**
>
> 本章和后面章节中的所有示例代码将假定你已经使用此方法加载了当前扩展。如果你打算使用dl()，请务必将相应的载入线添加到实例脚本中。

