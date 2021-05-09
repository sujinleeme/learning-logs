# Big O Notation

Q: How much time does it take to run this function = How does the runtime of this function grow?

A: Big O and time complexity

Q: Time Complexity

A: A way of showing how the runtime of a function increase as the size of the input increase.

- linear time O(N)
- constant time O(A)
- quadratic time O(N\*2)

## Big O notation equation

1. Find the fastest growing term
2. Take out the coefficient

`T = an + b = 0(n)` (a is 1, b is 2)

`T = cn^2 + dn + e = 0(n^2)` (c is 1, d and e are 2)

## How time scales with respect to some input variables

O(N) - where N is the size of the array (Linear)

```
boolean contains(array, x) {
  for each element in array {
    if element === x {
      return true
    }
  }
}
```

O(N^2)

```
void printPairs(array) {
  for each x in array {
    for each y in array {
      print x, y
    }
    // will print (5, 2) and (2, 5)
  }
}
```

### "N" is just a variable.

Plant carrots in garden with 100m(width) \* 100m(height)

- Option 1. `O(a)` : `a` = area of grass

- Option 2. `(S^2)`: `S` = length of one side

`O(a) === O(S^2)`, since `a = S^2`. Variables have meanings, and it matters what they are.

### Big O rules

**1. Different steps get added**

```
function something() {
  doStep(1) // O(a)
  doStep(2) // O(b)
  // O(a+b)
}
```

**2. Drop constants**

```

// O(2N)
function minMax1(array) {
  min, max <= NULL
  for each e in array   // O(N)
    min = MIN(e, min)
  for each e in array   // O(N)
    max = MAX(e, max)
}
```

```
// O(N)
fucntion minMax2(array) {
  min, max <= NULL
  for each e in array   // O(N)
    min = MIN(e, min)
    max = MAX(e, max)
}
```

**3. Different inputs => Different variables**

```
int intersectionSize(arrayA, arrayB) {
  int count = 0
  for a in arrayA {
    for b in arrayB {
      if a == b {
        count = count + 1
      }
    }
  }
  return count
}
```

`O(N^2)`? What does `N` mean? `N `is not the size of array. Instead, it can be described `O(a \* b)`.
`a` is length of `arrayA`, `b` is length of `arrayB`.

Big 0 expresses how the runtime changes, how its **scales**.

**4. Drop non-dominant terms**

```
// O(n+n^2)
function whyWouldIdoThis(array) {
  max = NULL
  // O(n)
  for each e in array {
    max = MAX(a, max)
    print max
  }

  // O(n^2)
  for each a in array{
    for b in array {
        print a, b
    }
  }
}}
```

Either the first for-loop or the last one can be dropped.

`O(N^2) <= O(N+N^2) <= O(N^2 + N^2)`

If left and right are equivalent, then center is too. `O(N^2) <= O(N+N^2)`.

#### References

- [CS Dojo - Introduction to Big O Notation and Time Complexity](https://www.youtube.com/watch?v=D6xkbGLQesk)
- [HackerRank - Big O](https://www.youtube.com/watch?v=v4cd1O4zkGw)
