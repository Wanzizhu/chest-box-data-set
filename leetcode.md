### 大数组排序

合并 *k* 个排序链表，返回合并后的排序链表。请分析和描述算法的复杂度。其中 *N* 是节点的总数目

方法1：逐元素比较法

比较k个节点，利用multimap的key值的严格弱排序得到最小值的节点。

将最小值接在有序链表的后面。

时间复杂度：

![image-20200313212945894](../img/image-20200313212945894.png)

```c++
class Solution
{
public:
    ListNode *mergeKLists(vector<ListNode *> &lists)
    {
        ListNode *pre = new ListNode(0);
        ListNode *new_head = pre;
        auto iter = lists.begin();
        multimap<int, ListNode*> map_list;
        for(auto i=0;i!=lists.size();++i){
            if(lists[i])
                map_list.insert(std::make_pair(lists[i]->val, lists[i]));
        }
        while (map_list.size() > 1)
        {
            pre->next = map_list.begin()->second;
            pre = pre->next;
            ListNode* node = (map_list.begin()->second)->next;
            map_list.erase(map_list.begin());
            if(node!=NULL)
                map_list.insert(std::make_pair(node->val, node));
            else
                delete node;
        }
        if (map_list.size() == 1)
            pre->next = map_list.begin()->second;
        return new_head->next;
    }
};
```

方法2：分治法

一开始的思想是采用逐一两两合并链表，这样就把合并k个链表的问题变成合并2个链表`k-1`次。时间复杂度是O(kN)

怎么优化这个过程呢

![image-20200314113156555](../img/image-20200314113156555.png)

```c++
class Solution(object):
    def mergeKLists(self, lists):
        """
        :type lists: List[ListNode]
        :rtype: ListNode
        """
        amount = len(lists)
        interval = 1
        while interval < amount:
            for i in range(0, amount - interval, interval * 2):
                lists[i] = self.merge2Lists(lists[i], lists[i + interval])
            interval *= 2
        return lists[0] if amount > 0 else lists

    def merge2Lists(self, l1, l2):
        head = point = ListNode(0)
        while l1 and l2:
            if l1.val <= l2.val:
                point.next = l1
                l1 = l1.next
            else:
                point.next = l2
                l2 = l1
                l1 = point.next.next
            point = point.next
        if not l1:
            point.next=l2
        else:
            point.next=l1
        return head.next
```

### 链表排序

#### 冒泡排序：

链表的冒泡的重点在于记录最后一个值。

```c
void bubbleSort (int arr[], int len)
{

	int i, j,temp;
	Boolean exchanged = true;
	
	for (i=0; exchanged && i<len-1; i++) /* 外迴圈為排序趟數，len個數進行len-1趟,只有交換過,exchanged值為true才有執行迴圈的必要,否則exchanged值為false不執行迴圈 */
		for (j=0; j<len-1-i; j++) 
		{ /* 內迴圈為每趟比較的次數，第i趟比較len-i次  */
		
		exchanged = false;
		
			if (arr[j] > arr[j+1])
			{ /* 相鄰元素比較，若逆序則互換（升序為左大於右，逆序反之） */
				temp = arr[j];
				arr[j] = arr[j+1];
				arr[j+1] = temp;
				exchanged = true; /*只有數值互換過, exchanged才會從false變成true,否則數列已經排序完成,exchanged值仍然為false,沒必要排序 */
			}
		}
}
```



```c++
struct ListNode
{
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(NULL) {}
};

class Solution
{
public:
    ListNode *sortList(ListNode *head)
    {
        if (head == 0 || head->next == 0)
            return head;
        ListNode *tail = 0;
        bool exchanged=false;
        while (tail != head->next)
        {
            ListNode *first = head;
            ListNode *second = head->next;
            while (second != tail)
            {
                if (first->val > second->val){
                    std::swap(first->val, second->val);
                    exchanged=true;
                }
                first = first->next;
                second = second->next;
            }
            if(!exchanged)
                break;
            tail = first;
        }
        return head;
    }
};
```

#### 归并排序(递归思路)

重点：双路归并和 cut 断链

![image-20200314220106344](../img/image-20200314220106344.png)

```c++

class Solution
{
public:
    ListNode *sortList(ListNode *head)
    {
        if (head == 0 || head->next == 0)
            return head;
        ListNode *slow = head;
        //这里fast比slow先走一步的原因是如果fast和slow一开始都指向head，当长度为2时，slow的位置不对，这样会造成死循环。而fast比slow先走一步保证了slow会指向中点，或者中点左边。
        ListNode *fast = head->next;
        while (fast && fast->next)
        {
            fast = fast->next->next;
            slow = slow->next;
        }
        ListNode *head2 = slow->next;
        slow->next=0;
        ListNode *left = sortList(head);
        ListNode *right = sortList(head2);
        return mergeTwoList(left, right);
    }

    ListNode *mergeTwoList(ListNode *head1, ListNode *head2)
    {
        ListNode *dummpy = new ListNode(-1);
        ListNode *node = dummpy;
        while (head1 && head2)
        {
            if (head1->val <= head2->val)
            {
                node->next = head1;
                head1 = head1->next;
            }
            else
            {
                node->next = head2;
                head2 = head2->next;
            }
            node = node->next;
        }
        if (head1)
            node->next = head1;
        else if (head2)
            node->next = head2;
        return dummpy->next;
    }
};
```
#### 归并排序（非递归思路）

![image-20200315090606869](../img/image-20200315090606869.png)

```c++
current = dummy.next;
tail = dummy;
for (step = 1; step < length; step *= 2) {
	while (current) {
		// left->@->@->@->@->@->@->null
		left = current;

		// left->@->@->null   right->@->@->@->@->null
		right = cut(current, step); // 将 current 切掉前 step 个头切下来。

		// left->@->@->null   right->@->@->null   current->@->@->null
		current = cut(right, step); // 将 right 切掉前 step 个头切下来。
		
		// dummy.next -> @->@->@->@->null，最后一个节点是 tail，始终记录
		//                        ^
		//                        tail
		tail.next = merge(left, right);
		while (tail->next) tail = tail->next; // 保持 tail 为尾部
	}
}
```

