### 如何引入 web.php
Laravel 有功能强大的路由，开启会话和CSRF保护等中间键的 web 路由规则写在 routes/web.php，这是一般应用最开始的地方。这个文件不是类，也不再 composer.json 的自动加载列表中，肯定在启动应用的某行代码引入进来了，使用 Xdebug 可以看到执行的顺序如下
* Kernel->handle
* Kernel->sendRequestThroughRouter
* Kernel->bootstrap
* Application->bootstrapWith
* BootProviders->bootstrap ①
* Application->boot
* Application->bootProvider
* RouteServiceProvider->boot ②
* RouteRegistrar->group
* Router->group
* Router->loadRoutes
* RouteFileRegistrar->register

之后便能看到 require 关键词，其中 $routes 是 web.php 的路径。上面能运行到 ① 是因为 BootProviders 在 Kernel 的 $bootstrappers 数组属性中，会被 Application 的 bootstrapWith 方法遍历启动。② 是因为 RouteServiceProvider 在 config/app.php 的 providers 数组中，会在 Application 的 boot 方法被遍历启动