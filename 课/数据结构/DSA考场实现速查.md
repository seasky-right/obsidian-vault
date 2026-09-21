# DSA 考场实现速查

整理范围：主要参考 `exercises` 里的作业、自测、课测、实验和 `enhance` 题解；明显标了 `LC/HDU/POJ` 的竞赛题不作为主要依据。`data structures` 文件夹只当教材背景，不作为这里判断实现风格的主来源。

下面代码默认：

```cpp
#include <bits/stdc++.h>
using namespace std;
const int INF = 0x3f3f3f3f;
```

## 总体考向

这些题最像“数据结构 A 的期末/课测题库”：重点不是写完整类库，而是在给定输入格式下快速手写核心结构和操作。

- 线性结构：顺序栈、循环队列、链表、链栈、链串，常考插删查、倒置、判空、模拟操作。
- 二叉树：建树、遍历、重建、统计、镜像、深度、完全二叉树、BST/AVL，是最高频方向。
- 图：邻接矩阵读入，邻接表存储，DFS/BFS 做连通、路径、回路；加权图考 MST、最短路、拓扑/关键路径。
- 查找排序：折半查找、分块查找、哈希；插入/希尔/冒泡/快排/选择/堆/归并/基数/计数。

## 最适合考场的构造方式

### 1. 线性结构优先数组模拟

顺序栈：`int data[N] + int top = -1`。  
循环队列：`front/rear/size` 或 `front/rear/count`，取模移动。  
双端队列：数组开两倍，`head = tail = N`，左闭右开。

这种写法比模板类短，边界也好检查。

### 2. 链式结构用轻量 struct

链表和树题里最常见的是：

```cpp
struct Node {
    int data;
    Node* next;
    Node(int x = 0) : data(x), next(nullptr) {}
};
```

链表建表常用：

- 需要保持输入顺序：尾插，维护 `r`。
- 需要逆序结果：头插到结果头结点后面。
- 删除节点：先保存 `temp`，再改链接，最后 `delete temp`。

### 3. 树题优先递归

二叉树题的稳定模板是：

```cpp
struct BTNode {
    int data;
    BTNode *lchild, *rchild;
    BTNode(int x) : data(x), lchild(nullptr), rchild(nullptr) {}
};
```

建树、遍历、统计、查找、镜像、判断对称，优先递归。层序遍历再用 `queue<BTNode*>`。

### 4. 图题优先 vector 邻接表

普通图：

```cpp
const int N = 1005;
vector<int> adj[N];
vector<bool> visited;
```

加权图：

```cpp
vector<pair<int, int>> adj[N]; // {to, weight}
```

如果输入是邻接矩阵，就边读边转邻接表：

```cpp
for(int i = 0; i < n; ++i) {
    for(int j = 0; j < n; ++j) {
        int x; cin >> x;
        if(x && i != j) adj[i].push_back(j);
    }
}
```

### 5. 排序查找用函数，不用封装

题里基本是 `vector<int> arr` + 一个排序/查找函数。  
考场建议统一升序，输出时用 `isFirst` 控制空格。

```cpp
bool isFirst = true;
for(int x : arr) {
    if(!isFirst) cout << " ";
    cout << x;
    isFirst = false;
}
```

# 线性结构

## ==循环队列实现==

适合操作模拟题：`IN/OUT/GET/SIZE`、缓冲区、固定容量队列。  
推荐用 `size` 记录元素数，判断空满最直观。

```cpp
struct CQueue {
    vector<int> data;
    int front = 0, rear = 0, sz = 0;

    CQueue(int n) : data(n) {}

    bool empty() { return sz == 0; }
    bool full() { return sz == (int)data.size(); }

    void push(int x) {
        if(full()) return;
        data[rear] = x;
        rear = (rear + 1) % data.size();
        sz++;
    }

    void pop() {
        if(empty()) return;
        front = (front + 1) % data.size();
        sz--;
    }

    int get() {
        if(empty()) return -1;
        return data[front];
    }

    int size() {
        return sz;
    }
};
```

