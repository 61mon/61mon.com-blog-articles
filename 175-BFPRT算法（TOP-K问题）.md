## 一：背景介绍
在一堆数中求其前k大或前k小的问题，简称TOP-K问题。而目前解决TOP-K问题最有效的算法即是"BFPRT算法"，又称为"中位数的中位数算法"，该算法由Blum、Floyd、Pratt、Rivest、Tarjan提出，最坏时间复杂度为$O(n)$。

在首次接触TOP-K问题时，我们的第一反应就是可以先对所有数据进行一次排序，然后取其前k即可，但是这么做有两个问题：

* 快速排序的平均复杂度为$O(nlogn)$，但最坏时间复杂度为$O(n^2)$，不能始终保证较好的复杂度。
* 我们只需要前k大的，而对其余不需要的数也进行了排序，浪费了大量排序时间。

除这种方法之外，堆排序也是一个比较好的选择，可以维护一个大小为k的堆，时间复杂度为$O(nlogk)$。

那是否还存在更有效的方法呢？受到快速排序的启发，通过**修改快速排序中主元的选取方法**可以降低快速排序在**最坏情况下的时间复杂度**，并且我们的目的只是求出前k，故递归的规模变小，速度也随之提高。下面来简单回顾下快速排序的过程，以升序为例：

（1）：选取主元（数组中随机一个元素）；
（2）：以选取的主元为分界点，把小于主元的放在左边，大于主元的放在右边；
（3）：分别对左边和右边进行递归，重复上述过程。


<!--more-->


## 二：BFPRT算法过程及代码

BFPRT算法步骤如下：

（1）：选取主元；
&emsp;&emsp;（1.1）：将n个元素划分为$⌊\frac n5⌋$个组，每组5个元素，若有剩余，舍去；
&emsp;&emsp;（1.2）：使用插入排序找到$⌊\frac n5⌋$个组中每一组的中位数；
&emsp;&emsp;（1.3）：对于（1.2）中找到的所有中位数，调用BFPRT算法求出它们的中位数，作为主元；
（2）：以（1.3）选取的主元为分界点，把小于主元的放在左边，大于主元的放在右边；
（3）：判断主元的位置与k的大小，有选择的对左边或右边递归。

上面的描述可能并不易理解，先看下面这幅图：

![](https://61mon.com/images/illustrations/BFPRT/1.png)

BFPRT()调用GetPivotIndex()和Partition()来求解第k小，在这过程中，GetPivotIndex()也调用了BFPRT()，即GetPivotIndex)和BFPRT()为互递归的关系。

下面为代码实现，其所求为**前K小的数**：

