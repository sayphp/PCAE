# 引用返回值

使用返回结构从函数返回值和变量引用是很好，但有时你想从函数返回多个值。你可以使用一个数组来完成这个任务，我们将在第8章“使用数组和哈希表”中进行探讨，也可以通过参数栈返回值。

## 调用时传引用

通过引用传递变量的更简单的方法之一是通过要求调用作用域在参数（如下面的一段PHP代码）中包含&符号：

```php
function sample_byref_calltime($a){
	$a .= ' (modified by ref!)';
}
$foo = 'I am a string';
sample_byref_calltime(&$foo);
echo $foo;
```

放置在参数调用中的 &符号导致\$foo使用的实际zval而不是其内容的副本被发送到该函数。者允许该函数直接修改值，并通过其传递的参数有效地返回信息。如果sample_byref_calltime()没有被放在\$foo之前，那么函数内部的变化不会影响原来的变量。

在C中重复这个不需要特别努力。在sample.c源文件中的sample_long()之后创建以下函数：

```c
PHP_FUNCTION(sample_byref_calltime){
	zval *a;
  	int addtl_len = sizeof(" (modified by ref!)") - 1;
  	if(zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "z", &a) == FAILURE){
    	RETURN_NULL();
  	}
  	if(!a->is_ref){
    	/* 参数不是按引用传
    	 * 直接返回
    	 */
      	return;
  	}
  	//*保证变量是字符串
  	convert_to_string(a);
  	//*放大a的缓冲区以保存附加数据
  	Z_STRVAL_P(a) = erealloc(Z_STRVAL_P(a), Z_STRLEN_P(a) + addtl_len + 1);
  	memcpy(Z_STRVAL_P(a) + Z_STRLEN_P(a), " (modified by ref!)", addtl_len + 1);
  	Z_STRLEN_P(a) += addtl_len;
}
```

一如既往，这个函数需要被添加到php_sample_functions结构中：

```c
PHP_FE(sample_byref_calltime, NULL)
```

## 编译时传引用

通过引用传递的更常见的方法是使用编译时传递参考。在这里，函数的参数被声明为传引用，并尝试传递常量或中间值，例如由于函数调用的结果而导致错误，因为没有函数将结果值存回。PHP的编译时传递函数可能如下所示：

```php
function sample_byref_compiletime(&$a){
  	$a .= ' (modified by ref!)';
}
$foo = 'I am a string';
sample_byref_compiletime($foo);
echo $foo;
```

正如你所看到的，仅在引用&符号的位置上，这与calltime版本有所不同。在C中查看这个函数时，功能代码的实现是完全一样的。唯一真正的区别在于它是如何在php_sample_functions块中声明的：

```c
PHP_FE(sample_byref_compiletime, php_sample_byref_arginfo)
```

其中php_sample_byref_arginfo是一个（明确命名）常量结构，您将明确需要定义在这个入口将编译。

> **注意**
>
> 对is_ref的检查实际上可能会被排除在编译时版本之外，因为它永远不会退出，但现在就不会造成任何伤害。

在Zend引擎1（PHP4）中，这将是一个简单的char \*列表，它由一个长度字节组成，后面依次是一组与各个函数参数相关的标志。

```c
static unsigned char php_sample_byref_arginfo[] = {1, BYREF_FORCE};
```

在这里，1表示向量只包含一个参数的参数信息。然后在随后的元素中跟随参数特定的参数信息，第一个参数在第二个元素中，如图所示。如果涉及第二或第三个论点，他们的旗帜将分别在第三和第四个 元素中去掉，等等。表6.2给出了给定参数元素的可能值。

| 引用类型             | 含义                                       |
| ---------------- | ---------------------------------------- |
| BYREF_NONE       | 这个参数永远不允许传递引用。试图使用调用时传引用将被忽略，参数将被复制。     |
| BYREF_FORCE      | 无论如何调用函数，参数总是通过引用传递。这项当与在PHP函数声明中使用&符号。  |
| BYREF_ALLOW      | 通过引用传递的参数由调用时语义决定。这相当于普通的PHP函数声明。        |
| BYREF_FORCE_REST | 当前的参数和所有后续的参数将应用BYREF_FORCE_REST之后放置其他标志将导致未定义的行为。 |