缓冲区题如果满了要覆盖最旧元素：

```cpp
void push_cover(int x) {
    data[rear] = x;
    rear = (rear + 1) % data.size();
    if(sz == (int)data.size()) {
        front = (front + 1) % data.size();
    } else {
        sz++;
    }
}
```

## 栈实现

顺序栈考得最多，`top = -1` 表示空。

```cpp
struct SeqStack {
    vector<int> data;
    int top = -1;

    SeqStack(int n) : data(n) {}

    bool empty() { return top == -1; }
    bool full() { return top == (int)data.size() - 1; }

    void push(int x) {
        if(full()) {
            cout << "FULL\n";
            return;
        }
        data[++top] = x;
    }

    void pop() {
        if(empty()) {
            cout << "EMPTY\n";
            return;
        }
        cout << data[top--] << "\n";
    }

    void gettop() {
        if(empty()) cout << "EMPTY\n";
        else cout << data[top] << "\n";
    }

    int size() {
        return top + 1;
    }
};
```

表达式、括号匹配、DFS 非递归，直接用 STL 更快：

```cpp
stack<char> st;
```

# 二叉树

## ==括号表示串建树==

适合 `A(B(D,E),C)` 这类广义表/家谱题。你的题解里递归指针移动最简洁。

```cpp
struct BTNode {
    char data;
    BTNode *lchild, *rchild;
    BTNode(char d) : data(d), lchild(nullptr), rchild(nullptr) {}
};

BTNode* createByBracket(const string& s, int& i) {
    if(i >= (int)s.size()) return nullptr;
    if(s[i] == ',' || s[i] == ')') return nullptr;

    BTNode* root = new BTNode(s[i++]);

    if(i < (int)s.size() && s[i] == '(') {
        i++;
        root->lchild = createByBracket(s, i);
        if(i < (int)s.size() && s[i] == ',') {
            i++;
            root->rchild = createByBracket(s, i);
        }
        if(i < (int)s.size() && s[i] == ')') i++;
    }

    return root;
}
```

## 层序遍历

考场推荐 `queue` 版；比按层 `vector<vector<int>>` 更短。

```cpp
void levelOrder(BTNode* root) {
    if(!root) return;

    queue<BTNode*> q;
    q.push(root);
    bool first = true;

    while(!q.empty()) {
        BTNode* p = q.front();
        q.pop();

        if(!first) cout << " ";
        cout << p->data;
        first = false;

        if(p->lchild) q.push(p->lchild);
        if(p->rchild) q.push(p->rchild);
    }
}
```

## ==NLR 建树：先序 + 中序==

核心：先序第一个是根；在中序里找到根，左边是左子树，右边是右子树。

```cpp
struct Node {
    int data;
    Node *lchild, *rchild;
    Node(int x) : data(x), lchild(nullptr), rchild(nullptr) {}
};

Node* buildPreIn(const vector<int>& pre, const vector<int>& in,
                 int& pi, int inL, int inR) {
    if(pi >= (int)pre.size() || inL > inR) return nullptr;

    int d = pre[pi++];
    Node* root = new Node(d);

    int pos = inL;
    while(pos <= inR && in[pos] != d) pos++;

    root->lchild = buildPreIn(pre, in, pi, inL, pos - 1);
    root->rchild = buildPreIn(pre, in, pi, pos + 1, inR);
    return root;
}
```

## ==LRN 建树：后序 + 中序==

核心：后序最后一个是根；为了自然递归，可以从后往前读，先建右子树再建左子树。

```cpp
Node* buildPostIn(const vector<int>& post, const vector<int>& in,
                  int& pi, int inL, int inR) {
    if(pi < 0 || inL > inR) return nullptr;

    int d = post[pi--];
    Node* root = new Node(d);

    int pos = inL;
    while(pos <= inR && in[pos] != d) pos++;

    root->rchild = buildPostIn(post, in, pi, pos + 1, inR);
    root->lchild = buildPostIn(post, in, pi, inL, pos - 1);
    return root;
}
```

