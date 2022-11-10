---
title: Algorithm exp2
date: 2022-11-10 15:38:16
tags: experiments
categories: Algorithm
---

---

本次实验完整代码(包含文件读写)已上传至[Github](https://github.com/movfishing/HNU-Algorithm/tree/main/Exp2)，欢迎借鉴

---

# 经典案例

### 单元最短路径问题

* 基本思想：

    使用Dijkstra算法解决。

* 算法思路：

    开始时，创建一个集合V，用于存放已经找到最短路径的点，很明显初始只有源点。创建一个数组Dist，用于存放当前计算出的每个点到源点的距离。每次循环选取一个未在V中且Dist最小的点prev加入集合V，然后将所有未在V中，且可以与刚刚存入V中的点prev直接连接的点，更新其Dist[i]为min{Dist[i],side(prev，i)}。当所有点均加入V后，Dist即为所有点到源点的最短路径长度。要得到最短路径的话，则需要额外维护一个数组Prevs，Prev[i]存放的是其最短路径的前一个点，在每次更新Dist[i]时同时更新Prevs[i]即可。

* 举例：

    ![](/img/algorithmexp2/Q1.png)

* 算法实现：

```c++
struct Graph
{
    int sign;
    vector<int> sides;
}; //初始化注意：无边则边长为INFINITY，为自身时则边长为0.

vector<Graph> G;

vector<int> Dist; //初始化为G大小

vector<int> Prevs; //初始化为G大小,均为-1

int side_length(int point1, int point2)
{
    return G[point1].sides[point2];
}

void Dijkstra(int src)
{
    vector<int> V(G.size(), 0); //初始化为G大小
    int prev = src;
    for (int i = 0; i < G.size(); i++)
    {
        Dist[i] = side_length(src, i);
        if (Dist[i] != INFINITY)
            Prevs[i] = src;
        V[i] = 0;
    }

    V[src] = 1;

    for (int p = 0; p < G.size(); p++)
    {
        int temp = INFINITY;
        for (int i = 0; i < G.size(); i++)
        {
            if ((!V[i]) && (Dist[i] < temp))
            {
                prev = i;
                temp = Dist[i];
            }
        }
        V[prev] = 1;
        for (int i = 0; i < G.size(); i++)
        {
            if ((!V[i]) && (side_length(prev, i) != INFINITY))
            {
                int newDist = side_length(prev, i) + Dist[prev];
                if (newDist < Dist[i])
                {
                    Dist[i] = newDist;
                    Prevs[i] = prev;
                }
            }
        }
    }
}
```

* 构造测试用例：

```c++
srand(time(nullptr));
out << num << endl;
while (num--)
{
    out << arrlen << endl;
    out << (rand() % arrlen) << endl;
    int **arr = new int *[arrlen];
    for (int i = 0; i < arrlen; i++)
    {
        arr[i] = new int[arrlen];
    }
    for (int i = 0; i < arrlen; i++)
    {
        arr[i][i] = 0;
        for (int j = 0; j < i; j++)
        {
            int seed = rand() % 10;
            if (seed > 6)
                arr[i][j] = -1;
            else
                arr[i][j] = (rand() % 20) + 1;
        }
    }
    for (int j = 0; j < arrlen; j++)
    {
        for (int i = 0; i < j; i++)
        {
            arr[i][j] = arr[j][i];
        }
    }
    for (int i = 0; i < arrlen; i++)
    {
        out << i;
        for (int j = 0; j < arrlen; j++)
        {
            out << " " << arr[i][j];
        }
        out << endl;
    }
    for (int i = 0; i < arrlen; i++)
    {
        delete[] arr[i];
    }
    delete[] arr;
}
printf("操作成功！\n");
```

* 简单用例的测试结果：

    ![](/img/algorithmexp2/Ans1.png)

* 不同规模测试用例下的时间统计：

    图中横坐标为图的节点个数，纵坐标为时间消耗，均测试1000组数据，可以看到趋近于一条乘方型曲线。

    ![](/img/algorithmexp2/Q1list.png)

* 时间复杂度分析：

    对于节点数为V，边数为E的图，每次循环需要找出距离最近的点，并更新Dist，直到所有点被找到，需要`|V|*(2*|V|)`时间,故时间复杂度为`O(|V|^2)`。

### 回溯法解决0-1背包问题

* 基本思想：

    回溯法，即枚举，遍历所有情况的分支，对于明显不满足条件的分支排除。

* 算法思路：

    (1)先使用贪心找到一个符合条件的组合

    (2)弹出组合最后选择的物品，在该物品之后使用贪心选择物品加入背包，若弹出的物品是最后一个物品，则需额外弹出一个物品(说明该分支已经遍历完毕)

    (3)重复(2)直到组合中只剩最后一个物品或为空。记录所有组合中价值的最大值以及对应的组合。

* 举例：

    ![](/img/algorithmexp2/Q2.png)

* 算法实现：

```c++
stack<int> ans;

int value = 0;

void packet(vector<int> &w, vector<int> &v, int c)
{
    stack<int> temp;
    int now_c = c;
    int t_value = 0;
    for (int i = 0; i < w.size(); i++)
    {
        if (now_c >= w[i])
        {
            now_c -= w[i];
            temp.push(i);
            t_value += v[i];
        }
    }
    if (t_value > value)
    {
        value = t_value;
        ans = temp;
    }
    while (1)
    {
        if (temp.size() == 0)
            break;
        int prev = temp.top();
        temp.pop();
        now_c += w[prev];
        t_value -= v[prev];
        if (prev == w.size() - 1)
        {
            if (temp.size() == 0)
                break;
            prev = temp.top();
            temp.pop();
            now_c += w[prev];
            t_value -= v[prev];
        }
        for (int i = prev + 1; i < w.size(); i++)
        {
            if (now_c >= w[i])
            {
                now_c -= w[i];
                temp.push(i);
                t_value += v[i];
            }
        }
        if (t_value > value)
        {
            value = t_value;
            ans = temp;
        }
        if (temp.size() == 0)
            break;
        else if (temp.top() == w.size() - 1 && temp.size() == 1)
            break;
    }
}
```

* 构造测试用例：

```c++
srand(time(nullptr));
out << num << endl;
while (num--)
{
    int things = rand() % 30;
    out << things << endl;
    int c = rand();
    out << c << endl;
    int w, v;
    for (int i = 0; i < things; i++)
    {
        w = rand() % 10000;
        v = rand() % 10000;
        out << w << " " << v << endl;
    }
}
printf("操作成功！\n");
```

* 简单用例的测试结果：

    ![](/img/algorithmexp2/Ans2.png)

* 不同规模测试用例下的时间统计：

    图中横坐标为物品个数，纵坐标为时间消耗，均测试1000组数据，可以看到是一条指数型曲线。

    ![](/img/algorithmexp2/Q2list.png)

* 时间复杂度分析：

    回溯法会遍历所有可能的分支，每一个物品有1或0两种状态，所以时间复杂度应为`O(2^n)`。

# 实现题

### 字符距离问题

* 基本思想：

    对于字符串的一位，仅有两种可能：与另一字符串的一位配对，或与空格配对。那么对于长度为m的字符串A与长度为n的字符串B，k为字符与空格的距离，其最短距离minDist(m,n)=min(minDist(m-1,n)+k,minDist(m,n-1)+k,minDist(m-1,n-1)+|A[m-1]-B[n-1]|))。

