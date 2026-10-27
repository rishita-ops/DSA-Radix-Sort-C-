# DSA — Radix Sort (C++)

A **non-comparison** sorting algorithm that sorts integers digit by digit, from the least significant digit to the most significant, using a **stable counting sort** at every pass.

---

## Problem Statement

Given an array of `n` **non-negative** integers, sort it in **ascending order** using Radix Sort (LSD — Least Significant Digit).

**Example**

```
Input:  n = 8
        170 45 75 90 802 24 2 66
Output: 2 24 45 66 75 90 170 802
```

---

## Solution

```cpp
#include<bits/stdc++.h>
using namespace std;
int main()
{
    int n;
    std::cin >> n;
    std::vector<int> arr(n);
    for (int i = 0; i < n; i++) {
        std::cin >> arr[i];
    }

    int max_val = arr[0];
    for (int i = 1; i < n; i++) {
        if (arr[i] > max_val) {
            max_val = arr[i];
        }
    }

    for (int exp = 1; max_val / exp > 0; exp *= 10) {
        std::vector<int> output(n);
        int count[10] = {0};

        for (int i = 0; i < n; i++) {
            count[(arr[i] / exp) % 10]++;
        }

        for (int i = 1; i < 10; i++) {
            count[i] += count[i - 1];
        }

        for (int i = n - 1; i >= 0; i--) {
            output[count[(arr[i] / exp) % 10] - 1] = arr[i];
            count[(arr[i] / exp) % 10]--;
        }

        for (int i = 0; i < n; i++) {
            arr[i] = output[i];
        }
    }

    for (int i = 0; i < n; i++) {
        std::cout << arr[i] << " ";
    }
    return 0;
}
```

---

## The Critical Idea

Radix Sort never compares two elements. Instead it sorts by **one digit at a time**, starting from the ones place:

1. Sort everything by the **ones** digit.
2. Sort everything by the **tens** digit.
3. Sort everything by the **hundreds** digit — and so on, until the largest number's digits are exhausted.

This works because every pass is **stable**: elements with equal digits keep the order from the previous pass. By the time the most significant digit is processed, the earlier passes have already ordered the ties correctly.

The stable per-digit sort is a **counting sort** on a tiny range (digits `0–9`), which is why only a 10-slot `count` array is needed.

---

## How the Loop Works

### Finding the number of passes

```cpp
for (int exp = 1; max_val / exp > 0; exp *= 10)
```

- `exp` is the place value: `1`, `10`, `100`, ...
- The loop continues while the maximum still has a digit at that position. The number of passes equals the number of digits in `max_val`.

### One counting-sort pass

For each `exp`, the digit of `arr[i]` is `(arr[i] / exp) % 10`.

| Step | Code | Purpose |
|------|------|---------|
| 1. Count | `count[digit]++` | How many elements have each digit |
| 2. Prefix sum | `count[i] += count[i-1]` | `count[d]` becomes the **end position** for digit `d` |
| 3. Place (right → left) | `output[count[d]-1] = arr[i]; count[d]--` | Puts each element in its slot; going right to left keeps the pass **stable** |
| 4. Copy back | `arr[i] = output[i]` | Prepares the array for the next digit |

---

## Pseudocode

```
RADIX-SORT(arr, n):
    max = maximum of arr
    for exp = 1; max / exp > 0; exp = exp * 10:
        COUNTING-SORT-BY-DIGIT(arr, n, exp)

COUNTING-SORT-BY-DIGIT(arr, n, exp):
    count[0..9] = 0
    for i = 0 to n-1:
        count[(arr[i] / exp) % 10]++
    for d = 1 to 9:
        count[d] += count[d-1]
    for i = n-1 down to 0:
        d = (arr[i] / exp) % 10
        output[count[d] - 1] = arr[i]
        count[d]--
    copy output back into arr
```

---

## Dry Run

Input: `arr = [170, 45, 75, 90, 802, 24, 2, 66]`, `max_val = 802` (3 digits → 3 passes)

