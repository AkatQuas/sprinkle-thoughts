# Sort

元素个数特别多，无法在主存储器中完成时，必须在磁盘或磁带上完成的排序，叫做外部排序。

* [插入排序](#insertion-sort)
* [一些简单排序的下界](#一些简单排序的下界)
* [ShellSort](#shellsort)

# insertion sort

插入排序由`N-1`趟排序组成。在第p趟，将位置p上多元素向左移动至它在前p+1个元素中正确的位置上。

```C#
tempalte <typename Comparable>
void insertionSort( vector<Comparable> & a)
{
    int j;

    for( int p = 1; p < a.size( ); p++ )
    {
        Comparable tmp = a[ p ];
        for( j = p; j > 0 && tmp < a[ j - 1]; j-- )
            a[ j ] = a[ j - 1 ];

        a[ j ] = tmp;
    }
} 
```

## 插入排序的STL实现

STL中，排序例程不采用具有可比性的项所组成的数组作为单一参数，而是接受一对迭代器来代表在某范围内的起始和终止标志。一个双参数排序例程使用一对迭代器，并假设所有项均可排序。一个三参数的排序例程则还需要有个函数对象作为比较使用。

```C#
# 写一个双参数的排序和一个三参数的排序例程。双参数排序将调用三参数，同时使用less<Object>()作为第三个参数
# 数组访问必须转换为迭代器访问。

template <typename Iterator>
void insertionSort( const Iterator & begin, const Iterator & end )
{
    if ( begin != end )
        insertionSortHelp(begin, end, *begin );
}

template <typename Iterator, typename Object> 
void insertionSortHelp( const Iterator & begin, const Iterator & end, const Object & obj )
{
    insertionSort( begin, end, less<OBject>( ) );
}

# 双参数排序通过使用一个辅助例程来调用三参数排序，该辅助例程将Object作为泛型类型处理
# 三参数排序只有Iterator和Comparator是泛型类型，需要使用相同的技巧来得到一个四参数例程，将一个Object类型的项作为第四个参数，仅仅是为添加一个辅助的泛型类型。

# 下面代码中使用迭代器来取代数组的索引，使用lessThan函数对象类取代operator<的调用

template <typename Iterator, typename Comparator>
void insertionSort( const Iterator & begin, const Iterator & end, Comparator lessThan)
{
    if ( begin != end)
        insertionSort( begin, end, lessThan, *begin );
}

template <typename Iterator, typename Comparator, typename Object>
void insertionSort( const Iterator & begin, const Iterator & end, Comparable lessThan, const Object & obj )
{
    Iterator j;

    for ( Iterator p = begin+1; p != end; p++ )
    {
        Object tmp = *p;
        for ( j = p; j != begin && lessThan( tmp, *( j - 1)); --j )
            *j = *(j-1);

        *j = tmp;
    }
}

# STL程序直接迭代器和函数对象。 
```

## 插入排序的分析

每个嵌套循环都花费N次迭代，为O(N<sup>2</sup>)。
如果输入数据已预先排序，那么运行时间为O(N)

# 一些简单排序的下界

数组的逆序指具有性质`i<j`但是`a[i]>a[j]`的序偶`(i,j)`。

交换两个不按顺序排列的相邻元素消除一个逆序。经过排序的数组没有逆序。

由于算法中还有O(N)项其他工作，插入排序的运行时间是O(I+N)，其中I是原始的逆序数。

于是若逆序是O(N)，则插入排序以线性时间运行。

数学上的定理有

- N个互异元素的数组的平均逆序数是N(N-1)/4
- 通过交换相邻元素进行排序的任何算法平均需要Ω(N<sup>2</sup>)时间

为了使一个排序算法以亚二次或者o(N<sup>2</sup>)的时间运行，必须执行一些比较，特别是要对相距较远的元素进行交换。

排序算法通过消除逆序得以继续进行，而为了有效的进行，还必须每次交换删除多个逆序。

# Shellsort

谢尔排序通过比较相距一定间隔的元素来工作，各趟比较所用的距离随着算法的进行而减小，直到只比较相邻元素的最后一趟排序为止。有时也叫做缩减增量排序。

// todo