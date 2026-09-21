# DSA 考场背诵全面版

整理目标：这份不是讲义，是直接背的统一模板。  
来源侧重：`exercises/enhance` 与 `exercises/homework` 中非 `LC/HDU/POJ` 题；空文件或半空文件按题名补标准考场写法。  
统一约定：每个独立模板里的节点都叫 `node`；图统一用 `addEdge`；能用一套写法解决的题，不拆成很多风格。

默认头文件：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1005;
const int INF = 0x3f3f3f3f;
```

## 0. 考场总规则

- 数组、顺序表、排序查找：统一 `vector<int> a`。
- 链表、树、哈希链、缓存里的节点：统一 `struct node`。
- 二叉树字段：统一 `data, lchild, rchild`。
- 链表字段：统一 `data, next`。
- 图：统一 `struct edge { int to, w; }; vector<edge> g[N]; void addEdge(...)`。
- 输出空格：统一 `bool first = true`。
- 递归建树或 DFS：边界永远先写。

空格输出模板：

```cpp
void printVector(const vector<int>& a) {
    bool first = true;
    for(int x : a) {
        if(!first) cout << " ";
        cout << x;
        first = false;
    }
    cout << "\n";
}
```

# 1. 顺序表、数组、矩阵

覆盖：顺序表插入删除查找、顺序表去重、有序顺序表合并、数组最大/次大、递增序列统计 k、颜色分类、三元组、主对角线、压缩矩阵相乘、查找有序数组对称点、两数之和。

## 1.1 顺序表基本操作

考场直接用 `vector`，比手写类更稳。

```cpp
vector<int> a;

int findPos(int x) {
    for(int i = 0; i < (int)a.size(); ++i)
        if(a[i] == x) return i;
    return -1;
}

void insertAt(int pos, int x) {
    if(pos < 0 || pos > (int)a.size()) return;
    a.insert(a.begin() + pos, x);
}

void deleteAt(int pos) {
    if(pos < 0 || pos >= (int)a.size()) return;
    a.erase(a.begin() + pos);
}
```

## 1.2 顺序表去重

保持原顺序：

```cpp
void uniqueKeepOrder(vector<int>& a) {
    vector<int> b;
    for(int x : a) {
        bool seen = false;
        for(int y : b) {
            if(x == y) {
                seen = true;
                break;
            }
        }
        if(!seen) b.push_back(x);
    }
    a = b;
}
```

如果允许用哈希：

```cpp
void uniqueFast(vector<int>& a) {
    unordered_set<int> seen;
    vector<int> b;
    for(int x : a) {
        if(!seen.count(x)) {
            seen.insert(x);
            b.push_back(x);
        }
    }
    a = b;
}
```

## 1.3 有序顺序表合并

双指针归并。

```cpp
vector<int> mergeSortedArray(const vector<int>& a, const vector<int>& b) {
    vector<int> c;
    int i = 0, j = 0;
    while(i < (int)a.size() && j < (int)b.size()) {
        if(a[i] <= b[j]) c.push_back(a[i++]);
        else c.push_back(b[j++]);
    }
    while(i < (int)a.size()) c.push_back(a[i++]);
    while(j < (int)b.size()) c.push_back(b[j++]);
    return c;
}
```

## 1.4 最大值和次大值

```cpp
pair<int, int> maxSecond(vector<int>& a) {
    int mx = INT_MIN, second = INT_MIN;
    for(int x : a) {
        if(x > mx) {
            second = mx;
            mx = x;
        } else if(x > second && x != mx) {
            second = x;
        }
    }
    return {mx, second};
}
```

## 1.5 递增序列中 k 出现次数

二分左右边界。

```cpp
int lowerBoundPos(const vector<int>& a, int x) {
    int l = 0, r = a.size();
    while(l < r) {
        int mid = l + (r - l) / 2;
        if(a[mid] < x) l = mid + 1;
        else r = mid;
    }
    return l;
}

int countK(const vector<int>& a, int k) {
    return lowerBoundPos(a, k + 1) - lowerBoundPos(a, k);
}
```

## 1.6 颜色分类

三个值时用三指针。

```cpp
void sortColors(vector<int>& a) {
    int l = 0, i = 0, r = a.size() - 1;
    while(i <= r) {
        if(a[i] == 0) swap(a[l++], a[i++]);
        else if(a[i] == 2) swap(a[i], a[r--]);
        else i++;
    }
}
```

## 1.7 有序数组对称点

常见题意：找 `a[i] == i` 或左右对称关系。最常考的是 `a[i] == i`，用二分。

```cpp
int findFixedPoint(const vector<int>& a) {
    int l = 0, r = a.size() - 1;
    while(l <= r) {
        int mid = l + (r - l) / 2;
        if(a[mid] == mid) return mid;
        if(a[mid] < mid) l = mid + 1;
        else r = mid - 1;
    }
    return -1;
}
```

## 1.8 压缩矩阵相乘

如果只存上三角矩阵，取值函数先写出来，乘法就按普通三重循环。

```cpp
int getUpper(const vector<int>& a, int n, int i, int j) {
    if(i > j) return 0;
    int idx = i * n - i * (i - 1) / 2 + (j - i);
    return a[idx];
}

vector<vector<int>> multiplyUpper(const vector<int>& A,
                                  const vector<int>& B,
                                  int n) {
    vector<vector<int>> C(n, vector<int>(n, 0));
    for(int i = 0; i < n; ++i) {
        for(int j = 0; j < n; ++j) {
            for(int k = 0; k < n; ++k) {
                C[i][j] += getUpper(A, n, i, k) * getUpper(B, n, k, j);
            }
        }
    }
    return C;
}
```

## 1.9 两数之和

```cpp
pair<int, int> twoSum(const vector<int>& a, int target) {
    unordered_map<int, int> pos;
    for(int i = 0; i < (int)a.size(); ++i) {
        int need = target - a[i];
        if(pos.count(need)) return {pos[need], i};
        pos[a[i]] = i;
    }
    return {-1, -1};
}
```

# 2. 链表与链串

覆盖：链表建表、两个有序链表合并、链表交集递减、链表逆置迭代/递归、链表分区、负整数前置、链表去重、链表重排、倒数第 k 个节点、判环、递归查找/删除第一个 x、排序链表、链串对称、链串递归模式匹配。

## 2.1 链表统一节点

```cpp
struct node {
    int data;
    node* next;
    node(int x = 0) : data(x), next(nullptr) {}
};
```

## 2.2 尾插建表

保留输入顺序。

```cpp
node* buildList(const vector<int>& a) {
    node* head = new node();
    node* r = head;
    for(int x : a) {
        r->next = new node(x);
        r = r->next;
    }
    return head;
}
```

## 2.3 打印链表

```cpp
void printList(node* head) {
    bool first = true;
    for(node* p = head->next; p; p = p->next) {
        if(!first) cout << " ";
        cout << p->data;
        first = false;
    }
    cout << "\n";
}
```

如果没有头结点，从 `head` 开始。

## 2.4 合并两个有序链表

```cpp
node* mergeList(node* a, node* b) {
    node* head = new node();
    node* r = head;

    while(a && b) {
        if(a->data <= b->data) {
            r->next = a;
            a = a->next;
        } else {
            r->next = b;
            b = b->next;
        }
        r = r->next;
    }

    r->next = a ? a : b;
    return head->next;
}
```

## 2.5 有序递增链表交集递减

两个递增链表找相同元素，用头插得到递减结果。

```cpp
node* intersectDesc(node* h1, node* h2) {
    node* ans = new node();
    node* p = h1->next;
    node* q = h2->next;

    while(p && q) {
        if(p->data < q->data) p = p->next;
        else if(p->data > q->data) q = q->next;
        else {
            node* s = new node(p->data);
            s->next = ans->next;
            ans->next = s;
            p = p->next;
            q = q->next;
        }
    }
    return ans;
}
```

## 2.6 迭代逆置

```cpp
void reverseIter(node* head) {
    node* pre = nullptr;
    node* cur = head->next;
    while(cur) {
        node* nxt = cur->next;
        cur->next = pre;
        pre = cur;
        cur = nxt;
    }
    head->next = pre;
}
```

## 2.7 递归逆置

无头结点版本。

```cpp
node* reverseRec(node* p) {
    if(!p || !p->next) return p;
    node* newHead = reverseRec(p->next);
    p->next->next = p;
    p->next = nullptr;
    return newHead;
}
```

## 2.8 链表分区：奇偶/负数前置

拆成两个链，再拼起来。

```cpp
node* partitionList(node* head) {
    node *h1 = new node(), *r1 = h1;
    node *h2 = new node(), *r2 = h2;

    for(node* p = head->next; p; ) {
        node* nxt = p->next;
        p->next = nullptr;
        if(p->data % 2 != 0) {
            r1->next = p;
            r1 = p;
        } else {
            r2->next = p;
            r2 = p;
        }
        p = nxt;
    }

    r1->next = h2->next;
    return h1;
}
```

负数前置只改判断：`if(p->data < 0)`。

## 2.9 链表去重

```cpp
void removeDuplicate(node* head) {
    unordered_set<int> seen;
    node* pre = head;
    node* p = head->next;

    while(p) {
        if(seen.count(p->data)) {
            pre->next = p->next;
            delete p;
            p = pre->next;
        } else {
            seen.insert(p->data);
            pre = p;
            p = p->next;
        }
    }
}
```

## 2.10 倒数第 k 个节点

快慢指针。

```cpp
node* kthFromEnd(node* head, int k) {
    node* fast = head->next;
    node* slow = head->next;
    while(k-- && fast) fast = fast->next;
    if(k >= 0) return nullptr;

    while(fast) {
        fast = fast->next;
        slow = slow->next;
    }
    return slow;
}
```

## 2.11 判环

如果题目给的是 `next` 数组，思想一样。

```cpp
bool hasCycle(node* head) {
    node* slow = head;
    node* fast = head;
    while(fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if(slow == fast) return true;
    }
    return false;
}
```

## 2.12 递归查找/删除第一个 x

```cpp
node* findRec(node* p, int x) {
    if(!p) return nullptr;
    if(p->data == x) return p;
    return findRec(p->next, x);
}

