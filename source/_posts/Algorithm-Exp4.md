---
title: Algorithm Exp4
date: 2022-12-16 19:33:11
tags: experiments
categories: Algorithm
---

---

本次实验完整代码(包含文件读写)已上传至[Github](https://github.com/movfishing/HNU-Algorithm/tree/main/Exp4)，欢迎借鉴。

本次实验略摆，没有测试自己生成的随机测试数据能不能跑，我觉得应该是能跑的...吧。

---

# 经典案例

### 优先队列式分支限界法求解0-1背包问题问题

* 基本思想：

    采用分支限界法，先将物品按每重量价值从大到小排序，价值上界由当前价值+剩余空间所能装的最大价值(装不下一整个的物品，按剩余空间乘以每重量价值添加)。

* 算法思路：

    对于每个物品(已排序)，若剩余空间可以放入，则放入，并计算价值上界；无论能不能放入，都要计算不放入该物品的价值上界。若到达叶子结点，且值大于当前所有进入队列结点的价值上界，那么这就是解。采用了Node的parent与choose参数和Node数组ans来判断物品的选择。

* 算法实现：

```c++
struct thing
{
    int sign;
    int weight;
    int value;
    double value_per_weight;

    bool operator<(const thing &t) const
    {
        return value_per_weight > t.value_per_weight;
    }
};

struct Node
{
    int bound;  // 最大可能价值
    int weight; // 背包剩余的重量
    int value;  // 当前价值
    int level;  // 当前物品的编号
    int sign;   // 在ans数组中的下标
    int parent; // 父节点在ans数组中的下标
    int choose; // 是否选择
    Node() {}
    Node(int w, int v, int l, int b, int s, int p, int c) : weight(w), value(v), level(l), bound(b), sign(s), parent(p), choose(c) {}
    Node &operator=(const Node &n)
    {
        bound = n.bound;
        weight = n.weight;
        value = n.value;
        level = n.level;
        sign = n.sign;
        parent = n.parent;
        choose = n.choose;
        return *this;
    }
};

struct cmp
{
    bool operator()(const Node &a, const Node &b)
    {
        return a.bound < b.bound;
    }
};

priority_queue<Node, vector<Node>, cmp> q;

thing *things;

int best_value = 0;

Node best_thing(0, 0, 0, 0, 0, 0, 0);

Node *ans;

int node_sign = 0;

int calculate_bound(int level, int value, int weight, int n, int c)
{
    int w = weight;
    int v = value;
    for (int i = level; i < n; i++)
    {
        if (w + things[i].weight <= c)
        {
            w += things[i].weight;
            v += things[i].value;
        }
        else
        {
            v += (c - w) * things[i].value_per_weight;
            break;
        }
    }
    return v;
}

void packet(int c, int n)
{
    sort(things, things + n);
    int w = 0;
    int v = 0;
    int level = 0;
    int bound = 0;
    bound = calculate_bound(0, 0, 0, n, c);
    Node init_node(0, 0, 0, bound, 0, -1, 0);
    q.push(init_node);
    ans[node_sign++] = init_node;
    while (!q.empty() && level != n)
    {
        Node temp = q.top();
        q.pop();
        level = temp.level + 1;
        if (temp.bound > best_value && (temp.level != n))
        {
            if (temp.weight + things[temp.level].weight <= c)
            {
                Node temp1(temp.weight + things[temp.level].weight, temp.value + things[temp.level].value, level, temp.bound, node_sign, temp.sign, 1);
                q.push(temp1);
                if (temp1.value > best_value)
                {
                    best_value = temp1.value;
                    best_thing = temp1;
                }
                ans[node_sign] = temp1;
                node_sign++;
            }
            Node temp2(temp.weight, temp.value, level, calculate_bound(level, temp.value, temp.weight, n, c), node_sign, temp.sign, 0);
            q.push(temp2);
            ans[node_sign] = temp2;
            node_sign++;
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
    int things = 10;
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

    ![](/img/algorithmexp4/Ans1.png)

* 时间复杂度分析：

    对于物品个数为n的输入，时间复杂度下界为`O(n)`，上界为`O(2^n)`。

### 分支限界法TSP问题

* 基本思想：

    同样的也是分支限界法，与上题类似，区别就是这是一个图的遍历，限界条件就是当前路径长度下界<计算得到的最短路径长度，这里的最短路径长度并不一定是最优解，而只是计算到某一步得到的最优解，很明显如果已经大于最短路径长度了，那么就一定不是最优解。

* 算法思路：

    从起始点拓展，步骤和上题基本一样。

* 算法实现：

```c++
struct node
{
    int cl;   // 当前走过的路径长度
    int rl;   // 剩余结点最小边权值之和
    int zl;   // cl+rl,路径长度下界
    int id;   // 处理的第几个景点
    int x[N]; // 记录当前路径
    node() {}
    node(int c, int r, int z, int i)
    {
        cl = c;
        rl = r;
        zl = z;
        id = i;
        memset(x, 0, sizeof(x));
    }
};

int m[N][N];  // 邻接矩阵存储无向带权图
int bestx[N]; // 最优解路径
int bestl;    // 最优解长度
int n, M;     // 景点数目,路径数目
int minv[N];  // 每个顶点最小边权值
int minsum;   // 每个顶点最小边权值之和

void init() // 初始化，注意初始化和计算函数调用的位置
{
    int i, j;
    for (i = 0; i < N; ++i)
        for (j = 0; j < N; ++j)
            m[i][j] = INF;
    memset(bestx, 0, sizeof(bestx));
    bestl = INF;
}

void cal() // 计算每个顶点的最小边权值
{
    minsum = 0;
    memset(minv, 0, sizeof(minv));
    int temp, i, j;
    for (i = 1; i <= n; ++i)
    {
        temp = INF;
        for (j = 1; j <= n; ++j)
            if (m[i][j] != INF && m[i][j] < temp)
                temp = m[i][j];
        minv[i] = temp;
        minsum += temp;
    }
}

struct cmp
{
    bool operator()(node n1, node n2) // 当前路径长度短的优先级高
    {
        return n1.zl > n2.zl; // 最小堆
    }
};

void TSP()
{
    priority_queue<node, vector<node>, cmp> q;
    node temp(0, minsum, minsum, 2); // 起点已经确定，从第2个景点开始
    int t;
    for (int i = 1; i <= n; ++i)
        temp.x[i] = i; // 初始化解向量
    q.push(temp);
    node live; // 活结点
    while (!q.empty())
    {
        live = q.top();
        q.pop();
        t = live.id;
        if (t == n) // 处理到倒数第二个景点
        {
            if (m[live.x[t - 1]][live.x[t]] != INF && m[live.x[t]][1] != INF) // 满足约束条件，有路径
            {
                if (live.cl + m[live.x[t - 1]][live.x[t]] + m[live.x[t]][1] < bestl) // 更新最优解
                {
                    bestl = live.cl + m[live.x[t - 1]][live.x[t]] + m[live.x[t]][1];
                    for (int i = 1; i <= n; ++i)
                        bestx[i] = live.x[i];
                }
            }
            continue;
        }

        if (live.cl >= bestl) // 不满足限界条件
            continue;

        for (int j = t; j <= n; ++j) // 排列树,j不能定义为整个函数的局部变量，循环过程中会出现混乱
        {
            if (m[live.x[t - 1]][live.x[j]] != INF) // 满足约束条件
            {
                int cl = live.cl + m[live.x[t - 1]][live.x[j]];
                int rl = live.rl - minv[live.x[j]];
                int zl = cl + rl;
                if (zl < bestl) // 满足限界条件
                {
                    temp = node(cl, rl, zl, t + 1);
                    for (int k = 1; k <= n; ++k)
                        temp.x[k] = live.x[k];
                    swap(temp.x[t], temp.x[j]);
                    q.push(temp);
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
        for (int j = 0; j < arrlen; j++)
        {
            out << arr[i][j] << " ";
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
break;
```

* 简单用例的测试结果：

    ![](/img/algorithmexp4/Ans2.png)


* 时间复杂度分析：

    在最差情况下至多会扩展到2^n个结点。对每个结点扩展的时间复杂度有以下几部分:弹出操作为O(logn),边选取操作为O(n^2),右子结点生成为O(n)，左子节点为O(n^2)，即结点扩展时间复杂度为O(n^2)。在最坏情况下算法的时间复杂度为`O(2^n * n^2)`。

### 无和集问题

* 基本思想：

    n表示我们最多只可以有n个子集，每个子集中任意的两个数的和不可以在这个子集中。这里我们用dfs的回溯法解决此问题。对于每一个新增的数，首先判断能不能加在子集中，如果能加在这个子集中有两个分支，一个是加在此子集，一个是判断能不能加在下一个子集。这里先判断加在子集中，然后就会判断下一个数能否加在子集中，一直判断到一个数不能加在所有子集中为止，然后回溯到上一个结点，恢复之前的条件，再判断上一个数能不能加在下一个子集，当当前的数不能加到所有的子集中时将当前值和最优值进行比较，如果当前值没有最优值好就剪掉。由此遍历所有可能出现的结果，最后输出最优解和最优值。

* 算法思路：

    从1开始回溯，建立两个两个二维数组，F[]存放中间值，answer[]存放最终的结果。

* 算法实现：

```c++
#define N 1000

int F[N][N], answer[N][N];

int n, maxValue;

int judge(int t, int k)
{
    int i, j;
    for (i = 1; i <= F[k][0]; i++)
    {
        for (j = i + 1; j <= F[k][0]; j++)
        {
            if (F[k][i] + F[k][j] == t)
                return 0;
        }
    }
    return 1;
}

void dfs(int t)
{
    if (t > maxValue)
    {
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j <= F[i][0]; j++)
            {
                answer[i][j] = F[i][j];
            }
        }
        maxValue = t;
    }

    for (int i = 0; i < n; i++)
    {
        F[i][F[i][0] + 1] = t;
        if (judge(t, i))
        {
            F[i][0] += 1;
            dfs(t + 1);
            F[i][0] -= 1;
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
    int k = rand() % 1000 + 1;
    out << k << endl;
}
printf("操作成功！\n");
```

* 简单用例的测试结果：

    ![](/img/algorithmexp4/Ans3.png)

* 时间复杂度分析：

    回溯所有情况，对于输入数n，时间复杂度上界应为`O(2^n)`.

### 工作分配问题

* 基本思想：

    让一方选另一方，这样就可以构成一棵排列树。我们让工作选人，那排列树的结点代表人，而层就代表工作。

* 算法思路：

    用一个has_work[]数组标记是否已经被安排工作，每次挑选一个未被分配工作的人进行dfs，抵达叶子结点时获得该路径的值，并与之前获得的最小值比较。

* 算法实现：

```c++
#define N 1000

int n;
int pay[N][N];
int Min = INT_MAX;
int sum = 0;
int has_work[N]; // 标记第i个人是否已经被安排工作

void dfs(int t)
{
    if (t >= n)
    {
        if (Min > sum)
        {
            Min = sum;
            return;
        }
    }
    for (int i = 0; i < n; i++)
    {
        if (!has_work[i])
        {
            has_work[i] = 1;
            sum += pay[t][i];
            if (sum < Min)
                dfs(t + 1);
            has_work[i] = 0;
            sum -= pay[t][i];
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
    int len = rand() % 1000 + 1;
    out << len << endl;
    for (int i = 0; i < len; i++)
    {
        for (int j = 0; j < len; j++)
        {
            out << rand() % 15 + 1 << " ";
        }
        out << endl;
    }
    out << endl;
}
printf("操作成功！\n");
```

* 简单用例的测试结果：

    ![](/img/algorithmexp4/Ans4.png)

* 时间复杂度分析：

    需要回溯所有情况，最差情况下的时间复杂度为`O(n!)`。

