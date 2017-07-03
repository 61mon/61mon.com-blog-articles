Dijkstra算法与Prim算法非常相似，甚至很多初学者觉得它们就是一样的。它们最直观的区别就是目的不同：前者求解最短路径，后者求解最小生成树。

![](http://oi0fekpsr.bkt.clouddn.com/Dijkstra%E7%AE%97%E6%B3%95%E4%B8%8EPrim%E7%AE%97%E6%B3%95%E7%9A%84%E5%8C%BA%E5%88%AB_1.png)

对比上图，

* 最短路径

  ```
  a->b  @length = 2
  a->c  @length = 3
  ```

* 最小生成树

  ```
  a->b->c  @sum = 4
  ```

结论：两者不一样！

好，最后让我们回归代码。

Dijkstra算法与Prim算法都有一个数组，不妨统一称为`R[]`，我们每次都是取`R[]`的最小值，接着更新`R[]`，再取其最小值，，，往复下去。而这两者的区别就发生在**更新**操作之中。

* 最短路径

  ```
  for the weight of arc(u->v)
      if R[v] > R[u] + weight
          R[v] = R[u] + weight
  ```

* 最小生成树

  ```
  for the weight of arc(u->v)
      if R[v] > weight
          R[v] = weight
  ```

其区别，从代码出现，显而易见。