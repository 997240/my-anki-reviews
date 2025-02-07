# 常考算法题（20道）

### 1、二分查找

- 问题描述：

  - 给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。

  示例 1:

  输入: nums = [-1,0,3,5,9,12], target = 9     
  输出: 4       
  解释: 9 出现在 nums 中并且下标为 4 

- 参考答案

  ```python
  class Solution:
      def search(self, nums, target):
          left, right = 0, len(nums) - 1
          
          while left <= right:
              middle = (left + right) // 2
  
              if nums[middle] < target:
                  left = middle + 1
              elif nums[middle] > target:
                  right = middle - 1
              else:
                  return middle
          return -1
  ```

- 

### 2、有序数组的平方

- 题目描述

  - 给你一个按非递减顺序排序的整数数组 nums，返回 每个数字的平方 组成的新数组，要求也按 非递减顺序 排序。

  示例 1： 

  输入：nums = [-4,-1,0,3,10] 

  输出：[0,1,9,16,100]

   解释：平方后，数组变为 [16,1,0,9,100]，排序后，数组变为 [0,1,9,16,100]

- 参考答案（双指针）

  ```python
  class Solution:
      def sortedSquares(self, nums):
          n = len(nums)
          i,j,k = 0,n - 1,n - 1
          ans = [-1] * n
          while i <= j:
              lm = nums[i] ** 2
              rm = nums[j] ** 2
              if lm > rm:
                  ans[k] = lm
                  i += 1
              else:
                  ans[k] = rm
                  j -= 1
              k -= 1
          return ans
  ```

### 3、移除链表元素

- 题目描述

  - 删除链表中等于给定值 val 的所有节点。

  示例 1：

  输入：head = [1,2,6,3,4,5,6], val = 6
  输出：[1,2,3,4,5]

  示例 2：
  输入：head = [], val = 1
  输出：[]

  示例 3：
  输入：head = [7,7,7,7], val = 7
  输出：[]

- 参考答案

  ```python
  # Definition for singly-linked list.
  # class ListNode:
  #     def __init__(self, val=0, next=None):
  #         self.val = val
  #         self.next = next
  class Solution:
      def removeElements(self, head, val):
          dummy_head = ListNode(next=head) #添加一个虚拟节点
          cur = dummy_head
          while(cur.next!=None):
              if(cur.next.val == val):
                  cur.next = cur.next.next #删除cur.next节点
              else:
                  cur = cur.next
          return dummy_head.next
  ```

### 4、反转链表

- 题目描述

  - 反转一个单链表。

  示例 1：

  输入: 1->2->3->4->5->NULL

  输出: 5->4->3->2->1->NULL

- 参考答案

  ```python
  #双指针
  # Definition for singly-linked list.
  # class ListNode:
  #     def __init__(self, val=0, next=None):
  #         self.val = val
  #         self.next = next
  class Solution:
      def reverseList(self, head):
          cur = head   
          pre = None
          while(cur!=None):
              temp = cur.next # 保存一下 cur的下一个节点，因为接下来要改变cur->next
              cur.next = pre #反转
              #更新pre、cur指针
              pre = cur
              cur = temp
          return pre
  ```

### 5、链表相交

- 题目描述

  - 给你两个单链表的头节点 headA 和 headB ，请你找出并返回两个单链表相交的起始节点。如果两个链表没有交点，返回 null 。

  示例 1：

![1661839880171](../5 项目简历/assets/链表相交.png)

- 参考答案：

  ```python
  
  class Solution:
      def getIntersectionNode(self, headA, headB):
          """
          根据快慢法则，走的快的一定会追上走得慢的。
          在这道题里，有的链表短，他走完了就去走另一条链表，我们可以理解为走的快的指针。
  
          那么，只要其中一个链表走完了，就去走另一条链表的路。如果有交点，他们最终一定会在同一个
          位置相遇
          """
          if headA is None or headB is None: 
              return None
          cur_a, cur_b = headA, headB     # 用两个指针代替a和b
  
  
          while cur_a != cur_b:
              cur_a = cur_a.next if cur_a else headB      # 如果a走完了，那么就切换到b走
              cur_b = cur_b.next if cur_b else headA      # 同理，b走完了就切换到a
  
          return cur_a
  ```

### 6、两数之和

- 题目描述

  - 给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。
  - 你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。

  示例 1：

  输入: 给定 nums = [2, 7, 11, 15], target = 9

  输出: 因为 nums[0] + nums[1] = 2 + 7 = 9；所以返回 [0, 1]

