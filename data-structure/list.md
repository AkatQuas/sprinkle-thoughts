# 表
(a<sub>1</sub>, a<sub>2</sub>,...,a<sub>n</sub>)

### 长度为1的表，
(a)

### 子表 
如果有表L = (a<sub>1</sub>, a<sub>2</sub>,...,a<sub>n</sub>)， 对满足1<=i<=j<=n的i和j来说，(a<sub>i</sub>, a<sub>i+1</sub>,...,a<sub>j</sub>)是L的子表。也就是说，子表是由从某个位置i开始到某个位置j结束的所有元素组成的。还可以说空表e是任意表的子表。

### 子序列
表 L = (a<sub>1</sub>, a<sub>2</sub>,...,a<sub>n</sub>) 的子序列是指从L中剔除0个或多个元素后形成的表。剩下的这些元素，也就是构成子序列的这些元素，必须按照与出现在L中相同的顺序排列，不过子序列的元素在L中不一定是连续的。请注意，空表e和表L本身总是L的子序列，而且L的子表也是L的子序列。

### 前缀与后缀

表前缀是指从表的开头开始的任意子表。表后缀是以表的结尾为末尾的子表。 表e是种特殊情况，它是任意表的前缀和后缀。

### 元素的位置

表中的每个元素都有与之关联的位置。如果有表(a<sub>1</sub>, a<sub>2</sub>,...,a<sub>n</sub>)，而且n>=1，a<sub>1</sub>就是第一个元素，a<sub>2</sub>就是第二个元素，以此类推，而a<sub>n</sub>是最后一个元素。还可以说ai出现在位置i。除此之外，a<sub>i</sub>是 a<sub>i-1</sub>， a<sub>i+1</sub> 前。而存放元素a的位置称作a的出现。

表中位置的数量就等于表的长度。同一元素是有可能出现在两个或多个位置的，因此不要把位置和出现在该位置的元素了弄混了。

### 基本操作有：

- 插入
- 删除
- 串接
- 取表头
- 取位置i的元素
- 取长度
- 询问是否为空

# 链表

```C
typedef struct CELL *LIST;
struct CELL {
    int element;
    LIST next
}
```

### 链表的查找
花费时间O(n), 平均查找次数 (n+1)/2
```C
BOOLEAN lookup(int x, LIST L)
{
    if ( L == NULL )
        return FALSE;
    else if ( x == L->element )
        return TRUE;
    else 
        return lookup(x, L->next);
}
```

### 链表的删除
花费时间O(n), 平均查找次数 (n+1)/2
```C
void delete(int x, LIST *pL)
{
    if ((*pL) != NULL)
        if (x == (*pL)->element)
            (*pL) = (*pL)->next;
        else
            delete(x, &((*pL)->next));
}
```

### 链表的末尾插入
```C
void insertLast(int x, LIST *pL)
{
    if ( (*pL) === NULL ){
        (*pL) = (LIST) malloc(sizeof(struct CELL));
        (*pL)->element = x;
        (*pL)->next = NULL;
    } else if ( x != (*pL)->element )
        insertLast(x, &((*pL)->next));
}
```

### 双向链表

```C
typedef struct CELL *LIST;
struct CELL {
    LIST previous;
    int element;
    LIST next;
}
```

# 栈

- LIFO
- 函数调用的实现
- 基本操作：
    - clear, 清空
    - isEmpty, 是否为空
    - isFull, 是否满了，满了不能再`push`
    - pop, 非空栈时删除并返回栈顶元素
    - push, 将元素压入非满栈

## 栈的数组实现
```C
typedef struct {
    int A[MAX];
    int top;
} STACK;

void clear(STACK *pS)
{
    pS->top = -1;
}

BOOLEAN isEmpty(STACK *pS)
{
    return (pS->top < 0);
}

BOOLEAN isFull(STACK *pS)
{
    return (pS->top >= MAX -1);
}

BOOLEAN pop(STACK *pS, int *px)
{
    if (isEmpty(pS))
        return FALSE;
    else {
        (*px) = pS->A[(pS->top)--];
        return TRUE;
    }
}

BOOLEAN push(int x, STACK *pS)
{
    if (isull(pS))
        return FALSE;
    else {
        pS->A[++(pS->top)] = x;
        return TRUE
    }
}
```

## 栈的链表实现

```C
DefCell (int, CELL, STACK);
typedef struct CELL *STACK;
struct CELL {
    int element;
    STACK next
};

void clear(STACK *pS)
{
    (*pS) = NULL;
}
BOOLEAN isEmpty(STACK *pS)
{
    return ((*pS) == NULL);
}
BOOLEAN isFull(STACK *pS)
{
    return FALSE;
}
BOOLEAN pop(STACK *pS, int *px)
{
    if (isEmpty(pS))
        return FALSE;
    else {
        (*px) = (*pS)->element;
        (*pS) = (*pS)->next;
        return TRUE;
    }
}
BOOLEAN push(int x, STACK *pS)
{
    STACK newCell;
    newCell = (STACK) malloc(sizeof(struct CELL));
    newCell->element = x;
    newCell->next = (*pS);
    (*pS) = newCell;
    return TRUE;
}
```

# 队列
- FIFO
- 基本操作
    - clear, 清空
    - dequeue, 出队 
    - enqueue, 入队
    - isEmpty, 是否为空
    - isFull, 是否已满

```C
DefCell(int, CELL, LIST);
typedef struct {
    LIST front, rear;
} QUEUE;
void clear(QUEUE *pQ)
{
    pQ->front = NULL;
}
BOOLEAN isEmpty(QUEUE *pQ)
{
    return (pQ->front == NULL);
}
BOOLEAN isFull(QUEUE *pQ)
{
    return FALSE;
}
BOOLEAN dequeue(QUEUE *pQ, int *px)
{
    if (isEmpty(pQ))
        return FALSE;
    else {
        (*px) = pQ->front->element;
        pQ->front = pQ->front->next;
        return TRUE;
    }
}
BOOLEAN enqueue(int x, QUEUE *pQ)
{
    if (isEmpty(pQ)) {
        pQ->front = (LIST) malloc(sizeof(struct CELL));
        pQ->rear = pQ->front;
    }
    else {
        pQ->rear->next = (LIST) malloc(sizeof(struct CELL));
        pQ->rear = pQ->rear->next;
    }
    pQ->rear->element = x;
    pQ->rear->next = NULL;
    return TRUE;
}
```

# 最长公共子序列(Longest Common Subsequence, LCS)

动态规划算法，amazing!