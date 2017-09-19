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

##即使你不需要线程

## 找寻迷失的tsrm_ls

