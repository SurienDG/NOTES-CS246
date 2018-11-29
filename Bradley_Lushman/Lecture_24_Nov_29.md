# CS246 - Lecture 24 - Nov 29, 2018

## Template Functions

```C++
template <typename T> T min(T x, T y) {return x < y ? x : y; }

int x = 1, y = 2;

int z = min(x, y); //T = int - compiler figures this out, based on the types of x and y - don't have to say min<int>(x, y)

auto f = min(1.0, 3.0); // T = double
```

min works for any type `T` that has `operator<`

C++ STL Library <algorithm> - suite of template f'ns, many of which work over iterators 

```C++
template <typename Iter, typename T>
Iter find (Iter first, Iter last, const T &val) {
    while (first != last) {
        if (*first == val) return first;
        ++first; 
    }
    return last;
}
```

count - like find, but returns # of occurences of val

```C++
template <typename InIter typename OutIter>
OutIter copy (InIter first, InIter last, OutIter result) {
    // - copies one container range [first, last) to another, starting at result
}
```

**Note:** does not allocate new memory.

**eg.**
```C++
vector <int> v {1, 2, 3, 4, 5, 6, 7};
vector <int> w(4); //0 0 0 0

copy(v.begin() + 1, v.begin() + 5, w.begin());

// w = 2 3 4 5
```

```C++
template <typename InIter, typename OutIter, typename Func>

OutIter transform (InIter first, InIter last, OutIter result, Func f) {
    while (first != last) {
        *result = f(*first);
        ++first;
        ++result;
    }
    return result;
}
```

**Eg.**

```C++
int add1 (int n) { return n + 1; }
...
vector v {2, 3, 5, 7, 11};
vector w (v.size()); // 0 0 0 0 0

transform(v.begin(), v.end(), w.begin(), add1); // w = 3, 4, 6, 8, 12
```

How general is this code?

1. What can we use for Func?
2. What can we use for InIter/OutIter? 


1. Func - how is f used? f(*first) 



