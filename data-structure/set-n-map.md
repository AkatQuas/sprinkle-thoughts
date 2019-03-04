# Overview

- `set`是一个排序后到容器，不允许重复。
- `map`是用来存储排序后的键值对的集合，map中键是保持逻辑排序后的顺序。
- `set`和`map`都保证了基本操作（插入、删除、查找等）的对数时间开销。
- `set`和`map`底层实现都是平衡二叉查找树，常常使用自顶向下的[红黑树](https://en.wikipedia.org/wiki/Red-black_tree)。
- `set`和`map`实现的一个重要问题是需要提供对迭代器类的支持，如何高效的将迭代器推进至下一个结点。

# Details

* [Set](#set)
* [Map](#map)

# SET

## 集合的运算

- 并集
- 交集
- 差集

## 计算法则

- 并集的交换律：S ∪ T = T ∪ S
- 并集的结合律：S ∪ (T ∪ R) = (S ∪ T) ∪ R
- 交集的交换律：S ∩ T = T ∩ S
- 交集的结合律：S ∩ (T ∩ R) = (S ∩ T) ∩ R
- 交集对并集的分配律：S ∩ (T ∪ R) = (S ∩ T) ∪ (S ∩ R)
- 并集对交集的分配律：S ∪ (T ∩ R) = (S ∪ T) ∩ (S ∪ R)  
- 并集和差集的结合律：S - (T ∪ R) = (S - T) - R
- 差集对并集的分配律：(S ∪ T) - R = (S - R) ∪ (T - R)
- 并集的幂等律: S ∪ S = S
- 交集的幂等律: S ∩ S = S
- 对空集：
    - S - S = Ø
    - Ø - S = Ø
    - Ø ∩ S = Ø

## 子集 与 真子集

## 幂集：
- 定义：对于任一集合S，其幂集就是指由S的所有子集组成的集合。
- 大小：如果S有`n`个成员，那么其幂集则有2<sup>n</sup>个成员。

### 集合的链表实现

如果集合有n个元素，插入、删除、查找点期望运行时间为O(n)。

特别要说明的是，为表排序可以显著改善并集、交集和差集运算的运行时间。



```C
/*
# assemble函数，先创建一个新单元，然后调用setUnion求L和M的并集，最后返回对应的新单元。

# setUnion函数会从给定的已排序的表中选出最小的元素，并将选定的元素与两个表其余部分一起传给assemble函数。对setUnion来说有6种情况，取决于两个表中有没有NULL，都没有时，看看那个表的表头元素先于另一个。

# 1）两个表都为NULL，setUnion返回NULL，递归结束。
# 2）L为NULL，M不为NULL，从M中取第一个元素，后面跟上NULL表及M的后续部分，调用assemble。
# 3）M为NULL，L不为NULL，与2）相反的工作，用L点第一个元素及其尾部调用assemble。
# 4）L和M的第一个元素相同，则创建一个副本，L->element，加上L的尾部和M的尾部调用assemble。
# 5）如果L的元素先于M，取L的元素，L的尾部，整个M调用assemble。
# 6）如果M的元素先于L，取M的元素，M的尾部，整个L调用assemble。
*/
DefCell(int, CELL, LIST);
LIST setUnion(LIST L, LIST M);
LIST assemble(int x, LIST L, LIST M);

LIST assemble(int x, LIST L, LIST M)
{
    LIST first;
    first = (LIST) malloc(sizeof(struct CELL));
    first->element = x;
    first->next = setUnion(L, M);
    return first;
}

LIST setUnion(LIST L, LIST M)
{
    if (L == NULL && M == NULL)
        return NULL;
    else if (L == NULL)
        return assemble(M->element, NULL, M->next);
    else if (M == NULL)
        return assemble(L->element, L->next, NULL);
    else if (L->element == M->element)
        return assemble(L->element, L->next, M->next);
    else if (L->element < M->element)
        return assemble(L->element, L-next, M);
    else 
        return assemble(M->element, L, M->next);
}
```

```C
/*
# assemble函数，先创建一个新单元，然后调用intersection求L和M的交集，最后返回对应的新单元。

# intersection函数会从给定的已排序的表中选出最小的元素，并将选定的元素与两个表其余部分一起传给assemble函数。对intersection来说有4种情况，取决于两个表中有没有NULL，都没有时，看看那个表的表头元素先于另一个。

# 1）**两个表都为NULL** OR **L为NULL，M不为NULL** OR **M为NULL，L不为NULL**，表示一个或两个表为空，剩余不再有交集，返回NULL。
# 2）L和M的第一个元素相同，则创建一个副本，L->element，加上L的尾部和M的尾部调用assemble。
# 3）如果L的元素先于M，扔掉L的较小元素，将L的尾部，整个M调用intersection。
# 4）如果M的元素先于L，扔掉M的较小元素，将M的尾部，整个L调用intersection。
*/
DefCell(int, CELL, LIST);
LIST setUnion(LIST L, LIST M);
LIST assemble(int x, LIST L, LIST M);

LIST assemble(int x, LIST L, LIST M)
{
    LIST first;
    first = (LIST) malloc(sizeof(struct CELL));
    first->element = x;
    first->next = intersection(L, M);
    return first;
}

LIST intersection(LIST L, LIST M)
{
    if (L == NULL || M == NULL)
        return NULL;
    else if (L->element == M->element)
        return assemble(L->element, L->next, M->next);
    else if (L->element < M->element)
        return intersection(L-next, M);
    else 
        return intersection(L, M->next);
}
```

```C
/*
# relative complement 
*/
LIST assemble(int x, LIST L, LIST M)
{
    LIST first;
    first = (LIST) malloc(sizeof(struct CELL));
    first->element = x;
    first->next = relativeComplement(L, M);
    return first;
}

LIST relativeComplement(LIST L, LIST M)
{
    if (L == NULL)
        return NULL;
    else if (M == NULL)
        return L;
    else if (L->element == M->element)
        return relativeComplement(L->next, M->next);
    else if (L->element < M->element)
        return assemble(L->element, L->next, M);
    else 
        return relativeComplement(L, M->next);
}
```

### 集合的特征向量实现

很多时候，要处理的集合是称为“全集”的某个小集合U的各子集。可以通过某种方式为U中的元素排定次序，那么U中的每个元素都可以与一个唯一的“位置”相关联，位置是从0到n-1的整数，其中n是U中元素的个数。
接着，给定一个包含于U的集合S，就可以用由0和1组成的*特征向量*来表示，规则是，对于U中的每个元素x，如果x在S中，对应x位置上就是1，否则就是0。

要表示某n元素全集各子集的特征向量，可以使用如下类型的布尔数组：
```C
typedef BOOLEAN USET[n]
# 对应位置i的元素插入到S中
void insert(int i)
{
    S[i] = TRUE;
}

# 对应位置i的元素不在S中
void clear(int i)
{
    S[i] = FALSE;
}

# 查找位置i的元素是否在S中
void search(int i)
{
    return S[i];
}
```

此时，插入、删除、查找的时间均为O(1)，缺点是数组可能会很大，而且初始化数组的开销也很大。

```C
# 交集
while ( 0 <= i && i <= n) {
    R[i] = S[i] || T[i];
    i++;
}
# 并集
while ( 0 <= i && i <= n) {
    R[i] = S[i] && T[i];
    i++;
}
# 差集 (S - T)
while ( 0 <= i && i <= n) {
    R[i] = S[i] || !T[i];
    i++;
}
```


# MAP 

