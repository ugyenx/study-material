# Data Structures & Algorithms

## 1. Problem-Solving Framework

The most important skill developed so far is not memorizing algorithms. It is learning how to move from an unfamiliar problem to a solution.

### Step 1 — Understand the Problem

Ask:

- What exactly must be found?
- Is the result a value, index, boolean, array, or length?
- Are there restrictions such as "in-place", "sorted", "distinct", or "contiguous"?

### Step 2 — Identify Inputs and Outputs

Clearly identify:

```text
Input → What information do I receive?
Output → What exactly must I return?
```

### Step 3 — Check Constraints and Edge Cases

Think about:

```text
[]
[5]
all equal
all negative
duplicates
no duplicates
already sorted
reverse sorted
```

Also pay attention to problem-specific constraints.

For example:

- A sorted array enables Binary Search.
- Positive numbers enable certain Sliding Window techniques.
- A lowercase-English constraint enables a fixed 26-element frequency array.

### Step 4 — Think of Brute Force First

Start with the most obvious correct solution.

Do not optimize prematurely.

The brute-force solution gives you something correct to improve.

### Step 5 — Identify the Bottleneck

Ask:

> What work am I repeating?

Examples:

```text
Two Sum
→ repeatedly searching the array

Product Except Self
→ repeatedly calculating left/right products

Range Sum
→ repeatedly calculating the same sums
```

### Step 6 — Look for a Pattern

Common patterns learned so far:

```text
Need best value while scanning
→ Running State

Need to remember previous values
→ Hashing

Sorted array / eliminate half
→ Binary Search

Two positions moving through an array
→ Two Pointers

Contiguous range
→ Sliding Window / Prefix Sum

Last item must be handled first
→ Stack
```

### Step 7 — Define the Invariant

An invariant is the property that remains true during every iteration.

Examples:

```text
Find Maximum
→ max is the largest value seen so far

Move Zeroes
→ everything before pointer A is already correct

Binary Search
→ if the target exists, it is inside [left, right]

Sliding Window
→ sum represents exactly the current window
```

### Step 8 — Dry Run

Before coding, manually execute the algorithm.

Track:

```text
pointer
index
current value
state
answer
```

### Step 9 — Analyze Complexity

Always state:

```text
Time Complexity
Space Complexity
```

And explain why.

### Step 10 — Implement

Only after the algorithm is understood should the code be written.

---

# 2. Big-O Complexity

## What Big-O Represents

Big-O describes how the amount of work grows as the input size `n` grows.

It is primarily concerned with **growth rate**, not exact operation counts.

### Common Complexities

| Complexity | General idea |
|---|---|
| `O(1)` | Constant work |
| `O(log n)` | Work decreases by a factor such as 2 each step |
| `O(n)` | One pass through the input |
| `O(n log n)` | Linear work combined with logarithmic work |
| `O(n²)` | Two nested linear-sized operations |
| `O(2ⁿ)` | Exponential growth |

## Dominant Term

For:

```text
n² + n
```

as `n` becomes very large, `n²` dominates.

Therefore:

```text
O(n²)
```

Similarly:

```text
n² - n → O(n²)
```

### Nested Loops

If:

```javascript
for (let i = 0; i < n; i++) {
    for (let j = 0; j < n; j++) {
        // work
    }
}
```

the inner loop runs `n` times for every outer iteration.

Therefore:

```text
n × n = n²
```

→ `O(n²)`.

For:

```javascript
for (let i = 0; i < n; i++) {
    for (let j = i; j < n; j++) {
        // work
    }
}
```

the work is approximately:

```text
n + (n - 1) + (n - 2) + ... + 1
```

which is still:

```text
O(n²)
```

### Logarithmic Growth

For:

```javascript
i *= 2
```

the values might be:

```text
1
2
4
8
16
32
...
```

The number of iterations grows logarithmically:

```text
O(log n)
```

---

# 3. Recursion

## Basic Idea

A recursive function calls itself on a smaller version of the problem.

Every recursive solution should have:

### Base Case

The condition that stops recursion.

### Recursive Case

The smaller problem used to make progress toward the base case.

---

## Example: Sum

```javascript
function sum(n) {
    if (n === 0) {
        return 0;
    }

    return n + sum(n - 1);
}
```

