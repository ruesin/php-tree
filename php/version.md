# 版本

## 8.0

### 命名参数

命名参数允许基于参数名而不是参数位置向函数传递参数。这使得参数的含义可以自我记录，使参数顺序独立，并允许任意跳过默认值。

命名参数可以让函数或者方法的调用更加清晰直观，传入参数的顺序是和定义无关的，可以跳过位置靠前的可选参数。

```php
htmlspecialchars($string, double_encode: false);
// Same as
htmlspecialchars($string, ENT_COMPAT | ENT_HTML401, 'UTF-8', false);
```

### 联合类型

联合类型（Union Types）是两种或多种类型的集合，用户可以使用其中一种。

请注意，`void`永远不能是联合类型的一部分，因为它表示 “根本没有返回值”。此外，可以使用 `|null` 或使用现有的`?`符号来写`nullable`的联合类型。

```php

# PHP 7
class Number {
  /** @var int|float */
  private $number;
  /**
   * @param float|int $number
   */
  public function __construct($number) {
    $this->number = $number;
  }
}
new Number('NaN'); // Ok

# PHP 8
class Number {
  public function __construct(
    private int|float $number
  ) {}
}
new Number('NaN'); // TypeError
```

### 注解

现在可以用原生的PHP语法来使用结构化的元数据，而不需要再依赖PHPDoc解析，性能也随之提升，可以通过反射直接获取元数据。

```php

// PHP 7
class PostsController
{
    /**
     * @Route("/api/posts/{id}", methods={"GET"})
     */
    public function get($id) { /* ... */ }
}

// PHP 8
class PostsController
{
    #[Route("/api/posts/{id}", methods: ["GET"])]
    public function get($id) { /* ... */ }
}

```

### 构造器属性提升

这个新的语法糖来用来创建值对象或数据传输对象。不用为类属性和构造函数指定它们，PHP 现在可以将它们合并为一个。

```php
// PHP 7
class Point {
  public float $x;
  public float $y;
  public float $z;
  public function __construct(
    float $x = 0.0,
    float $y = 0.0,
    float $z = 0.0
  ) {
    $this->x = $x;
    $this->y = $y;
    $this->z = $z;
  }
}

// PHP 8
class Point {
  public function __construct(
    public float $x = 0.0,
    public float $y = 0.0,
    public float $z = 0.0,
  ) {}
}

```

### Match 表达式

新的 match 类似于 switch，`match`可以返回值，不需要`break`语句，可以组合条件，使用严格的类型比较，并且不执行任何类型的强制。
- Match 是一个表达式，它可以储存到变量中亦可以直接返回。
- Match 分支仅支持单行，它不需要一个 break; 语句。
- Match 使用严格比较。

```php

$input = 8.0;

// PHP 7
switch ($input) {
  case '8.0':
  case '7.0':
    $result = "Oh no!";
    break;
  case 8.0:
    $result = "This is what I expected";
    break;
}
echo $result;

// PHP 8
echo match ($input) {
  '8.0', '7.0' => "Oh no!",
  8.0 => "This is what I expected",
};

```

### nullsafe 运算符
现在可以用新的`nullsafe`运算符链式调用，而不需要条件检查`null`。如果链条中的一个元素失败了，整个链条会中止并认定为`Null`。
```php

// PHP 7
$country =  null;
if ($session !== null) {
  $user = $session->user;
  if ($user !== null) {
    $address = $user->getAddress();
 
    if ($address !== null) {
      $country = $address->country;
    }
  }
}

// PHP 8
$country = $session?->user?->getAddress()?->country;

```
### JIT
PHP 8 引入了两个即时编译引擎。 Tracing JIT 在两个中更有潜力，它在综合基准测试中显示了三倍的性能， 并在某些长时间运行的程序中显示了 1.5-2 倍的性能改进。 典型的应用性能则和 PHP 7.4 不相上下。 

JIT作为PHP底层编译引擎，对于PHP8的性能贡献是非常之大，不过对于常规WEB应用来说，优势不明显。

### 新的类、接口、函数
#### WeakMap

`WeakMap` 保留对对象的引用，这不会阻止这些对象被垃圾回收。

以`ORM`为例，它们通常会实现缓存，其缓存保存对实体类的引用，以提高实体之间关系的性能。只要该缓存具有对这些实体对象的引用，就不能对其进行垃圾回收，即使该缓存是唯一引用它们的对象也是如此。

如果该缓存层使用了弱引用和映射，则 PHP 将在没有其他引用时对这些对象进行垃圾回收。尤其是对于 ORM，它可以管理一个请求中的数百个（乃至数千个）实体。Weak maps（弱映射）可以提供一种更好，对资源更友好的方式来处理这些对象。

```php
class Foo
{
    private WeakMap $cache;
 
    public function getSomethingWithCaching(object $obj): object
    {
        return $this->cache[$obj]
           ??= $this->computeSomethingExpensive($obj);
    }
}
```

#### Stringable接口
Stringable接口可用于类型提示任何字符串或实现`__toString()`的内容。此外，每当一个类实现`__toString()`时，它就会自动实现幕后接口，而无需手动实现。
```php
class Foo 
{ 
    public function __toString(): string 
    { 
        return 'foo'; 
    } 
} 
function bar(Stringable $stringable) { /* … */ } 
bar(new Foo()); 
bar('abc'); 

```

### str_contains()、str_starts_with()、str_ends_with()

```php 
if (strpos('string with lots of words', 'words') !== false) { /* … */ }

if (str_contains('string with lots of words', 'words')) { /* … */ }

str_starts_with('haystack', 'hay'); // true
str_ends_with('haystack', 'stack'); // true

```
#### fdiv()
新的fdiv()函数与fmod()和intdiv()函数的功能相似，允许被 0 除。根据情况你会得到INF、-INF或NAN，而不是错误。

#### get_debug_type()函数

get_debug_type()返回一个变量的类型。听起来像gettype()的功能？get_debug_type()为数组、字符串、匿名类和对象返回更有用的输出。

例如，在类\Foo\Bar上调用gettype()将返回object。使用get_debug_type()将返回类名称。

#### get_resource_id()函数

Resources 是 PHP 中的特殊变量，指的是外部资源。一个例子是 MySQL 连接，另一个是文件句柄。

这些资源中每一个都分配了一个 ID，但以前唯一知道该 ID 的方法是将资源转换为int：

$resourceId = (int) $resource; 

PHP 8 添加了get_resource_id()函数，让这个操作更加明显易懂，且类型安全：

$resourceId = get_resource_id($resource); 


#### token_get_all() 对象实现 

函数的作用是：返回值的是一个数组。
此 RFC 使用 PhpToken::getall () 方法添加一个 PhpToken 类。
此实现使用对象，而不是普通值。
它消耗更少的内存，更容易阅读。

#### New DOM Traversal and Manipulation APIs


### 类型系统与错误处理的改进

