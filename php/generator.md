# 生成器

(迭代)生成器就像一个普通的函数一样, 和普通函数只返回一次不同的是, 生成器可以根据需要 yield 多次，返回值是依次返回的，以便生成需要迭代的值。

生成器不是只返回一个单独的值，不需要在内存中创建一个数组, 较大的数据组会使内存达到上限、或者会占据可观的处理时间。

生成器允许你在 foreach 代码块中写代码来迭代一组数据，生成器使你能更方便的实现迭代器接口，相比较定义类实现 Iterator 接口的方式，性能开销和复杂性大大降低。

可以使用生成器来重新实现`range()`函数。标准的`range()`函数需要在内存中生成一个数组包含每一个在它范围内的值，然后返回该数组。如果调用 range(0, 1000000) 将导致内存占用超过 100 MB。

可以实现一个`xrange()`生成器, 只需要足够的内存来创建`Iterator`对象并在内部跟踪生成器的当前状态，这样只需要不到1K字节的内存。

优点是显而易见的，它可以让你在处理大数据集合的时候不用一次性的加载到内存中，甚至可以处理无限大的数据流。

当然，也可以不通过生成器来实现这个功能，而是可以通过继承`Iterator`接口实现。但通过使用生成器实现起来会更方便，不用再去实现iterator接口中的5个方法了。

```php
<?php
function xrange($start, $limit, $step = 1) {
    if ($start < $limit) {
        if ($step <= 0) {
            throw new LogicException('Step must be +ve');
        }

        for ($i = $start; $i <= $limit; $i += $step) {
            yield $i;
        }
    } else {
        if ($step >= 0) {
            throw new LogicException('Step must be -ve');
        }

        for ($i = $start; $i >= $limit; $i += $step) {
            yield $i;
        }
    }
}

foreach (xrange(1, 9, 2) as $number) {
    echo "$number ";
}
```

当第一次调用生成器函数时，将返回内部生成器类的对象。这个对象实现迭代器接口的方式，与只向前的迭代器对象非常相似，并且提供了可以调用的方法来操作生成器的状态，包括向生成器发送值和从生成器返回值。

当一个生成器被调用的时候，它返回一个可以被遍历的对象。当遍历这个对象的时候(例如通过一个foreach循环)，PHP将会在每次需要值的时候调用生成器函数，并在产生一个值之后保存生成器的状态，这样它就可以在需要产生下一个值的时候恢复调用状态。

一旦不再需要产生更多的值，生成器函数可以简单退出，而调用生成器的代码还可以继续执行，就像一个数组已经被遍历完了。

## yeild

生成器函数的核心是`yield`关键字。它最简单的调用形式看起来像一个return申明，不同之处在于普通return会返回值并终止函数的执行，而yield会返回一个值给循环调用此生成器的代码并且只是暂停执行生成器函数。

生成器是一种可中断的函数, 在它里面的yield构成了中断点。

调用迭代器的方法一次, 其中的代码运行一次。例如, 如果调用`$range->rewind()`, 那么`xrange()`里的代码就会运行到控制流第一次出现yield的地方。而函数内传递给yield语句的返回值可以通过`$range->current()`获取。

为了继续执行生成器中yield后的代码, 需要调用`$range->next()`方法。这将再次启动生成器, 直到下一次yield语句出现. 因此，连续调用`next()`和`current()`方法，就能从生成器里获得所有的值, 直到再没有yield语句出现。

对`xrange()`来说，这种情形出现在`$i`超过`$end`时。在这种情况下，控制流将到达函数的终点，因此将不执行任何代码。一旦这种情况发生，`vaild()`方法将返回假，这时迭代结束。

```php
<?php
function gen() {
    yield "Foo";
    yield "Bar";
}
$gen = gen();
echo $gen->current();
$gen->next();
echo $gen->current();
```

## 生成器类

Generator 对象是从 generators返回的。当调用生成器函数时，将返回内部生成器类的对象。

