# CMPS 6610 Problem Set 03
## Answers

**Name:** Marcus Rojo


Place all written answers from `problemset-03.md` here for easier grading.




- **1b.**
- `iterate` does one O(1) update per element, and each update depends on the result of the previous one.
W(n) = W(n−1) + O(1) = O(n)
S(n) = S(n−1) + O(1) = O(n)






- **1d.**
- The `map` checks each element independently, where W = O(n), S = O(1).
- The `reduce` splits in half each time, with the two halves in parallel:
         W(n) = 2W(n/2) + O(1) = O(n)
         S(n) = S(n/2) + O(1) = O(log n)
         Total: W = O(n), S = O(log n)





- **1e.**
- `ureduce` splits the input into multiple pieces of size n/3 and 2n/3 and runs them in parallel.
- W(n) = W(n/3) + W(2n/3) + O(1) = O(n). This is because each element is still combined once.
- S(n) = S(2n/3) + O(1) = O(log n), since the larger piece shrinks by a constant fraction each level. The base of the log is 3/2 instead of 2, but that's only a constant factor.
- `ureduce` calls `reduce` for its two pieces, so only the top split is uneven, but the bounds appear the same regardless.





- **2a.**
- Tag each element with its index, sort by value (ties broken by index) so duplicates become adjacent, keep only the first element of each run of equal values, then sort the surviving ones by index to restore the original order.
_______________________________________________________________________
dedup A =
  let
    P = ⟨ (A[i], i) : 0 ≤ i < |A| ⟩
    S = sort P                      (by value, ties broken by index)
    F = ⟨ S[i] : 0 ≤ i < |S| | i = 0 or first(S[i]) ≠ first(S[i−1]) ⟩
    T = sort F                      (by index)
  in
    ⟨ v : (v, _) ∈ T ⟩
  end
_______________________________________________________________________

- Work and span: The tabulate, filter, and final map each process every element independently, at O(n) work and O(log n) span or less than.
- The two sorts dominate, as a parallel merge sort takes O(n log n) work and O(log² n) span.
- Total: W = O(n log n), S = O(log² n)





- **2b.**
- Because order doesn't matter, I flatten all the lists into one sequence, then sort it so duplicates are adjacent, and keep each element that differs from its predecessor. In this situation, I treat it as unordered. 

______________________________________________________________
multiDedup A =
  let
    B = flatten A                   (N = m·n elements total)
    S = sort B
  in
    ⟨ S[i] : 0 ≤ i < |S| | i = 0 or S[i] ≠ S[i−1] ⟩
  end
______________________________________________________________

- Work and span: Flatten and filter take O(mn) work and O(log(mn)) span. The sort appears to dominate.
- Total: W = O(mn log(mn)), S = O(log²(mn))

- Comparison with 2a: It's the same form with n replaced by the total size N = mn. It's simpler/cheaper by a constant factor, because there's no index tagging and no second sort to restore order. In a real distributed system, each machine would first deduplicate its own list in parallel, at O(n log n) work and O(log² n) span. That shrinks the data before anything, and the partial results would then be combined with `reduce` using merge and deduplicate. I believe this would work because the set union is associative. 




- **2c.**
- Yes, several sequence operations are useful:
     - tabulate/map: pair each element with its index (2a), and compare each element with its left neighbor after sorting. Each element is handled independently, so these are fully parallel with O(1) span.
     - filter: it keeps only the first element of each run of equal values. Internally, it uses a scan to compute output positions, so its span is O(log n).
     - flatten: it combines the m lists into one sequence in 2b.
     - reduce: set union or merge and deduplicate on specifically sorted lists) is associative, meaning that results can be combined in a balanced tree. This fits the distributed setting, where each machine deduplicates locally first.
     - iterate: it gives a simple sequential solution, but each step depends on the previous one, so the span is O(n) with no parallelism.
- Sort is the main building block in this situation.







- **3b.**
- `iterate` does one O(1) update per character, and each update depends on the previous count, as seen in problem 1.
- W(n) = W(n−1) + O(1) = O(n)
- S(n) = S(n−1) + O(1) = O(n)





- **3d.**
- The `map` with `paren_map` handles each character independently: W = O(n), S = O(1).
- Contraction-based scan:
    - W(n) = W(n/2) + O(n) = O(n)
    - S(n) = S(n/2) + O(1) = O(log n)
- The final `reduce` with `min_f` adds O(n) work and O(log n) span.
- Total: W = O(n), S = O(log n)





- **3f.**
- Each call makes two half-size recursive calls (in parallel) and combines them in O(1):
     - W(n) = 2W(n/2) + O(1) = O(n)
     - S(n) = S(n/2) + O(1) = O(log n)
     - But this assumes O(1) splitting. Python's list slicing copies the list, which in practice should add O(n) per level.