Execution:

```text
sum(3)
 ↓
sum(2)
 ↓
sum(1)
 ↓
sum(0)
```

Then the calls return:

```text
sum(0) = 0
sum(1) = 1 + 0
sum(2) = 2 + 1
sum(3) = 3 + 3
```

Result:

```text
6
```

### Important Principle

A function cannot finish the outer calculation until the recursive call returns.

---

# 4. Call Stack

Every function call creates a stack frame containing information needed for that call.

Conceptually:

```text
sum(3)
sum(2)
sum(1)
sum(0)
```

The most recent call is completed first.

When recursion has no valid base case:

```javascript
function forever(n) {
    return forever(n + 1);
}
```

new stack frames keep accumulating until the call stack is exhausted.

This causes a **stack overflow**.

---

# 5. Stack vs Heap and References

A simplified mental model:

```text
Stack
    function calls
    local variables / references

Heap
    objects
    arrays
```

For:

```javascript
let a = { x: 10 };
let b = a;
```

conceptually:

```text
Stack

a ─────┐
       │
b ─────┘
        ↓

Heap

{x: 10}
```

`a` and `b` refer to the same object.

### Mutation vs Reassignment

Mutation:

```javascript
b.x = 20;
```

changes the shared object.

Reassignment:

```javascript
b = { x: 30 };
```

makes `b` refer to a different object.

Then:

```text
a → {x:20}
b → {x:30}
```

### JavaScript Pass-by-Value

JavaScript passes arguments by value.

For primitives:

```javascript
let a = 10;
change(a);
```

the parameter receives a copy of `10`.

For objects, the copied value is a reference to the object.

Therefore:

```javascript
function change(obj) {
    obj.x = 100;
}
```

can mutate the object visible through the original variable.

---

# 6. Arrays

Arrays are frequently the starting point for DSA problems.

Important operations include:

```text
traversal
searching
updating
in-place modification
```

The major array patterns learned so far are:

```text
Running State
Hashing
Two Pointers
Binary Search
Sliding Window
Prefix Sum
```

---

# 7. Running State

## Core Idea

Instead of remembering every previous result, maintain only the information needed to continue.

### Find Maximum

```javascript
function findMax(arr) {
    let max = arr[0];

    for (let i = 1; i < arr.length; i++) {
        if (arr[i] > max) {
            max = arr[i];
        }
    }

    return max;
}
```

Invariant:

> `max` is the largest element seen so far.

Complexity:

```text
Time:  O(n)
Space: O(1)
```

---

# 8. Second Largest Distinct Element

Maintain two pieces of state:

```text
max
second
```

When a new maximum is found:

```text
second = max
max = current
```

Otherwise, if the value is smaller than `max` but larger than `second`:

```text
second = current
```

The important distinction is **distinct**.

For:

```text
[10, 10, 7, 5]
```

the second largest is:

```text
7
```

not `10`.

Complexity:

```text
Time:  O(n)
Space: O(1)
```

### Common Mistake

Initializing:

```javascript
let second = 0;
```

can fail for arrays containing only negative numbers.

A more robust approach uses an appropriate sentinel such as `-Infinity` or explicitly tracks whether a second distinct value exists.

---

# 9. Hashing

## Hash Map

A `Map` stores key-value relationships.

Example:

```text
number → index
```

## Hash Set

A `Set` stores unique values and is useful for answering:

> Have I seen this value before?

---

# 10. Two Sum

## Problem

Given:

```text
nums
target
```

find two different elements whose sum equals the target.

## Brute Force

Check every possible pair.

Complexity:

```text
O(n²)
```

## Key Insight

For a current value:

```text
current
```

the required partner is:

```text
complement = target - current
```

Instead of repeatedly searching the array, remember values already seen.

Use:

```javascript
Map
```

because we need:

```text
value → index
```

### Pattern

```text
current
   ↓
calculate complement
   ↓
does Map contain complement?
   ↓
yes → return its index + current index
no  → store current
```

Complexity:

```text
Time:  O(n)
Space: O(n)
```

---

# 11. Contains Duplicate

## Brute Force

For every element, search for the same value elsewhere.

```text
O(n²)
```

## Optimized

Use a `Set`.