```c++

class Solution
{
public:
    ListNode *sortList(ListNode *head)
    {
        if (head == 0 || head->next == 0)
            return head;
        size_t length = 0, gap ;
        ListNode *node = head;
        while (node)
        {
            node = node->next;
            ++length;
        }

        ListNode *dummy = new ListNode(-1);
        dummy->next=head;
        for (gap=1;gap < length;gap<<=1)
        {
            ListNode *pre = dummy;
            ListNode *current = dummy->next;
            while (current)
            {
                ListNode *left = current;
                ListNode *right = cut(current, gap);
                current = cut(right, gap);

                pre->next = mergeTwoList(left, right);
                while(pre->next)
                    pre=pre->next;
            }
            
        }
        return dummy->next;
    }

    ListNode *cut(ListNode *head, size_t n)
    {
        ListNode *node = head;
        while (--n && node)
        {
            node = node->next;	
        }
        if (node == 0)
            return 0;
        auto next_node = node->next;
        node->next = 0;
        return next_node;
    }

    ListNode *mergeTwoList(ListNode *head1, ListNode *head2)
    {
        ListNode *dummpy = new ListNode(-1);
        ListNode *node = dummpy;
        while (head1 && head2)
        {
            if (head1->val <= head2->val)
            {
                node->next = head1;
                head1 = head1->next;
            }
            else
            {
                node->next = head2;
                head2 = head2->next;
            }
            node = node->next;
        }
        if (head1)
            node->next = head1;
        else if (head2)
            node->next = head2;
        return dummpy->next;
    }
};
```

#### 对stl中的list容器进行排序

源码

```c++
void sort()
{
    std::list<int> carry;
    std::list<int> counter[64];
    std::list<int> lst = {9, 8, 1, 2, 3};
    int fill = 0;
    while (!lst.empty())
    {
        carry.splice(carry.begin(), lst, lst.begin());
        int i = 0;
        while (i < fill && !counter[i].empty())
        {
            counter[i].merge(carry);
            carry.swap(counter[i++]);
        }
        carry.swap(counter[i]);
        if (i == fill)
            ++fill;
    }
    for (int i = 1; i < fill; ++i)
        counter[i].merge(counter[i - 1]);
    lst.swap(counter[fill - 1]);
}
```

自己写的

```c++

void print(const std::list<int> &p, const char *prefix)
{
    std::cout << prefix << std::endl;
    std::copy(p.begin(), p.end(), std::ostream_iterator<int>(std::cout, " "));
}

int cut(std::list<int> &newlist, std::list<int> &oldlist, int step)
{
    std::list<int>::iterator last = oldlist.begin();
    if (step <= oldlist.size())
        std::advance(last, step);
    else
        last = oldlist.end();
    newlist.splice(newlist.begin(), oldlist, oldlist.begin(), last);
    return 0;
}

void sort()
{
    std::list<int> lst = {9, 8, 1, 2, 3, 2, 0, 1};
    size_t length = lst.size();
    std::list<int> carry;
    std::list<int> temp = lst;
    for (int step = 1; step < length; step *= 2)
    {
        while (!temp.empty())
        {
            std::list<int> left, right;
            cut(left, temp, step);
            cut(right, temp, step);
            left.merge(right);
            carry.splice(carry.end(), left, left.begin(), left.end());
        }
        temp.swap(carry);
    }
    std::copy(temp.begin(), temp.end(), std::ostream_iterator<int>(std::cout, " "));
    return;
}

```



### 反转链表

```c++
//递归
ListNode *list_reverse(ListNode *head)
{
    ListNode *pre = 0;
    ListNode *cur = head;
    while (cur)
    {
        ListNode *aft = cur->next;
        cur->next = pre;
        pre = cur;
        cur = aft;
    }
    return pre;
}
```

```c++
//循环
ListNode *list_reverse2(ListNode *head)
{
    if (head == 0 || head->next == 0)
        return head;
    ListNode *new_head = list_reverse2(head->next);
    head->next->next = head;
    head->next = 0;
    return new_head;
}
```

```c++
//下面的反转链表的方法是不断移动第一个节点，将后面的节点移到pre的next节点上，保持pre的next节点一直是头节点，可用于反转链表中k个节点而不断开链表。
ListNode *list_reverse3(ListNode *head)
{
    if (head == 0 || head->next == 0)
        return head;
    ListNode *pre = new ListNode(-1);
    pre->next = head;
    ListNode *cur = pre->next;
    while (cur->next)
    {
        ListNode *aft = cur->next;
        cur->next = aft->next;
        aft->next = pre->next;
        pre->next = aft;
    }
    return pre->next;
}
```

### 八皇后

递归法：

关键点在于：把棋盘存储在一个N维数组a[N]中，其中数组下标代表row，数组元素代表col。判断是否冲突就变得十分简单，首先行冲突就不存在了，列冲突可以通过在row下标前是否有数组元素和其相等来判断，对角线冲突可表示为：
$$
abs(row-i)=abs(col-a[i])
$$

```c++
class Solution
{
public:
    vector<vector<string>> ans;
    vector<vector<string>> solveNQueens(int n)
    {
        vector<int> vet(n);
        int row = 0;
        f(n, row, vet);
        // std::copy(std::begin(vet), std::end(vet), std::ostream_iterator<int>(std::cout, " "));
        return ans;
    }

    int f(int length, int row, vector<int> &vet)
    {
        if (row >= length)
        {
            vector<string> tmp(length, string(length, '.'));
            for (auto i = 0; i != vet.size(); ++i)
            {
                tmp[i][vet[i]] = 'Q';
            }
            ans.push_back(tmp);
            return 0;
        }
        for (int col = 0; col < length; ++col)
        {
            if (not_access(row, col, vet))
            {
                vet[row] = col;
                f(length, row + 1, vet);
                // vet.pop_back();
            }
        }
        return 0;
    }

    bool not_access(int row, int col, vector<int> vet)
    {
        for (int i = 0; i < row; ++i)
        {
            if (vet[i] == col || abs(row - i) == abs(col - vet[i]))
                return false;
        }
        return true;
    }
};
```

 用循环代替递归法

关键：如何回溯？可以通过退回到上一行来回溯。

曾错点：

- 没有判断当row=0时，--row会出现数组越界的情况。

```c++

class Solution
{
public:
    vector<vector<string>> ans;
    vector<vector<string>> solveNQueens(int n)
    {
        vector<int> vet(n, 0);
        int row = 0, cnt = 0;
        while (row >= 0)
        {
            while (vet[row] < n && !is_valid(row, vet[row], vet))
            {
                ++vet[row];
            }

            if (vet[row] >= n)
            {
                if (row == 0)
                    break;
                --row;
                ++vet[row];
            }
            else if (row == n - 1)
            {
                format_print(vet);
                ++vet[row];
            }
            else
                vet[++row] = 0;
        }
        return ans;
    }

    bool is_valid(int row, int col, const vector<int> &vet)
    {
        for (int i = 0; i < row; ++i)
        {
            if (vet[i] == col || abs(row - i) == abs(col - vet[i]))
                return false;
        }
        return true;
    }

    void format_print(const vector<int> vet){
        const int n=vet.size();
        vector<string> tmp(n, string(n, '.'));
        for (auto i = 0; i != vet.size(); ++i)
        {
            tmp[i][vet[i]] = 'Q';
        }
        ans.push_back(tmp);
    }
};
```

