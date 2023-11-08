# GCC

![image-20220926102807142](gcc.assets\image-20220926102807142.png)

C 或者 C++ 程序从源代码生成可执行程序的过程，需经历 4 个过程，分别是预处理、编译、汇编和链接

1. 预处理操作，主要是处理那些源文件和头文件中以 # 开头的命令（比如 #include、#define、#ifdef 等），并删除程序中所有的注释 // 和 /* ... */，GCC预处理时需要知道源文件引用的头文件位置，可以使用-I选项指定头文件搜索目录，头文件修改了，所有引用了该头文件的源文件需要重新编译

```shell
gcc -E main.c -I ../inc -o main		#main.c中引用了../inc中的头文件
```

2. 编译是整个程序构建的核心部分，也是最复杂的部分之一。所谓编译，简单理解就是将预处理得到的程序代码，经过一系列的词法分析、语法分析、语义分析以及优化，加工为当前机器支持的汇编代码
3. 汇编操作将汇编代码翻译为二进制机器码，生成可链接文件，模块源文件修改了只需要重新编译汇编此模块即可，这是节省编译程序时间的主要原理，正常情况可以只关注.o文件，有了每个模块的.o文件直接链接就可以了，所以一般直接将源码编译为.o文件

```shell
gcc -c main.c -I ../inc -o main.o	#经典生成.o文件
```
4. 链接操作将各可链接文件链接起来生成最终的可执行文件



| input    | command | output | result                                                       |
| -------- | ------- | ------ | ------------------------------------------------------------ |
| .c       | -E      | .i     | 只预处理，如果包含了自己的头文件，需要告诉gcc头文件路径<br />gcc -E main.c -I path -o main.i<br />直接使用预处理后面的操作都需要这一步，这是预处理需要的 |
| .c .i    | -S      | .s     | 预处理并编译                                                 |
| .c .i .s | -c      | .o     | 预处理、编译和汇编，默认输出到.o，.o文件即可链接文件<br />某个模块的代码修改了可单独更新其.o文件，重新链接即可<br />常用指令，将源文件直接加工成可重定位目标程序<br />可重定位目标程序链接后成为可执行目标程序 |
|          | -o      |        | 指定输出文件                                                 |
|          | -g      |        | 增加调试信息，没有调试信息无法用gdb调试                      |

| command                    | result                                                       |
| -------------------------- | ------------------------------------------------------------ |
| gcc main.c -o main         | 指定可执行文件名称                                           |
| gcc -E main.c              | 预处理main.c，与处理结果打印，但不保存至文件                 |
| gcc -E main.c -o main.i    | 预处理main.c，并生成指定文件                                 |
| gcc -E -C main.c -o main.i | 预处理main.c，并生成指定文件，包含注释                       |
| gcc -S main.i              | 将文件编译成汇编文件（可以是.c或者.i），自动生成和源文件名相同的.s文件 |
| gcc -c file                | 将目标文件加工至二进制目标文件                               |
| gcc main.o -o main         | 对二进制目标文件执行链接操作，生成最终可执行文件             |
| gcc *.c -o main            | 将所有.c文件编译成一个可执行文件main                         |
| cpp -v                     | 查看编译相关的路径                                           |

| command                               | result      |
| ------------------------------------- | ----------- |
| gcc -o a.out main.c -Wl,-Map,main.map | 生成map文件 |

map文件记录了很多有用的信息，比如内存分配地址

```shell
root@ubuntuserver:/home/liteng/code/trunk/app# gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/9/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none:hsa
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 9.4.0-1ubuntu1~20.04.1' --with-bugurl=file:///usr/share/doc/gcc-9/README.Bugs --enable-languages=c,ada,c++,go,brig,d,fortran,objc,obj-c++,gm2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-9 --program-prefix=x86_64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-vtable-verify --enable-plugin --enable-default-pie --with-system-zlib --with-target-system-zlib=auto --enable-objc-gc=auto --enable-multiarch --disable-werror --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --enable-multilib --with-tune=generic --enable-offload-targets=nvptx-none=/build/gcc-9-Av3uEd/gcc-9-9.4.0/debian/tmp-nvptx/usr,hsa --without-cuda-driver --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu
Thread model: posix
gcc version 9.4.0 (Ubuntu 9.4.0-1ubuntu1~20.04.1)
```

## 链接库

| 文件后缀 | 作用                                                 |
| -------- | ---------------------------------------------------- |
| .o       | 可链接目标文件，每个源文件对应一个                   |
| .a       | 静态库文件，由.o文件生成，可以看作很多个.o文件的集合 |
| .so      | 动态库文件，和.a文件对应，但是在运行时               |

动态库和静态库的区别在于

1. 静态库被链接到可执行程序中，编译完成后原静态库可删除，动态库链接时不同，而是运行时载入内存，所以哪怕可执行程序编译完了，动态库也不能删
2. 静态库的可执行程序体积较大，动态库的较小
3. 动态库在内存只会存在一份，如果很多程序使用了相同的动态库，那么比静态库节约内存

### 动态链接库

1. 不会编译到可执行文件中
2. 多个进程可共享，可节约内存
3. 编译时指定`LIBRARY_PATH`，指定编译时寻找lib库的路径
4. 执行可执行程序时指定`LD_LIBRARY_PATH`，程序寻找lib库的路径

| 指令                                 | 作用          |
| ------------------------------------ | ------------- |
| gcc -fPIC -shared xxx.o -o libxxx.so | 生成lib库文件 |
| gcc -fPIC -shared xxx.c -o libxxx.so | 生成lib库文件 |

|      |                    |
| ---- | ------------------ |
| -L   | 指定动态链接库路径 |
| -I   | 指定头文件路径     |

```makefile
gcc -o main -I ./ -L ./ -lhello -llist#头文件路径./，动态链接库路径./，链接动态链接库hello和list，也就是寻找libhello.so和liblist.so
```

|                        |                |
| ---------------------- | -------------- |
| 可以查看链接的动态库   | ldd 可执行文件 |
| 可以查看libc版本       | ldd --version  |
| 查看编译相关的环境变量 | cpp -v         |

[Linux的so文件到底是干嘛的？浅析Linux的动态链接库 (zhihu.com)](https://www.zhihu.com/tardis/zm/art/235551437?source_id=1005)

[(20条消息) Linux查看GLIBC版本号_xflm的博客-CSDN博客](https://blog.csdn.net/qq_37858281/article/details/122506656#:~:text=通常libc.so会支持多个版本，即向前兼容，查看该文件中包含的字符串可以看到其支持的版本，通常是连续的。 %24 strings %2Flib64%2Flibc.so.6 | grep,^GLIBC GLIBC_2.2.5 GLIBC_2.2.6 GLIBC_2.3... GLIBC_2.12 %23 即版本号为2.12)

[linux 给运行程序指定动态库路径 - Silentdoer - 博客园 (cnblogs.com)](https://www.cnblogs.com/silentdoer/p/11413783.html)

## 编译选项

优化等级区别

[(99+ 封私信 / 80 条消息) Debug模式和Release模式有什么区别？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/443340911)

|       |                                  |
| ----- | -------------------------------- |
| -w    | 编译时不显示WARN                 |
| -Wall | 编译时显示全部常见WARN           |
| -W    | 编译时显示编译器认为是错误的WARN |

[(87条消息) gcc 的 -g 和 -ggdb 选项_zlzlei的博客-CSDN博客](https://blog.csdn.net/zlzlei/article/details/7781617)