- 参考答案

  ```python
  class Solution:
      def twoSum(self, nums, target):
          records = dict()
  
          # 用枚举更方便，就不需要通过索引再去取当前位置的值
          for idx, val in enumerate(nums):
              if target - val not in records:
                  records[val] = idx
              else:
                  return [records[target - val], idx] # 如果存在就返回字典记录索引和当前索引
  ```

### 7、两个数组的交集

- 题目描述

  - 给定两个数组，编写一个函数来计算它们的交集。

  示例1：

  输入：nums1 = [1,2,2,1], nums2=[2,2]

  输出：[2]

  示例1：

  输入：nums1 = [4,9,5], nums2=[9,4,9,8,4]

  输出：[9, 4]

- 参考答案

  ```python
  class Solution:
      def intersection(self, nums1, nums2):
          return list(set(nums1) & set(nums2))    # 两个数组先变成集合，求交集后还原为数组
  ```

### 8、反转字符串

- 题目描述

  - 编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 char[] 的形式给出。
  - 注意：不要给另外的数组分配额外的空间，你必须原地修改输入数组、使用 O(1) 的额外空间解决这一问题。

  示例 1：

  输入：["h","e","l","l","o"]
  输出：["o","l","l","e","h"]

- 参考答案

  ```python
  class Solution:
      def reverseString(self, s):
          """
          Do not return anything, modify s in-place instead.
          """
          left, right = 0, len(s) - 1
          
          # 该方法已经不需要判断奇偶数，经测试后时间空间复杂度比用 for i in range(right//2)更低
          # 推荐该写法，更加通俗易懂
          while left < right:
              s[left], s[right] = s[right], s[left]
              left += 1
              right -= 1
  ```

### 9、重复的子字符串

- 题目描述

  - 给定一个非空的字符串，判断它是否可以由它的一个子串重复多次构成。给定的字符串只含有小写英文字母，并且长度不超过10000。

  示例 1:
  输入: "abab"
  输出: True
  解释: 可由子字符串 "ab" 重复两次构成。

- 参考答案

  ```python
  class Solution:
      def repeatedSubstringPattern(self, s):  
          if len(s) == 0:
              return False
          nxt = [0] * len(s)
          self.getNext(nxt, s)
          if nxt[-1] != -1 and len(s) % (len(s) - (nxt[-1] + 1)) == 0:
              return True
          return False
      
      def getNext(self, nxt, s):
          nxt[0] = -1
          j = -1
          for i in range(1, len(s)):
              while j >= 0 and s[i] != s[j+1]:
                  j = nxt[j]
              if s[i] == s[j+1]:
                  j += 1
              nxt[i] = j
          return nxt
  ```

### 10、用队列实现栈

- 题目描述
  - 使用队列实现栈的下列操作：
    - push(x) -- 元素 x 入栈
    - pop() -- 移除栈顶元素
    - top() -- 获取栈顶元素
    - empty() -- 返回栈是否为空

- 参考答案

  - ```
    from collections import deque
    class MyStack:
    
        def __init__(self):
            self.que = deque()
    
        def push(self, x)e:
            self.que.append(x)
    
        def pop(self):
            if self.empty():
                return None
            for i in range(len(self.que)-1):
                self.que.append(self.que.popleft())
            return self.que.popleft()
    
        def top(self):
            if self.empty():
                return None
            return self.que[-1]
    
        def empty(self):
            return not self.que
    ```

### 11、有效的括号

- 题目描述

  - 给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。
  - 有效字符串需满足：
    - 左括号必须用相同类型的右括号闭合。
    - 左括号必须以正确的顺序闭合。
    - 注意空字符串可被认为是有效字符串。

  示例1：
  输入：“()”

  输出：true

  示例2：
  输入：“(]”

  输出：false

- 参考答案：

  - ```python
    class Solution:
        def isValid(self, s: str) -> bool:
            stack = []
            mapping = {
                '(': ')',
                '[': ']',
                '{': '}'
            }
            for item in s:
                if item in mapping.keys():
                    stack.append(mapping[item])
                elif not stack or stack[-1] != item: 
                    return False
                else: 
                    stack.pop()
            return True if not stack else False
    ```

### 12、删除字符串中的所有相邻重复项

- 题目描述

  - 给出由小写字母组成的字符串 S，重复项删除操作会选择两个相邻且相同的字母，并删除它们。
  - 在 S 上反复执行重复项删除操作，直到无法继续删除。
  - 在完成所有重复项删除操作后返回最终的字符串。答案保证唯一

  示例1：

  - 输入："abbaca"
  - 输出："ca"
  - 解释：例如，在 "abbaca" 中，我们可以删除 "bb" 由于两字母相邻且相同，这是此时唯一可以执行删除操作的重复项。之后我们得到字符串 "aaca"，其中又只有 "aa" 可以执行重复项删除操作，所以最后的字符串为 "ca"。

