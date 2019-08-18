---
title: leetcode算法刷题笔记（三）——排序
date: 2019-02-22 14:54:39
tags:
  - 算法
  - leetcode
  - 排序
  - javascript
categories: 算法
cover_img: https://i.screenshot.net/8onywck
feature_img: https://i.screenshot.net/8onywck
---

_该笔记只为个人所写算法，不一定是最优解法，仅供参考_

# [56] Merge Intervals

Given a collection of intervals, merge all overlapping intervals.
难度：Medium (34.95%)
思路：

1. 首先按 start 升序或按 end 升序排列
2. 如果前一项的 end 大于后一项的 start，说明要 merge
3. 因为已经做了升序排列，所以要 push 进结果的那一项的 start 一定为前一项的 start，而 end 为两项中 end 较大的那个值。

<!-- more -->

```javascript
var merge = function(intervals) {
  var res = [];
  if (intervals.length == 0) {
    return res;
  }
  intervals.sort(function(a, b) {
    return a.start !== b.start ? a.start - b.start : a.end - b.end;
  });
  var pre = intervals[0];
  res.push(pre);
  for (var cur of intervals) {
    if (pre.end >= cur.start) {
      if (cur.end > pre.end) {
        pre.end = cur.end;
      }
    } else {
      res.push(cur);
      pre = cur;
    }
  }
  return res;
};
```

# [75] Sort Colors

Given an array with n objects colored red, white or blue, sort them in-place so that objects of the same color are adjacent, with the colors in the order red, white and blue.
Here, we will use the integers 0, 1, and 2 to represent the color red, white, and blue respectively.
Note: You are not suppose to use the library's sort function for this problem.
难度：Medium (41.44%)
思路：遇到 0 放在最前面，遇到 2 放在最后面，1 不动

1. i 从 0 开始，如果 nums[i]等于 1，则 i++。
2. 如果 nums[i]等于 0，则把 nums[i]放到最前面
3. 如果 nums[i]等于 2，则把 nums[i]放到最后面

```javascript
var sortColors = function(nums) {
  if (nums.length == 0) {
    return -1;
  }
  var i = 0;
  var m = 0;
  var n = nums.length - 1;
  var temp;
  while (i <= n) {
    if (nums[i] == 1) {
      i++;
    } else if (nums[i] == 0) {
      temp = nums[m];
      nums[m] = nums[i];
      nums[i] = temp;
      m++;
      i++;
    } else {
      temp = nums[n];
      nums[n] = nums[i];
      nums[i] = temp;
      n--;
    }
  }
  return nums;
};
```

# [147] Insertion Sort List

Sort a linked list using insertion sort.
A graphical example of insertion sort. The partial sorted list (black) initially contains only the first element in the list.With each iteration one element (red) is removed from the input data and inserted in-place into the sorted list.
Algorithm of Insertion Sort:
Insertion sort iterates, consuming one input element each repetition, and growing a sorted output list. At each iteration, insertion sort removes one element from the input data, finds the location it belongs within the sorted list, and inserts it there. It repeats until no input elements remain.
难度：Medium (36.50%)
思路：新开辟一个空链表，作为以排序区域，每次拿出 head 来，与以排序区域的链表的 val 进行比较，找到插入位置，插入。则未排序区域的了链表长度减一。

```javascript
var insertionSortList = function(head) {
  var current = { val: -Number.MAX_VALUE, next: null };
  while (head) {
    var prev = current;
    while (prev.next && prev.next.val < head.val) {
      prev = prev.next;
    }
    var next = head.next;
    head.next = prev.next;
    prev.next = head;

    head = next;
  }
  return current.next;
};
```

# [148] Sort List

Sort a linked list in O(n log n) time using constant space complexity.
难度：Medium (34.12%)
思路：归并排序的链表实现