```c++

class Solution
{
public:
    int upperlim = 0;
    int cnt = 0;
    vector<vector<string>> ans;
    vector<vector<string>> solveNQueens(int n)
    {
        upperlim = (1 << n) - 1;
        r(0, 0, 0);
        std::cout << upperlim << "the count" << cnt << std::endl;
        return ans;
    }

    int r(int row, int ld, int rd)
    {
        int pos, p;
        if (row == upperlim)
        {
            std::cout << bitset<sizeof(int) * 8>(row) << endl;
            ++cnt;
        }
        else
        {
            pos = upperlim & (~(row | ld | rd));
            while (pos)
            {
                p = pos & (~pos + 1);
                pos -= p;
                r(row | p, (ld | p) << 1, (rd | p) >> 1);
            }
        }
        return 0;
    }
};
```





### 二进制的应用（面试官:如何用最少的老鼠试出有毒的牛奶？）

![image-20200318222626552](../img/image-20200318222626552.png)



![image-20200318222643202](../img/image-20200318222643202.png)

![image-20200318222700158](../img/image-20200318222700158.png)

### 环路中的机器人

![image-20200321222935374](../img/image-20200321222935374.png)

![image-20200321223014757](../img/image-20200321223014757.png)

关键点：

1、怎么表示方向

> 显然可以用四个数字代表方向，但是如何表示诸如，左左是下，右右是下，左左左是左这样的变换规则呢？
>
> 可以令方向为[0,1,2,3]，0表示向上，1表示左，2表示下，3表示右，然后进行取余操作。

2、怎么判断是否是环

> 假设经过一次操作（所以指令），到达位置(x,y),如果(x,y)==(0,0),是true。
>
> 如果不满足(x,y)==(0,0)，假设原始方向是上：
>
> 若此时为右，则可以回去，同样为左也可以回去。
>
> 如果此时为下，也可以回去。为上则不可以。
>
> 所以不是环的条件为，(x,y)！=(0,0) && current dire == initial dire

### 组合

题目：试编写一个递归函数，用来输出n 个元素的所有子集。例如，三个元素 {a, b, c} 的所有
子集是： { }（空集） ， {a}, {b}, {c}, {a, b}, {a, c}, {b, c} 和{a, b, c}。

```c++

int f(std::vector<int> vet, std::vector<int> ans, int start, int length)
{
    if (length == 0)
    {
        std::copy(std::begin(ans), std::end(ans), std::ostream_iterator<int>(std::cout, " "));
        std::cout << "\n";
        return 0;
    }
    else
    {
        for (auto i = start; i + length - 1 < vet.size(); ++i)
        {
            ans.push_back(vet[i]);
            f(vet, ans, i + 1, length - 1);
            ans.pop_back();
        }
    }
}

int main()
{
    std::vector<int> ans;
    std::vector<int> vet = {1, 2, 3, 4};
    for (auto i = 1; i <= vet.size(); ++i)
    {

        f(vet, ans, 0, i);
    }
    return 0;
}
```

### 柱状图中最大的矩形

![image-20200327083953203](../img/image-20200327083953203.png)

思路的关键点：

1.递增栈的思想，维持一个递增的栈，遇到比当前栈定的要小的，则将栈顶元素出栈直至维持递增栈。

2.如果根据递增栈计算面积？计算面积需要高度和宽度，高度是已知的，宽度怎么算？如图

维持了1,5,6的递增栈，当前即将入栈的2索引为4，首先将6出栈，宽度为4-2-1,2为6左边第一个高度小于6的柱子，再将5出栈，宽度为4-1-1。**因此元素的索引很关键，我们在栈里面存放元素的索引，而不是元素值。**

当第i个柱子进栈时，如果栈顶柱子（此处记作柱子A）的高度低于或等于第i个柱子，则第i个柱子进栈；
如果高于第i个柱子，则出栈，并计算以柱子A为高的矩形最大面积。

高度：就是柱子A的高度
右边沿：正好是i（由于单调栈的性质，第i个柱子就是右边第一个矮于A的柱子）
左边沿：单调栈中紧邻A的柱子。（如果A已经出栈，那么左边沿就是A出栈后的栈顶）而且是该柱子的右边，所以要+1.
当然，特殊情况是第二个柱子比第一个小，这样第一个出栈，栈就为空了，这时候宽度就是i-0=i。

3.最后我们会得到一个递增栈，为了让所有元素都出栈，**我们可以在原始数组最后加一个0，这样就可以保证让所有元素都出栈。**

```c++
int largestRectangleArea(std::vector<int> &heights)
{
    int max_area = 0;
    heights.push_back(0);
    std::stack<int> sta;
    const int length = heights.size();
    for (auto i = 0; i < length; ++i)
    {
        while (!sta.empty() && heights[sta.top()] > heights[i])
        {
            int cur_index = sta.top();
            sta.pop();
            int cur_area = heights[cur_index] * (sta.empty() ? i : (i - sta.top() - 1));
            max_area = std::max(cur_area, max_area);
        }
        sta.push(i);
    }
    return max_area;
}
```

- 分治法

![image-20200327094622665](../img/image-20200327094622665.png)

![image-20200327094714539](../img/image-20200327094714539.png)

- 优化的分治法

![image-20200327094737240](../img/image-20200327094737240.png)

### LRU机制

```c++

struct Node
{
    int key, value;
    Node *pre, *next;
    Node(int x, int y) : key(x), value(y), pre(nullptr), next(nullptr){};
};

class LRUCache
{
private:
    int cap;
    Node *head;
    Node *tail;
    std::map<int, Node *> keymap;

public:
    LRUCache(int capacity) : cap(capacity), head(nullptr), tail(nullptr) {}

    void removeAndInsert(Node *node)
    {
        if (node == head)
            return;
        else if (node == tail)
        {
            tail = tail->pre;
            if (tail)
                tail->next = 0;
        }
        else
        {
            node->pre->next = node->next;
            node->next->pre = node->pre;
        }
        node->next = head;
        node->pre = 0;
        head->pre = node;
        head = node;
    }

    int get(int key)
    {
        auto node_iter = keymap.find(key);
        if (node_iter != keymap.end())
        {
            removeAndInsert(node_iter->second);
            return node_iter->second->value;
        }
        return -1;
    }

    void put(int key, int value)
    {
        if (head == 0)
        {
            Node *temp = new Node(key, value);
            head = temp;
            tail = head;
            keymap.insert(std::make_pair(key, temp));
        }
        else
        {
            std::map<int, Node *>::iterator node_iter = keymap.find(key);
            if (node_iter != keymap.end())
            {
                node_iter->second->value = value;
                removeAndInsert(node_iter->second);
            }
            else
            {
                Node *temp = new Node(key, value);
                if (keymap.size() == cap)
                {
                    keymap.erase(tail->key);

                    if (head == tail)
                        head = 0;
                    Node *del = tail;
                    tail = tail->pre;
                    if (tail)
                        tail->next = 0;
                    delete del;
                }
                keymap.insert(std::make_pair(key, temp));
                if (head != 0)
                {
                    temp->next = head;
                    head->pre = temp;
                }
                head = temp;
            }
        }
    }
};

```

