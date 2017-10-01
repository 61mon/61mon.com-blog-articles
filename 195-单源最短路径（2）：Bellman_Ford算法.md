> 系列文章目录
>
> [单源最短路径（1）：Dijkstra算法](https://61mon.com/index.php/archives/194/)
> 单源最短路径（2）：Bellman_Ford算法
> [单源最短路径（3）：SPFA算法](https://61mon.com/index.php/archives/196/)
> [单源最短路径（4）：总结](https://61mon.com/index.php/archives/200/)

## 一：背景
Dijkstra算法是处理单源最短路径的有效算法，但它对存在负权回路的图就会失效。这时候，就需要使用其他的算法来应对这个问题，Bellman-Ford（中文名：贝尔曼-福特）算法就是其中一个。

Bellman-Ford算法不仅可以求出最短路径，也可以检测负权回路的问题。该算法由美国数学家理查德•贝尔曼（Richard Bellman, 动态规划的提出者）和小莱斯特•福特（Lester Ford）发明。


<!--more-->


## 二：算法过程分析
对于一个不存在负权回路的图，Bellman-Ford算法求解最短路径的方法如下：

设其顶点数为n，边数为m。设其源点为source，数组`dist[i]`记录从源点source到顶点i的最短路径，除了`dist[source]`初始化为0外，其它`dist[]`皆初始化为MAX。以下操作循环执行n-1次：

*  对于每一条边arc(u, v)，如果dist[u] + w(u, v) < dist[v]，则使dist[v] = dist[u] + w(u, v)，其中w(u, v)为边arc(u, v)的权值。

n-1次循环，Bellman-Ford算法就是利用已经找到的最短路径去更新其它点的`dist[]`。

接下来再看看Bellman-Ford算法是如何检测负权回路的？

![](https://61mon.com/images/illustrations/SingleSourceShortestPaths/5.png)

检测的方法很简单，只需在求解最短路径的n-1次循环基础上，再进行第n次循环：

* 对于所有边，只要存在一条边arc(u, v)使得dist[u] + w(u, v) < dist[v]，则该图存在负权回路，其中w(u, v)为边arc(u, v)的权值。

| 循环次数 | dist[0] | dist[1] | dist[2] |
| :--: | :-----: | :-----: | :-----: |
| 第1次  |    0    |   -5    |   MAX   |
| 第2次  |    0    |   -5    |   -3    |
| 第3次  |   -3    |   -8    |   -6    |

## 三：完整代码

```c++
/**
 *
 * author : 刘毅（Limer）
 * date   : 2017-05-21
 * mode   : C++
 */

#include <iostream>
#include <stack>

using namespace std;

#define MAX 10000  // 假设权值最大不超过 10000

struct Side
{
    int u;
    int v;
    int w;
};

Side side[10000]; // 记录所有边
int  dist[100];   // 源点到顶点 i 的最短距离
int  path[100];   // 记录最短路的路径
int  vertex_num;  // 顶点数
int  side_num;    // 边数
int  source;      // 源点  

bool BellmanFord()
{
    // 初始化
    for (int i = 0; i < vertex_num; i++)
        dist[i] = (i == source) ? 0 : MAX;

    // n-1 次循环求最短路径
    for (int i = 1; i <= vertex_num - 1; i++)
    {
        for (int j = 0; j < side_num; j++)
        {
            if (dist[side[j].v] > dist[side[j].u] + side[j].w)
            {
                dist[side[j].v] = dist[side[j].u] + side[j].w;
                path[side[j].v] = side[j].u;
            }
        }
    }

    bool flag = true;  // 标记是否有负权回路

    // 第 n 次循环判断负权回路
    for (int i = 0; i < side_num; i++)
    {
        if (dist[side[i].v] > dist[side[i].u] + side[i].w)
        {
            flag = false;
            break;
        }
    }

    return flag;
}

void Print()
{
    for (int i = 0; i < vertex_num; i++)
    {
        if (i != source)
        {
            int p = i;
            stack<int> s;
            cout << "顶点 " << source << " 到顶点 " << p << " 的最短路径是： ";

            while (source != p)  // 路径顺序是逆向的，所以先保存到栈
            {
                s.push(p);
                p = path[p];
            }

            cout << source;
            while (!s.empty())  // 依次从栈中取出的才是正序路径
            {
                cout << "--" << s.top();
                s.pop();
            }
            cout << "    最短路径长度是：" << dist[i] << endl;
        }

    }
}

int main()
{

    cout << "请输入图的顶点数，边数，源点：";
    cin >> vertex_num >> side_num >> source;

    cout << "请输入" << side_num << "条边的信息：\n";
    for (int i = 0; i < side_num; i++)
        cin >> side[i].u >> side[i].v >> side[i].w;

    if (BellmanFord())
        Print();
    else
        cout << "Sorry,it have negative circle!\n";

    return 0;
}
```

运行截图：

![](https://61mon.com/images/illustrations/SingleSourceShortestPaths/6.jpg)


## 四：算法优化

以下除非特殊说明，否则都默认是不存在负权回路的。

先来看看Bellman-Ford算法为何需要循环n-1次来求解最短路径？

读者可以从Dijkstra算法来考虑，想一下，Dijkstra从源点开始，更新`dist[]`，找到最小值，再更新`dist[]` ，，，每次循环都可以确定一个点的最短路。Bellman-Ford算法同样也是这样，它的每次循环也可以确定一个点的最短路，只不过代价很大，因为Bellman-Ford每次循环都是操作所有边。

既然代价这么大，相比Dijkstra算法，Bellman-Ford算法还有啥用？因为后者可以检测负权回路啊。

Bellman-Ford算法的时间复杂度为$O(nm)$，其中n为顶点数，m为边数。

$O(nm)$的时间，大多数都浪费了。考虑一个随机图（点和边随机生成），除了已确定最短路的顶点与尚未确定最短路的顶点之间的边，其它的边所做的都是无用的，大致描述为下图（分割线以左为已确定最短路的顶点）：

![](https://61mon.com/images/illustrations/SingleSourceShortestPaths/7.png)

其中红色部分为所做无用的边，蓝色部分为实际有用的边。

既然只需用到中间蓝色部分的边，那算法优化的方向就找到了，请接着看本系列第三篇文章：spfa算法。
