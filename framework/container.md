# 容器

以分析`Laravel 5.8.14`的`Container`来理解容器的概念。

## 概念
`Container`是`Laravel`框架的核心，`Laravel`中类的实例化、存储和管理都是由`Container`负责。`Container`本质上是一个 IOC (Inversion of Control/控制反转) 容器，是用来实现依赖注入（DI/Dependency Injection）的。

整体来看，在容器中绑定各个类的抽象和实例化的方法，在需要某个类的实例时，找到抽象与实现的绑定，通过PHP的反射机制，实现容器的自动实例化类。

从`./public/index.php`开始跟踪，在`./bootstrap/app.php`中实例化`Illuminate\Foundation\Application`类，然后依次调用了`singleton()`绑定抽象类和实现，`make()`解析类的对象，最后通过解析的对象处理`Request`请求，返回`Response`。

`Illuminate\Foundation\Application`类继承自`Illuminate\Container\Container`，容器的大部分实现都在`Container`类中，所以一下源码分析都是分析此类。

## 源码

### 属性

```php
/**
* 当前类的实例
*/
protected static $instance;

/**
* 记录实例化过的$abstract ，key为$abstract，velue 为布尔值
*/
protected $resolved = [];

/**
* 容器的 bindings （绑定关系），key 为 $abstract ，value 为程序处理过的 $concrete，为一个关联数组，模型如下：
* [
*  'concrete' => Closure,
*  'shared' => bool
* ]
*/
protected $bindings = [];

/**
* The container's method bindings.
*/
protected $methodBindings = [];

/**
* 可共享的 $abstract 的实例（单例）, 键为$abstract, 值为 $abstract 对应的实例
*/
protected $instances = [];

/**
* $abstract 的别名, key 是别名, value 是$abstract
*/
protected $aliases = [];

/**
* $abstract 的别名, key 是$abstract, value 是别名
*/
protected $abstractAliases = [];

/**
* The extension closures for services.
* 扩展的 $abstract ，为一个二维数组，key 为 $abstract, vaule 为 Closure 组成的数组
* Container 支持为一个 binding 添加 extender（扩展器/装饰器）。
* extender 本质上是一个闭包函数，其接收一个 $abstract 的实例作为参数，对此实例进行包装扩展后返回。
* 用户可以在 Container 上注册 $abstract 的 extender。这样在实例化的时候， Container 会使用这些 extender 对 $abstract 的实例进行装饰扩展。
* Container 通过 extender 的设计，可以实现相当丰富的功能，比如实现装饰器模式，对 $abstract 的实例添加装饰；
* 实现代理模式，为 $abstract 的实例构造代理；实现适配器模式，基于 $abstract 的实例构造适配器等
* @var array[]
*/
protected $extenders = [];

/**
* 所有已注册的 $abstract tags
*/
protected $tags = [];

/**
* 当前正在创建的 concretions 的堆栈
*/
protected $buildStack = [];

/**
* 参数栈
*/
protected $with = [];

/**
* The contextual binding map.
* Container 支持创建基于上下文的 binding。
* Container 在构造 $abstract 的实例的时候，是利用 $abstract 对应的 $concrete 的反射类来构造 $concrete 的，
* 如果 $concrete 的构造函数不存在参数，则可以直接构造出 $concrete；
* 如果 $concrete 的构造函数存在参数，且这些参数也为某些实例对象的时候，Container 会递归构造出这些实例参数，然后构造出 $concrete。
* 在 Container 构造 $concrete 构造函数的参数对象的时候，就处于 $concrete 的上下文中。
* Container 通过使用 $buildStack 这个私有属性记录当前正在构造 $concrete 的堆栈来实现这个功能。
*/
public $contextual = [];

/**
* All of the registered rebound callbacks.
*/
protected $reboundCallbacks = [];

/**
* All of the global resolving callbacks.
*/
protected $globalResolvingCallbacks = [];

/**
* All of the global after resolving callbacks.
*/
protected $globalAfterResolvingCallbacks = [];

/**
* All of the resolving callbacks by class type.
*/
protected $resolvingCallbacks = [];

/**
* All of the after resolving callbacks by class type.
*/
protected $afterResolvingCallbacks = [];
```

