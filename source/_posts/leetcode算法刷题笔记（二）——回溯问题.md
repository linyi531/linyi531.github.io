---
title: leetcode算法刷题笔记（二）——回溯问题
date: 2019-01-15 22:07:43
tags:
  - 算法
  - leetcode
  - 回溯
  - javascript
categories: 算法
cover_img: https://i.screenshot.net/o9k85cw
feature_img: https://i.screenshot.net/o9k85cw
---

_该笔记只为个人所写算法，不一定是最优解法，仅供参考_

# [17] Letter Combinations of a Phone Number

Given a string containing digits from 2-9 inclusive, return all possible letter combinations that the number could represent.
A mapping of digit to letters (just like on the telephone buttons) is given below. Note that 1 does not map to any letters.
Note:Although the above answer is in lexicographical order, your answer could be in any order you want.
难度：Medium (40.58%)
考点：回溯

<!-- more -->

```javascript
var letterCombinations = function(digits) {
  var res = [];
  var sort = [];
  if (digits.length == 0) {
    return res;
  }
  var phone = [
    "0",
    "1",
    ["a", "b", "c"],
    ["d", "e", "f"],
    ["g", "h", "i"],
    ["j", "k", "l"],
    ["m", "n", "o"],
    ["p", "q", "r", "s"],
    ["t", "u", "v"],
    ["w", "x", "y", "z"]
  ];
  finger(0);
  return res;

  function finger(index) {
    if (index == digits.length) {
      return res.push(sort.join(""));
    }
    var temp = phone[digits[index]];
    for (var i = 0; i < temp.length; i++) {
      sort.push(temp[i]);
      finger(index + 1);
      sort.pop();
    }
  }
};
```

# [22] Generate Parentheses

Given n pairs of parentheses, write a function to generate all combinations of well-formed parentheses.
难度：Medium (53.42%)
考点：回溯
思路：

1. 给定 n 值，则总共有 n 个左括号，n 个右括号。
2. 第一个添加的一定是左括号。
3. 当添加了一个左括号之后，才会有一个右括号可以添加。所以回溯时，left-1 的同时 right+1
4. 当已添加了 n 个左括号后，剩下的都应该添加右括号

```javascript
var generateParenthesis = function(n) {
  var res = [];
  var left = n - 1;
  var right = 1;
  function quote(left, right, str) {
    if (left <= 0) {
      if (right) {
        for (var i = 0; i < right; i++) {
          str = str + ")";
        }
      }
      return res.push(str);
    }
    quote(left - 1, right + 1, str + "(");
    if (right > 0) {
      quote(left, right - 1, str + ")");
    }
  }
  quote(left, right, "(");
  return res;
};
```

# [46] Permutations

Given a collection of distinct integers, return all possible permutations.
难度：Medium (53.67%)
考点：回溯
思路：选择一个元素之后，则下次可选择的元素就少一个。

```javascript
var permute = function(nums) {
  var res = [];
  var sort = [];
  if (nums.length == 0) {
    return res;
  }
  select(nums);
  return res;

  function select(nums) {
    if (nums.length < 1) {
      return res.push(sort.slice());
    }
    for (var i = 0; i < nums.length; i++) {
      var nextNums = nums.slice();
      sort.push(nextNums[i]);
      nextNums.splice(i, 1);
      select(nextNums);
      sort.pop();
    }
  }
};
```

# [47] Permutations II

Given a collection of numbers that might contain duplicates, return all possible unique permutations.
难度：Medium (39.35%)
考点：回溯
思路：思路同上题。注意筛选条件。

```javascript
var permuteUnique = function(nums) {
  var res = [];
  var sort = [];
  if (nums.length == 0) {
    return res;
  }
  nums = nums.sort((a, b) => {
    return a - b;
  });
  select(nums);
  return res;

  function select(nums) {
    if (nums.length < 1) {
      return res.push(sort.slice());
    }
    for (var i = 0; i < nums.length; i++) {
      if (nums[i] == nums[i - 1]) {
        continue;
      }
      var nextNums = nums.slice();
      sort.push(nextNums[i]);
      nextNums.splice(i, 1);
      select(nextNums);
      sort.pop();
    }
  }
};
```

# [60] Permutations II ☆☆

The set [1,2,3,...,n] contains a total of n! unique permutations.By listing and labeling all of the permutations in order, we get the following sequence for n = 3:
"123"
"132"
"213"
"231"
"312"
"321"
Given n and k, return the k^th permutation sequence.
Note:Given n will be between 1 and 9 inclusive. Given k will be between 1 and n! inclusive.
难度：Medium (32.42%)
考点：回溯

```javascript
var getPermutation = function(n, k) {
  (res = []), (pos = k - 1);
  var nums = [];
  if (n == 0) {
    return "error";
  }
  for (var i = 0; i < n; i++) {
    nums[i] = i + 1;
  }
  var numsSort = nums.reduce((a, b) => a * b);
  if (k < 1 || k > numsSort) {
    return "error";
  }

  for (var j = n; j >= 1; --j) {
    numsSort /= j;
    res.push(nums.splice(parseInt(pos / numsSort), 1)[0]);
    pos %= numsSort;
  }
  return res.join("");
};
```

# [77] ombinations ☆☆

Given two integers n and k, return all possible combinations of k numbers out of 1 ... n.
难度：Medium (46.23%)
考点：回溯
难点：下一次选择不能选择比上一次小的数，所以需注意 push 进去的条件

