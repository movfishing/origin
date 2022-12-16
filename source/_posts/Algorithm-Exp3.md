---
title: Algorithm Exp3
date: 2022-12-15 19:33:06
tags: experiments
categories: Algorithm
---

---

本次实验完整代码(包含文件读写)已上传至[Github](https://github.com/movfishing/HNU-Algorithm/tree/main/Exp3)，欢迎借鉴

---

# 经典案例

### 独立任务最优调度问题

* 基本思想：

    易看出此问题有最优子结构性质，可使用动态规划法解决。

* 算法思路：

    建立两个时间队列，一个是A处理的作业的队列，一个是B处理的作业的队列，那么总体耗时就是两者之间最长的那个。对于每一个任务，只有交给A处理或交给B处理两种情况，分析两者哪个总体耗时最少，交给总体耗时少的那方。

    有递推式，对于第n个任务，A耗时ta，B耗时tb，时间队列Ta与Tb，总体耗时为W(n)=min(max(Ta+ta,tb),max(Ta,Tb+tb))

* 算法实现：

```c++
int BestSchedule(vector<int> &a, vector<int> &b, int n)
{
    int timea = 0;
    int timeb = 0;
    for (int i = 0; i < n; i++)
    {
        if (max(timea + a[i], timeb) > max(timea, timeb + b[i]))
        {
            timeb += b[i];
        }
        else
        {
            timea += a[i];
        }
    }
    return max(timea, timeb);
}
```

* 构造测试用例：

```c++
srand(time(nullptr));
out << num << endl;
while (num--)
{
    int len = rand() % 1000 + 1;
    out << len << endl;
    for (int i = 0; i < len; i++)
    {
        int a = rand() % 10 + 1;
        out << a << " ";
    }
    out << endl;
    for (int i = 0; i < len; i++)
    {
        int b = rand() % 10 + 1;
        out << b << " ";
    }
    out << endl;
}
printf("操作成功！\n");
```

* 简单用例的测试结果：

    ![](/img/algorithmexp3/Ans1.png)

* 不同规模测试用例下的时间统计：

    图中横坐标为任务个数，纵坐标为时间消耗，均测试1000组数据，可以看到趋近于一条直线。

    ![](/img/algorithmexp3/Q1list.png)

* 时间复杂度分析：

    对于任务个数为n的输入，只需要遍历一次任务，所以时间复杂度为`O(n)`。

### 石子合并问题

* 基本思想：

    由于要得分最多或最少，那么就只需要保证每次得分都最多或最少即可，类似于贪心算法。

* 算法思路：

    进行n次遍历，每次遍历找到得分最高或最少的组合，将两者合并，即将两者之和存入其中一个元素位置，并删除另一个元素，这里需要使用到std::vector的erase函数。同时将该次合并的值加到得分中。

* 算法实现：

```c++
vector<int> StoneMerge(vector<int> &a, int n)
{
    vector<int> ans;
    if (n == 1)
    {
        ans.push_back(a[0]);
        ans.push_back(a[0]);
        return ans;
    }
    vector<int> minHeap;
    minHeap.assign(a.begin(), a.begin() + n);
    vector<int> maxHeap;
    maxHeap.assign(a.begin(), a.begin() + n);
    int max = 0;
    int min = 0;
    int flag = 0;
    while (minHeap.size() != 2)
    {
        int tempmin = INT_MAX;
        auto tempit = minHeap.begin();
        for (auto it = minHeap.begin() + 1; it != minHeap.end() - 1; it++)
        {
            if (*(it - 1) > *(it + 1))
            {
                if (*(it + 1) < tempmin)
                {
                    tempmin = *(it + 1);
                    tempit = it + 1;
                    flag = 0;
                }
            }
            else
            {
                if (*(it - 1) < tempmin)
                {
                    tempmin = *(it - 1);
                    tempit = it - 1;
                    flag = 1;
                }
            }
        }
        if (flag)
        {
            *tempit = *(tempit + 1) + *tempit;
            tempmin = *tempit;
            minHeap.erase(tempit + 1);
        }
        else
        {
            *tempit = *(tempit - 1) + *tempit;
            tempmin = *tempit;
            minHeap.erase(tempit - 1);
        }
        min += tempmin;
    }
    min += minHeap[0] + minHeap[1];

    while (maxHeap.size() != 2)
    {
        int tempmax = 0;
        auto tempit = maxHeap.begin();
        for (auto it = maxHeap.begin() + 1; it != maxHeap.end() - 1; it++)
        {
            if (*(it - 1) < *(it + 1))
            {
                if (*(it + 1) > tempmax)
                {
                    tempmax = *(it + 1);
                    tempit = it + 1;
                    flag = 0;
                }
            }
            else
            {
                if (*(it - 1) > tempmax)
                {
                    tempmax = *(it - 1);
                    tempit = it - 1;
                    flag = 1;
                }
            }
        }
        if (flag)
        {
            *tempit = *(tempit + 1) + *tempit;
            tempmax = *tempit;
            maxHeap.erase(tempit + 1);
        }
        else
        {
            *tempit = *(tempit - 1) + *tempit;
            tempmax = *tempit;
            maxHeap.erase(tempit - 1);
        }
        max += tempmax;
    }
    max += maxHeap[0] + maxHeap[1];
    ans.push_back(min);
    ans.push_back(max);
    return ans;
}
```

* 构造测试用例：

```c++
srand(time(nullptr));
out << num << endl;
while (num--)
{
    int len = rand() % 100 + 1;
    out << len << endl;
    for (int i = 0; i < len; i++)
    {
        int a = rand() % 10 + 1;
        out << a << " ";
    }
    out << endl;
}
printf("操作成功！\n");
```

* 简单用例的测试结果：

    ![](/img/algorithmexp3/Ans2.png)

* 不同规模测试用例下的时间统计：

    图中横坐标为石子堆个数，纵坐标为时间消耗，均测试1000组数据，可以看到是一条乘方型曲线。

    ![](/img/algorithmexp3/Q2list.png)

* 时间复杂度分析：

    对于规模为n的输入，会进行n-2次遍历，所以时间复杂度应为`O(n^2)`。

### 直线k中值问题

* 基本思想：

    对于每一个城市，只有两种情况，建或不建服务机构。对于城市i，D(i)为其到最近服务机构的距离， W(i)为其服务需求量，C(i)为其建站费用，t(i)为其建站导致的费用变化(即该站到前一站之间的居民点有一半距离的会换到新服务机构，费用发生变化)，那么有递推式：
    V(i)=min(V(i-1)+W(i)*D(i),V(i-1)+C(i)+t(i))

* 算法思路：

    建立三个数组：

    数组1 ans[i][j]存放i个居民点，最多建立j个站时的最优解。

    数组2 cost[i][j]存放前一个服务机构编号为i，新建一个服务机构编号为j所产生的费用变化。

    数组3 Prevs[i][j]存放i个居民点，最多建立j个站时的最优解的最后一个服务机构位置。

    最开始，可以初始化cost数组，随后开始对ans进行遍历，ans[i][j]=min(ans[i-1][j-1]+ cost[Prevs[i - 1][j - 1]][i],ans[i - 1][j] + W(i)*(X(i)-X(Prevs[i - 1][j])))

    最后，在ans[n][j]，0 < j <= k 中找到最小值，就是问题的解。

* 算法实现：

```c++
struct City
{
    bool operator<(City c)
    {
        return x <= c.x;
    }
    City()
    {
        x = w = c = 0;
    }
    static void initCity(int n);
    int x;
    int w;
    int c;
};

City *city;

vector<int> inittemp(1001, 0);

vector<vector<int>> ans(1001, inittemp);
vector<vector<int>> cost(1001, inittemp);
vector<vector<int>> Prevs(1001, inittemp);

int getCost(int i, int j)
{
    return city[i].w * (city[i].x - city[j].x);
}

void initCity(int n)
{
    for (int i = 1; i <= n; i++)
    {
        cost[0][i] = city[i].c;
        for (int j = 1; j < i; j++)
        {
            cost[0][i] -= getCost(j, i);
        }
    }

    int mid = 0;

    for (int i = 1; i < n; i++)
    {
        for (int j = i + 1; j <= n; j++)
        {
            mid = (city[i].x + city[j].x) / 2 + (city[i].x + city[j].x) % 2;
            cost[i][j] += city[j].c;
            for (int k = i + 1; k < j; k++)
            {
                if (city[k].x > mid)
                {
                    cost[i][j] -= getCost(k, i) + getCost(k, j);
                }
            }
        }
    }
}

int MidValue(int k, int n)
{
    initCity(n);
    int nowk = 0;
    for (int i = 1; i <= n; i++)
    {
        nowk = min(i, k);
        for (int j = 1; j <= nowk; j++)
        {
            int isi = ans[i - 1][j - 1] + cost[Prevs[i - 1][j - 1]][i];
            if (j == i)
            {
                Prevs[i][j] = i;
                ans[i][j] = isi;
            }
            else
            {
                int noti = ans[i - 1][j] + getCost(i, Prevs[i - 1][j]);
                if (isi <= noti)
                {
                    Prevs[i][j] = i;
                    ans[i][j] = isi;
                }
                else
                {
                    Prevs[i][j] = Prevs[i - 1][j];
                    ans[i][j] = noti;
                }
            }
        }
    }
    int min = ans[n][1];
    for (int i = 1; i <= k; i++)
    {
        if (ans[n][i] < min)
        {
            min = ans[n][i];
        }
    }
    return min;
}
```

* 构造测试用例：

```c++
srand(time(nullptr));
out << num << endl;
while (num--)
{
    int len = rand() % 1000 + 1;
    out << len << endl;
    int k = rand() % len + 1;
    out << k << endl;
    int prevx = 0;
    int nextx = 0;
    for (int i = 0; i < len; i++)
    {
        nextx += 3;
        int x = (rand() % (nextx - prevx)) + prevx + 1;
        out << x << " ";
        prevx = x;
        out << rand() % 9 + 1 << " ";
        out << rand() % 9 + 1 << endl;
    }
}
printf("操作成功！\n");
```

* 简单用例的测试结果：

    ![](/img/algorithmexp3/Ans3.png)

* 不同规模测试用例下的时间统计：

    图中横坐标为居民点的个数，纵坐标为时间消耗，均测试1000组数据，可以看到大致为一个乘方型曲线。

    ![](/img/algorithmexp3/Q3list.png)

* 时间复杂度分析：

    对于规模为n*3的输入，仅需对二维数组ans[n+1][n+1]进行遍历，时间复杂度为`O(n^2)`.

### 汽车加油问题

* 基本思想：

    很简单的贪心算法，在每次无法跑完下一段路时加一次油即可。

* 算法思路：

    遍历距离数组，当前油量 >= 下一距离，则削减油量；当前油量 < 下一距离，则+1加油次数。若距离大于最大油量，则返回并输出`No Solution`。

* 算法实现：

```c++
int AddOil(vector<int> &dist, int n, int k) // dist has k elements
{
    int ans = 0;
    int temp = 0;
    for (int i = 0; i <= k; i++)
    {
        if (dist[i] > n)
            return -1;
        temp += dist[i];
        if (temp > n)
        {
            ans++;
            temp = dist[i];
        }
    }
    return ans;
}
```

* 构造测试用例：

```c++
srand(time(nullptr));
out << num << endl;
while (num--)
{
    int len = rand() % 10 + 1;
    out << len << " ";
    int k = rand() % 1000 + 1; // 加油站个数
    out << k << endl;
    int choose = rand() % 100;
    for (int i = 0; i <= k; i++)
    {
        int a;
        if (choose == 99)
        {
            a = rand() % (len + 1) + 1;
        }
        else
            a = rand() % len + 1;
        out << a << " ";
    }
    out << endl;
}
printf("操作成功！\n");
```

* 简单用例的测试结果：

    ![](/img/algorithmexp3/Ans4.png)

* 不同规模测试用例下的时间统计：

    表中横坐标为加油站个数n，纵坐标为时间消耗，均测试1000组数据，可以看出其为一个曲线，推测可能是内存使用量的原因导致数据量大时用时激增。

    ![](/img/algorithmexp3/Q4list.png)

* 时间复杂度分析：

    对于一个长度为n的数组输入，需要进行一次遍历，所以算法的时间复杂度为O(n)。
