# 扩展刨析

实际上，通常还有第二个或第三个配置文件和一个或多个头文件。对于你的第一个扩展来说，你只需要使用这些类型文件中的一个即可。

## 配置文件

首先，在PHP源代码目录的扩展目录ext/下创建一个名为sample的目录。实际上，这个心目录可以放在任何地方，但是为了在本章的后面展示Win32和静态构建选项，我会要求你把它放在这里。

接下来，进入这个目录，创建一个名为config.m4的文件，其内容如下：

```autoconf
PHP_ARG_ENABLE(sample, [Whether to enable the "sample" extension], [	enable-sample	Enable "sample" extension support])

if test $PHP_SAMPLE != "no"; then
	PHP_SUBST(SAMPLE_SHARED_LIBADD)
	PHP_NEW_EXTENSION(sample, sample.c, $ext_shared)
fi
```

在这个极为简单的配置中设置了一个名为enable-sample的./configure选项。PHP_ARG_ENABLE的第二个参数将在./configure进程中显示，因为它到达 这个扩展的配置文件。如果最终用户发出./configurehelp,则第三个参数将显示为可用选项。

> **注意**
>
> 有没有想过，有的扩展使用enable-extname进行配置，而有的则使用with-extname进行配置？在功能上，两者没有区别。然而，实际上，enable-extname是指可以在不需要任何第三方库的情况下启用的功能。而相反，with-extname意味着具备具有这样先决条件的特征。
>
> 现在，你的实例扩展不需要链接到其他库，所以你将使用启用版本。第17章 外”外部库“将介绍如何使用并指示编译器使用额外的CFLAGS和LDFLAGS设置。

如果最终用户使用enable-sample选项调用./configure，那么本地环境变量$PHP_SAMPLE将被设置为yes。PHP_SUBST()是标准的autoconf AC_SUBST()宏的PHP修改版本，并且需要将扩展构建为共享模块。

最后，很重要的是，PHP_NEW_EXTENSION()声明模块并没据必须做为扩展的一部分进行变得所有源文件。如果需要多个文件，他们将一空个作为分隔符在第二个参数中，例如：

```autoconf
PHP_NEW_EXTENSION(sample, sample.c, sample2.c)
```

最后一个参数是PHP_SUBST(SAMPLE_SHARED_LIBADD)命令的对应部分，同样需要设置为共享模块。

## 头文件

在C中开发时，将某些类型的数据分隔到源文件包含的外部头文件中几乎总是有意义的。虽然PHP不需要这样做，但是当模块超出单个源文件的范围时，它就变得简单了。

你将从新的头文件php_sample.h中的以下内容开始：

```c
#ifndef PHP_SAMPLE_H
/* 防止双重包含 */
#define PHP_SAMPLE_H
/* 定义扩展属性 */
#define PHP_SAMPLE_EXTNAME "sample"
#define PHP_SAMPLE_EXTVER "1.0"
/* 在PHP源代码树之外构建时导入配置选项 */
#ifdef HAVE_CONFIG_H
#include "config.h"
#endif
/* 包含PHP标准头 */
#include "php.h"
/* 定义入口点符号
 * Zend将在加载这个模块时使用
 */
extern zend_module_entry sample_module_entry;
#define phpext_sample_ptr &sample_module_entry
#endif /* PHP_SAMPLE_H */
```

这个头文件完成了两个主要任务：如果使用phpize工具构建扩展，那么通过本书的大部分内容将构建它，HAVE_CONFIG_H被定义，config.h也将包含在内。

不管如何编译扩展，它也包含PHP源码树中的php.h。这个头文件随后包含了多个其他头文件，这些头文件遍布PHP源代码，提供对PHPAPI批量的访问。

接下来，有你的扩展使用的zend_module_entry结构被声明为外部的，这样Zend使用dlopen()和dlsym()在使用extension = line 加载这个模块时，Zend_module_entry结构可以被Zend拾取。

这个头文件还包括一些预处理器的定义，很快就会在源文件中使用。

## 源代码

最后，重中之重的，你将在文件sample.c中创建一个简单的扩展框架：

```c
#include "php_sample.h"
zend_modue_entry sample_module_entry = {
#if ZEND_MODULE_API_NO >= 20010901
	STANDARD_MODULE_HEADER,
#endif
  PHP_SAMPLE_EXTNAME,
  NULL, /* FUNCTIONS */
  NULL, /* MINIT*/
  NULL, /* MSHUTDOWN */
  NULL, /* RINIT */
  NULL, /* RSHUTDOWN */
  NULL, /* MINFO */
#if ZEND_MODULE_API_NO >= 20010901
    PHP_SAMPLE_EXTVER,
#endif
  	STANDARD_MODULE_PROPERTIES
};

#ifdef COMPILE_DL_SAMPLE
ZEND_GET_MODULE(sample)
#endif
```

就是这样！这三个文件是创建一个扩展所必需的基础。当然，他没有任何用处，但是这只是一个开始，你将通过本节的其余部分添加功能。首先，让我们来看看发生了什么。

开头的部分很简单：包括刚刚创建的头文件，以及来自源代码树的所有其他PHP核心头文件。

接下来，创建你在头文件中声明的zend_module_entry结构。你会注意到，模块条目的第一个元素是基于当前ZEND_MODULE_API_NO定义的条件。这个API号码大致等同于当前PHP的版本，所以如果你确定你的扩展程序永远不会构建在比这更早的任何版本上，那么你可以完全避开#ifdef行，直接包含STANDARD_MODULE_HEADER元素。

然而，考虑到编译时对程序的影响很小，二进制编码或处理所花费的时间并不多，因此在大多数情况下，最好只保留这个条件。同样适用于该结构前后的版本属性。

此结构中的其他六个元素现在已经初始化为NULL；你可以从这些行旁边的注释中看到提示，告诉他们最终会用到什么。

最后，在代码底部，你会发现每个PHP扩展都有一个简单的元素，它可以被构建为一个共享模块。这个简短的条件简单的添加了一个Zend使用的引用，当你的扩展动态加载。不要担心它做了什么或者做的太多了，只要确保他在附近或下一节将不起作用。