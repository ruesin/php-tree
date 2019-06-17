# 传值 & 传引用

传值：是把实参的值赋值给行参，那么对行参的修改，不会影响实参的值。

传引用 ：真正的以地址的方式传递参数传递以后，行参和实参都是同一个对象，只是他们名字不同而已对行参的修改将影响实参的值。

按值传递时，php必须复制值，而按引用传递则不需要复制值，引用传递一般用于大字符串或对象。

传值的话，如果是非对象，会传一个值的拷贝，对这个变量做任何改动都不影响原值。传引用或者传对象，是传真实的内存地址，对这个变量做的改动会影响原值。

## 引用

在 PHP 中引用意味着用不同的名字访问同一个变量内容。这并不像 C 的指针：例如你不能对他们做指针运算，他们并不是实际的内存地址……。 替代的是，引用是符号表别名。注意在PHP 中，变量名和变量内容是不一样的，因此同样的内容可以有不同的名字。最接近的比喻是 Unix 的文件名和文件本身——变量名是目录条目，而变量内容则是文件本身。引用可以被看作是 Unix 文件系统中的硬链接。

PHP 的引用允许用两个变量来指向同一个内容。
```php
$a =& $b;
```
`$a`和`$b`指向了同一个变量，`$a`和`$b`在这里是完全相同的，这并不是 $a 指向了 $b 或者相反，而是 $a 和 $b 指向了同一个地方。

如果具有引用的数组被拷贝，其值不会解除引用。对于数组传值给函数也是如此。

如果对一个未定义的变量进行引用赋值、引用参数传递或引用返回，则会自动创建该变量。

## 变量

变量有两个组成部分：变量名、变量值，PHP中可以将其对应为：zval、zend_value，PHP中变量的内存是通过引用计数进行管理的，而且PHP7中引用计数是在zend_value而不是zval上，变量之间的传递、赋值通常也是针对zend_value。

PHP中可以通过`$`关键词定义一个变量：`$a`;，在定义的同时可以进行初始化：`$a = "hi~";`，注意这实际是两步：定义、初始化，只定义一个变量也是可以的，可以不给它赋值。

在`./Zend/zend_types.h`中定义了变量的结构：

```c
typedef struct _zval_struct     zval;

typedef union _zend_value {
	zend_long         lval;				/* long value */ //int整型
    double            dval;				/* double value */ //浮点型
	zend_refcounted  *counted;
	zend_string      *str; //string字符串
	zend_array       *arr; //array数组
	zend_object      *obj; //object对象
	zend_resource    *res; //resource资源类型
	zend_reference   *ref; //引用类型，通过&$var_name定义的
	zend_ast_ref     *ast; //下面几个都是内核使用的value
	zval             *zv;
	void             *ptr;
	zend_class_entry *ce;
	zend_function    *func;
	struct {
		uint32_t w1;
		uint32_t w2;
	} ww;
} zend_value;

struct _zval_struct {
	zend_value        value;			/* value */ //变量实际的value
	union {
		struct {
			ZEND_ENDIAN_LOHI_4( //为了兼容大小字节序，小字节序就是下面的顺序，大字节序则下面4个顺序翻转
				zend_uchar    type,			/* active type */ //变量类型
				zend_uchar    type_flags,  //类型掩码，不同的类型会有不同的几种属性，内存管理会用到
				zend_uchar    const_flags,
				zend_uchar    reserved)	    /* call info for EX(This) */ //call info，zend执行流程会用到
		} v;
		uint32_t type_info; //上面4个值的组合值，可以直接根据type_info取到4个对应位置的值
	} u1;
	union {
		uint32_t     next;                 /* hash collision chain */ //哈希表中解决哈希冲突时用到
		uint32_t     cache_slot;           /* literal cache slot */
		uint32_t     lineno;               /* line number (for ast nodes) */
		uint32_t     num_args;             /* arguments number for EX(This) */
		uint32_t     fe_pos;               /* foreach position */
		uint32_t     fe_iter_idx;          /* foreach iterator index */
		uint32_t     access_flags;         /* class constant access flags */
		uint32_t     property_guard;       /* single property guard */
		uint32_t     extra;                /* not further specified */
	} u2; //一些辅助值
};
```

