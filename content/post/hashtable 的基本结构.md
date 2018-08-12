---
title: "hashtable 的基本结构"
date: 2018-08-13
tags: [php5, internal]
draft: false
---
#### 基础概念

c 语言的数组是一块可以通过偏移量访问的内存区，这也就暗含了偏移量必须是连续的整数，例如数组的前三个索引是 0， 1， 2， 那么他的下一个索引是 3， 不能是 214678462。相比之下，php 的数组结构就非常不同：他既支持字符串类型的 key 作为索引， 也支持非连续的整数 key 作为索引， 甚至可以混合使用。

c 语言可以使用两种方式实现这种数组结构（类 php 数组）。一种是二叉搜索树，使用这种实现方式，无论是查找还是插入都是 O(log n) 的时间复杂度(n 是表中的元素个数) 。第二种是使用 hashtable 实现，这种实现方式的查找和插入的时间复杂度的平均值是 O(1)， 也就是说，元素可以在很短的时间插入和获取。因此在多数情况下，hashtable 更被人喜欢， php 数组的实现也是用的这种技术。

hashtable 的原理是非常简单的：复杂的 key(例如 string) 被 hash 函数转换成整数。这个整数用做 c 数组的索引。 问题是，无论是 2^32 还是 2^64（我们的内存有限） 都要比字符串的数量少（因为字符串有无限种可能性）。因此 hash 函数会有冲突的情况发生，也就是说两个不同的字符串， 会得到相同的 hash 值。

对于 hash 冲突有两个常用的解决办法，一个是 open addressing(在这里我们不讨论)。另一个是链表，这是 php 选择的解决方案，将有相同 hash 值的所有元素放在一个链表中。当我们通过 key 来查询相应的值时，php 会先计算 key 的 hash 值，通过 hash 值找到对应的链表，然后遍历链表，直到发现匹配的元素。下面是对冲突情况的分析：

![](/uploads/Basic_structure_—_PHP_Internals_Book/basic_hashtable.svg)

链表中的元素叫做 bucket， c 数组中包含了链表的 head。c 数组叫 arBuckets。

