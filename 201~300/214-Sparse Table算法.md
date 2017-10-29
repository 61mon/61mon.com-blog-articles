## 一：背景

Sparse Table算法（以下简称ST算法）是用来解决以下问题的，

**区间最值查询（Range Minimum/Maximum Query，简称RMQ问题）**：对于长度为n的数组`array[]`，回答若干询问`RMQ(array, i, j)`$(0 ≤ i, j < n)$，返回数组`array[]`中下标在i，j之间的最小或最大值。

找一个区间最值，最简单的直接比较，复杂度是$O(n)$，所以如果查找次数很少，用ST算法没有意义。ST算法的应用场景就是要对一个数串查询多次的情况。

算法的基本思想就是对串中所有可能的区间组合的最值用二维数组保存，也就是所谓的预处理，查询时直接通过数组下标获取，$O(1)$的时间。下面采用动态规划来对数串进行预处理，也就是填充二维数组。


<!--more-->


## 二：算法过程分析

我们以区间最大值查询为例。

设$f[i][j]$表示数组$array[]$在区间$[i,i+2^j-1]$的最大值。（此即DP的状态，区间长度正好为$2^j$）

很容易发现对于$0≤i<n$，$f[i][0]=array[i]$。（此即DP的初值）

最后看下DP的转移方程。我们把$f[i][j]$所表示的区间平均分成两段，也就是：

$$
[i,i+2^j-1]=[i,i+2^{j-1}-1]+[i+2^{j-1},i+2^j-1] \tag{由 $2^j=2^{j-1}+2^{j-1}$ 得来}
$$

于是得到转移方程：

$$
f[i][j]=\max(f[i][j-1],f[i+2^{j-1}][j-1])
$$

代码如下：

```c++
void RMQ(int array[], int n)
{
    for (int i = 0; i < n; i++)
        f[i][0] = array[i];
        
    int k = log(n) / log(2.0);
    for (int j = 1; j <= k; j++)
        for (int i = 0; i < n; i++)
            if (i + (1 << j) - 1 < n)
                f[i][j] = max(f[i][j - 1], f[i + (1 << (j - 1))][j - 1]);
}
```

预处理完成。最后查询操作代码怎么写呢？对于查询区间$[i,j]$，我们分成两部分，即下图的两个矩形，长度相等，大小等于$⌊log_2(j-i+1)⌋$。

![](https://61mon.com/images/illustrations/sparse_table/1.png)

## 三：完整代码

```c++
/**
 *
 * author : 刘毅（Limer）
 * date   : 2017-08-05
 * mode   : C++
 */

#include <iostream>
#include <algorithm>
#include <cmath>

using namespace std;

const int ROW = 1000;
const int COLUMN = 10;  // log(1000)/log(2.0) ~ 9.96
int f[ROW][COLUMN];

void RMQ(int array[], int n)
{
    for (int i = 0; i < n; i++)
        f[i][0] = array[i];
        
    int k = log(n) / log(2.0);
    for (int j = 1; j <= k; j++)
        for (int i = 0; i < n; i++)
            if (i + (1 << j) - 1 < n)
                f[i][j] = max(f[i][j - 1], f[i + (1 << (j - 1))][j - 1]);
}

int main()
{
    int array[] = { 3,2,4,5,6,8,1,2,9,7 };
    int n = sizeof(array) / sizeof(int);

    RMQ(array, n);

    cout << "数组的下标范围为：0 -- " << n - 1 << "，";
    cout << "请输入需要查询的区间: \n";
    int i, j;
    while (cin >> i >> j)
    {
        int k = log(j - i + 1.0) / log(2.0);
        int max_ans = max(f[i][k], f[j - (1 << k) + 1][k]);
        cout << "最大值是： " << max_ans << endl << endl;
    }
    return 0;
}
```

运行截图如下：

![](https://61mon.com/images/illustrations/sparse_table/2.png)

## 四：总结

整体来说，ST算法的时间复杂度为$O(nlogn+Q)$，其中$Q$为查询次数。

当然对于RMQ问题，不止ST一个方法，线段树也可以，类似问题在ACM竞赛较为常见，有兴趣进一步深入的朋友可以去OJ找相关题源。
