# Priority Queue

优先队列针对于在队列基础上，有些元素具有更高优先级的情况。据称，优先队列的数据结构是计算机科学中最雅致对一种。

# Implementation

优先队列基础两种操作：`insert`（入队，等价于`enqueue`）和`deleteMin`（找出、返回和删除队列中最小的元素，出队）。

**简单的实现：**

* 使用简单链表，在表头以O(1)执行插入操作，遍历该表以O(N)删除最小元。
* 使用简单链表，始终保持表对排序状态，插入操作代价为O(N)，删除代价为O(1)。
* 使用二叉查找树，对于插入和删除对操作均为O(logN)。不过它支持大量不需要对操作，似乎有点冗余。

**二叉堆**

堆的结构性质和堆序性质。

堆是一棵被完全填满对二叉树，可能的例外是在底层，底层上的元素从左到右填入。

数学上，一棵高为h的完全二叉树有2<sup>h</sup>到2<sup>h+1</sup>-1个结点，即二叉树对高为[logN]。

观察上，完全二叉树可以用数组表示，不需要链表。

对于数组中，任一位置`i`上的元素，其左儿子在位置`2i`，其右儿子在位置`2i+1`，就在左儿子后面，其父亲在位置`[i/2]`上。

堆序性质：除根结点外，对于堆中其他任一结点X，其父亲的键小于（或等于）X中的键。因此最小元总可以在根处找到，得以常数时间得到附加操作`findMin`。

```C#
# 优先队列
class BinaryHeap
{
    public:
        explicit BinaryHeap( int capacity = 100 );

        explicit BinaryHeap( const vector<Comparable> & items ): array( items.size( ) + 10), currentSize ( items.size( ))
        {
            for( int i = 0; i < items.size( ); i++ )
                array[ i + 1 ] = items[ i ];
            buildHeap( );
        }

        bool isEmpty( ) const;
        const Comparable & findMin( ) const;

        void insert( const Comparable & x )
        {
            if ( currentSize == array.size( ) - 1 )
                array.resize( array.size( ) * 2 );
            
            int hole = ++currentSize;
            for ( ; hole > 1 && x < array[ hole / 2 ]; hole / 2)
                array [ hole ] = array[ hole / 2 ];
            
            array[ hole ] = x;
        }

        void deleteMin( )
        {
            if ( isEmpty( ) )
                throw UnderflowException( );
            
            array[ 1 ] = array[ currentSize-- ];
            percolateDown( 1 );
        }

        void deleteMin( Comparable & minItem )
        {
            if ( isEmpty( ) )
                throw UnderflowException( );

            minItem = array[ 1 ];
            array[ 1 ] = array[ currentSize-- ];
            percolateDown( 1 ); 

        }

        void makeEmpty( );

    private:
        int currentSize;
        vector<Comparable> array;
        
        void buildHeap( )
        {
            for ( int i = currentSize / 2; i > 2 ; i-- )
                percolateDown( i );
        }

        void percolateDown( int hole )
        {
            int child;
            Comparable tmp = array[ hole ];

            for ( ; hole * 2 <= currentSize; hole = child )
            {
                # find the children index
                child = hole * 2;
                # find the smaller child
                if ( child != currentSize && array[ child + 1 ] < array[ child ] )
                    child++;
                # check the child is small enough
                if ( array[ child ] < tmp )
                    array[ hole ] = array[ child ];
                else 
                    break; 
            }
            array[ hole ] = tmp;
        }
}
```

## Basic Operation

**insert**

为了将一个元素X插入到堆中，需要在下一个空闲位置创建一个空穴。

- 如果X能放入空穴而不破坏堆序，那么插入完成。
- 如果不能直接插入空穴，需要把空穴的父节点上的元素移入空穴，再次检测X是否能插入空穴，重复该过程直到X能被正确放入空穴为止。

上面这种策略称为上滤，`percolate up`。数学证明，执行一次插入平均需要2.607次比较，insert操作将元素平均上移1.607层。

**deleteMin**

最小元就在根结点。

当删除最小元之后，在根结点出现了一个空穴。直观地，将堆中最后一个元素X移动到堆的某个地方。

- 如果X能被放入空穴，那么直接放入，deleteMin完成。
- 不能直接放入时，可以把空穴中，较小的儿子放入空穴，这样就把空穴向下推了一层，重复这样对步骤，直到X能够被放入到空穴中。

上面这种策略称为下滤，`percolate down`。平均运行时间为O(logN)。实际情况中，将出现一个结点只有一个儿子的情况，所以附加了一个测试，检查儿子的情况。

另一种巧妙对解法是当堆大小为偶数对时候，从每个下滤开始处，可将其值大于堆中任何元素对标记放到堆的终端后面对位置上，这样做就不再需要测试右儿子存在与否，不过还是需要测试何时到达底层，对于每一个树叶，算法需要一个标记。这种方法需要慎重选择。

## Other Operation

**decreaseKey**

`decreaseKey(p,Δ)`操作用于减小位置p处元素的值，减小幅度为正的Δ。这有可能破坏堆序性质，需要通过*上滤*操作来调整。

**increaseKey**

`increaseKey(p,Δ)`操作增加位置p处元素的值，增加幅度为正的Δ。需要用*下滤*来调整。

**remove**

`remove(p)`操作删除堆中位置p上的结点。先执行`decreaseKey(p,Δ)`，再执行`deleteMin()`。

**buildHeap**

有时候二叉堆通过项的原始集合来构造。直观地，可以通过N次连续的insert来完成。算法的总运行时间为O(N)平均时间，最坏情形时间为O(NlogN)。

通常的构造过程是将N项以任意顺序放入树中，然后进行下滤过程，既可以生成堆序的树。

todo 6.4