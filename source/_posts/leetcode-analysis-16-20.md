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

## 13. [Roman to Integer](https://leetcode.com/problems/roman-to-integer/#/description)

#### Description

Given a roman numeral, convert it to an integer.
Input is guaranteed to be within the range from 1 to 3999.

#### Analysis

由上一题可知罗马数字和十进制数之间的转换规则。这里主要考察左减右加的转换：当左边的数字小于右边时，对应的大小为右边的数字减去左边的数字。当左边的数字大于等于右边时，正常加和即可。

#### Answer

```java
  public int romanToInt(String s) {
    if (s.length() == 0) {
      return 0;
    }
    int sum = 0, last = 0, cur = 0;
    for (int i = 0; i < s.length(); i++) {
      cur = convert(s.charAt(i));
      sum += last >= cur ? cur : cur - 2 * last;
      last = cur;
    }
    return sum;
  }

  private int convert(char c) {
    switch (c) {
      case 'I':
        return 1;
      case 'V':
        return 5;
      case 'X':
        return 10;
      case 'L':
        return 50;
      case 'C':
        return 100;
      case 'D':
        return 500;
      case 'M':
        return 1000;
    }
    return 0;
  }
```

## 14. [Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/#/description)

#### Description

Write a function to find the longest common prefix string amongst an array of strings.

#### Analysis

两种思路：
- 从第一个字符开始，逐一比较所有字符串，直到有不匹配的（字符不相同或者超出长度）
- 求出第0个和第1个字符串的公共前缀，再依次和其他字符串求公共前缀
第二种方法可以较多利用语言原生API，代码较为简洁。

#### Answer

```java
  public String longestCommonPrefix(String[] strs) {
      if (strs.length == 0) return "";
      String pre = strs[0];
      for (int i = 1; i < strs.length; i++) {
          while (strs[i].indexOf(pre) != 0)
              pre = pre.substring(0, pre.length() - 1);
      }
      return pre;
  }
```

## 15. [3Sum](https://leetcode.com/problems/3sum/#/description)

#### Description

Given an array S of n integers, are there elements a, b, c in S such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.
Note: The solution set must not contain duplicate triplets.
```
For example, given array S = [-1, 0, 1, 2, -1, -4],

A solution set is:
[
  [-1, 0, 1],
  [-1, -1, 2]
]
```

#### Analysis

前面我们已经解决过`Two sum`的问题，但这一题还是有些不同。这次我们需要给出所有可行解的全集，并且不能重复。
在固定三个数中的一个数之后，问题可以转化为在剩余的数字中求两数之和的问题。再加上这个问题的条件，合理的方式是对数组进行排序，然后再进行处理。
当数组排序后，遍历过程中可以跳过重复的相邻元素，来达到简化计算和去重的目的。

#### Answer

```java
  public List<List<Integer>> threeSum(int[] nums) {
    List<List<Integer>> list = new ArrayList<List<Integer>>();
    if (nums == null || nums.length < 3) {
      return list;
    }
    Arrays.sort(nums);
    for (int i = 0; i < nums.length - 2; i++) {
      if (i != 0 && nums[i] == nums[i - 1]) {
        continue;
      }
      int tar = -nums[i], j = i + 1, k = nums.length - 1;
      while (j < k) {
        if (nums[j] + nums[k] == tar) {
          list.add(Arrays.asList(nums[i], nums[j], nums[k]));
          j++;
          k--;
          while (j < k && nums[j] == nums[j - 1]) {
            j++;
          }
          while (j < k && nums[k] == nums[k + 1]) {
            k--;
          }
        } else if (nums[j] + nums[k] < tar) {
          j++;
        } else {
          k--;
        }
      }
    }
    return list;
  }
```