Generator 对象不能通过 new 实例化，只能通过调用生成器函数返回。

Generator 实现了 Iterator 接口：
- current() ：返回当前产生的值
- key() ：返回当前产生的键
- next() ：生成器继续执行
- rewind() ：重置迭代器
- send() ：向生成器中传入一个值
- throw() ：向生成器中抛入一个异常
- valid() ：检查迭代器是否被关闭
- __wakeup() ：序列化回调

需要注意的是`send()`方法，向生成器中传入一个值，当做 yield 表达式的结果，然后继续执行生成器。

如果当这个方法被调用时，生成器不在 yield 表达式，那么在传入值之前，它会先运行到第一个 yield 表达式。因此，不需要使用`next()`调用“启动”PHP生成器(就像在Python中所做的那样)。

当迭代器对象被foreach遍历的时候，`valid()`、`current()`、`key()`方法会依次被调用，其返回值便是foreach语句的key和value。循环的终止条件则根据`valid()`方法的返回而定。如果返回的是true则继续循环，如果是false则终止整个循环，结束遍历。当一次循环体结束之后，将调用`next()`进行下一次的循环直到`valid()`返回false。`rewind()`方法在整个循环开始前被调用，这样保证了我们多次遍历得到的结果都是一致的。

- yield只能用于函数内部，在非函数内部运用会抛出错误。
- 如果函数包含了yield关键字的，那么函数执行后的返回值永远都是一个Generator对象。
- 如果函数内部同事包含yield和return 该函数的返回值依然是Generator对象，但是在生成Generator对象时，return语句后的代码被忽略。
- Generator类实现了Iterator接口。
- 可以通过返回的Generator对象内部的方法，获取到函数内部yield后面表达式的值。
- 可以通过Generator的send方法给yield 关键字赋一个值。
- 一旦返回的Generator对象被遍历完成，便不能调用他的rewind方法来重置
- Generator对象不能被clone关键字克隆

## 协程

### 通信
协程的支持是在迭代生成器的基础上，关键是迭代器的`send()`方法，增加了可以回送数据给生成器的功能(调用者发送数据给被调用的生成器函数)。这就把生成器到调用者的单向通信转变为两者之间的双向通信。

向生成器传值：
```php
<?php
function logger($fileName) {
    $fileHandle = fopen($fileName, 'a');
    while (true) {
        //yield没有作为一个语句来使用，而是用作一个表达式，即它能被演化成一个值。这个值就是调用者传递给`send()`方法的值。
        fwrite($fileHandle, yield . "\n");
    }
}
 
$logger = logger(__DIR__ . '/log');
$logger->send('Foo'); //yield表达式被”Foo”替代写入Log
$logger->send('Bar'); //yield表达式被”Bar”替代写入Log
```

双向通信：
```php
<?php
function gen() {
    $ret = (yield 'yield1');
    var_dump($ret);
    $ret = (yield 'yield2');
    var_dump($ret);
}
 
$gen = gen();
var_dump($gen->current());    // string(6) "yield1"
var_dump($gen->send('ret1')); // string(4) "ret1"   (the first var_dump in gen)
                              // string(6) "yield2" (the var_dump of the ->send() return value)
var_dump($gen->send('ret2')); // string(4) "ret2"   (again from within gen)
                              // NULL               (the return value of ->send())
```

`send()`方法，向`yield`语句处传递一个值，同时从 yied 语句处继续执行，直至再次遇到 yield 后控制权回到外部。

yield表达式两边的括号在PHP7以前不是可选的，也就是说在PHP5.5和PHP5.6中圆括号是必须的。

生成迭代对象的时候已经隐含地执行了rewind操作，调用current()之前不用调用rewind()。

```php
function gen() {
    yield 'foo';
    yield 'bar';
}
//在send之前，当$gen迭代器被创建的时候一个renwind()方法已经被隐式调用
$gen = gen(); //$gen->rewind();
//var_dump($gen->current());
// renwind的执行将会重置生成器，send方法从第一个yield执行，并且忽略了他的返回值，此时得到的是第二个yield的值！导致第一个yield的值被忽略。
var_dump($gen->send('something'));
```