```c++
/**
 * BFPRT算法（前K小数问题）
 *
 * author : 刘毅（Limer）
 * date   : 2017-01-25
 * mode   : C++
 */
#include <iostream>
#include <algorithm>

using namespace std;

/* 插入排序，返回中位数下标 */
int InsertSort(int array[], int left, int right)
{
    int temp;
    int j;

    for (int i = left + 1; i <= right; i++)
    {
        temp = array[i];
        j = i - 1;
        while (j >= left && array[j] > temp)
            array[j + 1] = array[j--];
        array[j + 1] = temp;
    }

    return ((right - left) >> 1) + left;
}

/* 返回中位数的中位数下标 */

int BFPRT(int array[], int left, int right, const int & k);

int GetPivotIndex(int array[], int left, int right)
{
    if (right - left < 5)
        return InsertSort(array, left, right);

    int sub_right = left - 1;

    for (int i = left; i + 4 <= right; i += 5)
    {
        int index = InsertSort(array, i, i + 4);  // 找到五个元素的中位数的下标
        swap(array[++sub_right], array[index]);   // 依次放在左侧
    }

    return BFPRT(array, left, sub_right, ((sub_right - left + 1) >> 1) + 1);
}

/* 利用中位数的中位数的下标进行划分，返回分界线下标 */
int Partition(int array[], int left, int right, int pivot_index)
{
    swap(array[pivot_index], array[right]);  // 把主元放置于末尾
    int divide_index = left;                 // 跟踪划分的分界线

    for (int i = left; i < right; i++)
    {
        if (array[i] < array[right])
            swap(array[divide_index++], array[i]);  // 比主元小的都放在左侧
    }

    swap(array[divide_index], array[right]);  // 最后把主元换回来

    return divide_index;
}

int BFPRT(int array[], int left, int right, const int & k)
{
    int pivot_index = GetPivotIndex(array, left, right);            // 得到中位数的中位数下标
    int divide_index = Partition(array, left, right, pivot_index);  // 进行划分，返回划分边界
    int num = divide_index - left + 1;

    if (num == k)
        return divide_index;
    else if (num > k)
        return BFPRT(array, left, divide_index - 1, k);
    else
        return BFPRT(array, divide_index + 1, right, k - num);
}

int main()
{
    int k = 5;
    int array[10] = { 1,1,2,3,1,5,-1,7,8,-10 };

    cout << "原数组：";
    for (int i = 0; i < 10; i++)
        cout << array[i] << " ";
    cout << endl;

    cout << "第" << k << "小值为：" << array[BFPRT(array, 0, 9, k)] << endl;

    cout << "变换后的数组：";
    for (int i = 0; i < 10; i++)
        cout << array[i] << " ";
    cout << endl;

    return 0;
}
```

运行如下：

![](https://61mon.com/images/illustrations/BFPRT/2.PNG)

## 三：时间复杂度分析

BFPRT算法在最坏情况下的时间复杂度是$O(n)$，下面予以证明。令$T(n)$为所求的时间复杂度，则有：

$$
T(n)≤T(\frac n 5)+T(\frac {7n}{10})+c⋅n\tag{c为一个正常数}
$$

其中：

- $T(\frac n 5)$来自GetPivotIndex()，n个元素，5个一组，共有$⌊\frac n5⌋$个中位数；
- $T(\frac {7n}{10})$来自BFPRT()，在$⌊\frac n5⌋$个中位数中，主元x大于其中 $\frac 12⋅\frac n5=\frac n{10}$的中位数，而每个中位数在其本来的5个数的小组中又大于或等于其中的3个数，所以主元x至少大于所有数中的$\frac n{10}⋅3=\frac {3n}{10}$个。即划分之后，任意一边的长度至少为$\frac 3{10}$，在最坏情况下，每次选择都选到了$\frac 7{10}$的那一部分。
- $c⋅n$来自其它操作，比如InsertSort()，以及GetPivotIndex()和Partition()里所需的一些额外操作。

设$T(n)=t⋅n$，其中t为未知，它可以是一个正常数，也可以是一个关于n的函数，代入上式：

$$
\begin{align}
t⋅n&≤\frac {t⋅n}5+\frac{7t⋅n}{10}+c⋅n \tag{两边消去n}\\
t&≤\frac t 5+\frac {7t}{10}+c \tag{再化简}\\
t&≤10c \tag{c为一个正常数}
\end{align}
$$

其中c为一个正常数，故t也是一个正常数，即$T(n)≤10c⋅n$，因此$T(n)=O(n)$，至此证明结束。

接下来我们再来探讨下BFPRT算法为何选5作为分组主元，而不是2, 3, 7, 9呢？

首先排除偶数，对于偶数我们很难取舍其中位数，而奇数很容易。再者对于3而言，会有$T(n)≤T(\frac n 3)+T(\frac {2n}3)+c⋅n$，它本身还是操作了n个元素，与以5为主元的$\frac {9n}{10}$相比，其复杂度并没有减少。对于7，9，...而言，上式中的10c，其整体都会增加，所以与5相比，5更适合。

## 四：参考文献

- 算法导论（第3版）. Page 120.
