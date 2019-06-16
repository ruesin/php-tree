# 魔术方法

PHP 将所有以 __（两个下划线）开头的类方法保留为魔术方法。所以在定义类方法时，除了这些魔术方法，建议不要以 __ 为前缀。

## __construct()

类的构造函数，对象创建完成后第一个被对象自动调用的方法。在每个类中都有一个构造方法，如果没有显示地声明它，那么类中都会默认存在一个没有参数且内容为空的构造方法。

父类的构造函数可以被子类继承和重写，如果子类中定义了构造函数则不会隐式调用其父类的构造函数。要执行父类的构造函数，需要在子类的构造函数中调用 parent::__construct()。

当 __construct() 被与父类 __construct() 具有不同参数的方法覆盖时，PHP 不会产生一个 E_STRICT 错误信息。

如果子类没有定义构造函数则会如同一个普通的类方法一样从父类继承（假如没有被定义为 private 的话）。

可以为构造函数定义任意多个参数，只要在实例化时传入对应个数的参数即可。构造函数中出现的任何异常都会阻止对象的创建。

将构造函数声明为私有方法，可以防止在类外部创建对象，这在单例模式中经常使用。

## __destruct()

类的析构函数，会在到某个对象的所有引用都被删除或者当对象被显式销毁时执行。

父类的析构函数不会被引擎暗中调用。要执行父类的析构函数，必须在子类的析构函数体中显式调用 parent::__destruct()。

子类如果自己没有定义析构函数则会继承父类的。

析构函数不能带有任何参数。

析构函数即使在使用 exit() 终止脚本运行时也会被调用。在析构函数中调用 exit() 将会中止其余关闭操作的运行。

析构函数在脚本关闭时调用，此时所有的 HTTP 头信息已经发出。脚本关闭时的工作目录有可能和在 SAPI（如 apache）中时不同。

试图在析构函数（在脚本终止时被调用）中抛出一个异常会导致致命错误。

## __call()

```php
public mixed __call ( string $name , array $arguments )
```

在对象中调用一个不可访问方法时，__call() 会被调用。

该方法有两个参数，第一个参数 $name 参数是要调用的方法名称，第二个 $arguments 参数是一个枚举数组，包含着要传递给方法 $name 的参数。

## __callStatic()

```php
public static mixed __callStatic ( string $name , array $arguments )
```

在静态上下文中调用一个不可访问方法时，__callStatic()会被调用。

可见性未设置为 public 或未声明为 static 的时候会产生一个警告。

## __set()

```php
public void __set ( string $name , mixed $value )
```

在给不可访问属性赋值时，__set() 会被调用，参数是被设置的属性名和值。


## __get()

```php
public mixed __get ( string $name )
```

读取不可访问属性的值时，__get()会被调用，参数为属性名。

## __isset()

```php
public bool __isset ( string $name )
```

当对不可访问属性调用 isset() 或 empty()时，__isset()会被调用

## __unset()

```php
public void __unset ( string $name )
```

当对不可访问属性调用 unset()时，__unset() 会被调用。

属性重载只能在对象中进行。在静态方法中，这些魔术方法将不会被调用。所以这些方法都不能被 声明为 static。将这些魔术方法定义为 static 会产生一个警告。

## __sleep()

```php
public array __sleep ( void )
```

serialize() 函数会检查类中是否存在一个魔术方法 __sleep()。如果存在，则该方法会优先被调用，然后才执行序列化操作。

此方法常用于提交未提交的数据，或类似的清理对象操作。同时，如果有一些很大的对象，但不需要全部保存，这个功能就很好用，清理对象后并返回一个包含对象中所有应被序列化的变量名称的数组。

如果该方法未返回任何内容，则 NULL 被序列化，并产生一个 E_NOTICE 级别的错误。

__sleep() 不能返回父类的私有成员的名字。这样做会产生一个 E_NOTICE 级别的错误。可以用 Serializable 接口来替代。

## __wakeup()

```php
void __wakeup ( void )
```

执行unserialize()反序列化操作时，先会调用这个函数，预先准备对象需要的资源，例如重新建立数据库连接，或执行其它初始化操作。

## __toString()

```php
public string __toString ( void )
```

用于一个类被当成字符串时应怎样回应。例如 echo $obj; 应该显示些什么。

此方法必须返回一个字符串，否则将发出一条E_RECOVERABLE_ERROR 级别的致命错误。

不能在 __toString() 方法中抛出异常。这么做会导致致命错误。

## __invoke()

```php
mixed __invoke ([ $... ] )
```

当尝试以调用函数的方式调用一个对象时，__invoke()方法会被自动调用。

## __clone()

```php
void __clone ( void )
```

对象复制可以通过 clone 关键字来完成（如果可能，这将调用对象的 __clone() 方法）。对象中的 __clone() 方法不能被直接调用。

```php
$copy_of_object = clone $object;
```

当对象被复制后，PHP 会对对象的所有属性执行一个浅复制（shallow copy）。所有的引用属性 仍然会是一个指向原来的变量的引用。

当复制完成时，如果定义了 __clone() 方法，则新创建的对象（复制生成的对象）中的 __clone()方法会被调用，可用于修改属性的值（如果有必要的话）。

## __set_state()

```php
static object __set_state ( array $properties )
```
当调用 var_export() 导出类时，此方法会被调用。

本方法的唯一参数是一个数组，其中包含按 array('property' => value, ...) 格式排列的类属性。

## __debugInfo()，打印所需调试信息

```php
array __debugInfo ( void )
```

该方法在var_dump()类对象的时候被调用，如果没有定义该方法，则var_dump会打印出所有的类属性

## __autoload()

```php
__autoload ( string $class ) : void
```

当使用到的类不存在时会调用此函数，只是“魔术函数”，而不是类的魔术方法，并且在7.2已被废弃。[autoload](../framework/autoload.md)