来看一下删除一个指定的 bucket 的情况: 比如你想删除 "c"。为了实现这样的目的，你需要移动 bucket "a" 的指针指向 NULL。因此你需要首先获得 bucket "a" 的指针， 实现的方式有两种，一种是通过 hash 值遍历链表找到 "a"，还有一种是为每个 bucket 再增加一个相反方向的指针（变成双向链表）。php 采用了第二种实现方式：使每一个 bucket 包含一个指向下一个 bucket 的指针 (pNext），和一个指向前一个 bucket 的指针(pLast)。下面是这种结构的图解：

![](/uploads/Basic_structure_—_PHP_Internals_Book/doubly_linked_hashtable.svg)

进一步说， php 的 hashtable 必须是有序的：如果你插入一个元素，当你遍历数组的时候，你总是可以在他之前插入的位置得到他。为了实现这一点，必须引入另一条链表来维持插入顺序。把前一个 bucket 的指针存放在当前 bucket 的 pListNext 中，后一个 bucket 的指针存放在 当前 bucket 的 pListLast 中。 另外，在 hashtable 中这条链表有一个头指针 pListHead， 有一个尾指针 pListLast。下面是以 "a"， "b"， "c"(顺序 a-c) 为例这条链表的样子:

![](/uploads/Basic_structure_—_PHP_Internals_Book/ordered_hashtable.svg)

#### HashTable 和 bucket 数据结构(structure)

为了实现 hashtable， php 使用了两种数据结构，可以在 zend_hash.h 中找到。我们首先看一下 bucket 的数据结构:

```
typedef struct bucket {
    ulong h;
    uint nKeyLength;
    void *pData;
    void *pDataPtr;
    struct bucket *pListNext;
    struct bucket *pListLast;
    struct bucket *pNext;
    struct bucket *pLast;
    char *arKey;
} Bucket;
```

我们前面已经知道了 pNext， pLast， pListNext 和 pListLast 的用处。现在让我们看一下其他前面没提到的属性:  

h 是 bucket 对应的 hash 值。如果 key 是整数，h 就等于 key，nKeyLength 将会赋值为 0。如果 key 是字符串，h 是对字符串做 zend_hash_func() 的结果， arKey 会指向该字符串， nKeyLength 是字符串的长度。

pData 是指向储存值的指针。存储的值和通过插入函数传入的值不是一个，他是传入值的复制（没有存储在 bucket 中）。当存储的值是一个指针类型的话，pData 直接指向存储值的指针是非常没有效率的（指针指向另一个不在 bucket 中的指针），php 使用一个小技巧来提高效率：与其让 pData 指向 bucket 数据块外的指针，不如将存储值的指针放到 bucket 中（使用 pDataPtr 直接指向数据），然后让 pData 指向 bucket 中的 pDataPtr。

现在让我们来看一下重要的 HashTable 结构:
```
typedef struct _hashtable {
    uint nTableSize;
    uint nTableMask;
    uint nNumOfElements;
    ulong nNextFreeElement;
    Bucket *pInternalPointer;
    Bucket *pListHead;
    Bucket *pListTail;
    Bucket **arBuckets;
    dtor_func_t pDestructor;
    zend_bool persistent;
    unsigned char nApplyCount;
    zend_bool bApplyProtection;
#if ZEND_DEBUG
    int inconsistent;
#endif
} HashTable;
```
你已经知道 arrBuckets 是一个 c 数组，数组的每一个元素对应着一条链表，数组的索引对应着 php 数组键值的 hash。因为 php 的数组没有固定的大小，所以当元素的数量超过 arrBuckets 分配（内存）数量的时候， arBuckets 也必须增加内存分配。我们当然可以在 hashtable 中存储超过 nTableSize 的元素，但是随着插入元素的增多，也将会增大 hash 冲突的可能性，降低数组的性能表现。

nTableSize 总是 2 的幂次方，因此如果你的 hashtable 中有 12 个元素，nTableSize 的值将会是 16。注意虽然 arBuckets 会随着元素的增多自动增长，但是当你移除元素的时候，他并不会减少。如果你在 hashtable 中增加 1000000 个元素，接着再将它们移除，nTableSize 的值仍会是 1048576.

hash 函数的计算结果是 ulong 类型，但是 nTableSize 通常会比这个值小一点。因此 hash 值不能直接用作 arBuckets 的索引。真正采用的索引是 nIndex = h % nTableSize，因为 nTableSize 总是 2 的次方，所以这个表达式等价于 nIndex = h & (nTableSize -1)。为了分析原因，让我们先看 nTableSize - 1：

```
nTableSize     = 128 = 0b00000000.00000000.00000000.10000000
nTableSize - 1 = 127 = 0b00000000.00000000.00000000.01111111
```

nTableSize - 1 使得小于 nTableSize 的 bit 上都是 1 (如上图小于 128 的二进制位都是 1)。因此 h & (nTableSize -  1) 的结果只会保留小于 nTableSize 的部分，这和 h % nTableSize 做的同样的事情。

nTableSize - 1 我们叫做 table mask，他储存在 nTableMask 中。用“二进制与操作”代替“取余”是为了提高性能。

nNextFreeElement 指定下一个 bucket 被插入 arrBuckets 的索引。nNextFreeElement 将会比当前正在使用的 arrBuckets 的最大索引大 1。

你已经知道了 pListHead 和 pListTail 指针（他们是双向链表的头和尾）。 pInternalPointer 用作遍历数组，指向当前的 bucket。

当删除 hashtable 中的一个元素的时候，可以设置一个回调函数，这个回调函数存储在 pDestructor。例如，如果你在向 hashtable 存储 zval * 的元素，在你删除他的时候，zval_ptr_dtor 会被调用。

persistent 表示是否 buckets（和他们的值）应该一直不被回收。在大多数的情况下这个值应该是 0， 因为 hashtable 应该在请求完成之后销毁。  

bApplyProtection 指定 hashtable 是否需要嵌套保护（默认为1）。如果嵌套层数（nApplyCount）达到了某个数值， 嵌套保护将会抛出一个错误。这种保护应用在 hashtable 比对和 zend_hash_apply 函数中。

最后一个属性 inconsistent 用作 debug 编译和在当前 hashtable 状态下存储信息。当 hashtable 被不正确的使用，他被用作抛出错误，例如，你访问一个在销毁过程中的 hashtable 时，他就会派上用场。

[英文原文](http://www.phpinternalsbook.com/hashtables/basic_structure.html)
