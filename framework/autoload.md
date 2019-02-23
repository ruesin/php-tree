
# 自动加载

PHP自动加载可以通过`spl_autoload_register()`函数实现的，早年也有使用`__autoload()`的，此函数已不建议使用，在7.2.x版本中已经废弃。

`__autoload($class_name)`因为是一个函数，所以只可以定义一次。

`spl_autoload_register` — 注册给定的函数作为 __autoload 的实现，可以多次使用`spl_autoload_register`注册多个函数到SPL __autoload函数队列中。


## spl_autoload_register

`./AutoLoader.php` 自动加载类

```php
class AutoLoader
{
    private $directory;

    public function __construct()
    {
        $this->directory = __DIR__;
    }

    public static function register($prepend = false)
    {
        spl_autoload_register(array(new self(), 'autoload'), true, $prepend);
    }

    public function autoload($className)
    {
        foreach ($this->maps() as $key => $val) {
            if (0 === strpos($className, $key)) {
                $parts = explode('\\', substr($className, strlen($key)));
                $filename = $val . DIRECTORY_SEPARATOR . implode(DIRECTORY_SEPARATOR, $parts) . '.php';
                if (is_file($filename)) {
                    include $filename;
                }
                break;
            }
        }
    }

    private function maps()
    {
        return [
            'AliMns\\' => $this->directory . '/vendors/AliMns/src'
        ];
    }
}
```

`./index.php` 程序入口
```php
require __DIR__ . '/AutoLoader.php';
\Autoloader::register();

new \AliMns\Http\Client();

define('ROOT_DIR', __DIR__ . '/');

//注册第二个自动加载函数
spl_autoload_register(function ($className) {
    $filename = ROOT_DIR . '/vendors/' . implode(DIRECTORY_SEPARATOR, explode('\\', $className)) . '.php';
    include $filename;
}, true, true);

new \Mq\Client();
```

`./vendors/AliMns/src/Http/Client.php`
```php
namespace AliMns\Http;

class Client
{
    public function __construct()
    {
        echo 'AliMns Http Client' . PHP_EOL;
    }
}
```

`./vendors/Mq/Client.php`
```php
namespace Mq;

class Client
{
    public function __construct()
    {
        echo 'Mq Client' . PHP_EOL;
    }
}
```

## __autoload

`./index.php`
```php
define('ROOT_DIR', __DIR__.'/');

function __autoload($class_name) {
    $classes = explode('_',$class_name);
    if ($classes[0] == 'frame') {
        unset($classes[0]);
        $file_name = ROOT_DIR.implode('/', $classes).'.php';
        include($file_name);
    }
}

$lib = new frame_lib_payment();
$lib->type();

$model = new frame_model_payment();
$model->table();
```

```php
class frame_model_payment
{
    function table()
    {
        echo 'tb_payments'.PHP_EOL;
    }
}
```

```php
class frame_lib_payment
{
    function type()
    {
        echo 'alipay'.PHP_EOL;
    }
}
```