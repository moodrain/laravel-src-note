### PSR 11 中关于容器接口的约定

PSR 11 中定义了容器接口、容器通用异常和未找到注册异常，代码量非常少
```

interface ContainerInterface
{
    // 根据 id 取出注册在容器中的一个绑定
    public function get(string $id);

    // 判断该 id 是否在容器中注册了
    public function has(string $id);
}

// 从容器中取回绑定发生的异常
interface ContainerExceptionInterface {}

// 未在容器中找到相关注册
interface NotFoundExceptionInterface extends ContainerExceptionInterface {} //
```

### 实现一个简单的容器

```
class Container
{
    private $binds = [];

    public function bind($key, $entry)
    {
        $this->binds[$key] = $entry;
    }

    public function get($key)
    {
        if (! $this->has($key)) {
            throw new Exception('not found exception');
        }
        try {
            $toConcrete = $this->binds[$key];
            return new $toConcrete();
        } catch (Exception $e) {
            throw new Exception('container exception);
        }
    }

    public function has($key)
    {
        return isset($this->binds[$key]);
    }

}

interface CacheInterface
{
    public function get($key);
}

class RedisCache implements CacheInterface
{
    public function get($key) {
        return 'value of ' . $key;
    }
}

$container = new Container();
$container->bind(CacheInterface::class, RedisCache::class);
$cache = $container->get(CacheInterface::class);
echo $cache->get('a key');
```

这样在更换缓存实现时，只需要更换容器中关于缓存接口绑定的实现，而不用修改调用缓存的代码了。这种解耦的功能对框架来说十分重要，同时可见容器对于框架来说很重要，Laravel 中的容器远比上述复杂，功能也更加强大