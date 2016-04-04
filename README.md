###单例模式
就是一个类只让它实例化一个对象，比如数据库连接，有时只要一个连接，用一个对象就够了。
```php
<?php
namespace IMooc;

class Database
{
    static private $db;

    static function getInstance()
    {
        if (empty(self::$db)) {
            self::$db = new self;
            return self::$db;
        } else {
            return self::$db;
        }
    }

}
```

###策略模式
比如说要根据性别显示提供内容，但内容的种类都是相同的，这时可能要写if else来为每个类别输出不同的内容，当每个类别的内容都要写if else 就有点多了，而且万一以后有要增加判断就要改多处if。策略模式，以性别为例，就是先为类别统一接口，再分别为男性女性建立class.php实现接口，然后一开始就只用一次if,根据性别实例化类,以后就不用每次都判断了，因为实例化的对象就已经根据性别而不同了，调用它的已经被统一的属性方法就好了，以后要再添新的判断依据，新建一个类，在一个地方多加句else if就好了
接口文件
```php
interface UserStrategy {
    function showAd();
    function showCategory();
} 
```
男性
```php
<?php
class MaleUserStrategy implements UserStrategy  {

    function showAd()
    {
        echo "IPhone6";
    }

    function showCategory()
    {
        echo "电子产品";
    }
} 
```
女性
```php
<?php

class FemaleUserStrategy implements UserStrategy {
    function showAd()
    {
        echo "2014新款女装";
    }
    function showCategory()
    {
        echo "女装";
    }
} 
```
```php
class Page
{
    protected $strategy;
    function index()
    {
        echo "AD:";
        $this->strategy->showAd();
        echo "<br/>";

        echo "Category:";
        $this->strategy->showCategory();
        echo "<br/>";
    }

    function setStrategy(\UserStrategy $strategy)
    {
        $this->strategy = $strategy;
    }
}

$page = new Page;
if (isset($_GET['female'])) {
    $strategy = new FemaleUserStrategy();
} else {
    $strategy = new MaleUserStrategy();
}
$page->setStrategy($strategy);
$page->index();
```
###适配器模式
这个用了interface了， 我觉得就是统一规范用的。
```php
interface IDatabase
{
    function connect($host, $user, $passwd, $dbname);
    function query($sql);
    function close();
}
```
###工厂模式
用工厂类里面的方法来实例化对象或执行方法，还可以传参，比如建一个数据库工厂类，里面有个建立数据库方法，由因为各个数据库已经事先根据工厂里面的方法，编写了相应的方法，这时只要穿个参数，比如‘PDO’，‘MySQL'给它，工厂类就能生成相应的对象。用这种方式，以后编写的类就要按照工厂类里面要求编写，只要执行工厂里的方法就好了。还像工厂类还有几种，我还没有看。

