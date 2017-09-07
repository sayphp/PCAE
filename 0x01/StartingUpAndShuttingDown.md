# 启动和关闭

在启动和关闭过程中发生在两个单独的启动阶段和两个单独的关闭阶段。第一个是PHP解释器作为一个整体执行和初始设置结构和值，将持续循环存在于SAPI的生命周期中；第二个是短暂到只能持续一个页面的请求。

在初始启动期间，在任何请求完成之前，PHP会调用每个扩展的minit（模块初始化）方法。在这里，扩展被预期用做声明常量、定义类、注册资源、流和过滤处理。这些被设计为存在于所有请求中的特征被称为持久性。

常见的minit方法可能如下所示

```C
/* 初始化扩展模块
 * 当SAPI启动是立刻执行
 * PS:这里我使用的是PHP-7.2的standard中的file
 * 位于/ext/standard/file.c
 */
PHP_MINIT_FUNCTION(file)
{
  	le_stream_context = zend_register_list_destructors_ex(file_context_dtor, NULL, "stream-context", module_number);
#ifdef ZTS
  	ts_allocate_id(&file_globals_id, sizeof(php_file_globals), (ts_allocate_ctor) file_globals_ctor, (ts_allocate_dtor) file_globals_dtor);
#else
  	file_globals_ctor(&file_globals);
#endif
  	REGISTER_INI_ENTRIES();
  	REGISTER_LONG_CONSTANT("SEEK_SET", SEEK_SET, CONST_CS | CONST_PERSISTENT);
  	//……省略一大堆注册常量代码
  	//*最简单的情况，是直接返回SUCCESS的
    return SUCCESS;
}
```

在请求之后，PHP设置了一个包含符号表（存储变量）的操作环境，并同步每个目录的配置值，然后PHP再次循环便利其扩展，调用每一个扩展的RINIT（请求初始化）方法。这里，扩展可以重置全局变量为默认值，江边量预先填充到脚本的符号表中，或执行其他任务，例如将页面请求记录到文件中。RINIT可以被认为是所需脚本的一种自动前置指令。（auto_prepend_file）

> 符号表，symbol table。一种映射关系，具体实现常用Hash表。

RINIT方法如下图所示

```c
/* 在每个页面请求的开始运行
 * standard/file.c并没有找到RINIT，所以使用了自己的扩展	
 */
PHP_RINIT_FUNCTION(say)
{
#if defined(COMPILE_DL_SAY) && defined(ZTS)
  	ZEND_TSRMLS_CACHE_UPDATE();
#endif
  	return SUCCESS;
}
```

在请求完成处理之后，当到达脚本文件的末尾（EOL）或通过die()或exit()语句，PHP通过调用每个扩展的RSHUTDOWN（请求关闭）方法来地动清除进程。RSHUTDOWN对应与auto_append_file与RINIT育婴与auto_prepend_file相同。然而，两者最主要的区别在于rshutdown总是被执行，期中在PHP语言中调用die()或exit()将跳过任何auto_append_file在。

在符号表和其他资源被销毁前，所有的任务都可以在RSHUTDOWN中处理。在所有RSHUTDOWN方法完成后，符号表中的每个变量都相当于进行了隐式的*unset()*，在此期间调用的所有飞驰就行资源和对象析构函数才能正常释放。

```c
/* 在每个页面请求结束时运行
 * PS:使用了自己定义的扩展
 */
PHP_RSHUTDOWN_FUNCTION(say)
{
    return SUCCESS;
}
```

最后，当所有请求都已满足并且Web服务器或其他SAPI已经准备好关闭时，PHP将循环便利每个扩展的MSHUTDOWN（模块关闭）方法。这是一个扩展最后一次取消注册处理程序并释放在MINIT周期内分配的持久内存的机会。

```c
/* 该模块正在卸载
 * 常熟和韩数将会自动清除，持续资源，类条目和流处理必需后动取消注册
 * 由于自己的扩展依旧只是默认return，所以把书中的源码扔过来
 */
PHP_MSHUTDOWN_FUNCTION(myextnsion)
{
    UNREGISTER_INI_ENtRIES();
  	php_unregister_url_stream_wrapper("myproto", TSRMLS_CC);
  	php_stream_filter_unregister_factory("myfilter", TSRMLS_CC);
  	return SUCCESS;
}
```

