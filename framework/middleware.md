# 中间件

在 laravel 框架中，中间件为过滤访应用的HTTP请求提供了一个方便的机制。在处理逻辑之前，会通过中间件，且只有通过了中间件才会继续执行逻辑代码。它的主要作用就是过滤Http请求(php aritsan是没有中间件机制的)，同时也让系统的层次（Http过滤层）更明确，使用起来也很优雅。

中间件本身分为两种，一种是所有http的，另一种则是针对route的。一个有中间件的请求周期是：Request得先经过Http中间件，才能进行Router，再经过Requset所对应Route的Route中间件， 最后才会进入相应的Controller代码。

`Illuminate\Pipeline\Pipeline`类是实现 laravel 中间件功能的核心。他的作用是，将一系列有序可执行的任务依次执行。

## 请求
在`./bootstrap/app.php`中绑定了`Illuminate\Contracts\Http\Kernel`抽象的实现为`App\Http\Kernel`，在`./public/index.php`中调用了`App\Http\Kernel::handle`并传入了`$request`对象。

```php
/**
 * Handle an incoming HTTP request.
 * 处理传入的HTTP请求
 * @param  \Illuminate\Http\Request  $request
 * @return \Illuminate\Http\Response
 */
public function handle($request)
{
    try {
        $request->enableHttpMethodParameterOverride();

        $response = $this->sendRequestThroughRouter($request);
    } catch (Exception $e) {
        $this->reportException($e);

        $response = $this->renderException($request, $e);
    } catch (Throwable $e) {
        $this->reportException($e = new FatalThrowableError($e));

        $response = $this->renderException($request, $e);
    }

    $this->app['events']->dispatch(
        new Events\RequestHandled($request, $response)
    );

    return $response;
}

/**
 * Send the given request through the middleware / router.
 * 将request请求传递到中间件、路由
 * @param  \Illuminate\Http\Request  $request
 * @return \Illuminate\Http\Response
 */
protected function sendRequestThroughRouter($request)
{
$this->app->instance('request', $request);

Facade::clearResolvedInstance('request');

$this->bootstrap();

return (new Pipeline($this->app))
            ->send($request)
            ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
            ->then($this->dispatchToRouter());
}
```

## 管道

Pipeline（管道）顾名思义，就是将一系列任务按一定顺序在管道里面依次执行。其中任务可以是匿名函数，也可以是拥有特定方法的类或对象。

在`Kernel`类中分别调用了`Pipeline`的三个方法：`send()`是设置了管道调用时要发送的对象，`through()`设置管道的任务数组，在`then()`方法中会依次调用。

在`then()`方法中使用了`array_reduce()`函数：

```php
array_reduce ( array $array , callable $callback [, mixed $initial = NULL ] ) : mixed
```

array_reduce() 将回调函数 callback 迭代地作用到 array 数组中的每一个单元中，从而将数组简化为单一的值。

- array：输入的 array。
- callback ( mixed $carry , mixed $item ) : mixed
    - carry：携带上次迭代里的值； 如果本次迭代是第一次，那么这个值是 initial。
    - item：携带了本次迭代的值。
- initial：如果指定了可选参数 initial，该参数将在处理开始前使用，或者当处理结束，数组为空时的最后一个结果。

返回结果值，如果`array`为空并且没有`initial`，返回结果为`NULL`。

`./src/Illuminate/Routing/Pipeline.php`

```php
<?php

namespace Illuminate\Routing;

use Closure;
use Exception;
use Throwable;
use Illuminate\Http\Request;
use Illuminate\Contracts\Debug\ExceptionHandler;
use Illuminate\Pipeline\Pipeline as BasePipeline;
use Symfony\Component\Debug\Exception\FatalThrowableError;

/**
 * This extended pipeline catches any exceptions that occur during each slice.
 *
 * The exceptions are converted to HTTP responses for proper middleware handling.
 * 管道类的扩展，捕获异常并转换为HTTP响应，以进行适当的中间件处理。
 */
class Pipeline extends BasePipeline
{
    /**
     * Get the final piece of the Closure onion.
     * 准备管道调用 array_reduce() 函数的初始值initial
     * @param  \Closure  $destination
     * @return \Closure
     */
    protected function prepareDestination(Closure $destination)
    {
        return function ($passable) use ($destination) {
            try {
                return $destination($passable);
            } catch (Exception $e) {
                return $this->handleException($passable, $e);
            } catch (Throwable $e) {
                return $this->handleException($passable, new FatalThrowableError($e));
            }
        };
    }

    /**
     * Get a Closure that represents a slice of the application onion.
     * 获取一个管道调用 array_reduce() 函数的闭包。
     * @return \Closure
     */
    protected function carry()
    {
        return function ($stack, $pipe) {
            return function ($passable) use ($stack, $pipe) {
                try {
                    $slice = parent::carry();

                    $callable = $slice($stack, $pipe);

                    return $callable($passable);
                } catch (Exception $e) {
                    return $this->handleException($passable, $e);
                } catch (Throwable $e) {
                    return $this->handleException($passable, new FatalThrowableError($e));
                }
            };
        };
    }

    /**
     * Handle the given exception.
     * 处理异常
     * @param  mixed  $passable
     * @param  \Exception  $e
     * @return mixed
     *
     * @throws \Exception
     */
    protected function handleException($passable, Exception $e)
    {
        if (! $this->container->bound(ExceptionHandler::class) ||
            ! $passable instanceof Request) {
            throw $e;
        }

        $handler = $this->container->make(ExceptionHandler::class);

        $handler->report($e);

        $response = $handler->render($passable, $e);

        if (method_exists($response, 'withException')) {
            $response->withException($e);
        }

        return $response;
    }
}
```

