# CS246 - Lecture 21 - Nov 20, 2018

## Cohesion 

- how closely elements of a module are related to each other

**low**:
- arbitrary grouping of unrelated elements (e.g. `<utility>`)
- elements shows a common theme, otherwise unrelated
(e.g. `<algorithm>`)

-elements manipulate state over the lifetime of an object (e.g. open/read/close files)
- elements pass data to each other

**high cohesion**: 
- elements cooperate to perform entirely one task 

**Goal**: low coupling, high cohesion.

## Decoupling the Interface (MVC)

Your primary program classes should not print things.

```C++
class ChessBoard {
    ...
    cout << "Your move" << endl;
};
```

Bad design - inhibits code reuse

What if you want to reuse ChessBoard, but not have it communicate via stdout?

**One solution:** give the class stream objects for I/O:

```C++ 
class ChessBoard {
        istream &in;
        ostream &out;
    
    public:
        ChessBoard (istream &in, ostream &out): in {in}, out {out} {}

        ....
        out << "Your move" << endl;

};
```

Better - but what if we don't want to use streams at all?

ChessBoard should not be doing any communication at all.

### Single Responsibility Princriple 
"A class should only have one reason to change"

- game play and communication are **two** reasons.
- communicate with the ChessBoard via params/results/exceptions
- confine user communication outside the game class.

**Q**: Communicate in main?

**A**: No - Hard to reuse code if it's in main.

Have a class for interaction, that is separate from the game play classes.

## Architecture: Model-View-Controller (MVC)

- separate the distinct notions of the data ("model") the presentation of the data ("view") and controller manipulation of the data ("controller")

insert mvc uml diagram

By decoupling presentation and control, MVC promotes reuse.

## Exception Safety

Consider:

```C++
void f() {
    MyClass mc;
    MyClass *p = new MyClass;
    g();
    delete p;
}
```

No leaks - but what if `g` throws?
- What is guaranteed?
- During stack - unwinding, all stack-allocated data is cleaned up 
- dtors run, memory reclaimed.
- Heap-allocated memory is not destroyed

If `g` throws, *p is leaked (`mc` is not)

```C++
void f() {
    MyClass mc;
    MyClass *p = new MyClass;

    try {
        g();
    }
    catch (...) {
        delete p;
        throw;
    }
    delete p;
}
```

- This is ugly and error prone - duplication of code
- How else can we guarantee that something (i.e. `delete p`) will happen no matter how we exit `f`?  (normal or exception)

In some languages - "finally" clauses that guarantee certain final actions 
- not in C++

Only thing you can count on in C++ - dtors for stack-allocated data will run.

Therefore, use stack-allocated data with dtors as much as possible
- use the guarantee to your advantage.

## C++ Idiom: RAII - Resource Acquisition Is Initialization

Every resource should be wrapped in a stack-allocated object, whose dtor deletes it.

**Eg. files**

```C++

{
    ifstream f{"file"}; // Acquiring the resource ("file") = initializing the object (f)
}
```

The file is guaranteed to be released when `f` is popped from the stack (`f`'s dtor runs) 


This can be done with dynamic memory.

```C++
#include <memory>

class std::unique_ptr <T>
```

- Takes a `T*` in the ctor
- the dtor will delete the ptr
- in between - can dereference, just like a ptr.

```C++
void f() {
    MyClass mc;

    std::unique_ptr <MyClass> p { new MyClass};
    g();
}
```

**Or** (equiv:)

```C++
void f() {
    void f() {
        MyClass mc;
        // unique_ptr <MyClass> or auto
        auto p = std::make_unique <MyClass> (); // ctor args (if any) go here.
        g();
    }
}
```

No leaks. Guaranteed.

Difficulty:

```C++
class C {...};
unique_ptr <c> p { new C{...} };
unique_ptr <c> q = p; // this is wrong
```

What happens when a unique_ptr is copied? - don't want to delete the same ptr twice!

Instead - copying is disabled for unqiue_ptrs. They can only be moved.

### Sample implementation:

```C++
template <typename T> class unique_ptr {
        T *ptr;
    public:
        explicit unique_ptr (T *p) ptr {p} {}
        ~unique_ptr() {delete ptr;}
        unique_ptr (const unique_ptr <T> &other) = delete;
        unique_ptr &operators (const unique_ptr <T> &other) = delete;
        unique_ptr (unique_ptr <T> &&other): ptr {other.ptr} {other.ptr = nullptr; }
        unique-ptr <T> &operators (unique-ptr <T> &&other) {
            std::swap(ptr, other.ptr);
            return *this;
        }
        T &operator *() {return *ptr;}
};
```

If you need to be able to copy pointers - first answer the question of **ownership**.

- Who will own the resource?  Who will have the responsibility for freeing it?
- That ptr could be a `unique_ptr`
- all other ptrs can be raw pointers. (can fetch the underlying new ptr with p.get())

If there is true shared ownership, i.e. any of several pointers might face the resource - use `std::shared_ptr`.

