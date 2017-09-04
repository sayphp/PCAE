# 目录

[TOC]

### 0x00 介绍

### 0x01 PHP生命周期

* 始于SAPI
* 启动与关闭
* 生命周期
  * CLI的生命周期
  * 多进程生命周期
  * 多线程生命周期
  * 嵌入式生命周期
* Zend线程安全
  * 线程安全与非线程安全声明
  * 线程安全数据池
  * 何时不要线程
  * 不可知的全局访问
  * 即使你不需要线程
  * 找寻失去的tsrm_ls
* 总结

### 0x02 深入浅出变量

* 数据类型
* 数据值
* 数据创建
* 数据存储
* 数据检索
* 数据转换
* 总结

### 0x03 内存管理

* 内存
  * 释放Mallocs
  * 错误处理
  * Zend内存管理
* 引用计数
  * 写时拷贝
  * 写时变更
  * 分离的困扰
* 总结

### 0x04 构建环境

* 构建PHP
  * *nix 工具
  * Win32 工具
  * 获取PHP源代码
  * 通过Git检出

> 原书内容是通过CVS来检出代码的，但是PHP早已经迁移到了github上面了，所以这边改成git的检出

* 配置PHP进行开发
  * --enable-debug
  * --enable-maintainer-zts
  * --enable-embed
* UNIX下编译
* Win32下编译
* 总结

### 0x05 你的第一个扩展

* 扩展剖析
  * 配置文件
  * Header
  * Source
* 建立你的第一个扩展
  * 在*nix下构建
  * 在Windows下构建
  * 加载共享模块构建的扩展
* 构建静态扩展
  * 在*nix下构建
  * 在Windows下构建
* 实用函数
  * Zend内部函数
  * 函数别名
* 总结

### 0x06 返回值

* 变量 return_value
  * 包装你的宏
  * 是值得的麻烦么
  * 返回引用值（参考值）
* 根据引用返回值
  * 运行时传递引用
  * 编译是传递引用
* 总结

### 0x07 接收参数

* 使用zend_parse_parameters()进行自动类型转换
  * 可选参数
  * IS_NULL与NULL
  * 强制分离
* zend_get_arguments()
  * 处理可变数量参数
* 参数信息和类型提示
* 总结

### 0x08 使用数组与哈希表

* 向量与链表
  * 向量
  * 链表
  * 两全其美的哈希表
* Zend  Hash API
  * 创建
  * Population
  * Recall
  * QuickPopulation and Recall
  * Copying and Merging
  * Iteration by Hash Apply
  * Iteration by Move Forward
  * Preserving the Internal Pointer
  * Destruction
  * Sorting,Comparing, and Going to the Extreme(s)
* zval *Array API
  * Easy Array Creation
  * Easy Array Population
* 总结

### 0x09 The Resource Data Type

* Complex Structures
  * Defining Resource Types
  * Registering Resources
  * Destroying Resources
  * Decoding Resources
  * Forcing Destruction
* Persistent Resources
  * Memory Allocation
  * Delayed Destruction
  * Long Term Registration
  * Reuse
  * Liveness Checking and Early Departure
  * Agnostic Retrieval
* The Other refcounter
* 总结

### 0x0A PHP4 Objects

* The Evolution of the PHP Object Type Implementing Classes
  * Declaring Class Entries
  * Defining Method Implementations
  * Constrructors
  * Inheritance
* Working with Instances
  * Creating Instances
  * Accepting Instances
  * Accessing Properties
* 总结

### 0x0B PHP5 Objects

* Evolutionary Leaps
  * zend_class_entry
* Methods
  * Declaration
  * Special Methods
* Properties
  * Constants
* Interfaces
  * Implementing Interfaces
* Handlers
  * Starndard Handlers
  * Magic Methods, Part Deux
* 总结

### 0x0C Startup,Shutdown,and a Few Points in Between

* Cycles
  * Module Cycle
  * Thread Cycle
  * Request Cycle
* Exposing Information Through MINFO
* Constants
* Extension Globals
  * Declaring Extension Globals
  * Per-Thread Initializing and Shutdown
  * Accessing Extension Globals
* Userspace Superglobals
  * Auto Global Callback
* 总结

### 0x0D INI Settings

* Declaring and Accessing INI Settings
  * Simple INI Settings
  * Access Levels
  * Modification Events
  * Displaying INI  Settings
  * Binding to Extension Globals
* 总结

### 0x0E Accessing Streams

* Streams Overview
* Opening Streams
  * Fopen Wrappers
  * Transports
  * Directory Access
  * Special Streams