| Pass | exp | Digit of each element (in current order) | Array after the pass |
|------|-----|-------------------------------------------|----------------------|
| 1 | 1 | 170→0, 45→5, 75→5, 90→0, 802→2, 24→4, 2→2, 66→6 | `[170, 90, 802, 2, 24, 45, 75, 66]` |
| 2 | 10 | 170→7, 90→9, 802→0, 2→0, 24→2, 45→4, 75→7, 66→6 | `[802, 2, 24, 45, 66, 170, 75, 90]` |
| 3 | 100 | 802→8, 2→0, 24→0, 45→0, 66→0, 170→1, 75→0, 90→0 | `[2, 24, 45, 66, 75, 90, 170, 802]` |

**Output:** `2 24 45 66 75 90 170 802`

Notice that in pass 3 the elements with digit `0` (2, 24, 45, 66, 75, 90) stay in the order created by pass 2. That is stability at work.

---

## Complexity Analysis

Let `n` = number of elements, `d` = number of digits in the maximum, `b` = base (10).

| Case    | Time           | Space     |
|---------|----------------|-----------|
| Best    | O(d · (n + b)) | O(n + b)  |
| Average | O(d · (n + b)) | O(n + b)  |
| Worst   | O(d · (n + b)) | O(n + b)  |

- Each pass costs O(n + b); there are `d` passes.
- With `b = 10` constant, this simplifies to **O(d · n)** — effectively linear when `d` is small.
- **Space:** the `output` vector holds `n` elements, and `count` holds 10.
- **Stability:** Radix Sort (with a stable inner sort) is **stable**.

---

## Edge Cases

| Input | Behaviour |
|-------|-----------|
| `n = 1` | One element; one pass per digit, result unchanged |
| All elements single-digit | Exactly one pass |
| Already sorted | Same number of passes — no early exit |
| Duplicates | Sorted correctly and kept in original relative order |
| All zeros | `max_val = 0`, loop condition is false immediately; array is printed as is |
| **Negative numbers** | **Not supported** — `% 10` yields negative digits and breaks the indexing into `count` |
| `n = 0` | `arr[0]` is read before any check, which is **undefined behaviour** |
| Very large values | `exp *= 10` can overflow `int` when `max_val` is near `INT_MAX` |

---

## Code-Quality Notes

- The solution assumes **non-negative integers** and `n ≥ 1`. Add a guard for `n == 0`, and handle negatives by separating them, sorting their absolute values, and reversing.
- For values close to `INT_MAX` (about 2.1 billion), use `long long exp` to avoid overflow.
- `(arr[i] / exp) % 10` is computed twice per element per pass; storing it in a local variable would be cleaner.
- `output` is reallocated on every pass; declaring it once outside the loop avoids repeated allocation.
- The code mixes `std::` prefixes with `using namespace std;`. Pick one style.

---

## Natural Extension

- Extract `countingSortByDigit(arr, exp)` as a helper function.
- Use **base 256** or **base 1024** to reduce the number of passes.
- Support negative integers.
- Sort strings or fixed-length keys with **MSD** Radix Sort.

---

## Repository Structure

```
DSA/
└── Sorting/
    └── Radix-Sort/
        ├── radix_sort.cpp
        └── README.md
```

---

## How to Run

```bash
g++ radix_sort.cpp -o radix_sort
./radix_sort
```

**Sample session**

```
8
170 45 75 90 802 24 2 66
2 24 45 66 75 90 170 802
```

---

## Key Concepts

- **Non-comparison sorting** — breaks the O(n log n) lower bound for comparison sorts
- **LSD (least significant digit) processing**
- **Counting sort as a subroutine** — digit range is only `0–9`
- **Prefix sums** — turn counts into final positions
- **Stability** — the property that makes the digit-by-digit approach correct

---

## Why This Problem Matters

Radix Sort shows that sorting can be done **without comparing elements**, and it underpins many practical systems that sort large sets of integers or fixed-width keys.

| Problem | Concept Used |
|---------|--------------|
| Maximum Gap | Radix sort / bucket idea |
| Sort an Array | Sorting algorithms |
| Sort Integers by The Number of 1 Bits | Custom keys |
| Relative Sort Array | Counting-based ordering |
| H-Index | Counting sort |

---

## Contributing

Contributions are welcome! Fork the repository, create a feature branch, and open a pull request with a short description of the change.

---

## License

This project is licensed under the MIT License.
