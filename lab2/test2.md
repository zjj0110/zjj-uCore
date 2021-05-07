# 练习2：实现寻找虚拟地址对应的页表项（需要编程）

    通过设置页表和对应的页表项，可建立虚拟内存地址和物理内存地址的对应关系。其中的get_pte函数是设置页表项环节中的一个重要步骤。此函数找到一个虚地址对应的二级页表项的内核虚地址，如果此二级页表项不存在，则分配一个包含此项的二级页表。本练习需要补全get_pte函数 in kern/mm/pmm.c，实现其功能。
    请在实验报告中简要说明你的设计实现过程。请回答如下问题：
    请描述页目录项（Pag Director Entry）和页表（Page Table Entry）中每个组成部分的含义和以及对ucore而言的潜在用处。
    如果ucore执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？

## 阅读pmm.c中get_pte()函数注释，得到一些有用的宏和函数定义:

已知线性地址32位，分为三部分directory(前10位),Table(第二个10位),Offset(最后12位)。

PDX(la):获得虚拟地址la的页目录项索引。

PTX(la)：获得虚拟地址la的页表偏移

KADDR(pa)：根据物理地址并返回相应的内核虚拟地址。

set_page_ref(page,1)：表示该页被一次引用。

page2pa(page):获取这个页管理的内存的物理地址。

alloc_page():分配一个页。

memset(void *s, char c, size_t n)：将s指向的内存区域的前n个字节设置为指定的值c。

PTE_ADDR(pte)：返回页表项地址

PDE_ADDR(pde)：返回页目录项地址

PTE_P           0x001     // 页 表/目录 项标志位：存在

PTE_W           0x002     // 页 表/目录 项标志位：可写

PTE_U           0x004     // 页 表/目录 项标志位：用户可访问

## 实现：
<pre><code>
pte_t *
get_pte(pde_t *pgdir, uintptr_t la, bool create) {
 
    pde_t *pdep = &pgdir[PDX(la)];
    if (!(*pdep & PTE_P)) {
        struct Page *page;
        if (!create || (page = alloc_page()) == NULL) {
            return NULL;
        }
        set_page_ref(page, 1);
        uintptr_t pa = page2pa(page);
        memset(KADDR(pa), 0, PGSIZE);
        *pdep = pa | PTE_U | PTE_W | PTE_P;
    }
return &((pte_t *)KADDR(PDE_ADDR(*pdep)))[PTX(la)];
}
</pre></code>
首先尝试使用PDX函数，获取一级页表索引，通过查表找到页目录项。

如果找到，直接返回二级页表项地址；

如果找不到，则创建一个二级页表，若创建失败则返回NULL，创建成功后，修改该页引用，并将该页表的前第0字节全设为0，接下来设置控制位，设置PTE_U、PTE_W和PTE_P，分别代表用户态的软件可以读取对应地址的物理内存页内容、物理内存页内容可写、物理内存页存在，最后返回二级页表项地址。

## 回答问题：
1、请描述页目录项（Pag Director Entry）和页表（Page Table Entry）中每个组成部分的含义和以及对ucore而言的潜在用处。

页目录项存储某张页表的一些信息，与页表项中相同。

页表项存储某页的一些信息，每个页表项的高20位, 就是该页表项指向的物理页面的首地址的高20位, 而每个页表项的低12位, 是一些功能位，PTE_P(Present)，PTE_W(Writeable)，PTE_U(User)，PTE_PWT(Write-Through)，PTE_PCD(Cache-Disable)，PTE_A(Accessed)，PTE_D(Dirty)，PTE_PS(Page Size)，PTE_MBZ(Bits must be zero)，PTE_AVAIL(Available for software use)三位。

2、如果ucore执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？

进行换页操作 首先 CPU 将产生页访问异常的线性地址放到 cr2 寄存器中；然后就是和普通的中断一样，保护现场，将寄存器的值压入栈中；然后压入 error_code 中断服务例程将外存的数据换到内存中来；最后，退出中断回到进入中断前的状态。
