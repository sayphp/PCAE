# 数据值

与类型一样，可以使用三元组的宏来检查zvals的值。这些宏也以*Z_*开始，并且根据其间接程度可选地以*_P*或*_PPP*结尾。对于简单的标量类型，boolean、long和double宏是简短且一致的：*BVAL*、*LVAL*和*DVAL*。

```c
void display_values(zval boolzv, zval *longpzv, zval **doubleppzv)
{
  if(Z_TYPE(boolzv) == IS_BOOL){
    php_printf("The value of the boolean is: %s\n", Z_BVAL(boolzv)?"true":"false");
  }
  if(Z_TYPE_P(longpzv) == IS_LONG){
    php_printf("The value of the long is: %ld\n", Z_LVAL_P(longpzv));
  }
  if(Z_TYPE_PP(doubleppzv) == IS_DOUBLE){
    php_printf("The value of the double is: %f\n", Z_DVAL_PP(doubleppzv));
  }
}
```

字符串变量，因为它们包含两个属性，有一对代表char *(STRVAL)的宏三元组，和int(STRLEN)元素：

```c
void display_string(zval *zstr)
{
  if(Z_TYPE_P(zstr) != IS_STRING){
    php_printf("The wrong datatype was passed!\n");
    return;
  }
  PHPWRITE(Z_STRVAL_P(zstr), Z_STRLEN_P(zstr));
}
```

数组数据类型在内部存储为可以使用ARRVAL三元组访问的HashTable:*Z_ARRVAL(zv)*、*Z_ARRVAL_P(pzv)*，*Z_ARRVAL_PP(ppzv)*。当查看PHP内核和PECL模块中的旧代码时，你可能会遇到HASH_OF()宏，它需要一个zval *。这个宏一版是相当于Z_ARRVAL_P()宏；但是，这个宏已经被弃用，不应该与新代码一起使用。

对象表示复杂的内部结构，并具有多个访问宏：OBJ_HANDLE，它返回句柄标识符，用于处理程序表的OBJ_HT，类定义的OBJCE，属性HashTable的OBJPROP以及用于在OBJ_HT中操作特定处理程序方法的OBJ_HANDLER表。

不要担心这些各种对象宏的意思；在第10章“PHP4对象”和第11章“PHP5对象”中详细介绍它们。在zval中，可以通过简单的整数来使用资源数据类型（存储为可以使用RESVAL）。该证书传递给*zend_fetch_resource()*函数，它从其数字标识符中查找注册的资源。资源数据类型将在第9章深入讨论。