在Zend引擎2（PHP5+）中，你将使用更广泛的结构，其中包含最小和最大参数要求，类型提示以及是否强制引用等信息。

首先使用两个宏之一来声明arg info结构体。简单的宏ZEND_BEGIN_ARG_INFO_EX()需要两个参数：

```c
ZEND_BEGIN_ARG_INFO_EX(name, pass_rest_by_reference, return_reference, required_num_args)
```

name和pass_rest_by_reference有相同的意思。正如你在本章前面看到的return_reference给Zend提示你的函数将会用你自己的zval重写return_value_ptr。

最后一个参数required_num_args是Zend的另一个捷径，当该函数的原型被认为与它被调用的方式不兼容时，它允许完全跳过某些函数调用。

在你有一个合适的开始宏之后，它可以跟零或多个ZEND_ARG\_\*INFO元素。这些宏的类型和用法如下表所示。

| 参数信息宏                                    | 目的                                       |
| ---------------------------------------- | ---------------------------------------- |
| ZEND_ARG_PASS_INFO(by_ref)               | 所有后续宏的by_ref是一个二进制指示是否强制相应的参数作为通过引用。将此选项设置为1等同于在Zend引擎1矢量中使用BYREF_FORCE。 |
| ZEND_ARG_INFO(by_ref, name)              | 该宏提供了由内部生成的错误消息和反射API使用的附加名称属性。它应该被设置为有用和非神秘的东西。 |
| ZEND_ARG_ARRAY_INFO(by_ref, name, allow_null) ZEND_ARG_OBJ_INFO(by_ref, name, classname, allow_null) | 这两个宏为内部函数提供参数类型提示，指定数组或特定类实例作为参数。将allow_null设置为非零值将允许调用作用域传递NULL值来代替数组/对象。 |

最后，使用Zend引擎2的宏所有arg信息结构必须使用ZEND_END_ARG_INFO()来终止它们的列表。对于你的实例函数你可以选择如下所示的最终结构：

```c
ZEND_BEGIN_ARG_INFO(php_sample_byref_arginfo, 0)
	ZEND_ARG_PASS_INFO(1)
ZEND_END_ARG_INFO()
```

为了使扩展与Zend引擎1、2兼容，在这种情况下，必须使用\#ifdef语句并为两者定义相同的arg_info结构：

```c
#ifdef ZEND_ENGINE_2
static
  	ZEND_BEGIN_ARG_INFO(php_sample_byref_arginfo, 0)
  		ZEND_ARG_PASS_INFO(1)
  	ZEND_END_ARG_INFO()
#else /* Zend引擎 1*/
static unsigned char php_sample_byref_arginfo[] = {1, BYREF_FORCE};
#endif
```

现在所有的部分都汇聚在一起了，现在是时候创建一个实际的编译时传递引用实现。首先让我们把用于Zend引擎1、2的php_sample_byref_arginfo块定义到头文件php_sample.h中。

接下来你可以采取两种方法：

一种是复制粘贴PHP_FUNCTION(sample_byref_calltime)实现，并将其重命名为PHP_FUNCTION(sample_byref_compiletime)，然后将PHP_FE(sample_byref_compiletime, php_sample_byref_arginfo)行添加到php_sample_functions。

这种方法很简单，在从现在开始进行修改时可能不太容易混淆。

因为这只是示例代码，所以你可以使用PHP_FALIAS()函数稍微松散一点，避免代码重复，这是你在上一章看到的。

这一次，不是复制PHP_FUNCTION(sample_byref_calltime)，添加一行到php_sample_functions：

```c
PHP_FALIAS(sample_byref_compiletime, sample_byref_calltime, php_sample_byref_arginfo)
```

从第5章可以知道，这将创建一个名为sample_byref_compiletime()的PHP函数，其内部实现使用sample_byref_calltime()的代码。php_sample_byref_arginfo的添加使得这个版本是唯一的。

