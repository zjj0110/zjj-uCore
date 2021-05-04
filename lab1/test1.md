# 理解通过make生成执行文件的过程。（要求在报告中写出对下述问题的回答）

在此练习中，大家需要通过静态分析代码来了解：

	1. 操作系统镜像文件ucore.img是如何一步一步生成的？(需要比较详细地解释Makefile中每一条相关命令和命令参数的含义，以及说明命令导致的结果)
	2. 一个被系统认为是符合规范的硬盘主引导扇区的特征是什么？



## 1.操作系统镜像文件ucore.img是如何一步一步生成的？


第一步、进入Makefile文件所在位置执行make -v命令，显示make执行了那些命令。(已精简)


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


ld 命令将目标文件与库链接为可执行文件或库文件。

ld [OPTIONS] OBJFILES

-m <emulation>       模拟指定的链接器
   
-T <scriptfile>      使用 scriptfile 作为链接器脚本
   
-o <output>          指定输出文件的名称 
   
-N                   指定读取/写入文本和数据段

-e <entry>        	使用指定的符号作为程序的初始执行点  
   
-Ttext=<org>         使用指定的地址作为文本段的起始点 
 
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


dd 命令用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换。

if=文件名：输入文件名

of=文件名：输出文件名 

count=blocks：拷贝blocks个块

conv=notrunc：用不截短输出方式转换文件。

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

第二步、其次阅读Makefile代码分析make生成ucore.img的全过程

<pre><code>
# create ucore.img
UCOREIMG	:= $(call totarget,ucore.img)

$(UCOREIMG): $(kernel) $(bootblock)
	$(V)dd if=/dev/zero of=$@ count=10000
	$(V)dd if=$(bootblock) of=$@ conv=notrunc
	$(V)dd if=$(kernel) of=$@ seek=1 conv=notrunc

$(call create_target,ucore.img)
</code></pre>
生成ucore.img代码，需要生成kernel和bootblock文件

生成kernel
<pre><code>
# create kernel target
kernel = $(call totarget,kernel)

$(kernel): tools/kernel.ld

$(kernel): $(KOBJS)
	@echo + ld $@
	$(V)$(LD) $(LDFLAGS) -T tools/kernel.ld -o $@ $(KOBJS)
	@$(OBJDUMP) -S $@ > $(call asmfile,kernel)
	@$(OBJDUMP) -t $@ | $(SED) '1,/SYMBOL TABLE/d; s/ .* / /; /^$$/d' > $(call symfile,kernel)

$(call create_target,kernel)
</code></pre>

生成bootblock
<pre><code>
# create bootblock
bootfiles = $(call listf_cc,boot)
$(foreach f,$(bootfiles),$(call cc_compile,$(f),$(CC),$(CFLAGS) -Os -nostdinc))

bootblock = $(call totarget,bootblock)

$(bootblock): $(call toobj,$(bootfiles)) | $(call totarget,sign)
	@echo + ld $@
	$(V)$(LD) $(LDFLAGS) -N -e start -Ttext 0x7C00 $^ -o $(call toobj,bootblock)
	@$(OBJDUMP) -S $(call objfile,bootblock) > $(call asmfile,bootblock)
	@$(OBJCOPY) -S -O binary $(call objfile,bootblock) $(call outfile,bootblock)
	@$(call totarget,sign) $(call outfile,bootblock) $(bootblock)

$(call create_target,bootblock)
</code></pre>

//生成sign
<pre><code>
$(call add_files_host,tools/sign.c,sign,sign)
$(call create_target_host,sign,sign)
</code></pre>

因时间、精力、能力有限，Makefile文件中代码未能完全理解

总结大致执行过程：

1、编译libs和kern目录下所有的.c和.S文件，生成.o文件，并链接得到bin/kernel文件；

2、编译boot目录下所有的.c和.S文件，生成.o文件，并链接得到bin/bootblock.out文件；

3、编译tools/sign.c文件，得到bin/sign文件；

4、利用bin/sign工具将bin/bootblock.out文件转化为512字节的bin/bootblock文件，并将bin/bootblock的最后两个字节设置为0x55AA；

5、为bin/ucore.img分配内存空间，并将bin/bootblock复制到bin/ucore.img的第一个block，紧接着将bin/kernel复制到bin/ucore.img第二个block开始的位置。

## 2.一个被系统认为是符合规范的硬盘主引导扇区的特征是什么？

根据sign.c 可知，一个符合规范的引导扇区应当不大于512字节，且最后两个位一定是0x55和0xAA

sign.c的部分代码
<pre><code>
 printf("'%s' size: %lld bytes\n", argv[1], (long long)st.st_size);
    if (st.st_size > 510) {
        fprintf(stderr, "%lld >> 510!!\n", (long long)st.st_size);
        return -1;
    }
    char buf[512];
    memset(buf, 0, sizeof(buf));
    FILE *ifp = fopen(argv[1], "rb");
    int size = fread(buf, 1, st.st_size, ifp);
    if (size != st.st_size) {
        fprintf(stderr, "read '%s' error, size is %d.\n", argv[1], size);
        return -1;
    }
    fclose(ifp);
    buf[510] = 0x55;
    buf[511] = 0xAA;
    FILE *ofp = fopen(argv[2], "wb+");
size = fwrite(buf, 1, 512, ofp);
</code></pre>
