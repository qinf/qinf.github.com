title: 数据结构之堆  
date: 2015-06-30 15:30:16  
categories: Technology  
tags: [算法与数据结构]
---
堆是我们常的一种数据结构，这里我们结合`c++`来说明如何使用堆。
## 定义
> 堆通常是一个可以被看做一棵树的数组对象。我们常用的堆是二叉堆，二叉堆是完全二叉树或者是近似完全二叉树。二叉堆满足堆**特性**：父节点的键值总是保持固定的序关系于任何一个子节点的键值，且每个节点的左子树和右子树都是一个二叉堆。  
当父节点的键值总是大于或等于任何一个子节点的键值时为**最大堆**。 当父节点的键值总是小于或等于任何一个子节点的键值时为**最小堆**。
关于堆的更多细节，请参考[堆](https://zh.wikipedia.org/wiki/堆_(数据结构)) [二叉堆](https://zh.wikipedia.org/wiki/二叉堆)
<!-- more -->

## 操作
这里我借用`C++`中`algorithm`里的算法和`priority queue`来演示堆的相关操作。
### `algorithm`中关于堆的操作
1. 建堆操作

 ```c++  
 template< class RandomIt >
 void make_heap( RandomIt first, RandomIt last );
 template< class RandomIt, class Compare >
 void make_heap( RandomIt first, RandomIt last, Compare comp );
 这里在[first, last)范围上创建一个堆，默认使用 operator< 来比较元素。我们可以使用std::greater(小顶堆)和std::less(大顶堆)来作为Compare
 ＃ 假设我们是在vector<int> nums基础上建堆
 make_heap(nums.begin(), nums.end(), greater<int>()); // 小顶堆
 make_heap(nums.begin(), nums.end(), less<int>()); //  大顶堆
 ```
2. 插入元素

 ```c++
 template< class RandomIt >  
 void push_heap( RandomIt first, RandomIt last );
 template< class RandomIt, class Compare >
 void push_heap( RandomIt first, RandomIt last, Compare comp );
  ＃ 假设我们是在vector<int> nums基础执行插入操作
 1. 首先执行vector插入操作
 nums.push_back(val);
 2. 然后执行push_heap操作
 push_heap(nums.begin(), nums.end()); // 默认是大顶堆
 push_heap(nums.begin(), nums.end(), std::greater<int>()); // 小顶堆
 ```
3. 删除元素

 ```c++
 template< class RandomIt >
 void pop_heap( RandomIt first, RandomIt last );
 template< class RandomIt, class Compare >
 void pop_heap( RandomIt first, RandomIt last, Compare comp );
 ＃执行pop_heap的时候是对元素做相应的交换，将要删除的元素放到了相应数据结构的最后一个元素，也就是说讲first和last－1的值做交换，然后重新对[first, last - 1)建堆
 pop_heap(nums.begin(), nums.end()); // 大顶堆pop
 pop_heap(nums.begin(), nums.end(), std::greater<int>); // 小顶堆pop
 nums.pop_back();
 ```
 
### 使用`priority queue`来实现堆的操作
1. 建堆

 ```c++  
 priority_queue<int, vector<int> > minQueue; // 大顶堆
 priority_queue<int, vector<int>, std::greater<int>>  minQueue; // 小顶堆
 priority_queue<int, vector<int>, std::less<int>> maxQueue; // 大顶堆  
```
2. 插入操作  
 ```c++
minQueue.push(val);
```
3. 删除操作  
 ```c++
 minQueue.pop();
 ```
 
## 使用举例
> 如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。

代码如下：

```c++
// 使用 algorithm里面的算法实现
class Solution {
public:
    void Insert(int num) {
        if (!minHeap.empty() && num > minHeap.front()) {
            if (maxHeap.size() == minHeap.size()) {
                swap(num, minHeap.front());
                maxHeap.push_back(num);
                push_heap(maxHeap.begin(), maxHeap.end(), less<int>());
            } else {
                minHeap.push_back(num);
                push_heap(minHeap.begin(), minHeap.end(), greater<int>());
            }
        } else {
            maxHeap.push_back(num);
            push_heap(maxHeap.begin(), maxHeap.end(), less<int>());
            if (maxHeap.size() - minHeap.size() >= 2) {
                minHeap.push_back(maxHeap.front());
                push_heap(minHeap.begin(), minHeap.end(), greater<int>());
                pop_heap(maxHeap.begin(), maxHeap.end(), less<int>());
                maxHeap.pop_back();
            }
        }
    }
    
    double GetMedian() {
        if (maxHeap.size() > minHeap.size()) {
            return maxHeap.front();
        } else {
            return (maxHeap.front() + minHeap.front()) / 2.0;
        }
    }
private:
    vector<int> maxHeap;
    vector<int> minHeap;
};

// 使用优先级队列
class Solution2 {
public:
    void Insert(int num) {
        if (!minQueue.empty() && num > minQueue.top()) {
            if (maxQueue.size() > minQueue.size()) {
                minQueue.push(num);
            } else {
                int top = minQueue.top();
                minQueue.pop();
                minQueue.push(num);
                maxQueue.push(top);
            }
        } else {
            maxQueue.push(num);
            if (maxQueue.size() - minQueue.size() >= 2) {
                minQueue.push(maxQueue.top());
                maxQueue.pop();
            }
        }
    }
    double GetMedian() {
        if (maxQueue.size() > minQueue.size()) {
            return maxQueue.top();
        } else {
            return (maxQueue.top() + minQueue.top()) / 2.0;
        }
    }
private:
    priority_queue<int> maxQueue;
    priority_queue<int, vector<int>, greater<int>> minQueue;   // 大顶堆使用less<T>
};

```
