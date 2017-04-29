---
title: Leetcode题解 1-5
date: 2017-04-29 20:59:09
tags: algorithm
---
[Leetcode](https://leetcode.com/problemset/algorithms)上面有大量简洁又不失难度的算法题，是磨炼功底的好地方。
对于部分题目，网上的答案只有简单的`how to do it`，而缺少`why`的分析过程。因此在这里记录一下解题思路，以便温故而知新。
所有分析都是基于时间复杂度最优的解法，这里是我的[答案链接](https://github.com/solarknight/algorithm_exercise/tree/master/leetcode/src/main)，目前有java/golang/ruby三种语言的答案。

## 1. [Two Sum](https://leetcode.com/problems/two-sum/#/description)

#### Description

Given an array of integers, return indices of the two numbers such that they add up to a specific target.
You may assume that each input would have exactly one solution, and you may not use the same element twice.
```
Given nums = [2, 7, 11, 15], target = 9,
Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

#### Analysis

数组是未排序数组，因此不能通过二分查找这样的方法来寻找特定元素。直接暴力寻找的话，需要O(N<sup>2</sup>)的时间复杂度。由于我们至少要遍历一次数组，如果在遍历到下标为`i`的数组元素时，能以O(1)的时间从前面所有元素中找到符合的结果，即可达到O(N)的时间复杂度。基于哈希的结构可以做到这一点。

#### Answer

```java
  public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<Integer, Integer>();

    for (int i = 0; i < nums.length; i++) {
      int tmp = target - nums[i];
      if (map.containsKey(tmp)) {
        return new int[]{map.get(tmp), i};
      } else {
        map.put(nums[i], i);
      }
    }
    return null;
  }
```

## 2. [Add Two Numbers](https://leetcode.com/problems/add-two-numbers/#/description)

#### Description

You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.
You may assume the two numbers do not contain any leading zero, except the number 0 itself.
```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
```

#### Analysis

这道题主要考察链表的操作，以及计算两数之和后，对进位的处理。保存新结果的链表至少需要两个指针，一个指向当前位置，用于程序处理。另一个指向链表头，用于结果返回。在使用链表的指针时，还要注意判空。
为了方便程序处理，设置了一个额外的`ListNode`作为`Head`，这种技巧在后面的题目中也多次使用。

#### Answer

```java
  public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode head = new ListNode(0);
    ListNode cur = head;
    ListNode curL1 = l1;
    ListNode curL2 = l2;

    int carry = 0;
    int sum;
    while (curL1 != null || curL2 != null || carry != 0) {
      sum = (curL1 != null ? curL1.val : 0) + (curL2 != null ? curL2.val : 0) + carry;
      carry = sum / 10;
      cur.next = new ListNode(sum % 10);
      cur = cur.next;
      curL1 = curL1 != null ? curL1.next : null;
      curL2 = curL2 != null ? curL2.next : null;
    }
    return head.next;
  }
```

## 3. [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/#/description)

#### Description

Given a string, find the length of the longest substring without repeating characters.
```
Given "abcabcbb", the answer is "abc", which the length is 3.
Given "bbbbb", the answer is "b", with the length of 1.
Given "pwwkew", the answer is "wke", with the length of 3. Note that the answer must be a substring, "pwke" is a subsequence and not a substring.
```

#### Analysis

和第一题的思路类似，在一次遍历过程中，通过哈希结构记录各字符最近的位置。当找到相同的字符时，判断当前子字符串长度是否大于之前记录的最长不重复子串，如大于则更新。

#### Answer

```java
  public int lengthOfLongestSubstring(String s) {
    char[] c = s.toCharArray();
    Map<Character, Integer> map = new HashMap<Character, Integer>();
    int pIdx = 0, idx = 0, max = 0;

    for (; idx < c.length; idx++) {
      if (map.containsKey(c[idx])) {
        max = (idx - pIdx) > max ? (idx - pIdx) : max;
        pIdx = (map.get(c[idx]) + 1 > pIdx) ? (map.get(c[idx]) + 1) : pIdx;
      }
      map.put(c[idx], idx);
    }
    max = (idx - pIdx) > max ? (idx - pIdx) : max;
    return max;
  }
