# 使用zend_parse_parameters()自动进行类型转换

和上一章所看到的返回值一样，参数值使用间接的zval引用移动。获取 这些zval \*值的最简单方法是使用zend_parse_parameters()函数。

对zend_parse_parameters()的调用几乎总是从ZEND_NUM_ARGS()宏开始，后面是无处不在的TSRMLS_CC。顾名思义，ZEND_NUM_ARGS()返回一个int值，表示实际传递给函数的参数个数。由于zend_parse_parameters()在内部工作的方式，你可能永远不需要直接检查这个值，所以现在只需要传递它。

zend_parse_parameters()的下一个参数是格式参数，它由一系列字符或字符序列组成，对应Zend引擎支持的各种基本类型。下表显示了基本类型字符。

| 类型说明符 | PHP数据类型      |
| ----- | ------------ |
| b     | 布尔型          |
| l     | 整型           |
| d     | 浮点型          |
| s     | 字符串          |
| r     | 资源           |
| a     | 数组           |
| o     | 对象实例         |
| O     | 指定类型的对象实例    |
| z     | 非特定的zval     |
| Z     | 解除引用的非特定zval |

zend_parse_parameters()的其余参数取决于你在格式字符串中请求的具体类型。对于更简单的类型，这是一个取消引用的C语言语法。例如，一个长数据类型像这样被提取：

```c
PHP_FUNCTION(sample_getlong){
  	long foo;
  	if(zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "l", &foo) == FAILURE){
    	RETURN_NULL();
  	}
  	php_printf("The integer value of the parameter you passed is %ld", foo);
  	RETURN_TRUE;
}
```

尽管整数和长整数具有相同的数据存储大小是常见的，但它们不能总是互换使用。尝试将int数据类型解引用为long \*参数可能会导致意外的结果，特别是在64位平台变得更普遍的情况下。始终使用下表列出的适当的数据类型。

| 类型说明符 | C数据类型                       |
| ----- | --------------------------- |
| b     | zend_bool                   |
| l     | long                        |
| d     | double                      |
| s     | char \*, int                |
| r     | zval \*                     |
| a     | zval \*                     |
| o     | zval \*                     |
| O     | zval \*, zend_class_entry\* |
| z     | zval \*                     |
| Z     | zval \*\*                   |

请注意，所有更复杂的数据类型实际上解析为简单的zval。大多数情况下，这是由于使用RETURN_\*()宏组织返回复杂数据类型的相同限制：这些结构实际上没有C空间模拟。zend_parse_parameters()为你的功能做的是确保你所受到的zval \*是适当的类型。如果有必要，它甚至会执行隐式转换，如果数组类型转换为stdClass对象。

s和O类型也值得指出，因为他们需要每个调用的一对参数。当你在第10章“PHP4对像”和第11章“PHP5对象”中探索对象数据类型的，你会更加仔细地看到O。

在s类型的情况下，假设你正在扩展第5章“你的第一个扩展”中sample_hello_world()函数，已通过名称来迎接特定人员：

```php
function sample_hello_world($name){
  	echo "Hello $name!\n";
}
```

在C中，你将使用zend_parse_parameters()函数来请求一个字符串：

```c
PHP_FUNCTION(sample_hello_world){
  	char *name;
  	int name_len;
  	if(zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "s", &name, &name_len) == FAILURE){
    	RETURN_NULL();	
  	}
  	php_printf("Hello ");
  	PHPWRITE(name, name_len);
  	php_printf("!\n");
}
```

> **提示**
>
> zend_parse_parameters()函数可能会失败，因为函数被传递给太少的参数来满足格式字符串，或者因为传递的参数之一根本无法转换为请求的类型。在这种情况下，它会自动输出一个错误信息，所以你的扩展不需要。

要请求多个参数，请扩展格式说明符以包含其他字符，并将后续参数堆叠到zend_parse_parameters()调用上。就像在用户空间函数声明中一样从左向右分析参数：

```php
function sample_hello_world($name, $greeting){
  	echo "Hello $greeting $name!\n";
}
sample_hello_world('John Smith', 'Mr.');
```

或者：

```c
PHP_FUNCTION(sample_hello_world){
  	char *name;
  	int name_len;
  	char *greeting;
  	int greeting_len;
  
  	if(zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "ss", &name, &name_len, &greeting, &greeting_len) == FAILURE){
    	RETURN_NULL();	
  	}
  	php_printf("Hello ");
  	PHPWRITE(greeting, greeting_len);
  	php_printf(" ");
  	PHPWRITE(name, name_len);
  	php_printf("!\n");
}
```

除了类型基元之外，还有三个元字符用于修改参数的处理方式。下表列出了这些修饰符：

| 类型修饰符 | 意义                                       |
| ----- | ---------------------------------------- |
| \|    | 可选参数如下。当这个被指定时，全部以前的参数被认为是必需的和全部后续参数被认为是可选的。 |
| ！     | 如果对于与前面的参数说明符相对应的参数传递一个NULL，则提供的内部变量将被设置为实际的NULL指针，而不是IS_NULL的zval。 |
| /     | 如果前一个参数说明符对应的参数在写时复制引用集中，则会自动将其分隔为一个新的zval，期中is_ref==0，并且refcount==1。 |

