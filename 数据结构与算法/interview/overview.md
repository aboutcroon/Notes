## 1. 数组

### 哈希表，空间换时间

[1. 两数之和](https://leetcode.cn/problems/two-sum/)

[219. 存在重复元素2](https://leetcode.cn/problems/contains-duplicate-ii/)



### 双指针

空间换时间
情景：用于求和，比大小类的数组题
前提：数组必须有序



[11. 乘最多水的容器](https://leetcode.cn/problems/container-with-most-water/)
双指针+优化排除法

[15. 三数之和](https://leetcode.cn/problems/3sum/)
对撞指针法

[26. 删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)
读指针与写指针

[88. 合并两个有序数组](https://leetcode.cn/problems/merge-sorted-array/)



### 其他解法

[121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)

[122. 买卖股票的最佳时机2](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)



## 2. 字符串

### 反转字符串

```js
const res = str.split('').reverse().join('')
```



### 回文字符串

1. 判断字符串反转后是否和初始相同
2. ❗️对称性：从中间位置劈开，分别遍历两边看是否对称
3. ❗️双指针：[125. 验证回文串](https://leetcode.cn/problems/valid-palindrome/)



## 3. 栈

### 场景与规律

若涉及括号问题，则很有可能和栈相关。

栈结构可以帮我们避免重复操作。例如暴力解法双层循环每个都遍历一遍时，会浪费很多重复的遍历，此时也可以用栈来避免一些重复操作。（同理也可以用哈希表，双指针达到同样目的）



### 题目

[20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/)

[739. 每日温度](https://leetcode.cn/problems/daily-temperatures/)

递减栈（用栈记录遍历过的数据，避免重复操作。）

[155. 最小栈](https://leetcode.cn/problems/min-stack/)

递减栈



## 4. 队列

### 重点

1. 栈向队列的转化（基础）（需要使用两个栈，一个栈只负责进队，一个栈只负责出队）
2. 双端队列（中等）
   - 双端队列就是允许在队列的两端进行插入和删除的队列。（用数组的 pop、push、shift、unshift实现）
   - 双端队列衍生出的滑动窗口问题，是一个经久不衰的命题热点。
3. 优先队列（高级）



### 题目

[232. 用栈实现队列](https://leetcode.cn/problems/implement-queue-using-stacks/)

栈向队列的转化

[239. 滑动窗口问题](https://leetcode.cn/problems/sliding-window-maximum/)

双端队列法，维持一个递减的双端队列



## 5. 递归与回溯

### 技巧

递归：

1. 以后只要分析出重复的逻辑（排除掉类似数组遍历这种简单粗暴的重复），你都需要把递归从你的大脑内存里调度出来、将其列为“可以一试”的解法之一；只要想到递归，立刻回忆起 DFS 思想、然后尝试套用解题模板。这个脑回路未必 100% 准确，但确实有极高的成功概率——题，是有规律的。这，就是规律之一。
2. 多使用 Map 结构来进行标记哪些已用过哪些没用过
3. 注意递归边界
4. 当我们感到变化难以把握时，不如尝试先从不变的东西入手

> 回溯：“回溯”二字，大家可以理解为是在强调 `DFS` 过程中“退一步重新选择”这个动作。这样想的话， `DFS` 算法其实就是回溯思想的体现。
>
> 剪枝：在深度优先搜索中，**有时我们会去掉一些不符合题目要求的、没有作用的答案，进而得到正确答案。这个丢掉答案的过程，形似剪掉树的枝叶，所以这一方法被称为“剪枝”**。  



### DFS 与 BFS

DFS（二叉树先序遍历）

本质是栈结构

```js
// 所有遍历函数的入参都是树的根结点对象
function preorder(root) {
  // 递归边界，root 为空
  if(!root) {
      return 
  }

  // 输出当前遍历的结点值
  console.log('当前遍历的结点值是：', root.val)  
  // 递归遍历左子树 
  preorder(root.left)  
  // 递归遍历右子树  
  preorder(root.right)
}
```

BFS（二叉树层序遍历）

本质是队列

```js
function BFS(root) {
  const queue = [] // 初始化队列queue
  // 根结点首先入队
  queue.push(root)
  // 队列不为空，说明没有遍历完全
  while(queue.length) {
      const top = queue[0] // 取出队头元素  
      // 访问 top
      console.log(top.val)
      // 如果左子树存在，左子树入队
      if(top.left) {
          queue.push(top.left)
      }
      // 如果右子树存在，右子树入队
      if(top.right) {
          queue.push(top.right)
      }
      queue.shift() // 访问完毕，队头元素出队
  }
}
```

> 理论上来说只要我们拿到了 `top`，那么就不再关心队头元素了。因此这个 `shift` 出队的过程，其实是比较灵活的。一般只要我们拿到了 `top`，就可以执行 `shift`了。一些同学习惯于把`top`元素的访问和出队放在一起来做：`const top = queue.shift()` 



### 解题模版

如何总结出一套解题模板？其实很简单，大家只需要搞清楚三个问题：  

1. 什么时候用？（明确场景）
2. 为什么这样用？（提供依据）
3. 怎么用？（细化步骤）

#### 什么时候用

看两个特征：

1. 题目中暗示了一个或多个解，并且要求我们详尽地列举出每一个解的内容时，一定要想到 DFS、想到递归回溯。 
2. 🔴 题目经分析后，可以转化为**树形逻辑模型**求解。（一定要结合树形更好理解）

#### 为什么这样用

递归与回溯的过程，本身就是穷举的过程。题目中要求我们列举每一个解的内容，解从哪来？解是基于**穷举思想**、对**搜索树进行恰当地剪枝**后得来的。  

> 这里需要大家注意到另一种问法：
>
> 不问解的内容，只问解的个数。这类问题往往不用 DFS 来解，而是用动态规划。这里，大家先记下这个辨析，对以后做题会有帮助。

#### 怎么用

**一个模型**：树形逻辑模型；**两个要点**：递归式，递归边界

树形逻辑模型的构建，关键在于找“坑位”，一个坑位就对应树中的一层，每一层的处理逻辑往往是一样的，这个逻辑就是递归式的内容。至于递归边界，要么在题目中约束得非常清楚、要么默认为“坑位”数量的边界。 

用伪代码总结一下编码形式，大部分的题解都符合以下特征： 

```js
function dfs (入参) {
  前期的变量定义、缓存等准备工作 

  // 初始化结果数组
  const res = []
  // 定义路径栈
  const path = []

  // 进入 dfs
  dfs(起点) 

  // 定义 dfs
  dfs(递归参数) {
    if(到达了递归边界) {
      结合题意处理边界逻辑，往往和 path 内容有关
      res.push(path.slice())
      return
    }

    // 注意这里也可能不是 for，视题意决定
    for(遍历坑位的可选值) {
      path.push(当前选中值)
      处理坑位本身的相关逻辑
      dfs(i + 1)
      path.pop()
    }
  }

  // 返回结果数组
  return res
}
```

在面试中，如果你隐约觉得这道题用递归回溯来解可能有戏，却一时间没办法明确具体的解法，那么不妨尝试把这段伪代码记在脑子里。

在面试时，先把框架写出来，然后结合题意去调整和填充伪代码的内容——很多时候，**我们做题缺的不是知识储备，而是一个具体的切入点。**



### 题目

[46. 全排列](https://leetcode.cn/problems/permutations/)

DFS递归

[77. 组合](https://leetcode.cn/problems/combinations/)

DFS递归与回溯（剪枝）



## 6. 树

[100. 相同的树](https://leetcode.cn/problems/same-tree/)

[101. 对称二叉树](https://leetcode.cn/problems/symmetric-tree/)

[104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)



## 7. 排序算法



## 8. 动态规划



## 9. 避免暴力解法

### 两层循环遍历

两层循环每个都遍历一遍时，会浪费很多重复的遍历，这时我们需要用一些方法来避免这些重复计算。

解决方案：

1. 哈希表

   记录之前遍历过的数，避免重复遍历，或者更快的找到目标数。

2. 双指针

3. 栈

   栈结构可以帮我们避免重复操作。
   避免重复操作的秘诀就是及时地将不必要的数据出栈，避免它对我们后续的遍历产生干扰。

4. 队列

   求滑动窗口最大值时，可以维持一个递减的双端队列。