```javascript
var sortList = function(head) {
  if (head == null || head.next == null) {
    return head;
  }
  var fast = head;
  var slow = head;
  var pre = null;
  while (fast && fast.next != null) {
    pre = slow;
    slow = slow.next;
    fast = fast.next.next;
  }
  pre.next = null;
  return merge(sortList(head), sortList(slow));
};

function merge(left, right) {
  var result = {};
  var pre = result;
  while (left && right) {
    if (left.val < right.val) {
      pre.next = left;
      pre = pre.next;
      left = left.next;
    } else {
      pre.next = right;
      pre = pre.next;
      right = right.next;
    }
  }
  while (left) {
    pre.next = left;
    pre = pre.next;
    left = left.next;
  }
  while (right) {
    pre.next = right;
    pre = pre.next;
    right = right.next;
  }
  return result.next;
}
```

# [179] Largest Number

Given a list of non negative integers, arrange them such that they form the largest number.
难度：Medium (25.32%)

```javascript
var largestNumber = function(nums) {
  nums.sort(function(a, b) {
    var ab = a.toString() + b.toString();
    var ba = b.toString() + a.toString();
    return ba - ab;
  });
  var result = nums.join("");
  if (parseInt(result) == 0) {
    return "0";
  } else {
    return result;
  }
};
```

# [242] Valid Anagram

Given two strings s and t , write a function to determine if t is an anagram of s.
Note: You may assume the string contains only lowercase alphabets.
难度：Easy (51.12%)

```javascript
var isAnagram = function(s, t) {
  if (s.length != = t.length) {
    return false;
  }
  var res = new Array(26);
  res.fill(0);
  for (var i = 0; i < s.length; i++) {
    res[s.codePointAt(i) - 97]++;
    res[t.codePointAt(i) - 97]--;
  }
  for (var i = 0; i < res.length; i++) {
    if (res[i] !== 0) {
      return false;
    }
  }
  return true;
};
```

# [274] H-Index

Given an array of citations (each citation is a non-negative integer) of a researcher, write a function to compute the researcher's h-index.
According to the definition of h-index on Wikipedia: "A scientist has index h if h of his/her N papers have at least h citations each, and the other N − h papers have no more than h citations each."
难度：Medium (34.43%)
思路：一个人在其所有学术文章中有 N 篇论文分别被引用了至少 N 次，他的 H 指数就是 N。根据这个规则，首先讲数组倒序排列，判断数组中的第 i 个，是否大于等于 i+1，如果成立，则代表，至少有 i+1 篇文章，被引用了 i+1 次，则他的 h-index 就是 i+1.

```javascript
var hIndex = function(citations) {
  if (citations.length == 0) {
    return 0;
  }
  citations.sort((a, b) => b - a);
  var res = citations.length;
  for (var i = citations.length - 1; i >= 0; i--) {
    if (citations[i] >= res) {
      return res;
    }
    res--;
  }
  if (res == 0) {
    return 0;
  }
};
```

# [324] Wiggle Sort II

Given an unsorted array nums, reorder it such that nums[0] < nums[1] > nums[2] < nums[3]....
难度：Medium (27.54%)
思路：将数组正序排序之后，从中间分为两个数组，每次从小数组里拿一个数，从大数组里拿一个数，组成的 新数组就是一个数大一个数小的状态。

```javascript
var wiggleSort = function(nums) {
  if (nums.length <= 1) {
    return nums;
  }
  nums = nums.sort((a, b) => a - b);
  var mid = Math.floor((nums.length + 1) / 2);
  var small = nums.slice(0, mid);
  var big = nums.slice(mid);
  if (big.length > small.length) {
    return [];
  }
  var i = 0;
  var j = small.length - 1;
  var k = big.length - 1;
  while (i < nums.length && j >= 0 && k >= 0) {
    nums[i] = small[j];
    nums[i + 1] = big[k];
    i = i + 2;
    j--;
    k--;
  }
  while (i < nums.length && j >= 0) {
    nums[i] = small[j];
    i++;
    j--;
  }
  return nums;
};
```
