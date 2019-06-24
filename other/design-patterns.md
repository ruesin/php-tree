# 设计模式

## 创建型模式

创建型模式(Creational Pattern)对类的实例化过程进行了抽象，能够将软件模块中对象的创建和对象的使用分离。为了使软件的结构更加清晰，外界对于这些对象只需要知道它们共同的接口，而不清楚其具体的实现细节，使整个系统的设计更加符合单一职责原则。

创建型模式在创建什么(What)，由谁创建(Who)，何时创建(When)等方面都为软件设计者提供了尽可能大的灵活性。创建型模式隐藏了类的实例的创建细节，通过隐藏对象如何被创建和组合在一起达到使整个系统独立的目的。

### 简单工厂模式( Simple Factory Pattern )

简单工厂模式(Simple Factory Pattern)：又称为静态工厂方法(Static Factory Method)模式。简单工厂模式专门定义一个类来负责创建其他类的实例，可以根据参数的不同返回不同类的实例，被创建的实例通常都具有共同的父类。

简单工厂模式包含如下角色：
- Factory：工厂角色负责实现创建所有实例的内部逻辑。
- Product：抽象产品角色是所创建的所有对象的父类，负责描述所有实例所共有的公共接口。
- ConcreteProduct：具体产品角色是创建目标，所有创建的对象都充当这个角色的某个具体类的实例。

使用简单工厂模式可以将产品的“消费”和“生产”完全分开，客户端只需要知道自己需要什么产品，如何来使用产品就可以了，具体的产品生产任务由具体的工厂类来实现。工厂类根据传进来的参数生产具体的产品供消费者使用。这种模式使得更加利于扩展，当有新的产品加入时仅仅需要在工厂中加入新产品的构造就可以了。

简单工厂模式最大的问题在于工厂类的职责相对过重，增加新的产品需要修改工厂类的判断逻辑，这一点与开闭原则是相违背的。

```php
//创建抽象产品类“人”
abstract class Person
{
    abstract public function say();
}
//创建具体产品类（继承抽象产品类）：“男人”、“女孩”
class Man extends Person
{
    public function say()
    {
        echo 'I am a strong man!';
    }
}
class Girl extends Person
{
    public function say()
    {
        echo 'I am a smarty girl!';
    }
}

//创建工厂类，通过创建静态方法从而根据传入不同参数创建不同具体实现类的实例
class Factory
{
    public static function createPerson($person)
    {
        switch ($person) {
            case 'girl':
                return new Girl();
                break;
            case 'man':
                return new Man();
                break;
            default:
                return new Man();
                break;
        }
    }
}
Factory::createPerson('girl')->say(); //I am a smarty girl!
```

### 工厂方法模式(Factory Method Pattern)

工厂方法模式(Factory Method Pattern)又称为工厂模式，也叫虚拟构造器(Virtual Constructor)模式或者多态工厂(Polymorphic Factory)模式。

在工厂方法模式中，工厂父类负责定义创建产品对象的公共接口，而工厂子类则负责生成具体的产品对象，这样做的目的是将产品类的实例化操作下推到工厂子类中完成，即通过工厂子类来确定究竟应该实例化哪一个具体产品类。

工厂方法模式包含如下角色：
- Product：抽象产品
- ConcreteProduct：具体产品
- Factory：抽象工厂
- ConcreteFactory：具体工厂

工厂方法模式是简单工厂模式的进一步抽象和推广。由于使用了面向对象的多态性，工厂方法模式保持了简单工厂模式的优点，而且克服了它的缺点。

在工厂方法模式中，核心的工厂类不再负责所有产品的创建，而是将具体创建工作交给子类去做。核心类仅仅负责给出具体工厂必须实现的接口，而不负责哪一个产品类被实例化这种细节，这使得工厂方法模式可以允许系统在不修改工厂角色的情况下引进新产品，更加符合“开闭原则”。

```php
//产品类（Person及其子类）不做变更

//工厂类更新为 抽象类（接口） + 两个实现
interface Factory
{
    public static function createPerson();
}
class GirlFactory implements Factory
{
    public static function createPerson()
    {
        return new Girl();
    }
}
class ManFactory implements Factory
{
    public static function createPerson()
    {
        return new Man();
    }
}
ManFactory::createPerson()->say();
```

### 抽象工厂模式(Abstract Factory)
抽象工厂模式(Abstract Factory Pattern)：提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。抽象工厂模式又称为Kit模式。

在工厂方法模式中具体工厂负责生产具体的产品，每一个具体工厂对应一种具体产品，工厂方法也具有唯一性，一般情况下，一个具体工厂中只有一个工厂方法或者一组重载的工厂方法。但是有时候我们需要一个工厂可以提供多个产品对象，而不是单一的产品对象。