void deleteFirstRec(node*& p, int x) {
    if(!p) return;
    if(p->data == x) {
        node* t = p;
        p = p->next;
        delete t;
        return;
    }
    deleteFirstRec(p->next, x);
}
```

如果有头结点：`deleteFirstRec(head->next, x);`

## 2.13 链表归并排序

```cpp
node* mergeNoHead(node* a, node* b) {
    node dummy;
    node* r = &dummy;
    while(a && b) {
        if(a->data <= b->data) {
            r->next = a;
            a = a->next;
        } else {
            r->next = b;
            b = b->next;
        }
        r = r->next;
    }
    r->next = a ? a : b;
    return dummy.next;
}

node* sortList(node* head) {
    if(!head || !head->next) return head;

    node* slow = head;
    node* fast = head->next;
    while(fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }

    node* mid = slow->next;
    slow->next = nullptr;

    return mergeNoHead(sortList(head), sortList(mid));
}
```

## 2.14 链串对称

把链串字符入数组后双指针，考场最稳。

```cpp
bool isPalindromeStringList(node* head) {
    vector<int> s;
    for(node* p = head->next; p; p = p->next)
        s.push_back(p->data);

    int l = 0, r = s.size() - 1;
    while(l < r) {
        if(s[l++] != s[r--]) return false;
    }
    return true;
}
```

## 2.15 链串递归模式匹配

核心是从主串每个位置开始试匹配。

```cpp
bool matchFrom(node* s, node* t) {
    if(!t) return true;
    if(!s) return false;
    if(s->data != t->data) return false;
    return matchFrom(s->next, t->next);
}

bool containsPattern(node* s, node* t) {
    for(node* p = s; p; p = p->next)
        if(matchFrom(p, t)) return true;
    return false;
}
```

# 3. 栈、队列、双端队列、优先队列

覆盖：顺序栈基本操作、括号匹配、中缀转后缀、后缀求值、出栈序列合法性、栈排序、退格问题、偶数出栈、用栈实现队列、用队列实现栈、循环队列、顺序栈倒置循环队列、双端队列、优先队列、杨辉三角、任务等待、超市模拟、日志最大值、约瑟夫环、十进制转任意进制。

## 3.1 顺序栈

```cpp
struct seqStack {
    vector<int> data;
    int top = -1;

    seqStack(int n = 1005) : data(n) {}

    bool empty() { return top == -1; }
    bool full() { return top + 1 == (int)data.size(); }

    void push(int x) {
        if(!full()) data[++top] = x;
    }

    int pop() {
        if(empty()) return -1;
        return data[top--];
    }

    int gettop() {
        if(empty()) return -1;
        return data[top];
    }

    int size() { return top + 1; }
};
```

## 3.2 括号匹配

```cpp
bool checkBracket(const string& s) {
    stack<char> st;
    for(char c : s) {
        if(c == '(' || c == '[' || c == '{') st.push(c);
        else if(c == ')' || c == ']' || c == '}') {
            if(st.empty()) return false;
            char t = st.top();
            st.pop();
            if(c == ')' && t != '(') return false;
            if(c == ']' && t != '[') return false;
            if(c == '}' && t != '{') return false;
        }
    }
    return st.empty();
}
```

## 3.3 中缀转后缀

```cpp
int priority(char op) {
    if(op == '+' || op == '-') return 1;
    if(op == '*' || op == '/') return 2;
    return 0;
}

vector<string> infixToPostfix(const string& s) {
    vector<string> ans;
    stack<char> st;

    for(int i = 0; i < (int)s.size(); ) {
        if(isdigit(s[i])) {
            string num;
            while(i < (int)s.size() && isdigit(s[i]))
                num += s[i++];
            ans.push_back(num);
        } else if(s[i] == '(') {
            st.push(s[i++]);
        } else if(s[i] == ')') {
            while(!st.empty() && st.top() != '(') {
                ans.push_back(string(1, st.top()));
                st.pop();
            }
            if(!st.empty()) st.pop();
            i++;
        } else {
            char op = s[i++];
            while(!st.empty() && st.top() != '(' &&
                  priority(st.top()) >= priority(op)) {
                ans.push_back(string(1, st.top()));
                st.pop();
            }
            st.push(op);
        }
    }

    while(!st.empty()) {
        ans.push_back(string(1, st.top()));
        st.pop();
    }
    return ans;
}
```

## 3.4 后缀表达式求值

```cpp
int evalPostfix(const vector<string>& a) {
    stack<int> st;
    for(string s : a) {
        if(isdigit(s[0]) || (s.size() > 1 && s[0] == '-')) {
            st.push(stoi(s));
        } else {
            int b = st.top(); st.pop();
            int a = st.top(); st.pop();
            if(s == "+") st.push(a + b);
            else if(s == "-") st.push(a - b);
            else if(s == "*") st.push(a * b);
            else if(s == "/") st.push(a / b);
        }
    }
    return st.top();
}
```

## 3.5 出栈序列是否合法

```cpp
bool validPopSeq(const vector<int>& out, int n) {
    stack<int> st;
    int cur = 1;
    for(int x : out) {
        while(cur <= n && (st.empty() || st.top() != x))
            st.push(cur++);
        if(!st.empty() && st.top() == x) st.pop();
        else return false;
    }
    return true;
}
```

## 3.6 栈排序

用辅助栈保持有序。

```cpp
void sortStack(stack<int>& st) {
    stack<int> tmp;
    while(!st.empty()) {
        int x = st.top();
        st.pop();
        while(!tmp.empty() && tmp.top() > x) {
            st.push(tmp.top());
            tmp.pop();
        }
        tmp.push(x);
    }
    while(!tmp.empty()) {
        st.push(tmp.top());
        tmp.pop();
    }
}
```

## 3.7 退格问题

```cpp
string buildBackspace(const string& s) {
    string t;
    for(char c : s) {
        if(c == '#') {
            if(!t.empty()) t.pop_back();
        } else {
            t.push_back(c);
        }
    }
    return t;
}
```

## 3.8 用栈实现队列

```cpp
struct stackQueue {
    stack<int> in, out;

    void push(int x) { in.push(x); }

    int pop() {
        if(out.empty()) {
            while(!in.empty()) {
                out.push(in.top());
                in.pop();
            }
        }
        if(out.empty()) return -1;
        int x = out.top();
        out.pop();
        return x;
    }
};
```

## 3.9 用队列实现栈

```cpp
struct queueStack {
    queue<int> q;

    void push(int x) {
        q.push(x);
        int n = q.size();
        while(--n) {
            q.push(q.front());
            q.pop();
        }
    }

    int pop() {
        if(q.empty()) return -1;
        int x = q.front();
        q.pop();
        return x;
    }
};
```

## 3.10 循环队列

```cpp
struct cirQueue {
    vector<int> data;
    int front = 0, rear = 0, sz = 0;

    cirQueue(int n) : data(n) {}

    bool empty() { return sz == 0; }
    bool full() { return sz == (int)data.size(); }

    void push(int x) {
        if(full()) return;
        data[rear] = x;
        rear = (rear + 1) % data.size();
        sz++;
    }

    int pop() {
        if(empty()) return -1;
        int x = data[front];
        front = (front + 1) % data.size();
        sz--;
        return x;
    }

    int head() {
        if(empty()) return -1;
        return data[front];
    }

    int tail() {
        if(empty()) return -1;
        return data[(rear - 1 + data.size()) % data.size()];
    }
};
```

满时覆盖最旧元素：

```cpp
void pushCover(cirQueue& q, int x) {
    q.data[q.rear] = x;
    q.rear = (q.rear + 1) % q.data.size();
    if(q.sz == (int)q.data.size()) q.front = (q.front + 1) % q.data.size();
    else q.sz++;
}
```

## 3.11 顺序栈倒置循环队列

```cpp
void reverseQueueByStack(cirQueue& q) {
    stack<int> st;
    while(!q.empty()) st.push(q.pop());
    while(!st.empty()) {
        q.push(st.top());
        st.pop();
    }
}
```

## 3.12 双端队列

```cpp
struct dequeArray {
    static const int M = 200000 + 5;
    int data[M];
    int head = M / 2, tail = M / 2; // 左闭右开

