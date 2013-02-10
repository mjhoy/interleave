Given an unsorted array of length `n`, the _number of inversions_ is, more or
less, the number of times any two numbers in the array are flipped from their
normal order. So, in the array

  [ 1, 5, 3, 7, 6 ]

there are 4 invesions. 5 and 3 are swapped, 3 and 7, 3 and 6, and 7 and 6.
(Given two lists of rankings, this number can measure how similar they are.
[More here][inversions].)

One algorithm to determine this number is to sort the array using a merge sort,
and in the merge step, while merging sorted arrays `A` and `B`, walking up the
arrays with indices `ia` and `ib`, if `ib` > `ia`, add the remaining length of
`A` (`A[ia..length(A)]`) to the total number of inversions.

It's a little non-intuitive, but fun to work out. Here's my C implementation [1].

```c
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <errno.h>

long int countInversions(int a[], int size, int temp[]) {
  int i1, i2, tempi;
  long int inv1, inv2;
  long int inv3 = 0;

  // Base case
  if (size <= 1) return 0;

  // Recursive calls
  inv1 = countInversions(a, size/2, temp);
  inv2 = countInversions(a + size/2, size - size/2, temp);

  // Merge
  i1 = 0;
  i2 = size/2;
  tempi = 0;
  while (i1 < size/2 && i2 < size) {
    if (a[i1] <= a[i2]) {
      temp[tempi] = a[i1];
      i1++;
    } else {
      temp[tempi] = a[i2];
      i2++;
      // Count length remaining in a[i1..size/2]
      inv3 = inv3 + ((size/2) - i1);
    }
    tempi++;
  }

  // Finish up the merge
  while (i1 < size/2) {
    temp[tempi] = a[i1];
    i1++;
    tempi++;
  }
  while (i2 < size) {
    temp[tempi] = a[i2];
    i2++;
    tempi++;
  }

  memcpy(a, temp, size*sizeof(int));
  return (inv1 + inv2 + inv3);
}

int main(int argc, char* argv[]) {
  int i = 0;
  int sz = ... /* Set to the size of the array ... */

  int* arr  = malloc(sizeof(int) * sz);
  int* temp = malloc(sizeof(int) * sz);

  while (i < sz) {
    arr[i] = ... /* get your numbers from somewhere... */
    i++;
  }

  inversions = countInversions(arr, sz, temp);
  printf("Inversions: %ld\n", inversions);

  free(arr);
  free(temp);

  return 0;
}
```

I've left out the specific implementation for reading in the number list.

Note that there are two recursive calls on an array size `n/2` in the
countInversions function, and then the work to merge is O(n). The running time
of this algorithmn should be O(n log(n)), just like merge sort.

Interestingly, given an array up to 1e5, the algorithm _appears_ to run in
linear time. I have no idea why this is, may investigate further (may be that
1e5 is still too small for good results).

![C algorithm results](https://raw.github.com/mjhoy/interleave/master/_img/c_count_inversions_time.png)

I also wrote an algorithm in Haskell [2] for fun:

```haskell
countInversions :: [Int] -> (Int, [Int])
countInversions [] = (0, [])
countInversions (x:[]) = (0, [x])
countInversions xs = (total, mergedArray)
  where
    halfway = (length xs) `quot` 2
    (a, b) = splitAt halfway xs
    second = countInversions a
    first = countInversions b
    merged = mergeInversions (snd first) (snd second)
    total = (fst first) + (fst second) + (fst merged)
    mergedArray = snd merged

mergeInversions :: [Int] -> [Int] -> (Int, [Int])
mergeInversions a b = mergeInversions' a b 0 []

mergeInversions' [] [] acc accls = (acc, accls)
mergeInversions' xs [] acc accls = (acc, xs ++ accls)
mergeInversions' [] ys acc accls = (acc, ys ++ accls)
mergeInversions' (x:xs) (y:ys) acc accls
  | x > y     = mergeInversions' xs (y:ys) acc (x:accls)
  | otherwise = mergeInversions' (x:xs) ys (acc + (length (x:xs))) (y:accls)
```

I was trying to be smart with an iterative recursive function. The performance
still wasn't very good:

![Haskell algorithm results](https://raw.github.com/mjhoy/interleave/master/_img/haskell_count_inversions_time.png)

Note I've only gone up to 3e4, and it's already taking close to 3 seconds. But
the performance curve looks a little more like what I would expect; however,
there are some big jumps, probably from garbage collection. Also merits further
investigation.

I know both of these algorithms can likely be improved. Especially, hopefully,
the Haskell: at array length 3e4, the C version is 80 times faster!

[inversions]:http://www.cp.eng.chula.ac.th/~piak/teaching/algo/algo2008/count-inv.htm

---

[1]: Bear in mind I'm just learning C.<br>
[2]: Same as above.