* Accessing Streams
  * Reading
  * Reading Directory Entries
  * Writing
  * Seeking, Telling, and Flushing Closing
  * Exchanging Streams for zvals
* Static Stream Operations
* 总结

### 0x0F Implementing Streams

* PHP Streams Below the Surface
* Wrapper Operations
  * Static Wrapper Operations
* Implementing a Wrapper
  * Inside the Implementation
  * URL Parsing
  * opendir()
* Manipulation
  * unlink
  * rename, mkdir and rmdir
* Inspection
  * stat
  * url_stat
* 总结

### 0x10 Diverting the Stream

* Contexts
  * Setting Options
  * Retrieving Options
  * Parameters
  * The Default Context
* Filters
  * Applying Existing Filters to Streams
  * Defining a Filter Implementation
* 总结

### 0x11 Configuration and Linking

* Autoconf
* Looking for Libraries
  * Scanning for Headers
  * Testing for Functionality
  * Optional Functionality
  * Testing Actual Behavior
* Enforcing Module Dependencies
  * Configuretime Module Dependency
  * Runtime Module Dependency
* Speaking the Windows Dialect
* 总结

### 0x12 Extension Generators

* ext_skel
  * Generating Function Prototypes
* PECL_Gen
  * specfile.xml
  * About the Extension
  * Dependencies
  * Constatnts
  * Globals
  * INI Options
  * Functions
  * Custom Code
* 总结

### 0x13 Setting Up a Host Environment

* The Embed SAPI
* Building and Compiling a Host Application
* Re-creating CLI by Wrapping Embed
* Reusing Old Tricks
  * Seting Initial Variables
  * Overriding INI options
  * Declaring Additional Superglobals
* 总结

### 0x14 Advanced Embedding

* Calling Back into PHP
  * Alternatives to Script File Inclusion
  * Calling Userspace Functions
* Dealing with Errors
* Initializing PHP
* Overriding INI_SYSTEM and INI_PERDIR Options
  * Overriding the Default php.ini File
  * Overriding Embed Startup
* Capturing Output
  * Standard Out:ub_write
  * Buffering Ouput:Flush
  * Standard Error:log_message
  * Special Errors:sapi_error
* Extending and Embedding at Once
* 总结

### 0x15 Zend API Refernce

* Parameter Retrieval
* Classes
  * Properties
* Objects
* Exceptions
* Exceution
* INI Settings
* Array Manipulation
* Hash Tables
* Resources/Lists
* Linked Lists
* Memory
* Constants
* Variables
* Miscellaneous API Function
* 总结

### 0x16 PHPAPI

* Core PHP
  * Output
  * Error Reporting
  * Startup, Shutdown, and Exceution
  * Safe Mode and Open Basedir
  * String Formatting
  * Reentracy Safety
  * Miscellaneous
* Streams API
  * Stream Creation and Destruction
  * Stream I/O
  * Stream Manipulation
  * Internal/Userspace Conversion
  * Contexts
  * Filters
  * Plainfiles and Standard I/O
  * Transports
  * Miscellaneous
* Extension APIs
  * ext/standard/base64.h
  * ext/standard/exec.h
  * ext/standard/file.h
  * ext/standard/flock_compat.h
  * ext/standard/head.h
  * ext/standard/html.h
  * ext/standard/info.h
  * ext/standard/php_filstat.h
  * ext/standard/php_http.h
  * ext/standard/php_mail.h
  * ext/standard/php_math.h
  * ext/standard/php_rand.h
  * ext/standard/php_string.h
  * ext/standard/php_smart_str.h
  * ext/standard/php_uuencoed.h
  * ext/standard/php_var.h
  * ext/standard/php_versioning.h
  * ext/standard/reg.h
  * ext/standard/md5.h
  * ext/standard/sha1.h
* 总结

### 0x17 Extending and Embedding Cookbook

* Skeletons
  * Minimal Extension
  * Extension Life Cycle Methods
  * Declaring Module Info
  * Adding Functions
  * Adding Resources
  * Adding Objects
* Code Pantry
  * Calling Back Into Userspace
  * Evaluating and Executing Code
  * Testing and Linking External Libraries
  * Mapping Arrays to String Vectors
  * Accessing Streams
  * Accessing Transports
  * Computing a Message Digest
* 总结

### 0x18 Additional Resources

* Open Source Projects
  * The PHP Source Tree
  * PECL as a Source of Inspiration
  * Eleswhere in the PHP CVS Repository
* Places to Look for Help
  * PHP Mailing Lists
  * IRC
* 总结





