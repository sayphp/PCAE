# 启动和关闭

在启动和关闭过程中发生在两个单独的启动阶段和两个单独的关闭阶段。第一个是PHP解释器作为一个整体执行和初始设置结构和值，将持续循环存在于SAPI的生命周期中；第二个是短暂到只能持续一个页面的请求。

在初始启动期间，在任何请求完成之前，PHP会调用每个扩展的minit（模块初始化）方法。在这里，扩展被预期用做声明常量、定义类、注册资源、流和过滤处理。这些被设计为存在于所有请求中的特征被称为持久性。

常见的minit方法可能如下所示

```C
/*初始化扩展模块
 *当SAPI启动是立刻执行
 *PS:这里我使用的是PHP-7.2的standard中的file
 *位于/ext/standard/file.c
 */
PHP_MINIT_FUNCTION(file)
{
  	le_stream_context = zend_register_list_destructors_ex(file_context_dtor, NULL, "stream-context", module_number);
#ifdef ZTS
  	ts_allocate_id(&file_globals_id, sizeof(php_file_globals), (ts_allocate_ctor) file_globals_ctor, (ts_allocate_dtor) file_globals_dtor);
#else
  	file_globals_ctor(&file_globals);
#endif
  	//*最简单的情况，是直接返回SUCCESS的
    return SUCCESS;
}
```