```c++
class LRUCache
{
private:
    int cap;
    std::list<std::pair<int, int>> lst;
    std::unordered_map<int, std::list<std::pair<int, int>>::iterator> keymap;

public:
    LRUCache(int capacity = 0) : cap(capacity) {}

    int get(int key)
    {
        auto iter = keymap.find(key);
        if (iter != keymap.end())
        {
            lst.splice(lst.begin(), lst, iter->second);
            //this is unecessary,because iterator in keymap still take effct.
            // keymap[key] = lst.begin(); 
            std::cout << iter->second->second << " ";
            return iter->second->second;
        }
        else
        {
            std::cout << -1 << " ";
            return -1;
        }
    }

    void put(int key, int value)
    {
        auto iter = keymap.find(key);
        if (iter != keymap.end())
        {
            iter->second->second = value;
            lst.splice(lst.begin(), lst, iter->second);
            // keymap[key] = lst.begin();
        }
        else
        {
            if (keymap.size() == cap)
            {
                std::pair<int, int> to_del = lst.back();
                lst.pop_back();
                keymap.erase(to_del.first);
            }
            lst.push_front(std::make_pair(key, value));
            keymap[key] = lst.begin();
        }
    }
};
```

### LFU机制

https://www.jianshu.com/p/1f8e36285539

```c++

struct FreqNode;
struct KeyNode
{
    int key, value;
    KeyNode *pre, *next;
    FreqNode *freq;
    KeyNode(int x, int y) : key(x), value(y), pre(nullptr), next(nullptr), freq(nullptr){};
    KeyNode(int x, int y, KeyNode *p, KeyNode *n, FreqNode *f) : key(x), value(y), pre(p), next(n), freq(f){};
};

struct FreqNode
{
    int count;
    FreqNode *pre, *next;
    KeyNode *head, *tail;
    FreqNode() : count(1), pre(nullptr), next(nullptr), tail(nullptr), head(nullptr){};
    FreqNode(int cnt, FreqNode *p, FreqNode *n, KeyNode *h, KeyNode *t) : count(cnt), pre(p), next(n), head(h), tail(t){};
};

class LFUCache
{
public:
private:
    int cap;
    std::map<int, KeyNode *> keymap;
    FreqNode *freqhead, *freqtail;

public:
    LFUCache(int capacity) : cap(capacity), freqhead(nullptr), freqtail(nullptr){};

    void deleteFromFreqLink(FreqNode *freqnode)
    {
        // FreqNode *delfreq = freqnode;
        if (freqnode == freqhead)
        {
            freqhead = freqhead->next;
            if (freqhead)
                freqhead->pre = 0;
        }
        else if (freqnode == freqtail)
        {
            freqtail = freqtail->pre;
            if (freqtail)
                freqtail->next = 0;
        }
        else
        {
            freqnode->pre->next = freqnode->next;
            freqnode->next->pre = freqnode->pre;
        }
        // delete freqnode;
    }

    void deleteFromFreqnode(KeyNode *keynode)
    {
        FreqNode *cur_freq = keynode->freq;
        if (keynode == cur_freq->head)
        {
            cur_freq->head = cur_freq->head->next;
            if (cur_freq->head)
                cur_freq->head->pre = 0;
        }
        else if (keynode == cur_freq->tail)
        {
            cur_freq->tail = cur_freq->tail->pre;
            if (cur_freq->tail)
                cur_freq->tail->next = 0;
        }
        else
        {
            keynode->pre->next = keynode->next;
            keynode->next->pre = keynode->pre;
        }
        keynode->pre = 0;
        keynode->next = 0;

        if (cur_freq->head == 0)
        {
            deleteFromFreqLink(cur_freq);
        }
    }

    void oneStepUp(KeyNode *keynode)
    {
        bool singleNode = false;
        FreqNode *cur_freq = keynode->freq;
        if (cur_freq->head == cur_freq->tail)
            singleNode = true;

        if (cur_freq->next != NULL)
        {
            FreqNode *next_freq = cur_freq->next;
            if (next_freq->count == cur_freq->count + 1)
            {
                deleteFromFreqnode(keynode);
                keynode->freq = next_freq;
                next_freq->head->pre = keynode;
                keynode->next = next_freq->head;
                next_freq->head = keynode;

                if (cur_freq->head == 0)
                    delete cur_freq;
            }
            else if (next_freq->count > cur_freq->count + 1)
            {
                if (!singleNode)
                {
                    deleteFromFreqnode(keynode);
                    FreqNode *newFreq = new FreqNode(cur_freq->count + 1, cur_freq, next_freq, keynode, keynode);
                    keynode->freq = newFreq;
                    cur_freq->next = newFreq;
                    next_freq->pre = newFreq;
                }
                else
                    ++cur_freq->count;
            }
        }
        else
        {
            if (singleNode)
                ++cur_freq->count;
            else
            {
                deleteFromFreqnode(keynode);
                FreqNode *newFreq = new FreqNode(cur_freq->count + 1, cur_freq, cur_freq->next, keynode, keynode);
                keynode->freq = newFreq;
                cur_freq->next = newFreq;
            }
        }
    }

    int get(int key)
    {
        auto key_iter = keymap.find(key);
        if (key_iter != keymap.end())
        {
            oneStepUp(key_iter->second);
            return key_iter->second->value;
        }
        else
            return -1;
    }

    void put(int key, int value)
    {
        if (cap <= 0)
            return;
        auto key_iter = keymap.find(key);
        if (key_iter == keymap.end())
        {
            KeyNode *newKeyNode = new KeyNode(key, value);
            if (keymap.size() == cap)
            {
                KeyNode *delkeytail = freqhead->tail;
                keymap.erase(delkeytail->key);
                deleteFromFreqnode(delkeytail);
            }

            keymap.insert(std::make_pair(key, newKeyNode));

            //freq link is empty or first freqcount>1
            if (freqhead == NULL)
            {
                FreqNode *newFreqNode = new FreqNode(1, NULL, NULL, newKeyNode, newKeyNode);
                newKeyNode->freq = newFreqNode;
                freqhead = freqtail = newFreqNode;
            }
            else if (freqhead->count > 1)
            {
                FreqNode *newFreqNode = new FreqNode(1, NULL, freqhead, newKeyNode, newKeyNode);
                newKeyNode->freq = newFreqNode;
                freqhead->pre = newFreqNode;
                freqhead = newFreqNode;
            }
            else if (freqhead->count == 1)
            {
                newKeyNode->freq = freqhead;
                newKeyNode->next = freqhead->head;
                freqhead->head->pre = newKeyNode;
                freqhead->head = newKeyNode;
            }
        }
        else
        {
            key_iter->second->value = value;
            oneStepUp(key_iter->second);
        }
    }
};

```

