# Overview

散列表，`Hash Table`，实现称为散列，`hashing`，一种用于常数平均时间执行插入、删除和查找的技术，但是元素间的排序信息的树操作将不会得到有效的支持。

# Basic Context

理想的散列表数据结构是一个包含一些项的具有固定大小的数组。表的大小记作`TableSize`，这将理解为散列数据结构的一部分，而不是仅仅是浮动于全局的某个变量。通常的习惯是让表从`0`到`TableSize-1`变化。

将表的每个键映射到`0 ~ TableSize - 1 `的某个数，并将其放到适当的单元中。称这个映射为`散列函数(hash function)`，理想情况下，该函数应该运算简单并且保证任意两个不同的键映射到不同的单元。

要设计好的散列表，需要选择一个高效的函数，并且恰当处理当两个键散列到同一个值时的冲突，以及如何确定散列表的大小。

# Hash Function

一种计算散列中键的数值可以累加键每个位置上的字符的ASCII码，然后对`TablSize`取模。

```C#
/*
 本例程对于表的分布而言未必是最好的，不过简单快捷，如果键过长时，可以选择只计算奇数位置上的字符，这个做法包含“用计算散列函数节省下的时间来补偿由此产生的对均匀分布函数的轻微干扰”的想法
*/
int hash( const string & key, int tableSize )
{
    int hashVal = 0;

    for ( int i = 0, l = key.length() ; i < l ; i++ )
        hashVal = 37 * hashVal + key[ i ];
    
    hashVal %= tableSize;
    if (hashVal < 0 )
        hashVal += tableSize;

    return hashVal;
} 
```

# Load factor

装填因子λ，为散列表中的元素个数与散列表大小的比值。表的平均长度为λ。

# Collision

两个键经过`hash function`之后的指向为同一个值时，这种情况成为冲突，`Collision`。下面讨论处理冲突的最简单的两种方法，[分离链接法](#分离链接法) 和 [开放定址法](#开放定址法)。

## 分离链接法

分离链接法，`separate chaining`，做法是将散列到同一个值的元素保留到一个链表中。这个链表通常是双向链表（空间紧张时可以选择其他实现方式）。

* 执行search的时候，使用hash function来确定需要被遍历的链表，然后再在该链表中进行查询。
* 执行insert时，先检查相应的链表中是否有重复元（重复元的计数加1即可），新元素则插入到链表的前端。

在分离链表法中，`λ = 1.0`。执行一次查找所需的工作是计算`hashValue`所需要的常数时间加上遍历表所用的时间。在一次不成功的查找中，要考察的结点数平均为`λ`。成功的查找则需要遍历大约`1 + (λ/2)`个链。从下面的分析可以看出，散列表的实际大小并不重要，装填因子才重要。分离链表法一般做法是将表的大小尽量与预料的元素个数差不多，即`λ ≈ 1`。如果`λ > 1`，就进行`rehash`来扩充表的大小。另外，使得表的大小是素数来保证一个好的分布的想法是很不错的。
```
针对( 1 + λ/2 )的分析如下：
被搜索的表包含一个存储匹配的结点再加上0个或更多其他的结点。
在N个元素的表以及M个链表中“其他结点”的期望个数为(N-1)/M = λ - 1/M。
一般认为M很大，所以平均来看，一半的“其他结点”被搜索到，再结合匹配结点，
最终得到的平均查找开销为 ( 1 + λ/2 )
```

```C#
# 散列表结构存储一个链表数组，这个数组在构造函数中指定
# 下面是实现分离链接法所需的类接口及例程
# theLists声明中两个>之间需要一个空格，原因是>>是C++的一个运算符，避免编译歧义，所以多加一个空格。
class HashTable
{
    public:
        explicit HashTable( int size = 101 );
        bool contains( const HashedObj & x ) const
        {
            const list<HashedObj> & whichList = theLists[ myhash( x )];
            return find( whichList.begin( ), whichList.end( ), x ) != whichList.end( );
        }

        void makeEmpty( )
        {
            for ( int i = 0, l = theLists.size() ; i < l ; i++ )
                thisLists[i].clear();
        }

        void insert( const HashedObj & x )
        {
            list<HashedObj> & whichList = theLists[ myhash( x ) ];
            if ( find( whichList.begin( ), whicList.end( ), x ) != whichList.end( ) )
                return false;

            whichList.push_back( x );
            
                // rehash;
            if ( ++currentSize > theLists.size( ) )
                rehash( );
            
            return true;
        }

        void remove( const HashedObj & x )
        {
            list<HashedObj> & whichList = theLists[ myhash( x )];
            list<HashedObj>::iterator itr = find(whichList.begin( ), whichList.end( ), x );

            if ( itr == whichList.end( ) )
                return false;
            
            whichList.erase( itr );
            --currentSize;
            return true;
        }
    
    private:
        vector<list<HashedObj> > theLists; // The array of Lists
        int currentSize;

        void rehash( );

        # private function, 将结果分配到一个合适的数组索引中。
        int myhash( const HashedObj & x ) const
        {
            int hashVal = hash( x );
            
            hashVal %= theLists.size( );
            if ( hashVal < 0 )
                hashVal += theLists.size( );
            
            return hashVal;
        } 
};
int hash( const string & key );
int hash( int key );
```

分离链接散列算法的缺点是使用了一些链表，给新单元分配地址需要时间，所以会减慢算法的速度，同时该算法还要求第二种数据结构的实现。

所以另一种解决方法即在遇到冲突时，尝试选择另外一个单元，直到找到空的单元。 

## 开放定址法

### 探测散列表

单元 h<sub>0</sub>(x), h<sub>1</sub>(x), h<sub>2</sub>(x), ...依次进行试选，其中 h<sub>i</sub>(x) = ( hash(x) + f(i) ) mod TableSize, 且 f(0) = 0, f 是冲突解决函数。

这种策略需要将所有数据都置入表内，所以表的装填因子应低于0.5，这样的表称为**探测散列表(probing hash tables)**。

#### 线性探测

线性探测中，冲突解决函数f是i的线性函数，一般情况下`f(i) = i`。

现行探测插入的过程中有**一次聚集(primary clustering)**的现象，即特定区块的数据集中分布。

线性探测的预期探测次数对于插入和不成功的查找大约为 （ 1 + 1/(1-λ)<sup>2</sup>)/2，而对于成功的查找则是(1 + 1/(1-λ))/2。

