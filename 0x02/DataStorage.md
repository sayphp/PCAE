# 数据存储

你已经使用过了PHP，因此你已经熟悉了数组的概念。任何数量的PHP变量（zvals）都可以删除到单个容器（数组）中，并以数字活字富川的形式给出名称（标签）。

不要奇怪，PHP脚本中的每个变量都可以在数组中找到。创建变量时，通过为其分配一个值，Zend将该值存储到称为符号表的内部数组中。

定义全局范围的一个符号表在调用扩展RINIT方法之前的请求启动时初始化，然后再脚本完成和后续的RSHUTDOWN方法执行后被销毁。

当调用用户控件函数或对象方法时，将为该函数或方法的寿命分配一个新的符号表，并将其定义为活动符号表。如果当前脚本执行不再函数或方法中，则全局符号表被认为是活动的。

看看执行全局变量结构（在Zend/zend_globals.h中定义），你将找到一下定义的两个元素：

```c
struct _zend_execution_globals{
  …
  HashTable symbol_table;
  HashTable *active_symbol_table;
  …
}
```

作为EG(symbol_table)访问的symbol_table始终是全局变量范围，非常类似于PHP中的\$\_GLOBALS变量始终对应于PHP脚本的全局范围。实际上，\$\_GLOBALS变量只是从内部看到的EG(symbol_table)变量的用户空间包装。

另一部分，active_symbol_table与EG(active_symbol_table)类似地访问，并且表示当时活动的任何变量范围。

在这里要注意的主要区别是，EG(symbol_table)与使用PHP和Zend API时使用和与到的几乎所有其他HashTable不同，它是一个直接变量。然而，几乎所有在HashTables上运行的函数 都希望间接的HashTable \*作为参数。因此，你必须去学校引用EG(symbol_table)在使用时带有&符号。

考虑以下两段代码，它们在功能上是相同的：

```php
//*PHP
<?php
  	$foo = 'bar';
?>
```

```c
//*C
{
  zval *fooval;
  
  MAKE_STD_ZVAL(fooval);
  ZVAL_STRING(fooval, "bar", 1);
  ZEND_SET_SYMBOL(EG(active_symbol_table), "foo", fooval);
}
```

首先，使用MAKE_STD_ZVAL()分配一个新的zval，并将其值初始化为字符串“bar”。那么一个与赋值运算符（=）大致相等的新宏将该值与一个标签（foo）相结合，并将其添加到活动符号表中。因为当时没有PHP功能激活，EG(active_symbol_table) == &EG(symbol_table)，最终意味着这个变量存储在全局范围内。