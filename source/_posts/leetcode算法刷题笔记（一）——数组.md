---
title: leetcode算法刷题笔记（一）——数组
date: 2018-12-19 00:07:06
tags:
  - 算法
  - leetcode
  - 数组
  - javascript
categories: 算法
cover_img: https://i.screenshot.net/ognlzbx
feature_img: https://i.screenshot.net/ognlzbx
---

_该笔记只为个人所写算法，不一定是最优解法，仅供参考_

# [1] Two Sum

Given an array of integers, return indices of the two numbers such that they add up to a specific target.
难度：Easy (42.36%)
考点：哈希表
思路：用一遍循环 一边向哈希表中存值，一边比较判断

<!-- more -->

```javascript
var twoSum = function(nums, target) {
  var map = {};
  for (i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    if (complement in map) {
      return [i, map[complement]];
    }
    map[nums[i]] = i;
  }
  return -1;
};
```

# [11] Container With Most Water

Given n non-negative integers a1, a2, ..., an , where each represents a point at coordinate (i, ai). n vertical lines are drawn such that the two endpoints of line i is at (i, ai) and (i, 0). Find two lines, which together with x-axis forms a container, such that the container contains the most water.
Note: You may not slant the container and n is at least 2.
难度：Medium (42.93%)
考点：动态规划
思路：

1. 设定 i，j 分别指向数组的头和尾
2. 比较 i，j 所对应的位置的值，值较小的那一个移动（i++或 j--）

```javascript
var maxArea = function(height) {
  var maxArea = 0;
  var i = 0;
  var j = height.length - 1;
  while (i < j) {
    const long = Math.min(height[i], height[j]);
    const area = long * (j - i);
    if (area > maxArea) {
      maxArea = area;
    }
    if (height[i] < height[j]) {
      i++;
    } else {
      j--;
    }
  }
  return maxArea;
};
```

# [15] 3Sum

Given an array nums of n integers, are there elements a, b, c in nums such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.
Note: The solution set must not contain duplicate triplets.
难度：Medium (23.55%)
思路：

1. 数组排序（升序）
2. 设定三个指针，最外层循环从 0 开始，到数组的尾结束（i=0）
3. 第二层循环，一个指向上一个指针的下一个元素（j=i+1），另一个指向数组的尾部(k=nums.length-1)
4. 如果三个元素之和等于 0，则 push 进要返回的数组中；如果三个元素之和大于 0，说明第三个指针指向的元素过大，则第三个指针向前移(k--);如果三个元素之和小于 0，说明第二个指针指向的元素过小，则第二个指针向后移(j++);

```javascript
var threeSum = function(nums) {
  var rtn = [];
  if (nums.length < 3) {
    return rtn;
  }
  nums = nums.sort(function(a, b) {
    return a - b;
  });
  for (var i = 0; i < nums.length - 1; i++) {
    if (nums[i] > 0) {
      return rtn;
    }
    if (i > 0 && nums[i] == nums[i - 1]) {
      continue;
    }

    for (var j = i + 1, k = nums.length - 1; j < k; ) {
      if (nums[i] + nums[j] + nums[k] === 0) {
        rtn.push([nums[i], nums[j], nums[k]]);
        j++;
        k--;
        while (j < k && nums[j] == nums[j - 1]) {
          j++;
        }
        while (j < k && nums[k] == nums[k + 1]) {
          k--;
        }
      } else if (nums[i] + nums[j] + nums[k] > 0) {
        k--;
      } else {
        j++;
      }
    }
  }
  return rtn;
};
```

# [16] 3Sum Closest

Given an array nums of n integers and an integer target, find three integers in nums such that the sum is closest to target. Return the sum of the three integers. You may assume that each input would have exactly one solution.
难度：Medium (41.40%)
思路：与上题类似

```javascript
var threeSumClosest = function(nums, target) {
  var sum;
  if (nums.length < 3) {
    return sum;
  }
  nums = nums.sort(function(a, b) {
    return a - b;
  });
  var sum = nums[0] + nums[1] + nums[2];
  var distance = Math.abs(sum - target);
  for (var i = 0; i < nums.length - 2; i++) {
    for (var j = i + 1, k = nums.length - 1; j < k; ) {
      if (Math.abs(nums[i] + nums[j] + nums[k] - target) < distance) {
        sum = nums[i] + nums[j] + nums[k];
        distance = Math.abs(sum - target);
      }
      if (nums[i] + nums[j] + nums[k] === target) {
        break;
      }
      if (nums[i] + nums[j] + nums[k] > target) {
        k--;
      } else if (nums[i] + nums[j] + nums[k] < target) {
        j++;
      }
    }
  }
  return sum;
};
```

# [18] 4Sum

