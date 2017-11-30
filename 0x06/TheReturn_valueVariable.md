# *return_value*变量

你可能会试图相信你的内部函数应该返回一个即时的zval，或者更可能为zval分配内存并返回一个zval *，如下面带码所示：

```c
PHP_FUNCTION(sample_long_wrong){
	zval *retval;
  	MAKE_STD_ZVAL(retval);
	ZVAL_LONG(retval, 42);
  	return retval;
}
```

请注意，PHP_FUNCTION()实现没有直接返回任何内容。相反，*return_value*参数直接填充了适当的数据，Zend Engine将在内部函数完成执行后将其处理为值。

提醒一下，在这种情况下，ZVAL_LONG()宏是一组赋值操作的简单包装：

```c
Z_TYPE_P(return_value) = IS_LONG;
Z_LVAL_P(return_value) = 42;
```

或更原始：

```c
return_value->type = IS_LONG;
return_value->value.lval = 42;
```

> **注意**
>
> return_value变量的is_ref和refcount属性几乎不应该直接由内部函数修改。这些值在Zend引擎调用你的函数 时被初始化和处理。

让我们来看看这个特殊的函数，将它添加到sample_hello_world()函数下面的第五章"你的第一个扩展"中的示例扩展。你还需要展开php_sample_functions结构以包含sample_long()的函数条目，如下所示：

```c
static function_entry php_sample_functions[] = {
	PHP_FE(sample_hello_world, NULL)
    PHP_FE(sample_long, NULL)
    {NULL, NULL, NULL}
};
```

此时，可以通过从源目录发出make来重建扩展，或者从Windows的PHP源码根目录nmake php_sample.dll来重建。

如果一切顺利，现在可以运行PHP并执行新的函数：

```shell
$ php -r "var_dump(sample_long());"
```

## 被紧紧包裹的宏

为了代码的可读性、可维护性，ZVAL_\*()宏具有特定于*return_value*变量的重复副本。在任何情况下，宏的ZVAL部分被替换为RETVAL，并且将被省略，否则可能表示被修改的变量的初始参数被省略。

在前面的例子中，sample_long()的实现可以简化为以下内容：

```c
PHP_FUNCTION(sample_long){
	RETVAL_LONG(42);
	return;
}
```

表6.1列出了由Zend Engine定义的RETVAL系列宏。在除两个以外的所有情况下，RETVAL宏与其初始return_value参数被删除的ZVAL对应物相同。

| 通用ZVAL宏                                  | return_value特定对应              |
| ---------------------------------------- | ----------------------------- |
| ZVAL_NULL(return_value)                  | RETVAL_NULL()                 |
| ZVAL_BOOL(return_value, bval)            | RETVAL_BOOL(bval)             |
| ZVAL_TRUE(return_value)                  | RETVAL_TRUE                   |
| ZVAL_FALSE(return_false)                 | RETVAL_FALSE                  |
| ZVAL_LONG(return_value, lval)            | RETVAL_LONG(lval)             |
| ZVAL_DOUBLE(return_value, dval)          | RETVAL_DOUBLE(dval)           |
| ZVAL_STRING(return_value, str, dup)      | RETVAL_STRING(str, dup)       |
| ZVAL_STRINGL(return_value, str, len, dup) | RETVAL_STRINGL(str, len, dup) |
| ZVAL_RESOURCE(return_value, rval)        | RETVAL_RESOURcE(rval)         |

> **注意**
>
> 需要注意的是，TRUE和FALSE宏没有括号。这些被认为是变Zend/PHP编码标准中的差异，但主要是为了向后兼容而保留的。如果你构建扩展并收到读取未定义宏宏RETVAL_TRUE()的错误，请务必检查，你没有使用括号。

通常，在你的函数产生一个返回值之后，它将准备好退出并将控制权返回给调用范围。由于这个原因，还有一组专门为内部函数设计的宏：RETURN_*()系列。

```c
PHP_FUNCTION(sample_long){
	RETURN_LONG(42);
}
```

虽然它实际上不可见，但是该函数仍然在RETURN_LONG()宏调用结束时显式返回。这可以通过在函数结尾添加一个php_printf()调用来测试：

```c
PHP_FUNCTION(sample_long){
	RETURN_LONG(42);
  	php_printf("I will never be reached.\n");
}
```

php_printf()，正如其内容所暗示的，将永远不会执行，因为对RETURN_LONG()的调用隐式地离开了该函数。像RETVAL系列一样，对于表6.1所示的每种简单类型，都存在一个RETURN对象。与RETVAL系列一样，RETURN_TURE和RETURN_FALSE宏也不使用括号。

更复杂的类型，如对像和数组，也通过return_value参数返回；然而，他们的性质排除了一种简单的基于宏观的创作方法。即使是资源类型，虽然他有一个RETVAL宏，但需要额外的工作来生成。稍后将在第8章至第11章中看到如何返回这些类型。

## 值得的麻烦？

Zend内部函数的一个未被使用的特性是return_value_used参数。考虑下面一段PHP代码：

```php
function sample_array_range(){
  	$ret = array();
  	for($i=0; $i<1000;$i++){
  		$ret[] = $i;
  	}
  	return $ret;
}
sample_array_range();
```

