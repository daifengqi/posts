# 承前启后：中级数据结构

# Mid-level Data Structure

## 2021 年 4 月 25 日

## Apr 25 2021

---

### 引言

中级数据结构指的是单调栈、单调队列、优先队列等<u>在多数语言的标准库里内置或可直接用数组表示且要求某些特征</u>的数据结构；与之相比，初级数据结构指链表、栈、队列、二叉树等数据结构入门知识包括的内容；高级数据结构指并查集、线段树、树状数组等竞赛要求的数据结构。此外，还有终极数据结构：红黑树、跳跃表等...

> 如果面试官让你手写红黑树，那恐怕是在劝退了，建议直接起身说拜拜。
>
> ——[沃兹基·硕徳](https://cescdf.com)

### 单调栈

单调栈的本质是栈，但是从栈底到栈顶的元素数值大小，或元素为索引指向的数值大小符合（严格或非严格）单调性。

```javascript
// 单调递增栈，后面元素的比较用“>”还是“>=”要依据具体情况
while(stk.length > 0 && heights[stk.top()] >= heights[i]) {
  stk.pop();
}
```

[点击这里见题解：【柱状图中最大的矩形】](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/solution/javascript-dan-diao-zhan-by-cescdf-pzyo/)

单调栈的应用在<u>「移除」或「保留」</u>类的题目里也比较常见，这类题目通常要求剩下的字符或数字的有序性。这些题目的基题是[【移除K位数字】](https://leetcode-cn.com/problems/remove-k-digits/)，常见的变题有[【去除重复字母】](https://leetcode-cn.com/problems/remove-duplicate-letters/)，[【拼接最大数】](https://leetcode-cn.com/problems/create-maximum-number/)。

### 单调队列

单调队列既具有栈的性质，也具有队列的性质，所以它必须是一个双端队列。

1. `deque`内仅包含窗口内的元素 ⇒ 每轮窗口滑动移除了元素 `nums[i - k]`，需将 `deque`内的对应元素一起删除，这里利用了队列的性质；
2. `deque` 内的元素非严格递减 ⇒ 每轮窗口滑动添加了元素 `nums[j + 1]`，需将`deque`内所有 `< nums[j + 1]`的元素删除，因为要维护最大值，这里利用了栈的性质；

```javascript
// 单调队列：已知窗口长度k，当碰到（之前添加过的）该元素时，从队列首部取出
// 队列性质
if (deque[0] === nums[i-k]) {
  deque.shift();
}
// 栈性质
while (deque.length > 0 && deque.last() < nums[i]) {
  deque.pop();
}
```

[点击这里见题解：【滑动窗口的最大值】](https://leetcode-cn.com/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/solution/javascript-dan-diao-zhan-dan-diao-dui-li-3g3h/)

练习题：[【滑动窗口的中位数】](https://leetcode-cn.com/problems/sliding-window-median/)

### 优先队列

在<u>ECMAScript</u>标准中并不具备优先队列的实现，这里贴一个手动实现的最小堆，它用数组表示一个二叉树，树的左子节点索引为`2i+1`，右子节点索引为`2i+2`，所以通过子节点得到父节点索引的方式为`(i-1 >> 1)`。

优先队列是一种性质，二叉堆是优先队列的实现。

```javascript
// 最小堆
class MinHeap {
    constructor() {
        this.heap = [];
    }

    getParentIndex(i) {
        return (i - 1) >> 1;
    }

    getLeftIndex(i) {
        return i * 2 + 1;
    }

    getRightIndex(i) {
        return i * 2 + 2;
    }

    shiftUp(index) {
        if(index === 0) { return; }
        const parentIndex = this.getParentIndex(index);
        if(this.heap[parentIndex] > this.heap[index]){
            this.swap(parentIndex, index);
            this.shiftUp(parentIndex);
        }  
    }

    swap(i1, i2) {
        const temp = this.heap[i1];
        this.heap[i1]= this.heap[i2];
        this.heap[i2] = temp;
    }

    insert(value) {
        this.heap.push(value);
        this.shiftUp(this.heap.length - 1);
    }

    pop() {
        this.heap[0] = this.heap.pop();
        this.shiftDown(0);
        return this.heap[0];
    }

    shiftDown(index) {
        const leftIndex = this.getLeftIndex(index);
        const rightIndex = this.getRightIndex(index);
        if (this.heap[leftIndex] < this.heap[index]) {
            this.swap(leftIndex, index);
            this.shiftDown(leftIndex);
        }
        if (this.heap[rightIndex] < this.heap[index]){
            this.swap(rightIndex, index);
            this.shiftDown(rightIndex);
        }
    }

    peek() {
        return this.heap[0];
    }

    size() {
        return this.heap.length;
    }
}
```

### 字典树（前缀树）

> Trie（发音类似 "try"）或者说**前缀树**是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。这一数据结构有相当多的应用情景，例如自动补完和拼写检查。
>
> ——[力扣208题：实现Trie（前缀树）](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)

一个字典树应该有如下一个内部结构，和三种方法（`insert`，`search`，`startsWith`）：

```javascript
var Trie = function() {
    this.children = {};
};

Trie.prototype.insert = function(word) {
    let node = this.children;
    for (const ch of word) {
        if (!node[ch]) {
            node[ch] = {};
        }
        node = node[ch];
    }
    node.isEnd = true;
};

Trie.prototype.searchPrefix = function(prefix) {
    let node = this.children;
    for (const ch of prefix) {
        if (!node[ch]) {
            return false;
        }
        node = node[ch];
    }
    return node;
}

Trie.prototype.search = function(word) {
    const node = this.searchPrefix(word);
    return node !== undefined && node.isEnd !== undefined;
};

Trie.prototype.startsWith = function(prefix) {
    return this.searchPrefix(prefix);
};
```

字典树的根本思想是一棵有根树，树的每个节点包含以下字段：

- 指向子节点的指针数组`children`，对于本题而言，数组长度为26，即小写英文字母的数量。`children[0]`指向`a`，`children[25]`指向`z`；
- 布尔字段`isEnd`，表示该节点是否为字符串的结尾；

### 状态机

严格来说，状态机可能不算一种特定的数据结构，但在这里还是介绍一下。

状态及通常用于验证某个给定字符串是否满足某种要求，一般是遍历一次字符串，途中遇到不符合要求的地方就返回false，否则返回true。状态机的转移过程如下，

```js
let state = State.STATE_INITIAL; // 设置初始状态

// s：需要检查的字符串
for (const c of s) {
  const type = toCharType(c);
  // transfer：字典的字典，外部是State -> Map的映射
  // 内部嵌套的Map是 charType -> State
  if (!transfer.get(state).has(type)) {
    return false;
  } else {
    // 完成 state -> state 的转变
    state = transfer.get(state).get(type);
  }
}
```

解决一个状态机问题需要定义好，

- 所有状态（转移前、转移后）
- 转移模式（处于状态i时遇到c怎么转移）

通常可用一张图来表述，[有效数字](https://leetcode-cn.com/problems/valid-number/)的例子如下，

![fig1](https://assets.leetcode-cn.com/solution-static/65/1.png)

只要能清晰画出和理解状态机的这张图，就能解决相应的问题。

上图共有9个状态，如下，

```js
const State = {
  STATE_INITIAL : "STATE_INITIAL",
  STATE_INT_SIGN : "STATE_INT_SIGN",
  STATE_INTEGER : "STATE_INTEGER",
  STATE_POINT : "STATE_POINT",
  STATE_POINT_WITHOUT_INT : "STATE_POINT_WITHOUT_INT",
  STATE_FRACTION : "STATE_FRACTION",
  STATE_EXP : "STATE_EXP",
  STATE_EXP_SIGN : "STATE_EXP_SIGN",
  STATE_EXP_NUMBER : "STATE_EXP_NUMBER",
  STATE_END : "STATE_END"
}
```

字符串中的所有元素类型如下，

```js
const CharType = {
  CHAR_NUMBER : "CHAR_NUMBER",
  CHAR_EXP : "CHAR_EXP",
  CHAR_POINT : "CHAR_POINT",
  CHAR_SIGN : "CHAR_SIGN",
  CHAR_ILLEGAL : "CHAR_ILLEGAL"
}

const toCharType = (ch) => {
  if (!isNaN(ch)) {
    return CharType.CHAR_NUMBER;
  } else if (ch.toLowerCase() === 'e') {
    return CharType.CHAR_EXP;
  } else if (ch === '.') {
    return CharType.CHAR_POINT;
  } else if (ch === '+' || ch === '-') {
    return CharType.CHAR_SIGN;
  } else {
    return CharType.CHAR_ILLEGAL;
  }
}  
```

通过状态转移关系给出嵌套字典表示的路径，

```js
const transfer = new Map();
const initialMap = new Map();
// 初始状态有三种转出方式，所以内部字典有三个映射
initialMap.set(CharType.CHAR_NUMBER, State.STATE_INTEGER);
initialMap.set(CharType.CHAR_POINT, State.STATE_POINT_WITHOUT_INT);
initialMap.set(CharType.CHAR_SIGN, State.STATE_INT_SIGN);
// 设置外部字典，从状态到映射关系
transfer.set(State.STATE_INITIAL, initialMap);
```

上面只是一种状态，其他转移关系（图中的线）也需要分别定义。

### 双栈问题

一般经典的表达式处理问题，我们使用操作符栈（ops）+操作数栈（state）的经典双栈方式来进行模拟。

题目：[反转表达式值的最少操作次数](https://leetcode-cn.com/problems/minimum-cost-to-change-the-final-value-of-expression/)、题解

### 特殊结构

其他数据结构，比如[最小栈](https://leetcode-cn.com/problems/min-stack/)，也属于一种“中级”的数据结构，它们通常基于某种基础结构，然后加入了某些特定的方法，包装成一个新类。新方法往往有时间复杂度的限制。

练习题与题解：

- [LRU Cache](https://leetcode-cn.com/problems/lru-cache/)
- [四种方法解决TopK问题](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/solution/javascriptsi-chong-fang-shi-jie-topkwen-ti-by-user/)

### 简单结构的经典题

一道哈希表的例题：[【最长连续序列】](https://leetcode-cn.com/problems/longest-consecutive-sequence/)，这道题如果不知道要使用hash表来做，往往容易被误导，若是知道了就相对简单。

一道哈希表的题：[【直线上最多的点数】](https://leetcode-cn.com/problems/max-points-on-a-line/)，很体现哈希表优化一次遍历的时间复杂度的思想。