### 箱子排序

**链表的**箱子排序

![image-20200402162409734](../img/image-20200402162409734.png)

```c++

// Definition for singly-linked list.
struct ListNode
{
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(NULL) {}
};

class Solution
{
public:
    ListNode *sortList(ListNode *head)
    {

        if (head == NULL)
            return NULL;
        //find the sort range
        ListNode *p = head;
        int maxrange = std::numeric_limits<int>::min();
        int minrange = std::numeric_limits<int>::max();
        while (p)
        {
            maxrange = std::max(maxrange, p->val);
            minrange = std::min(minrange, p->val);
            p = p->next;
        }
        long long range = maxrange - minrange ;
        p = head;

        ListNode **bottom, **top;
        bottom = new ListNode *[range + 1]();
        top = new ListNode *[range + 1]();

        //get into bins
        while (p)
        {
            int bin = p->val - minrange;
            if (bottom[bin] == NULL)
            {
                bottom[bin] = top[bin] = p;
            }
            else
            {
                top[bin]->next = p;
                top[bin] = p;
            }
            p = p->next;
        }

        ListNode *newhead = NULL;
        ListNode *temp;
        for (int i = 0; i <= range; ++i)
        {
            if (bottom[i] != NULL)
            {
                if (newhead == NULL)
                    newhead = bottom[i];
                else
                    temp->next = bottom[i];
                temp = top[i];
            }
        }
        if (temp)
            temp->next = 0;
        delete[] bottom;
        delete[] top;
        return newhead;
    }
};
```

**应用于数组排序**

```c
#include<iostream>
#include<algorithm>
using namespace std;
 
int main()
{
	int data[] = {7,11,12,3,4,9,77,2,33,4,7,10,88,101,78,13,34,25,58,8,9};
	//找到最大值和最小值
	int max = *(max_element(data,data+sizeof(data)/sizeof(data[0])));
	int min = *(min_element(data,data+sizeof(data)/sizeof(data[0])));
	//建立连续有序的箱子序列
	int * Order = new int[max-min+1];
	for (int i = 0; i < max-min+1; i++)
	{
		Order[i] = 0;
	}
 
	//将元素对号入座放入箱子
	for (int i = 0; i < sizeof(data)/sizeof(data[0]); i++)
	{
		Order[data[i]-min]++;
	}
 
	//略过空箱子将有序的元素输出
	for (int i = 0; i < max-min+1; i++)
	{
		if(0==Order[i]) continue;
		for (int j = 0; j < Order[i]; j++)
		{
			cout<<i+min<<" ";
		}
	}
	cout<<endl;
}
```

### 基数排序

相当于是多次桶排序，对于如何选取基数r

定理：对于n个`b-bit`位的元素，当b<lgn时，r=b可以得到最小复杂度，当b>lgn时，r=lgn有最小复杂度。

#### 对数组

```c++
#include <iostream>
#include <algorithm>
#include <iterator>
#include <string>
#include <cmath>
#include <vector>
using namespace std;

/*
* 打印数组
*/
void printArray(int array[], int length)
{
    for (int i = 0; i < length; ++i)
    {
        cout << array[i] << " ";
    }
    cout << endl;
}
/*
*求数据的最大位数,决定排序次数
*/
int maxbit(int data[], int n)
{
    int d = 1; //保存最大的位数
    int p = 10;
    for (int i = 0; i < n; ++i)
    {
        while (data[i] >= p)
        {
            p *= 10;
            ++d;
        }
    }
    return d;
}

void radixsort(int data[], int n) //基数排序
{
    int d = maxbit(data, n);
    int tmp[n];
    int count[10]; //计数器
    int i, j, k;
    int radix = 1;
    for (i = 1; i <= d; i++) //进行d次排序
    {
        for (j = 0; j < 10; j++)
            count[j] = 0; //每次分配前清空计数器

        for (j = 0; j < n; j++)
        {
            k = (data[j] / radix) % 10; //统计每个桶中的记录数
            count[k]++;
        }
        for (j = 1; j < 10; j++)
            count[j] = count[j - 1] + count[j]; //将tmp中的位置依次分配给每个桶
        for (j = n - 1; j >= 0; --j)            //将所有桶中记录依次收集到tmp中,这里的逆序是为了保证上一次排序后的顺序没有被改变
        {
            k = (data[j] / radix) % 10;
            tmp[count[k] - 1] = data[j];
            count[k]--;
        }
        for (j = 0; j < n; j++) //将临时数组的内容复制到data中
            data[j] = tmp[j];
        radix = radix * 10;
        printArray(data, n);
    }
}

int main()
{
    int length = 4;
    int array[length] = {1, 111, 11, 1};
    radixsort(array, length);
    printArray(array, length);
    return 0;
}

//结果
//14 22 28 39 43 55 65 73 81 93
```

#### 对容器

```c++

int maxbit(const std::vector<int> &data)
{
    int d = 1; //保存最大的位数
    int p = 10;
    for (int i = 0; i < data.size(); ++i)
    {
        while (data[i] >= p)
        {
            p *= 10;
            ++d;
        }
    }
    return d;
}

void bucketSort(std::vector<int> &vet)
{
    int n = maxbit(vet);
    int range = 10, radix = 10;
    std::vector<std::vector<int>> buckets(range);
    for (auto num : vet)
    {
        buckets[num % 10].push_back(num);
    }
    for (int i = 1; i < n; ++i, radix *= 10)
    {
        std::vector<std::vector<int>> temp(range);
        for (auto buk : buckets)
        {
            for (auto num : buk)
            {
                temp[num / radix % 10].push_back(num);
            }
        }
        std::swap(buckets, temp);
    }

    int pos = 0;
    for (auto buk : buckets)
    {
        for (auto num : buk)
        {
            vet[pos++] = num;
        }
    }
}
```

### 处理负数的基数排序

