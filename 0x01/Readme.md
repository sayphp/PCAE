在常见的Web服务器环境中，你永远不会直接启动PHP解释器；而是启动Apache或一些其他的Web服务器，服务器会根据需要加载PHP和进程脚本——也就是请求***.php文件。

> 现在流行的Nginx+PHP-FPM相当于是通过反向代理的方式将请求发送给PHP FastCGI管理器，然后执行PHP，将结果返回Nginx，最后响应用户。