### 方法


```php
/**
* 判断给定 $abstract 是否已经被绑定
*/
public function bound($abstract);

/**
* 给 $abstract 起一个别名
* 可以为 $abstract 起多个别名，甚至 a 是 b 别名，b 是 c 的别名。
* 常见的别名有：对于某个拥有父类或者实现某个接口的类的 $abstract , 将其父类和实现的接口都起成其别名。
*/
public function alias($abstract, $alias);

/**
* 给一组 $abstracts 打上一组tag
*/
public function tag($abstracts, $tags);

/**
* 实例化（解析）指定 tag 下所有 $abstract 的实例
*/
public function tagged($tag);

/**
* 在容器中注册一个 $abstract 到 $concrete 的绑定
* 
* $abstract 是要实例化类的抽象，可以是类的全局名称，也可以是接口的全局名称，还可以是给类起的一个名字。
*
* $concrete 描述一个类如何实例化的信息，可以是一个返回一个类实例的匿名函数，也可以是一个可实例化类的全局名称，
* Container 会利用反射类自动创建其构造函数所需参数，并实例化这个类。
*
* 当 $concrete 为空时，将 $concrete 设置成 $abstruct, 并根据 $abstruct 和 $concrete 的值构造闭包作为最终的 $concrete。
*/
public function bind($abstract, $concrete = null, $shared = false);

/**
* 如果 $abstract 没有被注册的话，注册一个 $abstract 到 $concrete 的绑定
*/
public function bindIf($abstract, $concrete = null, $shared = false);

/**
* 注册一个可共享的绑定到容器（单例）
*/
public function singleton($abstract, $concrete = null);

/**
* 使用 $closure 扩展容器中的 $abstract
*/
public function extend($abstract, Closure $closure);

/**
* 注册一个可共享实例（单例）到容器中
*/
public function instance($abstract, $instance);

/**
* 添加一个上下文绑定到容器中
*/
public function addContextualBinding($concrete, $abstract, $implementation);

/**
* 定义一个上下文的绑定
* 有时侯我们可能有两个类使用同一个接口，但我们希望在每个类中注入不同实现，
* 例如，两个控制器依赖 Illuminate\Contracts\Filesystem\Filesystem 契约的不同实现
*
* $this->app->when(PhotoController::class)
* ->needs(Filesystem::class)
* ->give(function () {
*      return Storage::disk('local');
* });
*
* $this->app->when(VideoController::class)
* ->needs(Filesystem::class)
* ->give(function () {
*      return Storage::disk('s3');
* });
*/
public function when($concrete);

/**
* 获取一个闭包，用来实例化给定的实现
*/
public function factory($abstract);

/**
* 清空容器中所有的绑定和实例
*/
public function flush();

/**
* 根据容器中的绑定，解析出 $abstract 对应的实现的实例
*/
public function make($abstract, array $parameters = []);

/**
* 调用给定匿名函数或者 class@method 描述的类的方法，并且自动注入依赖参数
*/
public function call($callback, array $parameters = [], $defaultMethod = null);

/**
* 判断 $abstract 是否实例化（解析）过
*/
public function resolved($abstract);

/**
* 注册一个 $abstract 实例化的回调函数
*/
public function resolving($abstract, Closure $callback = null);

/**
* 注册一个 $abstract 实例化之后的回调函数
*/
public function afterResolving($abstract, Closure $callback = null);

/**
* 创建关于 $abstract 和 $concrete 的闭包，此闭包函数封装了创建 $concrete 对应对象的方法，
* 当需要解析 $abstract 时，会调用此闭包。
*/
protected function getClosure($abstract, $concrete);

/**
* 绑定一个回调到 $abstract的rebind事件
*/
public function rebinding($abstract, Closure $callback);

/**
* 包装给定的闭包，以便在执行时注入它的依赖项
*/
public function wrap(Closure $callback, array $parameters = []);

/**
* 用给定匿名函数或者 class@method 描述的类的方法，并且自动注入依赖参数
*/
public function call($callback, array $parameters = [], $defaultMethod = null);

/**
* 解析 $abstract，实例化 $abstract 绑定的 $concrete
*/
public function make($abstract, array $parameters = []);

/**
* 实例化 $abstract 绑定的 $concrete
*
* make 方法支持对没有绑定过的 $abstruct 进行构建，
* 在 getConcrete 方法里面，如果 $abstruct 不存在绑定的 $concrete 的话，会直接返回 $abstruct。
* 这样在 make 函数里面会调用 build 方法进行构建。
*/
protected function resolve($abstract, $parameters = [], $raiseEvents = true);

/**
* 获取给定抽象的具体类型,返回 $abstract 的 $concrete
* 如果 $abstruct 不存在绑定的 $concrete 的话，会直接返回 $abstruct。
*/
protected function getConcrete($abstract);

/**
* 获取给定抽象的上下文具体绑定。
*/
protected function getContextualConcrete($abstract);

/**
* 实例化给定类型的具体实例。构建 $concrete 对应的对象
*/
public function build($concrete);

```