#### 算术/位运算符更严格的类型检测
在 PHP 8 之前，可以在数组、资源或对象上应用算术或按位运算符。现在就不行了，新版将抛出TypeError：
```php
[] % [42]; 
$object + 4; 
```

#### traits 中的抽象方法改进

Traits 可以指定抽象方法，这些方法必须由使用它们的类实现。在PHP8，必须保持一致的方法定义，包括参数类型和返回类型。
``` php
trait MyTrait {
    abstract private function neededByTheTrait(): string;
 
    public function doSomething() {
        return strlen($this->neededByTheTrait());
    }
}
 
class TraitUser {
    use MyTrait;
 
    // This is allowed:
    private function neededByTheTrait(): string { }
 
    // This is forbidden (incorrect return type)
    private function neededByTheTrait(): stdClass { }
 
    // This is forbidden (non-static changed to static)
    private static function neededByTheTrait(): string { }
}
```

#### 确保魔术方法签名正确

#### PHP 引擎 warning 警告的重新分类

#### 不兼容的方法签名导致 Fatal 错误

#### 操作符 @ 不再抑制 fatal 错误。 
此更改可能会揭示在 PHP8 之前隐藏的错误。请确保在生产服务器上设置 display_errors=off ！

#### 私有方法继承

以前，PHP 曾经对公共、保护和私有方法应用相同的继承检查。换句话说：私有方法应遵循与保护方法和公共方法相同的方法签名规则。这是没有道理的，因为子类将无法访问私有方法。

该 RFC 更改了这个行为，因此不再对私有方法执行这些继承检查。此外，使用final private function也没有意义，因此，现在它将触发警告：

Warning: Private methods cannot be final as they are never overridden by other classes 

#### Mixed 类型

`mixed`本身是以下类型之一：
- array
- bool
- callable
- int
- float
- null
- object
- resource
- string

注意，mixed也可以用作参数或属性类型，而不仅仅是返回类型。

另外由于mixed已经包含null，因此不允许将其设置为nullable。以下内容将触发错误：
```php
function bar(): ?mixed {} 
```

#### Static 返回类型

虽然现在的 PHP 已经可以返回self，但是直到 PHP 8 中static才是有效的返回类型。考虑到 PHP 动态类型的性质，这个特性对许多开发人员都非常有用。
```php
class Foo 
{ 
    public function test(): static 
    { 
        return new static(); 
    } 
} 

```

#### 内部函数的类型

#### 扩展 Curl、 Gd、 Sockets、 OpenSSL、 XMLWriter、 XML 以 Opaque 对象替换 resource


### 其他语法调整和改进

#### 允许参数列表中的末尾逗号、闭包 use 列表中的末尾逗号

现在的 PHP，虽然可以调用函数时在尾部加逗号，但参数列表中仍然缺少对尾部逗号的支持。PHP 8 现在允许使用它，也就是说你可以执行以下操作：

```php
public function( 
    string $parameterA, 
    int $parameterB, 
    Foo $objectfoo, 
) { 
    // 注意上面最后一个逗号… 
} 
```

#### 无变量捕获的 catch
在 PHP 8 之前，每当你想捕获一个异常时都必须将其存储在一个变量中，不管你是否使用这个变量。现在使用非捕获 catches，你也可以忽略变量。

请注意，你必须始终指定类型，不允许使用空catch。如果要捕获所有的异常和错误，可以使用Throwable作为捕获类型。

```php
// PHP 7 
try { 
    // Something goes wrong 
} catch (MySpecialException $exception) { 
    Log::error("Something went wrong"); 
} 

// PHP 8
try { 
    // Something goes wrong 
} catch (MySpecialException) { 
    Log::error("Something went wrong"); 
} 

```
#### 变量语法的调整

#### Namespace 名称作为单个 token

#### Throw 表达式
将throw从语句变为表达式，这样就可以在许多新场景中抛出异常。
```php
$triggerError = fn () => throw new MyError(); 
$foo = $bar['offset'] ?? throw new OffsetDoesNotExist('offset'); 
```

#### 可以在对象上使用::class

一个小而有用的新特性：现在可以对对象使用`::class`，而不必对它们使用`get_class()`，工作方式与`get_class()`相同。

```php
$foo = new Foo();

var_dump($foo::class);
```

### 其它特性

#### 字符串与数字的比较更符合逻辑
PHP 8 比较数字字符串（numeric string）时，会按数字进行比较。 不是数字字符串时，将数字转化为字符串，按字符串比较。
```php
// PHP 7
0 == 'foobar' // true
// PHP 8
0 == 'foobar' // false
```

#### 从接口创建DateTime对象

你已经可以使用DateTime::createFromImmutable($immutableDateTime)从DateTimeImmutable对象创建DateTime对象，但反过来就很麻烦。

PHP 8 添加了DateTime::createFromInterface()和DatetimeImmutable::createFromInterface()，所以现在有一种通用的方法可以将DateTime和DateTimeImmutable对象彼此转换。

```php
DateTime::createFromInterface(DateTimeInterface $other); 
DateTimeImmutable::createFromInterface(DateTimeInterface $other); 
```

#### 内部函数类型错误的一致性
现在大多数内部函数在参数验证失败时抛出 Error 级异常。
```php
// PHP 7
strlen([]); // Warning: strlen() expects parameter 1 to be string, array given
array_chunk([], -1); // Warning: array_chunk(): Size parameter expected to be greater than 0

// PHP 8
strlen([]); // TypeError: strlen(): Argument #1 ($str) must be of type string, array given
array_chunk([], -1); // ValueError: array_chunk(): Argument #2 ($length) must be greater than 0
```

#### 默认错误报告级别

现在是E_ALL，而不是E_NOTICE和E_DEPRECATED。这意味着新版可能会弹出许多错误，这些错误在 PHP 8 以前会被静默忽略。


#### 串联优先级
虽然在 PHP7.4 中已不推荐使用，但此更改现在生效。
如果你这样写的话：

```php
echo "sum: " . $a + $b;

// PHP 7
// echo ("sum: " . $a) + $b;

// PHP 8
echo "sum: " . ($a + $b);
```

#### 反射方法签名更改

反射类的三个方法签名已更改：
```php
ReflectionClass::newInstance($args);
ReflectionFunction::invoke($args);
ReflectionMethod::invoke($object, $args);
```
现已成为：
```php
ReflectionClass::newInstance(...$args);
ReflectionFunction::invoke(...$args);
ReflectionMethod::invoke($object, ...$args);
```
升级指南指定，如果您扩展了这些类，并且仍然希望同时支持 PHP 7 和 PHP 8，则允许以下签名：
```php
ReflectionClass::newInstance($arg = null, ...$args);
ReflectionFunction::invoke($arg = null, ...$args);
ReflectionMethod::invoke($object, $arg = null, ...$args);
```

#### ext-json始终可用

