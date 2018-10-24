# CS246 - Lecture 11 - Oct 16, 2018

### Would be better to create the copy first then delete

```C++
Node &operator = (const Node &other) {
    if (this == &other) return *this;
    Node *tmp = next;
    next = other.next ? new Node {* other.next} : nullptr; // copy constructor
    data = other.data;
    delete tmp; // destructor
    return *this; // if new fails, Node has old data
}
```
### We can employ the copy-and-swap idiom to achieve the advantages of this previous version more concisely

```C++
#include <utility>
struct Node {
    ...
    void swap (Node &other) {
        using std::swap;
        swap (data, other.data);
        swap (next, other.next);
    }

    Node &operator = (const Node &other) {
        Node tmp = other; // copy constructor
        swap (tmp);
        return *this; // tmp is deleted by going out of scope
    }
};
```
## Rvalues and rvalues references

### Recall:
- an lvalue is with an address
- an lvalue reference (&) is like a const ptr with auto-deferencing: always initialized to a lvalue 

### Consider:
```C++
Node n {1, new Node{2, nullptr}};
Node m = n; // copy constructor
Node m2;
m2 = n; // copy assignment operator

Node plusOne (Node n) {
    for (Node *p = &n; p; p = p->next) {
        ++p.data;
    }
    return n;
}

Node m3 = plusOne (n); // what function is called?
```

If the definition of m3 invokes copy constructor (it does), what is other? Other is an lvalue, but what is the address of the object it points at?
The compiler creates a temporary object to hold the results of `plusOne(n);` other is a reference to temporary, and the copy constructor deep copies from the temp.

### But:
- the temp is going to be discarded anyway, as soon as `Node m3 = plusOne (n)` ends
- it is wasteful to have to copy from the temp, so let's steal from it instead
- we need to know if parameter is a temp or a standalone object (something that exists after that line of code)

In C++, an **rvalue reference** of type `Node &&` is a reference to a temporary object (on rvalue) of type `Node`. \
We can write a constructor that takes a `Node &&`:
```C++
struct Node {
    ...
    Node (Node &&other) {   // move constructor
        ...
    }
    // What should this do? steal others data!

    Node (Node &&other) : data{other.data}, next{other.next} {
        other.next = nullptr; // so the data we "stole" is not deleted when other is deleted
    } 
};
```

### Similary:
```C++
Node m;
m = plusOne (n); // assignment from a temp
```
Again, to avoid copy from the temp whose data is about to be deleted, write a, **move assignment operator**.

```C++
Node &operator = (Node &&other) { // steal other's data, and other will be destroyed
    using std::swap;
    swap (data, other.data);
    swap (next, other.next);
    return *this; // other goes out of scope and deletes our data
}
```
If you don't define a move constructor/assignment operator, the copy versions are used

If the move constructor/move assignment operator is defined, it will replace all calls to the copy constructor/copy assignment operator where the arg is a temp rvalue. The compiler takes care of this.

## Copy/move Elison
```C++
Vec makeVec () {return {0,0};}
Vec v = makeVec(); // what runs here?
```

Copy constructor or a move constructor could run. Not sure! In g++, just a basic constructor would be called, no copy no move.
In some circumstances, C++ is allowed to (but doesn't have to) skip calling copy/move constructors. In the example, `makeVec` writes the result directly into the space occupied by `v` in the caller, rather than creating the object in its own stack frame and then copying it.

### Another example:
```C++
Void doSomething(Vec v) { // pass-by-value, calls move/copy constructor
    ...
}
 doSomething (makeVec()); //normally calls move constructor to construct v in doSomething
 ```

 Under copy elison, the result of `makeVec()` would be written directly into the param of `doSomething`, svaing a move.

 Copy elison is allowed, even if dropping the constructor call would change the behaviour of the program (e.g. the constructor prints something)

 You are **not** expected to know when copy elison happens.

 You are expected to know its possible.

 If you really want all omitted constructors to be run, in `g++` you can pass the option `-fno-elide-constructors`, But this can slow down your program considerably.

### In summary, we have the Rule of 5:
If you need a custom version of any of
1. Copy constructor
2. Copy assignment operator
3. Destructor
4. Move constructor
5. Move assignment operator

You usually need a custom version of all 5.

## Member Operators
In the previous discussion, we had operator `=` as a member function and not a stand alone function. When an operator is a member function, this plays the role of LHS arg.

```C++
struct Vec {
    int x, y;
    ...
    Vec operator + (const Vec &rhs) {
        return {x + rhs.x, y + rhs.y};
    }
    Vec operator * (const int k) {
        return {x * k, y * k};
    }
}
```

**Note:** 
```C++
operator * v * k (vec * int)
```
What if I want `k * v`?
It cannot be a member function, because the first arg is an `int` not a `Vec`. So it must be a standalone:
```C++
Vec operator * (const int k, const vec &v) {
    return v * k;
}