如果聚集不是问题，即λ比较小的时候，可以证明，一次不成功的查找中期望的探测次数，等于找到一个空单元所期望的探测此时，亦即，可以把**插入一个新元素**的结果看作**一次不成功查找**的结果。可以使用一次不成功查找的开销来计算一次成功查找的平均开销。

数学表明，如果`λ=0.5`，插入的操作平均只需要2.5次探测，而成功的查找平均只要1.5次。 

#### 平方探测

平方探测中，冲突解决函数f是i的二次函数，一般情况下`f(i) = i^2`。

平方探测能够消除线性探测中**一次聚集**的问题。在平方探测中，如果表有至少一半是空的且表大小为素数，可以保证总能够插入一个新的元素。这意味着，哪怕表有比一半多一个位置被填满，插入都有可能失败（可能性虽小但存在）。

```C#
#平方探测的表的类接口以例程，包括嵌套的HashEntry类

class HashTable
{
    public: 
        explicit HashTable( int size = 101 );

        bool contains( const HashedObj & x ) const
            { return isActive( findPos( x ) ); }

        void makeEmpty( )
        {
            currentSize = 0;
            for ( int i = 0; i < array.size( ) ; i++ )
            array[i].info = EMPTY;
        }

        bool insert( const HashedObj & x )
        {
            int currentPos = findPos( x );
            if ( isActive( currentPos ) )
                return false;
            
            array[ currentPos ] = HashEntry( x, ACTIVE );

            if ( ++currentSize > array.size( ) / 2 )
                rehash( );
            
            return true;
        }

        bool remove( cosnt HashedObj & x )
        {
            int currentPos = findPos( x );
            if( !isActive( currentPos ) )
                return false;
            
            array[ currentPos ].info = DELETED;
            return true;
        }

        enum EntryType { ACTIVE, EMPTY, DELETED };

    private:
        struct HashEntry
        {
            HashedObj element;
            EntryType info;

            HashEntry( const HashedObj & e = HashedObj( ), EntryType i = EMPTY ): element( e ), info( i ) { }
        };

        vector<HashEntry> array;
        int currentSize;

        bool isActive( int currentPos ) const
            { return array[ currentPos ].info == ACTIVE; }

        int findPos( const HashedObj & x ) const
        {
            int offset = 1;
            int currentPos = myhash( x );

            while( array[ currentPost ].info != EMPTY && array[ currentPos ].element != x )
            {
                currentPos += offset;
                offset += 2;
                if ( currentPos >= array.size( ) )
                    currentPos -= array.size( );
            }

            return currentPos;
        }
        
        void rehash( );
        int myhash( const HashedObj & x ) const;
} 
```
平方探测排除了**一次聚集**，但是散列到同一个位置上的元素将探测相同的备选单元，称为**二次聚集(secondary clustering)**。模拟结果指出，对每次查找，一般要引起另外的少于一半的探测。