![image-20200403173206065](../img/image-20200403173206065.png)

#### 对容器

思路：

使用基数为16，桶的数目为16，32的int一共需要排序8次，最高为是符号位，这样可以使用

移位操作代替除法操作，提高了操作效率。

```c++
void bucket_sort(vector<int>& nums){
		vector<vector<int>> bucket(16, vector<int>());
		//第一轮，将数组按照最低四位2进制数存入所有的桶中
		for (auto i : nums){
			bucket[i & 15].push_back(i);
		}
		//循环七轮，进行桶排序
		for (int i = 1; i < 8; i++){
			vector<vector<int>> bucket2(16, vector<int>());
			//计算移位和掩码
			int shift = i * 4;
			int mask = 0xf<< shift;
			for (auto v : bucket)
				//注意 C++ unsigned的右移是逻辑右移，前面不补0
				for (unsigned num : v)
					bucket2[(num&mask) >> shift].push_back(num);
			//每轮之后交换原桶和临时桶
			swap(bucket, bucket2);
		}
		//排序之后，交换正负数的位置
		int pos = 0;
		for (int i = 0; i < 8; i++) swap(bucket[i], bucket[i + 8]);
		for (auto v : bucket)
			for (auto i : v)
				nums[pos++] = i;
	}
```

需要说明的是：负数是补码存放，补码的绝对值大小是`0xffffffff+num`。排序之后，0-7个桶存放的是正数，从小到大，8-15存放的是负数，也是从小到大，因此最后需要把8-15桶放到0-7桶前，完成排序。

#### 对链表

```c++

class Solution
{
public:
    ListNode *sortList(ListNode *head)
    {

        if (head == NULL)
            return NULL;
        int range = 16;
        int n = 8;

        for (int i = 0; i < n; ++i)
        {
            ListNode **bottom, **top;
            bottom = new ListNode *[range]();
            top = new ListNode *[range]();
            ListNode *p = head;

            int shift = i * 4;
            int mask = 0xf << shift;
            //get into bins
            while (p)
            {
                unsigned temp = p->val;
                unsigned bin = (temp & mask) >> shift;

                if (bottom[bin] == NULL)
                {
                    bottom[bin] = top[bin] = p;
                }
                else
                {
                    top[bin]->next = p;
                    top[bin] = p;
                }
                p = p->next;
            }

            ListNode *newhead = NULL;
            ListNode *temp;
            for (int j = 0; j < range; ++j)
            {
                if (bottom[j] != NULL)
                {
                    if (newhead == NULL)
                        newhead = bottom[j];
                    else
                        temp->next = bottom[j];
                    temp = top[j];
                }
            }
            if (temp)
                temp->next = 0;
            delete[] bottom;
            delete[] top;
            head = newhead;
        }

        ListNode *p = head;
        while (p && p->next && p->next->val > 0)
        {
            p = p->next;
        }
        if (p && p->next)
        {
            ListNode *head2 = p->next;
            p->next = NULL;
            ListNode *temp = head2;
            while (temp->next)
            {
                temp = temp->next;
            }
            temp->next = head;
            return head2;
        }
        return head;
    }
};
```

### 跳跃链表

https://blog.csdn.net/u013011841/article/details/39158585（跳表的具体实现）

https://www.jianshu.com/p/9d8296562806（如何动态维护索引，讲述概率算法）

#### 跳表的层数

理论上跳表的层数最好是logn，而且上下层结点数比为2，这样总共查找次数为2logn，复杂度保持在O(logn)。

#### 跳表的结构

跳跃表由多条链构成（L0，L1，L2 ……，Lh），且满足如下三个条件：

- 每条链必须包含两个特殊元素：头和尾两个指针。

- L0包含所有的元素，并且所有链中的元素按照升序排列。

- 每条链中的元素集合必须包含于序数较小的链的元素集合。

  ![image-20200408213559448](../img/image-20200408213559448.png)

#### 插入数据时如何更新索引 

```c++
1、理想的索引是在原始链表中每隔一个元素抽取一个元素作为一级索引。但是，也可以从原始链表中随机抽取n/2个元素作为一级索引，因为当数据足够大，抽取足够随机的话，得到的索引也是均匀的。如果数据量较少，那么带来的复杂度的增大也问题不大，毕竟数据量少。
所以：我们可以维护一个这样的索引：随机选 n/2 个元素做为一级索引、随机选 n/4 个元素做为二级索引、随机选 n/8 个元素做为三级索引，依次类推，一直到最顶层索引。这里每层索引的元素个数已经确定，且每层索引元素选取的足够随机，所以可以通过索引来提升跳表的查找效率。

2、那么当插入新数据时，这个数据应该有 1/2 的几率建立一级索引、1/4 的几率建立二级索引、1/8 的几率建立三级索引，以此类推。现在我们就需要一个概率算法帮我们把控这个 1/2、1/4、1/8 ... ，当每次有数据要插入时，先通过概率算法告诉我们这个元素需要插入到几级索引中，然后开始维护索引并把数据插入到原始链表中。下面开始讲解这个概率算法代码如何实现。

3、假设概率算法为randomLevel() 方法，该方法会随机生成 1~MAX_LEVEL 之间的数，假设randomLevel() 方法返回 1 表示当前插入的该元素不需要建索引，只需要存储数据到原始链表即可
randomLevel() 方法返回 2 表示当前插入的该元素需要建一级索引
randomLevel() 方法返回 3 表示当前插入的该元素需要建二级索引
依次类推

4、那么randomLevel()返回1的概率应该是多少呢？建立一级索引的概率为1/2，应该是返回值>1的概率，所以返回值为1的概率为1-1/2=1/2
同理，当 randomLevel() 方法返回值 > 2 时，会建立二级或二级以上索引，都会在二级索引中增加元素，因此二级索引中元素个数占原始数据的比率为 randomLevel() 方法返回值 > 2 的概率。 randomLevel() 方法返回值 > 2 的概率为 1 减去 randomLevel() = 1 或 =2 的概率，即 1 - 1/2 - 1/4 = 1/4。OK，达到了我们设计的目标：二级索引中元素个数占原始数据的 1/4。

所以：
randomLevel() 方法，随机生成 1~MAX_LEVEL 之间的数（MAX_LEVEL表示索引的最高层数），且有 1/2的概率返回 1、1/4的概率返回 2、1/8的概率返回 3 ...
randomLevel() 方法返回 1 不建索引、返回2建一级索引、返回 3 建二级索引、返回 4 建三级索引 ...

int randX(int &level)
{
    int i, j, t;
    t = rand();
    for (i = 0, j = 2; i < maxLevel; i++, j += j)
        if (t > RAND_MAX / j)
            break;
    if (i > level)
        level = i;
    return i;
}

//或者下面的代码生成ranndomlevel
int randomLevel(){
    int level=1;
    while(Math.random()<1/2 && level<MAX_LEVEL)
        level+=1;
    return level;
}
```



