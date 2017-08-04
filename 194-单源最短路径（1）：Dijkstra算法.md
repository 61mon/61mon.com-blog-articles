> 系列文章目录
>
> 单源最短路径（1）：Dijkstra算法
> [单源最短路径（2）：Bellman_Ford算法](http://www.61mon.com/index.php/archives/195/)
> [单源最短路径（3）：SPFA算法](http://www.61mon.com/index.php/archives/196/)
> [单源最短路径（4）：总结](http://www.61mon.com/index.php/archives/200/)

## 一：背景
Dijkstra算法（中文名：迪杰斯特拉算法）是由荷兰计算机科学家Edsger Wybe Dijkstra提出。该算法常用于路由算法或者作为其他图算法的一个子模块。举例来说，如果图中的顶点表示城市，而边上的权重表示城市间开车行经的距离，该算法可以用来找到两个城市之间的最短路径。


<!--more-->


## 二：算法过程
我们用一个例子来具体说明迪杰斯特拉算法的流程。
![](http://oi0fekpsr.bkt.clouddn.com/%E5%8D%95%E6%BA%90%E6%9C%80%E7%9F%AD%E8%B7%AF%E5%BE%84__1.png#mirages-width=450&mirages-height=380&mirages-cdn-type=1)

定义源点为0，`dist[i]`为源点0到顶点i的最短路径。其过程描述如下：

|  步骤  | dist[1] | dist[2] | dist[3] | dist[4] |     已找到的集合     |
| :--: | :-----: | :-----: | :-----: | :-----: | :------------: |
| 第1步  |    8    |    1    |    2    |   +∞    |     { 2 }      |
| 第2步  |    8    |    ×    |    2    |    4    |    { 2, 3 }    |
| 第3步  |    5    |    ×    |    ×    |    4    |  { 2, 3, 4 }   |
| 第4步  |    5    |    ×    |    ×    |    ×    | { 2, 3, 4, 1 } |
| 第5步  |    ×    |    ×    |    ×    |    ×    | { 2, 3, 4, 1 } |

第1步：从源点0开始，找到与其邻接的点：1，2，3，更新`dist[]`数组，因0不与4邻接，故`dist[4]`为正无穷。在`dist[]`中找到最小值，其顶点为2，即此时已找到0到2的最短路。

第2步：从2开始，继续更新`dist[]`数组：2与1不邻接，不更新；2与3邻接，因`0→2→3`比`dist[3]`大，故不更新`dist[3]` ；2与4邻接，因`0→2→4`比`dist[4]`小，故更新`dist[4]`为4。在`dist[]`中找到最小值，其顶点为3，即此时又找到0到3的最短路。

第3步：从3开始，继续更新`dist[]`数组：3与1邻接，因`0→3→1`比`dist[1]`小，更新`dist[1]`为5；3与4邻接，因`0→3→4`比`dist[4]`大，故不更新。在`dist[]`中找到最小值，其顶点为4，即此时又找到0到4的最短路。

第4步：从4开始，继续更新`dist[]`数组：4与1不邻接，不更新。在`dist[]`中找到最小值，其顶点为1，即此时又找到0到1的最短路。

第5步：所有点都已找到，停止。

对于上述步骤，你可能存在以下的疑问：

![](http://oi0fekpsr.bkt.clouddn.com/%E5%8D%95%E6%BA%90%E6%9C%80%E7%9F%AD%E8%B7%AF%E5%BE%84_3.png#mirages-width=460&mirages-height=360&mirages-cdn-type=1)

若A作为源点，与其邻接的只有B，C，D三点，其`dist[]`最小时顶点为C，即就可以确定`A→C`为A到C的最短路。但是我们存在疑问的是：**是否还存在另一条路径使A到C的距离更小？** 用反证法证明。

假设存在如上图的红色虚线路径，使`A→D→C`的距离更小，那么`A→D`作为`A→D→C`的子路径，其距离也比`A→C`小，这与前面所述“`dist[]`最小时顶点为C”矛盾，故假设不成立。因此这个疑问不存在。

根据上面的证明，我们可以推断出，**Dijkstra每次循环都可以确定一个顶点的最短路径，故程序需要循环n-1次。**

## 三：完整代码

```c++
/**
*
* author 刘毅（Limer）
* date   2017-05-17
* mode   C++
*/
#include<iostream>
using namespace std;

int matrix[100][100];  //邻接矩阵
bool visited[100];     //标记数组
int dist[100];         //源点到顶点i的最短距离
int path[100];         //记录最短路的路径
int source;            //源点
int vertex_num;        //顶点数
int arc_num;           //弧数

void Dijkstra(int source)
{
	memset(visited, 0, sizeof(visited));  //初始化标记数组
	visited[source] = true;
	for (int i = 0; i < vertex_num; i++)
	{
		dist[i] = matrix[source][i];
		path[i] = source;
	}

	int min_cost;        //权值最小
	int min_cost_index;  //权值最小的下标
	for (int i = 1; i < vertex_num; i++)  //找到源点到另外vertex_num-1个点的最短路径
	{
		min_cost = INT_MAX;
		for (int j = 0; j < vertex_num; j++)
		{
			if (visited[j] == false && dist[j] < min_cost)  //找到权值最小
			{
				min_cost = dist[j];
				min_cost_index = j;
			}
		}

		visited[min_cost_index] = true;  //该点已找到，进行标记

		for (int j = 0; j < vertex_num; j++)  //更新dist数组
		{
			if (visited[j] == false && 
				matrix[min_cost_index][j] != INT_MAX &&  //确保两点之间有弧
				matrix[min_cost_index][j] + min_cost < dist[j])
			{
				dist[j] = matrix[min_cost_index][j] + min_cost;
				path[j] = min_cost_index;
			}
		}
	}
}

int main()
{
	cout << "请输入图的顶点数（<100）：";
	cin >> vertex_num;
	cout << "请输入图的弧数：";
	cin >> arc_num;

	for (int i = 0; i < vertex_num; i++)
		for (int j = 0; j < vertex_num; j++)
			matrix[i][j] = (i != j) ? INT_MAX : 0;  //初始化matrix数组

	cout << "请输入弧的信息：\n";
	int u, v, w;
	for (int i = 0; i < arc_num; i++)
	{
		cin >> u >> v >> w;
		matrix[u][v] = matrix[v][u] = w;
	}

	cout << "请输入源点（<" << vertex_num << "）：";
	cin >> source;
	Dijkstra(source);

	for (int i = 0; i < vertex_num; i++)
	{
		if (i != source)
		{
			cout << source << "到" << i << "最短距离是：" << dist[i] << "，路径是：" << i;
			int t = path[i];
			while (t != source)
			{
				cout << "--" << t;
				t = path[t];
			}
			cout << "--" << source << endl;
		}
	}

	return 0;
}
```

输入数据，结果为：

![](http://oi0fekpsr.bkt.clouddn.com/%E5%8D%95%E6%BA%90%E6%9C%80%E7%9F%AD%E8%B7%AF%E5%BE%84_2.png#mirages-width=356&mirages-height=277&mirages-cdn-type=1)

## 四：时间复杂度
设图的边数为m，顶点数为n。

Dijkstra算法最简单的实现方法是用一个数组来存储所有顶点的`dist[]`（即本程序采用的策略），所以搜索`dist[]`中最小元素的运算需要线性搜索$O(n)$。这样的话算法的运行时间是$O(n^2)$。

对于边数远少于$n^2$的稀疏图来说，我们可以用邻接表来更有效的实现该算法。同时需要将一个二叉堆或者斐波纳契堆用作优先队列来查找最小的顶点。当用到二叉堆的时候，算法所需的时间为 $O((m+n)logn)$，斐波纳契堆能稍微提高一些性能，让算法运行时间达到$O(m+nlogn)$。然而，使用斐波纳契堆进行编程，常常会由于算法常数过大而导致速度没有显著提高。

关于$O((m+n)logn)$的由来，我简单的证明了下（仅个人看法，不保证其正确性）：

*  把`dist[]`数组调整成最小堆，需要$O(n)$的时间； 

*  因为是最小堆，所以每次取出最小值只需$O(1)$的时间，接着把数组尾的数放置堆顶，并花费$O(logn)$的时间重新调整成最小堆；

*  我们需要n-1次操作才可以找出剩下的n-1个点，在这期间，大约需要访问m次边，每次访问都可能造成`dist[]`的改变，因此还需要$O(logn)$的时间来进行最小堆的重新调整（从当前`dist[]`改变的位置往上调整）。

综上所述：总的时间复杂度为：$O(n)+O(nlogn)+O(mlogn)=O((m+n)logn)$

最后简单说下Dijkstra优化时二叉堆的两种实现方式：

* 优先队列，把每个顶点的序号和其`dist[]`压在一个结构体再放进队列里；
* 自己建一个小顶堆`heap[]`，存储顶点序号，再用一个数组`pos[]`记录第i个顶点在堆中的位置。

相比之下，前者的编码难度较低，因此在平时编程甚至算法竞赛中，都是首选。

## 五：该算法的缺陷

Dijkstra算法有个巨大的缺陷，请考虑下面这幅图：

![](http://oi0fekpsr.bkt.clouddn.com/%E5%8D%95%E6%BA%90%E6%9C%80%E7%9F%AD%E8%B7%AF%E5%BE%84_4.png#mirages-width=550&mirages-height=300&mirages-cdn-type=1)

`u→v`间存在一条负权回路（所谓的负权回路，维基和百科并未收录其名词，但从网上的一致态度来看，其含义为：如果存在一个环（从某个点出发又回到自己的路径），而且这个环上所有权值之和是负数，那这就是一个负权环，也叫负权回路），那么只要无限次地走这条负权回路，便可以无限制地减少它的最短路径权值，这就变相地说明最短路径不存在。一个不存在最短路径的图，Dijkstra算法无法检测出这个问题，其最后求解的`dist[]`也是错的。

那么对于上述的“一个不存在最短路径的图”，我们该用什么方法来解决呢？请接着看本系列第二篇文章。
