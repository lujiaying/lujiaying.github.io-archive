title: 在源文件间共享变量--C
date: 2015-06-04 20:59:53
tags: [C]
---
在c语言中，使用extern关键字是解决上述问题的最好方法。假设你正在编写一个含有多个源文件的程序，例如，在file1.c中定义了一个变量，在file2.c中需要引用该变量。
<!-- more -->

在具体讲解之前，需要先了解定义变量和声明变量间的区别：

> 当定义一个变量时，编译器将为该变量分配内存空间。
> 
> 当声明一个变量时，编译器被告知该变量存在（以及它的类型是..）；但此时并不会为其分配内存空间。

一个变量可以被声明多次（尽管一次就足够了），但在指定域中只能被定义一次。

## 实现方法

`file3.h`
```c

extern int global_variable;   /* Declaration of the variable */
```

`file1.c`
```c
#include "file3.h"   /* Declaration made available here */

int global_variable = 37;  /* Definition checked against declaration */
```

`file2.c`
```c
#include "file3.h"

#include <stdio.h>

int main(void)

{

        printf("Global variable: %d\n", global_variable++);

        return 0;

}
```

`运行结果`
```shell
$ gcc file1.c file2.c file3.h -o file

$ ./file

Global variable: 37
```

[更多实现方法](http://stackoverflow.com/questions/1433204/how-do-i-share-a-variable-between-source-files-in-c-with-extern-but-how)
