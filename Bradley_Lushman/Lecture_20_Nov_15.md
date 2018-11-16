# CS246 - Lecture 20 - Nov 15, 2018

```C++

class Book {
    public:
        virtual void accept (Book vistor &v) {v.visit (*this)}
};

class Test: public Book {
    public:
        void accept (BookVisitor &v) { v.visit (*this); }
};

// etc

class Book Visitor {
    public:
        virtual void visit (Book &b) = 0;
        virtual void visit (Text &t) = 0;
        virtual void visit (comic &c) = 0;
};
```

#### Application:

Track how many of each type of book I have:

Book - by Author
Texts - by Topic
Comics - by Hero

Use a `map <string, int>`. Could add `virtual void update (___)` to each class.

Or write a visitor: 

```C++
struct Catalogue : public BookVisitor {
    
    map <string, int> theCat;
    void visit (Book &b) override { ++theCat [b.getAuthor()]; }
    void visit (Text &t) override { ++theCat [t.getTopic()]; }
    void visit(Comic &c) override { ++theCat [c.getHero()]; }
};
```

But it won't compile! Why?

`book.h` and `BookVisitor.h`  include each other

- circular include dependency `book.h` won't be included a second time - include guard

Result: Text doesn't know what Book is.

- Are these includes **really** needed?


## Compilation Dependencies

When does a compilation dependency need to exist?

**Consider**: 
```C++
class A{...}; // A.h
```

```C++
#include "A.h"

class B: public A {
    ...
};
```
```C++
#include "A.h"
class C {
    A a;
};
```

```C++
class A;

class D {
    A *a;
};
```

```C++
class A;

class E {
    A f(A x);
}
```

```C++
#include "A.h"

class F {
    A f(A x) {... x.someMethod() ...}
};
```

If the code doesn't need an include, don't create a needless compilation dependency by including unnecessarily.

When class A changes, only A, B, C, F need recompilation

In the **implementations** of D, C: 

> d.cc

```C++
void D::f() {
    a->someMethod(); // Need to know about class A 
                     // A true compilation dependency 
}
```
Do the include in the .cc file instead of the .h file (where possible)

Now consider the XWindow class.

```C++
class XWindow {
    Display *d;
    Window w;
    int s;
    GC gc;

    unsigned long colours [10];

    public:
        ...
};
```

Class has private data above. Yet we can look at it.

Do we know what it all means? Do we care?

What if I add or change a private member?

All clients must recompile.

Would be better to hide these details away.

**Soln:** **pimpl idiom** ("pointer to implementation")

Create a second class XWindowImpl:

> XWindowImpl.h

```C++
#include <X11/Xlib.h>

struct XWindowImpl {
    Display *d;
    Window w;
    int s;
    GC gc;
    unsigned long colours [10];
};

```

> Window.h

```C++
class XWindowImpl; // No need to include Xlib.h - forward declare the impl class

class XWindow {
    XWindowImpl *pImpl; // No compilation dependency on XWindowImpl.h
                        // clients also don't depend on XWindowImpl.h
    public:
    ... // No change
}
```

> Window.cc

```C++

#include "Window.h"
#include "XWindowImpl.h"

XWindow::XWindow(____): pImpl { new XWindowImpl } {...}

```

Other methods - replace fields d,w,s, etc
with `pImpl->d`, `pImpl->w`, `pImpl->s`, etc.

If you keep all private fields in XWindowImpl, then only window.cc needs to be recompiled if you change XWindow's Implementation.

Generalization: What if there are several possible window implementations, say 

XWindows + YWindows

- then make the Impl struct a superclass:

insert UML1

pImpl idiom with a **class hierarchy of implementations** - call the **Bridge Pattern**

## Measures of Design Quality