```

## 4. [Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/#/description)

#### Description

There are two sorted arrays nums1 and nums2 of size m and n respectively.
Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).
```
nums1 = [1, 3]
nums2 = [2]
The median is 2.0
```

```
nums1 = [1, 2]
nums2 = [3, 4]
The median is (2 + 3)/2 = 2.5
```

#### Analysis

[这里解释的非常详细](https://leetcode.com/problems/median-of-two-sorted-arrays/#/solutions)。
注意其中为了方便处理奇数和偶数长度的数组，添加了额外的数组元素。

#### Answer

```java
  public static double findMedianSortedArrays(int[] nums1, int[] nums2) {
    int l1 = nums1.length, l2 = nums2.length;
    if (l1 > l2) {
      return findMedianSortedArrays(nums2, nums1);
    }

    if (l1 == 0) {
      return (nums2[(l2 - 1) / 2] + nums2[l2 / 2]) / 2.0;
    }

    int left = 0, right = 2 * l1;
    while (left <= right) {
      int mid1 = (left + right) / 2;
      int mid2 = l1 + l2 - mid1;

      int lv1 = mid1 == 0 ? Integer.MIN_VALUE : nums1[(mid1 - 1) / 2];
      int rv1 = mid1 == 2 * l1 ? Integer.MAX_VALUE : nums1[mid1 / 2];
      int lv2 = mid2 == 0 ? Integer.MIN_VALUE : nums2[(mid2 - 1) / 2];
      int rv2 = mid2 == 2 * l2 ? Integer.MAX_VALUE : nums2[mid2 / 2];

      if (lv1 <= rv2 && lv2 <= rv1) {
        return (Math.max(lv1, lv2) + Math.min(rv1, rv2)) / 2.0;
      } else if (lv1 > rv2) {
        right = mid1 - 1;
      } else {
        left = mid1 + 1;
      }
    }
    return -1;
  }
```

## 5. [Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/#/description)

#### Description

Given a string s, find the longest palindromic substring in s. You may assume that the maximum length of s is 1000.
```
Input: "babad"
Output: "bab"
Note: "aba" is also a valid answer.
```

```
Input: "cbbd"
Output: "bb"
```

#### Analysis

回文子串可能是奇数长度，也可能是偶数长度。因此，这里为了方便处理，也用了和上一题相似的技巧，在原字符串中加入`#`分隔符。
同时，在左右两边分别加入不相等的`^`、`$`符号以方便处理。
`Manacher's algorithm`可以在O(n)的时间内解决这道题。它的核心思想是保存一个记录回文串长度的数组，并利用回文串对称的特性，减少字符遍历和比较次数。
注意网上很多答案在计算出回文串长度的数组后，又对其进行遍历以求最大值，其实这个操作可以通过记录最大长度来优化。


#### Answer

```java
  public String longestPalindrome(String s) {
    if (s == null || s.length() <= 1) {
      return s;
    }

    char[] c = process(s);
    int[] p = new int[c.length];
    int idx = 0, right = 0, mIdx = 0;

    for (int i = 1; i < c.length - 1; i++) {
      p[i] = right > i ? Math.min(p[2 * idx - i], right - i) : 0;
      while (c[i + p[i] + 1] == c[i - p[i] - 1]) {
        p[i]++;
      }
      if (i + p[i] > right) {
        right = i + p[i];
        idx = i;
      }
      if (p[i] > p[mIdx]) {
        mIdx = i;
      }
    }

    int start = (mIdx - p[mIdx] - 1) / 2;
    return s.substring(start, start + p[mIdx]);
  }

  private char[] process(String s) {
    char[] origin = s.toCharArray();
    char[] after = new char[origin.length * 2 + 3];

    after[0] = '^';
    after[after.length - 1] = '$';
    for (int i = 1, j = 0; i < after.length - 1; i++) {
      after[i] = i % 2 != 0 ? '#' : origin[j++];
    }
    return after;
  }
```