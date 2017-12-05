---
title: '[备忘]C语言入门及编译'
date: 2017-12-05 17:09:48
tags:
categories:
- IT
- java
- other
---
<!-- toc -->

环境：CentOS 6
gcc version： 4.4.7

## Hello World(编译一个文件)
vi helloworld.c
```c
#include <stdio.h>

int main(int argc, char *argv[])
{
	printf("Hello, world\n");

	return 0;
}
```

```bash
# 编译
# -o 作用指定输出文件名，如果没有指定例子的helloworld将变成a.out
gcc -o helloworld  helloworld.c

# 运行
./helloworld
```

## 共享函数库
静态函数库：每次当应用程序和静态连接的函数库一起编译时，任何引用的库函数中的代码都会被直接包含进最终的二进制程序。
共享函数库：包含每个库函数的单一全局版本，它在所有应用程序之间共享。
sayHi.c
```c
#include <stdio.h>
void sayHi(void)
{
        printf("Hi,C\n");
}
```

main.c
```c
#include <stdio.h>

void sayHi(void);

int main(int argc, char *argv[])
{
    sayHi();

    return 0;
}
```

制作libsayhi.so共享库
```bash
gcc -fPIC -c sayHi.c

gcc -shared -o libsayhi.so sayHi.o

# 可以写成一条命令
gcc -fPIC -shared -o libsayhi.so sayHi.c

```

```bash
# 编译main.c
# -lsayhi 告诉编译器引用libsayhi.so库
# -L 告诉编译器库所在位置，"."表示当前目录
gcc -o main -lsayhi -L. main.c

# 运行
 ./main
```

可能会报错：
./main: error while loading shared libraries: libsayhi.so: cannot open shared object file: No such file or directory
程序找不到libsayhi.so库位置，可以使用ldd查看
```bash
[root@localtest c_test]# ./main
./main: error while loading shared libraries: libsayhi.so: cannot open shared object file: No such file or directory

[root@localtest c_test]# ldd main
        linux-vdso.so.1 =>  (0x00007fffbd3ff000)
        libsayhi.so => not found
        libc.so.6 => /lib64/libc.so.6 (0x0000003117200000)
        /lib64/ld-linux-x86-64.so.2 (0x0000003116e00000)
```
解决：
需要设置一个环境变量LD_LIBRARY_PATH来制定额外的共享函数库搜索路径
```bash
#把LD_LIBRARY_PATH设置为so库所在目录
export LD_LIBRARY_PATH=`pwd`
```