- 参考答案

  - ```python
    # 使用栈，推荐！
    class Solution:
        def removeDuplicates(self, s):
            res = list()
            for item in s:
                if res and res[-1] == item:
                    res.pop()
                else:
                    res.append(item)
            return "".join(res)  # 字符串拼接
    ```

### 13、滑动窗口最大值

- 题目描述

  - 给定一个数组 nums，有一个大小为 k 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 k 个数字。滑动窗口每次只向右移动一位
  - 返回滑动窗口中的最大值。

  示例 1：

  ![1661841717867](../5 项目简历/assets/滑动窗口.png)

- 参考答案

```python
class MyQueue: #单调队列（从大到小
    def __init__(self):
        self.queue = [] #使用list来实现单调队列
    
    #每次弹出的时候，比较当前要弹出的数值是否等于队列出口元素的数值，如果相等则弹出。
    #同时pop之前判断队列当前是否为空。
    def pop(self, value):
        if self.queue and value == self.queue[0]:
            self.queue.pop(0)#list.pop()时间复杂度为O(n),这里可以使用collections.deque()
            
    #如果push的数值大于入口元素的数值，那么就将队列后端的数值弹出，直到push的数值小于等于队列入口元素的数值为止。
    #这样就保持了队列里的数值是单调从大到小的了。
    def push(self, value):
        while self.queue and value > self.queue[-1]:
            self.queue.pop()
        self.queue.append(value)
        
    #查询当前队列里的最大值 直接返回队列前端也就是front就可以了。
    def front(self):
        return self.queue[0]
    
class Solution:
    def maxSlidingWindow(self, nums, k):
        que = MyQueue()
        result = []
        for i in range(k): #先将前k的元素放进队列
            que.push(nums[i])
        result.append(que.front()) #result 记录前k的元素的最大值
        for i in range(k, len(nums)):
            que.pop(nums[i - k]) #滑动窗口移除最前面元素
            que.push(nums[i]) #滑动窗口前加入最后面的元素
            result.append(que.front()) #记录对应的最大值
        return result
```

### 14、翻转一棵二叉树

- 题目描述

  示例 1：

  ![1661841976194](../5 项目简历/assets/翻转二叉树.png)

- 参考答案

  - ```python
    #（递归方法）
    class Solution:
        def invertTree(self, root):
            if not root:
                return None
            root.left, root.right = root.right, root.left #中
            self.invertTree(root.left) #左
            self.invertTree(root.right) #右
            return root
    ```

### 15、二叉树的最近公共祖先

- 题目描述

  - 给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

  示例 1： 

  ![1661842194410](../5 项目简历/assets/最近公共祖先.png)

  :输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1 

  输出: 3 

  解释: 节点 5 和节点 1 的最近公共祖先是节点 3。

- 参考答案

  - ```python
    class Solution:
        """二叉树的最近公共祖先 递归法"""
    
        def lowestCommonAncestor(self, root, p, q):
            if not root or root == p or root == q:
                return root
            
            left = self.lowestCommonAncestor(root.left, p, q)
            right = self.lowestCommonAncestor(root.right, p, q)
            
            if left and right:
                return root
            if left:
                return left
            return right
    ```

### 16、使用最小花费爬楼梯

- 题目描述

  - 数组的每个下标作为一个阶梯，第 i 个阶梯对应着一个非负数的体力花费值 cost[i]（下标从 0 开始）。每当你爬上一个阶梯你都要花费对应的体力值，一旦支付了相应的体力值，你就可以选择向上爬一个阶梯或者爬两个阶梯。
  - 请你找出达到楼层顶部的最低花费。在开始时，你可以选择从下标为 0 或 1 的元素作为初始阶梯。

  示例 1：

  输入：cost = [10, 15, 20] 

  输出：15 

  解释：最低花费是从 cost[1] 开始，然后走两步即可到阶梯顶，一共花费 15 。

  示例 2：

  输入：cost = [1, 100, 1, 1, 1, 100, 1, 1, 100, 1] 

  输出：6

   解释：最低花费方式是从 cost[0] 开始，逐个经过那些 1 ，跳过 cost[3] ，一共花费 6 。

- 参考答案：

```python
class Solution:
    def minCostClimbingStairs(self, cost):
        dp = [0] * (len(cost))
        dp[0] = cost[0]
        dp[1] = cost[1]
        for i in range(2, len(cost)):
            dp[i] = min(dp[i - 1], dp[i - 2]) + cost[i]
        return min(dp[len(cost) - 1], dp[len(cost) - 2])
```