## 可选参数

再看一下修改后的sample_hello_world()示例，下一步构建这个步骤可能是使\$greeting参数可选。在PHP中：

```php
function sample_hello_world($name, $greeting='Mr./Ms.'){
  	echo "Hello $greeting $name!\n";
}
```

sample_hello_world()现在可以用两个参数或名称来调用：

```php
sample_hello_world('Ginger Rogers', 'Ms.');
sample_hello_world('Fred Astaire');
```

当没有明确给出默认参数时使用。在C的实现中，可选参数以类似的方式指定。为了达到这个目的，在zend_parse_parameters()的格式字符串中使用管道字符(|)。管道左侧的参数将从调用堆栈中解析出来，而没有提供的右侧任何参数都将保持不变。例如：

```c
PHP_FUNCTION(sample_hello_world){
  	char *name;
  	int name_len;
  	char *greeting = "Mr./Mrs.";
  	int greeting_len = sizeof("Mr./Mrs.") - 1;
  
  	if(zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "s|s", &name, &name_len, &greeting, &greeting_len) == FAILURE){
    	RETURN_NULL();
  	}
  	php_printf("Hello ");
  	PHPWRITE(greeting, greeting_len);
  	php_printf(" ");
  	PHPWRITE(name, name_len);
  	php_printf("!\n");
}
```

由于可选参数没有从其初始值修改，除非它们是作为参数提供的，所以将任何参数初始化为某个默认值都很重要。在大多数情况下，这将是NULL/0,虽然有时上面的默认值是明智的。

## IS_NULL和NULL

每个zval，即使是超简单的IS_NULL数据类型，都占用一定量的内存开销。除此之外，需要一定的时钟周期来分配内存空间，初始化这些值，然后被认为不再有用时最终释放它。

对于许多函数来说，仅仅通过使用NULL承诺书来发现参数被调用范围标记为不重要是没有意义的。幸运的是，zend_parse_parameters()允许将参数标记为“NULL允许”，方法是在其格式说明符后附加一个感叹号。考虑以下两个代码片段，一个使用修饰符，另一个不使用：

```c
……
PHP_FUNCTION(sample_arg_fullnull){
  	zval *val;
  	if(zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "z", &val)==FAILURE){
    	RETURN_NULL();
  	}
  	if(Z_TYPE_P(val)==IS_NULL){
    	val = php_sample_make_defaultval(TSRMLS_C);	
  	}
}
……
PHP_FUNCTION(sample_arg_nullok){
	zval *val;
    if(zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "z!", &val)==FAILURE){
    	RETURN_NULL();
    }
    if(!val){
    	val = php_sample_make_defaultval(TSRMLS_C);
    }
}
……
```

这两个版本实际上没有太多不同的代码区别，虽然前者消耗了更多的处理器时间。一般来说，这个功能不会很有用，但确实可用的。

## 强制分离

当一个变量被传入一个函数时，无论是否被引用，其引用计数几乎总是至少为2；变量本身的一个引用，以及传递给函数的另一个引用。在对zval进行更改之前（如果直接在zval上进行操作），将它与可能包含的任何非引用集分开很重要。

这将是一个单调乏味的任务，它不是/格式说明符，它会自动分离任何写入时引用的变量，以便你的函数可以随心所欲地执行。就像NULL标志一样，这个修饰符在意味着影响的类型之后。也像NULL标志，你不会知道你需要这个功能，直到你真的有用它。

### zend_get_arguments()

如果你碰巧在设计你的代码来处理非常旧的PHP版本，或者你只是一个除了zval \*之外什么都不需要的函数，你可以考虑使用zend_get_parameters()API调用。

zend_get_parameters()调用与起脚新的解析对象在几个关键方面有所不同。首先，它不执行自动类型转换；相反，所有的参数都被提取为原始的zval \*数据类型。zend_get_parameters()最简单的用法可能如下：

```c
PHP_FUNCTION(sample_onearg){
	zval *firstarg;
  	if(zend_get_parameters(ZEND_NUM_ARGS(), 1, &firstarg) == FAILURE){
    	php_error_docref(NULL TSRMLS_CC, E_WARNING, "Expected at least 1 paramter.");
      	RETURN_NULL();
  	}
}
```

其次，从手动应用的错误消息中可以看出，zend_get_parameters()在失败时不会输出错误文本。处理可选参数也很差。具体来说，如果你要求它取四个参数，你最好确定提供了至少四个论据，否则将返回失败。

最后，与parse不同的是，这个特定的get变体会自动分离任何写时复制引用集。如果你仍然想跳过自动分离，你可以使用它的兄弟：zend_get_parameters_ex()。

zend_get_parameters_ex()除了不分开写时复制引用集外，它不同之处在于它返回的zval \*\*指针而不是简单的zval \*。虽然这个区别可能知道你使用你都才会知道你需要，它的使用最终是非常相似的：