以前，可以在不启用 JSON 扩展的情况下编译 PHP，以后就不行了。现在，开发人员知道 JSON 是一直能用的，而不需要提前确认扩展是否可用。由于 JSON 非常流行，所以这个改进很方便。


### 参考：
- https://www.php.net/releases/8.0/zh.php

## PHP7 性能提升

### zval

PHP5的Zval，内存占据24个字节：
```c
typedef struct _zval_struct zval;

struct _zval_struct {
	/* Variable information */
	zvalue_value value;		/* value */
	zend_uint refcount__gc;
	zend_uchar type;	/* active type */
	zend_uchar is_ref__gc;
};

typedef union _zvalue_value {
	long lval;					/* long value */
	double dval;				/* double value */
	struct {
		char *val;
		int len;
	} str;
	HashTable *ht;				/* hash table value */
	zend_object_value obj;
	zend_ast *ast;
} zvalue_value;
```
因为zval可以表示一切PHP中的数据类型，所以它包含了一个type字段，表示这个zval存储的是什么类型的值，常见的可能选项是IS_NULL、IS_LONG、IS_STRING、IS_ARRAY、IS_OBJECT等等。

根据type字段的值不同，就要用不同的方式解读value的值，这个value是个联合体，比如对于type是IS_STRING，应该用value.str来解读zval.value字段，而如果type是IS_LONG，就要用value.lval来解读。

另外，PHP是用引用计数来做基本的垃圾回收的，所以zval中有一个refcount__gc字段，表示这个zval的引用数目。is_ref，这个值表示了PHP中的一个类型是否是引用，这里可以看到是不是引用是一个标志位。


PHP7的Zval，通过union（联合体）使用更多的变量使Zval从24字节下降到16字节。：
```c
struct _zval_struct {
	zend_value        value;			/* value */
	union {
		struct {
			ZEND_ENDIAN_LOHI_4(
				zend_uchar    type,			/* active type */
				zend_uchar    type_flags,
				zend_uchar    const_flags,
				zend_uchar    reserved)	    /* call info for EX(This) */
		} v;
		uint32_t type_info;
	} u1;
	union {
		uint32_t     next;                 /* hash collision chain */
		uint32_t     cache_slot;           /* literal cache slot */
		uint32_t     lineno;               /* line number (for ast nodes) */
		uint32_t     num_args;             /* arguments number for EX(This) */
		uint32_t     fe_pos;               /* foreach position */
		uint32_t     fe_iter_idx;          /* foreach iterator index */
		uint32_t     access_flags;         /* class constant access flags */
		uint32_t     property_guard;       /* single property guard */
		uint32_t     extra;                /* not further specified */
	} u2;
};

typedef union _zend_value {
	zend_long         lval;				/* long value */
	double            dval;				/* double value */
	zend_refcounted  *counted;
	zend_string      *str;
	zend_array       *arr;
	zend_object      *obj;
	zend_resource    *res;
	zend_reference   *ref;
	zend_ast_ref     *ast;
	zval             *zv;
	void             *ptr;
	zend_class_entry *ce;
	zend_function    *func;
	struct {
		uint32_t w1;
		uint32_t w2;
	} ww;
} zend_value;
```
虽然看起来变得好大，但其实全部都是联合体，这个新的zval在64位环境下,现在只需要16个字节(2个指针size)，它主要分为俩个部分，value和扩充字段，而扩充字段又分为u1和u2俩个部分，其中u1是type info，u2是各种辅助字段。

其中value部分，是一个size_t大小(一个指针大小)，可以保存一个指针，或者一个long，或者一个double。

而type info部分则保存了这个zval的类型。扩充辅助字段则会在多个其他地方使用，比如next，就用在取代Hashtable中原来的拉链指针。

### zend_string

PHP当中的字符串（string）的载体时zend_string结构体，PHP7的zend_string结构体最后一个成员变量采用char数组，而不是使用char*，这里有一个小优化技巧，可以降低CPU的cache miss。

```c
struct _zend_string {
	zend_refcounted_h gc;
	zend_ulong        h;                /* hash value */
	size_t            len;
	char              val[1];
};
```

### [数组](./array.md)

PHP5的HashTable:

![HashTable-PHP5](../images/php/version-1.jpg)

PHP7的Zend Array:

![HashTable-PHP7](../images/php/version-2.jpg)

Zend Array将整块的数组元素和hash映射表全部连接在一起，被分配在同一块内存内，这样遍历一个整形的简单类型数据效率非常高，当然，最重要的是它能够避免CPU Cache Miss（CPU缓存命中率下降）。

### 函数调用机制（Function Calling Convention）：
PHP7改进了函数的调用机制，通过优化参数传递（减少重复发送send_val和recv参数）的环节，减少了一些指令，提高执行效率。

### 通过宏定义和内联函数（inline），让编译器提前完成部分工作：

PHP7在这方面做了不少的优化，将不少需要在运行阶段要执行的工作（例如固定的字符常量），放到了编译阶段。

## 5.6
### 向后不兼容
#### 使用数组标识符为类定义数组类型的属性时，数组的键不会被覆盖
在 PHP 5.6 之前的版本中，为类定义数组类型的属性时，如果数组中同时使用了显式数组键和隐式数组键，并且显式的键和隐式的序列键相同，那么数组的键将被覆盖。
```php
class C {
    const ONE = 1;
    public $array = [
        self::ONE => 'foo',
        'bar',
        'quux',
    ];
}

var_dump((new C)->array);
// 5.5
// array(2) {
//   [0]=>
//   string(3) "bar"
//   [1]=>
//   string(4) "quux"
// }

//5.6
// array(3) {
//   [1]=>
//   string(3) "foo"
//   [2]=>
//   string(3) "bar"
//   [3]=>
//   string(4) "quux"
// }
```
#### 严格的 json_decode()
对于 JSON 字面量 true，false 和 null，如果不采用小写格式，将会被 json_decode() 函数拒绝，同时相应的设置 json_last_error()。 在之前的版本中，json_decode() 函数可以接受这些字面量的 全部大写或者大小写混写的格式。

此变更仅会影响传入到 json_decode() 中的 JSON 格式无效的情况，有效的 JSON 输入不会受到影响并且能够正确解析。

### 新特性
#### 使用表达式定义常量
在之前的 PHP 版本中，必须使用静态值来定义常量，声明属性以及指定函数参数默认值。 现在你可以使用包括数值、字符串字面量以及其他常量在内的数值表达式来 定义常量、声明属性以及设置函数参数默认值。

现在可以通过 const 关键字来定义类型为 array 的常量。

```php
const ONE = 1;
const TWO = ONE * 2;
const ARR = ['a', 'b'];

class C {
    const THREE = TWO + 1;
    const ONE_THIRD = ONE / self::THREE;
    const SENTENCE = 'The value of THREE is '.self::THREE;

    public function f($a = ONE + self::THREE) {
        return $a;
    }
}
```