    bool empty() { return head == tail; }
    int size() { return tail - head; }

    void pushFront(int x) { data[--head] = x; }
    void pushBack(int x) { data[tail++] = x; }

    int popFront() {
        if(empty()) return -1;
        return data[head++];
    }

    int popBack() {
        if(empty()) return -1;
        return data[--tail];
    }

    int front() { return empty() ? -1 : data[head]; }
    int back() { return empty() ? -1 : data[tail - 1]; }
};
```

## 3.13 优先队列

小根堆：

```cpp
priority_queue<int, vector<int>, greater<int>> pq;
```

大根堆：

```cpp
priority_queue<int> pq;
```

## 3.14 杨辉三角

```cpp
void yanghui(int n) {
    queue<int> q;
    q.push(1);
    for(int i = 1; i <= n; ++i) {
        int pre = 0;
        for(int j = 1; j <= i; ++j) {
            int cur = q.front();
            q.pop();
            cout << cur << (j == i ? '\n' : ' ');
            q.push(pre + cur);
            pre = cur;
        }
        q.push(1);
    }
}
```

## 3.15 日志分析：支持最大值

```cpp
stack<int> st, mx;

void pushLog(int x) {
    st.push(x);
    if(mx.empty()) mx.push(x);
    else mx.push(max(mx.top(), x));
}

void popLog() {
    if(st.empty()) return;
    st.pop();
    mx.pop();
}

int getMaxLog() {
    return mx.empty() ? 0 : mx.top();
}
```

## 3.16 约瑟夫环

数组/队列版最短：

```cpp
vector<int> josephus(int n, int m) {
    queue<int> q;
    for(int i = 1; i <= n; ++i) q.push(i);

    vector<int> ans;
    while(!q.empty()) {
        for(int i = 1; i < m; ++i) {
            q.push(q.front());
            q.pop();
        }
        ans.push_back(q.front());
        q.pop();
    }
    return ans;
}
```

## 3.17 十进制转任意进制

```cpp
string convertBase(int x, int base) {
    if(x == 0) return "0";
    string digits = "0123456789ABCDEF";
    stack<char> st;
    while(x) {
        st.push(digits[x % base]);
        x /= base;
    }
    string ans;
    while(!st.empty()) {
        ans += st.top();
        st.pop();
    }
    return ans;
}
```

# 4. 串、KMP、Trie

覆盖：串的模式匹配、链串模式匹配、递归查找字符位置、判断对称字符串、编码前缀问题。

## 4.1 KMP 统计出现次数

```cpp
vector<int> getNext(const string& p) {
    vector<int> nxt(p.size() + 1);
    int j = 0, k = -1;
    nxt[0] = -1;
    while(j < (int)p.size()) {
        if(k == -1 || p[j] == p[k]) {
            j++;
            k++;
            nxt[j] = k;
        } else {
            k = nxt[k];
        }
    }
    return nxt;
}

int kmpCount(const string& s, const string& p) {
    vector<int> nxt = getNext(p);
    int i = 0, j = 0, cnt = 0;
    while(i < (int)s.size()) {
        if(j == -1 || s[i] == p[j]) {
            i++;
            j++;
        } else {
            j = nxt[j];
        }

        if(j == (int)p.size()) {
            cnt++;
            j = nxt[j];
        }
    }
    return cnt;
}
```

## 4.2 递归查找字符位置

```cpp
int findCharRec(const string& s, char c, int i = 0) {
    if(i >= (int)s.size()) return -1;
    if(s[i] == c) return i;
    return findCharRec(s, c, i + 1);
}
```

## 4.3 对称字符串

```cpp
bool isPalindrome(const string& s) {
    int l = 0, r = s.size() - 1;
    while(l < r) {
        if(s[l++] != s[r--]) return false;
    }
    return true;
}
```

## 4.4 Trie 判断前缀编码

适合“任意编码不能是另一个编码前缀”。

```cpp
int trie[250005][2];
bool isEnd[250005];
int tot = 0;

bool insertCode(const string& s) {
    int u = 0;
    bool hasNew = false;

    for(char c : s) {
        int v = c - '0';
        if(!trie[u][v]) {
            trie[u][v] = ++tot;
            hasNew = true;
        }
        u = trie[u][v];
        if(isEnd[u]) return false;
    }

    isEnd[u] = true;
    return hasNew;
}
```

# 5. 二叉树

覆盖：二叉树创建与遍历、括号表示串建树、先序空标记建树、先中建树、后中建树、层中建树、层序遍历、递归/非递归遍历、镜像、对称、相同树、深度高度、节点数、叶子数、倒序输出叶子、复制、序列化与反序列化、完全二叉树、值为 x 的所有子孙、LCA。

## 5.1 二叉树统一节点

```cpp
struct node {
    int data;
    node *lchild, *rchild;
    node(int x = 0) : data(x), lchild(nullptr), rchild(nullptr) {}
};
```

如果题目数据是字符，把 `int data` 改成 `char data`，其余不变。

## 5.2 先序 + 空标记建树

输入例：`1 2 -1 -1 3 -1 -1`。

```cpp
node* createPreNull() {
    int x;
    if(!(cin >> x) || x == -1) return nullptr;
    node* root = new node(x);
    root->lchild = createPreNull();
    root->rchild = createPreNull();
    return root;
}
```

## 5.3 括号表示串建树

适合 `1(2(4,5),3)`。支持多位数字。

```cpp
node* createBracket(const string& s) {
    stack<node*> st;
    node* root = nullptr;
    node* p = nullptr;
    bool left = true;

    for(int i = 0; i < (int)s.size(); ++i) {
        if(s[i] == '(') {
            st.push(p);
            left = true;
        } else if(s[i] == ',') {
            left = false;
        } else if(s[i] == ')') {
            st.pop();
        } else if(isdigit(s[i]) || s[i] == '-') {
            int sign = 1;
            if(s[i] == '-') {
                sign = -1;
                i++;
            }
            int x = 0;
            while(i < (int)s.size() && isdigit(s[i])) {
                x = x * 10 + s[i] - '0';
                i++;
            }
            i--;
            p = new node(sign * x);

            if(st.empty()) root = p;
            else if(left) st.top()->lchild = p;
            else st.top()->rchild = p;
        }
    }
    return root;
}
```

字符版更短：

```cpp
node* createCharBracket(const string& s, int& i) {
    if(i >= (int)s.size() || s[i] == ',' || s[i] == ')') return nullptr;
    node* root = new node(s[i++]);

    if(i < (int)s.size() && s[i] == '(') {
        i++;
        root->lchild = createCharBracket(s, i);
        if(i < (int)s.size() && s[i] == ',') {
            i++;
            root->rchild = createCharBracket(s, i);
        }
        if(i < (int)s.size() && s[i] == ')') i++;
    }
    return root;
}
```

## 5.4 先序遍历 NLR

```cpp
void preOrder(node* root, vector<int>& ans) {
    if(!root) return;
    ans.push_back(root->data);
    preOrder(root->lchild, ans);
    preOrder(root->rchild, ans);
}
```

## 5.5 中序遍历 LNR

```cpp
void inOrder(node* root, vector<int>& ans) {
    if(!root) return;
    inOrder(root->lchild, ans);
    ans.push_back(root->data);
    inOrder(root->rchild, ans);
}
```

## 5.6 后序遍历 LRN

```cpp
void postOrder(node* root, vector<int>& ans) {
    if(!root) return;
    postOrder(root->lchild, ans);
    postOrder(root->rchild, ans);
    ans.push_back(root->data);
}
```

## 5.7 层序遍历

```cpp
vector<int> levelOrder(node* root) {
    vector<int> ans;
    if(!root) return ans;

    queue<node*> q;
    q.push(root);
    while(!q.empty()) {
        node* p = q.front();
        q.pop();
        ans.push_back(p->data);
        if(p->lchild) q.push(p->lchild);
        if(p->rchild) q.push(p->rchild);
    }
    return ans;
}
```

## 5.8 非递归中序遍历

```cpp
vector<int> inOrderIter(node* root) {
    vector<int> ans;
    stack<node*> st;
    node* p = root;
    while(p || !st.empty()) {
        while(p) {
            st.push(p);
            p = p->lchild;
        }
        p = st.top();
        st.pop();
        ans.push_back(p->data);
        p = p->rchild;
    }
    return ans;
}
```

## 5.9 先序 + 中序建树

```cpp
node* buildPreIn(const vector<int>& pre, const vector<int>& in,
                 int& pi, int l, int r) {
    if(pi >= (int)pre.size() || l > r) return nullptr;

    int x = pre[pi++];
    node* root = new node(x);

    int pos = l;
    while(pos <= r && in[pos] != x) pos++;

    root->lchild = buildPreIn(pre, in, pi, l, pos - 1);
    root->rchild = buildPreIn(pre, in, pi, pos + 1, r);
    return root;
}
```

## 5.10 后序 + 中序建树

```cpp
node* buildPostIn(const vector<int>& post, const vector<int>& in,
                  int& pi, int l, int r) {
    if(pi < 0 || l > r) return nullptr;

    int x = post[pi--];
    node* root = new node(x);

    int pos = l;
    while(pos <= r && in[pos] != x) pos++;

    root->rchild = buildPostIn(post, in, pi, pos + 1, r);
    root->lchild = buildPostIn(post, in, pi, l, pos - 1);
    return root;
}
```

## 5.11 层序 + 中序建树

```cpp
int findIn(const vector<int>& in, int l, int r, int x) {
    for(int i = l; i <= r; ++i)
        if(in[i] == x) return i;
    return -1;
}

