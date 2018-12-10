# CS246 - Lecture 24 - Nov 29, 2018

## Multiple Inheritance

A class can inherit from more than one class

```C++
class A { int a; }
class B { int b; }

class C : public A, public B { ... };
```
- has fields a and b

insert diagram 1

**Now consider:**

```C++
class A { int a; }
class B { int a; }

class C : public A, public B { ... };

D d;
d.a // what is this? Ambiguous

// Need to say
d.B::a
d.C::a
```
insert diagram

**Challenges:**
What if we want a singular A?

### Deadly Diamond

Make `A` a **virtual base case** (virtual inheritance)

```C++
class B: virtual public A {...}
class C: virtual public A {...}
```

How would this be laid out?

insert diagram

insert vtable diagrams

What does g++ do?

B needs to be laid out so we can find its A part - but the distance to the A part is not always the same.
.
insert diagram

**Soln:** Location of base class object is stored in the vtable 

Diagram doesn’t look like all of A, B, C, D simultaneously but slices of it do look like A, B, C, D
Thus, ptr assignment among A, B, C, D changes the address shared in the ptr.

```C++
D *d = new D;
A *a = d;
```

a and d will not contain the same A.
d’s value shifted to point to the A part when assigning into A.
`static_cast`/`dynamic_cast` will take this adjustment; `reinterpret_cast` will not.

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

----------------------------
    
1. Func - how is f used? f(*first) 
   - `f` can be anything that can be called as a function

Can write `operator()` for objects

**eg.**
```C++
class Plus1 {
    public:
        int operator() (int n) { return n + 1; }
};

Plus1 p;
p(4); // produces 5
```

```C++
transform(v.begin(), v.end(), w.begin(), Plus1{});
//                              ctor call  ^
```
**Generalize:**
```C++
class Plus {
        int m;
    public:
        Plus(int m) m{m} {}
        int operator() (int n) { return n + m; }
};
```
```C++
transform(v.begin(), v.end(), w.begin(), Plus{1});
//                 called function objects  ^
```

Advantage of classes - can maintain state

```C++
class IncreasingPlus {
        int m = 0;
    public:
        int operator() { return n + (m++); }
        void reset() { m = 0;}
};
```

```C++
vector <int> v (5, 0); // 0 0 0 0 0
vector <int> w = v; // 0 0 0 0 0

transform (v.begin(), v.end(), w.begin(), IncreasingPlus{});

// w = 0 1 2 3 4
```

**Consider:**

How many ints in a vector `v` are even?

```C++
vector <int> v _______;
bool even (int n) { return n % 2 == 0; }
int x = count_if(v.begin(), v.end(), even);
```

Seems a waste to explicitly create the function even. 

In Racket, we would use lambda.

```C++
int x = count_if(v.begin(), v.end(), [](int n) {return n % 2 == 0; });

// [] is like lambda for C++
```
2. Iterators

- apply the notion of iteration to other sources of data, e.g. streams.

```C++
#include <iterator>

vector <int> v {1, 2, 3, 4, 5};
ostream_iterator <int> out { cout, ", "};

copy(v.begin(), v.end(), out); // Prints 1, 2, 3, 4, 5,

vector <int> v {1, 2, 3, 4, 5};
vector <int> w;
copy (v.begin(), v.end(), w.begin()); // wrong
```

Remember - copy doesn't allocate space in `w` - it can't - it doesn't even know that `w` is iterating over a vector

It doesn't know about `w` at all! Only has an iterator.

But it we had an iterator whose assignment operator inserts a new item?

```C++
copy (v.begin(), v.end(), back_inserter(w)); // copies v to the of w, adding new entries

// back_inserter() is available for any container with a push_back method.
```







