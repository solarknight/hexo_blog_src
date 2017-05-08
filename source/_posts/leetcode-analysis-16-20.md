---
title: Leetcode题解 16-20
date: 2017-05-07 13:03:50
tags: algorithm
---

我的[答案链接](https://github.com/solarknight/algorithm_exercise/tree/master/leetcode/src/main)，目前有java/golang/ruby三种语言的答案。

## 16. [3Sum Closest](https://leetcode.com/problems/3sum-closest/#/description)

#### Description

Given an array S of n integers, find three integers in S such that the sum is closest to a given number, target. Return the sum of the three integers. You may assume that each input would have exactly one solution.

```
For example, given array S = {-1 2 1 -4}, and target = 1.
The sum that is closest to the target is 2. (-1 + 2 + 1 = 2).
```

#### Analysis

和上一题类似，同样是求三数之和，这次的要求是找出最接近输入值的结果。借鉴上一题的思路，首先对数组进行排序，然后从左边开始选定一个数，在剩下的数中找出最接近的。
由于这一次的不是寻找相等，而是寻找接近值，因此用`curShift`保存当前三数之和相对于目标输入的偏移量，`minShift`保存最小的偏移量绝对值。
遍历过程中如果出现`abs(curShift) < minShift`的情况，则更新`minShift`，并保存此时的三数之和。

#### Answer

```java
  public int threeSumClosest(int[] nums, int target) {
    if (nums == null || nums.length < 3) {
      return 0;
    }
    Arrays.sort(nums);
    int sum = 0, curShift = 0, minShift = Integer.MAX_VALUE;
    for (int i = 0; i < nums.length - 2; i++) {
      int j = i + 1, k = nums.length - 1, tar = target - nums[i];
      while (j < k) {
        curShift = nums[j] + nums[k] - tar;
        int abs = curShift < 0 ? -curShift : curShift;
        if (abs < minShift) {
          minShift = abs;
          sum = nums[i] + nums[j] + nums[k];
        }

        if (curShift == 0) {
          return target;
        } else if (curShift < 0) {
          j++;
        } else {
          k--;
        }
      }
    }
    return sum;
  }
```

## 17. [Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/#/description)

#### Description

Given a digit string, return all possible letter combinations that the number could represent.
A mapping of digit to letters (just like on the telephone buttons) is given below.
```
Input:Digit string "23"
Output: ["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].
```

#### Analysis

题目本身理解起来并不难，保存0-9数字对应字符的常量表，在解析时查表即可。
这道题主要考察如何动态根据下一个字符可能的个数扩充现有的结果集。有递归和循环两种解法，下面给出的是使用循环的解法。
对递归和循环两种解法分析如下：
  - 循环中每次构建一个临时列表，通过该列表存储变更的中间结果，并将其作为当前结果集应用到下一次循环中。注意初始化时在结果集中添加一个空字符可以避免对边界条件的处理。
  - 递归要注意递归方法的构造，该方法需要有当前递归执行的必要参数和结果，在这里即为输入字符、当前位置、当前结果、保存最终结果的列表。递归方法中首先判断是否到达边界条件，如到达则返回，否则更新参数，展开后续的递归执行。

#### Answer

```java
    private static final char[][] LETTERS = {{' '}, {' '}, {'a', 'b', 'c'}, {'d', 'e', 'f'}, {'g', 'h', 'i'}, {'j', 'k', 'l'}, {'m', 'n', 'o'},
        {'p', 'q', 'r', 's'}, {'t', 'u', 'v'}, {'w', 'x', 'y', 'z'}};

    public List<String> letterCombinations(String digits) {
      if (digits.length() == 0) {
        return new ArrayList<>();
      }
      List<String> res = new ArrayList<>();
      res.add("");

      for (char c : digits.toCharArray()) {
        int t = c - '0';
        List<String> tmp = new ArrayList<>(res.size() * LETTERS[t].length);
        for (char p : LETTERS[t]) {
          for (String s : res) {
            tmp.add(s + p);
          }
        }
        res = tmp;
      }
      return res;
    }
```

## 18. [4Sum](https://leetcode.com/problems/4sum/#/description)

#### Description

Given an array S of n integers, are there elements a, b, c, and d in S such that a + b + c + d = target? Find all unique quadruplets in the array which gives the sum of target.
**Note**: The solution set must not contain duplicate quadruplets.
```
For example, given array S = [1, 0, -1, 0, -2, 2], and target = 0.
A solution set is:
[
  [-1,  0, 0, 1],
  [-2, -1, 1, 2],
  [-2,  0, 0, 2]
]
```

#### Analysis

和之前3Sum很类似的一道题，同样有多个可能解，同样要求对解集去重。参考3Sum的解法，首先对数组排序，然后从左边选定第一个元素，问题即转化为了3Sum的问题。

#### Answer

```java
  public List<List<Integer>> fourSum(int[] nums, int target) {
    List<List<Integer>> res = new ArrayList<>();
    if (nums == null || nums.length < 4) {
      return res;
    }
    Arrays.sort(nums);
    for (int i = 0; i < nums.length - 3; i++) {
      if (i != 0 && nums[i] == nums[i - 1]) {
        continue;
      }
      for (int j = i + 1; j < nums.length - 2; j++) {
        if (j != i + 1 && nums[j] == nums[j - 1]) {
          continue;
        }
        int k = j + 1, l = nums.length - 1, tar = target - nums[i] - nums[j];
        while (k < l) {
          if (nums[k] + nums[l] == tar) {
            res.add(Arrays.asList(nums[i], nums[j], nums[k], nums[l]));
            k++;
            l--;
            while (k < l && nums[k] == nums[k - 1]) {
              k++;
            }
            while (k < l && nums[l] == nums[l + 1]) {
              l--;
            }
          } else if (nums[k] + nums[l] < tar) {
            k++;
          } else {
            l--;
          }
        }
      }
    }
    return res;
  }
```

## 19. [Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/#/description)

#### Description

Given a linked list, remove the nth node from the end of list and return its head.
For example,
```
   Given linked list: 1->2->3->4->5, and n = 2.
   After removing the second node from the end, the linked list becomes 1->2->3->5.
```
**Note**:
Given n will always be valid.
Try to do this in one pass.

#### Analysis

参考判断单链表是否有环问题的解决思路。使用两个指针，其中一个指针比另一个多走`n + 1`步。然后同时移动两个指针，当前面那个指向`null`时，后面的指针正处在删除节点的合适位置。
注意当删除节点是首节点时，需要返回下一个节点作为首节点。这里可以采用初始化一个冗余节点作为首节点的方式，规避对于边界条件的处理。

#### Answer

```java
  public ListNode removeNthFromEnd(ListNode head, int n) {
    if (head.next == null) {
      return null;
    }
    ListNode dummy = new ListNode(0), cur1 = dummy, cur2 = dummy;
    dummy.next = head;
    for (int i = 0; i <= n; i++) {
      cur2 = cur2.next;
    }
    while (cur2 != null) {
      cur1 = cur1.next;
      cur2 = cur2.next;
    }
    cur1.next = cur1.next.next;
    return dummy.next;
  }
```

## 20. [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/#/description)

#### Description

Given a string containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.
The brackets must close in the correct order, "()" and "()[]{}" are all valid but "(]" and "([)]" are not.

#### Analysis

参考四则运算类编程问题，这类题目适合使用栈来处理。通过在push时放入对应的匹配元素，可以简化条件处理。如果在放入元素时，无法和栈顶元素匹配，并且非`(`、`[`、`{`，则可以直接返回false。

#### Answer

```java
  public boolean isValid(String s) {
    if (s.length() == 0) {
      return true;
    }
    if (s.length() == 1) {
      return false;
    }
    Stack<Character> stack = new Stack<>();
    for (char c : s.toCharArray()) {
      if (stack.size() != 0 && stack.peek() == c) {
        stack.pop();
        continue;
      }
      if (c == '(') {
        stack.push(')');
      } else if (c == '[') {
        stack.push(']');
      } else if (c == '{') {
        stack.push('}');
      } else {
        return false;
      }
    }
    return stack.size() == 0;
  }
```
