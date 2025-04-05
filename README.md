复杂度要求：

插入：随机情况下最坏O($\sqrt{n}$),头尾插入均摊复杂度O(1)

删除：随机情况下最坏O($\sqrt{n}$),头尾删除均摊复杂度O(1)

随机查询的复杂度最坏O($\sqrt{n}$)

——————————————————————————————————————————

通过这个复杂度以及github上的提示，我们很容易想到**分块**的做法，简单来说，就是块状链表的思想，笔者的做法是，将某个能够表示**块**的数据结构储存起来，比起常见的链表或者数组，笔者采用的写法是一个能够表示块的类。简单来说，将每个块状结点用一个结构体储存，记录块的头结点，尾结点，块的前驱，块的后继，块长与块的索引。于是我们采用这样的写法：

```
    public:
        class block_node;
    private:
        block_node* head;
        block_node* tail;
        size_t size_;
    public:
        class list_node {
        public:
            list_node* pre,nxt;
            T* val;
            list_node():pre(nullptr),nxt(nullptr),val(nullptr){}
        };
        class block_node {
        public:
            block_node* pre,nxt;
            list_node* head,tail;
            size_t length ,index;
            block_node():pre(nullptr),nxt(nullptr),head(nullptr),tail(nullptr),length(0),index(0){}
        };
```

其中$\text{block_node}$ 表示块状结点，$\text{list_node}$ 表示普通结点。事实上，本质没有脱离大链表套小链表的窠臼，但这样做避免了类成员的频繁创建与原$\text{double_list}$下各种冗余函数，提高了程序效率，本质上仍然是大链表套给定了初始长度的小链表。

$\text{iterator}$和$\text{const_iterater}$不在笔者考察的范畴之类，因为不涉及具体的复杂度分析，这个交给$\text{CR}:)$

下面笔者来考察具体的函数：

这里的$\text{deque}$采用的头尾节点均为空的双向链表写法，构造函数采用的是深拷贝的写法，重载了等号作为深拷贝赋值.

为行文方便，引出三个常数 $\text{block_size}$块长，$\text{lower_block_size}$需合并的两块之和下限，$\text{upper_block_size}$需分裂的块上限,$\text{n}$为链表总长,上下限都是块长的因子，需要进行手动调参，笔者选取的参数是$1和1/2$

对于$\text{merge}$函数而言，笔者首先连接两块，删除尾节点，更新块的基本属性，删除被合并块并更新后续块索引，显然只有更新索引一步的复杂度可能达到$O(\text{n/block_size})$ (严格意义上是该块的后续块数量)


对于$\text{split}$函数而言,笔者首先创建新块并设置指针，分割链表，创建新尾节点，重新连接链表，更新原块长度并更新后续块索引,显然只有重新连接链表和更新后续块索引两步不是$O(1)$复杂度，前者是通过遍历块内元素进行统计剩余块长，后者是$\text{merge}$操作是同理的，复杂度基本可以达到$O(\text{n/block_size}+\text{block_size})$


对于查询操作而言，采取朴素的分块思想，大块跳跃小块遍历，$\text{n/block_size+block_size}$的复杂度（事实上笔者觉得这个操作是可以优化的，比如开一个块长的前缀和和后缀和数组，利用二分查找这样就可以做到$O(\log n)$的单次复杂度，但是懒得写（））

下面考察具体函数的复杂度.上面的函数操作都是需要尽可能少调用的，获取从直觉上讲，及时的分裂与合并与动态调整块长操作会优化，但是这事实上会降低程序效率,按照隔壁几个去年写块状链表的说法，事实上只有分裂操作是必需的，为此，在不知道数据范围的前提下，我们不妨采取动态调整块长的策略，笔者在插入的三个函数里采取了以下写法:

```
//const int tmp = sqrt(size_) + 1;
//if (block_size + step < tmp) block_size = tmp;
```
$\text{step}$用于调整块长更新的频率，这样基本可以保证在$n$比较小时基本是固定块长，而$n$较大时可以做到$\sqrt{n}$每块，固然有大量空间上的冗余浪费。

下面分析每个具体函数的复杂度:

对于头部插入函数$\text{push_front}$：在第一个块的头部插入新节点，如果块大小超过上限（$\text{upper_block_size}$），则进行分裂，后者操作每$\text{block_size}$才进行一次，所以均摊复杂度是$O(1)$

对于头部删除函数$\text{pop_front}$：如果块为空则删除整个块，如果块与下一个块可以合并则合并，后者操作每$\text{block_size}$才进行一次，所以均摊复杂度是$O(1)$

尾部的插入与删除同理。

对于随机插入与删除，其分块导致的正常复杂度都是$O(\text{n/block_size+block_size})$，由于动态调整块长，所以基本在$O(\sqrt{n})$附近，而由于动态调整导致的合并与分裂由于均摊，平均复杂度仍然是$O(1)$，所以满足要求。

