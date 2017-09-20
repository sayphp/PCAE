# Zend线程安全

在PHP诞生之初，只是作为单进程CGI运行的，并不关心线程安全性，因为没有进程控件可以超过单个请求。内部变量可以在全据范围内被声明，只要其内容正确的初始化了，就可以随意访问或更改内部变量。任何没有被正确清理的资源都会在CGI进程终止时释放。

之后，PHP被嵌入到多进程Web服务器（如apache）中。一个给定的内部变量让然可以被全局定义并被请求安全访问，只要他每个请求开始时被正确的初始化，并在最后清理，因为每个进程控件的一个请求可以被激活一次。在这一点上，每个请求都被添加了内存管理，以防止资源泄漏造成程序失控。

随着单进程多线程Web服务器开始黯然失色，需要一中处理全局数据的新方法。最终，这将成为一个名为TSRM（线陈甘泉资源管理器）的新层。

## 线程安全与无线程安全的声明

在一个简单的单线程应用程序中，你很可能通过将它们放置在源文件的顶部来声明全局变量。而后，编译器将在程序的数据段中分配一个内存块以保存该单元的信息。

在多线程的应用程序中，每个线程都需要有自己的数据源素版本，因此必需为每个线程分配一个单独的内存块。然后，给定的线程在需要访问其数据的时候，选择正确的内存块和指针引用。

## 线程安全数据池

在一个扩展的MINIT阶段，扩展使用*ts_allocate_id()*函数通过TSRM层来存储一个或多个调用需要的数据。TSRM将该字节计数添加到其运行的数据空间需求总数，并为该段的线程数据池部分返回一个新的唯一标识符。

```c
typedef struct{
    int sampleint;
  	char *samplestring;
} php_sample_globals_id;
PHP_MINIT_FUNCTION(sample)
{
    ts_allocate_id(&sample_globals_id,
                  sizeof(php_sample_globals),
                  (ts_allocate_ctor) php_sample_globals_ctor,
                  (ts_allocate_dtor) php_sample_globals_dtor);
  	return SUCCESS;
}
```

当灾情求其减访问该数据段时，该扩展从TSRM层为当前线程的资源池请求一个指针，由*ts_allocate_id()*所返回的资源ID通过适当的索引将该指针进行了偏移。

换句话说，下面代码你可能在之前的MINIT语句关联的模块中看到。

```c
SAMPLE_G(sampleint) = 5;
```

在线程安全的构建环境下，此声明通过一些中间宏扩展到以下内容：

```c
(((php_sample_globals*)(*((void ***)tsrm_ls))[sample_globals_id-1])->sampleint = 5;
```

如果并不能很好的理解这段代码的内容，也不需要担心；这些都被很好的集成到了PHPAPI中，一般的开发人员也从来没有想过了解它是如何工作的。

## 什么时候不用线程

因为在线程安全的PHP构建中访问全局资源设计将正确的偏移量查找到正确的数据池中的开销，所以它的结果比起非线程的要慢，期中数据简单地从真正的全局地址在编译时计算。

再次考虑先前的例子，这次在一个非线程构建代码如下：

```c
typedef struct {
    int sampleint;
  	char * samplestring;
} php_sample_globals;
php_sample_globals sample_globals;
PHP_MINIT_FUNCTION(sample)
{
    php_sample_globals_ctor(&sample_globals TSRMLS_CC);
  	return SUCCESS;
}
```

你在这里的第一件事，不是声明一个int来标识对其他地方生命的全局变量结构的引用，而是简单地再进承德全局范围内定义结构。这意味着*SAMPLE_G(sampleint) = 5;*之前的语句只需要扩展为*sample_globals.sampleint = 5;*将会变得更加简单、快速、高效。

非线程构建还具有进程隔离的优点，以便如果给定的请求遇到完全以外的情况，择它可以将所有方式保证甚至是段错误，而不会使整个Web服务器瘫痪。事实上，Apache的MaxRequestsPerChild 指令旨在通过故意杀死其子进程，并在其他地方新开子进程加以利用。

