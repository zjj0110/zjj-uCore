# 练习5：实现函数调用堆栈跟踪函数 （需要编程）

      完成kdebug.c中函数print_stackframe的实现，可以通过函数print_stackframe来跟踪函数调用堆栈中记录的返回地址。
      执行make qemu 观察输出，并解释最后一行各个数值的含义

## 打开kdebug.c文件，找到有关函数print_stackframe的定义,阅读注释，我们可以得到一些有用的信息。
<pre><code>
/* LAB1 YOUR CODE : STEP 1 */
     /* (1) call read_ebp() to get the value of ebp. the type is (uint32_t);
      * (2) call read_eip() to get the value of eip. the type is (uint32_t);
      * (3) from 0 .. STACKFRAME_DEPTH
      *    (3.1) printf value of ebp, eip
      *    (3.2) (uint32_t)calling arguments [0..4] = the contents in address (uint32_t)ebp +2 [0..4]
      *    (3.3) cprintf("\n");
      *    (3.4) call print_debuginfo(eip-1) to print the C calling function name and line number, etc.
      *    (3.5) popup a calling stackframe
      *           NOTICE: the calling funciton's return addr eip  = ss:[ebp+4]
      *                   the calling funciton's ebp = ss:[ebp]
      */
</pre></code>
一、函数print_stackframe的功能为跟踪函数调用堆栈中记录的返回地址，并打印。

二、调用read_ebp()函数可以得到ebp的值，类型为(uint32_t)，调用read_eip()可以的到eip的值，类型为(uint32_t)，调用print_debuginfo(),可以打印函数调用的信息。

三、函数实现步骤：循环打印ebp的值，eip的值，函数参数，函数信息，执行出栈操作。

实现代码：
<pre><code>
void print_stackframe(void) {
 //定义ebp,eip接受函数返回
 uint32_t *ebp = (uint32_t *)read_ebp();
 uint32_t eip = read_eip();
 //循环执行
 while (ebp)
 {
     cprintf("ebp:0x%08x eip:0x%08x args:", (uint32_t)ebp, eip);
     cprintf("0x%08x 0x%08x 0x%08x 0x%08x\n", ebp[2], ebp[3], ebp[4], ebp[5]);
     print_debuginfo(eip - 1);
     eip = ebp[1];
     ebp = (uint32_t *)*ebp;
 }
</pre></code>
执行make qemu。

## 分析最后一行数值定义
<pre><code>
ebp:0x00007bf8 eip:0x00007d68 args:0xc031fcfa 0xc08ed88e 0x64e4d08e 0xfa7502a8
</pre></code>
ebp:0x00007bf8,ebp的值是kern_init函数的栈顶地址

eip:0x00007d6e,eip的值是kern_init函数的返回地址

args:0xc031fcfa 0xc08ed88e 0x64e4d08e 0xfa7502a8 一般来说，args存放的4个dword是对应4个输入参数的值。但这里比较特殊，由于bootmain函数调用kern_init并没传递任何输入参数，并且栈顶的位置恰好在boot loader第一条指令存放的地址的上面，而args恰好是kern_int的ebp寄存器指向的栈顶往上第2~5个单元，因此args存放的就是bootloader指令的前16个字节。
