# 功能函数

PHP语言代码和扩展代码之间最快的链接是PHP_FUNCTION()。首先在 \#include "php_sample.h"后面的sample.c文件的顶部附近添加以下代码块：

```c
PHP_FUNCTION(sample_hello_world){
	php_printf("Hello World!\n");
}
```

PHP_FUNCTION()宏的功能就像普通的C函数声明一样，因为这正是它扩展的方式：

```c
#define PHP_FUNCTION(name) void zif_##name(INTERNAL_FUNCTION_PARAMETERS)
```

在这种情况下，评估结果如下：

```c
void zif_sample_hello_world(zval *return_value, char return_value_used, zval *this_ptr TSRMLS_DC)
```

当然，简单地声明这个函数是不够的。引擎需要知道函数的地址以及如何将函数名称导出到用户空间。这是通过下一个代码块完成的，你可以在PHP_FUNCTION()块之后立即放置该代码块：

```c
static function_entry php_sample_functions[] = {
	PHP_FE(sample_hello_world, NULL);
  	{NULL, NULL, NULL}
};
```

从而为新功能提供了一个名称，并提供了一个指向其实现功能的指针。这个集合的第三个参数用于提供参数提示信息，例如要求某些参数通过引用传递。你将在第7章“接受参数”中看到此功能。

所以现在你已经有了一个可导出函数的列表，但是还没有把它连接到引擎。这是通过对sample.c的最后一个改变来完成的，它等于简单地用php_sample_functions替换sample_module_entry结构中的NULL，/\*FUNCTIONS\*/行（确保在那里保留逗号！）

现在按照前面的说明重新编译，并使用PHP命令行的-r选项进行测试，这样就可以运行简单的代码片段，而无需创建整个文件：

```shell
$ php -r 'sample_hello_world();'
```

如果一切顺利，你会看到“Hello World！”这个词。几乎立即输出。

## Zend内部函数

前缀为内部函数名称的zif_string表示“Zend内部函数”，用于避免可能的符号冲突。例如，用户空间strlen()函数无法实现为void strlen(INTERNAL_FUNCTION_PARAMTERS)，因为它会与C库的strlen实现冲突。

有时甚至zif_simply的默认前缀根本就不会。通常这是因为函数名扩展了另一个宏，并被C编译器误解。在这些情况下，可以使用PHP_NAMED_FUNCTION()宏为内部函数赋予任意名称；例如，PHP_NAMED_FUNCTION(zif_sample_hello_world)与之前使用的PHP_FUNCTION(sample_hello_world)相同。

添加使用PHP_NAMED_FUNCTION()声明的实现时，将使用PHP_NAMED_FE()宏将其链接到function_entry向量中。所以，如果你声明你的函数为PHP_NAMED_FUNCTION(purplefunc)，你可以使用PHP_NAMED_FE(sample_hello_world, purplefunc, NULL)，而不是使用PHP_FE(sample_hello_world, NULL)。

这种做法可以在ext/standard/file.c中看到，其中fopen()函数实际上是用PHP_NAMED_FUNCTION(php_if_fopen)声明的。就用户空间而言，通常没有什么功能。它仍然被称为fopen()。但是，在内部函数不受预处理器宏和过度编译器的影响。