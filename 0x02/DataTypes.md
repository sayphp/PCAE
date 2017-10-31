# 数据类型

PHP中数据存储的基本单位是`zval`，或者`ZendValue`。它是一个小的构造体具备四个元素，在*Zend/zend.h*中定义，代码如下：

```c
//*PHP5版本
typedef struct _zval_struct {
  zvalue_value value;//值
  zend_uint refcount;//引用计数
  zend_uchar type;//数据类型
  zend_uchar is_ref;//是否引用
} zval;
```

直观的看来，对于大多数成员来说，存储类型都是十分简单：无符号整型的引用计数、无符号字符的数据类型、无符号字符的是否引用。然而，值成员实际上是一个从PHP5开始定义的联合结构：

```c
typedef union _zvalue_value {
  long lval;
  double dval;
  struct {
    char *val;
    int len;
  } str;
  HashTable *ht;
  zend_object_value obj;
} zvalue_value;
```

该联合允许Zend存储许多不同类型的数据，PHP变量能够以单一的统一结构保存。Zend(*Zend/zend.h*)目前定义了八种数据类型，参考下表：

| 类型值         | 用意                                       |
| ----------- | ---------------------------------------- |
| IS_NULL     | 此类型在首次使用时自动分配给未初始化的变量，并且还可以使用内置的NULL常量在PHP中显示分配。该变量类型提供了一个特殊的“非值”，它与布尔值FALSE或整型0不同。 |
| IS_BOOL     | 布尔变量可以只有两种可能的状态：TRUE或FALSE。PHP控制结构中的条件表达式（if），循环条件判断（while），三目运算符(a?b:c)在判断时会隐式的将类型转换成布尔值。 |
| IS_LONG     | 使用主机系统的签名长数据类型存储PHP中的整数数据类型。在大多数32位平台上，它的存储范围为-21474483648～2147483647。除了一些例外情况，每当用户空间脚本尝试存储超出此范围的整数值时，它将自动转换为双精度浮点型（IS_DOUBLE） |
| IS_DOUBLE   | 浮点数据类型使用主机系统的双重数据类型。浮点数并不精确存储；而是使用一个公式来表示该值，作为有限精度（位数）乘以2的一部分提升到一定的幂（指数）。该表示允许计算机存储大范围的值表中列出的IS_*常熟存储在zval结构的类型元素中，并确定在检查其值时应查看zval结构的值元素的哪个部分。<br />检查类型值的最明显方法可能是从给定的zval中取消引用，如以下代码段所示：（正或负）从小到2.225×10^(-308)到约1.798的上限×10^308只有8个字节。<br />不幸的是，以十进制计算精确数字的数字并不总是作为二进制分数存储。例如，十进制表达式0.5将计算出精确的二进制数字为0.1，而小数点0.8变为0.1100110011的重复二进制表示。当转换回十进制时，截断的二进制数字产生一个稍微偏移的值，因为他们不能存储整个数字。想想它就像试图将1/3数字表示成十进制数：0.333333来的非常接近，但不清除，因为3×0.33333333不是1.0。当处理计算机上的浮点数时，这种不精确性通常会导致混淆。（这些范围限制基于常见的32位平台；范围可能因操作系统而存在差异）。 |
| IS_STRING   | PHP最通用的数据类型是以经验丰富的C程序员期望的否嗯是存储的字符串。分配足够容纳字符串的所有自己/字符的内存块，并将盖子富川的指针存储在主机zval中。关于PHP字符串值得注意的是，字符串的长度总是在zval结构中明确声明。者允许字符串包含NULL字节而不被阶段。PHP字符串的这一方面在下文中将被称为二进制安全性，因为它使得他们可以安全地包含任何类型的二进制数据。请注意为给定的PHP字符串分配的内存量始终至少为其长度加1。最后一个字节填充有一个终止NULL哈希字符串，因此不需要二进制安全的函数可以简单的将字符串指针传递给它们的底层方法。 |
| IS_ARRAY    | 数组是一个专用变量，其唯一的功能是携带其他变量。与C的数组概念不同，PHP数组不是同一数据类型的向量（如zval arrayofzvals[];）。相反，PHP数组是一组复杂的数据桶，链接到称为HashTable的结构中。每个HashTable元素（bucket）包含两个相关信息：标签和数据。在PHP数组的情况下，标签是数组中的关系或数字索引，数据是该键引用的变量（zval）。 |
| IS_OBJECT   | 对象采取数组的多元数据存储，并通过添加方法，访问修饰符，作用域常量和特殊事件处理程序进一步。作为扩展开发人员，构建面向对象的代码在PHP4和PHP5中同样出色，这是一个特别的挑战，因为内部对象模型在Zend Engine 1（PHP4）和Zend Engine 2（PHP5）之间变化很大。 |
| IS_RESOURCE | 一些数据类型根本无法映射到PHP。例如，stdio的FILE指针或libmysqlclient的链接句柄不能简单地映射到一个标量值数组，也不会有意义。为了屏蔽，让PHP脚本编写器不必处理这些问题，PHP提供了一般的资源数据类型。第九章“资源数据类型”将介绍资源的实现细节；现在只要知道他们存在即可。 |

