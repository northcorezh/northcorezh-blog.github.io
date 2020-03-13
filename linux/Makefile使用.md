
# Makefile 使用
  Makefile带来的好处就是——“自动化编译”，一但写好，只需要一个 make 命令，整个工程便可以完全编译，极大的提高了软件的开发效率（特别是对于那些项目较大、文件较多的工程）。  

  一个工程中的源文件不计其数，按其类型、功能、模块分别放在若干个目录中。makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至进行更复杂的功能操作(因为makefile就像一个shell脚本一样，可以执行操作系统的命令)。  

  make工具最主要也是最基本的功能就是根据makefile文件中描述的源程序至今的相互关系来完成自动编译、维护多个源文件工程。而makefile文件需要按某种语法进行编写，文件中需要说明如何编译各个源文件并链接生成可执行文件，要求定义源文件之间的依赖关系。  

## Makefile 基本规则
下面从一个简单实例入手，介绍如何编写Makefile。假设现在有一个简单的项目由几个文件组成：prog.c、 code.c、 code.h。这些文件的内容如下：  
prog.c  
```C
#include <stdio.h>
#include "code.h"

int main(void)
{
    int i = 1;      
    printf ("myfun(i) = %d\n", myfun(i));
}
```

code.c  
```C
#include "code.h"

int myfun(int in)
{
    return in + 1;
}
```

code.h  
```C
extern int myfun(int);
```

这些程序都比较短，结构也很清晰，因此使用下面的命令进行编译：  
```Shell
$ gcc -c code.c -o code.o
$ gcc -c prog.c -o prog.o
$ gcc prog.o code.o -o test
```

如上所示，这样就能生成可执行文件test，由于程序比较简单，而且数量也比较少，因此看不出来有多麻烦。但是，试想如果不只上面的3个文件，而是几十个或者是成百上千个甚至更多，那将是非常复杂的问题。  

那么如何是好呢？这里就是makefile的绝佳舞台，下面是一个简单的makefile的例子。  
首先$ vim Makefile  
```
test: prog.o code.o
        gcc prog.o code.o -o test
prog.o: prog.c code.h
        gcc -c prog.c -o prog.o
code.o: code.c code.h
        gcc -c code.c -o code.o
clean:
        rm -f *.o test
```

有了这个Makefile，不论什么时候修改源文件，只要执行一下make命令，所有必要的重新编译将自动执行。make程序利用Makefile中的数据，生成并遍历以test为根节点的树；现在我们以上面的实例，来学习一下Makefile的一般写法：  

```
test(目标文件): prog.o code.o(依赖文件列表)
tab(至少一个tab的位置) gcc prog.o code.o -o test(命令)
.......
```

最后就会多产生： porg.o code.o test这三个文件，执行./test查看结果  
还记得Makefile中的clean吗？ make clean就会去执行rm -f *.o test这条命令，完成 clean 操作。  

## 使用带宏的Makefile
Makefile还可以定义和使用宏(也称做变量)，从而使其更加自动化，更加灵活，在Makefile中定义宏的格式为：  
```
macroname = macrotext
```
使用宏的格式为：  
```
$(macroname)
```

用 “宏” 的方式，来改写上面的 Makefile 例子。  
```
OBJS = prog.o code.o
CC = gcc
test: $(OBJS)
        $(CC) $(OBJS) -o test
prog.o: prog.c code.h
        $(CC) -c prog.c -o prog.o
code.o: code.c code.h
        $(CC) -c code.c -o code.o
clean:
        rm -f *.o test
```