DB工厂类
```php
<?php

class DB {

	public static $db;

	public static function init($dbtype, $config) {
		self::$db = new $dbtype;
		self::$db->connect($config);
	}

	public static function query($sql){
		return self::$db->query($sql);
	}
	......
}

?>
```
###注册树模式
一般会在初始化用来将一些对象注册到一个全局的树上，供全局访问。其他地方就不用写都new来实例化，调用树上的对象即可。
```php
<?php
class Register {

    protected static  $objects;
    function set($alias, $object){
        self::$objects[$alias] = $object;
    }

    static function get($key)
    {
        if (!isset(self::$objects[$key]))
        {
            return false;
        }
        return self::$objects[$key];
    }
    
    function _unset($alias){
        unset(self::$objects[$alias]);
    }
}
```
###观察者模式
好心水这个。一个事件发生后还有一系列的对象要跟着做，以往就是写在后面，可这样代码耦合度就搞了，多了之后就不好维护了。
要建事件以及观察者
事件类里要由触发事件的方法，然后就是要有通知其他要更新的类的方法，可以把这些写在一个抽象类里面
```php
abstract class EventGenerator {

    private $observers = array();//被添加的观察者

    function addObserver (Observer $observer) { /添加观察者
        $this->observers[] = $observer;
    }

    function notify () { //通知观察者
        foreach($this->observers as $observer){
            $observer->updata();
        }
    }
}
```
接下来就是观察者的接口
```php
interface Observer {
    function updata();
}
```
下面是具体的事件类和观察者
```php
<?php
class touch extends \IMooc\EventGenerator{ //事件发生类

    function happen(){
        echo "事件发生\n";
        $this->notify();
    }
}

class Observer1 implements \IMooc\Observer { //事件发生时要跟新的类1
    function updata()
    {
        echo "逻辑1\n";
    }
}

class Observer2 implements \IMooc\Observer { //事件发生时要跟新的类2
    function updata()
    {
        echo "逻辑2\n";
    }
}

$event = new touch();
$event->addObserver(new Observer1());//添加一个观察者;
$event->addObserver(new Observer2());//可添加多个观察者
$event->happen();//触发事件
```
执行结果
```
事件发生
逻辑1
逻辑2
```
添加和删除观察者就可以方便的添加或去掉事件发生后的各种逻辑
###原型模式
clone对象。当要用一个类多次，初始化的计算开销比较大时，代码又重复一堆，可以初始化后克隆对象，再分别执行操作，节省资源，提高效率。
```php
$prototype = new IMooc\Canvas();//原型
$prototype->init();//初始化

$canvas1 = clone $prototype;//克隆
$canvas1->rect(3,6,4,12);
$canvas1->draw();

echo "===========================<br/>";

$canvas2 = clone $prototype;//克隆
$canvas2->rect(1, 3, 2, 6);
$canvas2->draw();
```
###装饰器模式
类似观察者模式，还是把装饰分别在外面加上了，添东西时就不用改里面，添加删除都可以方便的在外面执行
首先定义一个接口
```php
interface DrawDecorator
{
    function beforeDraw();
    function afterDraw();
}
```
可以按照接口写装饰器
颜色装饰器：
```php
class ColorDrawDecorator implements DrawDecorator
{
    protected $color;
    function __construct($color = 'red')
    {
        $this->color = $color;
    }
    function beforeDraw()
    {
        echo "<div style='color: {$this->color};'>";
    }
    function afterDraw()
    {
        echo "</div>";
    }
}
```
然后就能用了
```php

$canvas = new IMooc\Canvas();
$canvas->addDecorator(new \IMooc\ColorDrawDecorator('green'));//添加颜色撞色器
$canvas->addDecorator(new \IMooc\SizeDrawDecorator());//添加大小装饰器
$canvas->init();
$canvas->rect(3,6, 4,12);
$canvas->draw();
```
###迭代器
这个方法有点多，一开始看的一头雾水，看完代码觉得很奇怪，为什么方法自己就按顺序执行了，后来发现好像是foreach指挥的。先是rewind方法（返回到迭代器的第一个元素），再是valid方法（检查当前位置是否有效），再是current方法（返回当前元素），最后是key方法（返回当前元素的键）。第一次过后，后面除了rewind换成next方法（向前移动到下一个元素）外，其余仍旧相同。
我看不懂迭代器的有什么意义，网上讲得好抽象啊（主要是因为我水平不够），下面这句话是什么意思啊
> 不必关心多种多样的数据结构的具体细节

```php
<?php
class myIterator implements Iterator {
    private $position = 0;
    private $array = array(
        "firstelement",
        "secondelement",
        "lastelement",
    );  

    public function __construct() {
        $this->position = 0;
    }

    function rewind() {
        var_dump(__METHOD__);
        $this->position = 0;
    }

    function current() {
        var_dump(__METHOD__);
        return $this->array[$this->position];
    }

    function key() {
        var_dump(__METHOD__);
        return $this->position;
    }

    function next() {
        var_dump(__METHOD__);
        ++$this->position;
    }

    function valid() {
        var_dump(__METHOD__);
        return isset($this->array[$this->position]);
    }
}

$it = new myIterator;

foreach($it as $key => $value) {
    var_dump($key, $value);
    echo "\n";
}
?>
```