### 多任务协作
使用协程实现多任务协作，要解决的问题是如何并发地运行多任务(或者“程序”）。

CPU在一个时刻只能运行一个任务（不考虑多核的情况），因此处理器需要在不同的任务之间进行切换，而且总是让每个任务运行 “一小会儿”。

多任务协作这个术语中的“协作”很好的说明了如何进行这种切换的：它要求当前正在运行的任务自动把控制传回给调度器，这样就可以运行其他任务了。这与“抢占”多任务相反，抢占多任务是这样的：调度器可以中断运行了一段时间的任务，不管它喜欢还是不喜欢。

协作多任务在Windows的早期版本(windows95)和Mac OS中有使用，不过它们后来都切换到使用抢先多任务了。理由相当明确：如果你依靠程序自动交出控制的话，那么一些恶意的程序将很容易占用整个CPU，不与其他任务共享。

协程和任务调度之间的关系：yield指令提供了任务中断自身的一种方法，然后把控制交回给任务调度器。因此协程可以运行多个其他任务。更进一步来说，yield还可以用来在任务和调度器之间进行通信。

#### 任务类

```php
<?php
class Task {
    // 一个任务就是用任务ID标记的一个协程(函数)
    protected $taskId;
    protected $coroutine;
    protected $sendValue = null;
    //通过`beforeFirstYieldcondition`，可以确定第一个yield的值能被正确返回。
    protected $beforeFirstYield = true;
 
    public function __construct($taskId, Generator $coroutine) {
        $this->taskId = $taskId;
        $this->coroutine = $coroutine;
    }
 
    public function getTaskId() {
        return $this->taskId;
    }
 
    // 指定哪些值将被发送到下次的恢复
    public function setSendValue($sendValue) {
        $this->sendValue = $sendValue;
    }
 
    public function run() {
        if ($this->beforeFirstYield) {
            $this->beforeFirstYield = false;
            return $this->coroutine->current();
        } else {
            //调用`send()`方法的协同程序
            $retval = $this->coroutine->send($this->sendValue);
            $this->sendValue = null;
            return $retval;
        }
    }
 
    public function isFinished() {
        return !$this->coroutine->valid();
    }
}
```

#### 调度器
```php
<?php
class Scheduler {
    protected $maxTaskId = 0;
    protected $taskMap = []; // taskId => task
    protected $taskQueue;
 
    public function __construct() {
        $this->taskQueue = new SplQueue();
    }
 
    //使用下一个空闲的任务id，创建一个新任务，然后把这个任务放入任务map数组里
    public function newTask(Generator $coroutine) {
        $tid = ++$this->maxTaskId;
        $task = new Task($tid, $coroutine);
        $this->taskMap[$tid] = $task;
        $this->schedule($task);
        return $tid;
    }
 
    //通过把任务放入任务队列里来实现对任务的调度
    public function schedule(Task $task) {
        $this->taskQueue->enqueue($task);
    }
 
    //扫描任务队列，运行任务
    public function run() {
        while (!$this->taskQueue->isEmpty()) {
            $task = $this->taskQueue->dequeue();
            $task->run();
            //如果一个任务结束了，那么它将从队列里删除，否则它将在队列的末尾再次被调度。
            if ($task->isFinished()) {
                unset($this->taskMap[$task->getTaskId()]);
            } else {
                $this->schedule($task);
            }
        }
    }
}
```

示例：
```php
<?php
function task1() {
    for ($i = 1; $i <= 10; ++$i) {
        echo "This is task 1 iteration $i.\n";
        yield;
    }
}

function task2() {
    for ($i = 1; $i <= 5; ++$i) {
        echo "This is task 2 iteration $i.\n";
        yield;
    }
}
 
$scheduler = new Scheduler;
 
$scheduler->newTask(task1());
$scheduler->newTask(task2());
 
$scheduler->run();
```

#### 任务与调度器通信
使用进程用来和操作系统会话的同样的方式来通信：系统调用。

使用系统调用的理由是操作系统与进程相比它处在不同的权限级别上。

为了执行特权级别的操作（如杀死另一个进程)，就不得不以某种方式把控制传回给内核，这样内核就可以执行所说的操作了。这种行为在内部是通过使用中断指令来实现的。过去使用的是通用的int指令，如今使用的是更特殊并且更快速的syscall/sysenter指令。

