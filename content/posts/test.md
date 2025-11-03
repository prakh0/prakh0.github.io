+++ 
draft = false
date = 2025-04-04T22:52:31+05:30
title = "Binary Search: More Than Just Searching"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

###  Binary Search: More Than Just Searching


##  Introduction

When most people think of **binary search**, they imagine searching for an element in a sorted array. Simple, efficient, and classic. But what if I told you that binary search is capable of solving much more than just search problems?

In this blog, we’ll explore how binary search can be extended to solve optimization problems using a powerful idea called **monotonic functions**. This is known as **binary search on the answer** and it's one of the most underrated problemsolving techniques.

---

##  What Is a Monotonic Function?

A **monotonic function** is one that **never decreases** or **never increases** as the input grows.

There are two types:

- **Monotonically Increasing**:  
  If `x1 < x2`, then `f(x1) ≤ f(x2)`  
  → The function never decreases

- **Monotonically Decreasing**:  
  If `x1 < x2`, then `f(x1) ≥ f(x2)`  
  → The function never increases

###  Example:

Let’s say we define a boolean function `f(x)`:
```markdown
x:     1   2   3   4   5   6
f(x):  F   F   F   T   T   T
               ↑
         Transition Point
```

This is a **monotonically increasing** function once `f(x)` becomes `True`, it stays `True` for all greater `x`.

This **transition point** is exactly what binary search helps us find.

---

## Why and How to Use Monotonic Functions in Binary Search

###  The Logic Behind It

Traditional binary search works on sorted arrays because the order guarantees that one half of the array can be discarded after each comparison.

Now imagine we have a **decision function** `f(x)` that returns:
- `True` → the condition is satisfied at `x`
- `False` → the condition is not satisfied at `x`

If `f(x)` is **monotonic**, then:
- If `f(x)` is `True`, all values greater than `x` will also be `True` (monotonically increasing).
- If `f(x)` is `False`, all values less than `x` will also be `False`.

This means we can **binary search the smallest `x` where `f(x)` becomes True**, or vice versa.
## How It Works

Consider a scenario where our search space is `x = 1` to `10` and the transition point is at `x = 7`. The boolean function `f(x)` is defined as:
```markdown
x:     1   2   3   4   5   6   7   8   9   10 
f(x):  F   F   F   F   F   F   F   F   F    F
```

#### Binary Search Process:

1. **Initial Range:** `[1, 10]`
   - Calculate mid: `mid = (1 + 10) // 2 = 5`
   - Evaluate `f(5)`: `False`
   - Since `f(5)` is `False`, we discard the left half.
   - **New Range:** `[6, 10]`

2. **New Range:** `[6, 10]`
   - Calculate mid: `mid = (6 + 10) // 2 = 8`
   - Evaluate `f(8)`: `True`
   - Since `f(8)` is `True`, the answer might be at 8 or lower.
   - **New Range:** `[6, 7]`

3. **New Range:** `[6, 7]`
   - Calculate mid: `mid = (6 + 7) // 2 = 6`
   - Evaluate `f(6)`: `False`
   - Since `f(6)` is `False`, we discard the left side.
   - **New Range:** `[7, 7]`

4. **Final Range:** `[7, 7]`
   - `mid = 7`
   - Evaluate `f(7)`: `True`
   - **Answer found:** The smallest `x` where `f(x)` becomes `True` is **7**.

---

###  Step by Step: How to Apply Binary Search on Monotonic Functions

Let’s say we want to find the **smallest value of `x`** such that some condition `f(x)` becomes true.

####  Step 1: Identify the Search Space

Choose the minimum (`low`) and maximum (`high`) possible values of `x`.

####  Step 2: Define a Predicate Function `f(x)`

This function should return `True` or `False` based on whether a particular value of `x` satisfies your condition.

####  Step 3: Check Monotonicity

Ensure that `f(x)` is monotonic (once true, always true OR once false, always false).

####  Step 4: Apply Binary Search

```python
def binary_search_answer(low, high, f):
    result = high
    while low <= high:
        mid = (low + high) // 2
        if f(mid):
            result = mid         # possible answer, search left
            high = mid - 1
        else:
            low = mid + 1        # search right
    return result
```
## Real Life Use Cases

- **Minimize something:** Find the smallest value `x` for which some constraint becomes valid (`f(x) = True`).
- **Maximize something:** Find the largest `x` for which the constraint still holds (`f(x) = True`).

#### Examples:

| Goal                                   | `x` is the variable being optimized | Predicate Function (`f(x)`)             |
|----------------------------------------|---------------------------------------|-----------------------------------------|
| Smallest speed to finish work          | `x = speed`                           | Can finish within time `h`?             |
| Minimum capacity to ship in `d` days    | `x = capacity`                        | Can ship all packages in `d` days?      |
| Minimum max sum after splitting array  | `x = max sum allowed`                 | Can split array under that sum?         |
| Maximize minimum distance between cows | `x = distance`                        | Can place cows with this min gap?       |

---

##  Example: Koko Eating Bananas

### Problem:

Koko has piles of bananas and `h` hours to eat them. She can eat `k` bananas per hour (from one pile at a time). What is the **minimum `k`** such that she finishes all bananas in time?

### Input:

```python
piles = [3, 6, 7, 11]
h = 8
```
Step by Step Breakdown
Step 1: Define the Search Space
Koko eats at least 1 banana/hour, at most max(piles) = 11.
```python
left, right = 1, max(piles)
```
step 2: Define Predicate can_finish(k)
```python
import math

def can_finish(speed, piles, h):
    total_hours = sum(math.ceil(p / speed) for p in piles)
    return total_hours <= h
```
Step 3: Apply Binary Search
```python
def min_eating_speed(piles, h):
    left, right = 1, max(piles)
    result = right

    while left <= right:
        mid = (left + right) // 2
        if can_finish(mid, piles, h):
            result = mid
            right = mid - 1
        else:
            left = mid + 1

    return result
```
Search Space:
```markdown
x:     1   2   3   4   5   6   7  
f(x):  F   F   F   F   T   T   T
                       ↑
                    First T(Answer)
```
We use binary search to find the first value of `k` where `can_finish(k)` becomes True.

##  More Problems Using This Pattern

| Problem                               | Goal                                 | Monotonic Predicate                  |
|---------------------------------------|--------------------------------------|--------------------------------------|
| Koko Eating Bananas                   | Minimize eating speed                | `canFinish(speed)`                   |
| Capacity to Ship Packages in D Days   | Minimize ship capacity               | `canShip(capacity)`                  |
| Split Array Largest Sum               | Minimize largest subarray sum        | `canSplitWithMaxSum(maxSum)`          |
| Allocate Minimum Number of Pages      | Minimize max pages per student       | `canAllocate(pagesPerStudent)`       |
| Aggressive Cows                       | Maximize minimum distance            | `isDistanceFeasible(distance)`       |

##  Practice These Patterns

Try out these problems to master the concept:

-  [Leetcode 875: Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/)
-  [Leetcode 1011: Capacity To Ship Packages](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/)
-  [GFG: Allocate Minimum Number of Pages](https://www.geeksforgeeks.org/allocate-minimum-number-pages/)

##  Final Thoughts

Binary Search is not just a search algorithm—it's a powerful optimization tool.  
If you can define a search space and a monotonic predicate, you can use binary search to find the optimal solution in logarithmic time.

Learn to see monotonicity, and you'll see binary search opportunities everywhere.