`./src/Illuminate/Pipeline/Pipeline.php`

```php
<?php

namespace Illuminate\Pipeline;

use Closure;
use RuntimeException;
use Illuminate\Http\Request;
use Illuminate\Contracts\Container\Container;
use Illuminate\Contracts\Support\Responsable;
use Illuminate\Contracts\Pipeline\Pipeline as PipelineContract;

class Pipeline implements PipelineContract
{
    /**
     * The container implementation.
     *
     * @var \Illuminate\Contracts\Container\Container
     */
    protected $container;

    /**
     * The object being passed through the pipeline.
     * 传入 Pipeline 任务队列的参数
     * @var mixed
     */
    protected $passable;

    /**
     * The array of class pipes.
     * 依次要执行的管道任务队列
     * @var array
     */
    protected $pipes = [];

    /**
     * The method to call on each pipe.
     * 对于类或者对象表示的任务，执行任务要调用的方法
     * @var string
     */
    protected $method = 'handle';

    /**
     * Create a new class instance.
     *
     * @param \Illuminate\Contracts\Container\Container|null $container
     * @return void
     */
    public function __construct(Container $container = null)
    {
        $this->container = $container;
    }

    /**
     * Set the object being sent through the pipeline.
     * 设置传入任务的参数
     * @param mixed $passable
     * @return $this
     */
    public function send($passable)
    {
        $this->passable = $passable;
        return $this;
    }

    /**
     * Set the array of pipes.
     * 设置任务队列
     * @param array|mixed $pipes
     * @return $this
     */
    public function through($pipes)
    {
        $this->pipes = is_array($pipes) ? $pipes : func_get_args();
        return $this;
    }

    /**
     * Set the method to call on the pipes.
     * 设置执行类任务或者对象任务的调用方法
     * @param string $method
     * @return $this
     */
    public function via($method)
    {
        $this->method = $method;
        return $this;
    }

    /**
     * Run the pipeline with a final destination callback.
     * 设置最终任务，依次执行任务队列
     * @param \Closure $destination
     * @return mixed
     */
    public function then(Closure $destination)
    {
        $pipeline = array_reduce(
            array_reverse($this->pipes), $this->carry(), $this->prepareDestination($destination)
        );
        return $pipeline($this->passable);
    }

    /**
     * Run the pipeline and return the result.
     * 执行管道并返回结果
     * @return mixed
     */
    public function thenReturn()
    {
        return $this->then(function ($passable) {
            return $passable;
        });
    }

    /**
     * Get the final piece of the Closure onion.
     * 准备管道调用 array_reduce() 函数的初始值initial
     * 对任务 $destination 使用匿名函数进行包装
     * @param \Closure $destination
     * @return \Closure
     */
    protected function prepareDestination(Closure $destination)
    {
        return function ($passable) use ($destination) {
            return $destination($passable);
        };
    }

    /**
     * Get a Closure that represents a slice of the application onion.
     * 获取一个管道调用 array_reduce() 函数的闭包。
     * @return \Closure
     */
    protected function carry()
    {
        return function ($stack, $pipe) {
            return function ($passable) use ($stack, $pipe) {
                if (is_callable($pipe)) {
                    //如果要执行的任务 $pipe 是一个匿名函数的话，立即执行这个匿名函数并返回其结果；
                    return $pipe($passable, $stack);
                } elseif (!is_object($pipe)) {
                    //如果 $pipe 不是对象的话（为字符串），将从 $pipe 中解析出来任务名称和可能存在的参数
                    [$name, $parameters] = $this->parsePipeString($pipe);
                    $pipe = $this->getContainer()->make($name);
                    $parameters = array_merge([$passable, $stack], $parameters);
                } else {
                    //如果 $pipe 是一个对象的话，构建出任务执行所需的参数
                    $parameters = [$passable, $stack];
                }
                //调用任务对象并返回其结果
                $response = method_exists($pipe, $this->method)
                    ? $pipe->{$this->method}(...$parameters)
                    : $pipe(...$parameters);

                return $response instanceof Responsable
                    ? $response->toResponse($this->getContainer()->make(Request::class))
                    : $response;
            };
        };
    }

    /**
     * Parse full pipe string to get name and parameters.
     * 将字符串管道转为管道名和参数
     * 比如中间件 throttle:60,1 的设置，解析出任务名称 throttle，参数 [60,1]
     * @param string $pipe
     * @return array
     */
    protected function parsePipeString($pipe)
    {
        [$name, $parameters] = array_pad(explode(':', $pipe, 2), 2, []);

        if (is_string($parameters)) {
            $parameters = explode(',', $parameters);
        }

        return [$name, $parameters];
    }

    /**
     * Get the container instance.
     *
     * @return \Illuminate\Contracts\Container\Container
     *
     * @throws \RuntimeException
     */
    protected function getContainer()
    {
        if (!$this->container) {
            throw new RuntimeException('A container instance has not been passed to the Pipeline.');
        }

        return $this->container;
    }
}

```