## ==层序 + 中序建树==

核心：层序第一个是根；用中序位置把剩余层序拆成左右子树层序。

```cpp
int findIndex(const vector<char>& in, int l, int r, char x) {
    for(int i = l; i <= r; ++i)
        if(in[i] == x) return i;
    return -1;
}

BTNode* buildLevelIn(const vector<char>& level,
                     const vector<char>& in,
                     int inL, int inR) {
    if(level.empty() || inL > inR) return nullptr;

    BTNode* root = new BTNode(level[0]);
    int pos = findIndex(in, inL, inR, level[0]);

    vector<char> leftLevel, rightLevel;
    for(int i = 1; i < (int)level.size(); ++i) {
        int idx = findIndex(in, inL, inR, level[i]);
        if(idx == -1) continue;
        if(idx < pos) leftLevel.push_back(level[i]);
        else rightLevel.push_back(level[i]);
    }

    root->lchild = buildLevelIn(leftLevel, in, inL, pos - 1);
    root->rchild = buildLevelIn(rightLevel, in, pos + 1, inR);
    return root;
}
```

## ==BST 实现==

插入、查找、删除都用递归。删除双分支节点时，用右子树最小节点替换。

```cpp
struct BSTNode {
    int key;
    BSTNode *lchild, *rchild;
    BSTNode(int x) : key(x), lchild(nullptr), rchild(nullptr) {}
};

void insert(BSTNode*& root, int x) {
    if(!root) {
        root = new BSTNode(x);
        return;
    }
    if(x < root->key) insert(root->lchild, x);
    else if(x > root->key) insert(root->rchild, x);
}

BSTNode* find(BSTNode* root, int x) {
    if(!root || root->key == x) return root;
    if(x < root->key) return find(root->lchild, x);
    return find(root->rchild, x);
}

void remove(BSTNode*& root, int x) {
    if(!root) return;

    if(x < root->key) remove(root->lchild, x);
    else if(x > root->key) remove(root->rchild, x);
    else {
        if(!root->lchild || !root->rchild) {
            BSTNode* child = root->lchild ? root->lchild : root->rchild;
            delete root;
            root = child;
        } else {
            BSTNode* p = root->rchild;
            while(p->lchild) p = p->lchild;
            root->key = p->key;
            remove(root->rchild, p->key);
        }
    }
}
```

# 图

## ==BFS==

适合判断路径是否存在、最短边数、层次扩展。读邻接矩阵时直接转 `adj`。

```cpp
const int N = 1005;
vector<int> adj[N];

bool bfs(int s, int t, int n) {
    vector<bool> visited(n, false);
    queue<int> q;

    visited[s] = true;
    q.push(s);

    while(!q.empty()) {
        int u = q.front();
        q.pop();

        if(u == t) return true;

        for(int v : adj[u]) {
            if(!visited[v]) {
                visited[v] = true;
                q.push(v);
            }
        }
    }
    return false;
}
```

## ==DFS==

递归 DFS 最适合连通性、连通分量、所有路径/回路回溯。

```cpp
void dfs(int u, vector<bool>& visited) {
    visited[u] = true;
    for(int v : adj[u]) {
        if(!visited[v]) dfs(v, visited);
    }
}
```

枚举从 `u` 到 `v` 的所有简单路径：

```cpp
vector<vector<int>> paths;

void dfsPath(int u, int target, vector<int>& path, vector<bool>& visited) {
    visited[u] = true;
    path.push_back(u);

    if(u == target) {
        paths.push_back(path);
    } else {
        for(int v : adj[u])
            if(!visited[v]) dfsPath(v, target, path, visited);
    }

    path.pop_back();
    visited[u] = false;
}
```

## ==Prim 算法==

适合稠密图或要求从某点不断扩展 MST。题解里有普通版和堆优化版；考场普通版更容易默写。

