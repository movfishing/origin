---
title: HNU Algorithm exp1
date: 2022-10-21 19:33:47
tags: experiments
categories: Algorithm 
---

---

本次实验完整代码(包含文件读写)已上传至[Github](https://github.com/movfishing/HNU-Algorithm/tree/main/Exp1)，欢迎借鉴，~~虽然写得很丑~~

---

# 经典案例

### 分治法查找最大最小值

* 基本思想：

    将一组数据A拆分为B、C两组，那么A的最大值与最小值可以表示为：

    A<sub>max</sub> = max{B<sub>max</sub>, C<sub>max</sub>}

    A<sub>min</sub> = min{B<sub>min</sub>, C<sub>min</sub>}

    基于此思想，通过递归即可将大问题变为小问题的求解。

* 算法思路：

    递归分组 ——> 求每个小分组的解 ——> 合并结果得出最终解

    在一个分组仅有两个元素时，那么最大值与最小值仅需通过一次比较得出; 仅有一个元素时，那么最大值与最小值均为该值。

* 举例：

    ![](/img/algorithmexp1/Q1.png)

* 算法实现：

```c++
vector<int> ans;

void findm(vector<int> &nums, int left, int right)
{
    if (!(right - left) || right - left == 1) //当仅有1个或两个元素时
    {
        if (nums[right] > nums[left])
        {
            if (nums[right] > ans[1])
                ans[1] = nums[right];
            if (nums[left] < ans[0])
                ans[0] = nums[left];
        }
        else
        {
            if (nums[left] > ans[1])
                ans[1] = nums[left];
            if (nums[right] < ans[0])
                ans[0] = nums[right];
        }
    }
    else
    {
        int mid = (left + right) >> 1;
        findm(nums, left, mid);
        findm(nums, mid + 1, right); //二分，递归
    }
}
```

* 构造测试用例：

```c++
srand(time(nullptr));
out << num << endl;//测试用例个数
while (num--)
{
    out << arrlen << endl;//数组长度
    for (int i = 0; i < arrlen; i++)
    {
        int a = (rand() << 15) + rand();
        out << a << endl;
    }
}
printf("操作成功！\n");
```

* 简单用例的测试结果：

    ![](/img/algorithmexp1/Ans1.png)

* 不同规模测试用例下的时间统计：

    图中横坐标为数组长度，纵坐标为时间消耗，均测试1000组数据，可以看到除去数组长度为1000的时候，趋近于一条直线。

    ![](/img/algorithmexp1/L1.png)

* 时间复杂度分析：

    每次将问题分为`2`个子问题，子问题规模为`n/2`，合并解只需要进行一次比较，可以在常数时间内完成。故递推式为：

    `T(n)=2T(n/2)+O(1)`

    根据主定理，k=2>m<sup>d</sup>=2^0=1,故时间复杂度为`O(n)`。

### 分治法实现合并排序

* 基本思想：

    将一组数据分为A、B两组，对A、B两组分别进行排序，然后合并A、B两组即可得到排好序的原数据组。通过递归的方式即可分解该问题。

* 算法思路：

    递归分组 ——> 对每个分组分别进行排序 ——> 合并结果得出最终解

    在这里，递归可以通过循环按以下步骤消除。

    (1)设置一个分组大小s，初值为1.

    (2)对于全部数据按s进行分组，相邻的组两两合并.

    (3)s=s*2.

    (4)对于合并后的数据，重复(2)(3)，直到s>数据个数.

* 举例：

    ![](/img/algorithmexp1/Q2.png)

* 算法实现：

```c++
void MergeCtrl(vector<int> *from, vector<int> *to, int left, int mid, int right)//将*from从 left到mid 与 mid+1到right 的数据进行合并，合并到*to
{
    int l = left;
    int r = mid + 1;
    int n = l;
    while (l <= mid && r <= right)
    {
        if ((*from)[l] > (*from)[r])
        {
            (*to)[n] = (*from)[r];
            n++;
            r++;
        }
        else
        {
            (*to)[n] = (*from)[l];
            n++;
            l++;
        }
    }
    if (l > mid)
    {
        while (r <= right)
        {
            (*to)[n] = (*from)[r];
            n++;
            r++;
        }
    }
    else
    {
        while (l <= mid)
        {
            (*to)[n] = (*from)[l];
            n++;
            l++;
        }
    }
}

void Merge(vector<int> *from, vector<int> *to, int s)//将全部数据进行分组
{
    int len = (*from).size();
    int i = 0;
    while (i <= len - (s << 1))//直到剩下的数据个数不足2*s个
    {
        MergeCtrl(from, to, i, i + s - 1, i + (s << 1) - 1);
        i += (s << 1);
    }
    if (i < len - s)//若剩下的数据大于s个
    {
        MergeCtrl(from, to, i, i + s - 1, len - 1);
    }
    else//若剩下的数据不足s个，则直接复制，因为对于s个数据，一定是已经排好序的(由s/2时合并而来)
    {
        for (int j = i; j < len; j++)
        {
            (*to)[j] = (*from)[j];
        }
    }
}

vector<int> Hsort(vector<int> &a)
{
    int s = 1;
    vector<int> b(a);
    while (s < a.size())
    {
        Merge(&a, &b, s);
        s = s << 1;
        Merge(&b, &a, s);//使用两个数组，形成类似滚动数组的效果
        s = s << 1;
    }
    return a;
}
```

* 构造测试用例：

```c++
srand(time(nullptr));
out << num << endl;//测试用例个数
while (num--)
{
    out << arrlen << endl;//数组长度
    for (int i = 0; i < arrlen; i++)
    {
        int a = (rand() << 15) + rand();
        out << a << endl;
    }
}
printf("操作成功！\n");
```

* 简单用例的测试结果：

    ![](/img/algorithmexp1/Ans2.png)

* 不同规模测试用例下的时间统计：

    图中横坐标为数组长度，纵坐标为时间消耗，均测试1000组数据，可以看到是一条斜率缓缓增大的曲线。

    ![](/img/algorithmexp1/L2.png)

* 时间复杂度分析：

    递归方法：

    每次将问题分为`2`个子问题，子问题规模为`n/2`，合并解需要遍历一遍分组，时间为O(n)。故递推式为：

    `T(n)=2T(n/2)+O(n)`

    根据主定理,k=2=m<sup>d</sup>=2^1=2,故时间复杂度为`O(nlogn)`。

    循环方法：

    循环进行`logn`次，每次循环进行的合并时间为O(n)，故时间复杂度为`O(nlogn)`。

# 实现题

### 字典序问题

* 基本思想：

    对于一个长度为len的符合条件的字母串，其序号由两部分组成：

    (1)长度为1~len-1的符合条件的字母串个数

    (2)长度为len，且在该字母串之前的字母串个数

* 算法思路：

    对于一个长度为n的字母串，其符合条件的字母串个数为 $C_{26}^{n}$ ，因为对于一个符合条件的字母串而言，拥有完全相同的字母的话，顺序是唯一的(升序)，所以个数就是其在26个字母中选n个不重复字母的组合数。

    而对于长度为len的字母串，就只需通过一个简单的循环计算，比如字母串`bdg`，在其之前的字母串个数就是：

    首字母为a且长度为3的字母串个数+首字母为c且长度为2的字母串个数+首字母为e~g且长度为1的字母串个数

* 举例：

    ![](/img/algorithmexp1/Q31.png)

    ![](/img/algorithmexp1/Q32.png)

* 算法实现：

```c++
int cnm(int n, int m)
{ // n个数里选m个
    int sum = 1;
    for (int i = 0; i < m; i++)
    {
        sum = sum * (n - i);
        sum = sum / (i + 1);
    }
    return sum;
}

int lensum1(string s)
{//计算长度为s.length()且在s之前的字符串个数
    int sum = 0;
    int len = s.length();
    int ch = 'a';
    for (int i = 0; i < len; i++)
    {
        for (int j = 0; j < s[i] - ch; j++)
        {
            sum += cnm('z' - ch - j, len - i - 1);
        }
        ch = s[i] + 1;
    }
    return sum;
}

int DictionaryNo(string s)
{
    int ans = 0;
    int sumabove = 1;
    int len = s.length();
    for (int i = 1; i < len; i++)//计算长度为1~len-1的符合条件的字母串个数
        sumabove += cnm(26, i);
    ans = lensum1(s) + sumabove;
    return ans;//得出字母串的序号
}
```

* 构造测试用例：

```c++
string Q3()
{
    int nums = (rand() % 6) + 1;
    string s;
    char ch = 'a';
    for (int i = 0; i < nums; i++)
        s += '\0';
    int prev = rand() % (26 - nums);
    int last = 26;
    for (int i = 0; i < nums; i++)
    {
        s[i] = ch + prev;
        last -= (prev + 1);
        prev = rand() % (last - nums + i + 1);
        ch = s[i] + 1;
    }
    return s;
}

out << num << endl;//测试用例个数
srand(unsigned(time(0)));
while (num--)
{
    out << Q3() << endl;
}
printf("操作成功！\n");
break;
```

* 简单用例的测试结果：

    ![](/img/algorithmexp1/Ans3.png)

* 不同规模测试用例下的时间统计：

    图中横坐标为字母串的长度，纵坐标为时间消耗，均测试1000组数据，可以看到由于各种因素影响以及误差，曲线接近于一个略向上的线，因为题只要求字母串长度最多为6，所以时间消耗差异并不是很大。

    ![](/img/algorithmexp1/L3.png)

* 时间复杂度分析：

    对于一个长度为n的字符串输入，仅需进行两次遍历，时间复杂度为O(n).

### 金币阵列问题

* 基本思想：

    一个阵列需要经过行变换和列交换变成目标阵列，那么我们可以先将所有行变换处理完，再通过列交换得到结果。

* 算法思路：

    由于行是不能交换的，所以原阵列的每一行与目标阵列的对应行应该存在以下关系之一：

    (1)行中1与0的数目相等

    (2)原阵列1的数目等于目标阵列0的数目，这时需要进行一次翻转就可以变成情况1

    对每一行进行处理之后，再进行列交换。由于交换的时间消耗较高，我们可以通过简单的判断来完成。对于目标阵列的每一列，在原阵列中寻找是否有未被标记的相同列，若不存在，则可以直接返回-1；若存在，则将原阵列中的该列标记，且交换次数+1。

    由于是两两交换，故交换次数需要除以2；且由于存在相同列的列号相同无需交换的情况，在这时需要将交换次数-1。

    最后将交换次数加上翻转次数就是需要操作的最少次数。

* 举例：

    ![](/img/algorithmexp1/Q4.png)

* 算法实现：

```c++
int Coins(vector<vector<int>> &a, vector<vector<int>> &b, int x, int y)
{
    vector<int> is_comp(y, 0); //用于标记a中每一列是否有与之对应的列
    vector<vector<int>> temp;
    temp.assign(a.begin(), a.end());
    int turncount = 0;
    int switchcount = 0;
    for (int i = 0; i < x; i++)
    {
        int count_0 = 0;
        int count_1 = 0;
        for (int j = 0; j < y; j++)
        {
            if (a[i][j])
                count_1++;
            else
                count_0++;
            if (b[i][j])
                count_1--;
            else
                count_0--;
        }
        if (count_0 == 0 && count_1 == 0)
            ;
        else if (count_0 * (-1) == count_1)
        {
            turncount++;
            for (int k = 0; k < y; k++)
                temp[i][k] ^= 1;
        }
        else
        {
            return -1;
        }
    }

    int flag = 0;
    for (int p = 0; p < y; p++) //然后再对于目标的每一列，查找是否有与之相同的列
    {
        int compline = 0;
        int outflag = 0;
        int has_comp = 0;
        for (int q = 0; q < y; q++)
        {
            flag = 0;
            for (int r = 0; r < x; r++)
            {
                if (b[r][p] != temp[r][q])
                    break;
                if (r == x - 1)
                {
                    flag = 1;
                    outflag = 1;
                    if (is_comp[q] == 0 && !has_comp)
                    {
                        is_comp[q] = 1;
                        has_comp = 1;
                    }
                }
            }

            if (flag && p == q)//
            {
                switchcount--;
            }
        }
        if (outflag)
            switchcount++;
        else
            return -1;
    }
    turncount += switchcount / 2 + switchcount % 2;
    for (auto i : is_comp)
    {
        if (i == 0)
            return -1;
    }
    return turncount;
}
```

* 构造测试用例：

```c++
out << num << endl;//测试用例个数
num /= 2;
srand(unsigned(time(0)));
while (num--)
{
    x = rand() % 5 + 1;//矩阵行数
    y = rand() % 5 + 1;//矩阵列数
    out << x << " ";
    out << y << endl;
    for (int i = 0; i < x; i++)
    {
        for (int j = 0; j < y; j++)
        {
            temp = rand() % 2;
            out << temp << " ";
        }
        out << endl;
    }
    for (int i = 0; i < x; i++)
    {
        for (int j = 0; j < y; j++)
        {
            temp = rand() % 2;
            out << temp << " ";
        }
        out << endl;
    }
}
printf("操作成功！\n");
```

* 简单用例的测试结果：

    ![](/img/algorithmexp1/Ans4.png)

* 不同规模测试用例下的时间统计：

    表中横坐标为矩阵规模(n*n)，纵坐标为时间消耗，均测试1000组数据，可以看出其为一个指数型曲线。

    ![](/img/algorithmexp1/L4.png)

* 时间复杂度分析：

    对于一个规模为n * m的矩阵输入，进行了一次行遍历，然后在同一行的情况下对两个矩阵进行列遍历，采用了三层循环，时间复杂度为O(n * m^2).