## 未知全局访问

在创建扩展时，你不一定知道他所构建的环境是否需要线程安全。幸运的是，你将使用的标准include文件有条件的定义了ZTS预处理器令牌。当PHP构建为线程安全性时，由于SAPI需要它，或者通过*--enable-maintainer-zts*选项，该值被自动定义，并且可以使用通常的一组指（如*#ifdef ZTS*）令进行测试。

如前所述，如果线程池实际存在，择在线程安全池中分配控件是有一医德，并且只有在为线程安全编译PHP时，它才会存在。这就是为什么在前面的例子中，它包含在检查*ZTS*中，一个非线程的替代方法被调用于非*ZTS*的构建环境中。

在本章前面看到的*PHP_MINIT_FUNcTION(myextension)*的示例中，*#ifdef ZTS*用于有条件地调用争取版本的全局初始化代码。对于ZTS模式，它使用*ts_allocate_id()*来填充*myextension_globals_id*变量，而non-ZTS模式 直接成为*myextension_globals*的初始化方法。这两个变量将在你的扩展源文件中使用Zend宏命令声明：*DECLARE_MODULE_GLOBALS(extension);*它会根据ZTS是否启用自动处理ZTS测试并声明适当类型的正确主机变量。

当访问者写全局变量的时候，你将使用如前面所示的自定义宏，如SAMPLE_G()。在第12章中，你将虚吸入和设计此宏以根据是否启用ZTS来展开为正确的形式。

##即使你不需要线程

正常的PHP构建默认情况下是关闭线程安全的，只有当已知构建的SAPI需要线程安全或者有*./configure* 开关明确的打开线程安全设置时，才启用它。

考虑到全局查找的速度问题，以及缺少进程隔离，你可能会想知道为什么任何人在不需要时回顾已将TSRM层打开。在大多数情况下，它是扩展和SAPI开发人员，就像你即将成为——为了确保新代码在所有环境中都能正常运行。

当启用线程安全性时，将一个特殊的指针（称为*tsrm_ls*）添加到许多内部函数的原型中。它是这个指针，允许PHP将与一个线程相关联的数据与另一个线程区分开来。你可能会记得在本章前面的ZTS模式下看到它与*SAMPLE_G()*宏一起使用。没有它，执行函数将不知道要查训的符号表并设置一个对应的值；它甚至不知道哪个脚本被执行，引擎将完全无法跟踪其内部寄存器。着一个指针保持一个线程处理页面请求正在另一个的顶部运行。

指针参数可选的包含在原型中的方式是通过一组定义。当ZTS被禁用时，这些定义将所有求值设置为空白；然而，当他被打开时，它们看起来向下面这样：

```c
#define TSRMLS_D void ***tsrm_ls
#define TSRMLS_DC , void ***tsrm_ls
#define TSRMLS_C tsrm_ls
#define TSRMLS_CC , tsrm_ls
```

non-ZTS构建将在下面的代码中看到第一行具有两个参数，即int和char *。另一方面，在zts构建下，原型包含三个参数：int， char *和void ***。当你的程序调用此函数时，它将需要传入该参数，但仅适用于启用了ZTS的构建。一下代码中的第二行显示了CC宏如何完成这一点。

```c
int php_myext_action(int action_id, char *message TSRMLS_DC);
php_myext_action(42, "The meaning of life", TSRMLS_CC);
```

通过在函数调用中包含着个特殊变量，*php_myext_action*将能够使用*tsrm_ls*的值与*MYEXT_G()*宏一起访问其线程特定的全局数据。在non-ZTS构建中，*tsrm_ls*将不可用，但是没关系，因为*MYEXT_G()*和其他类似的宏将不会使用它。

现在想象你正在开发一个新的扩展，你已经有了以下的功能，使用CLI SAPI在你的本地构建下工作的很好，甚至使用Apache1的apxes SAPI。