#### 使用 ... 运算符定义变长参数函数
现在可以不依赖 func_get_args()，使用 ... 运算符 来实现 变长参数函数。
```php
function f($req, $opt = null, ...$params) {
    // $params 是一个包含了剩余参数的数组
    printf('$req: %d; $opt: %d; number of params: %d'."\n",
           $req, $opt, count($params));
}

f(1);
f(1, 2);
f(1, 2, 3);
f(1, 2, 3, 4);
f(1, 2, 3, 4, 5);
```

#### 使用 ... 运算符进行参数展开
在调用函数的时候，使用 ... 运算符，将 数组 和 可遍历 对象展开为函数参数。 在其他编程语言，比如 Ruby中，这被称为连接运算符，。
```php
function add($a, $b, $c) {
    return $a + $b + $c;
}

$operators = [2, 3];
echo add(1, ...$operators);
```
#### 使用 ** 进行幂运算
加入右连接运算符 ** 来进行幂运算。 同时还支持简写的 **= 运算符，表示进行幂运算并赋值。
```php
printf("2 ** 3 ==      %d\n", 2 ** 3); //2 ** 3 ==      8
printf("2 ** 3 ** 2 == %d\n", 2 ** 3 ** 2); //2 ** 3 ** 2 == 512
$a = 2;
$a **= 3;
printf("a ==           %d\n", $a); //a ==           8
```

#### use function 以及 use const
use 运算符 被进行了扩展以支持在类中导入外部的函数和常量。 对应的结构为 use function 和 use const。
```php
namespace Name\Space {
    const FOO = 42;
    function f() { echo __FUNCTION__."\n"; }
}

namespace {
    use const Name\Space\FOO;
    use function Name\Space\f;

    echo FOO."\n";
    f();
}
```

