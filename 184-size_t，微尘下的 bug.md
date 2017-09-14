这个问题是我在写KMP时遇到的，考虑到并不是每位读者都了解这个算法，这里我简化下问题，请看下面的代码，并猜测其输出什么：
```c++
string str = "123456";

if (-1 < str.size())
	cout << "win\n";
else
	cout << "lose\n";
```
你的答案是win，是么？那么很遗憾的告诉你，NO，答案是；lose！！！

为何？其实很简单的问题，类型不一致。

-1默认为`int`，`size()`返回类型为`size_t`即`unsigned int`。


<!--more-->


```c++
size_t x = 1;
int y = -1;
cout << x + y << endl;               // 0
cout << typeid(x+y).name() << endl;  // unsigned int
```

两种类型进行操作，int类型的-1会被自动转为`unsigned int`，即：

```c++
  1          (unsigned int) + -1         (int)
  0000...0001(unsigned int) + 1111...1111(int)
= 0000...0001(unsigned int) + 1111...1111(unsigned int)
= 0000...0000(unsigned int)
```

显而易见，int类型的-1转为unsigned int后，会变成一个非常大的正数。

就是因为这个问题，让我纠结了一个多小时，下面贴下代码纪念下：

```c++
int KMP(string S, string P, int next[])
{
	GetNext(P, next);

	int i = 0; 
	int j = 0;  
	
	while (i <  S.size() && j < P.size())  // 除了上面提及的问题，这里的 size() 还有一个问题，你知道么？
	{
		if (j == -1 || S[i] == P[j])  
		{
			i++;
			j++;
		}
		else
			j = next[j];  
	}

	if (j == P.size())  
		return i - j;

	return -1;
}
```
**所以，以后写代码一定要注意类型转换，因为一旦出现问题，很难发现它的bug。**