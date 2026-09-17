# Array DSA — Optimized Programs

> Reference notes from my DSA practice.  
> Language: JavaScript  
> Focus: clean interview-style solutions, time/space optimization, and reusable patterns.

---

## 1. Find Largest Element

```js
function largestNumber(arr) {
  let greatest = -Infinity;

  for (let value of arr) {
    if (value > greatest) {
      greatest = value;
    }
  }

  return greatest;
}
```

**Complexity:** O(n) time, O(1) space.

---

## 2. Find Smallest Element

```js
function smallestNumber(arr) {
  let smallest = Infinity;

  for (let value of arr) {
    if (value < smallest) {
      smallest = value;
    }
  }

  return smallest;
}
```

**Complexity:** O(n) time, O(1) space.

---

## 3. Sum of Array

```js
function sumOf(arr) {
  let sum = 0;

  for (let value of arr) {
    sum += value;
  }

  return sum;
}
```

**Complexity:** O(n) time, O(1) space.

---

## 4. Count Even Numbers

```js
function evenCounter(arr) {
  let counter = 0;

  for (let value of arr) {
    if (value % 2 === 0) {
      counter++;
    }
  }

  return counter;
}
```

**Complexity:** O(n) time, O(1) space.

---

## 5. Reverse Array In-Place

```js
function reverseArr(arr) {
  let a = 0;
  let b = arr.length - 1;

  while (a < b) {
    let temp = arr[a];
    arr[a] = arr[b];
    arr[b] = temp;

    a++;
    b--;
  }

  return arr;
}
```

**Complexity:** O(n) time, O(1) space.

**Pattern:** Two pointers.

---

## 6. Find First Index of Target

```js
function indexArr(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) {
      return i;
    }
  }

  return -1;
}
```

**Complexity:** O(n) time, O(1) space.

**Important:** `return` inside a normal `for` loop immediately exits the function.

---

## 7. Count Target Occurrences

```js
function countOccurrences(arr, target) {
  let counter = 0;

  for (let value of arr) {
    if (value === target) {
      counter++;
    }
  }

  return counter;
}
```

**Complexity:** O(n) time, O(1) space.

---

## 8. Find Second Largest Distinct Element

```js
function secondGreatest(arr) {
  let greatest = -Infinity;
  let second = -Infinity;

  for (let value of arr) {
    if (value > greatest) {
      second = greatest;
      greatest = value;
    } else if (value > second && value < greatest) {
      second = value;
    }
  }

  return second;
}
```

**Complexity:** O(n) time, O(1) space.

**Important JavaScript rule:** Do not write chained comparisons such as `second < value < greatest`. JavaScript does not interpret that as a mathematical chained comparison.

**Assumption:** The array contains at least two distinct values.

---

## 9. Remove Duplicates from Unsorted Array

```js
function removeDuplicate(arr) {
  let unique = [];

  for (let value of arr) {
    if (!unique.includes(value)) {
      unique.push(value);
    }
  }

  return unique;
}
```

**Complexity:** O(n²) time, O(n) space.

**Why:** `includes()` is O(n) in the worst case, and `unique` can contain O(n) elements.

**Interview note:** This is the optimized version under the practice constraint of not using `Set`, but it is not O(1) space.

---

## 10. Find Missing Number from 0 to n

Given an array containing `n` distinct numbers from `0` through `n`, find the missing number.

```js
function missingNum(arr) {
  let n = arr.length;
  let sum = (n * (n + 1)) / 2;
  let arrSum = 0;

  for (let value of arr) {
    arrSum += value;
  }

  return sum - arrSum;
}
```

**Complexity:** O(n) time, O(1) space.

**Formula:**

```text
1 + 2 + ... + n = n(n + 1) / 2
```

---

## 11. Move Zeros to the End — Swap Version

```js
function moveZero(arr) {
  let pointer = 0;

  for (let i = 0; i < arr.length; i++) {
    if (arr[i] !== 0) {
      if (i !== pointer) {
        let temp = arr[i];
        arr[i] = arr[pointer];
        arr[pointer] = temp;
      }

      pointer++;
    }
  }

  return arr;
}
```

**Complexity:** O(n) time, O(1) space.

**Pattern:** Two pointers / partitioning.

---

## 12. Maximum Subarray Sum — Kadane's Algorithm

```js
function maxSubArray(arr) {
  let sum = arr[0];
  let maxSum = arr[0];

  for (let i = 1; i < arr.length; i++) {
    if (arr[i] > sum + arr[i]) {
      sum = arr[i];
    } else {
      sum = sum + arr[i];
    }

    if (sum > maxSum) {
      maxSum = sum;
    }
  }

  return maxSum;
}
```

**Complexity:** O(n) time, O(1) space.

**Core recurrence:**

```text
current = max(arr[i], current + arr[i])
```

Without `Math.max()`:

```js
if (arr[i] > sum + arr[i]) {
  sum = arr[i];
} else {
  sum += arr[i];
}
```

---

## 13. Two Sum — Brute Force

Return the indices of two numbers whose sum equals `target`.

