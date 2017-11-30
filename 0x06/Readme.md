PHP语言使用*return*关键词将信息传递回其调用范围，方法与你在C应用程序中熟悉的方式相同，例如：

```php
function sampe_long(){
	return 42;
}
$bar = sample_long();
```

当调用*sample_long()*时，返回数字42并填充到$bar变量中。在C中，可以使用几乎相同的代码来完成：

```c
int sample_long(){
	return 42;
}
void main(void){
	int bar = sample_long();
}
```

当然，在C中，你总是知道被调用的函数将根据它的函数类型返回，所以你可以提前声明结果变量的类型。但是，在处理PHP时，变量类型是动态的，你必须去看看第二章“深入浅出变量”中介绍的zval类型。