Given an array nums of n integers and an integer target, are there elements a, b, c, and d in nums such that a + b + c + d = target? Find all unique quadruplets in the array which gives the sum of target.
Note: The solution set must not contain duplicate quadruplets.
难度：Medium (29.83%)
思路：思路同 3Sum，多一层循环。注意跳过相同的数（最外两层的循环变量）

```javascript
var fourSum = function(nums, target) {
  var rtn = [];
  if (nums.length < 4) {
    return rtn;
  }
  nums = nums.sort(function(a, b) {
    return a - b;
  });
  for (var m = 0; m < nums.length - 3; m++) {
    var complement = target - nums[m];
    for (var i = m + 1; i < nums.length - 2; i++) {
      for (var j = i + 1, k = nums.length - 1; j < k; ) {
        if (nums[i] + nums[j] + nums[k] === complement) {
          rtn.push([nums[m], nums[i], nums[j], nums[k]]);
          j++;
          k--;
          while (j < k && nums[j] == nums[j - 1]) {
            j++;
          }
          while (j < k && nums[k] == nums[k + 1]) {
            k--;
          }
        } else if (nums[i] + nums[j] + nums[k] > complement) {
          k--;
        } else {
          j++;
        }
        if (i < nums.length - 1 && nums[i] == nums[i + 1]) {
          ++i;
        }
      }
      if (m < nums.length - 1 && nums[m] == nums[m + 1]) {
        ++m;
      }
    }
  }
  return rtn;
};
```

# [26] Remove Duplicates from Sorted Array

Given a sorted array nums, remove the duplicates in-place such that each element appear only once and return the new length.
Do not allocate extra space for another array, you must do this by modifying the input array in-place with O(1) extra memory.
难度：Easy (39.80%)
思路：两个指针，一个指针负责寻找和后一个不相等的数，另一个指针负责一步步向后移去重。

```javascript
var removeDuplicates = function(nums) {
  if (nums.length == 0) {
    return 0;
  }
  var i = 0;
  var j = 0;
  for (i = 0; i < nums.length - 1; ) {
    if (nums[i] === nums[i + 1]) {
      i++;
    } else {
      if (i !== j) {
        nums[j + 1] = nums[i + 1];
      }
      j++;
      i++;
    }
  }
  nums = nums.slice(0, j + 1);
  return nums.length;
};
```

# [27] Remove Element

Given an array nums and a value val, remove all instances of that value in-place and return the new length.
Do not allocate extra space for another array, you must do this by modifying the input array in-place with O(1) extra memory.
The order of elements can be changed. It doesn't matter what you leave beyond the new length.
难度：Easy (43.73%)
思路：找到和 val 值相等的位置，将数组最后一个元素赋值过来（去掉这个 val，数组长度减一）

```javascript
var removeElement = function(nums, val) {
  var i = 0;
  var n = nums.length;
  for (i = 0; i < n; ) {
    if (nums[i] == val) {
      nums[i] = nums[n - 1];
      n--;
    } else {
      i++;
    }
  }
  return n;
};
```

# [31] Next Permutation

Implement next permutation, which rearranges numbers into the lexicographically next greater permutation of numbers.
If such arrangement is not possible, it must rearrange it as the lowest possible order (ie, sorted in ascending order).
The replacement must be in-place and use only constant extra memory.
Here are some examples. Inputs are in the left-hand column and its corresponding outputs are in the right-hand column.
1,2,3 → 1,3,2
3,2,1 → 1,2,3
1,1,5 → 1,5,1
难度：Medium (30.09%)
思路：

1. 从后向前比较相邻的两个元素，直到前一个元素小于后一个元素，停止（i）。
2. 若已经没有了前一个元素（i=0），则该序列为递减序列，没有 Next Permutation。按照题目要求，直接反转序列。
3. 前一个元素（j=i-1）小于后一个元素（i），找到前一个元素（j）要交换的元素，从 i 的后一个元素开始往后查找，找到最后一个比“前一个元素（j）”大的元素（k），也就是再往后的元素，就比元素 j 小了。交换 j 和 k 元素。
4. 从 i 开始，包括 i 到序列的尾部，反转。
   则得出的即是 Next Permutation

```javascript
var nextPermutation = function(nums) {
  var i = nums.length - 1;
  while (nums[i] <= nums[i - 1]) {
    i--;
  }
  if (i !== 0) {
    var j = i - 1;
    var k = i + 1;
    while (nums[j] < nums[k]) {
      k++;
    }
    var temp = nums[k - 1];
    nums[k - 1] = nums[j];
    nums[j] = temp;
    for (var m = i, n = nums.length - 1; m < n; m++, n--) {
      var temp = nums[n];
      nums[n] = nums[m];
      nums[m] = temp;
    }
  } else {
    nums = nums.reverse();
  }
};
```