```cpp
vector<pair<int, int>> wadj[N]; // {to, weight}

int prim(int n) {
    vector<int> dist(n + 1, INF);
    vector<bool> visited(n + 1, false);

    dist[1] = 0;
    int ans = 0;

    for(int i = 1; i <= n; ++i) {
        int u = -1, best = INF;
        
        // 更新离树最短距离
        for(int j = 1; j <= n; ++j) {
            if(!visited[j] && dist[j] < best) {
                best = dist[j];
                u = j;
            }
        }

		// 访问最近节点
        if(u == -1) return -1;
        visited[u] = true;
        ans += best;

		// 松弛 更新距离矩阵
        for(auto e : wadj[u]) {
            int v = e.first, w = e.second;
            if(!visited[v] && w < dist[v]) dist[v] = w;
        }
    }
    return ans;
}
```

## ==Kruskal 算法==

适合边集输入。核心是边排序 + 并查集。

```cpp
struct Edge {
    int u, v, w;
    bool operator<(const Edge& other) const {
        return w < other.w;
    }
};

vector<Edge> edges;
int parent[N];

int Find(int x) {
    if(parent[x] == x) return x;
    return parent[x] = Find(parent[x]);
}

int kruskal(int n) {
    sort(edges.begin(), edges.end());
    for(int i = 1; i <= n; ++i) parent[i] = i;

    int ans = 0, cnt = 0;
    for(auto e : edges) {
        int ru = Find(e.u), rv = Find(e.v);
        if(ru != rv) {
            parent[ru] = rv;
            ans += e.w;
            cnt++;
            if(cnt == n - 1) break;
        }
    }

    return cnt == n - 1 ? ans : -1;
}
```

## ==Dijkstra 算法==

非负权单源最短路。题解里普通版够用；堆优化版在边多时更稳。

```cpp
void dijkstra(int n, int s) {
    vector<int> dist(n + 1, INF);
    vector<bool> visited(n + 1, false);

    dist[s] = 0;

    for(int i = 1; i <= n; ++i) {
        int u = -1, best = INF;
        for(int j = 1; j <= n; ++j) {
            if(!visited[j] && dist[j] < best) {
                best = dist[j];
                u = j;
            }
        }

        if(u == -1) break;
        visited[u] = true;

        for(auto e : wadj[u]) {
            int v = e.first, w = e.second;
            if(!visited[v] && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
            }
        }
    }
}
```

堆优化：

```cpp
void dijkstraHeap(int n, int s) {
    vector<int> dist(n + 1, INF);
    vector<bool> visited(n + 1, false);
    priority_queue<pair<int, int>,
                   vector<pair<int, int>>,
                   greater<pair<int, int>>> pq;

    dist[s] = 0;
    pq.push({0, s});

    while(!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();

        if(visited[u]) continue;
        visited[u] = true;

        for(auto [v, w] : wadj[u]) {
            if(!visited[v] && d + w < dist[v]) {
                dist[v] = d + w;
                pq.push({dist[v], v});
            }
        }
    }
}
```

## Floyd 算法

题目目录里的非竞赛版是空壳，考场直接背三重循环。

```cpp
int dist[N][N];

void floyd(int n) {
    for(int k = 1; k <= n; ++k) {
        for(int i = 1; i <= n; ++i) {
            if(dist[i][k] == INF) continue;
            for(int j = 1; j <= n; ++j) {
                if(dist[k][j] == INF) continue;
                if(dist[i][j] > dist[i][k] + dist[k][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
}
```

初始化：

```cpp
for(int i = 1; i <= n; ++i)
    for(int j = 1; j <= n; ++j)
        dist[i][j] = (i == j ? 0 : INF);
```

## 关键路径

适合 AOE 网。流程固定：拓扑排序求 `ve`，逆拓扑求 `vl`，再找 `ee == el` 的活动。

