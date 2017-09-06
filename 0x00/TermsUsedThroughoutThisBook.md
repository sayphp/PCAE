# 本书中使用的术语

| 术语                 | 中文                  | 解释                                       |
| ------------------ | ------------------- | ---------------------------------------- |
| PHP                | 拍黄片                 | 指的是整个PHP解释器，包括Zend，TSRM，SAPI层和各种扩展       |
| PHP Core           | PHP核心               | PHP解释器的一小部分，在本章前面的“PHP与Zend”部分中有过定义      |
| Zend               | Zend、Zend引擎、Zend虚拟机 | Zend引擎处理解析、编译和执行脚操作码（Opcode）             |
| PEAR               | PHP扩展和应用程序库         | http://pear.php.net/                     |
| PECL               | PHP扩展社区库            | http://pecl.php.net/                     |
| PHP extension      | PHP扩展               | 也称为PHP模块。一组以便医德代码，用于定义PHP语言客房问函数、类、流实现、常量、INI配置选项和专用资源类型。在任何地方看到 XXX exntension，你都可以直接认为它是一个PHP扩展名 |
| Zend extension     | Zend扩展              | 专业系统(如字节码缓存Opcode Cache和编码器)使用的PHP扩展的变体，这超出了本书的范围。 |
| Userspace          | 用户空间，PHP语言层面        | 这里指的是PHP语言所包含的环境和API库。用户空间层面无法访问没有通过Zend引擎和各种PHP扩展明确提供的PHP内核或数据结构。 |
| Internals(C-space) | PHP内核，C语言层面         | 引擎和扩展代码。该术语用于代指PHP语言不能直接访问的所有内容          |



> 整个介绍中，一直充斥着大量的“userspace”，直白的翻译是用户控件；但是，还是觉得把它译做PHP语言更为合适一些。
>
> 这本书的内容主要是依赖于C语言，而实现的东西——PHP（解释器），却是可以将PHP代码转化为机器码去执行的。