由于调用sample_array_range()时不将结果存储到变量中，因此用于创建1000个元素的数组的工作和内存完全被浪费。当然，以这种方式调用sample_array_range()是愚蠢的，但是提前知道它的努力是徒劳的么？

尽管PHP函数不能访问，但是内部函数可以有条件地跳过这种无意义的行为，这取决于所有内部函数共有的return_value_used参数的设置。

```c
PHP_FUNCTION(sample_array_range){
  	if(return_value_used){
  		int i;
    	/* 返回 0-999的数组 */
      	array_init(return_value);
      	for(i=0; i<1000; i++){
        	add_next_index_long(return_value, i);
      	}
      	return;
    }else{
      	/* Save yourself the effort */
      	php_error_docref(NULL TSRMLS_CC, E_NOTICE, "Static return-only function called without processing output");
      	RETURN_NULL();
    }
}
```

要看到这个函数的操作，只需要将它添加到不断增长的sample.c源文件中，然后将匹配的条目添加到php_sample_functions结果中：

```c
PHP_FE(sample_array_range, NULL)
```

## 返回引用值

正如您在用户空间中所了解的，PHP函数也可能会通过引用返回值。由于实现的问题，在5.1之前版本的PHP应该避免从内部函数返回引用，因为它根本不起作用。参考下面的PHP代码：

```php
function &sample_reference_a(){
	/* If $a does not exsit in the global scope yet, create it with an initial value of NULL */
  	if(!isset($GLOBALS['a'])){
    	$GLOBALS['a'] = NULL;
  	}
  	return $GLOBALS['a'];
}
$a = 'Foo';
$b = sample_reference_a();
$b = 'Bar';
```

在这个代码片段中，\$b被创建为\$a的引用，就像它使用\$b = &$GLOBALS['a']设置的一样。

或者因为它在全球范围内已经完成了\$b=&\$a;。

当到达最后一行时，你将从第三章”内存管理“中回想的\$a和\$b都看着相同的实际值，包含值”Bar“。我们再来看看使用内部实现的同样的函数：

```c
#if (PHP_MAJOR_VERSION > 5) || (PHP_MAJOR_VERSION == 5 && PHP_MINOR_VERSION > 0)
PHP_FUNCTION(sample_reference_a){
	zval **a_ptr, *a;
  	/* fetch $a from the global symbol table */
  	if(zend_hash_find(&EG(symbol_table), "a", sizeof("a"), (void**)&a_ptr)==SUCCESS){
    	a = *a_ptr;
    }else{
      	/* $GLOBALS['a'] doesn't exist yet, create it */
      	ALLOC_INIT_ZVAL(a);
      	zend_hash_add(&EG(symbol_table), "a", sizeof("a"), &a, sizeof(zval *), NULL);
    }
  	/* Toss out the old return_value */
  	zval_ptr_dtor(return_value_ptr);
  	if(!a->is_ref && a->refcount > 1){
    	/* $a is in a copy-on-write reference set It must be separated before it can be used */
      	zval *newa;
      	MAKE_STD_ZVAL(newa);
      	*newa = *a;
      	zval_copy_ctor(newa);
      	newa->is_ref = 0;
      	newa->refcount = 1;
      	zend_hash_update(&EG(symbol_table), "a", sizeof("a"), &newa, sizeof(zval*), NULL);
      	a = newa;
  	}
  	a->is_ref = 1;
  	a->refcount++;
  	*return_value_ptr = a;
}
#endif
```

return_value_ptr参数是传递给所有内部函数的另一个通用参数，并且是一个包含指向return_value的指针的zval **。通过调用zval_ptr_dtor()，将释放默认的return_value zval *。然后，你可以自由地用你选择的新zval *替换它，在这种情况下，变量\$a已被提升为is_ref，并且可以从任何非券参考配对中分离。

如果你现在编译和运行这个代码，你会得到一个段错误。为了使它工作，你需要添加一个结构到你的php_sample.h文件中：

```c
#if(PHP_MAJOR_VERSION>5) || (PHP_MAJOR_VERSION==5) && PHP_MINOR_VERSION > 0)
static
	ZEND_BEGIN_ARG_INFO_EX(php_sample_retref_arginfo, 0, 1, 0)
  	ZEND_END_ARG_INFO()
#endif
```

然后在php_sample_functions中声明你的函数时使用这个结构：

```c
#if(PHP_MAJOR_VERSION>5) || (PHP_MAJOR_VERSION==5 && PHP_MINOR_VERSION > 0)
	PHP_FE(sample_reference_a, php_sample_retref_arginfo)
#endif
```

这个结构，你会在本章后面学到更多的内容，它为Zend Engine函数调用例程提供了重要的提示。在这种情况下，它告诉ZE return_value将学要被覆盖，并且它应该用正确的地址填充return_value_ptr。如果没有这个提示，ZE将简单地将NULL置于return_value_ptr中，这回使特定函数在到达zval_ptr_dtor()时崩溃。

> **注意**
>
> 这些代码片段中的每一个都被封装在\#if块之中，以通知编译器只有在PHP版本大于等于5.1时才支持它们。如果没有这些条件指令，扩展将无法在PHP4上进行编译（因为包括return_value_ptr在内的多个元素不存在），并且无法在PHP5.0上正常工作（其中一个错误会导致引用返回值被复制）。