```js
function twoSum(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] + arr[j] === target) {
        return [i, j];
      }
    }
  }

  return [];
}
```

**Complexity:** O(n²) time, O(1) extra space.

**Pattern:** Nested loops with `j = i + 1` to avoid reusing the same element.

---

## 14. Check if Array is Sorted Ascending

```js
function isSorted(arr) {
  for (let i = 0; i < arr.length - 1; i++) {
    if (arr[i] > arr[i + 1]) {
      return false;
    }
  }

  return true;
}
```

**Complexity:** O(n) time, O(1) space.

---

## 15. First Non-Repeating Element

```js
function firstNonRepeat(arr) {
  for (let i = 0; i < arr.length; i++) {
    let occurred = false;

    for (let j = 0; j < arr.length; j++) {
      if (i !== j && arr[i] === arr[j]) {
        occurred = true;
        break;
      }
    }

    if (!occurred) {
      return arr[i];
    }
  }

  return -1;
}
```

**Complexity:** O(n²) time, O(1) space.

**Pattern:** For each element, search the entire array for another occurrence.

---

## 16. Move Negative Numbers to the Left

Order does not need to be preserved.

```js
function moveNegative(arr) {
  let pointer = 0;

  for (let i = 0; i < arr.length; i++) {
    if (arr[i] < 0) {
      if (i !== pointer) {
        let temp = arr[pointer];
        arr[pointer] = arr[i];
        arr[i] = temp;
      }

      pointer++;
    }
  }

  return arr;
}
```

**Complexity:** O(n) time, O(1) space.

**Pattern:** One-pointer partition.

---

## 17. Maximum Difference `arr[j] - arr[i]`, where `j > i`

### Optimized O(n) solution

```js
function maxDiff(arr) {
  let diff = -Infinity;
  let min = Infinity;

  for (let i = 0; i < arr.length; i++) {
    if (arr[i] - min > diff) {
      diff = arr[i] - min;
    }

    if (arr[i] < min) {
      min = arr[i];
    }
  }

  return diff;
}
```

**Complexity:** O(n) time, O(1) space.

**Key idea:** Keep the smallest value seen **before** the current position.

**Important:** `diff` may legitimately be negative.

For:

```js
[9, 7, 5, 3]
```

the answer is `-2`.

---

## 18. Best Time to Buy and Sell Stock

Buy once and sell once. Buy must happen before selling. Return `0` if no profit is possible.

```js
function buySellStock(arr) {
  let minimum = Infinity;
  let profit = 0;

  for (let value of arr) {
    if (value < minimum) {
      minimum = value;
    }

    if (value - minimum > profit) {
      profit = value - minimum;
    }
  }

  return profit;
}
```

**Complexity:** O(n) time, O(1) space.

**Pattern:** Minimum-so-far + current value.

**Difference from Problem 17:** Stock profit cannot be negative, so `profit` starts at `0`.

---

## 19. Remove Duplicates from Sorted Array

Modify the original array and return the number of unique elements.

```js
function removeDuplicate(arr) {
  let pointer = 0;

  for (let i = 1; i < arr.length; i++) {
    if (arr[i] !== arr[pointer]) {
      pointer++;
      arr[pointer] = arr[i];
    }
  }

  return pointer + 1;
}
```

**Complexity:** O(n) time, O(1) space.

**Pattern:** Two pointers.

Only the first `pointer + 1` elements matter after the operation.

Example:

```js
[1, 1, 2, 2, 3, 4, 4]
```

becomes effectively:

```js
[1, 2, 3, 4, ...]
```

and returns `4`.

---

## 20. Move Zeros to the End — No Swapping

Preserve the order of non-zero elements.

```js
function moveZero(arr) {
  let pointer = 0;

  for (let i = 0; i < arr.length; i++) {
    if (arr[i] !== 0) {
      if (i !== pointer) {
        arr[pointer] = arr[i];
        arr[i] = 0;
      }

      pointer++;
    }
  }

  return arr;
}
```

**Complexity:** O(n) time, O(1) space.

**Pattern:** Two pointers / overwrite and fill.

**Important:** Use `arr[i] !== 0` rather than `arr[i] > 0` if negative numbers are allowed.

---

## 21. Rotate Array Right by One

```js
function rotateArray(arr) {
  let temp = arr[0];

  arr[0] = arr[arr.length - 1];

  for (let i = 1; i < arr.length; i++) {
    let temp1 = arr[i];
    arr[i] = temp;
    temp = temp1;
  }

  return arr;
}
```

**Complexity:** O(n) time, O(1) space.

Example:

```js
[1, 2, 3, 4, 5]
```

becomes:

```js
[5, 1, 2, 3, 4]
```

---

## 22. Rotate Array Right by K — Reversal Algorithm

This is the optimized **O(n) time, O(1) space** solution.

```js
function rotateByK(arr, k) {
  if (arr.length === 0) {
    return arr;
  }

  k = k % arr.length;

  function reverse(a, b) {
    while (a < b) {
      let temp = arr[a];
      arr[a] = arr[b];
      arr[b] = temp;

      a++;
      b--;
    }
  }

  reverse(0, arr.length - 1);
  reverse(0, k - 1);
  reverse(k, arr.length - 1);

  return arr;
}
```