node* buildLevelIn(const vector<int>& level, const vector<int>& in,
                   int l, int r) {
    if(level.empty() || l > r) return nullptr;

    int x = level[0];
    node* root = new node(x);
    int pos = findIn(in, l, r, x);

    vector<int> leftLevel, rightLevel;
    for(int i = 1; i < (int)level.size(); ++i) {
        int idx = findIn(in, l, r, level[i]);
        if(idx == -1) continue;
        if(idx < pos) leftLevel.push_back(level[i]);
        else rightLevel.push_back(level[i]);
    }

    root->lchild = buildLevelIn(leftLevel, in, l, pos - 1);
    root->rchild = buildLevelIn(rightLevel, in, pos + 1, r);
    return root;
}
```

## 5.12 高度、节点数、叶子数

```cpp
int height(node* root) {
    if(!root) return 0;
    return max(height(root->lchild), height(root->rchild)) + 1;
}

int countNode(node* root) {
    if(!root) return 0;
    return countNode(root->lchild) + countNode(root->rchild) + 1;
}

int countLeaf(node* root) {
    if(!root) return 0;
    if(!root->lchild && !root->rchild) return 1;
    return countLeaf(root->lchild) + countLeaf(root->rchild);
}
```

## 5.13 单分支节点数

```cpp
int countSingle(node* root) {
    if(!root) return 0;
    int self = (root->lchild == nullptr) ^ (root->rchild == nullptr);
    return self + countSingle(root->lchild) + countSingle(root->rchild);
}
```

## 5.14 镜像翻转

```cpp
void mirror(node* root) {
    if(!root) return;
    swap(root->lchild, root->rchild);
    mirror(root->lchild);
    mirror(root->rchild);
}
```

## 5.15 判断对称

```cpp
bool sameMirror(node* a, node* b) {
    if(!a && !b) return true;
    if(!a || !b) return false;
    return a->data == b->data &&
           sameMirror(a->lchild, b->rchild) &&
           sameMirror(a->rchild, b->lchild);
}

bool isSymmetric(node* root) {
    if(!root) return true;
    return sameMirror(root->lchild, root->rchild);
}
```

## 5.16 判断两棵树相同

```cpp
bool sameTree(node* a, node* b) {
    if(!a && !b) return true;
    if(!a || !b) return false;
    return a->data == b->data &&
           sameTree(a->lchild, b->lchild) &&
           sameTree(a->rchild, b->rchild);
}
```

## 5.17 复制二叉树

```cpp
node* copyTree(node* root) {
    if(!root) return nullptr;
    node* p = new node(root->data);
    p->lchild = copyTree(root->lchild);
    p->rchild = copyTree(root->rchild);
    return p;
}
```

## 5.18 倒序输出叶子

先右后左。

```cpp
void printLeafReverse(node* root) {
    if(!root) return;
    if(!root->lchild && !root->rchild) {
        cout << root->data << " ";
        return;
    }
    printLeafReverse(root->rchild);
    printLeafReverse(root->lchild);
}
```

## 5.19 完全二叉树判断

层序遍历，遇到空后不能再遇到非空。

```cpp
bool isComplete(node* root) {
    if(!root) return true;
    queue<node*> q;
    q.push(root);
    bool metNull = false;

    while(!q.empty()) {
        node* p = q.front();
        q.pop();

        if(!p) {
            metNull = true;
        } else {
            if(metNull) return false;
            q.push(p->lchild);
            q.push(p->rchild);
        }
    }
    return true;
}
```

## 5.20 值为 x 的节点所有子孙

```cpp
node* findNode(node* root, int x) {
    if(!root) return nullptr;
    if(root->data == x) return root;
    node* p = findNode(root->lchild, x);
    if(p) return p;
    return findNode(root->rchild, x);
}

void printDescendant(node* root) {
    if(!root) return;
    cout << root->data << " ";
    printDescendant(root->lchild);
    printDescendant(root->rchild);
}
```

调用时不要打印自己：

```cpp
node* p = findNode(root, x);
if(p) {
    printDescendant(p->lchild);
    printDescendant(p->rchild);
}
```

## 5.21 最近公共祖先 LCA

普通二叉树：

```cpp
node* lca(node* root, node* a, node* b) {
    if(!root || root == a || root == b) return root;
    node* left = lca(root->lchild, a, b);
    node* right = lca(root->rchild, a, b);
    if(left && right) return root;
    return left ? left : right;
}
```

BST 版本：

```cpp
node* lcaBST(node* root, int a, int b) {
    if(!root) return nullptr;
    if(a < root->data && b < root->data) return lcaBST(root->lchild, a, b);
    if(a > root->data && b > root->data) return lcaBST(root->rchild, a, b);
    return root;
}
```

## 5.22 序列化与反序列化

先序 + `#`。

```cpp
void serialize(node* root, vector<string>& ans) {
    if(!root) {
        ans.push_back("#");
        return;
    }
    ans.push_back(to_string(root->data));
    serialize(root->lchild, ans);
    serialize(root->rchild, ans);
}

node* deserialize(const vector<string>& a, int& i) {
    if(i >= (int)a.size()) return nullptr;
    if(a[i] == "#") {
        i++;
        return nullptr;
    }
    node* root = new node(stoi(a[i++]));
    root->lchild = deserialize(a, i);
    root->rchild = deserialize(a, i);
    return root;
}
```

# 6. BST、平衡判断、哈夫曼树

覆盖：BST 插入删除查找、判断 BST、第 k 小、范围查询、第一个大于 k、BST 转双向链表、平衡二叉树判断、哈夫曼树 WPL、哈夫曼编码、编码前缀。平衡树旋转实现不放主背诵模板。

## 6.1 BST 插入、查找、删除

```cpp
void bstInsert(node*& root, int x) {
    if(!root) {
        root = new node(x);
        return;
    }
    if(x < root->data) bstInsert(root->lchild, x);
    else if(x > root->data) bstInsert(root->rchild, x);
}

node* bstFind(node* root, int x) {
    if(!root || root->data == x) return root;
    if(x < root->data) return bstFind(root->lchild, x);
    return bstFind(root->rchild, x);
}

void bstDelete(node*& root, int x) {
    if(!root) return;
    if(x < root->data) bstDelete(root->lchild, x);
    else if(x > root->data) bstDelete(root->rchild, x);
    else {
        if(!root->lchild || !root->rchild) {
            node* child = root->lchild ? root->lchild : root->rchild;
            delete root;
            root = child;
        } else {
            node* p = root->rchild;
            while(p->lchild) p = p->lchild;
            root->data = p->data;
            bstDelete(root->rchild, p->data);
        }
    }
}
```

## 6.2 判断是否为 BST

中序递增最稳。

```cpp
bool isBSTDfs(node* root, long long& pre) {
    if(!root) return true;
    if(!isBSTDfs(root->lchild, pre)) return false;
    if(root->data <= pre) return false;
    pre = root->data;
    return isBSTDfs(root->rchild, pre);
}

bool isBST(node* root) {
    long long pre = LLONG_MIN;
    return isBSTDfs(root, pre);
}
```

## 6.3 第 k 小元素

```cpp
void kthDfs(node* root, int k, int& cnt, int& ans) {
    if(!root || cnt >= k) return;
    kthDfs(root->lchild, k, cnt, ans);
    if(++cnt == k) {
        ans = root->data;
        return;
    }
    kthDfs(root->rchild, k, cnt, ans);
}
```

## 6.4 BST 范围查询

```cpp
void rangeQuery(node* root, int L, int R, vector<int>& ans) {
    if(!root) return;
    if(root->data > L) rangeQuery(root->lchild, L, R, ans);
    if(root->data >= L && root->data <= R) ans.push_back(root->data);
    if(root->data < R) rangeQuery(root->rchild, L, R, ans);
}
```

## 6.5 第一个大于 k 的节点

```cpp
int firstGreater(node* root, int k) {
    int ans = -1;
    while(root) {
        if(root->data > k) {
            ans = root->data;
            root = root->lchild;
        } else {
            root = root->rchild;
        }
    }
    return ans;
}
```

## 6.6 BST 转有序双向链表

