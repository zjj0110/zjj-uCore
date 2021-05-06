# 练习1：实现 first-fit 连续物理内存分配算法（需要编程）

    在实现first fit 内存分配算法的回收函数时，要考虑地址连续的空闲块之间的合并操作。
    提示:在建立空闲页块链表时，需要按照空闲页块起始地址来排序，形成一个有序的链表。
    可能会修改default_pmm.c中的default_init，default_init_memmap，default_alloc_pages， default_free_pages等相关函数。请仔细查看和理解default_pmm.c中的注释。
    
打开default_pmm.c文件，阅读注释。

按照注释要求阅读库文件list.h,其中包含了双向指针数据结构结构list_entry_t(list_entry),以及list_init(双向指针初始化), list_add(list_add_after)(双向指针尾插), list_add_before(双向指针头插), list_del(删除双向指针), list_next(返回下一个双向指针), list_prev(返回前一个双向指针)函数。

阅读代码，寻找重要数据结构。

物理页数据结构Page (kern/mm/memlayout.h)
<pre><code>
struct Page {
    int ref;               	// page frame's reference counter
    uint32_t flags;          	// array of flags that describe the status of the page frame
    unsigned int property;    	// the num of free block, used in first fit pm manager
    list_entry_t page_link;    	// free list link
};
</pre></code>
ref:物理页帧引用计数；flags:页帧状态描述标志；property：从此页之后连续自由块数；page_link：链表指针；

空闲内存空间块列表结构free_area_t (kern/mm/mmlayout.h)
<pre><code>
typedef struct {
    list_entry_t free_list;         	// the list header
    unsigned int nr_free;         	// # of free pages in this free list
} free_area_t;
</pre></code>
free_link:链表头指针;nr_free:自由页数；

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

default_init原代码：
<pre><code>
free_area_t free_area;//实体化空闲空间块列表结构 free_area

//宏定义
#define free_list (free_area.free_list)
#define nr_free (free_area.nr_free)

static void
default_init(void) {
    list_init(&free_list);//调用list_init()函数初始化free_list
    nr_free = 0;//初始化nr_free
}
</pre></code>
不做更改。

kern_init --> pmm_init-->page_init-->init_memmap--> pmm_manager->init_memmap

内核初始化函数 -(调用)- 物理内存初始化函数 -(调用)- 整体物理地址的初始化函数 -(调用)- 初始化空闲块列表函数 -(调用)- 物理地址管理函数 -(调用)- 初始化空闲块列表函数
default_init_memmap()代码：
<pre><code>
static void
default_init_memmap(struct Page *base, size_t n) {
    assert(n > 0);
    struct Page *p = base;
    for (; p != base + n; p ++) {
        assert(PageReserved(p));
        p->flags = p->property = 0;
        set_page_ref(p, 0);
    }
    base->property = n;
    SetPageProperty(base);
    nr_free += n;
    list_add(&free_list, &(base->page_link));
}
</pre></code>
不做更改。

default_alloc_pages()代码：
<pre><code>
static struct Page *
default_alloc_pages(size_t n) {
    assert(n > 0);
    
    //如果请求页数大于物理页数，返回NULL，表示分配失败
    if (n > nr_free) {
        return NULL;
    }
    
    
    list_entry_t *le, *len;
    le = &free_list;
 
    while((le=list_next(le)) != &free_list) {
    //寻找一个可分配的连续页
      struct Page *p = le2page(le, page_link);
      if(p->property >= n){
        int i;
        for(i=0;i<n;i++){
          len = list_next(le);
          struct Page *pp = le2page(le, page_link);
          SetPageReserved(pp);
          ClearPageProperty(pp);
          list_del(le);//删链表
          le = len;
        }
        if(p->property>n){
          (le2page(le,page_link))->property = p->property - n;
        }
        ClearPageProperty(p);
        SetPageReserved(p);
        nr_free -= n;
        return p;
      }
    }
    return NULL;
}
</pre></code>
其中le2page(le, page_link)可将list_entry_t转换为page类型；
在默认的default_alloc_pages()函数中采用的分配方法是：如果请求页数大于物理页数，返回NULL；否则，扫描free_area_t找到第一个不小于待申请的物理页数目的空闲block，如果空闲block的物理页数目刚好等于待申请的数目，那么直接将该空闲block全部分配给申请者，如果空闲block的物理页数目大于待申请的数目，那么将空闲block拆分成两个block，第一个block的物理页数目等于待申请的数目，第二个block的物理页数目为剩余的数目，