#### phpdbg
PHP 的 SAPI 模块中实现了一个 交互式调试器，叫做 phpdbg。更多信息，请访问 » [phpdbg 文档](https://phpdbg.room11.org/)。

#### 默认字符编码
对于一些字符编码相关的函数，例如 htmlentities()，html_entity_decode() 以及 htmlspecialchars() 使用 default_charset 作为默认字符集。请注意，对于 iconv（现已废弃） 和 mbstring 相关的函数，如果分别设置了他们的编码，那么这些对应设置的优先级高于 default_charset。

default_charset 的默认值是 UTF-8。

#### php://input 是可重用的了
只要你需要，你可以多次打开并读取 php://input。 同时，这个特性使得在处理 POST 的数据的时候，可以明显降低对于内存的需求量。

#### 大文件上传
现在可以支持大于 2GB 的文件上传。

#### GMP 支持运算符重载
GMP 支持运算符重载，并且造型成数值类型。 这使得使用 GMP 的代码更加直观。
```php
$a = gmp_init(42);
$b = gmp_init(17);

if (version_compare(PHP_VERSION, '5.6', '<')) {
    echo gmp_intval(gmp_add($a, $b)), PHP_EOL;
    echo gmp_intval(gmp_add($a, 17)), PHP_EOL;
    echo gmp_intval(gmp_add(42, $b)), PHP_EOL;
} else {
    echo $a + $b, PHP_EOL; 
    echo $a + 17, PHP_EOL;
    echo 42 + $b, PHP_EOL;
}
//59
```
#### 使用 hash_equals() 比较字符串避免时序攻击
加入 hash_equals() 函数，以恒定的时间消耗来进行字符串比较，以避免时序攻击。 比如当比较 crypt() 密码散列值的时候，就可以使用此函数。 （假定你不能使用 password_hash() 和 password_verify()，这两个函数也可以抵抗时序攻击）
```php
<?php
$expected  = crypt('12345', '$2a$07$usesomesillystringforsalt$');
$correct   = crypt('12345', '$2a$07$usesomesillystringforsalt$');
$incorrect = crypt('1234',  '$2a$07$usesomesillystringforsalt$');

var_dump(hash_equals($expected, $correct)); //bool(true)
var_dump(hash_equals($expected, $incorrect)); //bool(false)
```

#### __debugInfo()
加入 __debugInfo()，当使用 var_dump() 输出对象的时候，可以用来控制要输出的属性和值。
```php
class C {
    private $prop;

    public function __construct($val) {
        $this->prop = $val;
    }

    public function __debugInfo() {
        return [
            'propSquared' => $this->prop ** 2,
        ];
    }
}

var_dump(new C(42));
```

### 废止的特性
#### 从不兼容的上下文调用方法
现在已废止从不兼容的上下文调用方法，并且产生 E_DEPRECATED 错误 （以前是 E_STRICT）。 在 PHP 的后续版本中可能彻底移除对此特性的支持。
```php
class A {
    function f() { echo get_class($this); }
}

class B {
    function f() { A::f(); }
}

(new B)->f(); 
//Deprecated: Non-static method A::f() should not be called statically, assuming $this from incompatible context in - on line 7 B
```

#### $HTTP_RAW_POST_DATA 和 always_populate_raw_post_data
使用 always_populate_raw_post_data 会导致在填充 $HTTP_RAW_POST_DATA 时产生 E_DEPRECATED 错误。 请使用 php://input 替代 $HTTP_RAW_POST_DATA，因为它可能在后续的 PHP 版本中被移除。 设置 always_populate_raw_post_data 为 -1 （这样会强制 $HTTP_RAW_POST_DATA 未定义，所以也不回导致 E_DEPRECATED 的错误） 来体验新的行为。

#### iconv 和 mbstring 编码设置 
iconv 和 mbstring 配置选项中 和编码相关的选项都已废弃，请使用 default_charset。

## 7.0
### 不向后兼容的变更
#### 错误和异常处理相关的变更
在 PHP 7 中，很多致命错误以及可恢复的致命错误，都被转换为异常来处理了。 这些异常继承自 Error 类，此类实现了 Throwable 接口 （所有异常都实现了这个基础接口）。

这也意味着，当发生错误的时候，以前代码中的一些错误处理的代码将无法被触发。 因为在 PHP 7 版本中，已经使用抛出异常的错误处理机制了。 （如果代码中没有捕获 Error 异常，那么会引发致命错误）。

PHP 7 改变了大多数错误的报告方式。不同于传统（PHP 5）的错误报告机制，现在大多数错误被作为 Error 异常抛出。

这种 Error 异常可以像 Exception 异常一样被第一个匹配的 try / catch 块所捕获。如果没有匹配的 catch 块，则调用异常处理函数（事先通过 set_exception_handler() 注册）进行处理。 如果尚未注册异常处理函数，则按照传统方式处理：被报告为一个致命错误（Fatal Error）。

Error 类并非继承自 Exception 类，所以不能用 catch (Exception $e) { ... } 来捕获 Error。你可以用 catch (Error $e) { ... }，或者通过注册异常处理函数（ set_exception_handler()）来捕获 Error。

##### set_exception_handler() 不再保证收到的一定是 Exception 对象
抛出 Error 对象时，如果 set_exception_handler() 里的异常处理代码声明了类型 Exception ，将会导致 fatal error。

想要异常处理器同时支持 PHP5 和 PHP7，应该删掉异常处理器里的类型声明。如果代码仅仅是升级到 PHP7，则可以把类型 Exception 替换成 Throwable。
```php
// PHP 5 时代的代码将会出现问题
function handler(Exception $e) { ... }
set_exception_handler('handler');

// 兼容 PHP 5 和 7
function handler($e) { ... }

// 仅支持 PHP 7
function handler(Throwable $e) { ... }
```

##### 当内部构造器失败的时候，总是抛出异常
在之前版本中，如果内部类的构造器出错，会返回 NULL 或者一个不可用的对象。 从 PHP 7 开始，如果内部类构造器发生错误，那么会抛出异常。

##### 解析错误会抛出 ParseError 异常
解析错误会抛出 ParseError 异常。 对于 eval() 函数，需要将其包含到一个 catch 代码块中来处理解析错误。

##### E_STRICT 警告级别变更
原有的 E_STRICT 警告都被迁移到其他级别。 E_STRICT 常量会被保留，所以调用 error_reporting(E_ALL|E_STRICT) 不会引发错误。

#### 关于变量处理的变化
PHP 7 现在使用了抽象语法树来解析源代码。这使得许多由于之前的PHP的解释器的限制所不可能实现的改进得以实现。 但出于一致性的原因导致了一些特殊例子的变动，而这些变动打破了向后兼容。

##### 关于间接使用变量、属性和方法的变化
对变量、属性和方法的间接调用现在将严格遵循从左到右的顺序来解析，而不是之前的混杂着几个特殊案例的情况。

![parse-variable](../images/php/version-3.png)

使用了旧的从右到左的解析顺序的代码必须被重写，明确的使用大括号来表明顺序（参见上表中间一列）。 这样使得代码既保持了与PHP 7.x的前向兼容性，又保持了与PHP 5.x的后向兼容性。

这同样影响了 global 关键字。如果需要的话可以使用大括号来模拟之前的行为。

```php
function f() {
    // Valid in PHP 5 only.
    global $$foo->bar;

    // Valid in PHP 5 and 7.
    global ${$foo->bar};
}
```

##### list() 不再以反向的顺序来进行赋值
list() 现在会按照变量定义的顺序来给他们进行赋值，而非反过来的顺序。 通常来说，这只会影响list() 与数组的[]操作符一起使用的案例。不要依赖list()的赋值顺序，因为这是一个在未来也许会变更的实现细节。
```php
list($a[], $a[], $a[]) = [1, 2, 3];
var_dump($a)
// php5
// [0]=>int(3)
// [1]=>int(2)
// [2]=>int(1)

// php7
// [0]=>int(1)
// [1]=>int(2)
// [2]=>int(3)
```

##### 空的list()赋值支持已经被移除
list() 结构现在不再能是空的。如下的例子不再被允许：
```php
list() = $a;
list(,,) = $a;
list($x, list(), $y) = $a;
```

##### list()不再能解开string
list() 不再能解开字符串（string）变量。 你可以使用str_split()来代替它。

##### 引用赋值过程中自动创建元素时的数组顺序已变更
当通过在引用赋值中引用元素自动创建这些元素时，数组中元素的顺序发生了变化。
```php
$array = [];
$array["a"] =& $array["b"];
$array["b"] = 1;
var_dump($array);

// php5
// ["b"]=>&int(1)
// ["a"]=>&int(1)

// php7
// ["a"]=>&int(1)
// ["b"]=>&int(1)
```
##### 函数参数附近的括号不再影响行为
在 PHP 5中，在以引用方式传递函数参数时，使用冗余的括号对可以隐匿严格标准的警告。现在，这个警告总会触发。
```php
function getArray() {
    return [1, 2, 3];
}

function squareArray(array &$a) {
    foreach ($a as &$v) {
        $v **= 2;
    }
}

// Generates a warning in PHP 7.
squareArray((getArray()));
//Notice: Only variables should be passed by reference in /tmp/test.php on line 13
```

#### foreach的变化
foreach发生了细微的变化，控制结构，主要围绕阵列的内部数组指针和迭代处理的修改。

##### foreach不再改变内部数组指针
在PHP7之前，当数组通过 foreach 迭代时，数组指针会移动。现在开始，不再如此，见下面代码
```php
$array = [0, 1, 2];
foreach ($array as &$val) {
    var_dump(current($array));
}
// php5 
// int(1)
// int(2)
// bool(false)

// php7
// int(0)
// int(0)
// int(0)
```
##### foreach 通过值遍历时，操作的值为数组的副本
当默认使用通过值遍历数组时，foreach 实际操作的是数组的迭代副本，而非数组本身。这就意味着，foreach 中的操作不会修改原数组的值。

##### foreach通过引用遍历时，有更好的迭代特性
当使用引用遍历数组时，现在 foreach 在迭代中能更好的跟踪变化。例如，在迭代中添加一个迭代值到数组中，参考下面的代码：
```php
$array = [0];
foreach ($array as &$val) {
    var_dump($val);
    $array[1] = 1;
}
// php5
// int(0)

// php7
// int(0)
// int(1)
```

##### 非Traversable 对象的遍历
迭代一个非Traversable对象将会与迭代一个引用数组的行为相同。 这将导致在对象添加或删除属性时，foreach通过引用遍历时，有更好的迭代特性也能被应用

#### 整数处理的变更

##### 无效的八进制字符（Invalid octal literals）
在之前，一个八进制字符如果含有无效数字，该无效数字将被静默删节(0128 将被解析为 012). 现在这样的八进制字符将产生解析错误。

##### 负位移运算（Negative bitshifts）
以负数形式进行的位移运算将会抛出一个 ArithmeticError：
```php
var_dump(1 >> -1);

// php5
// int(0)

// php7
// Fatal error: Uncaught ArithmeticError: Bit shift by negative number in /tmp/test.php:2
```

##### 超范围后产生位移
超出 integer 位宽的位移操作（无论哪个方向）将始终得到 0 作为结果。在从前，这一操作是结构依赖的。

#### string处理上的调整

##### 十六进制字符串不再被认为是数字
```php
var_dump("0x123" == "291");
var_dump(is_numeric("0x123"));
var_dump("0xe" + "0x1");
var_dump(substr("foo", "0x1"));
// php5
// bool(true)
// bool(true)
// int(15)
// string(2) "oo"

// php7
// bool(false)
// bool(false)
// int(0)
// Notice: A non well formed numeric value encountered in /tmp/test.php on line 5
// string(3) "foo"
```
filter_var() 函数可以用于检查一个 string 是否含有十六进制数字,并将其转换为integer:
```php
$str = "0xffff";
$int = filter_var($str, FILTER_VALIDATE_INT, FILTER_FLAG_ALLOW_HEX);
if (false === $int) {
    throw new Exception("Invalid integer!");
}
var_dump($int); // int(65535)
```

##### \u{ 可能引起错误
由于新的 [Unicode codepoint escape syntax](https://www.php.net/manual/zh/migration70.new-features.php#migration70.new-features.unicode-codepoint-escape-syntax)语法，紧连着无效序列并包含\u{ 的字串可能引起致命错误。 为了避免这一报错，应该避免反斜杠开头。

#### 被移除的函数（Removed functions）
##### call_user_method() and call_user_method_array()
这两个函数从PHP 4.1.0开始被废弃，应该使用call_user_func() 和 call_user_func_array()。 你也可以考虑使用 变量函数 或者 ... 操作符。

##### 所有的 ereg* 函数
所有 ereg 系列函数被删掉了。 [PCRE](https://www.php.net/manual/zh/book.pcre.php) 作为推荐的替代品。

##### mcrypt 别名
已废弃的 mcrypt_generic_end() 函数已被移除，请使用mcrypt_generic_deinit()代替。

此外，已废弃的 mcrypt_ecb(), mcrypt_cbc(), mcrypt_cfb() 和 mcrypt_ofb() 函数已被移除，请配合恰当的MCRYPT_MODE_* 常量来使用 mcrypt_decrypt()进行代替。

##### 所有 ext/mysql 函数
所有 ext/mysql 函数已被删掉了。 如何选择不同的 MySQL API，详情请见 选择 MySQL API。

##### 所有 ext/mssql 函数
所有 ext/mssql 函数已被删掉了。 替代品的列表，参见 MSSQL 介绍。

##### intl 别名
已废弃的 datefmt_set_timezone_id() 和 IntlDateFormatter::setTimeZoneID() 函数已被移除，请使用 datefmt_set_timezone() 与 IntlDateFormatter::setTimeZone()代替。

##### set_magic_quotes_runtime()
set_magic_quotes_runtime(), 和它的别名 magic_quotes_runtime()已被移除. 它们在PHP 5.3.0中已经被废弃,并且 在in PHP 5.4.0也由于魔术引号的废弃而失去功能。

##### set_socket_blocking()
已废弃的 set_socket_blocking() 函数已被移除，请使用stream_set_blocking()代替。

##### dl() in PHP-FPM
dl()在 PHP-FPM 不再可用，在 CLI 和 embed SAPIs 中仍可用。

##### GD Type1 functions
GD扩展中已删除对PostScript Type1字体的支持。


#### new 操作符创建的对象不能以引用方式赋值给变量
new 语句创建的对象不能 以引用的方式赋值给变量。
```php
class C {}
$c =& new C;
// php5
// Deprecated: Assigning the return value of new by reference is deprecated in /tmp/test.php on line 3

//php7 
// Parse error: syntax error, unexpected 'new' (T_NEW) in /tmp/test.php on line 3
```

#### 无效的类、接口以及 trait 命名
不能以下列名字来命名类、接口以及 trait：bool、int、float、string、NULL、TRUE、FALSE

此外，也不要使用下列的名字来命名类、接口以及 trait。虽然在 PHP 7.0 中，这并不会引发错误，但是这些名字是保留给将来使用的：resource、object、mixed、numeric。

#### 移除了 ASP 和 script PHP 标签
```php
<% %>
<%= %>
<script language="php"></script>
```

#### 从不匹配的上下文发起调用
在不匹配的上下文中以静态方式调用非静态方法，在 PHP 5.6 中已经废弃，但是在 PHP 7.0 中，会导致被调用方法中未定义 $this 变量，以及此行为已经废弃的警告。
```php
class A {
    public function test() { var_dump($this); }
}

// 注意：并没有从类 A 继承
class B {
    public function callNonStaticMethodOfA() { A::test(); }
}

(new B)->callNonStaticMethodOfA();
// php5
// Deprecated: Non-static method A::test() should not be called statically, assuming $this from incompatible context in /tmp/test.php on line 8

// php7
// Deprecated: Non-static method A::test() should not be called statically in /tmp/test.php on line 8
// Notice: Undefined variable: this in /tmp/test.php on line 3
```

#### yield 变更为右联接运算符
在使用 yield 关键字的时候，不再需要括号，并且它变更为右联接操作符，其运算符优先级介于 print 和 => 之间。 这可能导致现有代码的行为发生改变：
```php
echo yield -1;
// 在之前版本中会被解释为：
echo (yield) - 1;
// 现在，它将被解释为：
echo yield (-1);

yield $foo or die;
// 在之前版本中会被解释为：
yield ($foo or die);
// 现在，它将被解释为：
(yield $foo) or die;
```
#### 函数定义不可以包含多个同名参数
在函数定义中，不可以包含两个或多个同名的参数。 例如，下面代码中的函数定义会触发 E_COMPILE_ERROR 错误：
```php
function foo($a, $b, $unused, $unused) {
    //
}
```
#### Switch 语句不可以包含多个 default 块
在 switch 语句中，两个或者多个 default 块的代码已经不再被支持。 例如，下面代码中的 switch 语句会触发 E_COMPILE_ERROR 错误：
```php
switch (1) {
    default:
    break;
    default:
    break;
}
```
#### 在函数中检视参数值会返回 当前 的值
当在函数代码中使用 func_get_arg() 或 func_get_args() 函数来检视参数值，或者使用 debug_backtrace() 函数查看回溯跟踪，以及在异常回溯中所报告的参数值是指参数当前的值（有可能是已经被函数内的代码改变过的值），而不再是参数被传入函数时候的原始值了。
```php
function foo($x) {
    $x++;
    var_dump(func_get_arg(0));
}
foo(1);
// php5 
// 1
// php7
// 2
```

#### $HTTP_RAW_POST_DATA 被移除
不再提供 $HTTP_RAW_POST_DATA 变量。 请使用 php://input 作为替代。

#### INI 文件中 # 注释格式被移除
在 INI 文件中，不再支持以 # 开始的注释行，请使用 ;（分号）来表示注释。 此变更适用于 php.ini 以及用 parse_ini_file() 和 parse_ini_string() 函数来处理的文件。

#### JSON 扩展已经被 JSOND 取代
JSON 扩展已经被 JSOND 扩展取代。 对于数值的处理，有以下两点需要注意的： 第一，数值不能以点号（.）结束 （例如，数值 34. 必须写作 34.0 或 34）。 第二，如果使用科学计数法表示数值，e 前面必须不是点号（.） （例如，3.e3 必须写作 3.0e3 或 3e3）。 另外，空字符串不再被视作有效的 JSON 字符串。

#### 在数值溢出的时候，内部函数将会失败
将浮点数转换为整数的时候，如果浮点数值太大，导致无法以整数表达的情况下，在之前的版本中，内部函数会直接将整数截断，并不会引发错误。 在 PHP 7.0 中，如果发生这种情况，会引发 E_WARNING 错误，并且返回 NULL。

#### 自定义会话处理器的返回值修复
在自定义会话处理器中，如果函数的返回值不是 FALSE，也不是 -1，会引发致命错误。现在，如果这些函数的返回值不是布尔值，也不是 -1 或者 0，函数调用结果将被视为失败，并且引发 E_WARNING 错误。

#### 相等的元素在排序时的顺序问题
由于内部排序算法进行了提升，可能会导致对比时被视为相等的多个元素之间的顺序不稳定。

在对比时被视为相等的多个元素之间的排序顺序是不可信赖的，即使是相等的两个元素，他们的位置也可能被排序算法所改变。

#### Mhash 不再是一个单独的扩展了
Mhash 扩展已经被完全整合进 Hash 扩展了。 因此，不要再使用 extension_loaded() 函数来检测是否支持 MHash 扩展了，建议使用 function_exists() 函数来进行检测。 另外，函数 get_loaded_extensions() 以及相关的特性中，也不再报告 和 MHash 扩展相关的信息了。

### 新特性
#### 标量类型声明
标量类型声明 有两种模式: 强制 (默认) 和 严格模式。 现在可以使用下列类型参数（无论用强制模式还是严格模式）： 字符串(string), 整数 (int), 浮点数 (float), 以及布尔值 (bool)。它们扩充了PHP5中引入的其他类型：类名，接口，数组和 回调类型。
```php
// Coercive mode
function sumOfInts(int ...$ints)
{
    return array_sum($ints);
}

var_dump(sumOfInts(2, '3', 4.1)); //int(9)
```
要使用严格模式，一个 declare 声明指令必须放在文件的顶部。这意味着严格声明标量是基于文件可配的。 这个指令不仅影响参数的类型声明，也影响到函数的返回值声明

#### 返回值类型声明
PHP 7 增加了对返回类型声明的支持。 类似于参数类型声明，返回类型声明指明了函数返回值的类型。可用的类型与参数声明中可用的类型相同。
```php
function arraysSum(array ...$arrays): array
{
    return array_map(function(array $array): int {
        return array_sum($array);
    }, $arrays);
}

print_r(arraysSum([1,2,3], [4,5,6], [7,8,9]));
```

#### null合并运算符
由于日常使用中存在大量同时使用三元表达式和 isset()的情况，我们添加了null合并运算符 (??) 这个语法糖。如果变量存在且值不为NULL，它就会返回自身的值，否则返回它的第二个操作数。
```php
// Fetches the value of $_GET['user'] and returns 'nobody'
// if it does not exist.
$username = $_GET['user'] ?? 'nobody';
// This is equivalent to:
$username = isset($_GET['user']) ? $_GET['user'] : 'nobody';

// Coalesces can be chained: this will return the first
// defined value out of $_GET['user'], $_POST['user'], and
// 'nobody'.
$username = $_GET['user'] ?? $_POST['user'] ?? 'nobody';
```

#### 太空船操作符（组合比较符）
太空船操作符用于比较两个表达式。当$a小于、等于或大于$b时它分别返回-1、0或1。 比较的原则是沿用 PHP 的常规比较规则进行的。
```php
// 整数
echo 1 <=> 1; // 0
echo 1 <=> 2; // -1
echo 2 <=> 1; // 1

// 浮点数
echo 1.5 <=> 1.5; // 0
echo 1.5 <=> 2.5; // -1
echo 2.5 <=> 1.5; // 1
 
// 字符串
echo "a" <=> "a"; // 0
echo "a" <=> "b"; // -1
echo "b" <=> "a"; // 1
```
#### 通过 define() 定义常量数组
Array 类型的常量现在可以通过 define() 来定义。在 PHP5.6 中仅能通过 const 定义。
```php
define('ANIMALS', [
    'dog',
    'cat',
    'bird'
]);

echo ANIMALS[1]; // 输出 "cat"
```
#### 匿名类
现在支持通过new class 来实例化一个匿名类，这可以用来替代一些“用后即焚”的完整类定义。
```php
interface Logger {
    public function log(string $msg);
}

class Application {
    private $logger;

    public function getLogger(): Logger {
         return $this->logger;
    }

    public function setLogger(Logger $logger) {
         $this->logger = $logger;
    }
}

$app = new Application;
$app->setLogger(new class implements Logger {
    public function log(string $msg) {
        echo $msg;
    }
});
var_dump($app->getLogger());
```

#### Unicode codepoint 转译语法
这接受一个以16进制形式的 Unicode codepoint，并打印出一个双引号或heredoc包围的 UTF-8 编码格式的字符串。 可以接受任何有效的 codepoint，并且开头的 0 是可以省略的。
```php
echo "\u{aa}";
echo "\u{0000aa}";
echo "\u{9999}";
```

#### Closure::call()
Closure::call() 现在有着更好的性能，简短干练的暂时绑定一个方法到对象上闭包并调用它。
```php
class A {private $x = 1;}

// PHP 7 之前版本的代码
$getXCB = function() {return $this->x;};
$getX = $getXCB->bindTo(new A, 'A'); // 中间层闭包
echo $getX(); // 1

// PHP 7+ 及更高版本的代码
$getX = function() {return $this->x;};
echo $getX->call(new A); //1
```

#### 为unserialize()提供过滤
这个特性旨在提供更安全的方式解包不可靠的数据。它通过白名单的方式来防止潜在的代码注入。
```php
// 将所有的对象都转换为 __PHP_Incomplete_Class 对象
$data = unserialize($foo, ["allowed_classes" => false]);

// 将除 MyClass 和 MyClass2 之外的所有对象都转换为 __PHP_Incomplete_Class 对象
$data = unserialize($foo, ["allowed_classes" => ["MyClass", "MyClass2"]);

// 默认情况下所有的类都是可接受的，等同于省略第二个参数
$data = unserialize($foo, ["allowed_classes" => true]);
```

#### IntlChar
新增加的[IntlChar](https://www.php.net/manual/zh/class.intlchar.php)类旨在暴露出更多的 ICU 功能。这个类自身定义了许多静态方法用于操作多字符集的 unicode 字符。
```php
printf('%x', IntlChar::CODEPOINT_MAX); // 10ffff
echo IntlChar::charName('@'); //COMMERCIAL AT
var_dump(IntlChar::ispunct('!')); //bool(true)
```

#### 预期 
预期是向后兼用并增强之前的 assert() 的方法。 它使得在生产环境中启用断言为零成本，并且提供当断言失败时抛出特定异常的能力。

老版本的API出于兼容目的将继续被维护，assert()现在是一个语言结构，它允许第一个参数是一个表达式，而不仅仅是一个待计算的 string或一个待测试的boolean。

```php
ini_set('assert.exception', 1);

class CustomError extends AssertionError {}

assert(false, new CustomError('Some error message'));

//Fatal error: Uncaught CustomError: Some error message
```
#### Group use declarations
从同一 namespace 导入的类、函数和常量现在可以通过单个 use 语句 一次性导入了。
```php
// PHP 7 之前的代码
use some\namespace\ClassA;
use some\namespace\ClassB;
use some\namespace\ClassC as C;

use function some\namespace\fn_a;
use function some\namespace\fn_b;
use function some\namespace\fn_c;

use const some\namespace\ConstA;
use const some\namespace\ConstB;
use const some\namespace\ConstC;

// PHP 7+ 及更高版本的代码
use some\namespace\{ClassA, ClassB, ClassC as C};
use function some\namespace\{fn_a, fn_b, fn_c};
use const some\namespace\{ConstA, ConstB, ConstC};
```
#### 生成器可以返回表达式
此特性基于 PHP 5.5 版本中引入的生成器特性构建的。 它允许在生成器函数中通过使用 return 语法来返回一个表达式 （但是不允许返回引用值），可以通过调用 Generator::getReturn() 方法来获取生成器的返回值，但是这个方法只能在生成器完成产生工作以后调用一次。
```php
$gen = (function() {
    yield 1;
    yield 2;

    return 3;
})();

foreach ($gen as $val) {
    echo $val, PHP_EOL;
}

echo $gen->getReturn(), PHP_EOL;
//1 2 3 
```
在生成器中能够返回最终的值是一个非常便利的特性，因为它使得调用生成器的客户端代码可以直接得到生成器（或者其他协同计算）的返回值，相对于之前版本中客户端代码必须先检查生成器是否产生了最终的值然后再进行响应处理 来得方便多了。

#### Generator delegation
现在，只需在最外层生成其中使用 yield from，就可以把一个生成器自动委派给其他的生成器，Traversable 对象或者 array。
```php
function gen()
{
    yield 1;
    yield 2;

    yield from gen2();
}

function gen2()
{
    yield 3;
    yield 4;
}

foreach (gen() as $val)
{
    echo $val, PHP_EOL;
}
//1 2 3 4
```
#### 整数除法函数 intdiv() 
新加的函数 intdiv() 用来进行 整数的除法运算。
```php
var_dump(intdiv(10, 3)); // 3
```

#### 会话选项
session_start() 可以接受一个 array 作为参数，用来覆盖 php.ini 文件中设置的 会话配置选项。

在调用 session_start() 的时候，传入的选项参数中也支持 session.lazy_write 行为，默认情况下这个配置项是打开的。它的作用是控制 PHP 只有在会话中的数据发生变化的时候才 写入会话存储文件，如果会话中的数据没有发生改变，那么 PHP 会在读取完会话数据之后，立即关闭会话存储文件，不做任何修改，可以通过设置 read_and_close 来实现。

例如，下列代码设置 session.cache_limiter 为 private，并且在读取完毕会话数据之后马上关闭会话存储文件。
```php
session_start([
    'cache_limiter' => 'private',
    'read_and_close' => true,
]);
```

#### preg_replace_callback_array()
在 PHP 7 之前，当使用 preg_replace_callback() 函数的时候，由于针对每个正则表达式都要执行回调函数，可能导致过多的分支代码。 而使用新加的 preg_replace_callback_array() 函数，可以使得代码更加简洁。

现在，可以使用一个关联数组来对每个正则表达式注册回调函数，正则表达式本身作为关联数组的键，而对应的回调函数就是关联数组的值。

#### CSPRNG Functions 
新加入两个跨平台的函数： random_bytes() 和 random_int() 用来产生高安全级别的随机字符串和随机整数。

#### 可以使用 list() 函数来展开实现了 ArrayAccess 接口的对象
在之前版本中，list() 函数不能保证 正确的展开实现了 ArrayAccess 接口的对象，现在这个问题已经被修复。

#### 其他
允许在克隆表达式上访问对象成员，例如： (clone $foo)->bar()。

#### 放宽了保留词限制
现在允许全局保留词用于类/接口/Trait 中的属性、常量和方法名。 在引入新关键词时，此变更减少了对向后兼容的破坏，避免了 API 命名的限制。

使用流畅的接口实现内部 DSL 时，这非常有用：
```php
// 以前不能用  'new'、'private' 和 'for'
Project::new('Project Name')->private()->for('purpose here')->with('username here');
```
唯一的限制是： class关键词不能用于常量名，否则会和 类名解析语法冲突 (ClassName::class)。

#### 移除 date.timezone 警告
调用任意 date- 开头或者其他基于时间的函数时，未设置 date.timezone INI 设置的情况下，之前会产生警告。 现在移除了警告（date.timezone 默认仍然是 UTC）

### 弃用的功能 
#### PHP4 风格的构造函数
PHP4 风格的构造函数（方法名和类名一样）将被弃用，并在将来移除。 如果在类中仅使用了 PHP4 风格的构造函数，PHP7 会产生 E_DEPRECATED 警告。 如果还定义了 __construct() 方法则不受影响。
```php
class foo {
    function foo() {
        echo 'I am the constructor';
    }
}
//Deprecated: Methods with the same name as their class will not be constructors in a future version of PHP; foo has a deprecated constructor in example.php on line 3
```
#### 静态调用非静态的方法
废弃了 静态（Static） 调用未声明成 static 的方法，未来可能会彻底移除该功能。
```php
class foo {
    function bar() {
        echo 'I am not static!';
    }
}

foo::bar();
// Deprecated: Non-static method foo::bar() should not be called statically in - on line 8
// I am not static!
```

#### password_hash() 盐值选项
废弃了 password_hash() 函数中的盐值选项，阻止开发者生成自己的盐值（通常更不安全）。 开发者不传该值时，该函数自己会生成密码学安全的盐值。因此再无必要传入自己自定义的盐值。

#### 变更的函数
- debug_zval_dump() 现在打印 "int" 替代 "long", 打印 "float" 替代 "double"
- dirname() 增加了可选的第二个参数, depth, 获取当前目录向上 depth 级父目录的名称。
- getrusage() 现在支持 Windows.
- mktime() and gmmktime() 函数不再接受 is_dst 参数。
- preg_replace() 函数不再支持 "\e" (PREG_REPLACE_EVAL). 应当使用 preg_replace_callback() 替代。
- setlocale() 函数不再接受 category 传入字符串。 应当使用 LC_* 常量。
- exec(), system() and passthru() 函数对 NULL 增加了保护.
- shmop_open() 现在返回一个资源而非一个int，这个资源可以传给shmop_size(), shmop_write(), shmop_read(), shmop_close() 和 shmop_delete().
- substr() 现在当 start 的值与 string 的长度相同时将返回一个空字符串。
- 为了避免内存泄露，xml_set_object() 现在在执行结束时需要手动清除 $pars

参考：
- https://www.php.net/manual/zh/appendices.php
- https://zhuanlan.zhihu.com/p/34429176
- http://www.laruence.com/2018/04/08/3170.html