```cpp
struct AOEEdge {
    int to, w;
};

vector<AOEEdge> dag[N];

bool criticalPath(int n) {
    vector<int> indeg(n, 0), topo;

    for(int u = 0; u < n; ++u)
        for(auto e : dag[u]) indeg[e.to]++;

    queue<int> q;
    for(int i = 0; i < n; ++i)
        if(indeg[i] == 0) q.push(i);

    vector<int> ve(n, 0);
    while(!q.empty()) {
        int u = q.front();
        q.pop();
        topo.push_back(u);

        for(auto e : dag[u]) {
            int v = e.to;
            ve[v] = max(ve[v], ve[u] + e.w);
            if(--indeg[v] == 0) q.push(v);
        }
    }

    if((int)topo.size() != n) return false;

    int finish = *max_element(ve.begin(), ve.end());
    vector<int> vl(n, finish);

    for(int i = n - 1; i >= 0; --i) {
        int u = topo[i];
        for(auto e : dag[u]) {
            int v = e.to;
            vl[u] = min(vl[u], vl[v] - e.w);
        }
    }

    for(int u = 0; u < n; ++u) {
        for(auto e : dag[u]) {
            int v = e.to;
            int ee = ve[u];
            int el = vl[v] - e.w;
            if(ee == el) {
                cout << u << " " << v << "\n";
            }
        }
    }
    return true;
}
```

# 查找排序

## 二分与分块查找

普通折半查找：

```cpp
int binSearch(const vector<int>& a, int key) {
    int l = 0, r = (int)a.size() - 1;
    while(l <= r) {
        int mid = l + (r - l) / 2;
        if(a[mid] == key) return mid;
        else if(a[mid] < key) l = mid + 1;
        else r = mid - 1;
    }
    return -1;
}
```

如果题目要输出查找序列：

```cpp
void binSearchTrace(const vector<int>& a, int key, vector<int>& trace) {
    int l = 0, r = (int)a.size() - 1;
    while(l <= r) {
        int mid = l + (r - l) / 2;
        trace.push_back(a[mid]);
        if(a[mid] == key) return;
        else if(a[mid] < key) l = mid + 1;
        else r = mid - 1;
    }
}
```

分块查找：块间有序，块内顺序查。