### The formula

For a right rotation by `k`:

```text
1. Reverse the entire array
2. Reverse the first k elements
3. Reverse the remaining n-k elements
```

Example:

```text
[1, 2, 3, 4, 5], k = 2

Reverse all:
[5, 4, 3, 2, 1]

Reverse first k:
[4, 5, 3, 2, 1]

Reverse remaining:
[4, 5, 1, 2, 3]
```

**Complexity:** O(n) time, O(1) space.

**Important:** Normalize large `k` values:

```js
k = k % arr.length;
```

because rotating an array of length `5` by `7` is equivalent to rotating it by `2`.

---

# Core Patterns Learned

## 1. Two Pointers

Used in:

- Reverse array
- Move zeros
- Move negatives
- Remove duplicates from sorted array
- Rotate by K
- Palindrome checking

General idea:

```js
let left = 0;
let right = arr.length - 1;

while (left < right) {
  // work with both ends
  left++;
  right--;
}
```

---

## 2. One-Pointer Partition

Used in:

- Move negative numbers
- Move zeros

General idea:

```js
let pointer = 0;

for (let i = 0; i < arr.length; i++) {
  if (condition) {
    // place current element at pointer
    pointer++;
  }
}
```

`pointer` represents the next position where a qualifying element should go.

---

## 3. Minimum/Maximum So Far

Used in:

- Maximum difference
- Best time to buy and sell stock

General idea:

```js
let minimum = Infinity;

for (let value of arr) {
  if (value < minimum) {
    minimum = value;
  }

  // use minimum with current value
}
```

This often turns an O(n²) pair-comparison solution into O(n).

---

## 4. Kadane's Algorithm

Used for:

- Maximum subarray sum

Core idea:

```text
At every element:

Should I:
1. Start a new subarray here?
or
2. Continue the previous subarray?
```

That gives:

```js
if (arr[i] > currentSum + arr[i]) {
  currentSum = arr[i];
} else {
  currentSum += arr[i];
}
```

---

## 5. In-Place Array Modification

The main goal is to modify the original array without creating another array.

Common tools:

```js
// swap
let temp = arr[a];
arr[a] = arr[b];
arr[b] = temp;
```

or:

```js
// overwrite
arr[pointer] = arr[i];
```

This allows O(1) auxiliary space.

---

# Interview Lessons

### Correctness comes before optimization

A useful progression is:

```text
Brute force
    ↓
Correct solution
    ↓
Analyze time/space complexity
    ↓
Identify bottleneck
    ↓
Optimize
```

For example, Problem 17 started as:

```text
O(n²) pair comparison
```

and was optimized to:

```text
O(n) using minimum-so-far
```

---

### Watch the problem's constraints

A solution can produce the correct output but still be unacceptable if it violates the required complexity.

For example:

```js
let tempArray = new Array(arr.length);
```

works for many array transformations, but creates **O(n) extra space**.

If the interviewer specifically asks for O(1) space, you need an in-place approach.

---

### Negative answers are sometimes valid

For maximum difference:

```js
[9, 7, 5, 3]
```

answer:

```text
-2
```

So:

```js
let diff = -Infinity;
```

is appropriate.

For stock profit:

```js
[9, 7, 5, 3]
```

answer:

```text
0
```

because you simply don't make a transaction.

So:

```js
let profit = 0;
```

is appropriate.

---

# Complexity Quick Reference

| Problem | Time | Extra Space | Main Pattern |
|---|---:|---:|---|
| Largest element | O(n) | O(1) | Traversal |
| Smallest element | O(n) | O(1) | Traversal |
| Sum | O(n) | O(1) | Traversal |
| Count even | O(n) | O(1) | Traversal |
| Reverse array | O(n) | O(1) | Two pointers |
| Find index | O(n) | O(1) | Linear search |
| Count occurrences | O(n) | O(1) | Traversal |
| Second largest | O(n) | O(1) | Two variables |
| Remove duplicates, unsorted | O(n²) | O(n) | `includes()` |
| Missing number | O(n) | O(1) | Math formula |
| Move zeros | O(n) | O(1) | Two pointers |
| Maximum subarray | O(n) | O(1) | Kadane |
| Two Sum | O(n²) | O(1) | Nested loops |
| Sorted check | O(n) | O(1) | Adjacent comparison |
| First non-repeating | O(n²) | O(1) | Nested loops |
| Move negatives | O(n) | O(1) | Partition |
| Maximum difference | O(n) | O(1) | Minimum-so-far |
| Stock profit | O(n) | O(1) | Minimum-so-far |
| Remove duplicates, sorted | O(n) | O(1) | Two pointers |
| Move zeros, no swap | O(n) | O(1) | Overwrite |
| Rotate right by one | O(n) | O(1) | Carry/shift |
| Rotate right by k | O(n) | O(1) | Reversal |
