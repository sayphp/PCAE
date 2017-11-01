# 数据转换

现在你可以从符号表中获取变量，你将需要与它们进行交互。一个直接而痛苦的方法可能是检查变量并根据类型执行特定的操作。下面的switch语句可以达到想要的效果：

```c
void display_zval(zval *value)
{
  switch(Z_TYPE_P(value)){
    case IS_NULL:
      /*NULLs are echoed as nothing*/
      break;
    case IS_BOOL:
      if(Z_BVAL_P(value)){
        php_printf("1");
      }
      break;
    case IS_LONG:
      php_printf("%ld", Z_LVAL_P(value));
      break;
    case IS_DOUBLE:
      php_printf("%f", Z_DVAL_P(value));
      break;
    case IS_STRING:
      PHPWRITE(Z_STRVAL_P(value), Z_STRLEN_P(value));
      break;
    case IS_RESOURCE:
      php_printf("Resource #%ld", Z_RESVAL_P(value));
      break;
    case IS_ARRAY:
      php_printf("Array");
      break;
    case IS_OBJECT:
      php_printf("Object");
      break;
    default:
      /*Should never happen in practice, but it's dangerous to make assumptions*/
      php_printf("Unknow");
      break;
  }
}
```

与*<?php echo $value;?>*相比，不难想象这段代码变得难以管理。幸运的是，当脚本执行回显变量的动作时，引擎使用的相同例程也可用于扩展或嵌入环境。使用Zend导出的*convert_to_\*()*函数之一，此示例可以简化为：

```c
void display_zval(zval *value)
{
  convert_to_string(value);
  PHPWRITE(Z_STRVAL_P(value), Z_STRLEN_P(value));
}
```

你可能猜到，有一些函数可以转换成大多数的数据类型。一个值得注意的例外是*convert_to_resource()*，这是没有意义的，因为根据定义，资源不能映射到真正的PHP可以表达的值。如果你担心*convert_to_string()*调用不可逆转地更改了传递给函数的zval的值，那么这很好。在一个真正的代码段中，这通常是一个坏主意。当然，引擎在回显一个变量时不会发生什么。在下一章中，你将看到使用convert函数安全地将值的内容更改为可用的内容而不破坏其现有内容的方法。

