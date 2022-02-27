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
