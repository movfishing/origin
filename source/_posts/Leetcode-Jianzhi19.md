---
title: Leetcode 剑指Offerii-19 正则表达式匹配
date: 2023-03-10 00:25:28
tags: Algorithm
categories: Leetcode
---

# 剑指 Offer 19. 正则表达式匹配

### 题目描述

请实现一个函数用来匹配包含`.`和`*`的正则表达式。模式中的字符`.`表示任意一个字符，而`*`表示它前面的字符可以出现任意次（含0次）。在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串`aaa`与模式`a.a`和`ab*ac*a`匹配，但与`aa.a`和`ab*a`均不匹配。

* `s`可能为空，且只包含从`a-z`的小写字母。

* `p`可能为空，且只包含从`a-z`的小写字母以及字符`.`和`*`，无连续的`*`。

### 算法思想

对于两条表达式，**倒序**进行逐个字符比对，对于位置pos_s与pos_p上的两个字符，只有两种情况：匹配或不匹配。

若匹配，则继续往前比对；

若不匹配，看位置pos_p上的字符是否为`*`，若是，则进行额外的搜索：

* 取0个*之前的字符，即`s[pos_s]`与`p[pos_p-2]`继续进行比对；

* 取1个*之前的字符，即`s[pos_s-1]`与`p[pos_p-1]`继续进行比对；
  
* 取2个......
  
* 例：
```
aabc

aab*c

pos_s=2 && pos_p=3时，先取0个b，即pos_s=2 && pos_p=1进行比对，b!=a，return false;
再取1个b，即pos_s=2 && pos_p=1进行比对，b==b，继续向前搜索；
```

其实就是 枚举+DFS+剪枝 的思想，对于匹配到`*`的情况，就取每一种情况(取不同数量的*前字符)进行深度优先遍历，若最后能返回true，又或者能够搜索到字符串的首端，则说明能匹配成功。对于匹配错误的情况，就直接return false。

### 代码实现

```c++
class Solution {
public:
    bool isMatch(string s, string p) {
        int pos_s=s.length()-1,pos_p=p.length()-1;
        return dfs(s,p,pos_s,pos_p);
    }

    bool dfs(std::string &s, std::string &p, int pos_s, int pos_p)
    {
        if (pos_s >= 0 && pos_p < 0)//若p已经抵达首端而s还没有，则不可能匹配成功
            return false;
        else if (pos_p < 0 && pos_s < 0)//若两者都抵达首端，则匹配成功
            return true;
        else if (pos_s < 0 && pos_p >= 0)//若s抵达首端而p还没有，则有可能是p仅剩有带*的字符没遍历完
        {
            if (p[pos_p] == '*')
                return dfs(s, p, pos_s, pos_p - 2);
            else
                return false;
        }
        if (pos_s == 0 && pos_p == 0 && (s[pos_s] == p[pos_p] || p[pos_p] == '.'))//若刚好在首端且该位置字符可以匹配，则匹配成功
            return true;
        if (s[pos_s] == p[pos_p] || p[pos_p] == '.')//若字符成功匹配，则继续向前比对
            return dfs(s, p, pos_s - 1, pos_p - 1);
        else
        {
            if (p[pos_p] == '*')//进行dfs
            {
                bool temp = false;
                if (pos_p > 1)//取0个相应字符
                    temp = temp || dfs(s, p, pos_s, pos_p - 2);//用或运算符，保证只需要有一种情况能抵达首端匹配成功即可
                if (p[pos_p - 1] == '.')//取1~最多个数的.
                {
                    while (pos_s >= 0)
                    {
                        temp = temp || dfs(s, p, pos_s, pos_p - 1);
                        pos_s--;
                    }
                }
                else//取1~最多个数的相应字符
                {
                    while (pos_s >= 0 && s[pos_s] == p[pos_p - 1])
                    {
                        temp = temp || dfs(s, p, pos_s, pos_p - 1);
                        pos_s--;
                    }
                }
                return temp;
            }
        }
        return false;
    }
};
```