罗列了部分方法，看起来很多，实际只有两步：绑定、解析。

### 绑定

绑定的入口有几个：`singleton()`、`bind()`、`instance()`。

`singleton()`是直接调用的`bind()`，只是将第三个参数`$shared=false`的默认值传入的`true`，`instance()`是直接绑定`$abstract`和外部实例`$instance`，所以关于绑定，只看`bind()`方法即可。

```php
    /**
     * 在容器中注册一个 $abstract 到 $concrete 的绑定
     * 
     * $abstract 是要实例化类的抽象，可以是类的全局名称，也可以是接口的全局名称，还可以是给类起的一个名字。
     *
     * $concrete 描述一个类如何实例化的信息，可以是一个返回一个类实例的匿名函数，也可以是一个可实例化类的全局名称，
     * Container 会利用反射类自动创建其构造函数所需参数，并实例化这个类。
     *
     * 并根据 $abstruct 和 $concrete 的值构造闭包作为最终的 $concrete。
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        // 删除 $abstract 对应的已解析的实例，并且如果$abstract是一个别名的话，也删除这个别名的对应关系
        $this->dropStaleInstances($abstract);

        // 如果 $concrete 为空时，将 $concrete 设置成 $abstruct
        if (is_null($concrete)) {
            $concrete = $abstract;
        }

        // 如果 $concrete 不是闭包的话, 将其封装成一个闭包
        if (! $concrete instanceof Closure) {
            $concrete = $this->getClosure($abstract, $concrete);
        }

        // 注册 binding， ['concrete' => \Closure, 'shared' => bool]
        $this->bindings[$abstract] = compact('concrete', 'shared');

        // 如果 $abstract 已经实例化，重新实例化并调用`reboundCallbacks`
        if ($this->resolved($abstract)) {
            $this->rebound($abstract);
        }
    }
    /**
     * 创建关于 $abstract 和 $concrete 的闭包，此闭包函数封装了创建 $concrete 对应对象的方法
     */
    protected function getClosure($abstract, $concrete)
    {
        return function ($container, $parameters = []) use ($abstract, $concrete) {
            if ($abstract == $concrete) {
                //实例化 $concrete
                return $container->build($concrete);
            }
            //Note ： 这个函数调用的第一个参数是 $concrete ，而不是 $abstract
            //如果 $abstruct 不存在绑定的 $concrete，会直接返回 $abstruct，然后调用 build 方法进行构建。
            return $container->resolve(
                $concrete, $parameters, $raiseEvents = false
            );
        };
    }

    /**
     * 触发指定 abstract 的 rebound 回调
     */
    protected function rebound($abstract)
    {
        $instance = $this->make($abstract);

        foreach ($this->getReboundCallbacks($abstract) as $callback) {
            call_user_func($callback, $this, $instance);
        }
    }

```

