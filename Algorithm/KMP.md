# KMP算法

### 功能

以 $O(N+M)$ 的时间复杂度完成主串与模式串的匹配

### 暴力算法的缺陷

普通的暴力做法将主串每个字母作为起点与模式串逐字符比较，时间复杂度 $O(N*M)$ 。
首先，模式串的每个位置的信息肯定都会被访问，想要优化时间复杂度，就要从减少主串中作为起点的字母数量。

### 实现

要明确的是如果我们要降低时间复杂度的话，选择预处理模式串是更好的选择（通常来说比较短）。

##### 观察配对过程

模式串： "<font color="brown">$ABCBA$</font>$BEA$"
主串: "<font color="brown">$\cdots ABCBA$</font>$CDE\cdots$
匹配进行到这里时失效，模式串的指针是要回到第一个元素的，考虑主串的指针要到哪里
显然，主串的指针可以直接指向红色部分末尾的 $A$ 。

模式串： "<font color="brown">$ABCBAB$</font>$EA$"
主串: "<font color="brown">$\cdots ABCBAB$</font>$DE\cdots$
这里，主串的指针可以直接指向红色部分末尾的 $AB$ 中的 $B$ ，而模式串也可以直接指向开头 $AB$ 中的 $B$ 。

##### 理论

如果模式串的已匹配部分真前缀与真后缀交集非空，我们可以让模式串指针指向真前缀末尾，主串（这时与模式串部分序列相同）指针指向真后缀末尾，这样我们可以保证模式串指针之前元素保持配对状态。

##### 具体

我们用 $next[i]$ 表示模式串在第 $i$ 个位置失配时，指针需要移动到的位置。
$$next[i]=\max (n.length()),n\in P\cap S$$
其中$P,S$分别是由真前缀，真后缀组成的集合。

$next$ 数组求法
特别地, 让 $next[0]=-1,next[1]=0$ ，
通过将模式串 $p$ 错位进行自身匹配，实现前缀与后缀的比较，这里 $i$ 表示此次循环是求解 $P.substr(0, i+1)$ 的真公共前后缀。
匹配过程中利用已求出的 $next$ 数组优化时间复杂度（就是嵌套了一个KMP算法）

```
for (int i = 1, j = 0; i < plen; ++i)
{
    while (j && p[i] != p[j]) j = next[j];
    if (p[i] == p[j]) j++;
    next[i+1] = j;
}
```

##### 常数优化

面对这种 $ababa$ 这种重叠的相同字串（ $aba$ 与 $aba$ ）,
常规的 $next$ 数组，面临以下问题：
模式串： "<font color="brown">$ABAB$</font>$A$"
主串: "<font color="brown">$\cdots ABAB$</font>$C\cdots$
跳转后
模式串:$\quad \quad$"<font color="brown">$AB$</font>$ABA$"
主串: $\cdots AB$<font color="brown">$AB$</font>$C\cdots$
仍无法匹配，仍须跳转，所以这里可以进行路径优化。

```
for (int i = 1, j = 0; i < plen; ++i)
{
    while (j && p[i] != p[j]) j = next[j];
    if (p[i] == p[j]) j++;
    bool b = p[i] == p[j], c = p[i + 1] == p[j + 1];
    if (b) next[i+1] = next[(j++)+1];
    if (!b || !c) next[i+1] = j;
}
```

### 完整模板、

```
const int MAXN = 1e6 + 5;
int pmt[MAXN];
void get_pmt(const string& s) {
    for (int i = 1, j = 0; i < s.length(); ++i) {
        while (j && s[i] != s[j]) j = pmt[j - 1];
        if (s[i] == s[j]) j++;
        pmt[i] = j;
    }
}
void kmp(const string& s, const string& p) {
    for (int i = 0, j = 0; i < s.length(); ++i) {
        while (j && s[i] != p[j]) j = pmt[j - 1];
        if (s[i] == p[j]) j++;
        if (j == p.length()) {
            //cout << i - j + 2 << '\n'; // 1-index，所以是+2
            j = pmt[j - 1];
        }
    }
}
```