* 算法思路：

    对于递推式minDist(m,n)=min(minDist(m-1,n)+k,minDist(m,n-1)+k,minDist(m-1,n-1)+|A[m-1]-B[n-1]|))，当m为0时，minDist(0,n)=minDist(0,n-1)+k；当n为0时，minDist(m,0)=minDist(m-1,0)+k；m,n均为0时，minDist(0,0)=0。进行循环即可。

* 举例：

    ![](/img/algorithmexp2/Q3.png)

* 算法实现：

```c++
vector<vector<int>> Dist;

void minDist(string A, string B, int k)
{
    Dist[0][0] = 0;
    for (int i = 1; i < Dist.size(); i++)
    {
        Dist[i][0] = Dist[i - 1][0] + k;
    }
    for (int j = 1; j < Dist[0].size(); j++)
    {
        Dist[0][j] = Dist[0][j - 1] + k;
    }
    for (int i = 1; i < Dist.size(); i++)
    {
        for (int j = 1; j < Dist[0].size(); j++)
        {
            Dist[i][j] = min(min(Dist[i - 1][j] + k, Dist[i][j - 1] + k), Dist[i - 1][j - 1] + abs(A[i - 1] - B[j - 1]));
        }
    }
}
```

* 构造测试用例：

```c++
string Q3()
{
    string s;
    int len = rand() % 1000;
    char ch = 'a';
    while (len--)
    {
        int offset = rand() % 26;
        s += (ch + offset);
    }
    return s;
}

out << num << endl;
srand(unsigned(time(0)));
while (num--)
{
    out << Q3() << endl;
    out << Q3() << endl;
    int k = rand() % 100;
    out << k << endl;
}
printf("操作成功！\n");
```

