### [无重复字符的最长字串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)
> 输入"abcabcbb"，输出3；
> 输入"bbbbb"，输出1；

* 解题思路
    * 维持一个滑动串口，使用left哨兵标记左窗口边界，但不包含left，并使用map来记录出现元素及其位置，这样可以很方便的获取不重复最长字符串。

```cpp
int lengthOfLongestSubstring(string s) {
    int n = s.size(), left = -1, res = 0;
    map<int, int> mp;
    for(int i=0; i<n; ++i) {
        if(mp.count(s[i]) && mp[s[i]] > left) {
            left = mp[s[i]];
        }

        mp[s[i]] = i;
        res = std::max(res, i-left);
    }
    return res;
}
```

### [k个一组反转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)
> 给你一个链表，每k个节点一组进行翻转，请你返回翻转后的链表。
> k是一个正整数，它的值小于或等于链表的长度。
> 如果节点总数不是k的整数倍，那么请将最后剩余的节点保持原有顺序。

* 解题思路
    * 第一种，使用双端队列将k个元素存下来，然后按照规则k个一组翻转
    * 第二种，使用头插来翻转，无需额外的空间复杂度

```cpp
ListNode* reverseKGroup(ListNode* head, int k) {
    if(head == nullptr) return nullptr;

    deque<ListNode*> deq;
    ListNode newHaed(-1); 
    ListNode* p = &newHaed;
    while(head) {
        for(int i=0; i<k && head; ++i) {
            deq.push_back(head);
            head = head->next;
        }

        // reverse the list
        if(deq.size() == k) {
            while(!deq.empty()) {
                p->next = deq.back();
                p = p->next;
                deq.pop_back();
            }
        } else {
            while(!deq.empty()) {
                p->next = deq.front();
                p = p->next;
                deq.pop_front();
            }
        }
    }
    p->next = nullptr;
    return newHaed.next;
}
```

```cpp
pair<ListNode*, ListNode*> reverseHelper(ListNode* head, ListNode* tail) {
    ListNode* p = head;
    ListNode* newHead = tail->next;

    while(tail != newHead) {
        ListNode* next = p->next;
        p->next = newHead;
        newHead = p;
        p = next;
    }
    return {tail, head};
}
ListNode* reverseKGroup(ListNode* head, int k) {
    ListNode newHead(-1); newHead.next = head;
    ListNode* pre = &newHead;

    while(head) {
        ListNode* tail = pre;
        for(int i=0; i<k; ++i) {
            tail = tail->next;
            if(!tail) {
                return newHead.next;
            }
        }

        ListNode* next = tail->next;
        auto [head_r, tail_r] = reverseHelper(head, tail);
        head = head_r; tail = tail_r;

        pre->next = head;
        pre = tail;
        tail->next = next;
        head = tail->next;

    }
    return newHead.next;
}
```

### [翻转链表](https://leetcode-cn.com/problems/UHnkqh/)
> 给定单链表的头节点 `head` ，请反转链表，并返回反转后的链表的头节点。

* 解题思路
    * 头插法，或使用栈

```cpp
ListNode* reverseList(ListNode* phead) {
    ListNode* newHead = nullptr;
    while(phead) {
        ListNode* next = phead->next;
        phead->next = newHead;
        newHead = phead;
        phead = next;
    }
    return newHead;
}
```

### [有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)
> 给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串 `s` ，判断字符串是否有效。

* 解题思路
    * 匹配问题，使用栈

```cpp
bool isValid(string s) {
    stack<char> st;
    for(auto ch : s) {
        if(st.empty()) st.push(ch);
        else if(st.top() == '(' && ch == ')') st.pop();
        else if(st.top() == '{' && ch == '}') st.pop();
        else if(st.top() == '[' && ch == ']') st.pop();
        else st.push(ch);
    }

    return st.empty();
}
```

### 有效的括号字符串
> 

