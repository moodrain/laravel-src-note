### 从接口分析

Pipeline 实现了 Pipeline 契约接口，接口中只有 4 个方法：send、through、via、then，逐个查看源码的注释。
```
send($traveler)    // 为管道设置需要传递的对象
through($stops)    // 设置管道的处理点
via($method)       // 设置处理点的处理的方法（字符串类型）
then($destination) // 为管道设置目标容器并运行管道
```
从注释分析，该模式通过 send 传入一个对象，然后 through 一系列节点，并通过 via 设置的方法对经过一系列节点对对象进行处理，最后经由 then 的设置的容器输出结果


### 分析 then 方法
接口写的简单，但 Laravel 的实现比较绕
```
public function then(Closure $destination)
{
    $pipeline = array_reduce(
        array_reverse($this->pipes()), $this->carry(), $this->prepareDestination($destination)
    );
    return $pipeline($this->passable);
}
```
``array_reduce($array, $callback, $init)`` 函数的用法是将 $init 作为初始值，不断的与 $array 中的元素，通过 $callback 的归纳处理得出结果。在这里需要处理的数组是 ``$this->pipes()`` 返回的全局中间件数组，至于为什么要用 array_reverse 倒序后面就知道了

### 分析 prepareDestination 方法
```
protected function prepareDestination(Closure $destination)
{
    return function ($passable) use ($destination) {
        try {
            return $destination($passable);
        } catch (Throwable $e) {
            return $this->handleException($passable, $e);
        }
    };
}
```
这里先分析占 $initial 位置的 ``$this->prepareDestination($destination)``，查看 Router 中关于 Pipeline 的使用
```
(new Pipeline($this->container))
    ->send($request)
    ->through($middleware)
    ->then(function ($request) use ($route) {
        return $this->prepareResponse(
            $request, $route->run()
        );
    });
```
在 $destination 传的是一个闭包，传入请求运行路由得到响应，传到 prepareDestination 后再包了一层，实现了异常处理

### 分析 carry 方法
```
protected function carry()
{
    return function ($stack, $pipe) {
        return function ($passable) use ($stack, $pipe) {
            try {
                if (is_callable($pipe)) {
                    return $pipe($passable, $stack);
                } elseif (! is_object($pipe)) {
                    [$name, $parameters] = $this->parsePipeString($pipe);
                    $pipe = $this->getContainer()->make($name);
                    $parameters = array_merge([$passable, $stack], $parameters);
                } else {
                    $parameters = [$passable, $stack];
                }
                $carry = method_exists($pipe, $this->method)
                                ? $pipe->{$this->method}(...$parameters)
                                : $pipe(...$parameters);
                return $this->handleCarry($carry);
            } catch (Throwable $e) {
                return $this->handleException($passable, $e);
            }
        };
    };
}
```
该方法初次看很复杂，连续返回了两次闭包，首先第一层返回的是 array_reduce 归纳时实际要执行。第一次归纳时 $stack 是 prepareDestination 返回的闭包， $pipe 是中间件数组中最后一个。不看第二层闭包内部，那么 array_reduce 归纳的结果将会是这个第二层的闭包 $func($passable){...}

再看 carry 内部，$pipe 是 $stop 的类名，会进入到第二块代码，解析出中间件名称和参数，再从容器中取得 $stop 对象，把参数拼接成：``[$traveler, $destination, $parameters]`` 的形式，传递到的处理方法中（一般是 handle，也可以通过 via 方法设置）

### 中间件的 handle 方法
array_reduce 中处理 $stops 之前先将其倒序，这是必要的，先创建两个全局中间件
```
php artisan make:middleware TestMiddlewareOne
php artisan make:middleware TestMiddlewareTwo
```
并加入到 Kernel 的 middleware 数组的后面，修改它们的 handle 方法，输出它们的最后三个字母，访问应用，发现输出是 OneTwo，符合数组里的顺序。
```
public function handle(Request $request, Closure $next)
{
    echo 'Two';
    return $next($request);
}
```
返回的是 $stack($request) 即 $destination($request)，最终回到 array_reduce 的是 
```
func($passable) {
    ...
    echo 'Two';
    $destination($request);
}
```
然后参与到下一轮 array_reduce 的归纳， carry 里的闭包 $stack 和 $pipe 依次是 上一个闭包， TestMiddlewareOne::class，同理最终返回 array_reduce 的是
```
func($passable) {
    ...
    echo 'One';
    func($passable) {
        ...
        echo 'Two';
        $destination($request);
    }   
}
```

这样倒序传入的中间件数组在最后作为闭包返回，执行的时候是顺序。并且每一步都包装了异常处理，中间件中的异常能转换为适当的错误响应