```cpp
struct Index {
    int key;   // 本块最大值
    int start; // 本块起点
};

int blockSearch(const vector<Index>& idx,
                const vector<int>& a,
                int blockSize,
                int key) {
    int l = 0, r = (int)idx.size() - 1;
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

## ==折半插入排序==

先用二分找插入位置，再整体后移。相等时继续往右，可以保持稳定。

```cpp
void binInsertSort(vector<int>& a) {
    int n = a.size();
    for(int i = 1; i < n; ++i) {
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

## 希尔排序

题解里的增量是 `n / 2, n / 4, ...`，组内直接插入。

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

## 冒泡排序

修正版交换标记：只有真的交换才置 `true`。

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

## 快速排序

你题里常用“挖坑法”：先从右边找小的填左坑，再从左边找大的填右坑。

```cpp
void quickSort(vector<int>& a, int low, int high) {
    if(low >= high) return;

    int pivot = a[low];
    int i = low, j = high;

    while(i < j) {
        while(i < j && a[j] >= pivot) j--;
        if(i < j) a[i++] = a[j];

        while(i < j && a[i] <= pivot) i++;
        if(i < j) a[j--] = a[i];
    }

    a[i] = pivot;
    quickSort(a, low, i - 1);
    quickSort(a, i + 1, high);
}
```

## 简单选择排序

每轮从无序区选最小值放到当前位置。

```cpp
void selectSort(vector<int>& a) {
    int n = a.size();
    for(int i = 0; i < n - 1; ++i) {
        int minIdx = i;
        for(int j = i + 1; j < n; ++j)
            if(a[j] < a[minIdx]) minIdx = j;
        if(minIdx != i) swap(a[i], a[minIdx]);
    }
}
```

## 堆排序

0-based 大根堆，下滤函数 `heapify`。

```cpp
void heapify(vector<int>& a, int n, int i) {
    int largest = i;
    int l = 2 * i + 1;
    int r = 2 * i + 2;

    if(l < n && a[l] > a[largest]) largest = l;
    if(r < n && a[r] > a[largest]) largest = r;

    if(largest != i) {
        swap(a[i], a[largest]);
        heapify(a, n, largest);
    }
}

void heapSort(vector<int>& a) {
    int n = a.size();

    for(int i = n / 2 - 1; i >= 0; --i)
        heapify(a, n, i);

    for(int i = n - 1; i > 0; --i) {
        swap(a[0], a[i]);
        heapify(a, i, 0);
    }
}
```

## 归并排序

递归版最稳，逆序对题也可以在 `merge` 时顺手计数。

```cpp
void mergeRange(vector<int>& a, vector<int>& temp,
                int l, int mid, int r) {
    int i = l, j = mid + 1, k = l;

    while(i <= mid && j <= r) {
        if(a[i] <= a[j]) temp[k++] = a[i++];
        else temp[k++] = a[j++];
    }
    while(i <= mid) temp[k++] = a[i++];
    while(j <= r) temp[k++] = a[j++];

    for(int p = l; p <= r; ++p)
        a[p] = temp[p];
}

void mergeSortDfs(vector<int>& a, vector<int>& temp, int l, int r) {
    if(l >= r) return;
    int mid = l + (r - l) / 2;
    mergeSortDfs(a, temp, l, mid);
    mergeSortDfs(a, temp, mid + 1, r);
    mergeRange(a, temp, l, mid, r);
}

void mergeSort(vector<int>& a) {
    vector<int> temp(a.size());
    mergeSortDfs(a, temp, 0, (int)a.size() - 1);
}
```

## 基数排序

只适合非负整数。LSD：从个位开始，用 10 个队列做桶。

```cpp
void radixSort(vector<int>& a) {
    if(a.empty()) return;

    int maxVal = 0;
    for(int x : a) maxVal = max(maxVal, x);

    queue<int> bucket[10];

    for(int exp = 1; maxVal / exp > 0; exp *= 10) {
        for(int x : a) {
            int digit = (x / exp) % 10;
            bucket[digit].push(x);
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

## 计数排序

适合值域不大的整数。用 `minVal` 平移，倒序回填保证稳定。

```cpp
void countingSort(vector<int>& a) {
    if(a.empty()) return;

    int minVal = a[0], maxVal = a[0];
    for(int x : a) {
        minVal = min(minVal, x);
        maxVal = max(maxVal, x);
    }

    vector<int> cnt(maxVal - minVal + 1, 0);
    for(int x : a) cnt[x - minVal]++;

    for(int i = 1; i < (int)cnt.size(); ++i)
        cnt[i] += cnt[i - 1];

    vector<int> temp(a.size());
    for(int i = (int)a.size() - 1; i >= 0; --i) {
        int x = a[i];
        int pos = cnt[x - minVal] - 1;
        temp[pos] = x;
        cnt[x - minVal]--;
    }

    a = temp;
}
```

# 考场优先级

最值得优先背熟：

1. 循环队列、顺序栈、链表头插/尾插/删除。
2. 二叉树递归建树：括号串、先中、后中、层中。
3. 二叉树递归操作：遍历、高度、叶子、镜像、查找。
4. 图的 `vector<int> adj[N]` + DFS/BFS。
5. Kruskal + 并查集、Dijkstra 普通版。
6. 快排挖坑法、堆排下滤、归并排序。

容易现场写错的点：

- 循环队列取队尾：`(rear - 1 + capacity) % capacity`。
- 建树分治：先确认根在中序里的位置，再分左右区间。
- 层序中序建树：层序剩余元素要按中序位置分进左右子树。
- 快排挖坑法：先从右扫，因为最初坑在 `low`。
- 堆排序：建堆从 `n / 2 - 1` 开始。
- 计数排序前缀和从 `i = 1` 开始。
