Good. Since this is **the one cheatsheet source**, I’ll write it as dense A4 material, not as a tutorial. Rust-ish pseudocode, with data structures declared, invariants, traps, and proof skeletons. This is based on your uploaded CS-250 map and the course topic list: DP, heaps, BSTs, recurrences, MST/Union-Find, shortest paths, flows, randomized analysis, hashing, quicksort, and online algorithms are all part of the course coverage.   

# CS-250 Algorithms Cheatsheet Material

## 0. Global Pattern Recognition

```text
Need sorted order?              -> sorting / BST / heap
Need min/max repeatedly?        -> heap / priority queue
Need connectivity under unions? -> union-find
Need shortest path unweighted?  -> BFS
Need shortest path weighted >=0?-> Dijkstra
Need shortest path negative?    -> Bellman-Ford
Need dependency order?          -> topological sort
Need DAG optimization?          -> topo + DP
Need choose subset optimally?   -> DP / greedy
Need exchange/cut argument?     -> greedy / MST
Need route capacity?            -> max flow
Need matching?                  -> bipartite matching via max flow
Need edge-disjoint paths?       -> max flow with cap=1
Need expected count?            -> indicator variables
Need online unknown future?     -> competitive analysis / regret
```

Correctness proof templates:

```text
Loop invariant:
1. Init: true before first iteration
2. Maintenance: one iteration preserves it
3. Termination: invariant + stop condition => result

Induction:
Base small n.
Assume true for smaller problems.
Show combine step gives correct result.

Greedy:
1. Safe choice lemma
2. Exchange argument / cut property
3. Induct after taking greedy choice

DP:
1. Define state
2. Optimal substructure recurrence
3. Base cases
4. Fill order
5. Runtime = #states * transition cost
6. Reconstruction via parent/choice table

Graph:
Usually prove with invariant over explored/frontier/relaxed/finalized vertices.
```

---

# 1. Complexity / RAM Model / Asymptotics

```text
RAM model:
- arithmetic/load/store/compare/branch = O(1)
- input size n
- worst-case usually used

O(g):     f <= c g eventually
Ω(g):     f >= c g eventually
Θ(g):     both O and Ω
o(g):     f/g -> 0
ω(g):     f/g -> ∞

Common:
log n << sqrt n << n << n log n << n^2 << n^3 << 2^n << n!
```

Useful sums:

```text
Σ_{i=1..n} i        = n(n+1)/2 = Θ(n^2)
Σ_{i=1..n} 1        = n
Σ_{i=0..log n} n    = n log n
Σ_{i=0..∞} 1/2^i    = 2
Σ_{h=0..∞} h/2^h    = 2
```

Master theorem:

```text
T(n) = a T(n/b) + f(n)
d = log_b(a)

if f(n) = O(n^{d-eps})        -> T(n)=Θ(n^d)
if f(n) = Θ(n^d log^k n)      -> T(n)=Θ(n^d log^{k+1} n)
if f(n) = Ω(n^{d+eps}) and a f(n/b) <= c f(n), c<1
                                -> T(n)=Θ(f(n))
```

Common recurrences:

```text
Binary search:        T(n)=T(n/2)+1        = Θ(log n)
Merge sort:           T(n)=2T(n/2)+n       = Θ(n log n)
Max subarray D&C:     T(n)=2T(n/2)+n       = Θ(n log n)
Naive matmul D&C:     T(n)=8T(n/2)+n^2     = Θ(n^3)
Strassen:             T(n)=7T(n/2)+n^2     = Θ(n^{log_2 7}) ≈ Θ(n^2.807)
Heapify:              T(n)<=T(2n/3)+1      = O(log n)
Quicksort worst:      T(n)=T(n-1)+n        = Θ(n^2)
Quicksort balanced:   T(n)=2T(n/2)+n       = Θ(n log n)
Bellman-Ford:         (V-1)*E relaxations  = Θ(VE)
BFS/DFS adjacency:    visit V + scan E     = Θ(V+E)
```

---

# 2. Basic Data Structures Under the Hood

## Array / Vec

```text
Contiguous memory.
A[i] address = base + i * sizeof(T)
Random access O(1)
Insert/delete middle O(n)
Push end amortized O(1)
```

Rust-ish:

```rust
let mut a: Vec<i32> = Vec::new();
a.push(5);
let x: i32 = a[0];
let n: usize = a.len();

for i in 0..a.len() { /* a[i] */ }
for x in a.iter() { /* &i32 */ }
for &x in a.iter() { /* i32 if Copy */ }
```

Traps:

```text
- Rust indices are usize.
- Range 0..n excludes n.
- Inclusive: 0..=n.
```

---

## Stack

Under hood:

```text
Usually Vec.
LIFO.
push/pop from end.
```

```rust
let mut st: Vec<i32> = Vec::new();
st.push(x);
let top: Option<&i32> = st.last();
let x: Option<i32> = st.pop();
while let Some(x) = st.pop() {
    // process x
}
```

Ops:

```text
push O(1) amortized
pop O(1)
last O(1)
```

---

## Queue

Under hood:

```text
Circular buffer / deque.
FIFO.
```

```rust
use std::collections::VecDeque;

let mut q: VecDeque<usize> = VecDeque::new();
q.push_back(s);
while let Some(u) = q.pop_front() {
    // process u
}
```

Ops:

```text
push_back O(1)
pop_front O(1)
```

---

## Linked List

Under hood:

```text
Node { value, next }
Doubly: Node { prev, next }
Non-contiguous memory.
```

Ops:

```text
insert/delete at known node: O(1)
search: O(n)
random access: O(n)
```

Course point:

```text
Good for local insertion/deletion.
Bad for search/cache locality.
```

---

## Hash Table / HashMap / HashSet

Under hood:

```text
key -> hash(key) -> bucket index
bucket index = hash mod m
collision handled by chaining / probing
load factor α = n/m
```

Complexities:

```text
insert expected O(1), worst O(n)
search expected O(1), worst O(n)
delete expected O(1), worst O(n)

with chaining:
expected search = O(1 + α)
if m = Θ(n), α = Θ(1)
```

Rust:

```rust
use std::collections::{HashMap, HashSet};

let mut map: HashMap<i32, i32> = HashMap::new();
map.insert(k, v);

if let Some(v) = map.get(&k) {
    // v: &i32
}

let count: &mut i32 = map.entry(k).or_insert(0);
*count += 1;

let mut set: HashSet<i32> = HashSet::new();
set.insert(x);
if set.contains(&x) { }
set.remove(&x);
```

Traps:

```text
- HashSet O(1) means expected/amortized, not worst-case.
- Worst-case all keys collide -> O(n).
- Need good hash + resize.
```

---

## Binary Search Tree

Under hood:

```text
Node {
    key,
    left,
    right,
    parent
}
BST property:
left subtree keys < x.key
right subtree keys >= x.key
```

Ops:

```text
search O(h)
insert O(h)
delete O(h)
min/max O(h)
successor/predecessor O(h)
inorder traversal Θ(n)
balanced h=Θ(log n)
degenerate h=Θ(n)
```

Search:

```rust
fn tree_search(x: Node, k: i32) -> Option<Node> {
    let mut cur: Option<Node> = Some(x);

    while let Some(u) = cur {
        if k == u.key {
            return Some(u);
        } else if k < u.key {
            cur = u.left;
        } else {
            cur = u.right;
        }
    }

    None
}
```

Min / max:

```rust
fn tree_min(mut x: Node) -> Node {
    while x.left.is_some() {
        x = x.left.unwrap();
    }
    x
}

fn tree_max(mut x: Node) -> Node {
    while x.right.is_some() {
        x = x.right.unwrap();
    }
    x
}
```

Successor:

```rust
fn successor(x: Node) -> Option<Node> {
    if x.right.is_some() {
        return Some(tree_min(x.right.unwrap()));
    }

    let mut cur: Node = x;
    let mut y: Option<Node> = cur.parent;

    while y.is_some() && cur == y.unwrap().right.unwrap() {
        cur = y.unwrap();
        y = cur.parent;
    }

    y
}
```

Inorder:

```rust
fn inorder(x: Option<Node>) {
    if x.is_none() { return; }

    let u = x.unwrap();
    inorder(u.left);
    print(u.key);
    inorder(u.right);
}
```

Proof inorder sorted:

```text
Induct on subtree.
Left subtree all < root.
Right subtree all >= root.
inorder(left) sorted, then root, then inorder(right).
=> whole output sorted.
```