* 解题思路


```cpp

```

### 背包问题
> 在`n`个物品中挑选若干物品装入背包，最多能装多满？假设背包的大小为`m`，每个物品的大小为A(i)

* 解题思路
    * 动态规划，dp(i)(j)表示前i个物品，能否装下j的重量，则dp(i)(j) = dp(i-1)(j) || dp(i-1)(j-A(i-1))
    * 其中，dp(i-1)(j)表示前i-1个物品可以拼出j，dp(i-1)(j-A(i-1))表示前i-1个物品可以拼出j-A(i-1)
    * 注意A(i-1)表示第i个物品，i下标从0到n，共n+1
 
 ```cpp
int backPack(int m, vector<int> &A) {
    if(A.empty() || m == 0) return  0;
    int n = A.size();
    vector<vector<bool>> dp(n+1, vector<bool>(m+1, false));
    dp[0][0] = true;

    for(int i=1; i<=m; ++i) dp[0][i] = false;

    for(int i=1; i<=n; ++i) {
        for(int j=0; j<=m; ++j) {
            dp[i][j] = dp[i-1][j];
            if(j >= A[i-1]) dp[i][j] = dp[i][j] || dp[i-1][j-A[i-1]];
        }
    }

    for(int j=m; j>=0; --j) {
        if(dp[n][j]) return j;
    }

    return 0;
}
```

* 优化
    * 迭代式：dp(i)(j) = dp(i-1)(j) || dp(i-1)(j-A(i-1))，dp(i)的状态总是依赖dp(i-1)，如果我们使用一个1维数组将dp(i-1)的所有称重记录下来，那么可以推导dp(i)的状态。
    * 注意，重量采用了反向枚举，由于这是避免dp(i-1)的状态被新计算出的dp(i)覆盖。
```cpp
int backPack(int m, vector<int> &A) {
    if(A.empty() || m == 0) return   0;
    int n = A.size();
    vector<bool> dp(m+1, false);
    dp[0] = true;

    for(int i=1; i<=n; ++i) {
        for(int j=m; j>=A[i-1]; --j) {
            dp[j] = dp[j] || dp[j-A[i-1]];
        }
    } 

    for(int j=m; j>=0; --j) {
        if(dp[j]) return j;
    }

    return 0;
}
```

### [背包问题II](https://www.lintcode.com/problem/125)
> 有 `n` 个物品和一个大小为 `m` 的背包. 给定数组 `A` 表示每个物品的大小和数组 `V` 表示每个物品的价值.问最多能装入背包的总价值是多大?