### 17、最长上升子序列

- 题目描述

  - 给你一个整数数组 nums ，找到其中最长严格递增子序列的长度。
  - 子序列是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，[3,6,2,7] 是数组 [0,3,1,6,2,2,7] 的子序列。

  示例 1： 

  输入：nums = [10,9,2,5,3,7,101,18] 

  输出：4 

  解释：最长递增子序列是 [2,3,7,101]，因此长度为 4 。

  示例 2： 

  输入：nums = [0,1,0,3,2,3] 

  输出：4

- 参考答案：

  - ```python
    class Solution:
        def lengthOfLIS(self, nums):
            if len(nums) <= 1:
                return len(nums)
            dp = [1] * len(nums)
            result = 0
            for i in range(1, len(nums)):
                for j in range(0, i):
                    if nums[i] > nums[j]:
                        dp[i] = max(dp[i], dp[j] + 1)
                result = max(result, dp[i]) #取长的子序列
            return result
    ```

### 18、最长公共子序列

- 题目描述

  - 给定两个字符串 text1 和 text2，返回这两个字符串的最长公共子序列的长度

  示例 1： 

  输入：text1 = "abcde", text2 = "ace" 

  输出：3 

  解释：最长公共子序列是 "ace"，它的长度为 3。

  示例 2: 输入：text1 = "abc", text2 = "abc"

   输出：3 

  解释：最长公共子序列是 "abc"，它的长度为 3

- 参考答案：

```python
class Solution:
    def longestCommonSubsequence(self, text1, text2):
        len1, len2 = len(text1)+1, len(text2)+1
        dp = [[0 for _ in range(len1)] for _ in range(len2)] # 先对dp数组做初始化操作
        for i in range(1, len2):
            for j in range(1, len1): # 开始列出状态转移方程
                if text1[j-1] == text2[i-1]:
                    dp[i][j] = dp[i-1][j-1]+1
                else:
                    dp[i][j] = max(dp[i-1][j], dp[i][j-1])
        return dp[-1][-1]
```

### 19、最长回文子串

- 题目描述

  - 给你一个字符串 `s`，找到 `s` 中最长的回文子串。

  示例 1: 

  ```
  输入：s = "babad"
  输出："bab"
  解释："aba" 同样是符合题意的答案。
  ```

  示例 2: 

  ```
  输入：s = "cbbd"
  输出："bb"
  ```

- 参考答案：

  - ```python
    class Solution:
        def longestPalindrome(self, s):
            n = len(s)
            if n < 2:
                return s
            
            max_len = 1
            begin = 0
            # dp[i][j] 表示 s[i..j] 是否是回文串
            dp = [[False] * n for _ in range(n)]
            for i in range(n):
                dp[i][i] = True
            
            # 递推开始
            # 先枚举子串长度
            for L in range(2, n + 1):
                # 枚举左边界，左边界的上限设置可以宽松一些
                for i in range(n):
                    # 由 L 和 i 可以确定右边界，即 j - i + 1 = L 得
                    j = L + i - 1
                    # 如果右边界越界，就可以退出当前循环
                    if j >= n:
                        break
                        
                    if s[i] != s[j]:
                        dp[i][j] = False 
                    else:
                        if j - i < 3:
                            dp[i][j] = True
                        else:
                            dp[i][j] = dp[i + 1][j - 1]
                    
        # 只要 dp[i][L] == true 成立，就表示子串 s[i..L] 是回文，此时记录回文长度和起始位置
                    if dp[i][j] and j - i + 1 > max_len:
                        max_len = j - i + 1
                        begin = i
            return s[begin:begin + max_len]
    
    ```

### 20、打家劫舍

- 题目描述

  - 你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。

    给定一个代表每个房屋存放金额的非负整数数组，计算你 不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。

  示例 1：

   输入：[1,2,3,1] 

  输出：4 

  解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。   偷窃到的最高金额 = 1 + 3 = 4 。

  示例 2：

   输入：[2,7,9,3,1] 

  输出：12 

  解释：偷窃 1 号房屋 (金额 = 2), 偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。   偷窃到的最高金额 = 2 + 9 + 1 = 12 。

- 参考答案：

  - ```python
    class Solution:
        def rob(self, nums):
            if len(nums) == 0:
                return 0
            if len(nums) == 1:
                return nums[0]
            dp = [0] * len(nums)
            dp[0] = nums[0]
            dp[1] = max(nums[0], nums[1])
            for i in range(2, len(nums)):
                dp[i] = max(dp[i-2]+nums[i], dp[i-1])
            return dp[-1]
    ```












































