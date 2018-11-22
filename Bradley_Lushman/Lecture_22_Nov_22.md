# CS246 - Lecture 22 - Nov 22, 2018

**Recall:**

```C++
std::shared_ptr {
    auto p1 = std::make_shared <MyClass>();
    if (____) {
        auto p2 = p1;
    }
    // p2 is popped, ptr not deleted

} // p1 is popped, ptr is not deleted
```

shared_ptrs maintain a **reference count** - count of all shared_ptrs pointing at that object.

Memory is freed when the ref count reaches 0.

**Consider**:

```Racket
(define l1 (cons 1 (cons 2 (cons 3 empty))))
(define l2 (cons 4 (rest l1)))
```

insert memory diagram for lists

Use the type of ptr that accurately reflects the ptr's ownership role.

**Dramatically** fewer opportunities for leaks

### Back to exceptioin safety:

3 levels of exception safety guarantees.

1. Basic guarantee - if an exception occurs, the program will be in some valid state 
    - nothing is leaked, class invarients maintained, no corrupted data structures

2. Strong guarantee - if an exception is raised while executing `f`, the state of the program will be as if `f` had not been called

3. No-throw guarentee - `f` will never throw an exception, and will always achieve its purpose.

#### Eg

```C++
class A {...}; class B {...};

class C {
        A a;
        B b;

    public:
        void f() {
            a.g(); // may throw (strong guarantee)
            b.h(); // may throw (strong guarantee)
        }
};
```

Is `C::f` exception safe?

- if `a.g()` throws, nothing has happened yet. OK
- if `b.h()` throws, effects of `g` would have to be undone to offer the strong guarantee.
    - very hard or impossible if `g` has non-local side-effects

No, probably not exception safe.

If `g + h` do not have non-local side-effects, can use copy + swap:

```C++
class C {
    ...
    void f() {
        A atemp = a;
        B btemp = b;

        atemp.g(); // if this throws, original a still intact
        atemp.h(); // if this throws, original b still intact

        a = atemp; 
        b = btemp;
        // but what if copy assignment throws?

    }
};
```
**Soln:** Pimpl Idiom.

```C++

class C {
    unique.ptr < CImpl > pImpl;

    ...

    void f() {
        auto tmp = make_unique < CImpl > (*pImpl);
        tmp->a.g();
        tmp->b.h();
        std::swap(pImpl, tmp); // No throw
    }
};
```
## Exception Safety and the STL: vectors

vectors - encapsulate a heap-allocated array

- follow the RAII - when a stack-allocated vector goes out of scope, the internal heap-allocated array is freed.

```C++
void f() {
    vector <MyClass> v;
    ...
} // v goes out of scope - array is freed, MyClass dtor runs on all objects in the vector
```

**But**

```C++
void g() {
    vector <MyClass *> v;
    ...
}
```

Array is freed - ptrs don't have dtors, so any objs pointed to by ptrs are **not** deleted.

- `v` doesn't know whether the ptrs in the array **own** the objects they point at

```C++
for (auto &x: v) delete x;
```

**But**

```C++
void h() {
    vector <unique-ptr <MyClass>> v;
    ...
}
```

- array is freed, `unique_ptr` dtor runs, so the objects **are** deleted.

- **NO** explicit deallocation.

```C++
vector <T> :: emplace_back 
```

- offers the strong guarantee
- if the array is full (`size == cap)
    - allocate new array
    - copy objects over (copy ctor) 
    - delete old array

    if a copy ctor throws:
    - destroy new array
    - old array still intact

This is **strong guarentee**

**But**
- copying is expensive **and** old array will thrown array.
- wouldn't moving the object be more efficient?
- allocate new array
- **move** objects over (move ctor) 
- delete old array

But if move ctor throws:

- can't offer the strong guarantee
- original no longer intact.

If the move ctor offers the no throw guarantee, `emplace_back` will use the move ctor; else it uses the copy ctor, which may be slower.

So your move operations should provide the nothrow guarantee, and you should indicate they do:

```C++
class MyClass {
    public:
        MyClass (MyClass &&other) noexcept {...}
        MyClass &operator= (MyClass && other) {...}
}
```

If you know that a function will never or propagate an exception, declare it noexcept.

Facilitates optimization.

**At minimum:** moves and swaps should noexcept

## Casting

In C:

```C++
Node n;
int *ip = (int *) &n; // cast-forces C++ to treat a Node * as an int *.
```
C-style casts should be avoided in C++. If you **must** cast, use a C++ style exist.

**4 kinds:**

1. static-cast - "sensible" casts with well-defined meanings

**Eg.** double->int

```C++
double d;
void f (int x);
void f (double x);
f (static-cast<int>(d)); // calls void f (int x);
```
**Eg.** superclass ptr -> subclass ptr

```C++
Book *b = new Text {...};
Text *t = static_cast <Text *> (b);
```

- You are taking responsibility that `b` actually points at a `Text` - "Trust me"

2. reinterpret_cast - unsafe, implementation-specific, "weird conversions"

```C++
Student s;
Turtle *t = reinterpret_cast <Turtle *>(&s);
```