* 解题思路
    * 仍然，f(i)(j)表示前i个物品，重量限制为j的条件下，能拼出的最大重量
    * 还需要一个状态，表示那些拼不出来重量，用INT_MIN表示。
    * 那么迭代式为f(i)(j)=max(f(i-1)(j), V[i-1] + f(i-1)(j-A[i-1])

```cpp
int backPackII(int m, vector<int> &A, vector<int> &V) {
    // 问题是第i个物品装不装，找最大值
    // f[i][w] = max(f[i-1][w], V[i-1] + f[i-1][w-A[i-1]])
    // INT_MIN 表示拼不出来的重量
    int n = A.size();
    vector<vector<int>> dp(n+1, vector<int>(m+1, INT_MIN));
    dp[0][0] = 0;

    for(int i=1; i<=n; ++i) {
        for(int j=0; j<=m; ++j) {
            dp[i][j] = dp[i-1][j];
            if(j >= A[i-1] && dp[i-1][j-A[i-1]] != INT_MIN) {
                dp[i][j] = max(dp[i][j], V[i-1] + dp[i-1][j-A[i-1]]);
            }
        }
    }

    int ans = 0;
    for(int i=m; i>=0; --i) {
        ans = max(ans, dp[n][i]);
    }

    return ans;
}
```

* 优化
    * 类似的，f(i)(j)只依赖f(i-1)的状态，那么可以使用滚动数组，只记录f(i-1)的状态。
    * 但是我们为了不覆盖前一个f(i-1)的状态，我们需要反向枚举

```cpp
int backPackII(int m, vector<int> &A, vector<int> &V) {
    int n = A.size();
    vector<int> dp(m+1, INT_MIN);
    dp[0] = 0;

    for(int i=1; i<=n; ++i) {
        for(int j=m; j>=A[i-1]; --j) {
            if(dp[j-A[i-1]] != INT_MIN) {
                dp[j] = max(dp[j], V[i-1] + dp[j-A[i-1]]);
            }
        }
    }

    int ans = 0;
    for(int i=m; i>=0; --i) {
        ans = max(ans, dp[i]);
    }

    return ans;
}
```

### [背包问题III](https://www.lintcode.com/problem/440)
> 给定 `n` 种物品, 每种物品都有无限个. 第 `i` 个物品的体积为 `A[i]`, 价值为 `V[i]`. 再给定一个容量为 `m` 的背包. 问可以装入背包的最大价值是多少?

* 解题思路
    * f(i)(j)表示前i个物品，重量限制为j，且不限物品数量，能拼出的最大重量；
    * 那么f(i)(j) = max(f(i-1)(j), kV(i-1) + f(i-1)(j-kA(i-1)))
    * 其中k表示，每项物品在不超过重量的情况下，尽可能多拿。但这样需要3层循环，效率不高。
    * 转换后的迭代式f(i)(j) = max(f(i-1)(j), V(i-1) + f(i)(j-A(i-1)))
    * 其中f(i)(j) = V(i-1) + f(i)(j-A(i-1))在正向迭代表示可以使用此次迭代前面计算f(i)数据
    * 这就意味着，同一种物品在f(i)的迭代中，可以被多次使用。

```cpp
int backPackIII(vector<int> &A, vector<int> &V, int m) {
    int n = A.size();
    vector<vector<int>> dp(n+1, vector<int>(m+1, INT_MIN));
    dp[0][0] = 0;

    for(int i=1; i<=n; ++i) {
        for(int j=0; j<=m; ++j) {
            dp[i][j] = dp[i-1][j];
            if(j >= A[i-1] && dp[i][j-A[i-1]] != INT_MIN) {
                dp[i][j] = max(dp[i][j], V[i-1] + dp[i][j-A[i-1]]);
            }
        }
    }

    int ans = 0;
    for(int i=m; i>=0; --i) {
        ans = max(ans, dp[n][i]);
    }

    return ans;
}
```

* 优化
    * 类似的，每次迭代f(i)的结果之和f(i-1)和f(i)有依赖，可以使用滚动数组来优化空间。

```cpp
int backPackIII(vector<int> &A, vector<int> &V, int m) {
    int n = A.size();
    vector<int> dp(m+1, INT_MIN);
    dp[0] = 0;

    for(int i=1; i<=n; ++i) {
        for(int j=A[i-1]; j<=m; ++j) {
            if(dp[j-A[i-1]] != INT_MIN) {
                dp[j] = max(dp[j], V[i-1] + dp[j-A[i-1]]);
            }
        }
    }

    int ans = 0;
    for(int i=m; i>=0; --i) {
        ans = max(ans, dp[i]);
    }

    return ans;
}
```

### [背包问题IV](https://www.lintcode.com/problem/562/)
> 给出 n 个物品, 以及一个数组, `nums[i]`代表第`i`个物品的大小, 保证大小均为正数并且没有重复, 正整数 `target` 表示背包的大小, 找到能填满背包的方案数。 `每一个物品可以使用无数次`

* 解题思路
    * 完全背包问题与爬楼梯问题结合，迭代式：f(i)(j) = f(i-1)(j) + sum(f(i-1)(j-kA[i-1]))
    * 其中k使得j-kA[i-1]>=0即符合条件。优化后迭代式为：f(i)(j)=f(i-1)(j)+f(i)(j-A[i-1])
    * 和完全背包思路相似，f(i)(j-A[i-1])表示可以取当前迭代f(i)的值，即使用A[i-1]多次

```cpp
int backPackIV(vector<int> &nums, int target) {
    int m = target, n = nums.size();
    vector<vector<int>> dp(n+1, vector<int>(m+1, 0));
    dp[0][0] = 1;

    for(int i=1; i<=n; ++i) {
        for(int j=0; j<=m; ++j) {
            dp[i][j] = dp[i-1][j];

             if(j >= nums[i-1]) {
                dp[i][j] += dp[i][j-nums[i-1]];
             }
        }
    }

    return dp[n][target];
}
```

* 优化
    * 用滚动数组优化的迭代式：f(j)=f(j) + f(j-A[i-1])

```cpp
int backPackIV(vector<int> &nums, int target) {
    int m = target, n = nums.size();
    vector<int> dp(m+1, 0);
    dp[0] = 1;

    for(int i=1; i<=n; ++i) {
        for(int j=nums[i-1]; j<=m; ++j) {
            dp[j] += dp[j-nums[i-1]];
        }
    }

    return dp[target];
}
```

### [背包问题V](https://www.lintcode.com/problem/563/)
> 给出 n 个物品, 以及一个数组, `nums[i]` 代表第i个物品的大小, 保证大小均为正数, 正整数 `target` 表示背包的大小, 找到能填满背包的方案数。`每一个物品只能使用一次`

* 解题思路
    * 背包装满问题，dp(i)(j)表示前i个物品，装满j重量的种类数。
    * 那么，dp(i)(j)可以是不装nums[i-1], 则dp(i-1)(j)；
    * 可以是装nums[i-1]，则dp(i-1)(j-nums[i-1])

```cpp
int backPackV(vector<int> &nums, int target) {
    // dp[i][j] = dp[i-1][j] + dp[i-1][j-nums[i-1]]
    // dp[j] += dp[j-nums[i-1]]
    int n = nums.size();
    vector<int> dp(target+1, 0);
    dp[0] = 1;
    for(int i=1; i<=n; ++i) {
        for(int j=target; j>=nums[i-1]; --j) {
            dp[j] += dp[j-nums[i-1]];
        }
    }

    return dp[target];
}
```

### [背包问题VII](https://www.lintcode.com/problem/798/)
> 假设你身上有 `n` 元，超市里有多种大米可以选择，每种大米都是袋装的，必须整袋购买，给出每种大米的`重量`，`价格`以及`数量`，求`最多`能买多少公斤的大米

* 解题思路
    * 前面背包问题，是总重量限制，该题是在价格限制下，完成重量最大，本质上是一样的
    * 另外，由于物品不是无限取，所以不能用完全背包类似的迭代式化解，用正序枚举解决
    * 这里老老实实用滚动数据，并逆序枚举

```cpp
int backPackVII(int n, vector<int> &prices, vector<int> &weight, vector<int> &amounts) {
    // 前i种大米，在j元前的前提下，最大能拿的大米数
    // dp[i][j] = max(dp[i-1][j], k*weight[i-1] + dp[i-1][j-k*prices[i-1]]); k <= amounts[i-1]
    int m = amounts.size();
    vector<int> dp(n+1, 0);

    for(int i=1; i<=m; ++i) {
        for(int j=n; j>=0; --j) {
            for(int k=1; j-k*prices[i-1]>=0 && k <= amounts[i-1]; ++k) {
                dp[j] = max(dp[j], k*weight[i-1] + dp[j-k*prices[i-1]]);
            }
        }
    }

    return dp[n];
}
```

### [背包问题IX](https://www.lintcode.com/problem/800/)
> 你总共有`n` 万元，希望申请国外的大学，要申请的话需要交一定的申请费用，给出每个大学的申请费用以及你得到这个大学offer的成功概率，大学的数量是 `m`。如果经济条件允许，你可以申请多所大学。找到获得至少一份工作的最高可能性

* 解题思路
    * 至少一份工作的反面是：找不到任何一份工作。
    * 递归式为dp(i)(j) = min(dp(i-1)(j), (1-probaility(i-1))×dp(i-1)(j-prices(i-1)))
    * 这表示了在前i份工作中，利用j万元，找不到任何一份工作的概率
    * 初始条件，在前0份工作中，找不到工作概率为100%

```cpp
double backpackIX(int n, vector<int> &prices, vector<double> &probability) {
    int m = prices.size();
    vector<double> dp(n+1, 1.0);

    for(int i=1; i<=m; ++i) {
        for(int j=n; j>=prices[i-1]; --j) {
            dp[j] = min(dp[j], dp[j-prices[i-1]] * (1-probability[i-1]));
        }
    }
    
    return 1 - dp[n];
}
```

### [背包问题X](https://www.lintcode.com/problem/801/)
> 你总共有`n`元，商人总共有三种商品，它们的价格分别是150元,250元,350元，三种商品的数量可以认为是无限多的，购买完商品以后需要将剩下的钱给商人作为小费，求`最少`需要给商人多少小费

* 解题思路
    * 这属于完全背包问题。要求给商人的小费最少，即求花费最大值
    * 设迭代式dp(i)(j)表示前i种商品，不限次数购买，在总价j的限制下，最多能花多少钱
    * 那么dp(i)(j) = max(dp(i-1)(j), k * prices(i-1)+dp(i-1)(j-k * prices(i-1)))
    * 进一步，按照完全背包简化，dp(i)(j) = max(dp(i-1)(j), prices(i-1)+dp(i)(j-prices(i-1)))

```cpp
int backPackX(int n) {
    int prices[] = {150, 250, 350};
    vector<vector<int>> dp(4, vector<int>(n+1, 0));

    for(int i=1; i<=3; ++i) {
        for(int j=prices[i-1]; j<=n; ++j) {
            dp[i][j] = max(dp[i-1][j], prices[i-1] + dp[i][j-prices[i-1]]);
        }
    }

    return n - dp[3][n];
}
```

### [课程表](https://www.lintcode.com/problem/615/)
> 现在你总共有 n 门课需要选，记为 `0` 到 `n - 1`. 一些课程在修之前需要先修另外的一些课程，比如要学习课程 0 你需要先学习课程 1 ，表示为[0,1]. 给定n门课以及他们的先决条件，判断是否可能完成所有课程？

* 解题思路
    * 建图，然后使用拓扑排序思想，判断是否能访问到所有图节点
    * 建图时，可以使用邻接矩阵，或是邻接表
    * 拓扑排序思想是，从那些入度为0的节点开始入队，然后逐步加入那些有向边能到达的节点，且入度为0的节点，直到队列为空，那么可以得到一个连通图

```cpp
bool canFinish(int numCourses, vector<vector<int>> &prerequisites) {
    if(numCourses == 0 || prerequisites.empty()) return true;

    sort(prerequisites.begin(), prerequisites.end());
    auto it = unique(prerequisites.begin(), prerequisites.end());
    prerequisites.resize(distance(prerequisites.begin(), it));
    vector<int> ins(numCourses, 0); // in-degree
    vector<vector<bool>> adjcent(numCourses, vector<bool>(numCourses, false));

    for(auto& p : prerequisites) {
        adjcent[p[0]][p[1]] = true;
        ins[p[1]]++;
    }

    queue<int> Q;
    int visited = 0;
    for(int i=0; i<ins.size(); ++i) {
        if(ins[i] == 0) {
            visited++;
            Q.push(i);
        }
    }

    while(!Q.empty()) {
        int node = Q.front(); Q.pop();
        for(int i=0; i<numCourses; ++i) {
            if(node != i && adjcent[node][i]) {
                if(--ins[i] == 0) {
                    Q.push(i);
                    visited++;
                }
            }
        }
    }

    return visited == numCourses;
}
```

### [扫雷](https://www.lintcode.com/problem/1892/)
> 现在有一个简易版的扫雷游戏。你将得到一个n*m大小的二维数组作为游戏地图。  每个位置上有一个值（0或1，1代表此处没有雷，0表示有雷）。  你将获得一个起点的位置坐标（x，y），x表示所在行数，y表示所在列数（x，y均从0开始计数）。  若当下位置上没有雷，则上下左右四个方向均可以到达，若当下位置有雷，则不能再往新的方向移动。  返回所有可以到达的坐标。

```cpp
vector<vector<int>> Mine_sweeping(vector<vector<int>> &map, vector<int> &start) {
    if(map.empty()) return {};

    int rows = map.size(), cols = map[0].size();
    vector<vector<int>> res;

    if(map[start[0]][start[1]] == 0) {
        res.push_back(std::move(start));
        return res;
    }

    queue<pair<int, int> Q;
    vector<vector<bool> visited(rows, vector<bool>(cols, false));
    Q.push(make_pair(start[0], start[1]));
    visited[start[0]][start[1]] = true;

    // 可以踩雷结束！！！
    while(!Q.empty()) {
        auto [x, y] = Q.front(); Q.pop();
        res.push_back(vector<int>{x, y});
        if(map[x][y] == 0) {
            continue;
        }

        if(x+1 < rows && !visited[x+1][y]) {
            Q.push(make_pair(x+1, y));
            visited[x+1][y] = true;
        }

        if(x-1>=0 && !visited[x-1][y]) {
            Q.push(make_pair(x-1, y));
            visited[x-1][y] = true;
        }

        if(y+1 < cols && !visited[x][y+1]) {
            Q.push(make_pair(x, y+1));
            visited[x][y+1] = true;
        }

        if(y-1>=0 && !visited[x][y-1]) {
            Q.push(make_pair(x, y-1));
            visited[x][y-1] = true;
        }
    }

    return res;
}
```

### [取物资II](https://www.lintcode.com/problem/1899/)
> 在一个二维地图上有很多军营，每个军营的坐标为(x,y) (x, y)，现在你要在x x轴上设置一个补给站，补给站不一定要在整点上，现在想让军营到补给站的距离的最大值最小。请问最小的最大距离是多少。

* 解题思路
    * 三分搜索。
    * 补给站在x移动，可以看出两侧最大值最大，中间最大值较小。这是一个凹函数。
    * 搜索解时，我们取1/2，和3/4，进行比较，为了保证精度，我们只搜索100次。

```cpp
double check(double x, vector<vector<int>>& nums) {
    double maxd = 0;
    int n = nums.size();
    for(int i=0; i<n; ++i) {
        double tmp = sqrt((nums[i][0]-x)*(nums[i][0]-x) + nums[i][1]*nums[i][1]);
        maxd = max(maxd, tmp);
    }

    return maxd;
}

double fetchSuppliesII(vector<vector<int>> &nums) {
    double l = -10000, r = 10000;
    double mid;
    // 只搜索100次，保证精度
    for(int i=0; i<100; i++) {
        mid = l + (r-l)/2;
        double midmid = mid + (r-mid)/2;
        if(check(mid, nums) > check(midmid, nums)) {
            l = mid;
        } else {
            r = midmid;
        }
    }

    return check(mid, nums);
}
```

### [两数之和II](https://www.lintcode.com/problem/443/)
> 给一组整数，问能找出多少对整数，他们的和大于一个给定的目标值。请返回答案。

```cpp
int twoSum2(vector<int> &nums, int target) {
    if(nums.size() < 2) return   0;

    sort(nums.begin(), nums.end());

    int res = 0;
    int left = 0, right = nums.size()-1;
    while(left < right) {
        if(nums[left] + nums[right] > target) {
            res += (right - left);
            --right;
        } else {
            ++left;
        }
    }

    return res;
}
```

### [两数之和](https://www.lintcode.com/problem/56)
> 给一个整数数组，找到两个数使得他们的和等于一个给定的数 `target`。你需要实现的函数`twoSum`需要返回这两个数的下标, 并且第一个下标小于第二个下标。注意这里下标的范围是 `0` 到 `n-1`。

* 解题思路
    * 可是是使用排序，然后双指针；也可以使用一个unordered_map，存储已经出现过的元素及其下标。

```cpp
vector<int> twoSum(vector<int> &nums, int target) {
    if(nums.size() < 2) {
        return {};
    }

    vector<pair<int,int>> vec; vec.reserve(nums.size());

    for(int i=0; i<nums.size(); ++i) {
        vec.push_back(make_pair(nums[i], i));
    }

    sort(vec.begin(), vec.end(),
      [](const pair<int, int>& lhs,
      const pair<int, int>& rhs) {
        return lhs.first < rhs.first;
    });

    int left = 0, right = nums.size()-1;
    while(left < right) {
        if(vec[left].first + vec[right].first > target) --right;
        else if(vec[left].first + vec[right].first < target) ++left;
        else {
            return vec[left].second > vec[right].second ?
            vector<int>{vec[right].second, vec[left].second} :
            vector<int>{vec[left].second, vec[right].second};
        }
    }

    return {};
}
```

### [三数之和](https://www.lintcode.com/problem/57)
> 给出一个有`n`个整数的数组`S`，在`S`中找到三个整数`a`, `b`, `c`，找到所有使得`a + b + c = 0`的三元组。

* 解题思路
    * 题目不难，注意去重

```cpp
vector<vector<int>> threeSum(vector<int> &nums) {
    if(nums.size() < 3) return {};

    vector<vector<int>> res;

    sort(nums.begin(), nums.end());
    for(int i=0; i<nums.size()-2; ++i) {
        if(i > 0 && nums[i] == nums[i-1]) continue; //去重

        int left = i+1, right = nums.size()-1;
        while(left < right) {
            if(left > i+1 && nums[left] == nums[left-1]) { //去重
                ++left;
                continue;
            }

            if(nums[i] + nums[left] + nums[right] == 0) {
                res.push_back(vector<int>{nums[i], nums[left], nums[right]});
                ++left;
            } else   if(nums[i] + nums[left] + nums[right] > 0) {
                --right;
            } else {
                ++left;
            }
        }
    }

    return res;
}
```

### [四数之和](https://www.lintcode.com/problem/58/)
> 给一个包含`n`个数的整数数组`S`，在`S`中找到所有使得和为给定整数`target`的四元组`(a, b, c, d)`。

```cpp
vector<vector<int>> fourSum(vector<int> &nums, int target) {
    if(nums.size() < 4) return {};

    int n = nums.size();
    vector<vector<int>> res;
    sort(nums.begin(), nums.end());
    for(int i=0; i<n-3; ++i) {
        if(i>0 && nums[i] == nums[i-1]) continue;

        for(int j=i+1; j<n-2; ++j) {
            if(j>i+1 && nums[j] == nums[j-1]) continue;

            int left = j+1, right = nums.size()-1;
            while(left < right) {
                if(left>j+1 && nums[left] == nums[left-1]) {
                    ++left;
                    continue;
                }

                int sum = nums[i] + nums[j] + nums[left] + nums[right];
                if(sum == target) {
                    res.push_back({nums[i], nums[j], nums[left], nums[right]});
                    ++left;
                } else if(sum > target) {
                    --right;
                } else {
                    ++left;
                }
            }
        }
    }

    return res;
}
```