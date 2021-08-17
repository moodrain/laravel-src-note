### 判断维护模式
根据 ``maintenance`` 文件是否存在 ``file_exists(__DIR__.'/../storage/framework/maintenance.php')``，来判断是否处于维护模式（运行了 ``php artisan down``）

### 类的自动加载
引入 composer 生成的 autoload 文件，实现类的自动加载

### 新建应用实例（容器）
引入 ``bootstrap/app.php``，再看看 app.php 的内容。发现其中 new 了 ``Illuminate\Foundation\Application`` 类，这个类继承了 ``Illuminate\Container``，也就是说它是一个容器。容器的原理之后再研究，其作用大致是管理注册进来的各个类

然后往容器中绑定了三个单例 ``App\Http\Kernel`` ``App\Console\Kernel`` ``App\Exceptions\Handler``,并返回了容器到 index.php
> app.php 中的备注： 
>
> 在容器注册重要的接口来在需要时调用他们，内核需要处理应用中来自网络或终端的请求
> 
> 该脚本返回了应用实例给调用的脚本，因此可以从应用的实际运行和回复中，分离出应用实例的构建

### 生成内核实例（Kernel）
从容器中实例化 ``Illuminate\Contracts\Http\Kernel``，即刚刚上面绑定的 ``App\Http\Kernel``

### 处理并发送响应
捕获请求后传递给内核处理，并发送响应

### 处理应用结束前的中间键和回调