* 简单用例的测试结果：

    ![](/img/algorithmexp2/Ans2.png)

* 不同规模测试用例下的时间统计：

    图中横坐标为字母串的长度，纵坐标为时间消耗，均测试1000组数据，可以看到大致为一个乘方型曲线。

    ![](/img/algorithmexp2/Q3list.png)

* 时间复杂度分析：

    对于两个长度为n与m的字符串输入，仅需对二维数组Dist[n][m]进行遍历，时间复杂度为O(nm).

### 单调递增最长子序列问题

* 基本思想：

    将求解arr[0:n-1]的单调递增最长子序列转化为求解arr[0:n-2]与判断arr[n-1]能否加入子序列两个子问题。若arr[n-1]大于arr[0:n-2]的单调递增最长子序列的最后一个值，则将arr[n-1]加入子序列，反之则子序列不变。

* 算法思路：

    由于普通动态规划方法时间复杂度为`O(n^2)`，所以我们需要额外维护一个数组`tail`，以优化时间复杂度。先将数组第一个元素加入tail，随后从数组第二个元素开始遍历，若该元素大于tail的最后一个元素，则将其加入tail；否则在tail中搜索第一个大于该元素的值，并替换之。最后tail的长度就是最长子序列的长度。若想要得到最长子序列，则需额外维护两个数组`sign`和`update`，sign用于存放子序列的值，update用于存放tail进行替换值的位置，在tail进行替换时，update对应的位置变为1。在每次tail新加入元素时，将sign中位置区间为[update的末尾开始(update的末尾必须为1，否则不更新)，往前一直到连续的最后一个1为止]更新为tail中对应位置的值。最后sign就是单调递增最长子序列。

* 举例：

    ![](/img/algorithmexp2/Q4.png)

* 算法实现：

```c++
vector<int> maxOrder(vector<int> &arr)
{
    vector<int> temp(arr.size(), 1);
    vector<int> tail(arr.size(), 0);
    vector<int> sign(arr.size(), 0);
    vector<int> update(arr.size(), 0);
    tail[0] = arr[0];
    sign[0] = 0;
    int end = 0;
    for (int i = 1; i < arr.size(); i++)
    {
        if (arr[i] > tail[end])
        {
            if (update[end] == 1)
            {
                for (int j = end; j >= 0; j--)
                {
                    if (update[j])
                    {
                        sign[j] = tail[j];
                        update[j] = 0;
                    }
                    else
                    {
                        break;
                    }
                }
            }
            tail[++end] = arr[i];
            sign[end] = arr[i];
        }
        else
        { //找到tail中比arr[i]小的最大的数
            int left = 0;
            int right = end;
            while (left < right)
            {
                int mid = left + ((right - left) >> 1);
                if (tail[mid] < arr[i])
                {
                    left = mid + 1;
                }
                else
                {
                    right = mid;
                }
            }
            tail[left] = arr[i];
            update[left] = 1;
        }
    }
    return sign;
}
```

* 构造测试用例：

```c++
srand(time(nullptr));
        out << num << endl;
        while (num--)
        {
            int length = rand();
            out << length << endl;
            for (int i = 0; i < length; i++)
            {
                int a = (rand() << 15) + rand();
                out << a << endl;
            }
        }
        printf("操作成功！\n");
```

* 简单用例的测试结果：

    ![](/img/algorithmexp2/Ans4.png)

* 不同规模测试用例下的时间统计：

    表中横坐标为数组长度n，纵坐标为时间消耗，均测试1000组数据，可以看出其为一个nlogn的曲线。

    ![](/img/algorithmexp2/Q4list.png)

* 时间复杂度分析：

    对于一个长度为n的数组输入，需要进行一次遍历，在遍历中执行了查找操作，查找操作的时间复杂度为O(logn)，所以整体算法的时间复杂度为O(nlogn)。