# 练习3：分析bootloader进入保护模式的过程。（要求在报告中写出分析）

    BIOS将通过读取硬盘主引导扇区到内存，并转跳到对应内存中的位置执行bootloader。请分析bootloader是如何完成从实模式进入保护模式的。

第一步：清理环境。
<pre><code>
         .code16
	    cli
	    cld
	    xorw %ax, %ax
	    movw %ax, %ds
	    movw %ax, %es
	    movw %ax, %ss       
</pre></code>

第二步：开启A20，因为A20的初始值是0，地址线控制是被屏蔽的，而在保护模式下 A20 地址线控制是要打开的，所以需要通过将键盘控制器上的A20线置于高电位，使得全部32条地址线可用。
<pre><code>
	seta20.1:               # 等待8042键盘控制器不忙
	    inb $0x64, %al      # 
	    testb $0x2, %al     #
	    jnz seta20.1        #

	    movb $0xd1, %al     # 发送写8042输出端口的指令
	    outb %al, $0x64     #
	    
	seta20.1:               # 等待8042键盘控制器不忙
	    inb $0x64, %al      # 
	    testb $0x2, %al     #
	    jnz seta20.1        #

	    movb $0xdf, %al     # 打开A20
	    outb %al, $0x60     # 
</pre></code>

第三步：初始化GDT表，段寄存器只有16bit，保护模式需要64bit的段描述符，GDP存储此映射。
<pre><code>
	   lgdt gdtdesc
</pre></code>
第四步: 进入保护模式：通过将cr0寄存器PE位置1便开启了保护模式。
<pre><code>
            movl %cr0, %eax
	    orl $CR0_PE_ON, %eax
	    movl %eax, %cr0
</pre></code>
第五步：通过长跳转更新cs，设置段寄存器，建立堆栈，转到保护模式完成，进入boot主方法。
<pre><code>
    # Jump to next instruction, but in 32-bit code segment.
    # Switches processor into 32-bit mode.
    ljmp $PROT_MODE_CSEG, $protcseg

.code32                                             # Assemble for 32-bit mode
protcseg:
    # Set up the protected-mode data segment registers
    movw $PROT_MODE_DSEG, %ax                       # Our data segment selector
    movw %ax, %ds                                   # -> DS: Data Segment
    movw %ax, %es                                   # -> ES: Extra Segment
    movw %ax, %fs                                   # -> FS
    movw %ax, %gs                                   # -> GS
    movw %ax, %ss                                   # -> SS: Stack Segment

    # Set up the stack pointer and call into C. The stack region is from 0--start(0x7c00)
    movl $0x0, %ebp
    movl $start, %esp
    call bootmain
	
</pre></code>