zval结构内嵌一个union类型的zend_value保存具体变量类型的值或指针，zval中还有两个union：u1、u2:

u1：变量的类型就通过u1.v.type区分，另外一个值type_flags为类型掩码，在变量的内存管理、gc机制中会用到。

u2：是个辅助值，假如zval只有:value、u1两个值，整个zval的大小也会对齐到16byte，既然不管有没有u2大小都是16byte，把多余的4byte拿出来用于一些特殊用途还是很划算的，比如next在哈希表解决哈希冲突时会用到，还有fe_pos在foreach会用到。

从zend_value可以看出，除long、double类型直接存储值外，其它类型都为指针，指向各自的结构。

## 变量引用

Zend通过引用计数，多个变量可以共享同一份数据，避免频繁拷贝带来的大量消耗。

一个zval变量容器，除了包含变量的类型和值，还包括两个字节的额外信息。

- `is_ref` 标识这个变量是否是属于引用集合，（取值 false 或 1）。 
- `refcount` 记录zval被变量的引用次数，表示指向这个zval变量容器的变量个数，即引用计数。

利用 xdebug 可以查看变量的 is_ref 和 refcount 值。

在进行赋值操作时，zend将变量指向相同的zval同时ref_count++，在unset操作时，对应的ref_count-1。只有refcount减为0时才会真正执行销毁操作。如果是引用赋值，则zend会修改is_ref为1。

PHP变量通过引用计数实现变量共享数据，那如果改变其中一个变量值呢？当试图写入一个变量时，Zend若发现该变量指向的zval被多个变量共享，则为其复制一份refcount为1的zval，并递减原zval的refcount，这个过程称为“zval分离”。

当 is_ref为1 时，对指向zval的其中一个变量重新赋值，则直接改变zval的值，此时所有指向本zval的变量值都发生改变。

1. 将变量值赋值给另一个变量，不会生成新的zval，而是在本zval的引用计数refcount + 1。
```php
$a = 333;
xdebug_debug_zval('a'); //(refcount=1, is_ref=0),int 333 
$b = $a;
xdebug_debug_zval('a'); //(refcount=2, is_ref=0),int 333
```

2. 使用引用传值赋值给另一个变量，不会生成新的zval，而是在本zval的引用计数refcount + 1，同时将is_ref改为1；
```php
$a = 333;
xdebug_debug_zval('a'); //(refcount=1, is_ref=0),int 333 
$b = &$a;
xdebug_debug_zval('a'); //(refcount=2, is_ref=1),int 333
```

3. 当试图写入一个变量时，Zend若发现该变量指向的zval被多个变量共享，则为其复制一份refcount为1的zval，并递减原zval的refcount，这个过程称为“zval分离”。
```php
$a = 333;
xdebug_debug_zval('a'); //(refcount=1, is_ref=0),int 333 
$b = $a;
xdebug_debug_zval('a'); //(refcount=2, is_ref=0),int 333 
$b = 444;
xdebug_debug_zval('a'); //(refcount=1, is_ref=0),int 333 
xdebug_debug_zval('b'); //(refcount=1, is_ref=0),int 444
```

4. 引用传值时，对指向zval的其中一个变量重新赋值，则直接改变zval的值，此时所有指向本zval的变量值都发生改变。
```php
$a = 333;
xdebug_debug_zval('a'); //(refcount=1, is_ref=0),int 333 
$b = &$a;
xdebug_debug_zval('a'); //(refcount=2, is_ref=1),int 333 
$b = 444;
xdebug_debug_zval('a'); //(refcount=2, is_ref=1),int 444 
xdebug_debug_zval('b'); //(refcount=2, is_ref=1),int 444
```

参考：
- https://www.php.net/manual/zh/language.references.php