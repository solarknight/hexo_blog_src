---
title: Leetcode题解 11-15
date: 2017-04-30 12:10:39
tags: algorithm
---

我的[答案链接](https://github.com/solarknight/algorithm_exercise/tree/master/leetcode/src/main)，目前有java/golang/ruby三种语言的答案。

## 11. [Container With Most Water](https://leetcode.com/problems/container-with-most-water/#/description)

#### Description

Given n non-negative integers a<sub>1</sub>, a<sub>2</sub>, ..., a<sub>n</sub>, where each represents a point at coordinate (i, a<sub>i</sub>). n vertical lines are drawn such that the two endpoints of line i is at (i, a<sub>i</sub>) and (i, 0). Find two lines, which together with x-axis forms a container, such that the container contains the most water.

Note: You may not slant the container and n is at least 2.

#### Analysis

由`a[i]`和`a[j]`构成的容器，其面积为`(j - i) * min(a[i], a[j])`。最左和最右两点构成的容器拥有最大的宽度，可以从这里开始遍历。
如果要比初始的容器面积大，则需要有更大的高度。因此可以逐渐缩小范围，直至获得最大值。

#### Answer

```java
    public int maxArea(int[] height) {
        int max = 0;
        for (int i = 0, j = height.length - 1; i < j; ) {
          max = Math.max(max, (j - i) * Math.min(height[i], height[j]));
          if (height[i] < height[j]) {
            i++;
          } else {
            j--;
          }
        }
        return max;
    }
```

## 12. [Integer to Roman](https://leetcode.com/problems/integer-to-roman/#/description)

#### Description

Given an integer, convert it to a roman numeral.
Input is guaranteed to be within the range from 1 to 3999.

#### Analysis

首先要知道各个罗马数字，以及它们与十进制整数之间的转换关系。详细可见[维基百科](https://zh.wikipedia.org/wiki/%E7%BD%97%E9%A9%AC%E6%95%B0%E5%AD%97)。
罗马数字共有7个，即Ⅰ（1）、Ⅴ（5）、Ⅹ（10）、Ⅼ（50）、Ⅽ（100）、Ⅾ（500）和Ⅿ（1000）。按照下述的规则可以表示任意正整数。需要注意的是罗马数字中没有“0”，与进位制无关。
  - 重复数次：一个罗马数字重复几次，就表示这个数的几倍。
  - 右加左减：
    - 在较大的罗马数字的右边记上较小的罗马数字，表示大数字加小数字。
    - 在较大的罗马数字的左边记上较小的罗马数字，表示大数字减小数字。
    - 左减的数字有限制，仅限于I、X、C。比如45不可以写成VL，只能是XLV
    - 但是，左减时不可跨越一个位值。比如，99不可以用IC表示，而是用XCIX表示。（等同于阿拉伯数字每位数字分别表示。）
    - 左减数字必须为一位，比如8写成VIII，而非IIX。
    - 右加数字不可连续超过三位，比如14写成XIV，而非XIIII。（见下方“数码限制”一项。） 
  - 加线乘千: 
    - 在罗马数字的上方加上一条横线或者加上下标的Ⅿ，表示将这个数乘以1000，即是原数的1000倍。
    - 同理，如果上方有两条横线，即是原数的1000000倍。
  - 数码限制: 同一数码最多只能连续出现三次，如40不可表示为XXXX，而要表示为XL。

由于输入已经限制为了[1, 3999]，因此不需考虑加线乘千的规则。由于每一位的取值都是有限个，因此完全可以记录一个常量表，然后使用查表的方式来解决。
如果不想记录每一位所有数字，也可以保存一个所选范围内左减数字的全量表，这样只需处理右加的情况即可。

#### Answer

```java
    private static final String[] M = {"", "M", "MM", "MMM"};
    private static final String[] C = {"", "C", "CC", "CCC", "CD", "D", "DC", "DCC", "DCCC", "CM"};
    private static final String[] X = {"", "X", "XX", "XXX", "XL", "L", "LX", "LXX", "LXXX", "XC"};
    private static final String[] I = {"", "I", "II", "III", "IV", "V", "VI", "VII", "VIII", "IX"};
    
    public String intToRoman(int num) {
        if (num < 10) {
          return I[num];
        } else if (num < 100) {
          return X[num / 10] + I[num % 10];
        } else if (num < 1000) {
          return C[num / 100] + X[(num % 100) / 10] + I[num % 10];
        } else {
          return M[num / 1000] + C[(num % 1000) / 100] + X[(num % 100) / 10] + I[num % 10];
        }
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
    if (nums.length == 0 || nums.length < 3) {
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

