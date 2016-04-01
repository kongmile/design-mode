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