```javascript
function containsDuplicate(arr) {
    const seen = new Set();

    for (const value of arr) {
        if (seen.has(value)) {
            return true;
        }

        seen.add(value);
    }

    return false;
}
```

Complexity:

```text
Time:  O(n)
Space: O(n)
```

### Pattern

> Store what you have already seen so you don't search for it again.

---

# 12. Frequency Counting

Frequency counting means maintaining:

```text
value → count
```

A `Map` can do this for general values.

For lowercase English characters, a fixed array of 26 counters is possible:

```text
a → 0
b → 1
c → 2
...
z → 25
```

Character index:

```javascript
character.charCodeAt(0) - 'a'.charCodeAt(0)
```

---

# 13. Valid Anagram

Two strings are anagrams when they contain the same characters with the same frequencies.

## Map Approach

Maintain character counts.

## Fixed Frequency Array

For lowercase English letters:

```javascript
const frequency = new Array(26).fill(0);
```

For the first string:

```text
frequency[characterIndex]++
```

For the second:

```text
frequency[characterIndex]--
```

At the end:

```text
every counter === 0
```

means the strings are anagrams.

### Example

```text
"cat"
"atc"
```

All counts cancel.

Therefore:

```text
true
```

Complexity:

```text
Time:  O(n)
Space: O(1)
```

The space is constant because the array always has 26 entries.

---

# 14. Two Pointers

## Core Idea

Use two positions to traverse the data efficiently.

The meaning of each pointer depends on the problem.

This is critical:

> Do not think of pointers merely as indexes. Think about what property each pointer represents.

---

# 15. Move Zeroes

Problem:

```text
[0,1,0,3,12]
```

should become:

```text
[1,3,12,0,0]
```

in-place.

### Two-Pointer Meaning

```text
A → next position where a non-zero belongs

B → scanning pointer
```

If `B` sees zero:

```text
B++
```

If `B` sees a non-zero:

```text
swap(A, B)
A++
B++
```

Complexity:

```text
Time:  O(n)
Space: O(1)
```

### Important Insight

Instead of moving every zero, move every non-zero into its correct position.

---

# 16. Remove Duplicates from Sorted Array

Given:

```text
[1,1,2,2,3]
```

the first part should become:

```text
[1,2,3,...]
```

and return:

```text
3
```

### Pointer Meaning

```text
A → last position containing a unique value

B → scanning pointer
```

If:

```text
arr[A] === arr[B]
```

the value is a duplicate, so move `B`.

If a new value is found:

```javascript
A++;
arr[A] = arr[B];
B++;
```

Because the array is sorted, once equality fails, the new value is necessarily larger.

Complexity:

```text
Time:  O(n)
Space: O(1)
```

---

# 17. Binary Search

## Core Idea

Binary Search repeatedly eliminates half of the search space.

It requires a condition that makes the discarded half impossible, typically a **sorted array**.

### Search Boundaries

```text
left  → left boundary of current search space
right → right boundary of current search space
mid   → position used to decide which half to discard
```

Invariant:

> If the target exists, it must be between `left` and `right`.

### Algorithm

```javascript
function binarySearch(arr, target) {
    let left = 0;
    let right = arr.length - 1;

    while (left <= right) {
        const mid = Math.floor((left + right) / 2);

        if (arr[mid] === target) {
            return mid;
        }

        if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return -1;
}
```

Complexity:

```text
Time:  O(log n)
Space: O(1)
```

### Why `O(log n)`?

Because the search space repeatedly becomes approximately:

```text
n
n/2
n/4
n/8
...
1
```

### Common Mistake

Binary Search on an unsorted array is not valid.

---

# 18. Stack

## Core Idea

A Stack is:

> **LIFO — Last In, First Out**

Like a pile of plates.

Main operations:

```text
push
pop
```

A stack is useful whenever the **most recently added item must be handled first**.

---

# 19. Valid Parentheses

Given:

```text
()[]{}
```

return:

```text
true
```

The most recent opening bracket must be matched first.

That gives:

```text
LIFO
↓
Stack
```

### Algorithm

Opening bracket:

```text
push
```

Closing bracket:

```text
check top
```

If the top matches:

```text
pop
```

If there is no opening bracket or the top doesn't match:

