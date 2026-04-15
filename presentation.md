# Divide and Conquer

**Computer Science Fundamentals Series**

Master theorem · Merge sort · Quick sort · Strassen's algorithm · Closest pair · FFT

*Mid-level software engineer track -- 20 slides*

---

## Table of Contents

1. [The Paradigm: Divide, Conquer, Combine](#slide-02--the-paradigm-divide-conquer-combine)
2. [Recurrence Relations](#slide-03--recurrence-relations)
3. [The Master Theorem](#slide-04--the-master-theorem)
4. [Merge Sort](#slide-05--merge-sort)
5. [Merge Sort -- Analysis](#slide-06--merge-sort--analysis)
6. [Quick Sort](#slide-07--quick-sort)
7. [Partition Schemes -- Lomuto & Hoare](#slide-08--partition-schemes--lomuto--hoare)
8. [Binary Search & Variants](#slide-09--binary-search--variants)
9. [Strassen's Matrix Multiplication](#slide-10--strassens-matrix-multiplication)
10. [Closest Pair of Points](#slide-11--closest-pair-of-points)
11. [Karatsuba Multiplication](#slide-12--karatsuba-multiplication)
12. [Maximum Subarray Problem](#slide-13--maximum-subarray-problem)
13. [Median of Medians / Selection](#slide-14--median-of-medians--selection)
14. [FFT -- Fast Fourier Transform](#slide-15--fft--fast-fourier-transform)
15. [FFT -- Applications & Complexity](#slide-16--fft--applications--complexity)
16. [Parallelism in D&C](#slide-17--parallelism-in-dc)
17. [Common Pitfalls & Optimisations](#slide-18--common-pitfalls--optimisations)
18. [D&C in Practice](#slide-19--dc-in-practice)
19. [Summary & Further Reading](#slide-20--summary--further-reading)

---

## Slide 02 -- The Paradigm: Divide, Conquer, Combine

### Three steps

Every divide-and-conquer algorithm follows the same recursive blueprint:

1. **Divide** -- break the problem into smaller, independent subproblems of the same type
2. **Conquer** -- solve each subproblem recursively; if small enough, solve directly (base case)
3. **Combine** -- merge the subproblem solutions into a solution for the original problem

### Why it works

- Reduces a problem of size `n` to `a` subproblems of size `n/b`
- Each level of recursion does `O(n^d)` work to split and merge
- Total cost depends on the balance between branching factor `a` and shrinkage `b`

### When to use D&C

- Problem has **optimal substructure** -- solution builds from sub-solutions
- Subproblems are **independent** -- no shared mutable state
- Combining is efficient -- merging cost must not dominate
- Examples: sorting, searching, geometric problems, algebraic transforms

> **D&C vs dynamic programming:** both decompose problems. D&C subproblems are independent (no overlap); DP subproblems overlap and require memoisation.

---

## Slide 03 -- Recurrence Relations

### Defining runtime recursively

A D&C algorithm with `a` subproblems of size `n/b` and `O(n^d)` combine work has recurrence:

```
T(n) = a · T(n/b) + O(n^d)
```

### Common recurrence examples

| Algorithm | Recurrence | Solution |
|-----------|-----------|----------|
| Binary search | `T(n) = T(n/2) + O(1)` | `O(log n)` |
| Merge sort | `T(n) = 2T(n/2) + O(n)` | `O(n log n)` |
| Karatsuba | `T(n) = 3T(n/2) + O(n)` | `O(n^1.585)` |
| Strassen | `T(n) = 7T(n/2) + O(n^2)` | `O(n^2.807)` |
| Naive multiply | `T(n) = 4T(n/2) + O(n)` | `O(n^2)` |

### Solution methods

- **Substitution** -- guess the form, prove by induction
- **Recursion tree** -- draw the tree, sum work per level
- **Master theorem** -- closed-form for `T(n) = aT(n/b) + O(n^d)`

---

## Slide 04 -- The Master Theorem

### Statement

For recurrence `T(n) = a · T(n/b) + O(n^d)` where `a >= 1`, `b > 1`, `d >= 0`:

| Case | Condition | Result |
|------|----------|--------|
| **Case 1** | `d < log_b(a)` | `T(n) = O(n^{log_b(a)})` |
| **Case 2** | `d = log_b(a)` | `T(n) = O(n^d · log n)` |
| **Case 3** | `d > log_b(a)` | `T(n) = O(n^d)` |

### Intuition

- **Case 1:** leaves dominate -- recursion branches faster than work shrinks
- **Case 2:** balanced -- each level does equal work; `log n` levels total
- **Case 3:** root dominates -- combine cost dwarfs recursive cost

### Worked examples

- **Merge sort:** `a=2, b=2, d=1` → `log_2(2)=1=d` → Case 2 → `O(n log n)`
- **Binary search:** `a=1, b=2, d=0` → `log_2(1)=0=d` → Case 2 → `O(log n)`
- **Strassen:** `a=7, b=2, d=2` → `log_2(7)≈2.807>2` → Case 1 → `O(n^2.807)`
- **Karatsuba:** `a=3, b=2, d=1` → `log_2(3)≈1.585>1` → Case 1 → `O(n^1.585)`

> The Master Theorem does not cover all recurrences. Akra-Bazzi generalises it to unequal subproblem sizes.

---

## Slide 05 -- Merge Sort

### The canonical D&C sort

1. **Divide** -- split array into two halves
2. **Conquer** -- recursively sort each half
3. **Combine** -- merge two sorted halves in `O(n)`

### Pseudocode

```
merge_sort(A, lo, hi):
    if hi - lo <= 1: return
    mid = (lo + hi) / 2
    merge_sort(A, lo, mid)
    merge_sort(A, mid, hi)
    merge(A, lo, mid, hi)

merge(A, lo, mid, hi):
    L = A[lo..mid]
    R = A[mid..hi]
    i = j = 0, k = lo
    while i < |L| and j < |R|:
        if L[i] <= R[j]: A[k++] = L[i++]
        else:             A[k++] = R[j++]
    copy remaining L or R into A
```

### Properties

- **Time:** `O(n log n)` worst, average, and best case
- **Space:** `O(n)` auxiliary (for the merge buffer)
- **Stable:** yes -- equal elements preserve their original order
- **Not in-place:** requires extra memory proportional to input size

---

## Slide 06 -- Merge Sort -- Analysis

### Recursion tree

```
Level 0:   n work (1 merge of n)
Level 1:   n work (2 merges of n/2)
Level 2:   n work (4 merges of n/4)
  ...
Level k:   n work (2^k merges of n/2^k)
  ...
Level log n: n work (n merges of 1)

Total: n · log n
```

### Practical considerations

- **Cache performance:** bottom-up (iterative) merge sort is more cache-friendly
- **Timsort:** Python and Java use a hybrid -- natural merge sort + insertion sort for small runs. Exploits partially-sorted input for `O(n)` best case
- **External merge sort:** the standard algorithm for sorting data that does not fit in RAM. Split into sorted runs, merge runs from disk using a priority queue

### Inversion counting

Merge sort can count inversions (pairs where `i < j` but `A[i] > A[j]`) during the merge step. Every time a right-side element is chosen before a left-side element, it crosses over all remaining left elements -- add that count.

> Merge sort is the **comparison sort baseline** -- guaranteed `O(n log n)` regardless of input distribution.

---

## Slide 07 -- Quick Sort

### Algorithm

1. **Divide** -- choose a pivot, partition the array around it
2. **Conquer** -- recursively sort elements less than pivot and greater than pivot
3. **Combine** -- nothing; partitioning is in-place

### Pseudocode (Lomuto)

```
quicksort(A, lo, hi):
    if lo < hi:
        p = partition(A, lo, hi)
        quicksort(A, lo, p - 1)
        quicksort(A, p + 1, hi)
```

### Complexity

| Case | Time | When |
|------|------|------|
| **Best** | `O(n log n)` | Balanced partitions every time |
| **Average** | `O(n log n)` | Random or randomised pivot |
| **Worst** | `O(n^2)` | Already sorted + first/last pivot |

### Why Quick Sort wins in practice

- **In-place:** `O(log n)` stack space vs `O(n)` for merge sort
- **Cache-friendly:** sequential access pattern during partitioning
- **Small constant:** fewer data movements than merge sort on average
- **Randomised pivot** eliminates adversarial worst case with high probability

---

## Slide 08 -- Partition Schemes -- Lomuto & Hoare

### Lomuto partition

```
partition(A, lo, hi):
    pivot = A[hi]
    i = lo
    for j = lo to hi - 1:
        if A[j] <= pivot:
            swap(A[i], A[j])
            i++
    swap(A[i], A[hi])
    return i
```

- Simple, easy to understand and prove correct
- ~`n` comparisons, up to `n` swaps
- Poor on arrays with many duplicates (all equal → `O(n^2)`)

### Hoare partition

```
partition(A, lo, hi):
    pivot = A[lo]
    i = lo - 1, j = hi + 1
    loop:
        do i++ while A[i] < pivot
        do j-- while A[j] > pivot
        if i >= j: return j
        swap(A[i], A[j])
```

- Roughly `n/2` swaps on average -- fewer data movements
- Does not place pivot in final position -- recursive calls use `[lo..j]` and `[j+1..hi]`
- Better cache behaviour on nearly-sorted data

### Three-way partitioning (Dutch National Flag)

Partitions into `< pivot`, `= pivot`, `> pivot`. Essential when many duplicates are present. Used by `std::sort` variants and `pdqsort`.

---

## Slide 09 -- Binary Search & Variants

### Classic binary search

```
binary_search(A, target):
    lo, hi = 0, len(A) - 1
    while lo <= hi:
        mid = lo + (hi - lo) / 2
        if A[mid] == target: return mid
        elif A[mid] < target: lo = mid + 1
        else: hi = mid - 1
    return -1
```

- **Time:** `O(log n)` -- recurrence `T(n) = T(n/2) + O(1)`, Master Case 2
- **Space:** `O(1)` iterative, `O(log n)` recursive

### Variants

| Variant | Description |
|---------|------------|
| **Lower bound** | First index where `A[i] >= target` (`std::lower_bound`, `bisect_left`) |
| **Upper bound** | First index where `A[i] > target` (`std::upper_bound`, `bisect_right`) |
| **Exponential search** | Find range via doubling, then binary search within -- `O(log k)` for element at position `k` |
| **Interpolation search** | Estimate position by value distribution -- `O(log log n)` for uniform data |
| **Fractional cascading** | Search multiple sorted lists; preprocess to share results -- amortised `O(log n + k)` |

### Off-by-one pitfalls

- Use `lo + (hi - lo) / 2` not `(lo + hi) / 2` to avoid integer overflow
- Clarify inclusive vs exclusive bounds before coding
- Test on arrays of size 0, 1, 2 -- most binary search bugs hide there

---

## Slide 10 -- Strassen's Matrix Multiplication

### Naive matrix multiply

Multiply two `n x n` matrices: three nested loops → `O(n^3)`.

### Strassen's insight (1969)

Divide each matrix into four `n/2 x n/2` submatrices. Naive block multiply needs **8** recursive multiplications. Strassen uses clever linear combinations to reduce this to **7**.

### The seven products

```
M1 = (A11 + A22)(B11 + B22)
M2 = (A21 + A22) B11
M3 = A11 (B12 - B22)
M4 = A22 (B21 - B11)
M5 = (A11 + A12) B22
M6 = (A21 - A11)(B11 + B12)
M7 = (A12 - A22)(B21 + B22)
```

Result blocks computed from `M1..M7` using only addition/subtraction.

### Complexity

| Method | Multiplications | Time |
|--------|----------------|------|
| Naive | 8 per level | `O(n^3)` |
| Strassen | 7 per level | `O(n^2.807)` |
| Coppersmith-Winograd variants | — | `O(n^2.372)` (galactic) |

> Strassen is practical for `n >= 64` or so. Below that threshold, the constant factor from extra additions makes naive faster. Libraries like BLAS switch strategies based on matrix size.

---

## Slide 11 -- Closest Pair of Points

### Problem

Given `n` points in the plane, find the pair with minimum Euclidean distance.

### Brute force

Check all `n(n-1)/2` pairs → `O(n^2)`.

### D&C approach -- `O(n log n)`

1. **Sort** points by x-coordinate
2. **Divide** -- split into left and right halves by median x
3. **Conquer** -- recursively find closest pair in each half: distances `d_L`, `d_R`
4. **Combine** -- let `d = min(d_L, d_R)`. Check pairs that straddle the dividing line within a strip of width `2d`

### The strip trick

- Only points within `d` of the dividing line can form a closer pair
- For each point in the strip, compare to at most **7** subsequent points (sorted by y)
- Strip processing is `O(n)`, not `O(n^2)`

### Recurrence

```
T(n) = 2T(n/2) + O(n)  →  O(n log n)
```

> One of the most elegant D&C algorithms. The insight that the strip check is `O(n)` -- not `O(n^2)` -- is the key.

---

## Slide 12 -- Karatsuba Multiplication

### Problem

Multiply two `n`-digit integers. School algorithm: `O(n^2)`.

### Karatsuba's trick (1960)

Split each number into high and low halves:

```
x = x_H · B^m + x_L
y = y_H · B^m + y_L

x · y = x_H·y_H · B^2m
      + (x_H·y_L + x_L·y_H) · B^m
      + x_L·y_L
```

This needs **4** multiplications of `n/2`-digit numbers. Karatsuba observed:

```
z0 = x_L · y_L
z2 = x_H · y_H
z1 = (x_L + x_H)(y_L + y_H) - z0 - z2
```

Only **3** multiplications. Recurrence: `T(n) = 3T(n/2) + O(n)` → `O(n^1.585)`.

### In practice

- Python's `int` multiplication uses Karatsuba for large numbers
- GMP (GNU Multiple Precision) chains: schoolbook → Karatsuba → Toom-Cook → FFT-based as `n` grows
- Crossover to Karatsuba typically around 20--80 digits

---

## Slide 13 -- Maximum Subarray Problem

### Problem

Find the contiguous subarray with the largest sum in an array of integers.

### D&C approach -- `O(n log n)`

1. **Divide** -- split array at midpoint
2. **Conquer** -- recursively find max subarray in left half and right half
3. **Combine** -- find max subarray crossing the midpoint (linear scan left and right from mid)
4. Return the maximum of left, right, and crossing

```
T(n) = 2T(n/2) + O(n)  →  O(n log n)
```

### Kadane's algorithm -- `O(n)`

```
kadane(A):
    max_ending_here = max_so_far = A[0]
    for i = 1 to n-1:
        max_ending_here = max(A[i], max_ending_here + A[i])
        max_so_far = max(max_so_far, max_ending_here)
    return max_so_far
```

- Single pass, `O(1)` space
- Essentially dynamic programming (subproblem: best subarray ending at index `i`)

### D&C vs Kadane

| | D&C | Kadane |
|-|-----|--------|
| **Time** | `O(n log n)` | `O(n)` |
| **Parallelism** | Naturally parallel | Sequential |
| **Teaching value** | Demonstrates combine step | Demonstrates DP |

> The maximum subarray is a case where D&C is instructive but not optimal. Kadane wins for serial execution; D&C wins for parallel.

---

## Slide 14 -- Median of Medians / Selection

### The selection problem

Find the k-th smallest element in an unsorted array.

### Quickselect -- expected `O(n)`

Partition around a random pivot. Recurse only into the side containing index `k`. Expected `O(n)`, worst case `O(n^2)`.

### Median of medians -- worst-case `O(n)`

1. Divide array into groups of 5
2. Find the median of each group (brute force)
3. Recursively find the median of those medians → pivot
4. Partition around this pivot
5. Recurse into the correct side

### Why groups of 5?

The pivot is guaranteed to be between the 30th and 70th percentile. This ensures each recursive call eliminates at least 30% of elements:

```
T(n) = T(n/5) + T(7n/10) + O(n)
```

This solves to `O(n)` because `1/5 + 7/10 = 9/10 < 1`.

### Practical notes

- `std::nth_element` in C++ uses Introselect: Quickselect with median-of-medians fallback
- The constant factor is large -- Quickselect with random pivot is faster in practice
- Median of medians is primarily of theoretical importance: it proves linear selection is possible

---

## Slide 15 -- FFT -- Fast Fourier Transform

### The Discrete Fourier Transform (DFT)

Given a sequence of `n` complex numbers, the DFT computes `n` frequency components. Naive evaluation: `O(n^2)`.

### Cooley-Tukey FFT (1965)

Divide the DFT into two half-size DFTs on even-indexed and odd-indexed elements:

```
X[k] = E[k] + ω^k · O[k]      (k = 0..n/2 - 1)
X[k + n/2] = E[k] - ω^k · O[k]
```

where `E` = DFT of even elements, `O` = DFT of odd elements, `ω = e^{-2πi/n}`.

### Complexity

```
T(n) = 2T(n/2) + O(n)  →  O(n log n)
```

Master theorem Case 2. Reduces DFT from `O(n^2)` to `O(n log n)`.

### The butterfly operation

Each stage combines pairs of values using a "butterfly" pattern: one addition and one subtraction, multiplied by a twiddle factor `ω^k`. This structure maps naturally to hardware and SIMD.

---

## Slide 16 -- FFT -- Applications & Complexity

### Applications

- **Polynomial multiplication:** multiply two degree-n polynomials in `O(n log n)` instead of `O(n^2)` -- FFT both, pointwise multiply, inverse FFT
- **Big integer multiplication:** Schonhage-Strassen algorithm uses FFT for `O(n log n log log n)` integer multiplication
- **Signal processing:** spectral analysis, filtering, convolution
- **Audio/image compression:** MP3, JPEG rely on DCT (closely related to FFT)
- **String matching:** convolution-based pattern matching

### Number-Theoretic Transform (NTT)

FFT over finite fields (integers mod prime `p`) instead of complex numbers. Avoids floating-point errors entirely. Used in competitive programming and cryptographic polynomial multiplication.

### Related transforms

| Transform | Domain | Use |
|-----------|--------|-----|
| **FFT** | Complex numbers | General signal processing |
| **NTT** | Integers mod `p` | Exact polynomial arithmetic |
| **DCT** | Real numbers | Compression (JPEG, MP3) |
| **Walsh-Hadamard** | Binary/Boolean | Subset-sum convolutions |

---

## Slide 17 -- Parallelism in D&C

### Natural parallelism

D&C subproblems are independent -- they can run on separate cores, threads, or machines without synchronisation until the combine step.

### Fork-join model

```
fork-join quicksort(A, lo, hi):
    if hi - lo < THRESHOLD:
        insertion_sort(A, lo, hi)
        return
    p = partition(A, lo, hi)
    fork: quicksort(A, lo, p - 1)
    fork: quicksort(A, p + 1, hi)
    join
```

### Work and span

| Metric | Definition | Merge sort |
|--------|-----------|------------|
| **Work** `W(n)` | Total operations across all processors | `O(n log n)` |
| **Span** `S(n)` | Longest sequential dependency chain | `O(n)` (merge is sequential) |
| **Parallelism** | `W(n) / S(n)` | `O(log n)` |

### Practical frameworks

| Framework | Language | Model |
|-----------|---------|-------|
| **Fork/Join** | Java | `RecursiveTask`, work-stealing |
| **TBB** | C++ | `parallel_for`, `parallel_reduce` |
| **Cilk** | C/C++ | `cilk_spawn`, `cilk_sync` |
| **Rayon** | Rust | `par_iter`, work-stealing |
| **multiprocessing** | Python | Process pools (GIL workaround) |

> Amdahl's Law: speedup is limited by the sequential fraction. In merge sort, the final merge is sequential and dominates at high core counts.

---

## Slide 18 -- Common Pitfalls & Optimisations

### Pitfalls

- **Stack overflow** -- deep recursion on large inputs. Use iterative bottom-up or increase stack size
- **Small subproblem overhead** -- recursive calls on tiny arrays waste function-call overhead. Switch to insertion sort below a threshold (typically 16--32 elements)
- **Worst-case pivot selection** -- deterministic Quick Sort on sorted input is `O(n^2)`. Always randomise or use median-of-three
- **Unnecessary copying** -- naive merge sort copies arrays at every level. Use index-based merging with a shared buffer
- **Integer overflow** -- `(lo + hi) / 2` overflows for large arrays. Use `lo + (hi - lo) / 2`

### Optimisations

| Technique | Applies to | Benefit |
|-----------|-----------|---------|
| **Hybrid cutoff** | All D&C sorts | Insertion sort for `n < 16`; fewer function calls |
| **Tail recursion** | Quick Sort | Recurse on smaller partition first; iterate on larger → `O(log n)` stack |
| **Introspective sort** | Quick Sort | Switch to heap sort if recursion depth exceeds `2 log n` (introsort) |
| **Bottom-up merge** | Merge Sort | Iterative; better cache locality |
| **Bit-reversal permutation** | FFT | In-place iterative FFT avoids recursion overhead |

---

## Slide 19 -- D&C in Practice

### Standard library implementations

| Language | Sort algorithm | Notes |
|----------|--------------|-------|
| **C++** | Introsort (`std::sort`) | Quick Sort + Heap Sort + Insertion Sort |
| **Python** | Timsort (`sorted`) | Merge Sort + Insertion Sort; exploits runs |
| **Java** | Dual-pivot Quick Sort (primitives), Timsort (objects) | Stability matters for objects |
| **Rust** | pdqsort (`sort_unstable`) | Pattern-defeating Quick Sort |
| **Go** | pdqsort (since 1.19) | Replaced previous introsort |

### D&C beyond sorting

| Domain | Algorithm | D&C idea |
|--------|----------|----------|
| **Computational geometry** | Convex hull (merge hull) | Merge two convex hulls |
| **Linear algebra** | Strassen, FFT-based solvers | Block decomposition |
| **Databases** | External merge sort, merge join | Divide data across disk pages |
| **Distributed systems** | MapReduce | Map = divide, Reduce = combine |
| **Graphics** | BSP trees, k-d trees | Spatial subdivision |

> If your problem decomposes into independent subproblems of the same type, D&C is likely the right paradigm.

---

## Slide 20 -- Summary & Further Reading

### Key takeaways

- Divide and conquer is a fundamental algorithmic paradigm: divide, conquer, combine
- The Master Theorem provides `O`-notation for most D&C recurrences in closed form
- Merge Sort is the stable `O(n log n)` baseline; Quick Sort wins in practice due to cache and in-place operation
- Strassen, Karatsuba, and FFT show that "obvious" lower bounds can be beaten by clever decomposition
- Binary search is the simplest D&C -- understand its variants and off-by-one traps
- D&C algorithms are naturally parallel: independent subproblems map to cores/machines
- Real-world implementations are hybrid: switch strategies at small sizes and degenerate inputs

### Recommended reading

| Source | Description |
|--------|------------|
| **CLRS** | *Introduction to Algorithms* -- chapters 2, 4, 7, 9, 33 cover all major D&C topics |
| **Kleinberg & Tardos** | *Algorithm Design* -- excellent D&C chapter with closest pair, FFT |
| **Sedgewick** | *Algorithms* -- practical Quick Sort and Merge Sort analysis |
| **Jeff Erickson** | [jeffe.cs.illinois.edu/teaching/algorithms](http://jeffe.cs.illinois.edu/teaching/algorithms/) -- free textbook, outstanding recursion chapter |
| **MIT 6.046** | *Design and Analysis of Algorithms* -- D&C lectures on YouTube |