---

# 3. Sorting Algorithms

## Insertion Sort

Use:

```text
Good for small/nearly sorted arrays.
In-place.
Stable.
```

Invariant:

```text
Before outer iteration j:
A[0..j] or A[1..j-1] is sorted and contains original prefix.
```

0-index Rust-ish:

```rust
fn insertion_sort(a: &mut Vec<i32>) {
    let n: usize = a.len();

    for j in 1..n {
        let key: i32 = a[j];
        let mut i: isize = j as isize - 1;

        while i >= 0 && a[i as usize] > key {
            a[(i + 1) as usize] = a[i as usize];
            i -= 1;
        }

        a[(i + 1) as usize] = key;
    }
}
```

Complexity:

```text
best sorted: Θ(n)
worst reverse: Θ(n^2)
average: Θ(n^2)
space: O(1)
```

---

## Merge Sort

Use:

```text
Guaranteed Θ(n log n)
Stable
Not in-place in simple version
```

Pseudocode:

```rust
fn merge_sort(a: &mut Vec<i32>, l: usize, r: usize) {
    // sort a[l..r], r exclusive
    if r - l <= 1 { return; }

    let m: usize = (l + r) / 2;
    merge_sort(a, l, m);
    merge_sort(a, m, r);
    merge(a, l, m, r);
}

fn merge(a: &mut Vec<i32>, l: usize, m: usize, r: usize) {
    let mut tmp: Vec<i32> = Vec::new();

    let mut i: usize = l;
    let mut j: usize = m;

    while i < m && j < r {
        if a[i] <= a[j] {
            tmp.push(a[i]);
            i += 1;
        } else {
            tmp.push(a[j]);
            j += 1;
        }
    }

    while i < m {
        tmp.push(a[i]);
        i += 1;
    }

    while j < r {
        tmp.push(a[j]);
        j += 1;
    }

    for k in 0..tmp.len() {
        a[l + k] = tmp[k];
    }
}
```

Recurrence:

```text
T(n)=2T(n/2)+Θ(n)=Θ(n log n)
space Θ(n)
```

Correctness:

```text
Induct on length.
Recursive calls sort halves.
Merge of two sorted arrays returns sorted union.
```

---

## Quicksort

Use:

```text
Fast in practice.
In-place.
Worst Θ(n^2), randomized expected Θ(n log n).
```

Partition invariant:

```text
A[l..i] <= pivot
A[i+1..j] > pivot
A[j+1..r-1] unknown
A[r] pivot
```

Lomuto partition:

```rust
fn partition(a: &mut Vec<i32>, l: usize, r: usize) -> usize {
    // partition a[l..=r], pivot = a[r]
    let pivot: i32 = a[r];
    let mut i: isize = l as isize - 1;

    for j in l..r {
        if a[j] <= pivot {
            i += 1;
            a.swap(i as usize, j);
        }
    }

    a.swap((i + 1) as usize, r);
    (i + 1) as usize
}

fn quicksort(a: &mut Vec<i32>, l: usize, r: usize) {
    if l >= r { return; }

    let q: usize = partition(a, l, r);

    if q > 0 {
        quicksort(a, l, q - 1);
    }
    quicksort(a, q + 1, r);
}
```

Randomized:

```rust
fn randomized_partition(a: &mut Vec<i32>, l: usize, r: usize) -> usize {
    let p: usize = random_usize_between(l, r);
    a.swap(p, r);
    partition(a, l, r)
}
```

Complexity:

```text
best/balanced: Θ(n log n)
expected randomized: Θ(n log n)
worst sorted bad pivot: Θ(n^2)
space recursion: O(log n) expected, O(n) worst
```

Randomized quicksort proof idea:

```text
Each pair of elements compared at most once.
Let X_ij = 1 if i,j compared.
P(X_ij=1)=2/(j-i+1).
E[#comparisons]=Σ_i<j 2/(j-i+1)=O(n log n).
```

---

## Counting Sort / Linear Sorting

Use:

```text
Keys integers in [0..k].
Not comparison-based.
Stable if using prefix sums.
```

```rust
fn counting_sort(a: &Vec<usize>, k: usize) -> Vec<usize> {
    let n: usize = a.len();
    let mut count: Vec<usize> = vec![0; k + 1];
    let mut out: Vec<usize> = vec![0; n];

    for &x in a.iter() {
        count[x] += 1;
    }

    for i in 1..=k {
        count[i] += count[i - 1];
    }

    for idx in (0..n).rev() {
        let x: usize = a[idx];
        count[x] -= 1;
        out[count[x]] = x;
    }

    out
}
```

Complexity:

```text
Θ(n+k)
space Θ(n+k)
```

Comparison sort lower bound:

```text
Decision tree has >= n! leaves.
height >= log2(n!) = Ω(n log n).
Any comparison sort worst-case Ω(n log n).
```

---

# 4. Heaps / Priority Queues / Heapsort

## Heap under the hood

```text
Complete binary tree stored in array.

1-index:
parent(i)=floor(i/2)
left(i)=2i
right(i)=2i+1

0-index Rust:
parent(i)=(i-1)/2
left(i)=2i+1
right(i)=2i+2
```

Max-heap property:

```text
A[parent(i)] >= A[i]
root = maximum
```

## Max-Heapify

Assumption:

```text
left subtree is heap
right subtree is heap
A[i] may violate heap property
```

0-index:

```rust
fn max_heapify(a: &mut Vec<i32>, i: usize, heap_size: usize) {
    let l: usize = 2 * i + 1;
    let r: usize = 2 * i + 2;

    let mut largest: usize = i;

    if l < heap_size && a[l] > a[largest] {
        largest = l;
    }

    if r < heap_size && a[r] > a[largest] {
        largest = r;
    }

    if largest != i {
        a.swap(i, largest);
        max_heapify(a, largest, heap_size);
    }
}
```

Complexity:

```text
O(height(i)) <= O(log n)
```

Trap:

```text
heap_size != a.len() during heapsort
```

## Build Max Heap

```rust
fn build_max_heap(a: &mut Vec<i32>) {
    let n: usize = a.len();

    if n <= 1 { return; }

    for i in (0..=(n / 2)).rev() {
        max_heapify(a, i, n);
    }
}
```

More precise internal nodes:

```rust
for i in (0..(n / 2)).rev() {
    max_heapify(a, i, n);
}
```

Complexity proof:

```text
Naive: n calls * O(log n) = O(n log n)
Tight:
#nodes height h <= n/2^{h+1}
cost per node O(h)
Σ_h n/2^h * h = O(n) because Σ h/2^h = 2
=> BuildHeap Θ(n)
```

## Heapsort

```rust
fn heapsort(a: &mut Vec<i32>) {
    build_max_heap(a);

    let mut heap_size: usize = a.len();

    while heap_size > 1 {
        a.swap(0, heap_size - 1);
        heap_size -= 1;
        max_heapify(a, 0, heap_size);
    }
}
```

Complexity:

```text
Build Θ(n)
n extract-max, each O(log n)
total Θ(n log n)
space O(1)
not stable
```

## Priority Queue

Max priority queue ops:

```rust
fn heap_maximum(a: &Vec<i32>) -> i32 {
    a[0]
}

fn heap_extract_max(a: &mut Vec<i32>) -> Option<i32> {
    if a.is_empty() { return None; }

    let max_value: i32 = a[0];
    let last: i32 = a.pop().unwrap();

    if !a.is_empty() {
        a[0] = last;
        max_heapify(a, 0, a.len());
    }

    Some(max_value)
}

fn heap_increase_key(a: &mut Vec<i32>, mut i: usize, key: i32) {
    if key < a[i] { return; }

    a[i] = key;

    while i > 0 {
        let p: usize = (i - 1) / 2;

        if a[p] >= a[i] { break; }

        a.swap(i, p);
        i = p;
    }
}

fn max_heap_insert(a: &mut Vec<i32>, key: i32) {
    a.push(i32::MIN);
    let i: usize = a.len() - 1;
    heap_increase_key(a, i, key);
}
```

Rust std min-heap trick:

```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

let mut max_heap: BinaryHeap<i32> = BinaryHeap::new();
let mut min_heap: BinaryHeap<Reverse<i32>> = BinaryHeap::new();

max_heap.push(x);
let x: Option<i32> = max_heap.pop();

min_heap.push(Reverse(x));
let x: Option<Reverse<i32>> = min_heap.pop();
```