表2.1中列出的IS\_\*常量存储在zval结构的*type*元素中，并确定在检查其值时应该查看*zval*结构额值元素的哪个部分。检查类型值的最明显的方法可能是从给定的zval中取消引用它，代码如下：

```c
void describe_zval(zval *foo)
{
  if(foo->type == IS_NULL) {
    php_printf("The variable is NULL");
  } else {
    php_printf("The variable is of type %d", foo->type);
  }
}
```

虽然这并不错，但必然不是首选方案。Zend头文件包含大量的zval访问宏，扩展作者在检查zval数据时应该使用该宏。这样做的主要原因在引擎API改变时避免不兼容；但副作用是，代码阅读起来不是那么直接明了。这是同样的代码片段，这次使用*Z_TYPE_P()*宏：

```c
void describe_zval(zval *foo)
{
  if(Z_TYPE_P(foo) == IS_NULL){
    php_printf("The variable is NULL");
  }else{
    php_printf("The variable is of type %d", Z_TYPE_P(foo));
  }
}
```

此宏的_P后缀表示传递的参数包含单级间接。另外两个这个集合中存在的宏为*Z_TYPE*和*Z_TYPE_PP()*，它们期望类型分别为zval（无间接）的参数，以及zval **（两级间接）。

> **注意**
>
> 在这个例子中，使用了一个特殊的输出函数php_printf()，来显示一个数据。这个功能在语法上与stdio的printf()函数相同；然而，它处理Web服务器SAPI的特殊处理，并利用PHP的输出缓冲机制。你将在第5张“你的第一个扩展”中了解有关此功能及其“表弟”*PHPWRITE()*。



> PHP7的时候，鸟哥将zval的数据结构做了改造，以提升性能，可以理解为空间换时间的一种优化行为。在*Zend/zend_types.h*中定义

```c
//*PHP7版本
typedef struct _zval_struct zval;

struct _zval_struct {
  zend_value value;
  union {
    struct{
      ZEND_ENDIAN_LOHI_4(
      	zend_uchar type,
        zend_uchar type_flags,
        zend_uchar const_flags,
      	zend_uchar reserved)
    } v;
    uint32_t type_info;
  } u1;
  union {
    uint32_t next;
    uint32_t cache_slot;
    uint32_t lineno;
    uint32_t num_args;
    uint32_t fe_pos;
    uint32_t fe_iter_idx;
    uint32_t access_flags;
    uint32_t property_guard;
    uint32_t extra;
  } u2;
};
```

> 在PHP5中，数据类型存储与*Zend/zend.h*中，而在PHP7中，数据类型择存储在*Zend/zend_types.h*中，其中，IS_BOOL被替换成了_IS_BOOL，还增加了IS_TRUE、IS_FALSE。