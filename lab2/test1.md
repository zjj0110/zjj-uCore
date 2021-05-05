# 练习1：实现 first-fit 连续物理内存分配算法（需要编程）

    在实现first fit 内存分配算法的回收函数时，要考虑地址连续的空闲块之间的合并操作。
    提示:在建立空闲页块链表时，需要按照空闲页块起始地址来排序，形成一个有序的链表。
    可能会修改default_pmm.c中的default_init，default_init_memmap，default_alloc_pages， default_free_pages等相关函数。请仔细查看和理解default_pmm.c中的注释。
    
打开default_pmm.c文件，阅读注释。

阅读头文件代码，寻找重要数据结构。

物理页数据结构Page (kern/mm/memlayout.h)
<pre><code>
struct Page {
    int ref;               	// page frame's reference counter
    uint32_t flags;          	// array of flags that describe the status of the page frame
    unsigned int property;    	// the num of free block, used in first fit pm manager
    list_entry_t page_link;    	// free list link
};
</pre></code>
ref:物理页帧引用计数；flags:页帧状态描述标志；property：自由块数；page_link：链表指针；

空闲内存空间块数据结构free_area_t (kern/mm/mmlayout.h)
<pre><code>
typedef struct {
    list_entry_t free_list;         	// the list header
    unsigned int nr_free;         	// # of free pages in this free list
} free_area_t;
</pre></code>
free_link:链表指针;nr_free:自由页数；

物理内存管理类pmm_managerd 对象default_pmm_manager(kern/mm/pmm.h)
<pre><code>
struct pmm_manager {
    const char *name;   	// XXX_pmm_manager's name
    void (*init)(void);    	// initialize internal description&management data structure
                      	    // (free block list, number of free block) of XXX_pmm_manager 
    void (*init_memmap)(struct Page *base, size_t n); 
        // setup description&management data structcure according to the initial free physical memory space 
    struct Page *(*alloc_pages)(size_t n);            
        // allocate >=n pages, depend on the allocation algorithm 
    void (*free_pages)(struct Page *base, size_t n);  
        // free >=n pages with "base" addr of Page descriptor structures(memlayout.h)
    size_t (*nr_free_pages)(void);      // return the number of free pages 
    void (*check)(void);               	// check the correctness of XXX_pmm_manager 
};
const struct pmm_manager default_pmm_manager = {
    .name = "default_pmm_manager",
    .init = default_init,
    .init_memmap = default_init_memmap,
    .alloc_pages = default_alloc_pages,
    .free_pages = default_free_pages,
    .nr_free_pages = default_nr_free_pages,
    .check = default_check,
};
</pre></code>
<pre><code></pre></code>
对象名称：default_pmm_manager;方法函数：default_init(初始化空闲块),default_init_memmap(初始化空闲块列表),default_alloc_pages(分配空闲页),default_free_pages(),default_nr_free_pages(释放页),default_check(验证函数);

实现First fit算法的相关函数：default_init，default_init_memmap，default_alloc_pages， default_free_pages。