我们的任务调度系统将反映这种设计：不是简单地把调度器传递给任务（这样就允许它做它想做的任何事)，我们将通过给yield表达式传递信息来与系统调用通信。这儿yield即是中断，也是传递信息给调度器（和从调度器传递出信息）的方法。

为了说明系统调用, 我们对可调用的系统调用做一个小小的封装：
```php
<?php
class SystemCall {
    protected $callback;
 
    public function __construct(callable $callback) {
        $this->callback = $callback;
    }
 
    public function __invoke(Task $task, Scheduler $scheduler) {
        $callback = $this->callback;
        return $callback($task, $scheduler);
    }
}
```
它和其他任何可调用的对象(使用_invoke)一样的运行, 不过它要求调度器把正在调用的任务和自身传递给这个函数.

修改调度器的run方法：
```php
<?php
public function run() {
    while (!$this->taskQueue->isEmpty()) {
        $task = $this->taskQueue->dequeue();
        $retval = $task->run();
 
        if ($retval instanceof SystemCall) {
            $retval($task, $this);
            continue;
        }
 
        if ($task->isFinished()) {
            unset($this->taskMap[$task->getTaskId()]);
        } else {
            $this->schedule($task);
        }
    }
}
```

```php
<?php
//这个函数设置任务id为下一次发送的值, 并再次调度了这个任务。
function getTaskId() {
    return new SystemCall(function(Task $task, Scheduler $scheduler) {
        $task->setSendValue($task->getTaskId());
        $scheduler->schedule($task);
    });
}
```
由于使用了系统调用，所以调度器不能自动调用任务，我们需要手工调度任务。
示例：
```php
<?php
function task($max) {
    $tid = (yield getTaskId()); // <-- here's the syscall!
    for ($i = 1; $i <= $max; ++$i) {
        echo "This is task $tid iteration $i.\n";
        yield;
    }
}
 
$scheduler = new Scheduler;
 
$scheduler->newTask(task(10));
$scheduler->newTask(task(5));
 
$scheduler->run();
```

这段代码将给出与前一个例子相同的输出。请注意系统调用如何同其他任何调用一样正常地运行，只不过预先增加了yield。

要创建新的任务，然后再杀死它们的话，需要两个以上的系统调用：
```php
<?php
function newTask(Generator $coroutine) {
    return new SystemCall(
        function(Task $task, Scheduler $scheduler) use ($coroutine) {
            $task->setSendValue($scheduler->newTask($coroutine));
            $scheduler->schedule($task);
        }
    );
}
 
function killTask($tid) {
    return new SystemCall(
        function(Task $task, Scheduler $scheduler) use ($tid) {
            $task->setSendValue($scheduler->killTask($tid));
            $scheduler->schedule($task);
        }
    );
}
```

killTask函数需要在调度器里增加一个方法：
```php
<?php
public function killTask($tid) {
    if (!isset($this->taskMap[$tid])) {
        return false;
    }
 
    unset($this->taskMap[$tid]);
 
    // This is a bit ugly and could be optimized so it does not have to walk the queue,
    // but assuming that killing tasks is rather rare I won't bother with it now
    foreach ($this->taskQueue as $i => $task) {
        if ($task->getTaskId() === $tid) {
            unset($this->taskQueue[$i]);
            break;
        }
    }
 
    return true;
}
```

