# 练习3：释放某虚地址所在的页并取消对应二级页表项的映射（需要编程）
    当释放一个包含某虚地址的物理内存页时，需要让对应此物理内存页的管理数据结构Page做相关的清除处理，使得此物理内存页成为空闲；另外还需把表示虚地址与物理地址对应关系的二级页表项清除。请仔细查看和理解page_remove_pte函数中的注释。为此，需要补全在 kern/mm/pmm.c中的page_remove_pte函数。
    
    请在实验报告中简要说明你的设计实现过程。请回答如下问题：
    数据结构Page的全局变量（其实是一个数组）的每一项与页表中的页目录项和页表项有无对应关系？如果有，其对应关系是啥？
    如果希望虚拟地址与物理地址相等，则需要如何修改lab2，完成此事？ 鼓励通过编程来具体完成这个问题

## 打开文件kern/mm/pmm.c阅读有关page_remove_pte函数的注释
获得一些有用的宏及函数定义：

struct Page *page pte2page(*ptep)：从ptep的值中获取相应的页，ptep为页表项指针

free_page：释放一个页

page_ref_dec(page)：减少页的引用数

tlb_invalidate(pde_t *pgdir, uintptr_t la)：使tlb条目无效，但仅当正在编辑的页表是处理器当前正在使用的页表时。

PTE_P           0x001                   //页 表/目录 项存在标志位

## 代码实现：
<pre><code>
static inline void
page_remove_pte(pde_t *pgdir, uintptr_t la, pte_t *ptep) {
   	if (*ptep & PTE_P) {   //PTE_P代表页存在
        struct Page *page = pte2page(*ptep);
        if (page_ref_dec(page) == 0) {
            free_page(page);
        }
        *ptep = 0;
        tlb_invalidate(pgdir, la)
    }
}
</pre></code>

首先，判断虚拟地址对应的物理页是否存在，若存在，则获取该页地址，然后减少该页引用，若引用减为0，则释放该页，最后清理页表项，并使tlb表中该虚拟地址条目无效。
