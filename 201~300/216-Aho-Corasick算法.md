## 一：背景

Aho–Corasick算法（也称AC算法，AC自动机）是由Alfred V. Aho和Margaret J.Corasick 发明的字符串搜索算法，该算法在1975年产生于贝尔实验室，是著名的多模匹配算法之一。

一个典型应用就是：给出$k$个单词，再给出一段文章（长度是$n$），让你找出有多少个单词在文章里出现过。

与其它模式串匹配不同，KMP算法是单模式串的匹配算法，AC算法是多模式串的匹配算法，匹配所需的时间复杂度是$O(n)$。

AC算法建立在字典树基础上，如果您还不了解字典树，可以参考[字典树入门](http://www.61mon.com/index.php/archives/215/)。

## 二：算法过程分析

以上述所说的典型应用为例，现给定3个单词{"china", "hit", "use"}，再给定一段文本"chitchat"，求有多少个单词出现在文本中。

（1）

![](http://oi0fekpsr.bkt.clouddn.com/AC%E7%AE%97%E6%B3%95_1.png)

根据单词{"china", "hit", "use"}建立字典树。

（2）

![](http://oi0fekpsr.bkt.clouddn.com/AC%E7%AE%97%E6%B3%95_2.png)

根据所给文本“chitchat”依次匹配，图中所示“chi”为匹配成功的字符串。

（3）

![](http://oi0fekpsr.bkt.clouddn.com/AC%E7%AE%97%E6%B3%95_3.png)

当匹配到第四个字符时，“t”和“n”匹配失败。

（4）

![](http://oi0fekpsr.bkt.clouddn.com/AC%E7%AE%97%E6%B3%95_4.png)

我们此时是知道已匹配成功的字符串的，即“chi”。

AC算法的核心就是**在所有给定的单词中，找到这样的一个单词，使其与已匹配成功字符串的相同前后缀最长，利用这个最长的相同前后缀实现搜索跳转。**如上图，单词“hit”与已匹配成功字符串“chi”的最长相同前后缀为“hi”，因此下一步从单词“hit”的“t”开始搜索。

（4）

![](http://oi0fekpsr.bkt.clouddn.com/AC%E7%AE%97%E6%B3%95_5.png)

此时“t”是匹配的，在文本“chitchat”中找到一个单词“hit”。

其实到这里，AC算法的思想已经基本呈现在大家面前了。剩下的问题就是如何解决第（3）步所述的“核心”。

#### AC算法核心

在每个结点里设置一个指针（我们称之为fail指针），指向跳转的位置。

![](http://oi0fekpsr.bkt.clouddn.com/AC%E7%AE%97%E6%B3%95_6.png)

对于跳转位置的选择，基于以下两点：

1. 对于根结点的所有儿子结点，它们的fail指针指向根结点；
2. 而对于其它结点，不妨设该结点上的字符为`ch`，沿着它的父亲结点的fail指针走，直到走到一个结点，它的儿子中也有字符为`ch`的结点，然后把该结点的fail指针指向那个字符为`ch`的结点。如果一直走到了根结点都没找到，那就把fail指针指向根结点。

对于第2点，初学者可能有点不理解，我这里稍微解释下。请仔细阅读上面的第（3）步过程，利用父亲结点的最长相同前后缀来找到儿子的最长相同前后缀。

## 三：完整代码

```c++
/**
*
* author 刘毅（Limer）
* date   2017-08-10
* mode   C++
*/
#include <iostream>
#include <queue>
using namespace std;

#define TREE_WIDTH 26

struct Node
{
    int end;
    Node * fail;
    Node * next[TREE_WIDTH];
    Node()
    {
        this->end = 0;
        this->fail = nullptr;
        for (int i = 0; i < TREE_WIDTH; i++)
            this->next[i] = nullptr;
    }
};

class AC
{
private:
    Node * root;
public:
    AC();
    ~AC();
    void _destroy(Node * _t);
    void add(char * s);
    void build_fail_pointer();
    int ac_automaton(char * t);
};

AC::AC()
{
    root = new Node;
}

AC::~AC()
{
    _destroy(root);
}

void AC::_destroy(Node * _t)
{
    for (int i = 0; i < TREE_WIDTH; i++)
        if (_t->next[i])
            _destroy(_t->next[i]);
    delete _t;
}

void AC::add(char * s)
{
    Node * t = root;
    while (*s)
    {
        if (t->next[*s - 'a'] == nullptr)
            t->next[*s - 'a'] = new Node;
        t = t->next[*s - 'a'];
        s++;
    }
    t->end++;  // 假设单词可重复
}

void AC::build_fail_pointer()
{
    queue<Node*> Q;

    for (int i = 0; i < TREE_WIDTH; i++)
    {
        if (root->next[i])
        {
            Q.push(root->next[i]);
            root->next[i]->fail = root;
        }
    }

    Node * parent = nullptr;
    Node * son = nullptr;
    Node * p = nullptr;
    while (!Q.empty())
    {
        parent = Q.front();
        Q.pop();
        for (int i = 0; i < TREE_WIDTH; i++)
        {
            if (parent->next[i])
            {
                Q.push(parent->next[i]);
                son = parent->next[i];
                p = parent->fail;
                while (p)
                {
                    if (p->next[i])
                    {
                        son->fail = p->next[i];
                        break;
                    }
                    p = p->fail;
                }
                if (!p)  son->fail = root;
            }
        }
    }
}

int AC::ac_automaton(char * t)
{
    int ans = 0;

    int pos;
    Node * pre = root;
    Node * cur = nullptr;
    while (*t)
    {
        pos = *t - 'a';
        if (pre->next[pos])
        {
            cur = pre->next[pos];
            while (cur != root)
            {
                if (cur->end >= 0)
                {
                    ans += cur->end;
                    cur->end = -1;  // 避免重复查找
                }
                else
                    break;  // 等于 -1 说明以前这条路径已找过，现在无需再找
                cur = cur->fail;
            }
            pre = pre->next[pos];
            t++;
        }
        else
        {
            if (pre == root)
                t++;
            else
                pre = pre->fail;
        }
    }
    return ans;
}

int main()
{
    int n;
    char s[1000];
    while (1)
    {
        cout << "请输入单词个数：";
        cin >> n;

        AC tree;
        cout << "请输入" << n << "个单词：\n";
        while (n--)
        {
            cin >> s;
            tree.add(s);
        }
      
        cout << "请输入搜索文本：";
        cin >> s;
      
        tree.build_fail_pointer();
        cout << "共有" << tree.ac_automaton(s) << "个单词匹配" << endl << endl;
    }
    return 0;
}
```

运行截图如下：

![](http://oi0fekpsr.bkt.clouddn.com/AC%E7%AE%97%E6%B3%95_7.png)
