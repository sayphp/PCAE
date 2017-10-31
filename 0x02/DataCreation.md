# 数据创建

现在，你已经看到如何从zval中提取数据，现在创建属于自己的数据。虽然zval可以简单地声明为一个函数顶部的直接变量，但它会使变量的数据存储本地，并且必需复制它才能离开函数并达到PHP。

因为你几乎总是希望创建一个可以以某种形式访问PHP的zvals，所以你需要为其分配一块内存块，并将该块分配给一个zval*指针。再次使用malloc(sizeof(zval))显然是错误的答案。相反，你需要使用另外一个Zend宏：MAKE_STD_ZVAL(pzv)。此宏将在其他zvals附近优化的大部分内存中分配空间，自动处理内存不足错误（你将在下一章中进一步介绍），并处实话新zval的引用计数(refcount)和是否引用(is_ref)属性。

> **注意**
>
> 除了MAKE_STD_ZVAL()之外，你还会经常看到PHP源中使用的另一个zval \*创建宏：ALLOC_INT_ZVAL()。该宏仅与MAKE_STD_ZVAL()不同，因为它将zval \*的数据类型初始化为IS_NULL。

一旦数据存储空间可用，就可以使用一些信息填充你的全新zval。在更早的阅读数据存储部分之后，你可能全部使用这些Z_TYPE_P()和Z_SOMEVAL_P()宏来设置新的变量。看上去似乎是个是显而易见的解决方案？

事实上，这个方案不够。

Zend开放了另一组用于设置zval \*值的宏。以下是这些新宏以及它们扩展到你熟悉的那些地方：

```c
ZVAL_NULL(pvz);				Z_TYPE_P(pvz) = IS_NULL;
```

虽然这个宏并没有为使用更直接的版本提供任何节省，但它背包括在内。

```c
ZVAL_BOOL(pzv, b);					Z_TYPE_P(pzv) = IS_BOOL;
									Z_BVAL_P(pzv) = b?1:0;
ZVAL_TRUE(pzv);						ZVAL_BOOL(pzv, 1);
ZVAL_TRUE(pzv);						ZVAL_BOOL(pzv, 0);
```

需要注意，提供给ZVAL_BOOL()的任何非零值都将产生一个真值。这当然是有道理的，因为在PHP中转换为布尔值的任何非零值类型都将表现出相同的行为。当将值硬编码到内部代码中时，明确使用值1作为真值被认为是很好的做法。提供宏ZVAL_TRUE()和ZVAL_FALSE()作为方便，有时可以提供代码可读性。

```c
ZVAL_LONG(pzv, 1);					Z_TYPE_P(pzv) = IS_LONG;
									Z_LVAL_P(pzv) = 1;
ZVAL_DOUBLE(pzv, d)					Z_TYPE_P(pzv) = IS_DOUBLE;
									Z_DVAL_P(pzv) = d;
```

基本的标量宏就像它们一样简单。设置zval的类型，并为其分配一个数值。

```c
ZVAL_STRINGL(pzv, str, len, dup);	Z_TYPE_P(pzv) = IS_STRING;
									Z_STRLEN_P(pzv) = len;
									if(dup){
										Z_STRVAL_P(pzv) = estrndup(str, len+1);	
                                    }else{
                                      	Z_STRVAL_P(pzv) = str;
                                    }
ZVAL_STRING(pzv, str, dup);			ZVAL_STRINGL(pzv, str, strlen(str), dup);
```

这里是zval创建开始变得有趣的地方。字符串，如数组、对象和资源一样，需要分配额外的存储器用于数据存储。你将在下一章中探讨记忆管理的陷阱；对于现在，只要注意，dup值为1将分配新的内存并复制字符串的内容，而值为0将简单地将zval指向已存在的字符串数据。

```c
ZVAL_RESOURCE(pzv, res);			Z_TYPE_P(pzv) = IS_RESOURCE;
									Z_RESVAL_P(pzv) = res;
```

从前面回想一个资源存储在一个zval中，作为引用由Zend管理的查找表的简单整数。因此，ZVAL_RESOURCE()宏的行为非常像ZVAL_LONG()宏，但使用不同的类型。