```c++
#include <iostream>
#include <iterator>
#include <cmath>
#include <vector>
#include <list>
#include <algorithm>

const int maxLevel = 8;
struct Node
{
    int data;
    Node *forward[maxLevel];
};

int randX(int &level)
{
    int i, j, t;
    t = rand();
    for (i = 0, j = 2; i < maxLevel; i++, j += j)
        if (t > RAND_MAX / j)
            break;
    if (i > level)
        level = i;
    return i;
}

Node *search(Node *head, int key, int level)
{
    Node *p = head;
    for (; level >= 0 && p; --level)
    {
        while (p->forward[level]->data < key)
        {
            p = p->forward[level];
        }
    }
    if (p != NULL && p->data == key)
        return p;
    else
        return NULL;
}

void insert(Node *head, int key, int &level)
{
    Node *p = head;
    int newlevel = randX(level);
    Node *update[maxLevel];
    for (int i = level; i >= 0; --i)
    {
        while (p->forward[i] != NULL && p->forward[i]->data < key)
        {
            p = p->forward[i];
        }
        update[i] = p;
    }
    Node *newNode = new Node;
    newNode->data = key;
    for (int i = 0; i < maxLevel; ++i)
    {
        newNode->forward[i] = 0;
    }
    for (int i = newlevel; i >= 0; --i)
    {
        if (update[i])
        {
            newNode->forward[i] = update[i]->forward[i];
            update[i]->forward[i] = newNode;
        }
    }
}

int deleteNode(Node *head, int &level)
{
    int key;
    std::cout << "input number to delete";
    std::cin >> key;
    Node *r = search(head, key, level);
    Node *p, *q;
    if (r)
    {
        for (int i = level; i >= 0; --i)
        {
            p = q = head;
            for (; p && p != r; q = p, p = p->forward[i])
            {
            }
            if (p)
            {
                if (i == level && q == head && p->forward[i] == NULL)
                    --level;
                else
                    q->forward[i] = p->forward[i];
            }
        }
        delete r;
        return 0;
    }
    return 1;
}
```

### 交换连个数的值（异或）

```c++
int a,b;
a=a^b;
b=a^b;
a=a^b;
```

### 二叉树的非递归遍历

#### 前序

> 首先判断根是否为空，将根节点入栈
>
> 1.若栈为空，则退出循环 
> 2.将栈顶元素弹出，访问弹出的节点 
> 3.若弹出的节点的右孩子不为空则将右孩子入栈 
> 4.若弹出的节点的左孩子不为空则将左孩子入栈 
> 5.返回1

```c++
void preOrder(){
    Stack<TreeNode> stack = new Stack<TreeNode>();
    if (root != null) {
        stack.push(root);
        while (!stack.isEmpty()) {
            root = stack.pop();
            visit(root);
            if (root.right != null) {
                stack.push(root.right);
            }
            if (root.left != null) {
                stack.push(root.left);
            }
        }
    }
}
```

#### 后序

思路1：

![image-20200421093224224](../img/image-20200421093224224.png)

```c++
ArrayList<Integer> postOrder(TreeNode root) {
    ArrayList<Integer> list = new ArrayList<Integer>();
    if (root != null) {
        Stack<TreeNode> stack = new Stack<TreeNode>();
        stack.push(root);
        while (!stack.isEmpty()) {
            TreeNode node = stack.pop();
            list.add(node.val);
            if (node.left != null) {
                stack.push(node.left);
            }
            if (node.right != null) {
                stack.push(node.right);
            }
        }
        //反转
        Collections.reverse(list);
    }
    return list;
}
```

思路2：

后序遍历的难点在于：需要判断上次访问的节点是位于左子树，还是右子树。若是位于左子树，则需跳过根节点，先进入右子树，再回头访问根节点；若是位于右子树，则直接访问根节点。

所以，方法是在代码中加一个上次访问过的节点，如果一个节点的右节点为null或者右节点是上次访问过的节点，这个节点就可以被访问。(或者，：给每个节点附加一个标记(left,right)。如果该节点的左子树已被访问过则置标记为left；若右子树被访问过，则置标记为right。显然，只有当节点的标记位是right时，才可访问该节点；否则，必须先进入它的右子树，代码是差不多的)

```c++
//后序遍历实现的另一种方法
void PostOrderTraverse(Node *head)
{
    std::stack<Node *> a;
    Node *p, *lastVisit; //临时指针
    p = head;
    lastVisit = NULL;
    //当p为NULL或者栈为空时，表明树遍历完成
    while (p || !a.empty())
    {
        //如果p不为NULL，将其压栈并遍历其左子树
        while (p)
        {
            a.push(p);
            p = p->left;
        }
        //如果p==NULL，表明左子树遍历完成，需要遍历上一层结点的右子树

        p = a.top();
        if (p->right == NULL || lastVisit == p->right)
        {
            std::cout << p->data;
            a.pop();
            lastVisit = p;
            p = NULL;
        }
        else
            p = p->right;
    }
}

```



#### 中序

```c++
void InOrder(Node *head)
{
    std::stack<Node *> sta;
    Node *p = head;
    while (p || !sta.empty())
    {
        while (p)
        {
            sta.push(p);
            p = p->left;
        }
        p = sta.top();
        std::cout << p->data << " ";
        sta.pop();
        p = p->right;
    }
}
```

### 层次遍历的反序列化

> 数据储存在数组中，空节点为0，如下图所示
>
> ![image-20200423103744015](../img/image-20200423103744015.png)

```c++

void Deserialize(const std::vector<int> &data)
{
    if (data.empty())
        return;
    std::vector<TreeNode *> nodes(data.size());
    for (int i = 0; i != nodes.size(); ++i)
    {
        nodes[i] = (data[i] == 0) ? NULL : new TreeNode(data[i]);
    }
    for (int i = 0; i != nodes.size(); ++i)
    {
        if (nodes[i] == NULL)
            continue;
        if (2 * i + 1 < nodes.size())
            nodes[i]->left = nodes[2 * i + 1];
        else
            nodes[i]->left = NULL;
        if (2 * i + 2 < nodes.size())
            nodes[i]->right = nodes[2 * i + 2];
        else
            nodes[i]->right = NULL;
    }
    TreeNode *root = nodes[0];
    PreOrder(root);
    return;
}
```

另一种写法，可用于这样的数组描述[4,null,2,1,2],也就是不再严格遵循第i个数的左节点是`2×i+1`，右节点是`2×i+2`