示例：
```php
<?php
function childTask() {
    $tid = (yield getTaskId());
    while (true) {
        echo "Child task $tid still alive!\n";
        yield;
    }
}
 
function task() {
    $tid = (yield getTaskId());
    $childTid = (yield newTask(childTask()));
 
    for ($i = 1; $i <= 6; ++$i) {
        echo "Parent task $tid iteration $i.\n";
        yield;
 
        if ($i == 3) yield killTask($childTid);
    }
}
 
$scheduler = new Scheduler;
$scheduler->newTask(task());
$scheduler->run();
```

经过三次迭代以后子任务将被杀死，因此这就是”Child is still alive”消息结束的时候。但这还不是真正的父子关系，因为在父任务结束后子任务仍然可以运行，子任务甚至可以杀死父任务，可以修改调度器使它具有更层级化的任务结构。

现在你可以实现许多进程管理调用。例如 wait（它一直等待到任务结束运行时)，exec（它替代当前任务)和fork（它创建一个当前任务的克隆)。fork非常酷，而且你可以使用PHP的协程真正地实现它，因为它们都支持克隆。

### 非阻塞IO
很明显，我们的任务管理系统的真正很酷的应用应该是web服务器。它有一个任务是在套接字上侦听是否有新连接，当有新连接要建立的时候，创建一个新任务来处理新连接。

Web服务器最难的部分通常是像读数据这样的套接字操作是阻塞的。例如PHP将等待到客户端完成发送为止。对一个Web服务器来说，这有点不太高效。因为服务器在一个时间点上只能处理一个连接。

解决方案是确保在真正对套接字读写之前该套接字已经“准备就绪”。为了查找哪个套接字已经准备好读或者写了，可以使用`流选择函数`。

添加两个新的 syscall, 它们将等待直到指定socket 准备好：
```php
<?php
function waitForRead($socket) {
    return new SystemCall(
        function(Task $task, Scheduler $scheduler) use ($socket) {
            $scheduler->waitForRead($socket, $task);
        }
    );
}
 
function waitForWrite($socket) {
    return new SystemCall(
        function(Task $task, Scheduler $scheduler) use ($socket) {
            $scheduler->waitForWrite($socket, $task);
        }
    );
}
```

这些 syscall 只是在调度器中代理其各自的方法：

```php
<?php
// resourceID => [socket, tasks]
protected $waitingForRead = [];
protected $waitingForWrite = [];
 
public function waitForRead($socket, Task $task) {
    if (isset($this->waitingForRead[(int) $socket])) {
        $this->waitingForRead[(int) $socket][1][] = $task;
    } else {
        $this->waitingForRead[(int) $socket] = [$socket, [$task]];
    }
}
 
public function waitForWrite($socket, Task $task) {
    if (isset($this->waitingForWrite[(int) $socket])) {
        $this->waitingForWrite[(int) $socket][1][] = $task;
    } else {
        $this->waitingForWrite[(int) $socket] = [$socket, [$task]];
    }
}
```
`waitingForRead`及`waitingForWrite`属性是两个承载等待的`socket`及等待它们的任务的数组。

有趣的部分在于下面的方法，它将检查`socket`是否可用，并重新安排各自任务：
```php
<?php
 
protected function ioPoll($timeout) {
    $rSocks = [];
    foreach ($this->waitingForRead as list($socket)) {
        $rSocks[] = $socket;
    }
 
    $wSocks = [];
    foreach ($this->waitingForWrite as list($socket)) {
        $wSocks[] = $socket;
    }
 
    $eSocks = []; // dummy
 
    if (!stream_select($rSocks, $wSocks, $eSocks, $timeout)) {
        return;
    }
 
    foreach ($rSocks as $socket) {
        list(, $tasks) = $this->waitingForRead[(int) $socket];
        unset($this->waitingForRead[(int) $socket]);
 
        foreach ($tasks as $task) {
            $this->schedule($task);
        }
    }
 
    foreach ($wSocks as $socket) {
        list(, $tasks) = $this->waitingForWrite[(int) $socket];
        unset($this->waitingForWrite[(int) $socket]);
 
        foreach ($tasks as $task) {
            $this->schedule($task);
        }
    }
}
```

