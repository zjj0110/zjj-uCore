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
 set architecture i8086
 target remote :1234
 b *0x7c00
 c
 </code></pre>
 在lab1下执行make debug，开始调试。
 
## 3、从0x7c00开始跟踪代码运行,将单步跟踪反汇编得到的代码与bootasm.S和 bootblock.asm进行比较。
## 4、自己找一个bootloader或内核中的代码位置，设置断点并进行测试。
