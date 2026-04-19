# BM算法

### 功能

BM算法是一种以 $O(N)$ 的时间复杂度完成主串与模式串的匹配的算法，可以认为是KMP的进阶算法。

> 较于传统的BF算法，BM算法的优化方向与KMP算法一致

### 定义

##### <font color="brown">坏字符法则(Rule 1)</font>

失配时，记主串失配字符为坏字符，模式串向右移动位数计算：
模式串失配字符位置-模式串中最右侧坏字符位置(若不含坏字符，则记为-1)

> 实际上就是把坏字符对齐或者避开。

##### <font color="brown">好后缀法则(Rule 2)</font>

已实现配对的字符串均称为好后缀，失配时，模式串向右移动位数计算：
好后缀在模式串中的位置 - 好后缀在模式串上一次出现的位置

> 实际上就是把某个好后缀对齐。

### 流程

0.将模式串与主串左对齐，指针分别位于模式串末尾及模式串末尾对应主串位置。1.向前逐个比较字符。2.失配时，主串指针后移位数为Rule 1与Rule 2中的最大值，模式串指针回到末尾。3.重复上述流程至模式串指针移至模式串开头，表示匹配成功。

### 代码实现

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define ASIZE 256

// 生成坏字符表
void generate_bc(const char *pattern, int bc[])
{
    int m = strlen(pattern);
    for (int i = 0; i < ASIZE; i++)
        bc[i] = -1;
    for (int i = 0; i < m; i++)
        bc[(unsigned char)pattern[i]] = i;
}

// 生成好后缀 suffix 和 prefix 数组
void generate_gs(const char *pattern, int suffix[], char prefix[], int m)
{
    for (int i = 0; i < m; i++)
    {
        suffix[i] = -1;
        prefix[i] = 0;
    }

    for (int i = 0; i < m - 1; i++)
    {
        int j = i, k = 0;
        while (j >= 0 && pattern[j] == pattern[m - 1 - k])
        {
            j--;
            k++;
            suffix[k] = j + 1;
        }
        if (j == -1)
            prefix[k] = 1;
    }
}

// 计算好后缀应移动的步数
int get_gs_move(int suffix[], char prefix[], int j, int m)
{
    int len = m - j - 1;
    if (suffix[len] != -1)
        return j + 1 - suffix[len];

    for (int k = j + 2; k < m; k++)
    {
        if (prefix[m - k])
            return k;
    }
    return m;
}

// BM 匹配主函数
int bm_match(const char *text, const char *pattern)
{
    int n = strlen(text);
    int m = strlen(pattern);
    if (m == 0 || n < m)
        return -1;

    int bc[ASIZE];
    int *suffix = (int *)malloc(m * sizeof(int));
    char *prefix = (char *)malloc(m * sizeof(char));

    generate_bc(pattern, bc);
    generate_gs(pattern, suffix, prefix, m);

    int i = 0;
    while (i <= n - m)
    {
        int j = m - 1;
        while (j >= 0 && pattern[j] == text[i + j])
            j--;

        if (j < 0)
        {
            free(suffix);
            free(prefix);
            return i;
        }

        int move_bc = j - bc[(unsigned char)text[i + j]];
        int move_gs = (j == m - 1) ? 0 : get_gs_move(suffix, prefix, j, m);
        i += (move_bc > move_gs) ? move_bc : move_gs;
    }

    free(suffix);
    free(prefix);
    return -1;
}

int main()
{
    char text[] = "HERE IS A SIMPLE EXAMPLE";
    char pattern[] = "MPLE";
    printf("%d\n", bm_match(text, pattern));
    return 0;
}
```
