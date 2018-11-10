# CS246 - Lecture 17 - Nov 6, 2018

## Templates

```C++
class List {
  struct Node {
    int data;
    Node * next;
  };
  Node *theList;
};
```

But what if you want to store something else? A whole new class?


**Templates:** class parameterized by a type

### Stack Class:
```C++
template <typename T> class Stack {	// we have a stack of T’s
    // we don’t want to commit to certain type at the moment
    int size, cap;
    T *contents;

  public:
    Stack() {}
    void push(T x) {}
    T &top() {}
    void pop() {}
};

template <typename T> class List {
  
    struct Node {
      T data;
      Node *next;
    };
    
    Node *theList;
  
  public:
    class Iterator {
      public:
        T &operator*() {}
    };
    
    T &ith(int i) {}
      void addToFront(T n);
    
};
```
#### Client Use

```C++
// Client
// I want a list of ints
List <int> &1;	// this tells it that you want a list of int’s
List <List <int>> l2;
l1.addToFront(3);
l2.addToFront(&1);

for (List<int>::Iterator it = l1.begin(); it != l1.end(); ++i) {
	cout << *it << endl;
}
```
#### OR
```C++
for (auto n:l1) {
  cout << n << endl;
}
```

When the C++ sees `List<int>`, it recognizes that it has seen this version of `List` before, so it creates a new class for the specialization using the template for you.
Compiler specializes templates at the source code level, before compilation.

**Q:** Where do we put the templates? .cc or .h file? \
**A:** All of it must be in the .h file. There is no .cc file AT ALL.

## The Standard Template Library (STL)
- Large # of useful templates.

**Eg.**
Dynamic Length Arrays
Arrays allocated by `new`
- Vectors

```C++
# include <vector>
std::vector<int> x {4, 5};	// 4, 5
x.emplace_back(6);	// emplace_back not push_back - more efficient 
v.push_back({4, 5}); // use push_back for implicit contructors
v.emplace_back(4, 5); // use emplace_back for explicit constructors
```
### Never use a heap-allocated array! Just use vectors

#### Note:
Brace bracket initialization.

```C++
vector<int> v {4, 5};
vector<int> v (4, 5);	// This is NOT the same. => v is 5 5 5 5
```
#### Looping over vectors:
```C++
for (int r = 0; r < v.size(); ++i) {	// v.size() is constant time.
	cout << v[i] << endl;
}
for (vector<int>::iterator it = v.begin(); it != v.end(); ++it) {
	cout << *it << endl;
}
```
#### Or

```C++
for (auto n = v) {
	cout << n << endl;
}
```
Iterate a vector in reverse!

```C++
for (vector<int>::reverse_iterator it = v.rbegin(); it != v.rend(); ++i) {

}
// can use auto as well
v.pop_back() // - removes last element
```

To remove item from the middle of inside a vector: You need to use an iterator!
```C++
auto it = v.erase(v.begin());	// this removes the first item
it = v.erase(v.begin() + 3);	// erases element 3 (4th)
```
**Note:** `v.erase()` returns an iterator that is pointing at the point of erasure, which you can use to erase a bunch of elements at that point.

```C++
it = v.erase(it);
v.erase(v.end() - 1);	// erase last item
```
**Q:** Does this properly call the destructor for the erased item? \
**A:** Yup, the item’s destructor will run.

`v[i]` - ith element of `v`.
This is unchecked - if you go out of bounds, it’s undefined behaviour.

`v.at(i)`
- checked version of `v[i]`
- what happens when you go out of bounds? what should happen?
- vector can detect error - but does not know what to do about it
- client can respond, but can’t detect the error

## Error Handling
- is a partnership between the detector and responder.

### C Solution:
- Functions return a status code, or set the global variable `errno` \
**Issue:** Error is very easy to ignore.
- Encourages programmers to ignore error-checks

### C++ Solution:
- Aborted - program stopped running (No segmentation fault, std::out_of_range)
- when an error condition arises, the function raises an exception
- By default, execution stops.

But we can write handlers to catch exceptions!

`vector<T>::at` raises the exception `std::out_of_range` when it fails.

```C++
#include <stdexcept>

...

try {
  cout << v.at(10000);
  ... // do things at success
} 
catch (out_of_range r) 
{ //       ^        ^
  //     class   object
  cerr << “Range error” << r.what() << endl; 
  // do something for range error
}
r.what() // - extra information about the error
```

#### Now consider:
```C++
void f() {
  // raises an exception  - throw out_of_range 
  throw out_of_range {“I am ERROR”};	
  // ^ is a constructor call. ^ is what .what() will produce
  // the constructor takes in what it will say for what()
}

void g()  { f(); }
void h() { g(); }

int main() {
  try 
  {
    h();
  } 
  catch (out_of_range r) 
  { 
      ...
  }
}
```

Once the exception is thrown, it doesn’t return to all the nested class, just goes directly to the catch statement at the highest level.

### What happens:
`main` calls `h`, `h` calls `g`, `g` calls `f`, `f` throws 

Control goes back through the call chain (unwinds the stack), until a handler is found.
- all the way back to main; main handles the exception 

No matching handler found => program terminates.

A handler can do part of a recovery job. Like - execute some corrective code and throw another exception.

```C++
try {...}
catch (ErrorType s) {
  ...
  throw SomeOtherError {...};
}
```
Or rethrow the same exception:

```C++
try {...}
catch (ErrorType s) {
  ...
  throw;
}
```
`throw` vs `throw s`

Exception can be subclasses. \
SomeErrorType => SpecialErrorType

`s` could be either of these.

`throw s` - throws a new expression of type SomeErrorType (sliced copy of `s`)
`throw` - actual type of `s` is retained