为了更清晰地理解工厂方法模式，需要先引入两个概念：
- 产品等级结构：产品等级结构即产品的继承结构，如一个抽象类是电视机（男人），其子类有海尔电视机（黑男人）、海信电视机（白男人）、TCL电视机（黄男人），则抽象电视机（男人）与具体品牌（肤色）的电视机（男人）之间构成了一个产品等级结构，抽象电视机（男人）是父类，而具体品牌（肤色）的电视机（男人）是其子类。
- 产品族 ：在抽象工厂模式中，产品族是指由同一个工厂生产的，位于不同产品等级结构中的一组产品，如海尔电器工厂（黑色任重）生产的海尔电视机（黑男人）、海尔电冰箱（黑女孩），海尔电视机（黑男人）位于电视机（男人）产品等级结构中，海尔电冰箱（黑女孩儿）位于电冰箱（女孩儿）产品等级结构中。

当系统所提供的工厂所需生产的具体产品并不是一个简单的对象，而是多个位于不同产品等级结构中属于不同类型的具体产品时需要使用抽象工厂模式。

抽象工厂模式是所有形式的工厂模式中最为抽象和最具一般性的一种形态。

抽象工厂模式与工厂方法模式最大的区别在于，工厂方法模式针对的是一个产品等级结构，而抽象工厂模式则需要面对多个产品等级结构，一个工厂等级结构可以负责多个不同产品等级结构中的产品对象的创建 。当一个工厂等级结构可以创建出分属于不同产品等级结构的一个产品族中的所有对象时，抽象工厂模式比工厂方法模式更为简单、有效率。

抽象工厂模式包含如下角色：
- Factory：抽象工厂
- ConcreteFactory：具体工厂
- AbstractProduct：抽象产品
- Product：具体产品

```php
//人
interface Person
{
    public function say();
}
interface Man extends Person
{
}
interface Girl extends Person
{
}

class BlackMan implements Man
{
    public function say()
    {
        echo 'I am a black man!' . PHP_EOL;
    }
}
class WhiteMan implements Man
{
    public function say()
    {
        echo 'I am a white man!' . PHP_EOL;
    }
}
class BlackGirl implements Girl
{
    public function say()
    {
        echo 'I am a black girl!' . PHP_EOL;
    }
}
class WhiteGirl implements Girl
{
    public function say()
    {
        echo 'I am a white girl!' . PHP_EOL;
    }
}

//工厂
interface Factory
{
    public function createWhite();

    public function createBlack();
}
class ManFactory implements Factory
{
    public function createWhite()
    {
        return new WhiteMan();
    }

    public function createBlack()
    {
        return new BlackMan();
    }
}
class GirlFactory implements Factory
{
    public function createWhite()
    {
        return new WhiteGirl();
    }

    public function createBlack()
    {
        return new BlackGirl();
    }
}

//使用
$manFactory = new ManFactory();
$manFactory->createWhite()->say();
$manFactory->createBlack()->say();

$girlFactory = new GirlFactory();
$girlFactory->createWhite()->say();
$girlFactory->createBlack()->say();
```

如果此时需要增加增加一个“男孩儿”产品等级，只需要增加
```php
interface Boy extends Person
{
}
class BlackBoy implements Man
{
    public function say()
    {
        echo 'I am a black boy!' . PHP_EOL;
    }
}
class WhiteBoy implements Man
{
    public function say()
    {
        echo 'I am a white boy!' . PHP_EOL;
    }
}

class BoyFactory implements Factory
{
    public function createWhite()
    {
        return new WhiteBoy();
    }

    public function createBlack()
    {
        return new BlackBoy();
    }
}
```

如果此时需要增加一个肤色 - 黄种人，需要增加实现
```php
class YellowMan implements Man
{
    public function say()
    {
        echo 'I am a yellow man!' . PHP_EOL;
    }
}

class YellowGirl implements Girl
{
    public function say()
    {
        echo 'I am a yellow girl!' . PHP_EOL;
    }
}

class YellowBoy implements Boy
{
    public function say()
    {
        echo 'I am a yellow boy!' . PHP_EOL;
    }
}

interface Factory
{
    //...
    // 工厂接口增加 createYellow 方法约束
    public function createYellow();
    //...
}

class ManFactory implements Factory
{
    //...
    // 工厂 ManFactory 增加 createYellow 实现
    public function createYellow()
    {
        return new YellowMan();
    }
    //...
}

class GilrFactory implements Factory
{
    //...
    // 工厂 GilrFactory 增加 createYellow 实现
    public function createYellow()
    {
        return new YellowGilr();
    }
    //...
}

class BoyFactory implements Factory
{
    //...
    // 工厂 BoyFactory 增加 createYellow 实现
    public function createYellow()
    {
        return new YellowBoy();
    }
    //...
}
```

参考：https://www.zhihu.com/question/20367734

