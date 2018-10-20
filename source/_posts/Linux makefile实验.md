---
title: Linux makefile实验
date: 2018-04-25 00:08:35
tags:
	- "Linux实验"

categories:
	- "Linux"

---

按照以下目录结构存放程序，然后制作makefile文件，把makefile文件内容填入表格中。



现有一个程序由5个文件组成:

```cpp
/* ./main.c */
#include "mytool1.h"
#include "mytool2.h"
int main()
{
	mytool1_print("hello mytool1!\n");
	mytool2_print("hello mytool2!\n");
	return 0;
}
```
<!--more-->
```cpp
/* ./functions/mytool1.c */
#include "mytool1.h"
#include <stdio.h>
void mytool1_print(char *print_str)
{
	printf("This is mytool1 print : %s ",print_str);
}
```

```cpp
/* ./functions/mytool1.h */
#ifndef _MYTOOL_1_H
#define _MYTOOL_1_H
	void mytool1_print(char *print_str);
#endif
```

```cpp
/* ./functions/mytool2.c */
#include "mytool2.h"
#include <stdio.h>
 
void mytool2_print(char *print_str)
{
	printf("This is mytool2 print : %s ",print_str);
}
```

```cpp
/* ./functions/mytool2.h */
#ifndef _MYTOOL_2_H
#define _MYTOOL_2_H
	void mytool2_print(char *print_str);
#endif
```

```bash
./Makefile
main:mytool1.o mytool2.o main.o
	gcc mytool1.o mytool2.o main.o -o main
main.o:main.c ./functions/mytool1.h ./functions/mytool2.h
	gcc -c main.c -o main.o -I functions
mytool1.o:./functions/mytool1.c ./functions/mytool1.h
	gcc -c ./functions/mytool1.c -o mytool1.o
mytool2.o:./functions/mytool2.c ./functions/mytool2.h
	gcc -c ./functions/mytool2.c -o mytool2.o
clean:
	rm -rf *.o main
```

输出结果：

![20180425214632493.jpg](https://i.loli.net/2018/09/12/5b99240c3c17d.jpg)