---

# 5. Divide & Conquer Algorithms

## Maximum Sum Subarray

Problem:

```text
Input A[0..n)
Output max sum contiguous nonempty subarray
```

D&C idea:

```text
best subarray is:
1. entirely left
2. entirely right
3. crosses middle
```

Crossing:

```rust
fn max_crossing(a: &Vec<i32>, l: usize, m: usize, r: usize) -> i32 {
    // a[l..r], m splits left a[l..m], right a[m..r]
    let mut left_sum: i32 = i32::MIN;
    let mut sum: i32 = 0;

    for i in (l..m).rev() {
        sum += a[i];
        left_sum = left_sum.max(sum);
    }

    let mut right_sum: i32 = i32::MIN;
    sum = 0;

    for j in m..r {
        sum += a[j];
        right_sum = right_sum.max(sum);
    }

    left_sum + right_sum
}

fn max_subarray_dc(a: &Vec<i32>, l: usize, r: usize) -> i32 {
    if r - l == 1 {
        return a[l];
    }

    let m: usize = (l + r) / 2;

    let left: i32 = max_subarray_dc(a, l, m);
    let right: i32 = max_subarray_dc(a, m, r);
    let cross: i32 = max_crossing(a, l, m, r);

    left.max(right).max(cross)
}
```

Complexity:

```text
T(n)=2T(n/2)+Θ(n)=Θ(n log n)
```

Linear Kadane also useful:

```rust
fn kadane(a: &Vec<i32>) -> i32 {
    let mut best: i32 = a[0];
    let mut cur: i32 = a[0];

    for i in 1..a.len() {
        cur = a[i].max(cur + a[i]);
        best = best.max(cur);
    }

    best
}
```

Kadane invariant:

```text
cur = best subarray ending at i
best = best subarray seen so far
```

---

## Matrix Multiplication

Naive:

```rust
fn matmul(a: &Vec<Vec<i32>>, b: &Vec<Vec<i32>>) -> Vec<Vec<i32>> {
    let n: usize = a.len();
    let mut c: Vec<Vec<i32>> = vec![vec![0; n]; n];

    for i in 0..n {
        for j in 0..n {
            for k in 0..n {
                c[i][j] += a[i][k] * b[k][j];
            }
        }
    }

    c
}
```

Complexity:

```text
Θ(n^3), space Θ(n^2)
```

D&C block formula:

```text
C11 = A11B11 + A12B21
C12 = A11B12 + A12B22
C21 = A21B11 + A22B21
C22 = A21B12 + A22B22

8 sub-multiplications size n/2:
T(n)=8T(n/2)+Θ(n^2)=Θ(n^3)
```

Strassen:

```text
7 multiplications instead of 8.

M1=(A11+A22)(B11+B22)
M2=(A21+A22)B11
M3=A11(B12-B22)
M4=A22(B21-B11)
M5=(A11+A12)B22
M6=(A21-A11)(B11+B12)
M7=(A12-A22)(B21+B22)

C11=M1+M4-M5+M7
C12=M3+M5
C21=M2+M4
C22=M1-M2+M3+M6

T(n)=7T(n/2)+Θ(n^2)=Θ(n^{log2 7})
```

Trap:

```text
Strassen is about recurrence reduction, not practical constants.
```

---

# 6. Dynamic Programming

## DP Meta-template

```text
State:
    dp[...] = optimal value for subproblem ...

Base:
    smallest subproblems

Transition:
    dp[state] = min/max over choices using smaller states

Order:
    compute dependencies first

Answer:
    dp[target]

Reconstruction:
    store choice[state], then walk backward
```

Rust-ish skeleton:

```rust
let mut dp: Vec<i32> = vec![0; n + 1];
let mut choice: Vec<usize> = vec![0; n + 1];

for state in 0..=n {
    for option in options {
        // if option valid:
        // candidate = ...
        // if candidate better:
        //     dp[state] = candidate
        //     choice[state] = option
    }
}
```

Proof:

```text
Optimal substructure:
Take first/last decision of optimal solution.
Remaining part must be optimal for subproblem; otherwise replace it.

Overlapping:
Same subproblem appears from multiple choices.

Correctness:
Induct in fill order.
When computing dp[state], all dependency states are already correct.
Transition checks all valid choices.
```

---

## Rod Cutting

Problem:

```text
length n, prices p[i] for length i
maximize revenue
```

State:

```text
r[j] = max revenue for rod length j
s[j] = first cut length used in optimal solution
```

Recurrence:

```text
r[0]=0
r[j]=max_{1<=i<=j} (p[i]+r[j-i])
```

Rust-ish:

```rust
fn rod_cut(p: &Vec<i32>, n: usize) -> (Vec<i32>, Vec<usize>) {
    // p[0] unused or p[i] = price length i
    let mut r: Vec<i32> = vec![0; n + 1];
    let mut s: Vec<usize> = vec![0; n + 1];

    for j in 1..=n {
        let mut best: i32 = i32::MIN;

        for i in 1..=j {
            let cand: i32 = p[i] + r[j - i];

            if cand > best {
                best = cand;
                s[j] = i;
            }
        }

        r[j] = best;
    }

    (r, s)
}

fn print_cuts(s: &Vec<usize>, mut n: usize) {
    while n > 0 {
        let cut: usize = s[n];
        print(cut);
        n -= cut;
    }
}
```

Runtime:

```text
states n
transition up to n
Θ(n^2)
```

---

## Matrix Chain Multiplication

Problem:

```text
Matrices A1..An
Ai has dimensions p[i-1] x p[i]
Find parenthesization minimizing scalar multiplications.
```

State:

```text
m[i][j] = min cost to multiply Ai..Aj
split[i][j] = best k
```

Recurrence:

```text
m[i][i]=0
m[i][j]=min_{i<=k<j} m[i][k]+m[k+1][j]+p[i-1]*p[k]*p[j]
```

0-index variant:

```text
matrix i has dims p[i] x p[i+1]
dp[i][j] = min cost to multiply matrices i..j inclusive
dp[i][i]=0
dp[i][j]=min_{i<=k<j} dp[i][k]+dp[k+1][j]+p[i]*p[k+1]*p[j+1]
```

Rust-ish:

```rust
fn matrix_chain(p: &Vec<usize>) -> (Vec<Vec<usize>>, Vec<Vec<usize>>) {
    let n: usize = p.len() - 1;

    let mut dp: Vec<Vec<usize>> = vec![vec![0; n]; n];
    let mut split: Vec<Vec<usize>> = vec![vec![0; n]; n];

    for len in 2..=n {
        for i in 0..=(n - len) {
            let j: usize = i + len - 1;
            dp[i][j] = usize::MAX;

            for k in i..j {
                let cost: usize =
                    dp[i][k] +
                    dp[k + 1][j] +
                    p[i] * p[k + 1] * p[j + 1];

                if cost < dp[i][j] {
                    dp[i][j] = cost;
                    split[i][j] = k;
                }
            }
        }
    }

    (dp, split)
}
```

Order:

```text
increasing chain length
len = 2..n
```

Runtime:

```text
states Θ(n^2)
transition Θ(n)
Θ(n^3)
```

---

## Longest Common Subsequence

Problem:

```text
A[0..n), B[0..m)
not necessarily contiguous
preserve order
```

State:

```text
dp[i][j] = LCS length of A[0..i), B[0..j)
```

Base:

```text
dp[0][j]=0
dp[i][0]=0
```

Recurrence:

```text
if A[i-1] == B[j-1]:
    dp[i][j] = 1 + dp[i-1][j-1]
else:
    dp[i][j] = max(dp[i-1][j], dp[i][j-1])
```

Rust-ish:

```rust
fn lcs(a: &Vec<char>, b: &Vec<char>) -> usize {
    let n: usize = a.len();
    let m: usize = b.len();

    let mut dp: Vec<Vec<usize>> = vec![vec![0; m + 1]; n + 1];

    for i in 1..=n {
        for j in 1..=m {
            if a[i - 1] == b[j - 1] {
                dp[i][j] = 1 + dp[i - 1][j - 1];
            } else {
                dp[i][j] = dp[i - 1][j].max(dp[i][j - 1]);
            }
        }
    }

    dp[n][m]
}
```

Reconstruct:

```rust
fn reconstruct_lcs(a: &Vec<char>, b: &Vec<char>, dp: &Vec<Vec<usize>>) -> Vec<char> {
    let mut i: usize = a.len();
    let mut j: usize = b.len();
    let mut ans: Vec<char> = Vec::new();

    while i > 0 && j > 0 {
        if a[i - 1] == b[j - 1] {
            ans.push(a[i - 1]);
            i -= 1;
            j -= 1;
        } else if dp[i - 1][j] >= dp[i][j - 1] {
            i -= 1;
        } else {
            j -= 1;
        }
    }

    ans.reverse();
    ans
}
```

Memory optimized:

```rust
let mut prev: Vec<usize> = vec![0; m + 1];
let mut cur: Vec<usize> = vec![0; m + 1];

for i in 1..=n {
    for j in 1..=m {
        if a[i - 1] == b[j - 1] {
            cur[j] = 1 + prev[j - 1];
        } else {
            cur[j] = prev[j].max(cur[j - 1]);
        }
    }
    std::mem::swap(&mut prev, &mut cur);
    cur.fill(0);
}
```

Runtime:

```text
Θ(nm), space Θ(nm) or Θ(m)
```

Trap:

```text
equal -> diagonal + 1
not equal -> max(up, left)
```

---

## Knapsack 0/1

Problem:

```text
items i with weight w[i], value val[i]
capacity W
take each item at most once
```

State:

```text
dp[i][cap] = best value using first i items with capacity cap
```

Recurrence:

```text
not take: dp[i-1][cap]
take: val[i-1] + dp[i-1][cap - w[i-1]]
```

Rust-ish 2D:

```rust
fn knapsack(weights: &Vec<usize>, values: &Vec<i32>, w_cap: usize) -> i32 {
    let n: usize = weights.len();
    let mut dp: Vec<Vec<i32>> = vec![vec![0; w_cap + 1]; n + 1];

    for i in 1..=n {
        for cap in 0..=w_cap {
            dp[i][cap] = dp[i - 1][cap];

            let wi: usize = weights[i - 1];
            let vi: i32 = values[i - 1];

            if wi <= cap {
                dp[i][cap] = dp[i][cap].max(vi + dp[i - 1][cap - wi]);
            }
        }
    }

    dp[n][w_cap]
}
```

1D optimized:

```rust
let mut dp: Vec<i32> = vec![0; w_cap + 1];

for i in 0..n {
    let wi: usize = weights[i];
    let vi: i32 = values[i];

    for cap in (wi..=w_cap).rev() {
        dp[cap] = dp[cap].max(vi + dp[cap - wi]);
    }
}
```

Trap:

```text
0/1 knapsack: cap loop descending
unbounded knapsack: cap loop ascending
```

Runtime:

```text
Θ(nW)
pseudo-polynomial
```

---

## Coin Change / Change-Making

Problem:

```text
coins denominations, amount W
minimum #coins to make W
usually unlimited coins
```

State:

```text
dp[x] = min coins for amount x
```

Base:

```text
dp[0]=0
dp[x]=INF initially
```

Recurrence:

```text
dp[x] = min_{coin<=x} 1 + dp[x-coin]
```

Rust-ish:

```rust
fn coin_change(coins: &Vec<usize>, amount: usize) -> Option<usize> {
    let inf: usize = amount + 1;
    let mut dp: Vec<usize> = vec![inf; amount + 1];

    dp[0] = 0;

    for x in 1..=amount {
        for &c in coins.iter() {
            if c <= x {
                dp[x] = dp[x].min(1 + dp[x - c]);
            }
        }
    }

    if dp[amount] == inf {
        None
    } else {
        Some(dp[amount])
    }
}
```

Runtime:

```text
Θ(amount * #coins)
```

---

## Longest Increasing Subsequence

DP O(n²):

```text
dp[i] = LIS length starting/ending at i
```

Ending at i:

```text
dp[i] = 1 + max dp[j] where j<i and a[j]<a[i]
base dp[i]=1
ans=max dp[i]
```

```rust
fn lis_n2(a: &Vec<i32>) -> usize {
    let n: usize = a.len();
    let mut dp: Vec<usize> = vec![1; n];

    for i in 0..n {
        for j in 0..i {
            if a[j] < a[i] {
                dp[i] = dp[i].max(1 + dp[j]);
            }
        }
    }

    *dp.iter().max().unwrap()
}
```

O(n log n) tails:

```text
tails[len-1] = minimum possible tail value of increasing subsequence length len
for x:
    find first tails[pos] >= x
    tails[pos]=x
answer = tails.len()
```

```rust
fn lis_nlogn(a: &Vec<i32>) -> usize {
    let mut tails: Vec<i32> = Vec::new();

    for &x in a.iter() {
        let mut l: usize = 0;
        let mut r: usize = tails.len();

        while l < r {
            let m: usize = (l + r) / 2;

            if tails[m] < x {
                l = m + 1;
            } else {
                r = m;
            }
        }

        if l == tails.len() {
            tails.push(x);
        } else {
            tails[l] = x;
        }
    }

    tails.len()
}
```

---

## DAG Longest Path

Use:

```text
DAG -> topological order
Relax edges in topo order
```

```rust
let mut dist: Vec<i32> = vec![i32::MIN / 4; n];
dist[source] = 0;

for &u in topo.iter() {
    for &(v, w) in adj[u].iter() {
        if dist[u] + w > dist[v] {
            dist[v] = dist[u] + w;
        }
    }
}
```

Runtime:

```text
Θ(V+E)
```

---

# 7. Graph Representation

Adjacency list:

```rust
let n: usize = ...;
let mut adj: Vec<Vec<usize>> = vec![Vec::new(); n];

for (u, v) in edges {
    adj[u].push(v);
    // if undirected:
    adj[v].push(u);
}
```

Weighted:

```rust
let mut adj: Vec<Vec<(usize, i32)>> = vec![Vec::new(); n];

for (u, v, w) in edges {
    adj[u].push((v, w));
}
```

Matrix:

```rust
let mut mat: Vec<Vec<bool>> = vec![vec![false; n]; n];
mat[u][v] = true;
```

Complexities:

```text
Adj list:
space Θ(V+E)
iterate neighbors Θ(deg(u))
edge lookup O(deg(u))

Adj matrix:
space Θ(V^2)
iterate neighbors Θ(V)
edge lookup Θ(1)
```

---

# 8. BFS

Use:

```text
unweighted shortest paths
levels
connected components
bipartite check
```

Data structures:

```rust
use std::collections::VecDeque;

let mut q: VecDeque<usize> = VecDeque::new();
let mut color: Vec<i32> = vec![0; n]; // 0 white, 1 gray, 2 black
let mut dist: Vec<i32> = vec![i32::MAX; n];
let mut parent: Vec<Option<usize>> = vec![None; n];
```

BFS:

```rust
fn bfs(adj: &Vec<Vec<usize>>, s: usize) -> (Vec<i32>, Vec<Option<usize>>) {
    use std::collections::VecDeque;

    let n: usize = adj.len();
    let mut dist: Vec<i32> = vec![i32::MAX; n];
    let mut parent: Vec<Option<usize>> = vec![None; n];
    let mut q: VecDeque<usize> = VecDeque::new();

    dist[s] = 0;
    q.push_back(s);

    while let Some(u) = q.pop_front() {
        for &v in adj[u].iter() {
            if dist[v] == i32::MAX {
                dist[v] = dist[u] + 1;
                parent[v] = Some(u);
                q.push_back(v);
            }
        }
    }

    (dist, parent)
}
```

Invariant:

```text
Queue contains frontier.
Vertices are discovered in nondecreasing distance from s.
First time discovered => shortest unweighted path.
```

Runtime:

```text
Θ(V+E)
```

Bipartite:

```rust
fn is_bipartite(adj: &Vec<Vec<usize>>) -> bool {
    use std::collections::VecDeque;

    let n: usize = adj.len();
    let mut color: Vec<i32> = vec![-1; n];

    for s in 0..n {
        if color[s] != -1 { continue; }

        let mut q: VecDeque<usize> = VecDeque::new();
        color[s] = 0;
        q.push_back(s);

        while let Some(u) = q.pop_front() {
            for &v in adj[u].iter() {
                if color[v] == -1 {
                    color[v] = 1 - color[u];
                    q.push_back(v);
                } else if color[v] == color[u] {
                    return false;
                }
            }
        }
    }

    true
}
```

---

# 9. DFS / Edge Types / Topological Sort / SCC

Data:

```rust
let mut color: Vec<i32> = vec![0; n]; // 0 W, 1 G, 2 B
let mut parent: Vec<Option<usize>> = vec![None; n];
let mut discover: Vec<usize> = vec![0; n];
let mut finish: Vec<usize> = vec![0; n];
let mut time: usize = 0;
```

DFS:

```rust
fn dfs_visit(
    u: usize,
    adj: &Vec<Vec<usize>>,
    color: &mut Vec<i32>,
    parent: &mut Vec<Option<usize>>,
    discover: &mut Vec<usize>,
    finish: &mut Vec<usize>,
    time: &mut usize,
) {
    *time += 1;
    discover[u] = *time;
    color[u] = 1; // gray

    for &v in adj[u].iter() {
        if color[v] == 0 {
            parent[v] = Some(u);
            dfs_visit(v, adj, color, parent, discover, finish, time);
        }
    }

    color[u] = 2; // black
    *time += 1;
    finish[u] = *time;
}

fn dfs(adj: &Vec<Vec<usize>>) {
    let n: usize = adj.len();
    let mut color: Vec<i32> = vec![0; n];
    let mut parent: Vec<Option<usize>> = vec![None; n];
    let mut discover: Vec<usize> = vec![0; n];
    let mut finish: Vec<usize> = vec![0; n];
    let mut time: usize = 0;

    for u in 0..n {
        if color[u] == 0 {
            dfs_visit(u, adj, &mut color, &mut parent, &mut discover, &mut finish, &mut time);
        }
    }
}
```

Edge classification directed:

```text
Tree:    v white when exploring (u,v)
Back:    v gray ancestor of u              -> cycle
Forward: v black descendant, non-tree
Cross:   other black edge
```

Undirected DFS:

```text
only tree/back edges
cycle iff see visited neighbor != parent
```

Parenthesis theorem:

```text
Intervals [d[u], f[u]] and [d[v], f[v]]:
- disjoint
- one contains the other
descendant iff interval contained
```

White path theorem:

```text
v descendant of u iff at time d[u] exists path u -> v of white vertices.
```

---

## Topological Sort

Use:

```text
DAG ordering so every edge u->v has u before v.
```

DFS version:

```rust
fn topo_dfs(adj: &Vec<Vec<usize>>) -> Vec<usize> {
    fn visit(u: usize, adj: &Vec<Vec<usize>>, seen: &mut Vec<bool>, order: &mut Vec<usize>) {
        seen[u] = true;

        for &v in adj[u].iter() {
            if !seen[v] {
                visit(v, adj, seen, order);
            }
        }

        order.push(u);
    }

    let n: usize = adj.len();
    let mut seen: Vec<bool> = vec![false; n];
    let mut order: Vec<usize> = Vec::new();

    for u in 0..n {
        if !seen[u] {
            visit(u, adj, &mut seen, &mut order);
        }
    }

    order.reverse();
    order
}
```

Kahn indegree:

```rust
fn topo_kahn(adj: &Vec<Vec<usize>>) -> Option<Vec<usize>> {
    use std::collections::VecDeque;

    let n: usize = adj.len();
    let mut indeg: Vec<usize> = vec![0; n];

    for u in 0..n {
        for &v in adj[u].iter() {
            indeg[v] += 1;
        }
    }

    let mut q: VecDeque<usize> = VecDeque::new();

    for u in 0..n {
        if indeg[u] == 0 {
            q.push_back(u);
        }
    }

    let mut order: Vec<usize> = Vec::new();

    while let Some(u) = q.pop_front() {
        order.push(u);

        for &v in adj[u].iter() {
            indeg[v] -= 1;

            if indeg[v] == 0 {
                q.push_back(v);
            }
        }
    }

    if order.len() == n {
        Some(order)
    } else {
        None // cycle
    }
}
```

Cycle relation:

```text
Directed graph acyclic iff DFS has no back edges.
Topo sort = decreasing finish times.
```

---

## Strongly Connected Components

Definition:

```text
u,v in same SCC iff u reaches v and v reaches u.
Condensation graph of SCCs is a DAG.
```

Kosaraju:

```rust
fn kosaraju(adj: &Vec<Vec<usize>>) -> Vec<Vec<usize>> {
    let n: usize = adj.len();

    let mut order: Vec<usize> = topo_dfs_finish_order(adj); // increasing push on finish

    let mut rev: Vec<Vec<usize>> = vec![Vec::new(); n];
    for u in 0..n {
        for &v in adj[u].iter() {
            rev[v].push(u);
        }
    }

    order.reverse(); // decreasing finish time

    let mut seen: Vec<bool> = vec![false; n];
    let mut comps: Vec<Vec<usize>> = Vec::new();

    for &s in order.iter() {
        if seen[s] { continue; }

        let mut comp: Vec<usize> = Vec::new();
        let mut stack: Vec<usize> = vec![s];
        seen[s] = true;

        while let Some(u) = stack.pop() {
            comp.push(u);

            for &v in rev[u].iter() {
                if !seen[v] {
                    seen[v] = true;
                    stack.push(v);
                }
            }
        }

        comps.push(comp);
    }

    comps
}
```

Runtime:

```text
Θ(V+E)
```

---

# 10. Union-Find / Disjoint Sets

Under hood:

```text
Forest of trees.
parent[x] points upward.
root is representative.
rank[x] upper bound on height.
```

Data:

```rust
struct DSU {
    parent: Vec<usize>,
    rank: Vec<usize>,
}
```

Implementation:

```rust
impl DSU {
    fn new(n: usize) -> Self {
        let mut parent: Vec<usize> = Vec::new();

        for i in 0..n {
            parent.push(i);
        }

        DSU {
            parent,
            rank: vec![0; n],
        }
    }

    fn find(&mut self, x: usize) -> usize {
        if self.parent[x] != x {
            let root: usize = self.find(self.parent[x]);
            self.parent[x] = root; // path compression
        }

        self.parent[x]
    }

    fn union(&mut self, x: usize, y: usize) -> bool {
        let mut rx: usize = self.find(x);
        let mut ry: usize = self.find(y);

        if rx == ry {
            return false;
        }

        if self.rank[rx] < self.rank[ry] {
            std::mem::swap(&mut rx, &mut ry);
        }

        self.parent[ry] = rx;

        if self.rank[rx] == self.rank[ry] {
            self.rank[rx] += 1;
        }

        true
    }
}
```

Complexities:

```text
linked list weighted union: O(m + n log n)
forest + union by rank + path compression: O(m α(n))
α(n) <= 5 practically
```

Use cases:

```text
connected components
cycle detection undirected graph
Kruskal MST
```

Connected components:

```rust
let mut dsu: DSU = DSU::new(n);

for (u, v) in edges {
    dsu.union(u, v);
}

if dsu.find(a) == dsu.find(b) {
    // same component
}
```

Cycle detection undirected:

```rust
let mut dsu: DSU = DSU::new(n);

for (u, v) in edges {
    if !dsu.union(u, v) {
        // u and v already connected -> adding edge creates cycle
    }
}
```

---

# 11. Minimum Spanning Trees

Definitions:

```text
Spanning tree:
- connects all vertices
- acyclic
- exactly |V|-1 edges if connected

MST:
spanning tree with min total edge weight
```

## Cut / Cycle Properties

Cut:

```text
Cut (S, V-S).
Edge crosses cut if one endpoint in S, other outside.
Light edge = minimum-weight crossing edge.
Cut property:
if cut respects A, then light edge crossing cut is safe for A.
```

Proof sketch cut:

```text
Let T be MST not containing light edge e=(u,v).
Add e to T -> creates cycle.
Cycle has another edge f crossing same cut.
w(e) <= w(f).
T' = T - f + e is spanning tree.
w(T') <= w(T).
Therefore exists MST containing e.
```

Cycle property:

```text
For any cycle, heaviest edge on cycle is not in some MST.
If weights distinct: heaviest edge is in no MST.
```

Proof sketch cycle:

```text
Assume MST T contains heaviest edge e on cycle.
Remove e -> cut separates tree into two parts.
Cycle has another edge f crossing cut.
w(f) < w(e).
T' = T - e + f cheaper -> contradiction.
```

---

## Kruskal

Data structures:

```rust
struct Edge {
    u: usize,
    v: usize,
    w: i32,
}

let mut edges: Vec<Edge> = ...;
let mut dsu: DSU = DSU::new(n);
let mut mst: Vec<Edge> = Vec::new();
let mut total: i32 = 0;
```

Algorithm:

```rust
fn kruskal(n: usize, mut edges: Vec<Edge>) -> (i32, Vec<Edge>) {
    edges.sort_by(|a, b| a.w.cmp(&b.w));

    let mut dsu: DSU = DSU::new(n);
    let mut total: i32 = 0;
    let mut mst: Vec<Edge> = Vec::new();

    for e in edges {
        if dsu.union(e.u, e.v) {
            total += e.w;
            mst.push(e);

            if mst.len() == n - 1 {
                break;
            }
        }
    }

    (total, mst)
}
```

Complexity:

```text
sort edges O(E log E)
DSU O(E α(V))
total O(E log E)
```

Correctness:

```text
At each step, components define a cut.
Kruskal picks lightest edge connecting two components.
By cut property edge is safe.
Induct until |V|-1 edges.
```

Traps:

```text
- If graph disconnected -> MST does not exist; returns forest.
- Need sort by weight increasing.
- Skip edge if endpoints already connected.
```

---

## Prim

Use:

```text
Grow one tree.
At each step choose min-weight edge crossing cut (tree, outside).
```

Data:

```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

let mut adj: Vec<Vec<(usize, i32)>> = vec![Vec::new(); n];
let mut used: Vec<bool> = vec![false; n];
let mut heap: BinaryHeap<Reverse<(i32, usize, usize)>> = BinaryHeap::new();
// (weight, vertex, parent)
```

Lazy Prim:

```rust
fn prim(adj: &Vec<Vec<(usize, i32)>>, start: usize) -> i32 {
    use std::cmp::Reverse;
    use std::collections::BinaryHeap;

    let n: usize = adj.len();
    let mut used: Vec<bool> = vec![false; n];
    let mut heap: BinaryHeap<Reverse<(i32, usize)>> = BinaryHeap::new();

    heap.push(Reverse((0, start)));

    let mut total: i32 = 0;
    let mut count: usize = 0;

    while let Some(Reverse((w, u))) = heap.pop() {
        if used[u] { continue; }

        used[u] = true;
        total += w;
        count += 1;

        for &(v, weight) in adj[u].iter() {
            if !used[v] {
                heap.push(Reverse((weight, v)));
            }
        }
    }

    if count != n {
        // disconnected
    }

    total
}
```

Complexity:

```text
Binary heap lazy: O(E log E) = O(E log V)
```

Correctness:

```text
At each step used vertices define cut.
Chosen edge is lightest crossing cut.
By cut property safe.
```

---

# 12. Shortest Paths

## Relaxation

```text
if d[v] > d[u] + w(u,v):
    d[v] = d[u] + w(u,v)
    parent[v] = u
```

Rust:

```rust
fn relax(
    u: usize,
    v: usize,
    w: i32,
    dist: &mut Vec<i32>,
    parent: &mut Vec<Option<usize>>,
) {
    if dist[u] != i32::MAX / 4 && dist[v] > dist[u] + w {
        dist[v] = dist[u] + w;
        parent[v] = Some(u);
    }
}
```

Use `INF = i32::MAX / 4` to avoid overflow.

---

## Dijkstra

Use:

```text
single-source shortest paths
requires all edge weights >= 0
```

Data:

```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

let mut dist: Vec<i32> = vec![INF; n];
let mut parent: Vec<Option<usize>> = vec![None; n];
let mut heap: BinaryHeap<Reverse<(i32, usize)>> = BinaryHeap::new();
```

Algorithm:

```rust
fn dijkstra(adj: &Vec<Vec<(usize, i32)>>, s: usize) -> Vec<i32> {
    use std::cmp::Reverse;
    use std::collections::BinaryHeap;

    let n: usize = adj.len();
    let inf: i32 = i32::MAX / 4;

    let mut dist: Vec<i32> = vec![inf; n];
    let mut heap: BinaryHeap<Reverse<(i32, usize)>> = BinaryHeap::new();

    dist[s] = 0;
    heap.push(Reverse((0, s)));

    while let Some(Reverse((du, u))) = heap.pop() {
        if du != dist[u] {
            continue; // stale heap entry
        }

        for &(v, w) in adj[u].iter() {
            if dist[v] > dist[u] + w {
                dist[v] = dist[u] + w;
                heap.push(Reverse((dist[v], v)));
            }
        }
    }

    dist
}
```

Invariant:

```text
When u extracted with minimum dist[u], dist[u] is final.
Because all edges nonnegative, no future path through unvisited vertices can improve it.
```

Complexity:

```text
O((V+E) log V) with binary heap
often O(E log V)
```

Trap:

```text
Negative edge breaks invariant.
Example: s->a cost 2, s->b cost 5, b->a cost -10.
Dijkstra finalizes a too early.
```

---

## Bellman-Ford

Use:

```text
negative edges allowed
detect negative cycles reachable from source
```

Data:

```rust
struct Edge {
    u: usize,
    v: usize,
    w: i32,
}

let mut dist: Vec<i32> = vec![INF; n];
let mut parent: Vec<Option<usize>> = vec![None; n];
```

Algorithm:

```rust
fn bellman_ford(n: usize, edges: &Vec<Edge>, s: usize) -> Option<Vec<i32>> {
    let inf: i32 = i32::MAX / 4;
    let mut dist: Vec<i32> = vec![inf; n];

    dist[s] = 0;

    for _ in 0..(n - 1) {
        let mut changed: bool = false;

        for e in edges.iter() {
            if dist[e.u] != inf && dist[e.v] > dist[e.u] + e.w {
                dist[e.v] = dist[e.u] + e.w;
                changed = true;
            }
        }

        if !changed {
            break;
        }
    }

    for e in edges.iter() {
        if dist[e.u] != inf && dist[e.v] > dist[e.u] + e.w {
            return None; // negative cycle reachable
        }
    }

    Some(dist)
}
```

Correctness invariant:

```text
After i full passes, dist[v] <= length of shortest s->v path using at most i edges.
Any simple shortest path has at most |V|-1 edges if no negative cycle.
After |V|-1 passes all shortest paths found.
If still relaxable on pass |V| -> negative cycle reachable.
```

Complexity:

```text
Θ(VE)
```

---

## Shortest Paths in DAG

Use:

```text
works with negative weights if DAG
topological order
one relaxation per edge
```

```rust
fn dag_shortest(adj: &Vec<Vec<(usize, i32)>>, s: usize) -> Vec<i32> {
    let n: usize = adj.len();
    let topo: Vec<usize> = topo_dfs_weighted(adj);
    let inf: i32 = i32::MAX / 4;

    let mut dist: Vec<i32> = vec![inf; n];
    dist[s] = 0;

    for &u in topo.iter() {
        if dist[u] == inf { continue; }

        for &(v, w) in adj[u].iter() {
            if dist[v] > dist[u] + w {
                dist[v] = dist[u] + w;
            }
        }
    }

    dist
}
```

Complexity:

```text
Θ(V+E)
```

---

# 13. Flow Networks

Definitions:

```text
Directed graph G=(V,E)
source s, sink t
capacity c(u,v) >= 0
flow f(u,v)
capacity constraint: 0 <= f(u,v) <= c(u,v)
flow conservation: for u != s,t, inflow = outflow
value |f| = outflow(s) - inflow(s)
```

Residual capacity:

```text
forward residual: c_f(u,v)=c(u,v)-f(u,v)
backward residual: c_f(v,u)=f(u,v)
```

Augmenting path:

```text
path s->t in residual graph with all residual capacities > 0
bottleneck = min residual capacity along path
augment by bottleneck
```

---

## Ford-Fulkerson / Edmonds-Karp

Data:

```rust
let mut cap: Vec<Vec<i32>> = vec![vec![0; n]; n];
let mut adj: Vec<Vec<usize>> = vec![Vec::new(); n];
// for every original edge u->v:
// cap[u][v] += c
// adj[u].push(v); adj[v].push(u); // include reverse in residual adjacency
```

BFS augmenting path:

```rust
fn bfs_path(
    s: usize,
    t: usize,
    adj: &Vec<Vec<usize>>,
    cap: &Vec<Vec<i32>>,
    parent: &mut Vec<Option<usize>>,
) -> i32 {
    use std::collections::VecDeque;

    let n: usize = adj.len();
    parent.fill(None);

    let mut q: VecDeque<(usize, i32)> = VecDeque::new();
    q.push_back((s, i32::MAX));
    parent[s] = Some(s);

    while let Some((u, flow)) = q.pop_front() {
        for &v in adj[u].iter() {
            if parent[v].is_none() && cap[u][v] > 0 {
                parent[v] = Some(u);
                let new_flow: i32 = flow.min(cap[u][v]);

                if v == t {
                    return new_flow;
                }

                q.push_back((v, new_flow));
            }
        }
    }

    0
}
```