### 建造者模式（Builder Pattern）
建造者模式(Builder Pattern)：又称生成器模式，将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

建造者模式是一步一步创建一个包含多个组成部件的复杂对象，它允许用户只通过指定复杂对象的类型和内容就可以构建它们，用户不需要知道内部的具体构建细节，相同的构建过程可以创建不同的产品。

复杂对象相当于一辆有待建造的汽车，而对象的属性相当于汽车的部件，建造产品的过程就相当于组合部件的过程。

对大多数用户而言，无须知道这些部件的装配细节，也几乎不会使用单独某个部件，而是使用一辆完整的汽车，可以通过建造者模式对其进行设计与描述，建造者模式可以将部件和其组装过程分开，一步一步创建一个复杂的对象。用户只需要指定复杂对象的类型就可以得到该对象，而无须知道其内部的具体构造细节。

由于组合部件的过程很复杂，因此，这些部件的组合过程往往被“外部化”到一个称作建造者的对象里，建造者返还给客户端的是一个已经建造完毕的完整产品对象，而用户无须关心该对象所包含的属性以及它们的组装方式，这就是建造者模式的模式动机。

在软件开发中，也存在大量类似汽车一样的复杂对象，它们拥有一系列成员属性，这些成员属性中有些是引用类型的成员对象。而且在这些复杂对象中，还可能存在一些限制条件，如某些属性没有赋值则复杂对象不能作为一个完整的产品使用；有些属性的赋值必须按照某个顺序，一个属性没有赋值之前，另一个属性可能无法赋值等。

建造者模式包含如下角色：
- Builder：抽象建造者，将建造的具体过程交与它的子类来实现，更容易扩展。一般至少会有两个抽象方法，一个用来建造产品，一个是用来返回产品。
- ConcreteBuilder：具体建造者。
- Director：指挥者，负责调用适当的建造者来组建产品，一般不与产品类发生依赖关系，与建造者类接交互，一般指挥者被用来封装程序中易变的部分。
- Product：产品角色，一般是一个创建过程较为复杂的对象，一般会有比较多的代码量。实际编程中，产品类可以是由一个抽象类与它的不同实现组成，也可以是由多个抽象类与他们的实现组成。

```php
//产品角色
class Product
{
    private $name;
    private $type;

    public function showProduct()
    {
        echo "名称：" . $this->name . PHP_EOL;
        echo "型号：" . $this->type . PHP_EOL;
    }

    public function setName($name)
    {
        $this->name = $name;
    }

    public function setType($type)
    {
        $this->type = $type;
    }
}
//抽象建造者
interface Builder
{
    public function setPart($name, $type);

    public function getProduct();
}
//具体建造者
class ConcreteBuilder implements Builder
{
    private $product = null;

    public function __construct()
    {
        $this->product = new Product();
    }

    public function getProduct()
    {
        return $this->product;
    }

    public function setPart($name, $type)
    {
        $this->product->setName($name);
        $this->product->setType($type);
    }
}
//指挥者
class Director
{
    private $builder = null;
    public function __construct()
    {
        $this->builder = new ConcreteBuilder();
    }

    public function getAudi()
    {
        $this->builder->setPart("奥迪汽车", "Q5");
        return $this->builder->getProduct();
    }

    public function getBmw()
    {
        $this->builder->setPart("宝马汽车", "X7");
        return $this->builder->getProduct();
    }
}

$director = new Director();

$audi = $director->getAudi();
$audi->showProduct();

$bmw = $director->getBmw();
$bmw->showProduct();
```

### 单例模式
单例模式(Singleton Pattern)：单例模式确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例，这个类称为单例类，它提供全局访问的方法。

单例模式的要点有三个：一是某个类只能有一个实例；二是它必须自行创建这个实例；三是它必须自行向整个系统提供这个实例。

单例模式包含如下角色：
- Singleton：单例

单例模式的目的是保证一个类仅有一个实例，并提供一个访问它的全局访问点。单例模式包含的角色只有一个，就是单例类——Singleton。

单例类拥有一个私有构造函数，确保用户无法通过new关键字直接实例化它。除此之外，该模式中包含一个静态私有成员变量与静态公有的工厂方法，该工厂方法负责检验实例的存在性并实例化自己，然后存储在静态成员变量中，以确保只有一个实例被创建。

在单例模式的实现过程中，需要注意如下三点：
- 单例类的构造函数为私有；
- 提供一个自身的静态私有成员变量；
- 提供一个公有的静态工厂方法。

```php

class DB
{
    private static $instance = null;

    private function __construct()
    {
    }

    private function __clone()
    {
    }

    public static function getInstance()
    {
        if (self::$instance == null) {
            self::$instance = new self();
        }
        return self::$instance;
    }
}
```

参考：
- https://github.com/me115/design_patterns
- https://www.zhihu.com/question/20367734