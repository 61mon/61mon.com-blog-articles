## 一：背景

上一篇文章，我介绍了[KMP算法](https://61mon.com/index.php/archives/183/)。

但是，它并不是效率最高的算法，实际采用并不多。各种文本编辑器的"查找"功能（Ctrl+F），大多采用Boyer-Moore算法。Boyer-Moore算法不仅效率高，而且构思巧妙。1977年，德克萨斯大学的Robert S. Boyer教授和J Strother Moore教授发明了这种算法。


<!--more-->


## 二：算法过程

以下摘自阮一峰的[字符串匹配的Boyer-Moore算法](http://www.ruanyifeng.com/blog/2013/05/boyer-moore_string_search_algorithm.html)，并作稍微修改。

（1）

![](https://61mon.com/images/illustrations/boyer_moore/1.png)

假定字符串为"HERE IS A SIMPLE EXAMPLE"，搜索词为"EXAMPLE"。

（2）

![](https://61mon.com/images/illustrations/boyer_moore/2.png)

首先，"字符串"与"搜索词"头部对齐，从尾部开始比较。

这是一个很聪明的想法，因为如果尾部字符不匹配，那么只要一次比较，就可以知道前7个字符（整体上）肯定不是要找的结果。

我们看到，"S"与"E"不匹配。这时，**"S"就被称为"坏字符"（bad character），即不匹配的字符**。我们还发现，"S"不包含在搜索词"EXAMPLE"之中，这意味着可以把搜索词直接移到"S"的后一位。

（3）

![](https://61mon.com/images/illustrations/boyer_moore/3.png)

依然从尾部开始比较，发现"P"与"E"不匹配，所以"P"是"坏字符"。但是，"P"包含在搜索词"EXAMPLE"之中。所以，将搜索词后移两位，两个"P"对齐。

（4）

![](https://61mon.com/images/illustrations/boyer_moore/4.png)

我们由此总结出**"坏字符规则"：后移位数 = 坏字符的位置 - 搜索词中的上一次出现位置。**


我们约定，如果"坏字符"不包含在搜索词之中，则上一次出现位置为-1。

举个例子，以前面第三步的"P"为例，它作为"坏字符"，出现在（相对）搜索词的第6位（从0开始编号），在搜索词中的上一次出现位置为4，所以后移 6 - 4 = 2位。再以前面第二步的"S"为例，它出现在第6位（相对搜索词），上一次出现位置是-1（即未出现），则整个搜索词后移 6 - (-1) = 7位。

（5）

![](https://61mon.com/images/illustrations/boyer_moore/5.png)

依然从尾部开始比较，"E"与"E"匹配。

（6）

![](https://61mon.com/images/illustrations/boyer_moore/6.png)

比较前面一位，"LE"与"LE"匹配。

（7）

![](https://61mon.com/images/illustrations/boyer_moore/7.png)

比较前面一位，"PLE"与"PLE"匹配。

（8）

![](https://61mon.com/images/illustrations/boyer_moore/8.png)

比较前面一位，"MPLE"与"MPLE"匹配。**我们把这种情况称为"好后缀"（good suffix），即所有尾部匹配的字符串。**注意，"MPLE"、"PLE"、"LE"、"E"都是好后缀。

为了接下来方便讲解，在这里作一下区分：**我们统一称"MPLE"、"PLE"、"LE"、"E"为"好后缀"，其中"MPLE"为"最长好后缀"，"PLE"、"LE"、"E"统一称为"真好后缀"。**

（9）

![](https://61mon.com/images/illustrations/boyer_moore/9.png)

比较前一位，发现"I"与"A"不匹配。所以，"I"是"坏字符"。

（10）

![](https://61mon.com/images/illustrations/boyer_moore/10.png)

根据"坏字符规则"，此时搜索词应该后移 2 - （-1）= 3 位。问题是，此时有没有更好的移法？

（11）

![](https://61mon.com/images/illustrations/boyer_moore/11.png)

我们知道，此时存在"好后缀"。

所以，可以采用**"好后缀规则"：后移位数 = 好后缀的位置 - 搜索词中的上一次出现位置。**

这个规则有三个注意点：

1. "好后缀"的位置以最后一个字符为准。假定"ABCDEF"的"好后缀"是"EF"、"F"，则它的位置以"F"为准，即5（从0开始计算）。
2. 如果"好后缀"在搜索词中都只出现一次，则它们的上一次出现位置为 -1。比如，ABCDEF"的好后缀是"EF"、"F"，但它们在"ABCDEF"之中都只出现一次，则它们的上一次出现位置为-1（即未出现）。
3. 如果有多个"好后缀"在搜索词中都出现多次，那我们应该选哪个作为上一次出现的位置呢？我们采取的策略是：先查看"最长好后缀"是否也出现多次，如果是，直接选用这个"最长好后缀"作为上一次出现的位置；如果不是，则在"真好后缀"中选取"长度最长且也属于前缀"的那个"好后缀"。

举几个例子来具体说明上述的3个注意点。

1. 如果"ABCDEF"的"好后缀"是"EF"、"F"。那么"好后缀"的位置是 5（从 0 开始计算，取最后的 "F" 的值），观察发现"好后缀"只出现一次，则它们的上一次出现位置为-1，则后移位数为5-（-1）=6；
2. 如果"ABEFCDEF"的"好后缀"是"EF"、"F"。那么"好后缀"的位置是7，观察发现这两个"好后缀"都出现多次，则先查看"最长好后缀(EF)"是否出现多次，观察发现出现2次，所以选用"EF"为上一次出现的位置，"EF"上一次出现的位置为3，则后移位数为7-3=4；
3. 如果"EFABCDEF"的"好后缀"是"CDEF"、"DEF"、"EF"、"F"。那么"好后缀"的位置是7，观察发现有两个"好后缀"出现多次，则先查看"最长好后缀(CDEF)"是否出现多次，观察发现不是，则在"真好后缀(DEF、EF、F)"中选取"长度最长且也属于前缀"的那个"好后缀"，是"EF"，其上一次出现的位置是1，则后移位数为7-1=6。

回到当前步骤的这个例子。此时，所有的"好后缀"（MPLE、PLE、LE、E）之中，只有"E"在"EXAMPLE"还出现在头部，所以后移 6 - 0 = 6位。

（12）

![](https://61mon.com/images/illustrations/boyer_moore/12.png)

可以看到，"坏字符规则"只能移3位，"好后缀规则"可以移6位。所以，Boyer-Moore算法的基本思想是，每次后移这两个规则之中的较大值。

更巧妙的是，这两个规则的移动位数，只与搜索词有关，与原字符串无关。因此，可以预先计算生成《坏字符规则表》和《好后缀规则表》。使用时，只要查表比较一下就可以了。

（13）

![](https://61mon.com/images/illustrations/boyer_moore/13.png)

继续从尾部开始比较，"P"与"E"不匹配，因此"P"是"坏字符"。根据"坏字符规则"，后移 6 - 4 = 2位。

（14）

![](https://61mon.com/images/illustrations/boyer_moore/14.png)

从尾部开始逐位比较，发现全部匹配，于是搜索结束。如果还要继续查找（即找出全部匹配），则根据"好后缀规则"，后移 6 - 0 = 6位，即头部的"E"移到尾部的"E"的位置。

## 三：如何求坏字符表和好后缀表

### 3.1 坏字符算法（Bad Character）

当出现一个坏字符时， BM算法向右移动搜索词，让搜索词中最靠右的对应字符与坏字符相对，然后继续匹配。坏字符算法有两种情况。

**（1）搜索词中有对应的坏字符时，让搜索词中最靠右的对应字符与坏字符相对。**

![](https://61mon.com/images/illustrations/boyer_moore/15.png)

第二个$x$即为移动后的位置。

**（2）搜索词中不存在坏字符，那就直接右移整个搜索词长度这么大步数。**

![](https://61mon.com/images/illustrations/boyer_moore/16.png)

第二个$x$即为移动后的位置。

**代码实现：**

利用哈希空间换时间的思想，使用字符作为下标而不是位置作为下标。这样只需要遍历一遍即可。如果是纯8位字符也只需要256个空间大小，而且对于大模式，可能本身长度就超过了256，所以这样做是值得的（这也是为什么数据越大，BM算法越高效的原因之一）。

```c++
#define MAX_CHAR 256

/* 计算坏字符数组 */
void GetBC(string & p, int & m, int bc[])
{
    for (int i = 0; i < MAX_CHAR; i++)
        bc[i] = m;
    for (int i = 0; i < m; i++)
        bc[p[i]] = m - 1 - i;
}
```

###  3.2 好后缀算法（Good Suffix）

如果程序匹配了一个"好后缀"，那就把前面的"好后缀"移动到当前"好后缀"位置。这样，好后缀算法有三种情况。

**（1）搜索词中有子串和"最长好后缀"完全匹配，则将最靠右的那个完全匹配的子串移动到"最长好后缀"的位置继续进行匹配。**

![](https://61mon.com/images/illustrations/boyer_moore/17.png)

第二个$x$即为移动后的位置。

**（2）如果不存在和"最长好后缀"完全匹配的子串，则选取长度最长且也属于前缀的那个"真好后缀"。**

![](https://61mon.com/images/illustrations/boyer_moore/18.png)

第二个$x$即为移动后的位置。

**（3）如果完全不存在和"好后缀"匹配的子串，则右移整个搜索词。**

![](https://61mon.com/images/illustrations/boyer_moore/19.png)

第二个$x$即为移动后的位置。

**代码实现：**

```c++
#define MAX_LENGTH 1000  // 假定字符串长度不超过1000

/* 计算好后缀数组 */
void GetGS(string & p, int & m, int gs[])
{
    int suff[MAX_LENGTH];
    Suffixes(p, m, suff);

    // 第三种情况
    for (int i = 0; i < m; i++)
        gs[i] = m;

    // 第二种情况
    int j = 0;
    for (int i = m - 2; i >= 0; i--)
    {
        if (suff[i] == i + 1)
            for (; j < m - 1 - i; j++)
                gs[j] = m - 1 - i;
    }
  
    // 第一种情况
    for (int i = 0; i <= m - 2; i++)
        gs[m - 1 - suff[i]] = m - 1 - i;
}
```

看到这段代码，你可能会有很多的疑问，接下来我一步一步地讲解。

**疑问 1：Suffixes(p, m, suff) 的作用**

假定搜索词为p，长度为m，则数组`suff[i]`定义为：`p[0...i]`与p的最长公共后缀。举个例子：

|    i    |  0   |  1   |  2   |  3   |  4   |  5   |  6   |
| :-----: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|  p[i]   |  E   |  L   |  E   |  M   |  E   |  L   |  E   |
| suff[i] |  1   |  0   |  3   |  0   |  1   |  0   |  7   |

那该怎么去求解它呢？

![](https://61mon.com/images/illustrations/boyer_moore/20.png)

如上图，串p的长度为m，i是当前正准备计算`suff[]`值的那个位置。再设置两个变量，f和g。g代表以f为起始位置的字符匹配成功的最左边界，即"g = 最后一个成功匹配位置 - 1"。

当i处于f和g之间并且`suff[m - 1 - (f - i)] < i - g`，根据`suff[f]`，我们可以推断出`suff[i] = suff[m - 1 - (f - i)]`，图中椭圆的长度即为`suff[m - 1 - (f - i)]`。

```c++
/* 计算后缀数组 */
void Suffixes(string & p, int & m, int suff[])
{
    int f;
    int g = m - 1; // 这里 g 必须要大于 m - 2，因为必须保证 i 等于 m - 2 的时候执行 else 语句
    suff[m - 1] = m;
    for (int i = m - 2; i >= 0; i--)
    {
        if (i > g && suff[m - 1 - f + i] < i - g)
            suff[i] = suff[m - 1 - f + i];
        else
        {
            if (i < g)
                g = i;
            f = i;
            while (g >= 0 && p[g] == p[m - 1 - f + g])
                g--;
            suff[i] = f - g;
        }
    }
}
```

**疑问 2：第二种情况的代码怎么理解**

```c++
// 第二种情况
int j = 0;
for (int i = m - 2; i >= 0; i--)
{
    if (suff[i] == i + 1)
        for (; j < m - 1 - i; j++)
            gs[j] = m - 1 - i;
}
```

第二种情况下，即如果不存在和"最长好后缀"完全匹配的子串，则选取长度最长且也属于前缀的那个"真好后缀"。代码为何这么写？

![](https://61mon.com/images/illustrations/boyer_moore/21.png)

对于代码`if (suff[i] == i + 1)`，如上图，两个椭圆的长度都是`i + 1`，其所表示的字符都对应相等。正如下面浅黑色字所述，若有`if (suff[i] == i + 1)`，因为下标0前面没有字符，故[0, (m-1-i)-1]这部分的都只能找到"真好后缀"。

但如果是下面的情况呢？

![](https://61mon.com/images/illustrations/boyer_moore/22.png)

假设**[i, m-1]是"最长好后缀"**，但在整个串中只能找到"真好后缀"可以匹配，假设图中的两个椭圆互相匹配。那我们不是可以把区间[k, s]移到区间[j, m-1]的位置么？事实并非这么简单，我们观察`k-1`和`j-1`这两个位置的字符，它们是不等的，所以这样的移动毫无意义，因此第二种情况，只需利用`if (suff[i] == i + 1)`就可以找到所有的部分后缀匹配。

**疑问3：代码的顺序问题**

```c++
// 第三种情况
for (int i = 0; i < m; i++)
    gs[i] = m;

// 第二种情况
int j = 0;
for (int i = m - 2; i >= 0; i--)
{
    if (suff[i] == i + 1)
        for (; j < m - 1 - i; j++)
            gs[j] = m - 1 - i;
}

// 第一种情况
for (int i = 0; i <= m - 2; i++)
    gs[m - 1 - suff[i]] = m - 1 - i;
```

首先要明确我们的目标：获得最精准的`gs[]`。最精准也就是尽可能小，因为如果值稍大的话，搜索词在跳转的时候就可能遗漏一个完全匹配。

**for循环中的运行顺序**和**三种情况代码的顺序**之所以按如上代码那样安排，都是为了**获得最精准的`gs[]`**。下面我就从这两个方面解释。

- **for循环中的运行顺序**
  - 第三种情况，i从0到m。这很简单，不用解释；
  - 第二种情况，i从m-2到0。若$i_1<i_2$，则有$m-1-i_1>m-1-i_2$，因此需要从后往前算；
  - 第一种情况，i从0到m-2。若$suff[i_1]==suff[i_2]$，其中$i_1<i_2$，则有$m-1-i_1>m-1-i_2$。
- **三种情况代码的顺序**
  - 第三种情况放在第一个位置，这很好理解；
  - 第二种情况和第一种情况，我们可以举个同时满足这两个情况的例子。假设搜索词为`a....ba......ba`， 读 者自己手算一遍就懂了。


不知道读者有没有发现，第一和第二种情况的代码都没有`i = m - 1`。考虑第二种情况的代码，我们把`i = m - 1`代进去计算，发现程序会直接跳过。再看第一种情况的代码，我们把`i = m - 1`代进去计算，发现`m - 1 - suff[i]`为负，此时下标会溢出。

## 四：完整代码

```c++
/**
 * 
 * author : 刘毅（Limer）
 * date   : 2017-07-26
 * mode   : C++ 
 */

#include <iostream>
#include <string>
#include <algorithm>

#define MAX_CHAR 256
#define MAX_LENGTH 1000

using namespace std;

/* 计算坏字符数组 */
void GetBC(string & p, int & m, int bc[])
{
    for (int i = 0; i < MAX_CHAR; i++)
        bc[i] = m;
    for (int i = 0; i < m; i++)
        bc[p[i]] = m - 1 - i;
}

/* 计算后缀数组 */
void Suffixes(string & p, int & m, int suff[])
{
    int f;
    int g = m - 1; // 这里 g 必须要大于 m - 2，因为必须保证 i 等于 m - 2 的时候执行 else 语句
    suff[m - 1] = m;
    for (int i = m - 2; i >= 0; i--)
    {
        if (i > g && suff[m - 1 - f + i] < i - g)
            suff[i] = suff[m - 1 - f + i];
        else
        {
            if (i < g)
                g = i;
            f = i;
            while (g >= 0 && p[g] == p[m - 1 - f + g])
                g--;
            suff[i] = f - g;
        }
    }
}

/* 计算好后缀数组 */
void GetGS(string & p, int & m, int gs[])
{
    int suff[MAX_LENGTH];
    Suffixes(p, m, suff);

    // 第三种情况
    for (int i = 0; i < m; i++)
        gs[i] = m;

    // 第二种情况
    int j = 0;
    for (int i = m - 2; i >= 0; i--)
    {
        if (suff[i] == i + 1)
            for (; j < m - 1 - i; j++)
                gs[j] = m - 1 - i;
    }

    // 第一种情况
    for (int i = 0; i <= m - 2; i++)
        gs[m - 1 - suff[i]] = m - 1 - i;
}

void BM(string & s, int & n, string & p, int & m)
{
    int bc[MAX_LENGTH];
    int gs[MAX_LENGTH];
    GetBC(p, m, bc);
    GetGS(p, m, gs);

    int j = 0, i;
    while (j <= n - m)
    {
        for (i = m - 1; i >= 0 && p[i] == s[i + j]; i--)
            ;
        if (i < 0)
        {
            cout << "在下标 " << j << " 位置找到匹配\n";
            j += gs[0];
        }
        else
            j += max(bc[s[i + j]] - m + 1 + i, gs[i]);
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
        BM(s, n, p, m);
        cout << endl;
    }
  
    return 0;
}
```

数据测试如下：

![](https://61mon.com/images/illustrations/boyer_moore/23.png)