复用 `lchild` 当 `prev`，`rchild` 当 `next`。

```cpp
node* listHead = nullptr;
node* listTail = nullptr;

void bstToList(node* root, node*& pre) {
    if(!root) return;
    bstToList(root->lchild, pre);

    if(!pre) listHead = root;
    root->lchild = pre;
    if(pre) pre->rchild = root;
    pre = root;
    listTail = root;

    bstToList(root->rchild, pre);
}
```

## 6.7 判断平衡二叉树

```cpp
bool checkBalance(node* root, int& h) {
    if(!root) {
        h = 0;
        return true;
    }

    int lh = 0, rh = 0;
    if(!checkBalance(root->lchild, lh)) return false;
    if(!checkBalance(root->rchild, rh)) return false;

    h = max(lh, rh) + 1;
    return abs(lh - rh) <= 1;
}
```

## 6.8 哈夫曼树 WPL

```cpp
long long huffmanWPL(vector<int> w) {
    priority_queue<int, vector<int>, greater<int>> pq;
    for(int x : w) pq.push(x);

    long long ans = 0;
    while(pq.size() > 1) {
        int a = pq.top(); pq.pop();
        int b = pq.top(); pq.pop();
        int s = a + b;
        ans += s;
        pq.push(s);
    }
    return ans;
}
```

# 7. 图

覆盖：图的存储、邻接矩阵转邻接表、DFS、BFS、连通性、连通分量、路径存在、简单路径、经过顶点的回路、避开顶点的路径、最远顶点、单词接龙 BFS、拓扑排序、课程表、Prim、Kruskal、MST 唯一性、Dijkstra、次短路径、Floyd、关键路径、并查集。

## 7.1 图统一写法

所有图都用 `edge` 和 `addEdge`。无权图把权重当 1。

```cpp
struct edge {
    int to, w;
};

vector<edge> g[N];

void clearGraph(int n) {
    for(int i = 0; i <= n; ++i) g[i].clear();
}

void addEdge(int u, int v, int w = 1, bool undirected = false) {
    g[u].push_back({v, w});
    if(undirected) g[v].push_back({u, w});
}
```

邻接矩阵转邻接表：

```cpp
void readMatrixGraph(int n, bool undirected = false) {
    clearGraph(n);
    for(int i = 0; i < n; ++i) {
        for(int j = 0; j < n; ++j) {
            int x;
            cin >> x;
            if(x != 0 && x != INF && i != j) addEdge(i, j, x, false);
        }
    }
}
```

如果矩阵只是 0/1，则 `w` 就是 1。

## 7.2 DFS

```cpp
void dfs(int u, vector<bool>& vis) {
    vis[u] = true;
    for(edge e : g[u]) {
        int v = e.to;
        if(!vis[v]) dfs(v, vis);
    }
}
```

## 7.3 BFS

```cpp
vector<int> bfsOrder(int s, int n) {
    vector<int> order;
    vector<bool> vis(n, false);
    queue<int> q;

    vis[s] = true;
    q.push(s);

    while(!q.empty()) {
        int u = q.front();
        q.pop();
        order.push_back(u);

        for(edge e : g[u]) {
            int v = e.to;
            if(!vis[v]) {
                vis[v] = true;
                q.push(v);
            }
        }
    }
    return order;
}
```

## 7.4 判断路径是否存在

```cpp
bool existPath(int s, int t, int n) {
    vector<bool> vis(n, false);
    queue<int> q;

    vis[s] = true;
    q.push(s);

    while(!q.empty()) {
        int u = q.front();
        q.pop();
        if(u == t) return true;

        for(edge e : g[u]) {
            int v = e.to;
            if(!vis[v]) {
                vis[v] = true;
                q.push(v);
            }
        }
    }
    return false;
}
```

## 7.5 连通分量数量

```cpp
int countComponent(int n) {
    vector<bool> vis(n, false);
    int cnt = 0;
    for(int i = 0; i < n; ++i) {
        if(!vis[i]) {
            cnt++;
            dfs(i, vis);
        }
    }
    return cnt;
}
```

无向图连通性：`countComponent(n) == 1`。

## 7.6 所有简单路径

```cpp
vector<vector<int>> paths;

void dfsPath(int u, int t, vector<int>& path, vector<bool>& vis) {
    vis[u] = true;
    path.push_back(u);

    if(u == t) {
        paths.push_back(path);
    } else {
        for(edge e : g[u]) {
            int v = e.to;
            if(!vis[v]) dfsPath(v, t, path, vis);
        }
    }

    path.pop_back();
    vis[u] = false;
}
```

避开顶点 `ban`：

```cpp
vector<bool> vis(n, false);
vis[ban] = true;
dfsPath(s, t, path, vis);
```

## 7.7 判断经过顶点 v 的简单回路

从 `v` 的每个邻点出发，看能不能回到 `v`。

```cpp
bool cycleThrough(int v, int n) {
    for(edge e : g[v]) {
        int s = e.to;
        vector<bool> vis(n, false);
        vis[v] = true;
        if(existPath(s, v, n)) return true;
    }
    return false;
}
```

更稳的 DFS 版本：

```cpp
bool dfsBackTo(int u, int target, vector<bool>& vis) {
    if(u == target) return true;
    vis[u] = true;
    for(edge e : g[u]) {
        int v = e.to;
        if(!vis[v] && dfsBackTo(v, target, vis)) return true;
    }
    return false;
}
```

## 7.8 求距离顶点 v 最短路径中最远的顶点

无权图 BFS 距离。

```cpp
int farthestByBfs(int s, int n) {
    vector<int> dist(n, -1);
    queue<int> q;
    dist[s] = 0;
    q.push(s);

    while(!q.empty()) {
        int u = q.front();
        q.pop();
        for(edge e : g[u]) {
            int v = e.to;
            if(dist[v] == -1) {
                dist[v] = dist[u] + 1;
                q.push(v);
            }
        }
    }

    int ans = s;
    for(int i = 0; i < n; ++i) {
        if(dist[i] > dist[ans]) ans = i;
    }
    return ans;
}
```

## 7.9 单词接龙 BFS

```cpp
int ladderLength(string beginWord, string endWord, unordered_set<string>& words) {
    if(!words.count(endWord)) return 0;

    queue<pair<string, int>> q;
    q.push({beginWord, 1});
    words.erase(beginWord);

    while(!q.empty()) {
        auto [word, step] = q.front();
        q.pop();

        if(word == endWord) return step;

        for(int i = 0; i < (int)word.size(); ++i) {
            char old = word[i];
            for(char c = 'a'; c <= 'z'; ++c) {
                if(c == old) continue;
                word[i] = c;
                if(words.count(word)) {
                    q.push({word, step + 1});
                    words.erase(word);
                }
            }
            word[i] = old;
        }
    }
    return 0;
}
```

## 7.10 并查集

```cpp
int fa[N], rk[N];

void initDSU(int n) {
    for(int i = 1; i <= n; ++i) {
        fa[i] = i;
        rk[i] = 0;
    }
}

int Find(int x) {
    if(fa[x] == x) return x;
    return fa[x] = Find(fa[x]);
}

bool unite(int a, int b) {
    int ra = Find(a), rb = Find(b);
    if(ra == rb) return false;
    if(rk[ra] < rk[rb]) swap(ra, rb);
    fa[rb] = ra;
    if(rk[ra] == rk[rb]) rk[ra]++;
    return true;
}
```

## 7.11 Prim

```cpp
int prim(int n) {
    vector<int> dist(n + 1, INF);
    vector<bool> vis(n + 1, false);

    dist[1] = 0; // dist[v]：v 到当前生成树的最小边权，不是最短路
    int ans = 0;

    for(int i = 1; i <= n; ++i) {
        // 每轮选一个离生成树最近的未访问点
        int u = -1, best = INF;
        for(int j = 1; j <= n; ++j) {
            if(!vis[j] && dist[j] < best) {
                best = dist[j];
                u = j;
            }
        }
        if(u == -1) return -1;

        vis[u] = true;
        ans += best;

        // 用新加入的 u 去更新其他点到生成树的最小边权
        for(edge e : g[u]) {
            int v = e.to, w = e.w;
            if(!vis[v] && w < dist[v]) dist[v] = w;
        }
    }
    return ans;
}
```

堆优化：

```cpp
int primHeap(int n) {
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    vector<bool> vis(n + 1, false);
    pq.push({0, 1});

    int ans = 0, cnt = 0;
    while(!pq.empty()) {
        auto [w, u] = pq.top();
        pq.pop();

        if(vis[u]) continue;
        vis[u] = true;
        ans += w;
        cnt++;

        for(edge e : g[u])
            if(!vis[e.to]) pq.push({e.w, e.to});
    }
    return cnt == n ? ans : -1;
}
```

## 7.12 Kruskal