```javascript
var combine = function(n, k) {
  var nums = [];
  var res = [];
  var temp = [];
  if (n == 0 || k <= 0 || k > n) {
    return "error";
  }
  for (var i = 0; i < n; i++) {
    nums[i] = i + 1;
  }

  select(0, nums);
  return res;

  function select(start, nums) {
    if (temp.length == k) {
      return res.push(temp.slice());
    }
    for (var i = start; i < n; i++) {
      if (temp.length >= 1 && temp[temp.length - 1] > i) {
        continue;
      }
      temp.push(nums[i]);
      select(start + 1, nums);
      temp.pop();
    }
  }
};
```

# [78] Subsets

Given a set of distinct integers, nums, return all possible subsets (the power set).
Note: The solution set must not contain duplicate subsets.
难度：Medium (51.26%)
考点：回溯
难点：下一次选择不能选择比上一次小的数，所以需注意 push 进去的条件。

```javascript
var subsets = function(nums) {
  var res = [];
  var subsets = [];
  var used = [];
  res.push(subsets.slice());
  if (nums.length == 0) {
    return res;
  }
  nums = nums.sort((a, b) => a - b);
  for (var j = 1; j <= nums.length; j++) {
    findSubsets(0, j);
  }
  return res;

  function findSubsets(start, k) {
    for (var i = 0; i < nums.length; i++) {
      if (subsets.length == k) {
        return res.push(subsets.slice());
      }
      if (used[i]) {
        continue;
      }
      if (start > 0 && nums[i] < subsets[subsets.length - 1]) {
        continue;
      }
      subsets.push(nums[i]);
      used[i] = true;
      findSubsets(start + 1, k);
      subsets.pop();
      used[i] = false;
    }
  }
};
```

# [79] Word Search

Given a 2D board and a word, find if the word exists in the grid.
The word can be constructed from letters of sequentially adjacent cell, where "adjacent" cells are those horizontally or vertically neighboring. The same letter cell may not be used more than once.
难度：Medium (30.52%)
考点：回溯
思路：要分四个方向分别回溯。

```javascript
var exist = function(board, word) {
  var row = board.length;
  var col = board[0].length;
  if (word.length > row * col) {
    return false;
  }
  function search(i, j, n) {
    if (
      i >= row ||
      j >= col ||
      i < 0 ||
      j < 0 ||
      board[i][j] != word[n] ||
      n > word.length
    ) {
      return false;
    }
    if (n == word.length - 1) {
      return true;
    }
    board[i][j] = true;
    if (search(i + 1, j, n + 1)) {
      return true;
    }
    if (search(i - 1, j, n + 1)) {
      return true;
    }
    if (search(i, j + 1, n + 1)) {
      return true;
    }
    if (search(i, j - 1, n + 1)) {
      return true;
    }
    board[i][j] = word[n];
    return false;
  }
  for (var i = 0; i < row; i++) {
    for (var j = 0; j < col; j++) {
      if (search(i, j, 0)) {
        return true;
      }
    }
  }
  return false;
};
```

# [89] Gray Code ☆☆

The gray code is a binary numeral system where two successive values differ in only one bit.
Given a non-negative integer n representing the total number of bits in the code, print the sequence of gray code. A gray code sequence must begin with 0.
难度：Medium (45.03%)
考点：回溯
思路：可根据格雷码的特性考虑
解法一（普通解法）：

```javascript
var grayCode = function(n) {
  var result = [];

  var graycodeFn = function(n) {
    var graycode = [];

    if (n == 1) {
      graycode[0] = "0";
      graycode[1] = "1";
      return graycode;
    }

    var last = arguments.callee(n - 1); // arguments.callee(n-1) == graycodeFn(n-1)

    for (var i = last.length - 1; i >= 0; --i) {
      graycode.unshift("0" + last[i]);
      graycode.push("1" + last[i]);
    }

    return graycode;
  };

  var graycode = n == 0 ? ["0"] : graycodeFn(n);

  for (var i = 0; i < graycode.length; ++i) {
    result.push(parseInt(parseInt(graycode[i], 2), 10)); // String To Number
  }

  return result;
};
```

解法二（大神解法）：

```javascript
var grayCode = function(n) {
  let nums = [0],
    c = -1;
  while (c++ < n - 1)
    nums = [...nums, ...nums.map(num => num + Math.pow(2, c)).reverse()];
  return nums;
};
```

# [90] Subsets II

Given a collection of integers that might contain duplicates, nums, return all possible subsets (the power set).
Note: The solution set must not contain duplicate subsets.
难度：Medium (41.57%)
考点：回溯

```javascript
var subsetsWithDup = function(nums) {
  var sub = [];
  var res = [];
  res.push(sub.slice());
  if (nums.length == 0) {
    return res;
  }
  nums = nums.sort((a, b) => a - b);

  for (var i = 1; i <= nums.length; i++) {
    findSub(0, i);
  }
  return res;

  function findSub(start, k) {
    if (sub.length == k) {
      return res.push(sub.slice());
    }
    for (var j = start; j < nums.length; j++) {
      if (j > start && nums[j] == nums[j - 1]) {
        continue;
      }
      sub.push(nums[j]);
      findSub(j + 1, k);
      sub.pop();
    }
  }
};
```