```text
false
```

At the end:

```text
stack.length === 0
```

must be true.

### Complexity

```text
Time:  O(n)
Space: O(n)
```

---

# 20. Stock — One Transaction

Given prices, buy once and sell once.

Key idea:

> Maintain the cheapest price seen so far.

State:

```text
minimumPrice
maximumProfit
```

For each current price:

```text
minimumPrice = smallest price seen so far
profit = currentPrice - minimumPrice
```

Update the maximum profit.

### Critical Insight

`minimumPrice` does **not** necessarily mean:

> "The stock has already been bought."

It means:

> "This is the cheapest candidate buying price seen so far."

A later lower price can replace the previous candidate.

Complexity:

```text
Time:  O(n)
Space: O(1)
```

---

# 21. Stock — Unlimited Transactions

When multiple transactions are allowed, we can capture every positive daily increase.

For:

```text
[1,2,3,4,5]
```

profit can be viewed as:

```text
(2-1) + (3-2) + (4-3) + (5-4)
```

which equals:

```text
5 - 1 = 4
```

Algorithm:

```javascript
if (prices[i] > prices[i - 1]) {
    profit += prices[i] - prices[i - 1];
}
```

Complexity:

```text
Time:  O(n)
Space: O(1)
```

### Lesson

A change in problem constraints can completely change the algorithm.

---

# 22. Sliding Window

## Core Idea

A Sliding Window is used when the problem concerns a **contiguous section** of an array or string.

Instead of recalculating the entire range, maintain information about the current window.

Two major forms have been learned.

---

# 23. Fixed-Size Sliding Window

Problem:

> Find the maximum sum of exactly `k` consecutive elements.

Example:

```text
[2,1,5,1,3,2]
k = 3
```

Windows:

```text
[2,1,5] → 8
[1,5,1] → 7
[5,1,3] → 9
[1,3,2] → 6
```

Answer:

```text
9
```

### Key Insight

When the window moves:

```text
old sum
- outgoing element
+ incoming element
```

So we don't recalculate the entire window.

### Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 24. Variable-Size Sliding Window

Problem:

> Find the smallest contiguous subarray whose sum is at least `target`.

Important constraint:

> The array contains positive integers.

That matters because removing a positive element decreases the sum.

### Pattern

```text
sum < target
→ expand right

sum >= target
→ record answer
→ shrink left

sum < target
→ expand again
```

Example:

```text
target = 7

[2,3,1,2] = 8
```

Since the window is valid, shrink:

```text
[3,1,2] = 6
```

Now invalid.

Expand again:

```text
[3,1,2,4] = 10
```

Shrink:

```text
[1,2,4] = 7
```

Shrink again:

```text
[2,4] = 6
```

Then later:

```text
[4,3] = 7
```

Minimum length:

```text
2
```

### Complexity

```text
Time: O(n)
Space: O(1)
```

### Why Isn't It `O(n²)`?

Even though there are nested-looking operations:

- `left` only moves forward.
- `right` only moves forward.

Every element enters the window at most once and leaves at most once.

Therefore total pointer movement is linear.

---

# 25. Prefix Sum

## Core Idea

Precompute cumulative information so range queries can be answered quickly.

For:

```text
arr = [2,4,1,5,3,6]
```

use:

```text
prefix = [0,2,6,7,12,15,21]
```

The extra `0` is useful.

### Meaning

```text
prefix[i]
```

represents the sum of the first `i` elements.

Therefore a range:

```text
left ... right
```

has sum:

```text
prefix[right + 1] - prefix[left]
```

### Example

```text
arr = [2,4,1,5,3,6]

left = 2
right = 5
```

Range:

```text
[1,5,3,6]
```

Sum:

```text
prefix[6] - prefix[2]
= 21 - 6
= 15
```

### Complexity

Preprocessing:

```text
O(n)
```

Each query:

```text
O(1)
```

For `q` queries:

```text
Total = O(n + q)
```

This improves on brute force:

```text
O(qn)
```

when many queries are present.

---

# 26. Product of Array Except Self

## Problem

For each position, return the product of all elements except itself.

Example:

```text
[1,2,3,4]
```

Result:

```text
[24,12,8,6]
```

## Brute Force

For each index:

```text
product of left side
×
product of right side
```