可以使用**双散列**来消除这个理论上的缺憾，代价则是计算额外的散列函数。

#### 双散列

双散列，`double hashing`，一种流行的选择是 f(i) = i * hash<sub>2</sub>(x)，意即将第二个散列函数应用到x并在距离hash<sub>2</sub>(x), 2 * hash<sub>2</sub>(x), ... 进行等距探测。

一个比较好的二次散列函数是 hash<sub>2</sub>(x) = R - ( x mod R )，R 为小于TableSize的素数。

#### summary on 探测散列表:

* 表的大小为素数是重要的，如果不是素数，备选单元可能被提前用完。
* 双散列能够正确实现的时候，预测的探测次数几乎和随机冲突解决方法的情形相同。
* 平方探测不需要使用第二个散列函数，在实践中可以更快更简单。当键为字符串时，其散列函数的计算更加耗时。

## Rehash

如果表的元素填的太满，操作的运行时间将开始变长，且插入操作可能失败！一个解决办法就是建立另外一个大约两倍大的表（使用新的散列函数），扫描整个原始散列表，计算每个有效的元素的新散列值，进行插入。

这样的操作称为再散列，`rehashing`，这种操作是非常昂贵的，运行时间为O(N)，不过因为不是经常发生，所以实际效果还好，特别的，在rehash后的表中已经存在`N/2`次插入，添加到每个插入上的花费基本上是一个常数开销。

如果这种数据结构是程序的一部分，那么其影视不明显的，如果rehash作为交互系统的一部分运行，那么其插入引起的rehash会让严重减慢速度。

rehash可以借助平方探测以多种策略实现。可以预见的是装填因子增加会导致表的性能下降，因此以好的截止点来实现第三种策略，收益相对最大。

* 当表满到一半后，rehash
* 当表出现过插入失败时，rehash
* 途中策略： 如果表到达某一个装填因子时，rehash

```C#
# Rehashing for quadratic probing hash table

void rehash ( )
{
    vector<HashEntry> oldArray = array;

    array.resize( nextPrim( 2 * oldArray.size( ) ) );
    for ( int j = 0; j < array.size( ) ; j++ )
        array[ j ].info = EMPTY;

    currentSize = 0
    for ( int i = 0; i < oldArray.size( ); i++ )
        if ( oldArray[ i ].info == ACTIVE )
            insert( oldArray[ i ].element ); 
}

# Rehashing for separate chaining hash table

void rehash ( )
{
    vector<list<HasheObj> > oldLists = theLists;
    theLists.resize( nextPrime( 2 * theLists.size( ) ) );
    for ( int j = 0; j < theLists.size( ); j++ )
        theLists[ j ].clear( );

    currentSize = 0;
    for ( int i = 0; i < oldLists.size( ); i++ )
    {
        list<HashedObj>::iterator itr = oldLists[ i ].begin( );
        while( itr != oldLists[ i ].end( ) )
            insert( *itr++ );
    }
}
```