`stream_select`函数接受承载读取、写入以及待检查的socket的数组（我们无需考虑最后一类)。数组将按引用传递，函数只会保留那些状态改变了的数组元素。我们可以遍历这些数组，并重新安排与之相关的任务。

为了正常地执行上面的轮询动作, 我们将在调度器里增加一个特殊的任务：
```php
<?php
protected function ioPollTask() {
    while (true) {
        if ($this->taskQueue->isEmpty()) {
            $this->ioPoll(null);
        } else {
            $this->ioPoll(0);
        }
        yield;
    }
}
```

需要在某个地方注册这个任务，例如可以在`run()`方法的开始增加`$this->newTask($this->ioPollTask())`。然后就像其他任务一样每执行完整任务循环一次就执行轮询操作一次（这么做一定不是最好的方法)，`ioPollTask`将使用0秒的超时来调用`ioPoll`，也就是`stream_select`将立即返回（而不是等待）。

只有任务队列为空时，才使用null超时，这意味着它一直等到某个套接口准备就绪。如果没有这么做，那么轮询任务将一而再，再而三的循环运行，直到有新的连接建立，这将导致100%的CPU利用率。相反，让操作系统做这种等待会更有效。

现在编写服务器就相对容易了：
```php
<?php
 
function server($port) {
    echo "Starting server at port $port...\n";
 
    $socket = @stream_socket_server("tcp://localhost:$port", $errNo, $errStr);
    if (!$socket) throw new Exception($errStr, $errNo);
 
    stream_set_blocking($socket, 0);
 
    while (true) {
        yield waitForRead($socket);
        $clientSocket = stream_socket_accept($socket, 0);
        yield newTask(handleClient($clientSocket));
    }
}
 
function handleClient($socket) {
    yield waitForRead($socket);
    $data = fread($socket, 8192);
 
    $msg = "Received following request:\n\n$data";
    $msgLength = strlen($msg);
 
    $response = <<<res
HTTP/1.1 200 OK\r
Content-Type: text/plain\r
Content-Length: $msgLength\r
Connection: close\r
\r
$msg
RES;
 
    yield waitForWrite($socket);
    fwrite($socket, $response);
 
    fclose($socket);
}
 
$scheduler = new Scheduler;
$scheduler->newTask(server(8000));
$scheduler->run();
```

这段代码实现了接收`localhost:8000`上的连接，然后返回发送来的内容作为HTTP响应。当然它还能处理真正的复杂HTTP请求，上面的代码片段只是演示了一般性的概念。

可以使用类似于`ab -n 10000 -c 100 localhost:8000/`这样命令来测试服务器。这条命令将向服务器发送10000个请求，并且其中100个请求将同时到达。使用这样的数目，我得到了处于中间的10毫秒的响应时间。不过还有一个问题：有少数几个请求真正处理的很慢（如5秒)，这就是为什么总吞吐量只有2000请求/秒（如果是10毫秒的响应时间的话, 总的吞吐量应该更像是10000请求/秒)。

### 协程堆栈

如果你试图用我们的调度系统建立更大的系统的话，你将很快遇到问题：我们习惯了把代码分解为更小的函数，然后调用它们。然而，如果使用了协程的话，就不能这么做了。

例如：
```php
<?php
function echoTimes($msg, $max) {
    for ($i = 1; $i <= $max; ++$i) {
        echo "$msg iteration $i\n";
        yield;
    }
}
 
function task() {
    echoTimes('foo', 10); // print foo ten times
    echo "---\n";
    echoTimes('bar', 5); // print bar five times
    yield; // force it to be a coroutine
}
 
$scheduler = new Scheduler;
$scheduler->newTask(task());
$scheduler->run();
```

