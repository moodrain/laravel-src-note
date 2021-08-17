上一篇 index.php 浅层的阅读，最多也就到了 ``bootstrap/app.php``，如果想要完整的梳理完请求处理的细节，可以通过 Xdebug 配合 PHPStorm 调试，以下是 Windows 系统的流程

### 下载拓展

https://xdebug.org/download 下载对应 PHP 版本的 dll 文件

### 修改 php.ini
增加以下配置项，值根据实际情况修改
```
[xdebug]
zend_extension=xdebug
xdebug.mode=debug
xdebug.client_host=127.0.0.1
xdebug.client_port=9001
```

在终端中输入 ``php -v``，如果出现 Xdebug 版本信息则表明安装成功

### 配置 PHPStorm
``Language & Framework > PHP >  Debug`` 修改 Debug port 为 9001（9000 通常为 php-fpm 监听的端口）

### 测试

``php artisan serve``

随意在 index.php 中打一个断点（在编辑器中行号和代码中间的间隔，点击左键），访问 http://127.0.0.1:8000/?XDEBUG_SESSION_START=1

> 对于其他路径的调试也一样，只需要传递 XDEBUG_SESSION_START 参数即可