```cpp
struct graphEdge {
    int u, v, w;
    bool operator<(const graphEdge& other) const {
        return w < other.w;
    }
};

vector<graphEdge> edges;

int kruskal(int n) {
    sort(edges.begin(), edges.end()); // 按边权从小到大尝试
    initDSU(n);

    int ans = 0, cnt = 0;
    for(auto e : edges) {
        // 不成环就选这条边
        if(unite(e.u, e.v)) {
            ans += e.w;
            cnt++;
            if(cnt == n - 1) break;
        }
    }
    return cnt == n - 1 ? ans : -1;
}
```

## 7.13 判断 MST 唯一性

同权边分组；如果某组内存在多种可选连接，可能不唯一。考场简化版：每次 Kruskal 选边时，如果存在另一条同权边连接不同集合，也能替代，则不唯一。

```cpp
bool mstUnique(int n) {
    sort(edges.begin(), edges.end());
    initDSU(n);
    bool unique = true;

    for(int i = 0; i < (int)edges.size(); ) {
        int j = i;
        while(j < (int)edges.size() && edges[j].w == edges[i].w) j++;

        vector<pair<int, int>> cand;
        for(int k = i; k < j; ++k) {
            int u = Find(edges[k].u), v = Find(edges[k].v);
            if(u != v) cand.push_back({u, v});
        }

        set<pair<int, int>> seen;
        for(auto p : cand) {
            if(p.first > p.second) swap(p.first, p.second);
            if(seen.count(p)) unique = false;
            seen.insert(p);
        }

        for(int k = i; k < j; ++k)
            unite(edges[k].u, edges[k].v);

        i = j;
    }
    return unique;
}
```

## 7.14 Dijkstra

普通版：

```cpp
vector<int> dijkstra(int n, int s) {
    vector<int> dist(n + 1, INF);
    vector<bool> vis(n + 1, false);
    dist[s] = 0; // dist[v]：源点 s 到 v 的最短路

    for(int i = 1; i <= n; ++i) {
        // 每轮确定一个 dist 最小的未访问点
        int u = -1, best = INF;
        for(int j = 1; j <= n; ++j) {
            if(!vis[j] && dist[j] < best) {
                best = dist[j];
                u = j;
            }
        }
        if(u == -1) break;
        vis[u] = true;

        // 用 u 松弛它的所有出边
        for(edge e : g[u]) {
            int v = e.to, w = e.w;
            if(!vis[v] && dist[u] + w < dist[v])
                dist[v] = dist[u] + w;
        }
    }
    return dist;
}
```

堆优化：

```cpp
vector<int> dijkstraHeap(int n, int s) {
    vector<int> dist(n + 1, INF);
    vector<bool> vis(n + 1, false);
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;

    dist[s] = 0;
    pq.push({0, s});

    while(!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();

        if(vis[u]) continue;
        vis[u] = true;

        for(edge e : g[u]) {
            int v = e.to, w = e.w;
            if(d + w < dist[v]) {
                dist[v] = d + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

## 7.15 次短路径

维护最短 `d1` 和严格次短 `d2`。

```cpp
int secondShortest(int n, int s, int t) {
    vector<int> d1(n + 1, INF), d2(n + 1, INF);
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;

    d1[s] = 0;
    pq.push({0, s});

    while(!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();
        if(d > d2[u]) continue;

        for(edge e : g[u]) {
            int v = e.to, nd = d + e.w;
            if(nd < d1[v]) {
                swap(nd, d1[v]);
                pq.push({d1[v], v});
            }
            if(nd > d1[v] && nd < d2[v]) {
                d2[v] = nd;
                pq.push({d2[v], v});
            }
        }
    }
    return d2[t] == INF ? -1 : d2[t];
}
```

## 7.16 Floyd

```cpp
int distMat[N][N];

void initFloyd(int n) {
    for(int i = 1; i <= n; ++i)
        for(int j = 1; j <= n; ++j)
            distMat[i][j] = (i == j ? 0 : INF);
}

void floyd(int n) {
    // k 是允许经过的中转点；必须放最外层
    for(int k = 1; k <= n; ++k)
        for(int i = 1; i <= n; ++i) {
            if(distMat[i][k] == INF) continue;
            for(int j = 1; j <= n; ++j) {
                if(distMat[k][j] == INF) continue;
                distMat[i][j] = min(distMat[i][j],
                                    distMat[i][k] + distMat[k][j]);
            }
        }
}
```

## 7.17 拓扑排序 Kahn

```cpp
bool topSort(int n, vector<int>& order) {
    vector<int> indeg(n, 0);
    for(int u = 0; u < n; ++u)
        for(edge e : g[u])
            indeg[e.to]++;

    // 入度为 0 的点可以先学/先做/先输出
    queue<int> q;
    for(int i = 0; i < n; ++i)
        if(indeg[i] == 0) q.push(i);

    while(!q.empty()) {
        int u = q.front();
        q.pop();
        order.push_back(u);

        for(edge e : g[u]) {
            int v = e.to;
            if(--indeg[v] == 0) q.push(v);
        }
    }
    return (int)order.size() == n;
}
```

## 7.18 DFS 拓扑排序与判环

`state = 0` 未访问，`1` 正在访问，`-1` 已完成。

```cpp
bool topDfs(int u, vector<int>& state, vector<int>& order) {
    state[u] = 1;
    for(edge e : g[u]) {
        int v = e.to;
        if(state[v] == 0) {
            if(!topDfs(v, state, order)) return false;
        } else if(state[v] == 1) {
            return false;
        }
    }
    state[u] = -1;
    order.push_back(u);
    return true;
}
```

## 7.19 关键路径

AOE 网：拓扑求 `ve`，逆拓扑求 `vl`。

```cpp
bool criticalPath(int n) {
    vector<int> order;
    if(!topSort(n, order)) return false;

    // ve[v]：事件 v 的最早发生时间，按拓扑序正向推
    vector<int> ve(n, 0);
    for(int u : order) {
        for(edge e : g[u]) {
            int v = e.to;
            ve[v] = max(ve[v], ve[u] + e.w);
        }
    }

    int finish = *max_element(ve.begin(), ve.end());
    vector<int> vl(n, finish);

    // vl[u]：事件 u 的最晚发生时间，按逆拓扑序反推
    for(int i = n - 1; i >= 0; --i) {
        int u = order[i];
        for(edge e : g[u]) {
            int v = e.to;
            vl[u] = min(vl[u], vl[v] - e.w);
        }
    }

    for(int u = 0; u < n; ++u) {
        for(edge e : g[u]) {
            int v = e.to;
            int ee = ve[u];
            int el = vl[v] - e.w;
            // 活动最早开始 == 最晚开始，就是关键活动
            if(ee == el) cout << u << " " << v << "\n";
        }
    }
    return true;
}
```

# 8. 查找与哈希

覆盖：顺序查找、二分查找、二分边界、输出折半查找序列、统计折半查找次数、分块查找、哈希查找、拉链法 ASL、删除哈希关键字。

## 8.1 顺序查找

```cpp
int seqSearch(const vector<int>& a, int key) {
    for(int i = 0; i < (int)a.size(); ++i)
        if(a[i] == key) return i;
    return -1;
}
```

哨兵版：

```cpp
int seqSearchSentinel(vector<int> a, int key) {
    a.push_back(key);
    int i = 0;
    while(a[i] != key) i++;
    return i == (int)a.size() - 1 ? -1 : i;
}
```

## 8.2 二分查找

```cpp
int binSearch(const vector<int>& a, int key) {
    int l = 0, r = a.size() - 1;
    while(l <= r) {
        int mid = l + (r - l) / 2;
        if(a[mid] == key) return mid;
        if(a[mid] < key) l = mid + 1;
        else r = mid - 1;
    }
    return -1;
}
```

## 8.3 输出折半查找序列

```cpp
vector<int> binTrace(const vector<int>& a, int key) {
    vector<int> trace;
    int l = 0, r = a.size() - 1;
    while(l <= r) {
        int mid = l + (r - l) / 2;
        trace.push_back(a[mid]);
        if(a[mid] == key) break;
        if(a[mid] < key) l = mid + 1;
        else r = mid - 1;
    }
    return trace;
}
```

## 8.4 二分边界

```cpp
int lowerBound(const vector<int>& a, int x) {
    int l = 0, r = a.size();
    while(l < r) {
        int mid = l + (r - l) / 2;
        if(a[mid] < x) l = mid + 1;
        else r = mid;
    }
    return l;
}

int upperBound(const vector<int>& a, int x) {
    int l = 0, r = a.size();
    while(l < r) {
        int mid = l + (r - l) / 2;
        if(a[mid] <= x) l = mid + 1;
        else r = mid;
    }
    return l;
}
```

## 8.5 分块查找

```cpp
struct indexNode {
    int key;
    int start;
};

int blockSearch(const vector<int>& a, const vector<indexNode>& idx,
                int blockSize, int key) {
    int l = 0, r = idx.size() - 1;
    while(l <= r) {
        int mid = l + (r - l) / 2;
        if(idx[mid].key >= key) r = mid - 1;
        else l = mid + 1;
    }

    if(l >= (int)idx.size()) return -1;

    int start = idx[l].start;
    int end = min(start + blockSize, (int)a.size());
    for(int i = start; i < end; ++i)
        if(a[i] == key) return i;
    return -1;
}
```

## 8.6 开放定址哈希：线性探测

```cpp
const int M = 200;
const int NULLKEY = -1;
const int DELKEY = -2;
int table[M];

