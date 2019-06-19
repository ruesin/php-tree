# 数组对象

## ArrayObject
这个类允许像数组一样使用对象。

```
ArrayObject::append — 追加新的值作为最后一个元素。
ArrayObject::asort — Sort the entries by value
ArrayObject::__construct — Construct a new array object
ArrayObject::count — 统计 ArrayObject 内 public 属性的数量
ArrayObject::exchangeArray — Exchange the array for another one
ArrayObject::getArrayCopy — Creates a copy of the ArrayObject
ArrayObject::getFlags — Gets the behavior flags
ArrayObject::getIterator — Create a new iterator from an ArrayObject instance
ArrayObject::getIteratorClass — Gets the iterator classname for the ArrayObject
ArrayObject::ksort — Sort the entries by key
ArrayObject::natcasesort — Sort an array using a case insensitive "natural order" algorithm
ArrayObject::natsort — Sort entries using a "natural order" algorithm
ArrayObject::offsetExists — Returns whether the requested index exists
ArrayObject::offsetGet — Returns the value at the specified index
ArrayObject::offsetSet — 为指定索引设定新的值
ArrayObject::offsetUnset — Unsets the value at the specified index
ArrayObject::serialize — Serialize an ArrayObject
ArrayObject::setFlags — Sets the behavior flags
ArrayObject::setIteratorClass — Sets the iterator classname for the ArrayObject
ArrayObject::uasort — Sort the entries with a user-defined comparison function and maintain key association
ArrayObject::uksort — Sort the entries by keys using a user-defined comparison function
ArrayObject::unserialize — Unserialize an ArrayObject
```

ArrayObject 类实现了四个接口：
- IteratorAggregate：聚合式迭代器接口，创建外部迭代器的接口。
- ArrayAccess：提供像访问数组一样访问对象的能力的接口。
- Serializable：序列化接口，实现此接口的类将不再支持 __sleep() 和 __wakeup()。不论何时，只要有实例需要被序列化，serialize 方法都将被调用。它将不会调用 __destruct() 或有其他影响，除非程序化地调用此方法。当数据被反序列化时，类将被感知并且调用合适的 unserialize() 方法而不是调用 __construct()。如果需要执行标准的构造器，应该在这个方法中进行处理。
- Countable：类实现 Countable 可被用于 count() 函数.


```php
IteratorAggregate extends Traversable {
    abstract public getIterator ( void ) : Traversable
}

ArrayAccess {
    abstract public offsetExists ( mixed $offset ) : boolean
    abstract public offsetGet ( mixed $offset ) : mixed
    abstract public offsetSet ( mixed $offset , mixed $value ) : void
    abstract public offsetUnset ( mixed $offset ) : void
}

Serializable {
    abstract public serialize ( void ) : string
    abstract public unserialize ( string $serialized ) : mixed
}

Countable {
    abstract public count ( void ) : int
}
```

## ArrayIterator

这个迭代器允许在遍历数组和对象时删除和更新值与键，实际上是对ArrayObject类的补充，为后者提供遍历功能。

当你想多次遍历相同数组时你需要实例化 ArrayObject，然后让这个实例创建一个 ArrayIteratror 实例。 当你想遍历相同数组时多次你需要实例 ArrayObject 并且让这个实例创建一个 ArrayIteratror 实例，然后使用foreach 或者 手动调用 getIterator() 方法。

```php
ArrayIterator implements ArrayAccess , SeekableIterator , Countable , Serializable {
    /* 常量 */
    const integer STD_PROP_LIST = 1 ;
    const integer ARRAY_AS_PROPS = 2 ;
    /* 方法 */
    public append ( mixed $value ) : void //添加新元素
    public asort ( void ) : void
    public __construct ([ mixed $array = array() [, int $flags = 0 ]] )
    public count ( void ) : int //返回迭代器中元素个数
    public current ( void ) : mixed //返回当前数组元素
    public getArrayCopy ( void ) : array
    public getFlags ( void ) : int
    public key ( void ) : mixed //返回当前数组key
    public ksort ( void ) : void
    public natcasesort ( void ) : void
    public natsort ( void ) : void
    public next ( void ) : void //指向下个数组元素
    public offsetExists ( mixed $index ) : bool //判断提交的值是否存在
    public offsetGet ( mixed $index ) : mixed //指定 name 获取值
    public offsetSet ( mixed $index , mixed $newval ) : void //修改指定 name 的值
    public offsetUnset ( mixed $index ) : void //删除数据
    public rewind ( void ) : void //重置数组指针到头
    public seek ( int $position ) : void //查找数组中某一位置
    public serialize ( void ) : string
    public setFlags ( string $flags ) : void
    public uasort ( callable $cmp_function ) : void
    public uksort ( callable $cmp_function ) : void
    public unserialize ( string $serialized ) : string
    public valid ( void ) : bool  //检查数组是否还包含其他元素
}
```
