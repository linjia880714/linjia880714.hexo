---
title: except自动登录远程服务器
date: 2017-04-12 17:14:23
tags: Linxu
categories:
- IT
- Linux
---


简单的一个例子，备份下省得忘记了

```bash
#!/usr/bin/expect
spawn ssh root@127.0.0.1
expect "*password:"
send "123\r"
expect "*#"
interact
```

如果ssh禁止root远程登录，需要设置ssh