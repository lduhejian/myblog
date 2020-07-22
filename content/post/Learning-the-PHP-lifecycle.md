---
title: "PHP7 中的数组"
date: 2020-07-17
draft: true
tags: [PHP7, Array, 数组]
description: "PHP7 中的数组"
---

PHP 的生命周期

PHP 是一个复杂的机器, 任何想深入了解他的人应该掌握他的执行生命周期。主要的时间线如下：

PHP 启动。在命令行(CLI)或者通过FPM执行，然后 main() 函数执行。如果 PHP 作为 web 服务器的一个模块运行，就像 apxs2sapi(Apache 2) 那样，那么 PHP 是在 Apache 自身启动后一点点再启动的。Apache 启动后开始运行他的一系列模块，PHP 就是其中的一个。"Starting up" 也叫做模块启动，我们也简称为 MINIT。

一旦启动，PHP 将等待处理一个或多个请求。当我们在谈到 PHP CLI 时，该模式下只有一个请求：请求运行当前的脚本。然而，当我们在说 web 服务时（可以是 PHP-FPM 或者 web 服务器的模块），PHP 可以处理多个请求，处理完一个再处理下一个。这都取决于你如何配置的服务器：在关闭或者重用一个进程之前，你可以告诉他处理无限的请求，或者特定数量的请求。每当一个新的请求出现在待处理状态时，PHP 会运行一个请求处理步骤。我们将该步骤简称为 RINT。

请求已送达，一些内容（可能）已生成。好的，是时候关闭请求并准备处理另一个请求了。关闭请求称为“请求关闭步骤”。我们叫做 RSHUTDOWN.

在处理了一些请求之后，PHP 会被关停。关闭 PHP 进程叫做“模块关闭步骤”。我们简称为 MSHUTDOWN.

如果画下来上面说的步骤，将会如下图所示：

...[图]

并行模型

在 CLI 模式下，一切都很简单：只会有一个 PHP 进程处理一个请求：只会运行一个 PHP 脚本。CLI 模式是 Web 模式的特例，相对于 CLI, Web 模式要更复杂。

为了同时处理多个请求，你需要使用并发模型。在 PHP 有两种并发模型：一个是基于进程的并发，一个是基于线程的并发。

基于进程的模型中，每个 PHP 解释器都作为操作系统（OS）的一个进程对待。这种模型在 Unix 系统中非常常见。每个请求独立的对应一个进程。PHP-CLI， PHP-FPM 和 PHP-CGI 都是使用的这种模型。

基于线程的模型中，每个 PHP 解释器都作为一个独立的线程对待。该模型主要在 windows 操作系统中使用，但是大多数的 unix 系统也可以使用。要使用线程模型 PHP 和扩展需要在 ZTS 模式下编译。

下面的是基于进程的模型：

。。。【图】


下面的是基于线程的模型：


。。。【图】


注意：作为一个扩展开发者就，PHP 的多进程模型不是可选的，你必须支持他。你必须支持这样一个事实：您的扩展可以在线程环境中运行，特别是在Windows平台下，并且您必须针对它进行编程。

PHP 拓展钩子函数

你可能已经猜到了，PHP 引擎会在生命周期的某些阶段触发你的拓展。也就是钩子函数。你的扩展可以在针对引擎注册时声明函数钩子，从而声明对特定生存期点的兴趣。

你可以在 PHP 源码 zend_module_entry 结构体中清晰的看到这些钩子函数。

```
struct _zend_module_entry {
        unsigned short size;
        unsigned int zend_api;
        unsigned char zend_debug;
        unsigned char zts;
        const struct _zend_ini_entry *ini_entry;
        const struct _zend_module_dep *deps;
        const char *name;
        const struct _zend_function_entry *functions;
        int (*module_startup_func)(INIT_FUNC_ARGS);        /* MINIT() */
        int (*module_shutdown_func)(SHUTDOWN_FUNC_ARGS);   /* MSHUTDOWN() */
        int (*request_startup_func)(INIT_FUNC_ARGS);       /* RINIT() */
        int (*request_shutdown_func)(SHUTDOWN_FUNC_ARGS);  /* RSHUTDOWN() */
        void (*info_func)(ZEND_MODULE_INFO_FUNC_ARGS);     /* PHPINFO() */
        const char *version;
        size_t globals_size;
#ifdef ZTS
        ts_rsrc_id* globals_id_ptr;
#else
        void* globals_ptr;
#endif
        void (*globals_ctor)(void *global);                /* GINIT() */
        void (*globals_dtor)(void *global);                /* GSHUTDOWN */
        int (*post_deactivate_func)(void);                 /* PRSHUTDOWN() */
        int module_started;
        unsigned char type;
        void *handle;
        int module_number;
        const char *build_id;
};
```
让我们来具体看一下该如何编写这些钩子函数。

模块初始化： MINIT()

这是 PHP 进程启动步骤。在你的拓展中的 MINIT 函数中，你应该加载或者分配持久化的对象，或者在每个请求中需要用到的信息。他们中的大部分应该是只读的。

....


http://www.phpinternalsbook.com/php7/extensions_design/php_lifecycle.html