这段代码试图把重复循环“输出n次“的代码嵌入到一个独立的协程里，然后从主任务里调用它。然而它无法运行，正如在这篇文章的开始所提到的，调用生成器（或者协程）将没有真正地做任何事情，它仅仅返回一个对象。这也出现在上面的例子里：echoTimes调用除了放回一个（无用的）协程对象外不做任何事情。

为了仍然允许这么做，我们需要在这个裸协程上写一个小小的封装。我们将调用它：“协程堆栈”。因为它将管理嵌套的协程调用堆栈。这将是通过生成协程来调用子协程成为可能：
```php
$retval = (yield someCoroutine($foo, $bar));
```
使用yield，子协程也能再次返回值：
```php
yield retval("I'm a return value!");
```

```php
<?php
 
class CoroutineReturnValue {
    protected $value;
 
    public function __construct($value) {
        $this->value = $value;
    }
 
    public function getValue() {
        return $this->value;
    }
}
 
//retval函数除了返回一个值的封装外没有做任何其他事情。这个封装将表示它是一个返回值。
function retval($value) {
    return new CoroutineReturnValue($value);
}
```

为了把协程转变为协程堆栈（它支持子调用），我们将不得不编写另外一个函数（很明显，它是另一个协程）：

```php
<?php
 
function stackedCoroutine(Generator $gen) {
    $stack = new SplStack;
 
    for (;;) {
        $value = $gen->current();
 
        if ($value instanceof Generator) {
            $stack->push($gen);
            $gen = $value;
            continue;
        }
 
        $isReturnValue = $value instanceof CoroutineReturnValue;
        if (!$gen->valid() || $isReturnValue) {
            if ($stack->isEmpty()) {
                return;
            }
 
            $gen = $stack->pop();
            $gen->send($isReturnValue ? $value->getValue() : NULL);
            continue;
        }
 
        $gen->send(yield $gen->key() => $value);
    }
}
```

这个函数在调用者和当前正在运行的子协程之间扮演着简单代理的角色。在`$gen->send(yield $gen->key()=>$value);`这行完成了代理功能。

另外它检查返回值是否是生成器，万一是生成器的话，它将开始运行这个生成器，并把前一个协程压入堆栈里。

一旦它获得了CoroutineReturnValue的话，它将再次请求堆栈弹出，然后继续执行前一个协程。

为了使协程堆栈在任务里可用，任务构造器里的`$this->coroutine =$coroutine;`这行需要替代为`$this->coroutine = StackedCoroutine($coroutine);`。

现在我们可以稍微改进上面web服务器例子：把wait+read(和wait+write和warit+accept)这样的动作分组为函数。为了分组相关的功能，我将使用下面类：

```php
<?php
 
class CoSocket {
    protected $socket;
 
    public function __construct($socket) {
        $this->socket = $socket;
    }
 
    public function accept() {
        yield waitForRead($this->socket);
        yield retval(new CoSocket(stream_socket_accept($this->socket, 0)));
    }
 
    public function read($size) {
        yield waitForRead($this->socket);
        yield retval(fread($this->socket, $size));
    }
 
    public function write($string) {
        yield waitForWrite($this->socket);
        fwrite($this->socket, $string);
    }
 
    public function close() {
        @fclose($this->socket);
    }
}
```
现在服务器可以编写的稍微简洁点了：
```php
<?php
 
function server($port) {
    echo "Starting server at port $port...\n";
 
    $socket = @stream_socket_server("tcp://localhost:$port", $errNo, $errStr);
    if (!$socket) throw new Exception($errStr, $errNo);
 
    stream_set_blocking($socket, 0);
 
    $socket = new CoSocket($socket);
    while (true) {
        yield newTask(
            handleClient(yield $socket->accept())
        );
    }
}
 
function handleClient($socket) {
    $data = (yield $socket->read(8192));
 
    $msg = "Received following request:\n\n$data";
    $msgLength = strlen($msg);
 
    $response = <<<res
HTTP/1.1 200 OK\r
Content-Type: text/plain\r
Content-Length: $msgLength\r
Connection: close\r
\r
$msg
RES;
 
    yield $socket->write($response);
    yield $socket->close();
}
```

