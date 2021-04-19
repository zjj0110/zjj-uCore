# 理解通过make生成执行文件的过程。（要求在报告中写出对下述问题的回答）

在此练习中，大家需要通过静态分析代码来了解：

1. 操作系统镜像文件ucore.img是如何一步一步生成的？(需要比较详细地解释Makefile中每一条相关命令和命令参数的含义，以及说明命令导致的结果)
2. 一个被系统认为是符合规范的硬盘主引导扇区的特征是什么？



## 1.操作系统镜像文件ucore.img是如何一步一步生成的？

首先进入Makefile文件所在位置执行make -v命令，显示make执行了那些命令。(已精简)


<pre><code> 
//编译kern/init/init.c   生成obj/kern/init/init.o
+ cc kern/init/init.c
gcc -Ikern/init/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/init/init.c -o obj/kern/init/init.o 
</code></pre>

详解:

  +cc kern/init/init.c    表示编译了 kern/init 文件中的 init.c 文件

  gcc -Ikern/init/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/init/init.c -o obj/kern/init/init.o   (实际指令)

   + -fno-builtin          不使用C语言的内建函数

   + -Wall                 生成程序中一系列的常见错误警告

   + -ggdb                 生成可供gdb使用的调试信息

   + -m32                  针对32进行代码优化

   + -gstabs               生成stabs格式的调试信息
   
   + -nostdinc             不在标准系统文件夹寻找头文件，只在-I等参数指定的文件夹中搜索头文件 

   + -Idir                 制定搜索头文件的路径

   + -fno-stack-protector  不生成用于检测缓冲区溢出的代码

   + -Os                   为减小代码大小而进行优化。
   
   + -g                    生成调试信息。

   + -c                    编译并生成目标文件

   + -o                    生成指定的输出文件

   + -c kern/init/init.c -o obj/kern/init/init.o    表示将kern/init/init.c文件编译为输出文件obj/kern/init/init.o
 
<pre><code> 
+ cc kern/libs/readline.c//编译kern/libs/readline.c   生成obj/kern/libs/readline.o
gcc -Ikern/libs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/libs/readline.c -o obj/kern/libs/readline.o

+ cc kern/libs/stdio.c//编译kern/libs/stdio.c   生成obj/kern/libs/stdio.o
gcc -Ikern/libs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/libs/stdio.c -o obj/kern/libs/stdio.o

+ cc kern/debug/kdebug.c//编译 kern/debug/kdebug.c   生成obj/kern/debug/kdebug.o
gcc -Ikern/debug/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/debug/kdebug.c -o obj/kern/debug/kdebug.o

+ cc kern/debug/kmonitor.c//编译kern/debug/kmonitor.c   生成obj/kern/debug/kmonitor.o
gcc -Ikern/debug/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/debug/kmonitor.c -o obj/kern/debug/kmonitor.o

+ cc kern/debug/panic.c//编译kern/debug/panic.c   生成obj/kern/debug/panic.o
gcc -Ikern/debug/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/debug/panic.c -o obj/kern/debug/panic.o

+ cc kern/driver/clock.c//编译kern/driver/clock.c   生成obj/kern/driver/clock.o
gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/clock.c -o obj/kern/driver/clock.o

+ cc kern/driver/console.c//编译kern/driver/console.c   生成obj/kern/driver/console.o
gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/console.c -o obj/kern/driver/console.o

+ cc kern/driver/intr.c//编译kern/driver/intr.c   生成obj/kern/driver/intr.o
gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/intr.c -o obj/kern/driver/intr.o

+ cc kern/driver/picirq.c//编译kern/driver/picirq.c   生成obj/kern/driver/picirq.o
gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/picirq.c -o obj/kern/driver/picirq.o

+ cc kern/trap/trap.c//编译kern/trap/trap.c   生成obj/kern/trap/trap.o
gcc -Ikern/trap/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/trap/trap.c -o obj/kern/trap/trap.o
</code></pre>

 .S 文件是经过预编译的汇编语言源代码文件

<pre><code>
+ cc kern/trap/trapentry.S//编译kern/trap/trapentry.S   生成obj/kern/trap/trapentry.o
gcc -Ikern/trap/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/trap/trapentry.S -o obj/kern/trap/trapentry.o

+ cc kern/trap/vectors.S//编译kern/trap/vectors.S   生成obj/kern/trap/vectors.o
gcc -Ikern/trap/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/trap/vectors.S -o obj/kern/trap/vectors.o

+ cc kern/mm/pmm.c//编译kern/mm/pmm.c   生成obj/kern/mm/pmm.o
gcc -Ikern/mm/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/mm/pmm.c -o obj/kern/mm/pmm.o

+ cc libs/printfmt.c//编译libs/printfmt.c   生成obj/libs/printfmt.o
gcc -Ilibs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/  -c libs/printfmt.c -o obj/libs/printfmt.o

+ cc libs/string.c//编译libs/string.c   生成obj/libs/string.o
gcc -Ilibs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/  -c libs/string.c -o obj/libs/string.o
</code></pre>

ld 命令目标文件转化成可执行程序
- O2 编译器优化级别第二级

<pre><code>
+ ld bin/kernel//生成kernel执行程序
ld -m    elf_i386 -nostdlib -T tools/kernel.ld -o bin/kernel  obj/kern/init/init.o obj/kern/libs/readline.o obj/kern/libs/stdio.o obj/kern/debug/kdebug.o obj/kern/debug/kmonitor.o obj/kern/debug/panic.o obj/kern/driver/clock.o obj/kern/driver/console.o obj/kern/driver/intr.o obj/kern/driver/picirq.o obj/kern/trap/trap.o obj/kern/trap/trapentry.o obj/kern/trap/vectors.o obj/kern/mm/pmm.o  obj/libs/printfmt.o obj/libs/string.o

+ cc boot/bootasm.S//编译-c boot/bootasm.S   生成obj/boot/bootasm.o
gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootasm.S -o obj/boot/bootasm.o

+ cc boot/bootmain.c//编译boot/bootmain.c   生成obj/boot/bootmain.o
gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootmain.c -o obj/boot/bootmain.o

+ cc tools/sign.c//编译tools/sign.c   生成obj/sign/tools/sign.o
gcc -Itools/ -g -Wall -O2 -c tools/sign.c -o obj/sign/tools/sign.o
gcc -g -Wall -O2 obj/sign/tools/sign.o -o bin/sign

+ ld bin/bootblock//生成kernel执行程序
ld -m    elf_i386 -nostdlib -N -e start -Ttext 0x7C00 obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o
'obj/bootblock.out' size: 472 bytes
</code></pre>

dd 命令将c程序放入一个虚拟硬盘

<pre><code>
//生成一个ucore.img虚拟硬盘
dd if=/dev/zero of=bin/ucore.img count=10000
10000+0 records in
10000+0 records out
5120000 bytes (5.1 MB) copied, 0.0741708 s, 69.0 MB/s

//将bootblock放入ucore.img
dd if=bin/bootblock of=bin/ucore.img conv=notrunc
1+0 records in
1+0 records out
512 bytes (512 B) copied, 0.000117749 s, 4.3 MB/s

//将kernel放入ucore.img
dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc
138+1 records in
138+1 records out
70775 bytes (71 kB) copied, 0.00569453 s, 12.4 MB/s
</code></pre>