# [33] Search in Rotated Sorted Array

Suppose an array sorted in ascending order is rotated at some pivot unknown to you beforehand.(i.e., [0,1,2,4,5,6,7] might become [4,5,6,7,0,1,2]).
You are given a target value to search. If found in the array return its index, otherwise return -1.
You may assume no duplicate exists in the array.
Your algorithm's runtime complexity must be in the order of O(log n).
难度：Medium (32.68%)
考点：二分法
注意：判断和循环的边界条件

```javascript
var search = function(nums, target) {
  if (nums.length == 0) {
    return -1;
  }
  var start = 0;
  var end = nums.length - 1;
  while (start <= end) {
    var middle = parseInt((start + end) / 2);
    if (nums[middle] == target) {
      return middle;
    }
    if (nums[middle] > nums[end]) {
      if (target >= nums[start] && target < nums[middle]) {
        end = middle - 1;
      } else {
        start = middle + 1;
      }
    } else {
      if (target > nums[middle] && target <= nums[end]) {
        start = middle + 1;
      } else {
        end = middle - 1;
      }
    }
  }
  return -1;
};
```

# [34] Find First and Last Position of Element in Sorted Array

Given an array of integers nums sorted in ascending order, find the starting and ending position of a given target value.
Your algorithm's runtime complexity must be in the order of O(log n).
If the target is not found in the array, return [-1, -1].
难度：Medium (33.06%)
考点：二分法
注意：判断和循环的边界条件

```javascript
var searchRange = function(nums, target) {
  if (nums.length == 0) {
    return [-1, -1];
  }
  var startFind = 0;
  var endFind = nums.length - 1;
  var start;
  var end;
  while (startFind <= endFind) {
    var mid = parseInt((startFind + endFind) / 2);
    if (nums[mid] == target) {
      start = mid;
      end = mid;
      while (nums[start - 1] == target || nums[end + 1] == target) {
        if (nums[start - 1] == target) {
          start--;
        }
        if (nums[end + 1] == target) {
          end++;
        }
      }
      return [start, end];
    } else if (nums[mid] < target) {
      startFind = mid + 1;
    } else {
      endFind = mid - 1;
    }
  }
  return [-1, -1];
};
```

# [35] Search Insert Position

Given a sorted array and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order.
You may assume no duplicates in the array.
难度：Easy (40.50%)

```javascript
var searchInsert = function(nums, target) {
  if (nums.length == 0) {
    return 0;
  }
  let out = nums.indexOf(target);
  if (out == -1) {
    let i = 0;
    for (; target > nums[i]; i++) {}
    out = i;
  }
  return out;
};
```

# [39] Combination Sum

Given a set of candidate numbers (candidates) (without duplicates) and a target number (target), find all unique combinations in candidates where the candidate numbers sums to target.
The same repeated number may be chosen from candidates unlimited number of times.
Note:All numbers (including target) will be positive integers.The solution set must not contain duplicate combinations.
难度：Medium (46.97%)
考点：递归

```javascript
var combinationSum = function(candidates, target) {
  var rt = [];
  var solution = [];
  if (candidates.length == 0) {
    return rt;
  }
  candidates = candidates.sort(function(a, b) {
    return a - b;
  });
  sarch(0, target);

  function sarch(start, target) {
    if (start == candidates.length) {
      return;
    }
    if (target == 0) {
      return rt.push(solution.slice());
    }
    if (target < 0) {
      return;
    }
    solution.push(candidates[start]);
    sarch(start, target - candidates[start]);
    solution.pop();
    sarch(start + 1, target);
  }
  return rt;
};
```

# [40] Combination Sum II

Given a collection of candidate numbers (candidates) and a target number (target), find all unique combinations in candidates where the candidate numbers sums to target.
Each number in candidates may only be used once in the combination.
Note:All numbers (including target) will be positive integers.The solution set must not contain duplicate combinations.
难度：Medium (40.37%)
考点：递归
注意：限制边界条件，过滤重复的结果

```javascript
var combinationSum2 = function(candidates, target) {
  var rt = [];
  var solution = [];
  if (candidates.length == 0) {
    return rt;
  }
  candidates = candidates.sort(function(a, b) {
    return a - b;
  });
  search(0, target);
  return rt;

  function search(start, target) {
    if (target === 0 && start === candidates.length) {
      return rt.push(solution.slice());
    }
    if (target < 0 || start === candidates.length) {
      return;
    }

    solution.push(candidates[start]);
    search(start + 1, target - candidates[start]);
    solution.pop();
    if (solution[solution.length - 1] !== candidates[start]) {
      search(start + 1, target);
    }
  }
};
```