This repeatedly recalculates products.

Complexity:

```text
O(n²)
```

## Optimization

Calculate left products in one pass.

Then calculate right products in a second pass.

### First Pass

```text
nums:          [2,3,4,5]

left products: [1,2,6,24]
```

Meaning:

```text
index 0 → nothing left → 1
index 1 → 2
index 2 → 2×3 = 6
index 3 → 2×3×4 = 24
```

### Second Pass

Calculate right products from right to left:

```text
right products:
[60,20,5,1]
```

Then multiply:

```text
left × right
```

Result:

```text
[60,40,30,24]
```

### Complexity

```text
Time:  O(n)
Space: O(1)
```

excluding the required output array.

### Important Pattern

> Compute information from the left and right separately, then combine it.

---

# 27. Current Problem — Maximum Subarray

## Goal

Find the contiguous subarray with the largest sum.

Example:

```text
[-2,1,-3,4,-1,2,1,-5,4]
```

The best subarray is:

```text
[4,-1,2,1]
```

with sum:

```text
6
```

## Brute Force

Try every possible starting index and extend the subarray to the right.

Complexity:

```text
O(n²)
```

## Key Observation

If the current sum becomes negative, carrying that negative sum into a future subarray can only make the future sum smaller.

For example:

```text
currentSum = -2
next = 1
```

Choices:

```text
continue → -2 + 1 = -1
restart  → 1
```

Choose:

```text
1
```

This leads to **Kadane's Algorithm**.

### State

```text
currentSum
maxSum
```

`currentSum` means:

> Best sum of a subarray ending at the current position.

`maxSum` means:

> Best sum found anywhere so far.

### Current Learning Status

The key idea has been discovered, but the implementation has **not yet been completed** in the learning sequence.

---

# 28. Pattern Recognition Cheat Sheet

| Problem clue | Think about |
|---|---|
| Need best value while scanning | Running State |
| Frequency / duplicates | `Map` / `Set` |
| Need value → index | `Map` |
| Only need existence | `Set` |
| Sorted array | Binary Search / Two Pointers |
| Repeatedly halve search space | Binary Search |
| Contiguous subarray / substring | Sliding Window |
| Fixed number of elements | Fixed Sliding Window |
| Variable contiguous range | Variable Sliding Window |
| Many range-sum queries | Prefix Sum |
| Need left/right accumulated information | Prefix/Suffix |
| Most recent item must be handled first | Stack |
| Move elements while preserving order | Two Pointers |
| Need to remember previous values | Hashing |

---

# 29. Interview Habits Developed So Far

When solving an interview problem:

> **Don't start with code.**

Start with:

```text
What exactly is being asked?
```

Then:

```text
What are the inputs?
What is the output?
What are the constraints?
What are the edge cases?
```

Then:

```text
What is the brute-force solution?
```

Then ask:

> **What work am I repeating?**

Then look for the appropriate pattern.

Finally:

```text
Explain invariant
→ dry run
→ code
→ complexity
→ test
```

---

# 30. Progress So Far

## Completed

- Big-O
- Worst / best-case reasoning
- Nested-loop analysis
- Recursion
- Base cases
- Recursive cases
- Call stack
- Stack overflow
- Stack vs heap mental model
- References
- Mutation vs reassignment
- Pass-by-value
- Running state
- Maximum element
- Second largest distinct element
- Hash Map
- Hash Set
- Two Sum
- Contains Duplicate
- Frequency counting
- Valid Anagram
- Two Pointers
- Move Zeroes
- Remove Duplicates
- Binary Search
- Stack
- Valid Parentheses
- Stock I
- Stock II
- Fixed Sliding Window
- Variable Sliding Window
- Prefix Sum
- Product Except Self

## Currently Learning

- Kadane's Algorithm / Maximum Subarray

## Next Major Patterns

```text
Kadane's Algorithm
      ↓
More Sliding Window
      ↓
Linked Lists
      ↓
Queues
      ↓
Sorting
      ↓
Trees
      ↓
BST
      ↓
Heaps
      ↓
Graphs
      ↓
Backtracking
      ↓
Greedy
      ↓
Dynamic Programming
      ↓
Tries / Bit Manipulation / Advanced Patterns
```
