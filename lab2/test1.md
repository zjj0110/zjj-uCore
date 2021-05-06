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

default_init代码：
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
        p->flags = 0;
        SetPageProperty(p);    //更改页的状态
        p->property = 0;
        set_page_ref(p, 0);
        list_add_before(&free_list, &(p->page_link));
    }
    nr_free += n;
    //first block
    base->property = n;
}
</pre></code>

default_alloc_pages()代码：
<pre><code>
static struct Page *
default_alloc_pages(size_t n) {
    assert(n > 0);
    if (n > nr_free) {
        return NULL;
    }
    list_entry_t *le, *len;
    le = &free_list;
 
    while((le=list_next(le)) != &free_list) {//寻找一个可分配的连续页
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

首先判断空闲页的大小是否大于所需的页块大小，如果请求页数大于物理页数，返回NULL；

否则，扫描free_area_t找到适合的页，使该页p->property >= n，则重新设置标志位，调用SetPageReserved(pp)和ClearPageProperty(pp)，设置当前页面预留，以及清空该页面的连续空闲页面数量值。然后从空闲链表，即free_area_t中，记录空闲页的链表，删除此项。

如果当前空闲页的大小大于所需大小，则分割页块，具体操作就是，刚刚分配了n个页，如果分配完了，还有连续的空间，则在最后分配的那个页的下一个页（未分配），更新它的连续空闲页值。如果正好合适，则不进行操作。

最后计算剩余空闲页个数并返回分配的页块地址。

default_free_pages()代码：
<pre><code>
static void
default_free_pages(struct Page *base, size_t n) {
    assert(n > 0);
    assert(PageReserved(base));
 
    list_entry_t *le = &free_list;
    struct Page * p;
    while((le=list_next(le)) != &free_list) {
      p = le2page(le, page_link);
      if(p>base){
        break;
      }
}		//找到释放的位置
 
    //list_add_before(le, base->page_link);
    for(p=base;p<base+n;p++){
      list_add_before(le, &(p->page_link));
    }		//在这个位置开始，插入释放数量的空页
    base->flags = 0;
    set_page_ref(base, 0);//引用次数
    ClearPageProperty(base);
    SetPageProperty(base);
    base->property = n;
    
    p = le2page(le,page_link) ;		//此时，p已经到达了插入完释放数量空页的后一个页的位置上。此时，一般会满足base+n==p，因此，尝试向后合并空闲页。如果能合并，那么base的连续空闲页加上p的连续空闲页，且p的连续空闲页置为0,；如果之后的页不能合并，那么p的property一直为0，下面的代码不会对它产生影响。
    if( base+n == p ){
      base->property += p->property;
      p->property = 0;
    }
    le = list_prev(&(base->page_link));		//获取基地址页的前一个页，如果为空，那么循环查找之前所有为空，能够合并的页
    p = le2page(le, page_link);
    if(le!=&free_list && p==base-1){
      while(le!=&free_list){
        if(p->property){
          p->property += base->property;
          base->property = 0;
          break;		//不断更新前一个页p的property值，并清除base
        }
        le = list_prev(le);
        p = le2page(le,page_link);
      }
    }
 
    nr_free += n;		//最后的最后，空闲页数量加n
    return ;
}

default_free_pages主要完成的是对于页的释放操作。
首先使用一个assert语句断言这个基地址所在的页是否为预留，如果不是预留页，那么说明它已经是free状态，无法再次free，也就是之前所述，只有处在占用的页，才能有free操作。

之后，声明一个页p，p遍历一遍整个物理空间，直到遍历到base所在位置停止，开始释放操作。

找到了这个基地址之后呢，就可以将空闲页重新加进来（之前在分配的时候，删除了），之后就是一系列与初始化空闲页一样的设置标记位操作了。

之后，如果插入基地址附近的高地址或低地址可以合并，那么需要更新相应的连续空闲页数量，向高合并和向低合并。

</pre></code>
