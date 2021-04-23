# 练习2：使用qemu执行并调试lab1中的软件。（要求在报告中简要写出练习过程）
为了熟悉使用qemu和gdb进行的调试工作，我们进行如下的小练习：

    1、从CPU加电后执行的第一条指令开始，单步跟踪BIOS的执行。
    2、在初始化位置0x7c00设置实地址断点,测试断点正常。
    3、从0x7c00开始跟踪代码运行,将单步跟踪反汇编得到的代码与bootasm.S和 bootblock.asm进行比较。
    4、自己找一个bootloader或内核中的代码位置，设置断点并进行测试。
## 1、从CPU加电后执行的第一条指令开始，单步跟踪BIOS的执行。
 修改lab1/tools/gdbinit 内容为：
 <pre><code>
 set architecture i8086
 target remote :1234
  </code></pre>
 在lab1下执行make debug，弹出qemu框和gdb调试框，在dgb调试界面执行si进行单步跟踪。
 
 执行x /2i $pc可查看BIOS代码。
 
## 2、在初始化位置0x7c00设置实地址断点,测试断点正常。
gdb的地址断点：
在gdb命令行中，使用b*地址便可以在指定内存地址设置断点，当qemu中的cpu执行到指定地址时，便会将控制权交给gdb。

 <pre><code>
 set architecture i8086 //设置当前调试的CPU是8086
 target remote :1234    //配置qemu和gdb通讯网络端口：1234。
 b *0x7c00              //在0x7c00处设置断点。
 c                      //continue简称，表示继续执行
 x/2i $pc               //显示当前eip处的汇编指令
 </code></pre>
 在lab1下执行make debug，开始调试。
 
## 3、从0x7c00开始跟踪代码运行,将单步跟踪反汇编得到的代码与bootasm.S和 bootblock.asm进行比较。

通过改写Makefile文件为：
 <pre><code>
	debug: $(UCOREIMG)
		$(V)$(TERMINAL) -e "$(QEMU) -S -s -d in_asm -D $(BINDIR)/q.log -parallel stdio -hda $< -serial null"
		$(V)sleep 2
		$(V)$(TERMINAL) -e "gdb -q -tui -x tools/gdbinit"
 </code></pre>
在调用qemu时增加-d in_asm -D q.log参数，便可以将运行的汇编指令保存在q.log中。

执行make debug,在bin文件里会生成一个q.log文件，打开文件。

部分代码
 <pre><code>
----------------
IN: 
0x00007c00:  cli    
0x00007c01:  cld    
0x00007c02:  xor    %ax,%ax
0x00007c04:  mov    %ax,%ds
0x00007c06:  mov    %ax,%es
0x00007c08:  mov    %ax,%ss

----------------
IN: 
0x00007c0a:  in     $0x64,%al

----------------
IN: 
0x00007c0c:  test   $0x2,%al
0x00007c0e:  jne    0x7c0a

----------------
IN: 
0x00007c10:  mov    $0xd1,%al
0x00007c12:  out    %al,$0x64
0x00007c14:  in     $0x64,%al
0x00007c16:  test   $0x2,%al
0x00007c18:  jne    0x7c14

----------------
IN: 
0x00007c1a:  mov    $0xdf,%al
0x00007c1c:  out    %al,$0x60
0x00007c1e:  lgdtw  0x7c6c
0x00007c23:  mov    %cr0,%eax
0x00007c26:  or     $0x1,%eax
0x00007c2a:  mov    %eax,%cr0

----------------
IN: 
0x00007c2d:  ljmp   $0x8,$0x7c32

----------------
IN: 
0x00007c32:  mov    $0x10,%ax
0x00007c36:  mov    %eax,%ds

----------------
IN: 
0x00007c38:  mov    %eax,%es
 </code></pre>
将其与bootasm.S和bootblock.asm进行比较，可以发现，反汇编的代码与bootblock.asm基本相同，而与bootasm.S有所差别：

1.反汇编的代码中的指令不带指示长度的后缀，而bootasm.S的指令则有。

2.反汇编的代码中的通用寄存器是32位（带有e前缀），而bootasm.S的代码中的通用寄存器是16位（不带e前缀）。

## 4、自己找一个bootloader或内核中的代码位置，设置断点并进行测试。
 修改lab1/tools/gdbinit 内容为：
 <pre><code>
 set architecture i8086 
 target remote :1234    
 b *0x7caa      //设置断点位置      
 c              
 x/2i $pc
  </code></pre>
  
 在lab1下执行make debug。
 