### 错误处理
作为一个优秀的程序员，相信你已经察觉到上面的例子缺少错误处理。几乎所有的`socket`都是易出错的。我没有这样做的原因一方面固然是因为错误处理的乏味（特别是 socket)，另一方面也在于它很容易使代码体积膨胀。

不过，我仍然想讲下常见的协程错误处理：协程允许使用 throw() 方法在其内部抛出一个错误。

`throw()`方法接受一个`Exception`，并将其抛出到协程的当前悬挂点，看看下面代码：
```php
<?php
function gen() {
    echo "Foo\n";
    try {
        yield;
    } catch (Exception $e) {
        echo "Exception: {$e->getMessage()}\n";
    }
    echo "Bar\n";
}
 
$gen = gen();
$gen->rewind();                     // echos "Foo"
$gen->throw(new Exception('Test')); // echos "Exception: Test"
                                    // and "Bar"
```
这非常好, 有没有? 因为我们现在可以使用系统调用以及子协程调用异常抛出了.

不过我们要对系统调用Scheduler::run() 方法做一些小调整：
```php
<?php
if ($retval instanceof SystemCall) {
    try {
        $retval($task, $this);
    } catch (Exception $e) {
        $task->setException($e);
        $this->schedule($task);
    }
    continue;
}
```
Task 类也要添加 throw 调用处理：
```php
<?php
class Task {
    // ...
    protected $exception = null;
 
    public function setException($exception) {
        $this->exception = $exception;
    }
 
    public function run() {
        if ($this->beforeFirstYield) {
            $this->beforeFirstYield = false;
            return $this->coroutine->current();
        } elseif ($this->exception) {
            $retval = $this->coroutine->throw($this->exception);
            $this->exception = null;
            return $retval;
        } else {
            $retval = $this->coroutine->send($this->sendValue);
            $this->sendValue = null;
            return $retval;
        }
    }
 
    // ...
}
```
现在, 我们已经可以在系统调用中使用异常抛出了！例如,要调用 killTask,让我们在传递 ID 不可用时抛出一个异常：
```php
<?php
function killTask($tid) {
    return new SystemCall(
        function(Task $task, Scheduler $scheduler) use ($tid) {
            if ($scheduler->killTask($tid)) {
                $scheduler->schedule($task);
            } else {
                throw new InvalidArgumentException('Invalid task ID!');
            }
        }
    );
}
```

试试看：
```php
<?php
function task() {
    try {
        yield killTask(500);
    } catch (Exception $e) {
        echo 'Tried to kill task 500 but failed: ', $e->getMessage(), "\n";
    }
}
```
这些代码现在尚不能正常运作,因为 stackedCoroutine 函数无法正确处理异常.要修复需要做些调整：
```php
<?php
function stackedCoroutine(Generator $gen) {
    $stack = new SplStack;
    $exception = null;
 
    for (;;) {
        try {
            if ($exception) {
                $gen->throw($exception);
                $exception = null;
                continue;
            }
 
            $value = $gen->current();
 
            if ($value instanceof Generator) {
                $stack->push($gen);
                $gen = $value;
                continue;
            }
 
            $isReturnValue = $value instanceof CoroutineReturnValue;
            if (!$gen->valid() || $isReturnValue) {
                if ($stack->isEmpty()) {
                    return;
                }
 
                $gen = $stack->pop();
                $gen->send($isReturnValue ? $value->getValue() : NULL);
                continue;
            }
 
            try {
                $sendValue = (yield $gen->key() => $value);
            } catch (Exception $e) {
                $gen->throw($e);
                continue;
            }
 
            $gen->send($sendValue);
        } catch (Exception $e) {
            if ($stack->isEmpty()) {
                throw $e;
            }
 
            $gen = $stack->pop();
            $exception = $e;
        }
    }
}
```