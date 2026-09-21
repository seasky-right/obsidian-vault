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

## 0.0 全题型核心思路一览

这一节先背“题怎么想”，后面再背代码。每条都对应后文一个模板。

### 顺序表、数组、矩阵

- <a id="idea-001"></a>[顺序表基本操作](#code-1-1)：数组或 `vector` 维护连续空间；插入时从后往前挪，删除时从前往后覆盖，查找顺序扫。
- <a id="idea-002"></a>[顺序表去重](#code-1-2)：保序去重就维护一个新数组，没出现过才加入；允许快写时用 `unordered_set` 判重。
- <a id="idea-003"></a>[有序顺序表合并](#code-1-3)：两个指针分别指向两个有序表，每次取较小者进入结果，剩余部分直接接上。
- <a id="idea-004"></a>[最大值和次大值](#code-1-4)：一次扫描维护 `mx` 和 `second`，遇到更大值时旧最大变次大。
- <a id="idea-005"></a>[递增序列中 k 出现次数](#code-1-5)：找第一个 `>= k` 和第一个 `>= k + 1` 的位置，二者相减。
- <a id="idea-006"></a>[颜色分类](#code-1-6)：三指针，左边放 0，右边放 2，中间扫 1；遇 2 换到右边后当前位置还要继续判断。
- <a id="idea-007"></a>[有序数组对称点](#code-1-7)：本质是二分判断 `a[mid]` 和目标关系；若题意是 `a[i] == i`，按差值单调移动边界。
- <a id="idea-008"></a>[压缩矩阵相乘](#code-1-8)：先写 `getVal` 把压缩下标还原为矩阵值，再按普通矩阵三重循环乘。
- <a id="idea-009"></a>[顺序表逆置](#code-1-9)：左右双指针向中间交换；若用 `vector`，也可以直接 `reverse`。

### 链表与链串

- <a id="idea-010"></a>[链表建表](#code-2-2)：要保留输入顺序用尾插；要逆序输出或递减结果用头插。
- <a id="idea-011"></a>[打印链表](#code-2-3)：从头结点的 `next` 开始，布尔变量控制空格。
- <a id="idea-126"></a>[单链表基本操作](#code-2-16)：默认带头结点；插入先找前驱，删除也先找前驱，再改 `next`。
- <a id="idea-012"></a>[合并两个有序链表](#code-2-4)：双指针比较，把较小节点接到结果尾部；不要新建数据，直接改链。
- <a id="idea-013"></a>[有序递增链表交集递减](#code-2-5)：两个递增链表双指针找相等值，相等时把节点头插到结果链表。
- <a id="idea-014"></a>[迭代逆置](#code-2-6)：三个指针 `pre, cur, nxt`，逐个把 `cur->next` 指回前驱。
- <a id="idea-015"></a>[递归逆置](#code-2-7)：递归到尾节点当新头，回溯时令 `p->next->next = p`，再断开 `p->next`。
- <a id="idea-016"></a>[链表分区](#code-2-8)：拆成两条链，各自尾插，最后把第一条尾巴接到第二条开头。
- <a id="idea-017"></a>[链表去重](#code-2-9)：保留第一次出现；`pre` 指向当前有效尾，发现重复就跳过并删除。
- <a id="idea-018"></a>[倒数第 k 个节点](#code-2-10)：快指针先走 k 步，然后快慢同走；快到尾时慢就是答案。
- <a id="idea-019"></a>[链表判环](#code-2-11)：快慢指针；快每次两步，慢每次一步，相遇则有环。
- <a id="idea-020"></a>[递归查找/删除第一个 x](#code-2-12)：查找是命中返回，否则递归后继；删除用引用指针改当前链头。
- <a id="idea-021"></a>[链表归并排序](#code-2-13)：快慢指针找中点断链，递归排序两半，再合并有序链表。
- <a id="idea-022"></a>[链串对称](#code-2-14)：链表不方便从后往前，先转数组或用栈，再双指针比较。
- <a id="idea-023"></a>[链串递归模式匹配](#code-2-15)：主串每个位置都尝试作为起点，子函数负责从当前位置连续匹配模式串。

### 栈、队列、双端队列、优先队列

- <a id="idea-024"></a>[顺序栈](#code-3-1)：`top = -1` 表示空；入栈先加 `top`，出栈先取再减。
- <a id="idea-025"></a>[括号匹配](#code-3-2)：左括号入栈；右括号必须和栈顶配对；最后栈空才合法。
- <a id="idea-026"></a>[中缀转后缀](#code-3-3)：数字直接输出，运算符按优先级弹栈，左括号只入栈，右括号弹到左括号。
- <a id="idea-027"></a>[后缀表达式求值](#code-3-4)：遇数字入栈，遇运算符弹两个数，注意先弹的是右操作数。
- <a id="idea-028"></a>[出栈序列是否合法](#code-3-5)：按入栈序列模拟，能弹就弹；目标弹不出来就是非法。
- <a id="idea-029"></a>[栈排序](#code-3-6)：用辅助栈维护有序；新元素插入辅助栈前，把比它大的元素倒回原栈。
- <a id="idea-030"></a>[退格问题](#code-3-7)：普通字符入结果，退格符删除结果最后一个字符。
- <a id="idea-031"></a>[用栈实现队列](#code-3-8)：入队进 `in` 栈，出队时若 `out` 空，就把 `in` 全倒入 `out`。
- <a id="idea-032"></a>[用队列实现栈](#code-3-9)：新元素入队后，把前面的元素轮转到队尾，让新元素来到队头。
- <a id="idea-033"></a>[循环队列](#code-3-10)：数组取模移动；用 `sz/count` 区分空和满最稳。
- <a id="idea-034"></a>[顺序栈倒置循环队列](#code-3-11)：队列元素依次出队入栈，再从栈出栈入队，顺序自然反转。
- <a id="idea-035"></a>[双端队列](#code-3-12)：数组开大一点，`head/tail` 从中间开始；前端插就 `--head`，后端插就 `tail++`。
- <a id="idea-036"></a>[优先队列](#code-3-13)：小根堆用 `greater<int>`；Top K、哈夫曼、Prim/Dijkstra 堆优化都靠它。
- <a id="idea-037"></a>[杨辉三角](#code-3-14)：队列保存上一行相邻关系；新值由前一个旧值和当前旧值相加得到。
- <a id="idea-038"></a>[日志最大值](#code-3-15)：一个普通栈存数据，一个最大值栈同步存当前最大值。
- <a id="idea-039"></a>[约瑟夫环](#code-3-16)：队列模拟报数；前 `m-1` 个移到队尾，第 `m` 个出队。
- <a id="idea-040"></a>[进制转换](#code-3-17)：不断取余入栈，再倒序弹出，得到高位到低位。

### 串、链串、Trie

- <a id="idea-042"></a>[递归查找字符](#code-4-1)：当前位置命中就返回，否则递归查下一个位置。
- <a id="idea-043"></a>[对称字符串](#code-4-2)：左右指针向中间收缩，不等立即失败。
- <a id="idea-044"></a>[Trie 前缀编码](#code-4-3)：插入每个编码时，若走到已结束节点，说明旧串是新串前缀；若全程没新节点，说明新串是旧串前缀。

### 二叉树

- <a id="idea-045"></a>[二叉树节点](#code-5-1)：统一 `data/lchild/rchild`，所有树题先保证建树函数能复用。
- <a id="idea-046"></a>[先序空标记建树](#code-5-2)：先读根，再递归建左、右；读到空标记立即返回空。
- <a id="idea-047"></a>[括号表示串建树](#code-5-3)：读到节点后，如果后面有 `(`，递归读左子树，逗号后递归读右子树，右括号收尾。
- <a id="idea-048"></a>[先序遍历](#code-5-4)：访问根的位置在递归左右子树之前。
- <a id="idea-049"></a>[中序遍历](#code-5-5)：访问根的位置夹在左、右子树之间。
- <a id="idea-050"></a>[后序遍历](#code-5-6)：访问根的位置在左右子树之后。
- <a id="idea-051"></a>[层序遍历](#code-5-7)：队列保存待访问节点，出队访问，左右孩子入队。
- <a id="idea-052"></a>[非递归中序](#code-5-8)：沿左链一路压栈，弹出访问，再转向右子树。
- <a id="idea-053"></a>[先序 + 中序建树](#code-5-9)：先序当前元素是根；根在中序中把区间切成左、右子树。
- <a id="idea-054"></a>[后序 + 中序建树](#code-5-10)：后序当前尾元素是根；从后往前建时要先建右子树，再建左子树。
- <a id="idea-055"></a>[层序 + 中序建树](#code-5-11)：层序第一个是根；剩余层序元素按其中序位置分进左右层序序列。
- <a id="idea-056"></a>[高度、节点数、叶子数](#code-5-12)：都按“空树返回 0，非空树由左右子树结果合成”递归。
- <a id="idea-057"></a>[单分支节点数](#code-5-13)：当前节点恰好只有一个孩子就计 1，再加左右子树答案。
- <a id="idea-058"></a>[镜像翻转](#code-5-14)：每个节点交换左右孩子，然后递归处理左右子树。
- <a id="idea-059"></a>[判断对称](#code-5-15)：左子树的左边要和右子树的右边比，左子树的右边要和右子树的左边比。
- <a id="idea-060"></a>[判断两棵树相同](#code-5-16)：空空相同，单空不同，非空则根值相同且左右子树分别相同。
- <a id="idea-061"></a>[复制二叉树](#code-5-17)：先新建当前节点，再递归复制左右子树。
- <a id="idea-062"></a>[倒序输出叶子](#code-5-18)：想从右到左，就先递归右子树，再递归左子树。
- <a id="idea-063"></a>[完全二叉树](#code-5-19)：层序遍历时，一旦遇到空节点，后面不能再遇到非空节点。
- <a id="idea-064"></a>[输出某节点子孙](#code-5-20)：先 DFS 找到值为 x 的节点，再从它的左右孩子开始遍历输出。
- <a id="idea-065"></a>[LCA](#code-5-21)：普通树中，若左右子树分别找到目标，当前根就是最近公共祖先；BST 可按大小关系向一边走。
- <a id="idea-066"></a>[序列化/反序列化](#code-5-22)：先序遍历输出值，空节点输出 `#`；读回时按相同顺序递归恢复。
- <a id="idea-121"></a>[先序输出分支节点](#code-5-23)：仍然是先序遍历，只在当前节点有孩子时输出。
- <a id="idea-122"></a>[三元组建树找根](#code-5-24)：按 `父 左 右` 建节点和孩子指针，同时标记谁当过孩子，最后没当过孩子的就是根。
- <a id="idea-123"></a>[树高边数](#code-5-25)：普通高度返回节点数；若题目要边数，答案是 `height(root) - 1`。
- <a id="idea-124"></a>[根到叶最大瓶颈值](#code-5-26)：DFS 过程中维护路径最小值，叶子处取所有路径最小值的最大者。
- <a id="idea-125"></a>[表达式树求值](#code-5-27)：叶子是数字直接返回，内部运算符先递归求左右子树再计算。

### BST、平衡判断、哈夫曼

- <a id="idea-067"></a>[BST 插入/查找/删除](#code-6-1)：小往左，大往右；删除双孩子节点时用右子树最小值替换。
- <a id="idea-068"></a>[判断 BST](#code-6-2)：中序遍历必须严格递增。
- <a id="idea-069"></a>[第 k 小元素](#code-6-3)：BST 中序就是升序，第 k 次访问就是答案。
- <a id="idea-070"></a>[BST 范围查询](#code-6-4)：小于范围左边界就不用走左，大于范围右边界就不用走右。
- <a id="idea-071"></a>[第一个大于 k](#code-6-5)：遇到大于 k 的节点先记答案，再去左边找更小的可行答案。
- <a id="idea-072"></a>[BST 转双向链表](#code-6-6)：中序遍历过程中，把前驱和当前节点互相连接。
- <a id="idea-073"></a>[判断平衡二叉树](#code-6-7)：递归返回高度；任意节点左右高度差超过 1 就失败。
- <a id="idea-074"></a>[哈夫曼 WPL](#code-6-8)：每次取两个最小权值合并，合并代价累加后再放回小根堆。

### 图

- <a id="idea-075"></a>[图统一存储](#code-7-1)：无权边也用 `edge{to, 1}`，所有建边都走 `addEdge`，需要无向边就双向加。
- <a id="idea-076"></a>[DFS](#code-7-2)：从起点一路递归深入，适合连通、路径、回路、枚举简单路径。
- <a id="idea-077"></a>[BFS](#code-7-3)：队列逐层扩展，适合路径存在、最短边数、层次问题。
- <a id="idea-078"></a>[判断路径存在](#code-7-4)：从起点 BFS/DFS，能访问到终点就存在。
- <a id="idea-079"></a>[连通分量数量](#code-7-5)：每遇到一个未访问点就启动一次 DFS，启动次数就是分量数。
- <a id="idea-080"></a>[所有简单路径](#code-7-6)：DFS 时进点标记，回溯时撤销标记，保证路径不重复点。
- <a id="idea-081"></a>[经过顶点 v 的回路](#code-7-7)：从 v 的邻点出发，看能不能不重复地回到 v。
- <a id="idea-082"></a>[最远顶点](#code-7-8)：无权图从起点 BFS 记录距离，取距离最大的点。
- <a id="idea-083"></a>[单词接龙](#code-7-9)：每次改一个字符生成邻居，用 BFS 找最少转换步数。
- <a id="idea-084"></a>[并查集](#code-7-10)：集合根代表连通块；`Find` 路径压缩，`unite` 合并两个根。
- <a id="idea-085"></a>[Prim](#code-7-11)：维护每个点到当前生成树的最小边权，每轮选最小边权点加入树。
- <a id="idea-086"></a>[Kruskal](#code-7-12)：边按权值排序，能连通两个不同集合的边就选，否则跳过防成环。
- <a id="idea-087"></a>[MST 唯一性](#code-7-13)：关键看同权边是否存在可替代选择；同权边越多越要小心。
- <a id="idea-088"></a>[Dijkstra](#code-7-14)：每轮确定一个当前最短距离点，用它松弛所有出边；只适合非负权。
- <a id="idea-089"></a>[次短路径](#code-7-15)：每个点维护最短 `d1` 和严格次短 `d2`，松弛时更新两档距离。
- <a id="idea-090"></a>[Floyd](#code-7-16)：`k` 放最外层，表示允许经过前 k 个点作为中转，逐步更新任意两点最短路。
- <a id="idea-091"></a>[拓扑排序 Kahn](#code-7-17)：入度为 0 的点先出队，删除它的出边；出不完说明有环。
- <a id="idea-092"></a>[DFS 拓扑与判环](#code-7-18)：访问状态 0/1/-1；遇到正在访问的点就是有环。
- <a id="idea-093"></a>[关键路径](#code-7-19)：先拓扑正推最早发生时间 `ve`，再逆拓扑反推最晚发生时间 `vl`，`ee == el` 是关键活动。

### 查找

- <a id="idea-094"></a>[顺序查找](#code-8-1)：从头扫到尾；哨兵版把目标放末尾，减少循环边界判断。
- <a id="idea-095"></a>[二分查找](#code-8-2)：有序数组中每次舍弃一半，注意 `l <= r` 和 `mid` 更新。
- <a id="idea-096"></a>[输出折半查找序列](#code-8-3)：每次检查 `mid` 时把 `a[mid]` 记录下来。
- <a id="idea-097"></a>[二分边界](#code-8-4)：`lower_bound` 找第一个 `>= x`，`upper_bound` 找第一个 `> x`。
- <a id="idea-098"></a>[分块查找](#code-8-5)：先在索引表里定位块，再在块内顺序查找。

### 排序与选择

- <a id="idea-102"></a>[直接插入排序](#code-9-1)：前面始终有序，把当前元素向前插入到正确位置。
- <a id="idea-103"></a>[折半插入排序](#code-9-2)：用二分找插入位置，再整体后移；只减少比较，不减少移动。
- <a id="idea-104"></a>[希尔排序](#code-9-3)：按增量分组做插入排序，增量不断缩小到 1。
- <a id="idea-105"></a>[冒泡排序](#code-9-4)：相邻逆序就交换；一轮没有交换说明已有序。
- <a id="idea-106"></a>[快速排序](#code-9-5)：挖坑法先保存基准，左右交替找小/大元素填坑，最后基准回坑。
- <a id="idea-107"></a>[简单选择排序](#code-9-6)：每轮从无序区选最小值，和当前位置交换。
- <a id="idea-108"></a>[堆排序](#code-9-7)：先建大根堆，再反复把堆顶最大值放到末尾并下滤调整。
- <a id="idea-109"></a>[判断大根堆](#code-9-7)：每个父节点都必须不小于左右孩子。
- <a id="idea-110"></a>[归并排序](#code-9-8)：递归分成两半，分别排好后线性合并。
- <a id="idea-111"></a>[非递归归并](#code-9-8)：段长从 1 开始翻倍，相邻有序段两两合并。
- <a id="idea-112"></a>[逆序对](#code-9-9)：归并时右边元素先出，说明它比左边剩余所有元素都小，贡献 `mid - i + 1`。
- <a id="idea-113"></a>[基数排序](#code-9-10)：按个位、十位、百位依次入桶出桶，低位到高位保持稳定。
- <a id="idea-114"></a>[计数排序](#code-9-11)：统计次数，做前缀和定位最终位置，倒序回填保证稳定。
- <a id="idea-115"></a>[Top K](#code-9-12)：求第 k 大用容量为 k 的小根堆，堆顶就是当前第 k 大。
- <a id="idea-116"></a>[快速选择](#code-9-12)：按快排分区，只递归包含目标下标的一侧。
- <a id="idea-117"></a>[稳定性](#code-9-13)：相等元素相对次序不变就是稳定；选择、希尔、快排、堆通常不稳定。

### DP、缓存、综合应用

- <a id="idea-118"></a>[最小编辑距离](#code-10-1)：`dp[i][j]` 表示前 i 个字符变成前 j 个字符的最小操作数，转移来自删、插、改。
- <a id="idea-119"></a>[LRU](#code-10-2)：映射表定位节点，双向链表维护最近使用顺序；访问就移到表头，淘汰表尾。
- <a id="idea-120"></a>[LFU](#code-10-3)：映射表存值、频率、时间；`set` 按频率和时间排序，淘汰最小者。

## 0. 考场总规则

- 数组、顺序表、排序查找：统一 `vector<int> a`。
- 链表、树、缓存里的节点：统一 `struct node`。
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

覆盖：顺序表插入删除查找、顺序表去重、顺序表逆置、有序顺序表合并、数组最大/次大、递增序列统计 k、颜色分类、三元组、主对角线、压缩矩阵相乘、查找有序数组对称点。

<a id="code-1-1"></a>

## 1.1 顺序表基本操作

思路：[顺序表基本操作](#idea-001)

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

<a id="code-1-2"></a>

## 1.2 顺序表去重

思路：[顺序表去重](#idea-002)

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

<a id="code-1-3"></a>

## 1.3 有序顺序表合并

思路：[有序顺序表合并](#idea-003)

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

<a id="code-1-4"></a>

## 1.4 最大值和次大值

思路：[最大值和次大值](#idea-004)

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

<a id="code-1-5"></a>

## 1.5 递增序列中 k 出现次数

思路：[递增序列中 k 出现次数](#idea-005)

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

<a id="code-1-6"></a>

## 1.6 颜色分类

思路：[颜色分类](#idea-006)

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

<a id="code-1-7"></a>

## 1.7 有序数组对称点

思路：[有序数组对称点](#idea-007)

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

<a id="code-1-8"></a>

## 1.8 压缩矩阵相乘

思路：[压缩矩阵相乘](#idea-008)

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

<a id="code-1-9"></a>

## 1.9 顺序表逆置

思路：[顺序表逆置](#idea-009)

```cpp
void reverseSeq(vector<int>& a) {
    int l = 0, r = a.size() - 1;
    while(l < r) {
        swap(a[l], a[r]);
        l++;
        r--;
    }
}
```

# 2. 链表与链串

覆盖：链表建表、两个有序链表合并、链表交集递减、链表逆置迭代/递归、链表分区、负整数前置、链表去重、链表重排、倒数第 k 个节点、判环、递归查找/删除第一个 x、排序链表、链串对称、链串递归模式匹配。

<a id="code-2-1"></a>

## 2.1 链表统一节点

```cpp
struct node {
    int data;
    node* next;
    node(int x = 0) : data(x), next(nullptr) {}
};
```

<a id="code-2-2"></a>

## 2.2 尾插建表

思路：[链表建表](#idea-010)

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

<a id="code-2-3"></a>

## 2.3 打印链表

思路：[打印链表](#idea-011)

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

<a id="code-2-4"></a>

## 2.4 合并两个有序链表

思路：[合并两个有序链表](#idea-012)

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

<a id="code-2-5"></a>

## 2.5 有序递增链表交集递减

思路：[有序递增链表交集递减](#idea-013)

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

<a id="code-2-6"></a>

## 2.6 迭代逆置

思路：[迭代逆置](#idea-014)

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

<a id="code-2-7"></a>

## 2.7 递归逆置

思路：[递归逆置](#idea-015)

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

<a id="code-2-8"></a>

## 2.8 链表分区：奇偶/负数前置

思路：[链表分区](#idea-016)

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

<a id="code-2-9"></a>

## 2.9 链表去重

思路：[链表去重](#idea-017)

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

<a id="code-2-10"></a>

## 2.10 倒数第 k 个节点

思路：[倒数第 k 个节点](#idea-018)

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

<a id="code-2-11"></a>

## 2.11 判环

思路：[链表判环](#idea-019)

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

<a id="code-2-12"></a>

## 2.12 递归查找/删除第一个 x

思路：[递归查找/删除第一个 x](#idea-020)

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

<a id="code-2-13"></a>

## 2.13 链表归并排序

思路：[链表归并排序](#idea-021)

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

<a id="code-2-14"></a>

## 2.14 链串对称

思路：[链串对称](#idea-022)

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

<a id="code-2-15"></a>

## 2.15 链串递归模式匹配

思路：[链串递归模式匹配](#idea-023)

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

<a id="code-2-16"></a>

## 2.16 单链表基本操作：插入删除遍历

思路：[单链表基本操作](#idea-126)

默认带头结点，位置从 1 开始。

```cpp
node* buildListByInput(int n) {
    node* head = new node();
    node* r = head;
    for(int i = 0; i < n; ++i) {
        int x;
        cin >> x;
        r->next = new node(x);
        r = r->next;
    }
    return head;
}

node* getPre(node* head, int pos) {
    node* p = head;
    for(int i = 1; p && i < pos; ++i)
        p = p->next;
    return p;
}

bool insertList(node* head, int pos, int x) {
    node* pre = getPre(head, pos);
    if(!pre) return false;
    node* s = new node(x);
    s->next = pre->next;
    pre->next = s;
    return true;
}

bool deleteList(node* head, int pos) {
    node* pre = getPre(head, pos);
    if(!pre || !pre->next) return false;
    node* p = pre->next;
    pre->next = p->next;
    delete p;
    return true;
}
```

# 3. 栈、队列、双端队列、优先队列

覆盖：顺序栈基本操作、括号匹配、中缀转后缀、后缀求值、出栈序列合法性、栈排序、退格问题、偶数出栈、用栈实现队列、用队列实现栈、循环队列、顺序栈倒置循环队列、双端队列、优先队列、杨辉三角、任务等待、超市模拟、日志最大值、约瑟夫环、十进制转任意进制。

<a id="code-3-1"></a>

## 3.1 顺序栈

思路：[顺序栈](#idea-024)

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

<a id="code-3-2"></a>

## 3.2 括号匹配

思路：[括号匹配](#idea-025)

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

<a id="code-3-3"></a>

## 3.3 中缀转后缀

思路：[中缀转后缀](#idea-026)

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

<a id="code-3-4"></a>

## 3.4 后缀表达式求值

思路：[后缀表达式求值](#idea-027)

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

<a id="code-3-5"></a>

## 3.5 出栈序列是否合法

思路：[出栈序列是否合法](#idea-028)

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

<a id="code-3-6"></a>

## 3.6 栈排序

思路：[栈排序](#idea-029)

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

<a id="code-3-7"></a>

## 3.7 退格问题

思路：[退格问题](#idea-030)

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

<a id="code-3-8"></a>

## 3.8 用栈实现队列

思路：[用栈实现队列](#idea-031)

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

<a id="code-3-9"></a>

## 3.9 用队列实现栈

思路：[用队列实现栈](#idea-032)

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

<a id="code-3-10"></a>

## 3.10 循环队列

思路：[循环队列](#idea-033)

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

<a id="code-3-11"></a>

## 3.11 顺序栈倒置循环队列

思路：[顺序栈倒置循环队列](#idea-034)

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

<a id="code-3-12"></a>

## 3.12 双端队列

思路：[双端队列](#idea-035)

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

<a id="code-3-13"></a>

## 3.13 优先队列

思路：[优先队列](#idea-036)

小根堆：

```cpp
priority_queue<int, vector<int>, greater<int>> pq;
```

大根堆：

```cpp
priority_queue<int> pq;
```

<a id="code-3-14"></a>

## 3.14 杨辉三角

思路：[杨辉三角](#idea-037)

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

<a id="code-3-15"></a>

## 3.15 日志分析：支持最大值

思路：[日志最大值](#idea-038)

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

<a id="code-3-16"></a>

## 3.16 约瑟夫环

思路：[约瑟夫环](#idea-039)

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

<a id="code-3-17"></a>

## 3.17 十进制转任意进制

思路：[进制转换](#idea-040)

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

# 4. 串、链串、Trie

覆盖：链串模式匹配、递归查找字符位置、判断对称字符串、编码前缀问题。

<a id="code-4-1"></a>

## 4.1 递归查找字符位置

思路：[递归查找字符](#idea-042)

```cpp
int findCharRec(const string& s, char c, int i = 0) {
    if(i >= (int)s.size()) return -1;
    if(s[i] == c) return i;
    return findCharRec(s, c, i + 1);
}
```

<a id="code-4-2"></a>

## 4.2 对称字符串

思路：[对称字符串](#idea-043)

```cpp
bool isPalindrome(const string& s) {
    int l = 0, r = s.size() - 1;
    while(l < r) {
        if(s[l++] != s[r--]) return false;
    }
    return true;
}
```

<a id="code-4-3"></a>

## 4.3 Trie 判断前缀编码

思路：[Trie 前缀编码](#idea-044)

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

覆盖：二叉树创建与遍历、括号表示串建树、先序空标记建树、先中建树、后中建树、层中建树、层序遍历、递归/非递归遍历、镜像、对称、相同树、深度高度、节点数、叶子数、倒序输出叶子、复制、序列化与反序列化、完全二叉树、值为 x 的所有子孙、LCA、先序输出分支节点、三元组建树找根、根到叶综合 DFS、表达式树求值。

<a id="code-5-1"></a>

## 5.1 二叉树统一节点

思路：[二叉树节点](#idea-045)

```cpp
struct node {
    int data;
    node *lchild, *rchild;
    node(int x = 0) : data(x), lchild(nullptr), rchild(nullptr) {}
};
```

如果题目数据是字符，把 `int data` 改成 `char data`，其余不变。

<a id="code-5-2"></a>

## 5.2 先序 + 空标记建树

思路：[先序空标记建树](#idea-046)

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

<a id="code-5-3"></a>

## 5.3 括号表示串建树

思路：[括号表示串建树](#idea-047)

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

<a id="code-5-4"></a>

## 5.4 先序遍历 NLR

思路：[先序遍历](#idea-048)

```cpp
void preOrder(node* root, vector<int>& ans) {
    if(!root) return;
    ans.push_back(root->data);
    preOrder(root->lchild, ans);
    preOrder(root->rchild, ans);
}
```

<a id="code-5-5"></a>

## 5.5 中序遍历 LNR

思路：[中序遍历](#idea-049)

```cpp
void inOrder(node* root, vector<int>& ans) {
    if(!root) return;
    inOrder(root->lchild, ans);
    ans.push_back(root->data);
    inOrder(root->rchild, ans);
}
```

<a id="code-5-6"></a>

## 5.6 后序遍历 LRN

思路：[后序遍历](#idea-050)

```cpp
void postOrder(node* root, vector<int>& ans) {
    if(!root) return;
    postOrder(root->lchild, ans);
    postOrder(root->rchild, ans);
    ans.push_back(root->data);
}
```

<a id="code-5-7"></a>

## 5.7 层序遍历

思路：[层序遍历](#idea-051)

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

<a id="code-5-8"></a>

## 5.8 非递归中序遍历

思路：[非递归中序](#idea-052)

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

<a id="code-5-9"></a>

## 5.9 先序 + 中序建树

思路：[先序 + 中序建树](#idea-053)

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

<a id="code-5-10"></a>

## 5.10 后序 + 中序建树

思路：[后序 + 中序建树](#idea-054)

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

<a id="code-5-11"></a>

## 5.11 层序 + 中序建树

思路：[层序 + 中序建树](#idea-055)

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

<a id="code-5-12"></a>

## 5.12 高度、节点数、叶子数

思路：[高度、节点数、叶子数](#idea-056)

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

<a id="code-5-13"></a>

## 5.13 单分支节点数

思路：[单分支节点数](#idea-057)

```cpp
int countSingle(node* root) {
    if(!root) return 0;
    int self = (root->lchild == nullptr) ^ (root->rchild == nullptr);
    return self + countSingle(root->lchild) + countSingle(root->rchild);
}
```

<a id="code-5-14"></a>

## 5.14 镜像翻转

思路：[镜像翻转](#idea-058)

```cpp
void mirror(node* root) {
    if(!root) return;
    swap(root->lchild, root->rchild);
    mirror(root->lchild);
    mirror(root->rchild);
}
```

<a id="code-5-15"></a>

## 5.15 判断对称

思路：[判断对称](#idea-059)

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

<a id="code-5-16"></a>

## 5.16 判断两棵树相同

思路：[判断两棵树相同](#idea-060)

```cpp
bool sameTree(node* a, node* b) {
    if(!a && !b) return true;
    if(!a || !b) return false;
    return a->data == b->data &&
           sameTree(a->lchild, b->lchild) &&
           sameTree(a->rchild, b->rchild);
}
```

<a id="code-5-17"></a>

## 5.17 复制二叉树

思路：[复制二叉树](#idea-061)

```cpp
node* copyTree(node* root) {
    if(!root) return nullptr;
    node* p = new node(root->data);
    p->lchild = copyTree(root->lchild);
    p->rchild = copyTree(root->rchild);
    return p;
}
```

<a id="code-5-18"></a>

## 5.18 倒序输出叶子

思路：[倒序输出叶子](#idea-062)

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

<a id="code-5-19"></a>

## 5.19 完全二叉树判断

思路：[完全二叉树](#idea-063)

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

<a id="code-5-20"></a>

## 5.20 值为 x 的节点所有子孙

思路：[输出某节点子孙](#idea-064)

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

<a id="code-5-21"></a>

## 5.21 最近公共祖先 LCA

思路：[LCA](#idea-065)

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

<a id="code-5-22"></a>

## 5.22 序列化与反序列化

思路：[序列化/反序列化](#idea-066)

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

<a id="code-5-23"></a>

## 5.23 先序输出分支节点

思路：[先序输出分支节点](#idea-121)

分支节点就是至少有一个孩子的节点。

```cpp
void prePrintBranch(node* root, bool& first) {
    if(!root) return;
    if(root->lchild || root->rchild) {
        if(!first) cout << " ";
        cout << root->data;
        first = false;
    }
    prePrintBranch(root->lchild, first);
    prePrintBranch(root->rchild, first);
}
```

<a id="code-5-24"></a>

## 5.24 三元组建树找根

思路：[三元组建树找根](#idea-122)

输入每行 `父 左 右`，空孩子常用 `#` 表示。

```cpp
const int TREE_N = 10005;
node* nodes[TREE_N];
bool isChild[TREE_N];

node* getTreeNode(int x) {
    if(!nodes[x]) nodes[x] = new node(x);
    return nodes[x];
}

node* buildByTriple(int n) {
    for(int i = 0; i < n; ++i) {
        string ps, ls, rs;
        cin >> ps >> ls >> rs;

        int p = stoi(ps);
        node* root = getTreeNode(p);

        if(ls != "#") {
            int l = stoi(ls);
            root->lchild = getTreeNode(l);
            isChild[l] = true;
        }
        if(rs != "#") {
            int r = stoi(rs);
            root->rchild = getTreeNode(r);
            isChild[r] = true;
        }
    }

    for(int i = 0; i < TREE_N; ++i)
        if(nodes[i] && !isChild[i]) return nodes[i];
    return nullptr;
}
```

<a id="code-5-25"></a>

## 5.25 树高边数

思路：[树高边数](#idea-123)

空树返回 `-1`，叶子返回 `0`，这样答案直接是边数。

```cpp
int heightByEdge(node* root) {
    if(!root) return -1;
    return max(heightByEdge(root->lchild),
               heightByEdge(root->rchild)) + 1;
}
```

<a id="code-5-26"></a>

## 5.26 根到叶最大瓶颈值

思路：[根到叶最大瓶颈值](#idea-124)

一条路径的瓶颈值是路径上最小节点值；题目要所有根到叶路径里瓶颈值最大的那条。

```cpp
int maxBottleneckPath(node* root, int curMin) {
    if(!root) return -1;
    curMin = min(curMin, root->data);

    if(!root->lchild && !root->rchild)
        return curMin;

    return max(maxBottleneckPath(root->lchild, curMin),
               maxBottleneckPath(root->rchild, curMin));
}

int solveBottleneck(node* root) {
    if(!root) return -1;
    return maxBottleneckPath(root, root->data);
}
```

<a id="code-5-27"></a>

## 5.27 表达式树求值

思路：[表达式树求值](#idea-125)

叶子是操作数，内部节点是运算符；本质就是后序遍历。

```cpp
int evalExprTree(node* root) {
    if(!root->lchild && !root->rchild)
        return root->data - '0';

    int a = evalExprTree(root->lchild);
    int b = evalExprTree(root->rchild);

    if(root->data == '+') return a + b;
    if(root->data == '-') return a - b;
    if(root->data == '*') return a * b;
    if(root->data == '/') return a / b;
    return 0;
}
```

多位数表达式树把 `data` 改成 `string`，叶子用 `stoi(root->data)`。

# 6. BST、平衡判断、哈夫曼树

覆盖：BST 插入删除查找、判断 BST、第 k 小、范围查询、第一个大于 k、BST 转双向链表、平衡二叉树判断、哈夫曼树 WPL、哈夫曼编码、编码前缀。

<a id="code-6-1"></a>

## 6.1 BST 插入、查找、删除

思路：[BST 插入/查找/删除](#idea-067)

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

<a id="code-6-2"></a>

## 6.2 判断是否为 BST

思路：[判断 BST](#idea-068)

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

<a id="code-6-3"></a>

## 6.3 第 k 小元素

思路：[第 k 小元素](#idea-069)

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

<a id="code-6-4"></a>

## 6.4 BST 范围查询

思路：[BST 范围查询](#idea-070)

```cpp
void rangeQuery(node* root, int L, int R, vector<int>& ans) {
    if(!root) return;
    if(root->data > L) rangeQuery(root->lchild, L, R, ans);
    if(root->data >= L && root->data <= R) ans.push_back(root->data);
    if(root->data < R) rangeQuery(root->rchild, L, R, ans);
}
```

<a id="code-6-5"></a>

## 6.5 第一个大于 k 的节点

思路：[第一个大于 k](#idea-071)

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

<a id="code-6-6"></a>

## 6.6 BST 转有序双向链表

思路：[BST 转双向链表](#idea-072)

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

<a id="code-6-7"></a>

## 6.7 判断平衡二叉树

思路：[判断平衡二叉树](#idea-073)

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

<a id="code-6-8"></a>

## 6.8 哈夫曼树 WPL

思路：[哈夫曼 WPL](#idea-074)

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

<a id="code-7-1"></a>

## 7.1 图统一写法

思路：[图统一存储](#idea-075)

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

<a id="code-7-2"></a>

## 7.2 DFS

思路：[DFS](#idea-076)

```cpp
void dfs(int u, vector<bool>& vis) {
    vis[u] = true;
    for(edge e : g[u]) {
        int v = e.to;
        if(!vis[v]) dfs(v, vis);
    }
}

void dfsPrint(int u, vector<bool>& vis, bool& first) {
    vis[u] = true;
    if(!first) cout << " ";
    cout << u;
    first = false;

    for(edge e : g[u]) {
        int v = e.to;
        if(!vis[v]) dfsPrint(v, vis, first);
    }
}
```

<a id="code-7-3"></a>

## 7.3 BFS

思路：[BFS](#idea-077)

```cpp
vector<int> bfsOrder(int s, int n) {
    vector<int> order;
    vector<bool> vis(n + 1, false);
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

void bfsPrint(int s, int n) {
    vector<bool> vis(n + 1, false);
    queue<int> q;
    bool first = true;

    vis[s] = true;
    q.push(s);
    while(!q.empty()) {
        int u = q.front();
        q.pop();

        if(!first) cout << " ";
        cout << u;
        first = false;

        for(edge e : g[u]) {
            int v = e.to;
            if(!vis[v]) {
                vis[v] = true;
                q.push(v);
            }
        }
    }
}
```

<a id="code-7-4"></a>

## 7.4 判断路径是否存在

思路：[判断路径存在](#idea-078)

```cpp
bool existPath(int s, int t, int n) {
    vector<bool> vis(n + 1, false);
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

<a id="code-7-5"></a>

## 7.5 连通分量数量

思路：[连通分量数量](#idea-079)

```cpp
int countComponent(int n) {
    vector<bool> vis(n + 1, false);
    int cnt = 0;
    for(int i = 1; i <= n; ++i) {
        if(!vis[i]) {
            cnt++;
            dfs(i, vis);
        }
    }
    return cnt;
}
```

无向图连通性：`countComponent(n) == 1`。

<a id="code-7-6"></a>

## 7.6 所有简单路径

思路：[所有简单路径](#idea-080)

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

<a id="code-7-7"></a>

## 7.7 判断经过顶点 v 的简单回路

思路：[经过顶点 v 的回路](#idea-081)

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

<a id="code-7-8"></a>

## 7.8 求距离顶点 v 最短路径中最远的顶点

思路：[最远顶点](#idea-082)

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

<a id="code-7-9"></a>

## 7.9 单词接龙 BFS

思路：[单词接龙](#idea-083)

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

<a id="code-7-10"></a>

## 7.10 并查集

思路：[并查集](#idea-084)

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

<a id="code-7-11"></a>

## 7.11 Prim

思路：[Prim](#idea-085)

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

<a id="code-7-12"></a>

## 7.12 Kruskal

思路：[Kruskal](#idea-086)

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

<a id="code-7-13"></a>

## 7.13 判断 MST 唯一性

思路：[MST 唯一性](#idea-087)

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

<a id="code-7-14"></a>

## 7.14 Dijkstra

思路：[Dijkstra](#idea-088)

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

<a id="code-7-15"></a>

## 7.15 次短路径

思路：[次短路径](#idea-089)

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

<a id="code-7-16"></a>

## 7.16 Floyd

思路：[Floyd](#idea-090)

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

<a id="code-7-17"></a>

## 7.17 拓扑排序 Kahn

思路：[拓扑排序 Kahn](#idea-091)

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

<a id="code-7-18"></a>

## 7.18 DFS 拓扑排序与判环

思路：[DFS 拓扑与判环](#idea-092)

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

<a id="code-7-19"></a>

## 7.19 关键路径

思路：[关键路径](#idea-093)

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

# 8. 查找

覆盖：顺序查找、二分查找、二分边界、输出折半查找序列、统计折半查找次数、分块查找。

<a id="code-8-1"></a>

## 8.1 顺序查找

思路：[顺序查找](#idea-094)

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

<a id="code-8-2"></a>

## 8.2 二分查找

思路：[二分查找](#idea-095)

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

<a id="code-8-3"></a>

## 8.3 输出折半查找序列

思路：[输出折半查找序列](#idea-096)

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

<a id="code-8-4"></a>

## 8.4 二分边界

思路：[二分边界](#idea-097)

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

<a id="code-8-5"></a>

## 8.5 分块查找

思路：[分块查找](#idea-098)

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

# 9. 排序与选择

覆盖：插入排序、折半插入排序、希尔排序、冒泡排序、快速排序、简单选择排序、堆排序、判断大根堆、归并排序递归/非递归、基数排序、计数排序、稳定性分析、Top K、快速选择、第 k 大、逆序对、排序链表。

<a id="code-9-1"></a>

## 9.1 直接插入排序

思路：[直接插入排序](#idea-102)

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

<a id="code-9-2"></a>

## 9.2 折半插入排序

思路：[折半插入排序](#idea-103)

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

<a id="code-9-3"></a>

## 9.3 希尔排序

思路：[希尔排序](#idea-104)

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

<a id="code-9-4"></a>

## 9.4 冒泡排序

思路：[冒泡排序](#idea-105)

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

<a id="code-9-5"></a>

## 9.5 快速排序：挖坑法

思路：[快速排序](#idea-106)

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

<a id="code-9-6"></a>

## 9.6 简单选择排序

思路：[简单选择排序](#idea-107)

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

<a id="code-9-7"></a>

## 9.7 堆排序

思路：[堆排序](#idea-108) / [判断大根堆](#idea-109)

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

<a id="code-9-8"></a>

## 9.8 归并排序

思路：[归并排序](#idea-110) / [非递归归并](#idea-111)

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

<a id="code-9-9"></a>

## 9.9 归并排序求逆序对

思路：[逆序对](#idea-112)

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

<a id="code-9-10"></a>

## 9.10 基数排序

思路：[基数排序](#idea-113)

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

<a id="code-9-11"></a>

## 9.11 计数排序

思路：[计数排序](#idea-114)

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

<a id="code-9-12"></a>

## 9.12 Top K 与第 k 大

思路：[Top K](#idea-115) / [快速选择](#idea-116)

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

<a id="code-9-13"></a>

## 9.13 排序稳定性

思路：[稳定性](#idea-117)

稳定：直接插入、折半插入、冒泡、归并、基数、计数。  
不稳定：希尔、快速、简单选择、堆排序。

# 10. DP、缓存、综合应用

覆盖：最小编辑距离、LRU、LFU、收入计算、图书馆统计、成绩排名系统、任务等待、超市模拟。

<a id="code-10-1"></a>

## 10.1 最小编辑距离

思路：[最小编辑距离](#idea-118)

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

<a id="code-10-2"></a>

## 10.2 LRU 缓存

思路：[LRU](#idea-119)

双向链表 + 映射表。节点仍叫 `node`。

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

<a id="code-10-3"></a>

## 10.3 LFU 缓存

思路：[LFU](#idea-120)

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

<a id="code-11-1"></a>

## 11.1 enhance 非 OJ 考点

- 栈与表达式：中缀转后缀、栈排序、用栈实现队列、用队列实现栈、表达式树求值。
- 链表：单链表基本操作、约瑟夫环、两个有序链表合并、链表分区、单链表迭代/递归逆置、链表判环、倒数第 k、排序链表。
- 顺序表和数组：有序顺序表合并、顺序表插删查、顺序表逆置、Top K、快速选择第 k 大。
- 二叉树：对称、镜像、深度、高度、序列化、相同树、叶子数、节点数、创建遍历、分支节点、三元组建树、树高边数、根到叶瓶颈值、表达式树求值、LCA、哈夫曼、重建、BST LCA、BST 转双向链表、BST 插删查、判断 BST、第 k 小、范围查询。
- 图：并查集、拓扑排序、关键路径、DFS、BFS、存储、连通性、MST 唯一性、Prim、Kruskal、次短路径、Floyd、Dijkstra、课程表、单词接龙。
- 查找排序：二分边界、顺序查找、堆排、归并递归/非递归、基数、计数、快排、逆序对、选择、希尔、冒泡、插入。
- 综合：最小编辑距离、LRU、LFU。

<a id="code-11-2"></a>

## 11.2 homework 非 OJ 考点

- 第 1-3 周：基础输入输出、数组模拟、顺序表去重、颜色分类、有序链表交集、链表重排、翻译映射、三元组、收入统计、倒数第 k、链表判环、链表去重、负数前置。
- 第 4 周：顺序栈操作、出栈序列合法性、偶数出栈、退格、括号匹配、中缀转后缀、后缀求值、栈排序、日志最大值。
- 第 5 周：循环队列、倒置队列、双端队列、优先队列、杨辉三角、缓冲区、任务等待、超市模拟。
- 第 6-7 周：压缩矩阵、链串模式匹配、递归查找字符、递归删除链表节点、进制转换。
- 第 8-10 周：二叉树建树遍历、非递归遍历、镜像、复制、倒序叶子、对称、叶子数、深度、子孙输出、完全二叉树。
- 第 11-12 周：DFS 邻接表、图连通性、哈夫曼 WPL、编码前缀 Trie、BFS 路径、连通分量、最远顶点、简单路径、简单回路。
- 第 13 周：折半查找序列和次数、数组对称点、BST 第一个大于 k、平衡二叉树判断。
- 第 15-16 周：希尔排序、快速排序、判断大根堆、基数排序、排序稳定性。
- 期末练习：顺序栈倒置循环队列、数组最大/次大、递增序列 k 次数、折半查找输出序列。

<a id="code-11-3"></a>

## 11.3 课测补充考点

- 队列完整操作模拟：本质是循环队列，维护 `front/rear/size`。
- 括号表示串建树 + 后序遍历：先建树，后序递归输出。
- 括号表示串建树 + 先序输出分支节点：先序遍历中只输出有孩子的节点。
- `父 左 右` 三元组建树：用数组保存节点，`isChild` 找根。
- 根到叶综合 DFS：树高边数、路径瓶颈最大值都属于“递归返回子树答案 / 沿路维护状态”。

# 12. 最短背诵顺序

1. `node` 链表：建表、逆置、合并、判环、倒数第 k。
2. 栈队列：顺序栈、循环队列、括号、中缀后缀、后缀求值。
3. `node` 二叉树：先序空标记、括号串、遍历、高度、叶子、镜像、对称、完全树。
4. 树重建：先中、后中、层中。
5. BST 和平衡判断：插入、删除、判断、范围、第 k 小、判断是否平衡。
6. 图统一模板：`edge/g/addEdge`，DFS、BFS、路径、连通分量。
7. 图算法：并查集、Prim、Kruskal、Dijkstra、Floyd、拓扑、关键路径。
8. 查找排序：二分、分块查找、快排、堆排、归并、基数、计数、Top K。
9. 综合：Trie、编辑距离、LRU、LFU。