void initHash() {
    for(int i = 0; i < M; ++i) table[i] = NULLKEY;
}

int hashPos(int key) {
    return key % 199;
}

bool hashInsert(int key) {
    int d = hashPos(key);
    while(table[d] != NULLKEY && table[d] != DELKEY) {
        if(table[d] == key) return false;
        d = (d + 1) % M;
    }
    table[d] = key;
    return true;
}

int hashSearch(int key, vector<int>& trace) {
    int d = hashPos(key);
    while(table[d] != NULLKEY) {
        trace.push_back(table[d]);
        if(table[d] == key) return d;
        d = (d + 1) % M;
    }
    return -1;
}
```

## 8.7 拉链法哈希

```cpp
struct node {
    int key, val;
    node* next;
    node(int k, int v) : key(k), val(v), next(nullptr) {}
};

node* htable[11];

void chainInsert(int key, int val) {
    int d = key % 11;
    node* p = new node(key, val);
    p->next = htable[d];
    htable[d] = p;
}

bool chainDelete(int key) {
    int d = key % 11;
    node* p = htable[d];
    node* pre = nullptr;

    while(p) {
        if(p->key == key) {
            if(pre) pre->next = p->next;
            else htable[d] = p->next;
            delete p;
            return true;
        }
        pre = p;
        p = p->next;
    }
    return false;
}
```

拉链法 ASL：

```cpp
double chainASL() {
    int total = 0, cnt = 0;
    for(int i = 0; i < 11; ++i) {
        int len = 0;
        for(node* p = htable[i]; p; p = p->next) {
            len++;
            total += len;
            cnt++;
        }
    }
    return cnt == 0 ? 0 : 1.0 * total / cnt;
}
```

# 9. 排序与选择

覆盖：插入排序、折半插入排序、希尔排序、冒泡排序、快速排序、简单选择排序、堆排序、判断大根堆、归并排序递归/非递归、基数排序、计数排序、稳定性分析、Top K、快速选择、第 k 大、逆序对、排序链表。

## 9.1 直接插入排序

```cpp
void insertSort(vector<int>& a) {
    for(int i = 1; i < (int)a.size(); ++i) {
        int x = a[i];
        int j = i - 1;
        while(j >= 0 && a[j] > x) {
            a[j + 1] = a[j];
            j--;
        }
        a[j + 1] = x;
    }
}
```

## 9.2 折半插入排序

```cpp
void binInsertSort(vector<int>& a) {
    for(int i = 1; i < (int)a.size(); ++i) {
        int x = a[i];
        int l = 0, r = i - 1;

        while(l <= r) {
            int mid = l + (r - l) / 2;
            if(a[mid] > x) r = mid - 1;
            else l = mid + 1;
        }

        for(int j = i - 1; j >= l; --j)
            a[j + 1] = a[j];
        a[l] = x;
    }
}
```

## 9.3 希尔排序

```cpp
void shellSort(vector<int>& a) {
    int n = a.size();
    for(int d = n / 2; d > 0; d /= 2) {
        for(int i = d; i < n; ++i) {
            int x = a[i];
            int j = i - d;
            while(j >= 0 && a[j] > x) {
                a[j + d] = a[j];
                j -= d;
            }
            a[j + d] = x;
        }
    }
}
```

## 9.4 冒泡排序

```cpp
void bubbleSort(vector<int>& a) {
    int n = a.size();
    for(int i = 0; i < n - 1; ++i) {
        bool swapped = false;
        for(int j = 0; j < n - 1 - i; ++j) {
            if(a[j] > a[j + 1]) {
                swap(a[j], a[j + 1]);
                swapped = true;
            }
        }
        if(!swapped) break;
    }
}
```

## 9.5 快速排序：挖坑法

```cpp
void quickSort(vector<int>& a, int l, int r) {
    if(l >= r) return;

    int pivot = a[l]; // 初始坑在 l，所以先从右边找小的填坑
    int i = l, j = r;

    while(i < j) {
        while(i < j && a[j] >= pivot) j--;
        if(i < j) a[i++] = a[j];

        while(i < j && a[i] <= pivot) i++;
        if(i < j) a[j--] = a[i];
    }

    a[i] = pivot;
    quickSort(a, l, i - 1);
    quickSort(a, i + 1, r);
}
```

## 9.6 简单选择排序

```cpp
void selectSort(vector<int>& a) {
    for(int i = 0; i < (int)a.size() - 1; ++i) {
        int minIdx = i;
        for(int j = i + 1; j < (int)a.size(); ++j)
            if(a[j] < a[minIdx]) minIdx = j;
        if(minIdx != i) swap(a[i], a[minIdx]);
    }
}
```

## 9.7 堆排序

```cpp
void heapify(vector<int>& a, int n, int i) {
    int largest = i;
    int l = 2 * i + 1;
    int r = 2 * i + 2;

    // 左右子树已经是堆时，下滤 i 才能把整棵子树调成堆
    if(l < n && a[l] > a[largest]) largest = l;
    if(r < n && a[r] > a[largest]) largest = r;

    if(largest != i) {
        swap(a[i], a[largest]);
        heapify(a, n, largest);
    }
}

void heapSort(vector<int>& a) {
    int n = a.size();
    // 从最后一个非叶子节点开始建大根堆
    for(int i = n / 2 - 1; i >= 0; --i)
        heapify(a, n, i);

    // 堆顶最大值放到末尾，再调整剩余堆
    for(int i = n - 1; i > 0; --i) {
        swap(a[0], a[i]);
        heapify(a, i, 0);
    }
}
```

判断是否为大根堆：

```cpp
bool isMaxHeap(const vector<int>& a) {
    int n = a.size();
    for(int i = 0; i < n; ++i) {
        int l = 2 * i + 1, r = 2 * i + 2;
        if(l < n && a[i] < a[l]) return false;
        if(r < n && a[i] < a[r]) return false;
    }
    return true;
}
```

## 9.8 归并排序

```cpp
void mergeRange(vector<int>& a, vector<int>& tmp, int l, int mid, int r) {
    int i = l, j = mid + 1, k = l;
    // 两个有序段 [l, mid] 和 [mid + 1, r] 合并
    while(i <= mid && j <= r) {
        if(a[i] <= a[j]) tmp[k++] = a[i++];
        else tmp[k++] = a[j++];
    }
    while(i <= mid) tmp[k++] = a[i++];
    while(j <= r) tmp[k++] = a[j++];
    for(int p = l; p <= r; ++p) a[p] = tmp[p];
}

void mergeSortDfs(vector<int>& a, vector<int>& tmp, int l, int r) {
    if(l >= r) return;
    int mid = l + (r - l) / 2;
    // 先分治排左右，再合并
    mergeSortDfs(a, tmp, l, mid);
    mergeSortDfs(a, tmp, mid + 1, r);
    mergeRange(a, tmp, l, mid, r);
}

void mergeSort(vector<int>& a) {
    vector<int> tmp(a.size());
    mergeSortDfs(a, tmp, 0, a.size() - 1);
}
```

非递归归并：

```cpp
void mergeSortIter(vector<int>& a) {
    int n = a.size();
    vector<int> tmp(n);
    for(int w = 1; w < n; w *= 2) {
        for(int l = 0; l < n; l += 2 * w) {
            int mid = min(l + w - 1, n - 1);
            int r = min(l + 2 * w - 1, n - 1);
            if(mid < r) mergeRange(a, tmp, l, mid, r);
        }
    }
}
```

## 9.9 归并排序求逆序对

```cpp
long long mergeCount(vector<int>& a, vector<int>& tmp, int l, int mid, int r) {
    int i = l, j = mid + 1, k = l;
    long long cnt = 0;

    while(i <= mid && j <= r) {
        if(a[i] <= a[j]) tmp[k++] = a[i++];
        else {
            tmp[k++] = a[j++];
            cnt += mid - i + 1;
        }
    }
    while(i <= mid) tmp[k++] = a[i++];
    while(j <= r) tmp[k++] = a[j++];
    for(int p = l; p <= r; ++p) a[p] = tmp[p];
    return cnt;
}