Max flow:

```rust
fn max_flow(
    s: usize,
    t: usize,
    adj: &Vec<Vec<usize>>,
    cap: &mut Vec<Vec<i32>>,
) -> i32 {
    let n: usize = adj.len();
    let mut parent: Vec<Option<usize>> = vec![None; n];
    let mut flow: i32 = 0;

    loop {
        let pushed: i32 = bfs_path(s, t, adj, cap, &mut parent);

        if pushed == 0 {
            break;
        }

        flow += pushed;

        let mut v: usize = t;
        while v != s {
            let u: usize = parent[v].unwrap();
            cap[u][v] -= pushed;
            cap[v][u] += pushed;
            v = u;
        }
    }

    flow
}
```

Complexity:

```text
Ford-Fulkerson integer capacities: O(E * |f*|)
Edmonds-Karp with BFS: O(V E^2)
```

Correctness:

```text
If no augmenting path exists, let S = vertices reachable from s in residual graph.
No residual edge from S to T.
Flow across cut saturated, reverse zero enough.
|f| = capacity(S,T).
Thus max flow = min cut.
```

---

## Max-Flow Min-Cut

Statement:

```text
Maximum s-t flow value = minimum s-t cut capacity.
```

Cut capacity:

```text
c(S,T) = sum c(u,v) for u in S, v in T
```

Use:

```text
To prove optimal flow:
show flow value F
show cut capacity F
=> flow is maximum, cut is minimum
```

---

## Bipartite Matching via Max Flow

Graph:

```text
Left L, Right R
source s -> each l in L capacity 1
each original edge l-r capacity 1
each r in R -> sink t capacity 1
```

Then:

```text
max flow value = maximum matching size
edge l->r with flow 1 = matched pair
```

Rust-ish construction:

```rust
let s: usize = 0;
let left_offset: usize = 1;
let right_offset: usize = 1 + n_left;
let t: usize = 1 + n_left + n_right;
let n: usize = t + 1;

let mut cap: Vec<Vec<i32>> = vec![vec![0; n]; n];
let mut adj: Vec<Vec<usize>> = vec![Vec::new(); n];

fn add_edge(u: usize, v: usize, c: i32, adj: &mut Vec<Vec<usize>>, cap: &mut Vec<Vec<i32>>) {
    adj[u].push(v);
    adj[v].push(u);
    cap[u][v] += c;
}

for l in 0..n_left {
    add_edge(s, left_offset + l, 1, &mut adj, &mut cap);
}

for (l, r) in bip_edges {
    add_edge(left_offset + l, right_offset + r, 1, &mut adj, &mut cap);
}

for r in 0..n_right {
    add_edge(right_offset + r, t, 1, &mut adj, &mut cap);
}
```

Proof:

```text
Capacity 1 forces each left/right vertex used at most once.
Integral capacities -> integral max flow.
Integral flow corresponds exactly to matching.
```

---

## Edge-Disjoint Paths via Max Flow

Problem:

```text
maximum number of edge-disjoint s-t paths
```

Construction:

```text
give every edge capacity 1
max flow value = max # edge-disjoint s-t paths
```

Reason:

```text
unit capacity -> each edge used by at most one path
integral max flow decomposes into paths
```

Vertex-disjoint variant:

```text
split each vertex v into v_in -> v_out capacity 1
replace edge u->v by u_out -> v_in capacity INF
```

---

# 14. Hash Tables

Already above, but algorithmic points:

Universal hashing:

```text
Choose hash function randomly from family H.
For x != y:
P[h(x)=h(y)] <= 1/m
```

Chaining expected search:

```text
α = n/m
successful search expected O(1+α)
unsuccessful expected O(1+α)
if α=O(1), operations expected O(1)
```

Common proof:

```text
Let X_y = indicator that key y collides with x.
E[#collisions] = Σ_y E[X_y] = Σ_y P[h(y)=h(x)] <= n/m = α.
```

---

# 15. Randomized / Probabilistic Analysis

Indicator variable:

```text
X_A = 1 if event A occurs, 0 otherwise
E[X_A] = P(A)
```

Linearity:

```text
E[X1 + ... + Xn] = E[X1]+...+E[Xn]
No independence needed.
```

Hiring problem:

```text
Candidates random order.
Hire candidate i if best among first i.
X_i = 1 if candidate i is hired.
P(X_i=1)=1/i.
E[#hires]=Σ_{i=1..n} 1/i = H_n = Θ(log n).
```

Hat-check / fixed points:

```text
n people randomly receive hats.
X_i = 1 if person i gets own hat.
P(X_i=1)=1/n.
E[#fixed points]=Σ 1/n = 1.
```

Randomized quicksort:

```text
Random pivot avoids input-dependent worst case.
Expected Θ(n log n).
Worst still Θ(n^2).
```

---

# 16. Online Algorithms / Competitive Analysis / Experts

Online:

```text
Input arrives over time.
Must decide before future known.
Compare ALG vs OPT that knows future.
```

Competitive ratio minimization:

```text
ALG(I) <= r * OPT(I) + c
r-competitive
```

Ski rental:

```text
rent cost 1/day
buy cost B
unknown T days
OPT(T)=min(T,B)

Algorithm:
rent until paid B, then buy
```

Cost:

```text
if T <= B:
    ALG=T=OPT
if T > B:
    ALG <= B + B = 2B = 2OPT
=> 2-competitive
```

Lower bound deterministic:

```text
Any deterministic algorithm buys day x.
Adversary stops just when bad:
cannot beat 2 in worst case.
```

Weighted majority / experts:

```text
N experts predict.
weights initially 1.
Predict weighted majority.
After outcome, multiply weights of wrong experts by β < 1.
Goal: mistakes close to best expert.
```

Regret:

```text
loss(ALG) <= loss(best expert) + small
```

Secretary problem:

```text
random order, choose one irrevocably.
Classic strategy:
observe first n/e candidates, then take next candidate better than all seen.
Success probability ~ 1/e.
```

Prophet inequality:

```text
online threshold vs prophet who sees all values.
Known distributions.
```

---

# 17. Algorithm Table

```text
Algorithm/Data Struct        Time                         Space         Notes
Array access                 O(1)                         -             contiguous
Vec push                     amortized O(1)               -             resize doubles
Linked list search           O(n)                         O(n)          bad locality
Stack push/pop               O(1)                         O(n)          Vec
Queue push/pop               O(1)                         O(n)          VecDeque
Hash search/insert/delete    expected O(1), worst O(n)    O(n)          α=n/m
BST ops                      O(h)                         O(n)          h log n balanced, n worst
Inorder BST                  Θ(n)                         O(h)          sorted output
Insertion sort               best Θ(n), worst Θ(n²)       O(1)          stable
Merge sort                   Θ(n log n)                   Θ(n)          stable
Quicksort randomized         expected Θ(n log n)          O(log n)      worst Θ(n²)
Counting sort                Θ(n+k)                       Θ(n+k)        integer keys
Heapify                      O(log n)                     recursion     assumes children heaps
Build heap                   Θ(n)                         O(1)          bottom-up
Heapsort                     Θ(n log n)                   O(1)          not stable
PQ insert/extract            O(log n)                     O(n)          heap
BFS                          Θ(V+E)                       Θ(V)          unweighted shortest paths
DFS                          Θ(V+E)                       Θ(V)          topo/cycles/SCC
Topological sort             Θ(V+E)                       Θ(V)          DAG only
Kosaraju SCC                 Θ(V+E)                       Θ(V+E)        2 DFS
Union-Find op                O(α(n))                      O(n)          rank+compression
Kruskal                      O(E log E)                   O(V)          sort + DSU
Prim                         O(E log V)                   O(V+E)        PQ
Dijkstra                     O(E log V)                   O(V+E)        no negative edges
Bellman-Ford                 Θ(VE)                        O(V)          negative edges OK
DAG shortest path            Θ(V+E)                       O(V)          topo order
Ford-Fulkerson               O(E|f*|)                     O(V²/E)       integer caps
Edmonds-Karp                 O(VE²)                       O(V²)         BFS paths
Rod cutting                  Θ(n²)                        Θ(n)          DP
Matrix chain                 Θ(n³)                        Θ(n²)         DP
LCS                          Θ(nm)                        Θ(nm)         DP
Knapsack 0/1                 Θ(nW)                        Θ(nW)/Θ(W)    pseudo-poly
Coin change                  Θ(W*#coins)                  Θ(W)          unbounded
```