```c++

void Deserialize(const std::vector<int> &data)
{
    if (data.empty())
        return;
    std::queue<TreeNode *> q;
    TreeNode *root = new TreeNode(data[0]);
    int i = 0;
    q.push(root);
    while (!q.empty() && i < data.size())
    {
        TreeNode *node = q.front();
        q.pop();
        if (data[i] == 0)
        {
            ++i;
            continue;
        }
        if (2 * i + 1 < data.size() && data[2 * i + 1] != 0)
            node->left = new TreeNode(data[2 * i + 1]);
        else
            node->left = NULL;

        if (2 * i + 2 < data.size() && data[2 * i + 2] != 0)
            node->right = new TreeNode(data[2 * i + 2]);
        else
            node->right = NULL;
        q.push(node->left);
        q.push(node->right);
        ++i;
    }
    PreOrder(root);
    return;
}
```

或者下面这个写法

```c++

void Deserialize(const std::vector<int> &data)
{
    if (data.empty())
        return;
    std::queue<TreeNode *> q;
    TreeNode *root = new TreeNode(data[0]);
    q.push(root);
    for (int i = 1; i < data.size();)
    {
        auto node = q.front();
        q.pop();
        if (data[i] != 0)
        {
            auto left = new TreeNode(data[i]);
            node->left = left;
            q.push(node->left);
        }
        ++i;
        if (data[i] != 0)
        {
            auto rigth = new TreeNode(data[i]);
            node->right = rigth;
            q.push(node->right);
        }
        ++i;
    }
    }
}
```

### 表达式树

根据表达式建树，同时根据树计算表达式结果

https://www.cnblogs.com/edisonchou/p/4649954.html

```c++

struct Node
{
    char data;
    bool is_op; //判断当前节点是op还是数字
    Node *left, *right;
    Node(const char c, bool val, Node *lth = nullptr, Node *rth = nullptr) : data(c), is_op(val), left(lth), right(rth) {}
};

//得到运算符的优先级
size_t precedent(const char &op)
{
    if (op == '+' || op == '-')
        return 1;
    else if (op == '*' || op == '/')
        return 2;
    else
        throw std::runtime_error("wrong op");
}


//对整棵树进行层次遍历
void dfs(Node *head)
{
    if (head == nullptr)
        return;
    std::queue<Node *> q;
    q.push(head);
    while (!q.empty())
    {
        int length = q.size();
        for (int i = 0; i < length; ++i)
        {
            Node *node = q.front();
            q.pop();
            std::cout << node->data << " ";
            if (node->left)
                q.push(node->left);
            if (node->right)
                q.push(node->right);
        }
        std::cout << std::endl;
    }
    return;
}

//根据字符初始化Node
Node *getNode(const char &c)
{
    std::set<char> ops = {'+', '-', '*', '/'};
    if (ops.find(c) != ops.end())
        return new Node(c, true, nullptr, nullptr);
    else
        return new Node(c, false, nullptr, nullptr);
}

//根据表达式数计算表达式结果
int calNode(Node *head)
{
    if (head)
    {
        if (!head->is_op)
            return head->data - '0';
        int left = calNode(head->left);
        int right = calNode(head->right);
        switch (head->data)
        {
        case '+':
            return left + right;
        case '-':
            return left - right;
        case '*':
            return left * right;
        case '/':
            if (right == 0)
                std::runtime_error("divion error");
            return left / right;
        default:
            return 0;
        }
    }
    return 0;
}

/*
根据表达式建立表达式树,算法如下：
1、第一个节点首先成为表达式树的根。
2、插入后序节点：
如果根节点为数字，而插入节点为op，则插入节点变为根节点，原来的根节点变为插入节点的左节点。
如果根节点为op：
    插入节点为op，若插入节点的优先级大于根节点优先级，则插入节点为根节点的右孩子，原先根节点的右子树成为插入节点的左孩子。
                若插入接地的优先级小于等于根节点优先级，则插入节点为根节点，原来根节点为插入节点的左孩子。
    插入节点为数组，沿着根节点右链插入到最右段。
*/
Node *build(std::string s)
{
    if (s.empty())
        return nullptr;
    int index = 0;
    Node *head = nullptr;
    while (index < s.size())
    {
        Node *tmp = getNode(s[index++]);
        if (head == NULL)
            head = tmp;
        else if (head->is_op == false)
        {
            if (tmp->is_op == false)
                std::runtime_error("wrong expression");
            tmp->left = head;
            head = tmp;
        }
        else if (tmp->is_op == true)
        {
            if (precedent(tmp->data) > precedent(head->data))
            {
                tmp->left = head->right;
                head->right = tmp;
            }
            else
            {
                tmp->left = head;
                head = tmp;
            }
        }
        else
        {
            Node *node = head;
            while (node->right != nullptr)
            {
                node = node->right;
            }
            node->right = tmp;
        }
    }
    return head;
}

```

#### 中缀表达式转后缀表达式

> 算法：
>
> 中缀 1+2x3/4-4，-2+4x5
>
> 后缀 123x4/+4-    -245x+
>
> ![image-20200426075950622](../img/image-20200426075950622.png)

这个代码还有负数和括号部分没有处理好

```c++
#include <vector>
#include <iostream>
#include <iterator>
#include <stack>
#include <set>
#include <algorithm>

//得到运算符的优先级
size_t precedent(const char &op)
{
    if (op == '+' || op == '-')
        return 1;
    else if (op == '*' || op == '/')
        return 2;
    else
        throw std::runtime_error("wrong op");
}

std::set<char> ops = {'+', '-', '*', '/'};

void transform(std::string Instr)
{
    std::stack<char> ops;
    std::vector<std::string> nums;
    int index = 0;
    while (index < Instr.size())
    {
        size_t i = 0;
        std::stoi(Instr.substr(index), &i);
        nums.push_back(Instr.substr(index, i));
        index += i;
        if (index < Instr.size())
        {
            if (ops.empty() || precedent(Instr[index]) > precedent(ops.top()))
                ops.push(Instr[index]);
            else
            {
                while (!ops.empty() && precedent(Instr[index]) <= precedent(ops.top()))
                {
                    nums.push_back(ops.top() + std::string());
                    ops.pop();
                }
                ops.push(Instr[index]);
            }
        }
        ++index;
    }

    while (!ops.empty())
    {
        nums.push_back(ops.top() + std::string());
        ops.pop();
    }

    for (auto i : nums)
    {
        std::cout << i << " ";
    }
}

int main()
{
    std::string s = "-c2*3/2+5";
    transform(s);
}
```

#### 后缀表达式求值

![image-20200426080624702](../img/image-20200426080624702.png)