```c
static int php_myext_isset(char *varname, int varname_len)
{
    zval **dummy;
  	if(zend_hash_find(EG(active_symbol_table),
                     varname, varname_len + 1,
                     (void**)&dummy) == SUCCESS){
        /* Variable exists */
      	return 1;
    }else{
        /* Undefined variable */
      	return 0;
    }
}
```

满意就好，你打包你的扩展，并发送到另一个办公室，在生产服务器上建立和运行。令你感到沮丧的是，远程办公室告诉你扩展无法编译。

事实证明，他们在线程模式下使用Apache 2.0，所以他们的PHP版本启用了ZTS。当编译遇到你使用*EG()*宏时，它尝试在本地 范围内找到tsrm_ls，因为你没有声明它，并且从未将它传递给你的函数。

当然这个修复很简单；只需将TSRMLS_DC添加到*php_myext_isset*()的声明中，并将TSRMLS_CC抛出到调用它的每一行上。不幸的是，远程办公室的生产流水线现在已经不再赘述，因此希望推出几个星期的时间。如果只是这个问题可能早已被抓住了！

这就是 --enable-maintainer-zts。在构建PHP时，通过将这一行添加到*./configure*语句中，即使你当前的sapi（如cli）不需要，你的够将也将自动包含ZTS。启用次开关，可以避免这种常见且不必要的编程错误。

> **注意**
>
> 在PHP4中， --enable-maintainer-zts标志被写做--enable-experimental-zts；在编译你的PHP的时候，一定要针对版本使用正确的标志。

## 找寻迷失的tsrm_ls

有些时候，不可能将tsrm_ls指针传递到需要它的函数中。这通常是因为你的扩展是使用某个库的接口，并且不给要返回的抽象指针提供控件。可以参考下面的代码：

```c
void php_myext_event_callback(int eventtype, char *message)
{
    zval *event;
  	/* $event = array('event' => $eventtype, 'message' => $message); */
  	MAKE_STD_ZVAL(event);
  	array_init(event);
  	add_assoc_long(event, "type", eventtype);
  	add_assoc_string(event, "message", message, 1);
  	/* $eventlog[] = $event; */
  	add_next_index_zval(EXT_G(eventlog), event);
}
PHP_FUNCTION(myext_startloop)
{
    /* Theeventlib_loopme() function,
     * exported by an external library,
     * waits for an event to happen,
     * then disatches it to the
     * callback handler specified.
     */
  	eventlib_llome(php_myext_event_callback);
}
```

尽管不是所有这些代码段都有意义，但是你马上会注意到回调函数使用*EXT_G()*宏，该宏已知需要在线程构建下的tsrm_ls指针。改变函数原型将不会很好，因为外部库没有PHP的线程安全模型的概念（也不应该有）。那么tsrm_ls怎么能以这样的方式被恢复呢？

这个解决方案采用名为*TSRMLS_FETCH()*的Zend宏的形式。当放置在代码段的顶部时，该宏将基于当前的线程上下文执行查找，并声明tsrm_ls指针的本地副本。

尽管在任何地方使用这个宏都是很诱人的，并且不用大绕通过函数调用传递tsrm_ls，但中要的是要注意，*TSRMLS_FETCH()*调用需要相当多的处理时间才能完成。当然，在弹磁碟袋中不可察觉，但是你的线程技术部端增加，并且你调用*TSRMLS_FETCH()*的实例数量增长，你的扩展将逐渐开始显示此瓶颈。一定要谨慎使用。

> **注意：**
>
> 为了确保与C++编译器的兼容性，请确保在任何与据之前，将*TSRMLS_FETCH()*和该事项的所有变量声明放在给定块范围的顶部。因为*TSRMLS_FETCH()*宏本身可以通过集中不同的方式解决，最好使它成为在给定声明中声明的最后一个变量。





