2026/06/29  -  21:08
status: 

Tags: [[Data Structures]] 

Here’s a concise, exam‑ready summary of all the key sorting algorithms.  
Master this table and you’ll handle almost any Arshad question on sorting.

---

## 1. Comparison‑Based Sorts (Lower Bound: Ω(n log n))

| Algorithm | Core Idea | Best | Average | Worst | Space (aux) | Stable? | In‑place? | Notes |
|-----------|-----------|------|---------|-------|-------------|---------|-----------|-------|
| **Selection Sort** | Repeatedly select minimum from unsorted part, swap to front | Θ(n²) | Θ(n²) | Θ(n²) | O(1) | No (swapping breaks order) | Yes | Always does n‑1 swaps (minimal writes). Number of comparisons: n(n‑1)/2 always. Not adaptive. |
| **Bubble Sort** | Repeatedly swap adjacent out‑of‑order elements | Θ(n) (already sorted, with flag) | Θ(n²) | Θ(n²) | O(1) | Yes | Yes | Adaptive if optimized (stops early). Max comparisons: n(n‑1)/2; swaps equal to inversions. |
| **Insertion Sort** | Build sorted part by inserting each new element into correct position | Θ(n) (already sorted) | Θ(n²) | Θ(n²) | O(1) | Yes | Yes | Very efficient for small n or nearly sorted data (adaptive). Number of swaps = inversions. Best choice for k‑sorted arrays (O(nk)). |
| **Shell Sort** | Insertion sort on widely spaced elements, reducing gap | Ω(n log n) (depends on gap) | O(n^(4/3)?) | O(n^(3/2)) (depends) | O(1) | No | Yes | Gap sequence matters (Hibbard, Knuth). Not stable. Useful for medium data. |
| **Merge Sort** | Divide & conquer: split in halves, sort recursively, merge | Θ(n log n) | Θ(n log n) | Θ(n log n) | O(n) (merging) | Yes | No (requires extra array) | Predictable O(n log n). Best for linked lists (in‑place with O(1) extra? tricky but possible). Comparisons: ≈ n log n – n + 1. Ideal for external sorting. |
| **Quick Sort** | Partition around a pivot, recursively sort partitions | Θ(n log n) (if good pivot) | Θ(n log n) | O(n²) (if bad pivot, e.g., sorted) | O(log n) (recursion stack) | No (naive) | Yes (in‑place version) | Fastest in practice (cache‑friendly). Use random or median‑of‑three pivot to avoid worst case. Number of comparisons ≈ 2n ln n. Often unstable; stable variants exist but costly. |
| **Heap Sort** | Build max‑heap, repeatedly extract max | Θ(n log n) | Θ(n log n) | Θ(n log n) | O(1) (if iterative) | No | Yes | Guaranteed O(n log n) worst case, but not stable. Build‑heap is O(n), extraction O(n log n). Comparisons ≈ 2n log n. Slower than quicksort in practice. |
| **Tree Sort** | Insert all elements into a balanced BST, then inorder traversal | Θ(n log n) | Θ(n log n) | O(n²) (if unbalanced) | O(n) | Yes | No | Not used in practice unless BST is balanced (AVL). |
| **Tim Sort** | Hybrid: merge + insertion, uses natural runs | O(n) | O(n log n) | O(n log n) | O(n) | Yes | No | Python/Java default. Not usually asked in Iran Arshad, but mention for completeness. |

---

## 2. Non‑Comparison Sorts (Break Ω(n log n))

| Algorithm | Core Idea | Time (best = avg = worst) | Space (aux) | Stable? | In‑place? | Notes |
|-----------|-----------|--------------------------|-------------|---------|-----------|-------|
| **Counting Sort** | Count frequencies of each distinct key, compute prefix sums, place directly | Θ(n + k) where k = range of keys | O(n + k) | Yes | No | Only for integer keys with small range (k not too large). Stable if implemented carefully. |
| **Radix Sort** | Sort digit by digit (LSD or MSD) using a stable linear sort (e.g., counting) | Θ(d*(n + b)) where d = digits, b = base | O(n + b) | Yes | No | Works on integers/strings. LSD radix sort sorts least significant digit first; must use stable sub‑sort. Complexity: O(n) if d constant and b=O(n). Often b=10 or 256. |
| **Bucket Sort** | Distribute elements into uniformly spaced buckets, sort each bucket (e.g., insertion) | O(n) average; O(n²) worst | O(n + k) | Yes (depends on inner sort) | No | Assumes uniform distribution. Works on floating numbers in [0,1). Bucket count k ≈ n. |

---

## 3. Key Concepts & Memory Hooks

- **Stability:** A sort is stable if equal keys keep their original relative order.  
  *Stable:* Bubble, Insertion, Merge, Counting, Radix, Bucket.  
  *Unstable:* Selection, Quick, Heap, Shell.
- **In‑place:** Uses O(1) extra space (or O(log n) for recursion).  
  *In‑place:* Selection, Bubble, Insertion, Heap, Shell, Quick (with tail recursion).  
  *Not in‑place:* Merge (needs O(n)), Counting/Radix/Bucket (need extra arrays).
- **Adaptivity:** Does it perform fewer steps when data is nearly sorted?  
  Insertion, Bubble (optimized), Tim Sort are adaptive.  
  Selection, Merge, Heap are not adaptive.
- **Minimum number of comparisons** for comparison sort: \(\lceil \log_2(n!) \rceil\) ≈ n log n – 1.44n (decision tree lower bound).
- **When to use which?**
  - Small n (< 50): Insertion sort.
  - General large n, need stability: Merge sort (or Tim Sort).
  - In‑place, average fast: Quick sort.
  - Guaranteed O(n log n) in‑place: Heap sort.
  - Integer keys, small range: Counting sort.
  - Integer keys with many digits: Radix sort.
  - Uniform floats: Bucket sort.

---

## 4. Space & Time Summary (Cheat Sheet)

| Algorithm | Time (Worst) | Space (Aux) | Stable |
|-----------|--------------|-------------|--------|
| Selection | O(n²) | O(1) | No |
| Bubble | O(n²) | O(1) | Yes |
| Insertion | O(n²) | O(1) | Yes |
| Shell | ≈O(n^(1.5)) | O(1) | No |
| Merge | O(n log n) | O(n) | Yes |
| Quick | O(n²) (but rare) | O(log n) | No |
| Heap | O(n log n) | O(1) | No |
| Counting | O(n + k) | O(n + k) | Yes |
| Radix | O(d·(n+b)) | O(n + b) | Yes |
| Bucket | O(n²) (worst) | O(n) | Depends |

Use this sheet for last‑minute review. Let me know if you’d like a set of typical multiple‑choice questions to test this knowledge.


# References