## 流程

暂且不管`Kernel`或`Router`是怎么拼装中间件的，假设现在只有两个中间件：
```php
protected $middleware = [
    \App\Http\Middleware\A::class,
    \App\Http\Middleware\B::class,
];
```

在`Kernel`中调用链如下：
```php
return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
```

- 通过`send()`方法，设置`$request`对象为传入任务的参数。
- 通过`through()`方法，设置管道任务为中间件。
- 通过`then()`方法开始执行整个管道，传入了`dispatchToRouter()`方法作为管道的初始值。

`dispatchToRouter()`返回的是一个接收`$request`的闭包。
```php
protected function dispatchToRouter()
{
    return function ($request) {
        $this->app->instance('request', $request);

        return $this->router->dispatch($request);
    };
}
```

### then()
管道的核心是`then()`方法，在此方法中，首先翻转了中间件数组，然后依次调用`carry()`方法，此方法返回的是个闭包函数，在then的最后会执行此闭包函数。

所以最后的流程为：

#### 第一次调用

调用`carry()`返回的闭包，此闭包调用后返回的依然是个闭包，同时将此闭包当做第一个参数传给下一次调用
```
function carry() {
    //$stack 为 dispatchToRouter()返回的闭包；$pipe为中间件B;
    return function ($stack, $pipe) {
        //此闭包调用后返回的依然是个闭包，同时将此闭包当做第一个参数传给下一次调用
        return function ($passable) use ($stack, $pipe) {
        };
    }
}
```

#### 第二次调用
`array_reduce()`函数的第二个参数（闭包）的两个参数分别为
- $stack 为第一次调用返回的闭包；
- $pipe 为中间件A;

#### 执行
经过两次调用后，得到的结果为一个闭包，此闭包会在`then()`方法的最后调用`$pipeline($this->passable);`
```php
function ($passable) use ($stack, $pipe) {}
```
- `$passable` 为 $request 对象
- `$stack` 为第一次调用返回的闭包；
- `$pipe` 为中间件A;

目前只分析当前闭包的对象部分，闭包、字符串等逻辑一样：
```php
//如果 $pipe 是一个对象的话，构建出任务执行所需的参数
$parameters = [$passable, $stack];
//调用任务对象并返回其结果
$response = method_exists($pipe, $this->method)
    ? $pipe->{$this->method}(...$parameters)
    : $pipe(...$parameters);
```

可见，此闭包最后调用的是`A`中间件的`handle`方法，传递的参数分别是`$request`对象，上一次调用返回的闭包。

我们看一个常见的中间件处理方法：
```php
public function handle($request, Closure $next)
{
    //TODO something
    return $next($request);
}
```
在`A`中间件中处理完逻辑后，最后返回的是调用’上一次调用返回的闭包‘，等同于`$pipeline($this->passable);`。

## 总结

中间件的整个流程可以理解为，先反向注册，然后从反向的最后开始依次调用。

注册栈：dispath()、B、A，注册完成后返回的回调为最后一个注册的A，然后在A中调用前一个B，依次执行。