long long inversionDfs(vector<int>& a, vector<int>& tmp, int l, int r) {
    if(l >= r) return 0;
    int mid = l + (r - l) / 2;
    long long ans = 0;
    ans += inversionDfs(a, tmp, l, mid);
    ans += inversionDfs(a, tmp, mid + 1, r);
    ans += mergeCount(a, tmp, l, mid, r);
    return ans;
}
```

## 9.10 基数排序

非负整数。

```cpp
void radixSort(vector<int>& a) {
    if(a.empty()) return;
    int mx = *max_element(a.begin(), a.end());
    queue<int> bucket[10];

    // exp = 1, 10, 100... 从低位到高位
    for(int exp = 1; mx / exp > 0; exp *= 10) {
        for(int x : a) {
            int d = (x / exp) % 10;
            bucket[d].push(x);
        }

        int idx = 0;
        for(int i = 0; i < 10; ++i) {
            while(!bucket[i].empty()) {
                a[idx++] = bucket[i].front();
                bucket[i].pop();
            }
        }
    }
}
```

## 9.11 计数排序

```cpp
void countingSort(vector<int>& a) {
    if(a.empty()) return;
    int mn = *min_element(a.begin(), a.end());
    int mx = *max_element(a.begin(), a.end());

    vector<int> cnt(mx - mn + 1, 0);
    for(int x : a) cnt[x - mn]++;

    // 前缀和：cnt[i] 表示 <= 这个值的元素个数
    for(int i = 1; i < (int)cnt.size(); ++i)
        cnt[i] += cnt[i - 1];

    // 倒序回填，保证稳定性
    vector<int> tmp(a.size());
    for(int i = a.size() - 1; i >= 0; --i) {
        int x = a[i];
        int pos = cnt[x - mn] - 1;
        tmp[pos] = x;
        cnt[x - mn]--;
    }
    a = tmp;
}
```

## 9.12 Top K 与第 k 大

求第 k 大，用容量为 k 的小根堆。

```cpp
int topK(vector<int>& a, int k) {
    priority_queue<int, vector<int>, greater<int>> pq;
    for(int x : a) {
        if((int)pq.size() < k) pq.push(x);
        else if(x > pq.top()) {
            pq.pop();
            pq.push(x);
        }
    }
    return pq.top();
}
```

快速选择：第 k 大，`k` 从 1 开始。

```cpp
int quickSelectKthLargest(vector<int>& a, int k, int l, int r) {
    if(l >= r) return a[l];

    int pivot = a[l];
    int i = l, j = r;

    while(i < j) {
        while(i < j && a[j] <= pivot) j--;
        if(i < j) a[i++] = a[j];

        while(i < j && a[i] >= pivot) i++;
        if(i < j) a[j--] = a[i];
    }
    a[i] = pivot;

    int target = k - 1;
    if(i == target) return a[i];
    if(i < target) return quickSelectKthLargest(a, k, i + 1, r);
    return quickSelectKthLargest(a, k, l, i - 1);
}
```

## 9.13 排序稳定性

稳定：直接插入、折半插入、冒泡、归并、基数、计数。  
不稳定：希尔、快速、简单选择、堆排序。

# 10. DP、缓存、综合应用

覆盖：最小编辑距离、LRU、LFU、收入计算、图书馆统计、成绩排名系统、任务等待、超市模拟。

## 10.1 最小编辑距离

```cpp
int editDistance(const string& a, const string& b) {
    int n = a.size(), m = b.size();
    vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));

    for(int i = 0; i <= n; ++i) dp[i][0] = i;
    for(int j = 0; j <= m; ++j) dp[0][j] = j;

    for(int i = 1; i <= n; ++i) {
        for(int j = 1; j <= m; ++j) {
            if(a[i - 1] == b[j - 1]) dp[i][j] = dp[i - 1][j - 1];
            else {
                dp[i][j] = min({
                    dp[i - 1][j] + 1,
                    dp[i][j - 1] + 1,
                    dp[i - 1][j - 1] + 1
                });
            }
        }
    }
    return dp[n][m];
}
```

## 10.2 LRU 缓存

双向链表 + 哈希。节点仍叫 `node`。

```cpp
struct node {
    int key, val;
    node *prev, *next;
    node(int k = 0, int v = 0) : key(k), val(v), prev(nullptr), next(nullptr) {}
};

struct LRU {
    int cap;
    unordered_map<int, node*> mp;
    node *head, *tail;

    LRU(int c) : cap(c) {
        head = new node();
        tail = new node();
        head->next = tail;
        tail->prev = head;
    }

    void remove(node* p) {
        p->prev->next = p->next;
        p->next->prev = p->prev;
    }

    void addFront(node* p) {
        p->next = head->next;
        p->prev = head;
        head->next->prev = p;
        head->next = p;
    }

    int get(int key) {
        if(!mp.count(key)) return -1;
        node* p = mp[key];
        remove(p);
        addFront(p);
        return p->val;
    }

    void put(int key, int val) {
        if(mp.count(key)) {
            node* p = mp[key];
            p->val = val;
            remove(p);
            addFront(p);
        } else {
            if((int)mp.size() == cap) {
                node* old = tail->prev;
                remove(old);
                mp.erase(old->key);
                delete old;
            }
            node* p = new node(key, val);
            mp[key] = p;
            addFront(p);
        }
    }
};
```

## 10.3 LFU 缓存

题解用 `set((freq,time), key)`，好背。

```cpp
struct LFU {
    int cap, timer = 0;
    unordered_map<int, int> value, freq, tim;
    set<pair<pair<int, int>, int>> st;

    LFU(int c) : cap(c) {}

    void touch(int key) {
        st.erase({{freq[key], tim[key]}, key});
        freq[key]++;
        tim[key] = ++timer;
        st.insert({{freq[key], tim[key]}, key});
    }

    int get(int key) {
        if(!value.count(key)) return -1;
        touch(key);
        return value[key];
    }

    void put(int key, int val) {
        if(cap == 0) return;
        if(value.count(key)) {
            value[key] = val;
            touch(key);
            return;
        }

        if((int)value.size() == cap) {
            int oldKey = st.begin()->second;
            st.erase(st.begin());
            value.erase(oldKey);
            freq.erase(oldKey);
            tim.erase(oldKey);
        }

        value[key] = val;
        freq[key] = 1;
        tim[key] = ++timer;
        st.insert({{1, tim[key]}, key});
    }
};
```

# 11. 覆盖清单

## 11.1 enhance 非 OJ 考点

- 栈与表达式：中缀转后缀、栈排序、用栈实现队列、用队列实现栈、表达式树求值。
- 链表：约瑟夫环、两个有序链表合并、链表分区、单链表迭代/递归逆置、排序链表。
- 顺序表和数组：有序顺序表合并、顺序表插删查、两数之和、Top K、快速选择第 k 大。
- 二叉树：对称、镜像、深度、高度、序列化、相同树、叶子数、节点数、创建遍历、LCA、哈夫曼、重建、BST LCA、BST 转双向链表、BST 插删查、判断 BST、第 k 小、范围查询。
- 图：并查集、拓扑排序、关键路径、DFS、BFS、存储、连通性、MST 唯一性、Prim、Kruskal、次短路径、Floyd、Dijkstra、课程表、单词接龙。
- 查找排序：二分边界、顺序查找、堆排、归并递归/非递归、基数、计数、快排、逆序对、选择、希尔、冒泡、插入。
- 综合：最小编辑距离、LRU、LFU。

## 11.2 homework 非 OJ 考点

- 第 1-3 周：基础输入输出、数组模拟、顺序表去重、颜色分类、有序链表交集、链表重排、翻译映射、三元组、收入统计、倒数第 k、链表判环、链表去重、负数前置。
- 第 4 周：顺序栈操作、出栈序列合法性、偶数出栈、退格、括号匹配、中缀转后缀、后缀求值、栈排序、日志最大值。
- 第 5 周：循环队列、倒置队列、双端队列、优先队列、杨辉三角、缓冲区、任务等待、超市模拟。
- 第 6-7 周：压缩矩阵、KMP、链串模式匹配、递归查找字符、递归删除链表节点、进制转换。
- 第 8-10 周：二叉树建树遍历、非递归遍历、镜像、复制、倒序叶子、对称、叶子数、深度、子孙输出、完全二叉树。
- 第 11-12 周：DFS 邻接表、图连通性、哈夫曼 WPL、编码前缀 Trie、BFS 路径、连通分量、最远顶点、简单路径、简单回路。
- 第 13 周：折半查找序列和次数、数组对称点、BST 第一个大于 k、平衡二叉树判断。
- 第 15-16 周：哈希查找、拉链法删除和 ASL、希尔排序、快速排序、判断大根堆、基数排序、排序稳定性。
- 期末练习：顺序栈倒置循环队列、数组最大/次大、递增序列 k 次数、折半查找输出序列。

# 12. 最短背诵顺序

1. `node` 链表：建表、逆置、合并、判环、倒数第 k。
2. 栈队列：顺序栈、循环队列、括号、中缀后缀、后缀求值。
3. `node` 二叉树：先序空标记、括号串、遍历、高度、叶子、镜像、对称、完全树。
4. 树重建：先中、后中、层中。
5. BST 和平衡判断：插入、删除、判断、范围、第 k 小、判断是否平衡。
6. 图统一模板：`edge/g/addEdge`，DFS、BFS、路径、连通分量。
7. 图算法：并查集、Prim、Kruskal、Dijkstra、Floyd、拓扑、关键路径。
8. 查找排序：二分、哈希、快排、堆排、归并、基数、计数、Top K。
9. 综合：KMP、Trie、编辑距离、LRU、LFU。
