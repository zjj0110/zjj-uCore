# 练习2：使用qemu执行并调试lab1中的软件。（要求在报告中简要写出练习过程）
为了熟悉使用qemu和gdb进行的调试工作，我们进行如下的小练习：

    1、从CPU加电后执行的第一条指令开始，单步跟踪BIOS的执行。
    2、在初始化位置0x7c00设置实地址断点,测试断点正常。
    3、从0x7c00开始跟踪代码运行,将单步跟踪反汇编得到的代码与bootasm.S和 bootblock.asm进行比较。
    4、自己找一个bootloader或内核中的代码位置，设置断点并进行测试。
## 1、从CPU加电后执行的第一条指令开始，单步跟踪BIOS的执行。
 首先，查看 labcodes_answer/lab1_result/tools/lab1init 文件
<code>
file bin/kernel
    
target remote :1234

set architecture i8086

b *0x7c00

continue

x /2i $pc 
</code>
## 2、在初始化位置0x7c00设置实地址断点,测试断点正常。
## 3、从0x7c00开始跟踪代码运行,将单步跟踪反汇编得到的代码与bootasm.S和 bootblock.asm进行比较。
## 4、自己找一个bootloader或内核中的代码位置，设置断点并进行测试。
