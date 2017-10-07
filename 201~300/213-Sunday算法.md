## 一：背景

Sunday算法是Daniel M.Sunday于1990年提出的字符串模式匹配。其效率在匹配随机的字符串时比其他匹配算法还要更快。Sunday算法的实现可比KMP，BM的实现容易太多。


<!--more-->


## 二：算法过程

假定主串为 "HERE IS A SIMPLE EXAMPLE"，模式串为 "EXAMPLE"。

（1）

![](https://61mon.com/images/illustrations/sunday/1.png)

从头部开始比较，发现不匹配。则Sunday算法要求如下：找到主串中位于模式串后面的第一个字符，即红色箭头所指的“空格”，再在模式串中从后往前找“空格”，没有找到，则直接把模式串移到“空格”的后面。

（2）

![](https://61mon.com/images/illustrations/sunday/2.png)

依旧从头部开始比较，发现不匹配。找到主串中位于模式串后面的第一个字符“L”，模式串中存在“L”，则移动模式串使两个“L”对齐。


（3）

![](https://61mon.com/images/illustrations/sunday/3.png)

找到匹配。

## 三：完整代码

```c++
/**
 *
 * author : 刘毅（Limer）
 * date   : 2017-07-30
 * mode   : C++
 */

#include <iostream>
#include <string>

#define MAX_CHAR 256
#define MAX_LENGTH 1000

using namespace std;

void GetNext(string & p, int & m, int next[])
{
    for (int i = 0; i < MAX_CHAR; i++)
        next[i] = -1;
    for (int i = 0; i < m; i++)
        next[p[i]] = i;
}

void Sunday(string & s, int & n, string & p, int & m)
{
    int next[MAX_CHAR];
    GetNext(p, m, next);

    int j;  // s 的下标
    int k;  // p 的下标
    int i = 0;
    while (i <= n - m)
    {
        j = i;
        k = 0;
        while (j < n && k < m && s[j] == p[k])
            j++, k++;

        if (k == m)
            cout << "在" << i << " 处找到匹配\n";

        if (i + m < n)
            i += (m - next[s[i + m]]);
        else
            return;
    }

    // PS: 匹配失败不作处理
}

int main()
{
    string s, p;
    int n, m;

    while (cin >> s >> p)
    {
        n = s.size();
        m = p.size();
        Sunday(s, n, p, m);
        cout << endl;
    }

    return 0;
}
```

数据测试如下图：

![](https://61mon.com/images/illustrations/sunday/4.png)