---

# 18. Common Proof Skeletons

## Insertion Sort Correctness

```text
Invariant:
At start of iteration j, A[0..j) sorted and contains original prefix.

Init:
j=1, one element sorted.

Maintenance:
while loop inserts A[j] into correct place, shifting larger elements.
Then A[0..j+1) sorted.

Termination:
j=n, A[0..n) sorted.
```

## Merge Correctness

```text
Invariant:
tmp contains smallest elements of two input prefixes in sorted order.
i,j point to first unmerged items.
Choose smaller head each step.
When one side empty, append rest.
```

## BFS Shortest Path

```text
Invariant:
When vertex v discovered, dist[v] equals shortest path length from s to v.
Queue processed in nondecreasing dist.
For edge u->v, if v undiscovered, dist[v]=dist[u]+1.
No shorter path exists because all smaller-distance vertices processed earlier.
```

## DFS Topo

```text
For edge u->v in DAG:
DFS finish[u] > finish[v].
If v discovered from u, v finishes before u.
If v already black, also finish[v] < finish[u].
No back edge in DAG.
Therefore decreasing finish time is topological order.
```

## Dijkstra

```text
Let S = finalized vertices.
Invariant: for u in S, dist[u] is true shortest distance.
Pick x outside S with min dist.
Any alternative path to x must first cross from S to outside at edge (u,v).
Since weights >=0:
dist[x] <= dist[v] <= dist[u]+w(u,v) <= alternative path.
So dist[x] final.
```

## Bellman-Ford

```text
After i passes:
dist[v] <= shortest path from s to v using at most i edges.
Induct on i.
Shortest simple path has at most V-1 edges.
If improvement on pass V, then shorter walk uses >=V edges -> contains cycle.
If improvement reduces distance, cycle is negative.
```

## Kruskal

```text
Before each step A subset of some MST.
Kruskal chooses lightest edge e crossing cut between two DSU components.
Cut respects A.
By cut property e is safe.
Add e; invariant preserved.
At |V|-1 edges, A is MST.
```

## Prim

```text
S = vertices already in tree.
Choose lightest edge crossing (S,V-S).
By cut property safe.
Add vertex.
Repeat until all vertices included.
```

## Ford-Fulkerson

```text
Augmenting preserves capacity and conservation.
If no augmenting path:
S = vertices reachable from s in residual graph.
No residual edge S->T.
All original S->T edges saturated.
All T->S edges have zero flow.
Thus |f| = c(S,T).
Since any flow <= any cut, f max and cut min.
```

## DP Correctness

```text
Induct over fill order.
Base states correct.
For state x:
transition enumerates all possible first/last decisions.
Each candidate uses already-correct smaller states.
Optimal solution chooses one of these decisions.
Therefore dp[x] equals optimum.
```

---

# 19. Common Exam Traps

```text
HashMap O(1) is expected, worst O(n).
BuildHeap is Θ(n), not Θ(n log n).
Dijkstra fails with negative edges.
Bellman-Ford detects reachable negative cycles only.
BFS shortest path only for unweighted/equal weights.
DFS topo works only on DAG.
Kruskal skips edges inside same component.
Prim needs cheapest crossing edge, not cheapest global unused edge.
Max flow needs reverse residual edges.
Bipartite matching needs capacities 1.
0/1 knapsack 1D loops capacity descending.
Unbounded coin change loops amount ascending.
LCS equal case uses diagonal +1, not max(up,left)+1.
Matrix-chain loop by chain length, not i then j arbitrary.
Quicksort worst Θ(n²) even if average good.
Counting sort needs bounded integer keys.
BST operations O(h), not always O(log n).
```

---

# 20. Minimal Rust-ish Snippets

Sorting edges:

```rust
edges.sort_by(|a, b| a.w.cmp(&b.w));
```

Min heap:

```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

let mut heap: BinaryHeap<Reverse<(i32, usize)>> = BinaryHeap::new();
heap.push(Reverse((dist, node)));
let Reverse((d, u)) = heap.pop().unwrap();
```

Queue:

```rust
use std::collections::VecDeque;

let mut q: VecDeque<usize> = VecDeque::new();
q.push_back(s);
let u: usize = q.pop_front().unwrap();
```

HashMap count:

```rust
use std::collections::HashMap;

let mut cnt: HashMap<i32, i32> = HashMap::new();

for &x in a.iter() {
    *cnt.entry(x).or_insert(0) += 1;
}
```

Vec init:

```rust
let mut dist: Vec<i32> = vec![i32::MAX / 4; n];
let mut seen: Vec<bool> = vec![false; n];
let mut parent: Vec<Option<usize>> = vec![None; n];
let mut adj: Vec<Vec<usize>> = vec![Vec::new(); n];
let mut wadj: Vec<Vec<(usize, i32)>> = vec![Vec::new(); n];
```

Ranges:

```rust
for i in 0..n { }          // 0 to n-1
for i in 1..=n { }         // 1 to n
for i in (0..n).rev() { }  // n-1 downto 0
for i in (1..=n/2).rev() { } // floor(n/2) downto 1
```

Safe INF:

```rust
let inf: i32 = i32::MAX / 4;
```

Max/min:

```rust
x = x.max(y);
x = x.min(y);
```

---

# 21. Concrete Tiny Examples for Proof Memory

## Cut property example

```text
Cut S={a,b}, V-S={c,d}
Crossing edges:
a-c weight 2
b-c weight 5
b-d weight 7

Lightest crossing edge a-c weight 2 is safe.
If MST didn't have it, add it -> cycle -> remove another crossing edge >=2.
```

## Cycle property example

```text
Cycle edges weights: 2, 4, 9, 5
Heaviest 9 cannot be in MST.
If tree uses 9, remove 9 and add some other edge on cycle to reconnect cheaper.
```

## Dijkstra negative edge failure

```text
s->a = 2
s->b = 5
b->a = -10

Dijkstra extracts a with 2 before b.
But true shortest to a is -5 through b.
```

## Bellman-Ford |V|-1

```text
simple path has at most V-1 edges.
After i passes, shortest paths with <=i edges known.
So after V-1 passes all simple shortest paths known.
```

## LCS table memory

```text
A[i-1] == B[j-1] -> take diagonal + 1
else              -> max(up, left)
```

## Knapsack memory

```text
dp[i][w] = max(
    don't take item i,
    take item i + remaining capacity
)
```

## Rod cutting memory

```text
Try first cut length i.
Then solve remaining rod n-i optimally.
```

---

# 22. Final “Algorithm Does What” Index

```text
Insertion Sort:
incrementally keeps prefix sorted.

Merge Sort:
sort halves recursively, merge sorted halves.

Maximum Subarray:
best left/right/crossing; or Kadane.

Strassen:
matrix multiplication with 7 recursive multiplications.

Heap Sort:
build max heap, repeatedly move max to end.

Priority Queue:
heap supporting insert/extract-max/min.

Rod Cutting:
DP over rod length, try first cut.

Matrix Chain:
DP over interval of matrices, try split point.

LCS:
2D DP over prefixes, diagonal on equal chars.

BFS:
queue, layers, unweighted shortest paths.

DFS:
recursive exploration, timestamps, edge types.

Topological Sort:
DFS reverse finish time or Kahn indegree.

Ford-Fulkerson:
augment flow along residual s-t paths.

Bipartite Matching:
source-left-right-sink, all capacities 1.

Edge Disjoint Paths:
capacities 1 on edges, max flow value.

Prim:
grow MST from one component using cheapest crossing edge.

Kruskal:
sort edges, add if DSU says no cycle.

Bellman-Ford:
relax all edges V-1 times; detects negative cycles.

Dijkstra:
priority queue; nonnegative shortest paths.

Quicksort:
partition by pivot, recurse; randomized expected n log n.

Hash Tables:
hash keys into buckets; expected O(1) with good hashing/load.
```

If you shrink this onto A4, I’d keep the pseudocode blocks for **DSU, heapify/build-heap, Kruskal, Prim, Dijkstra, Bellman-Ford, max-flow, LCS, matrix-chain, rod-cutting, knapsack**, and compress the rest into formulas/traps.
