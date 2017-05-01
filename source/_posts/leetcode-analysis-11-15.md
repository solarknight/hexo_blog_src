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

## 8. [String to Integer (atoi)](https://leetcode.com/problems/string-to-integer-atoi/#/description)

#### Description

Implement atoi to convert a string to an integer.

Requirements for atoi:
The function first discards as many whitespace characters as necessary until the first non-whitespace character is found. Then, starting from this character, takes an optional initial plus or minus sign followed by as many numerical digits as possible, and interprets them as a numerical value.
The string can contain additional characters after those that form the integral number, which are ignored and have no effect on the behavior of this function.
If the first sequence of non-whitespace characters in str is not a valid integral number, or if no such sequence exists because either str is empty or it contains only whitespace characters, no conversion is performed.
If no valid conversion could be performed, a zero value is returned. If the correct value is out of the range of representable values, INT_MAX (2147483647) or INT_MIN (-2147483648) is returned.

#### Analysis

不看提示很难跑过所有Test case的一道题。将字符串转换为整数的操作和上一题接近，都是在遍历过程中，将中间结果乘以10，再加上新的数字。
和上一题不同的是，即使使用64bit-int来保存中间结果，也有可能发生越界的问题。因此最好的处理方式是在遍历过程中加以判断。

#### Answer

```java
  public int myAtoi(String str) {
    if (str == null || str.length() == 0) {
      return 0;
    }
    char[] c = str.toCharArray();
    int idx = 0, flag = 1;
    long sum = 0;
    while (c[idx] == ' ') {
      idx++;
    }

    if (c[idx] == '-') {
      flag = -1;
      idx++;
    } else if (c[idx] == '+') {
      idx++;
    }

    for (; idx < c.length; idx++) {
      int t = c[idx] - '0';
      if (t < 0 || t > 9) {
        break;
      }
      sum = sum * 10 + t;
      if (sum > Integer.MAX_VALUE) {
        return flag == 1 ? Integer.MAX_VALUE : Integer.MIN_VALUE;
      }
    }
    return (int) sum * flag;
  }
```

## 9. [Palindrome Number](https://leetcode.com/problems/palindrome-number/#/description)

#### Description

Determine whether an integer is a palindrome. Do this without extra space.

#### Analysis

回文串是对称的，这一特性放到数字上，也就意味着对称轴一侧的数字反转后和另一侧相等。参考上面的数字反转算法，再对边界条件做处理，即可得到答案。
回文串长度为奇数和偶数时程序上判断条件不同，为奇数时要对较大的部分除以10再进行比较。

#### Answer

```java
  public boolean isPalindrome(int x) {
    if (x < 0) {
      return false;
    }
    if (x >= 0 && x < 10) {
      return true;
    }
    if (x % 10 == 0) {
      return false;
    }
    int rev = 0;
    for (; x > rev; x /= 10) {
      rev = 10 * rev + x % 10;
    }
    return x == rev || x == rev / 10;
  }
```

## 10. [Regular Expression Matching](https://leetcode.com/problems/regular-expression-matching/#/description)

#### Description

Implement regular expression matching with support for `'.'` and `'*'`.
```
'.' Matches any single character.
'*' Matches zero or more of the preceding element.

The matching should cover the entire input string (not partial).

The function prototype should be:
bool isMatch(const char *s, const char *p)

Some examples:
isMatch("aa","a") → false
isMatch("aa","aa") → true
isMatch("aaa","aa") → false
isMatch("aa", "a*") → true
isMatch("aa", ".*") → true
isMatch("ab", ".*") → true
isMatch("aab", "c*a*b") → true
```

#### Analysis

s[0..i]和p[0..j]是否匹配，取决于之前字符串的匹配情况，因此这一题比较适合用动态规划来解决。构建一个长度为`bool[len(s) + 1][len(p) + 1]`的数组，以减少对边界条件的处理。
初始化时，`dp[0][0]`处为true，其余元素为false。然后判断`dp[0][j](1 < j <= len(p))`时的值。
对于`dp[i][j](1 <= i <= len(s), 1 <= j <= len(p))`，如果`p[j] == s[i] || p[j] == '.'`，则有`dp[i + 1][j + 1] = dp[i][j]`。
如果`p[j] == '*'`，又需要对`p[j - 1] == s[i] || p[j - 1] == '.'`及不满足的情况分别考虑。

#### Answer

```java
  public boolean isMatch(String s, String p) {
    boolean[][] dp = new boolean[s.length() + 1][p.length() + 1];
    dp[0][0] = true;
    for (int i = 1; i < p.length(); i++) {
      dp[0][i + 1] = p.charAt(i) == '*' && dp[0][i - 1];
    }

    for (int i = 0; i < s.length(); i++) {
      for (int j = 0; j < p.length(); j++) {
        if (p.charAt(j) == s.charAt(i)) {
          dp[i + 1][j + 1] = dp[i][j];
        }
        if (p.charAt(j) == '.') {
          dp[i + 1][j + 1] = dp[i][j];
        }
        if (p.charAt(j) == '*') {
          if (p.charAt(j - 1) != s.charAt(i) && p.charAt(j - 1) != '.') {
            dp[i + 1][j + 1] = dp[i + 1][j - 1];
          } else {
            // zero || one || multi (count >= 2)
            dp[i + 1][j + 1] = dp[i + 1][j - 1] || dp[i + 1][j] || dp[i][j + 1];
          }
        }
      }
    }
    return dp[s.length()][p.length()];
  }
```