### 解析

解析的入口有几个：`make()`、`resolve()`、`build()`。

```php
    /**
     * 解析 $abstract 绑定的 $concrete
     */
    public function make($abstract, array $parameters = [])
    {
        return $this->resolve($abstract, $parameters);
    }
    
    /**
     * 解析 $abstract 绑定的 $concrete
     * 支持对没有绑定过的 $abstruct 进行构建
     * 在 getConcrete 方法里面，如果 $abstruct 不存在绑定的 $concrete，会直接返回 $abstruct，然后调用 build 方法进行构建。
     */
    protected function resolve($abstract, $parameters = [], $raiseEvents = true)
    {
        $abstract = $this->getAlias($abstract);

        //是否需要构造上下文扩展
        $needsContextualBuild = ! empty($parameters) || ! is_null(
            $this->getContextualConcrete($abstract)
        );

        // 如果 Instance 数组里面存储有 $abstract 里面对应的对象，并且没有扩展
        if (isset($this->instances[$abstract]) && ! $needsContextualBuild) {
            return $this->instances[$abstract];
        }

        $this->with[] = $parameters;

        //获取抽象的实现
        $concrete = $this->getConcrete($abstract);

        //如果 实现 === 抽象 || 实现是一个闭包，可以构造实例
        if ($this->isBuildable($concrete, $abstract)) {
            $object = $this->build($concrete);
        } else {
            //如果 实现 != 抽象 && 实现不是闭包，将实现作为抽象进行递归，主要用与处理多层alia的情况
            $object = $this->make($concrete);
        }

        // 如果 $abstract 存在 extender , 利用这些 extender 对 $object 实例进行扩展装饰
        foreach ($this->getExtenders($abstract) as $extender) {
            $object = $extender($object, $this);
        }

        // 如果 $abstract 是可共享的，则将其放入 instances 数组中
        if ($this->isShared($abstract) && ! $needsContextualBuild) {
            $this->instances[$abstract] = $object;
        }

        //触发 resolving 回调
        if ($raiseEvents) {
            $this->fireResolvingCallbacks($abstract, $object);
        }

        //标记 $abstract 实例化过
        $this->resolved[$abstract] = true;

        //去掉参数
        array_pop($this->with);

        return $object;
    }

    /**
     * 构建 $concrete 对应的对象
     */
    public function build($concrete)
    {
        // 如果 $concrete 是一个闭包，则直接调用这个闭包方法创建对象并返回
        if ($concrete instanceof Closure) {
            return $concrete($this, $this->getLastParameterOverride());
        }

        //创建 $concrete 的反射类，并解析反射类创建对象
        $reflector = new ReflectionClass($concrete);

        //检查类是否可实例化
        if (! $reflector->isInstantiable()) {
            return $this->notInstantiable($concrete);
        }

        //加入构造栈
        $this->buildStack[] = $concrete;

        //获取构造函数的反射
        $constructor = $reflector->getConstructor();

        //如果构造函数为空，从构造栈中弹出本次构造，返回实例化的对象
        if (is_null($constructor)) {
            array_pop($this->buildStack);
            return new $concrete;
        }

        //获取实例化需要的参数，即依赖
        $dependencies = $constructor->getParameters();

        //解析依赖的对象，实例化依赖的对象
        $instances = $this->resolveDependencies(
            $dependencies
        );

        array_pop($this->buildStack);

        return $reflector->newInstanceArgs($instances);
    }

    /**
     * 获取指定抽象$abstract的具体实现$concrete
     */
    protected function getConcrete($abstract)
    {
        //如果 $abstruct 存在上下文绑定
        if (! is_null($concrete = $this->getContextualConcrete($abstract))) {
            return $concrete;
        }
       
        if (isset($this->bindings[$abstract])) {
            return $this->bindings[$abstract]['concrete'];
        }

        //如果 $abstruct 不存在绑定的 $concrete ，直接返回 $abstruct。   
        return $abstract;
    }

    /**
     * 判断指定的实现是否可构建
     */
    protected function isBuildable($concrete, $abstract)
    {
        //如果 实现 === 抽象 || 实现是一个闭包，可以构建
        return $concrete === $abstract || $concrete instanceof Closure;
    }

    /**
     * Throw an exception that the concrete is not instantiable.
     *
     * @param  string  $concrete
     * @return void
     *
     * @throws \Illuminate\Contracts\Container\BindingResolutionException
     */
    protected function notInstantiable($concrete)
    {
        if (! empty($this->buildStack)) {
            $previous = implode(', ', $this->buildStack);

            $message = "Target [$concrete] is not instantiable while building [$previous].";
        } else {
            $message = "Target [$concrete] is not instantiable.";
        }

        throw new BindingResolutionException($message);
    }


    /**
     * 从构造函数的反射参数中解析所有的依赖
     */
    protected function resolveDependencies(array $dependencies)
    {
        $results = [];

        foreach ($dependencies as $dependency) {
            //如果依赖已经在解析的参数中，直接使用
            if ($this->hasParameterOverride($dependency)) {
                $results[] = $this->getParameterOverride($dependency);
                continue;
            }

            //如果class是null，表示依赖项不是类，无法解析
            //否则就解析类
            $results[] = is_null($dependency->getClass())
                            ? $this->resolvePrimitive($dependency)
                            : $this->resolveClass($dependency);
        }

        return $results;
    }

    /**
     * 判断指定依赖是否已经被参数覆盖
     */
    protected function hasParameterOverride($dependency)
    {
        return array_key_exists(
            $dependency->name, $this->getLastParameterOverride()
        );
    }

    /**
     * 获取依赖的参数
     */
    protected function getParameterOverride($dependency)
    {
        return $this->getLastParameterOverride()[$dependency->name];
    }

    /**
     * 获取当前堆栈中最后一个参数（在make的时候推入的）
     */
    protected function getLastParameterOverride()
    {
        return count($this->with) ? end($this->with) : [];
    }

    /**
     * 解析类的依赖
     */
    protected function resolveClass(ReflectionParameter $parameter)
    {
        try {
            return $this->make($parameter->getClass()->name);
        } catch (BindingResolutionException $e) {
            //如果无法解析类，如果是可选项返回默认值，否则报错
            if ($parameter->isOptional()) {
                return $parameter->getDefaultValue();
            }

            throw $e;
        }
    }

    /**
     * 解析一个不是类的依赖
     */
    protected function resolvePrimitive(ReflectionParameter $parameter)
    {
        //如果能获取到当前依赖名称的上下文
        if (! is_null($concrete = $this->getContextualConcrete('$'.$parameter->name))) {
            return $concrete instanceof Closure ? $concrete($this) : $concrete;
        }

        //如果有默认值
        if ($parameter->isDefaultValueAvailable()) {
            return $parameter->getDefaultValue();
        }

        //抛错
        $this->unresolvablePrimitive($parameter);
    }

```

## 总结
在容器初始化后，进行了一系列的服务绑定（抽象或别名和实现的绑定），在使用的时候只需要make抽象或别名即可使用，当然也可以不提前绑定，直接调用make时会将传的第一个参数“抽象”当做实现来解析。

解析时会判断实现的构造函数是否有依赖，并通过对依赖的解析实现依赖注入。依赖的解析又走回了第一步，一次递归下去。

只简单整理了基于构造函数的服务注册、解析，关于方法中的依赖注入在依赖注入中分析。