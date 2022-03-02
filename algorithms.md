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
