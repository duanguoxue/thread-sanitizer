#summary Description of some most popular data races



# Simple race #

A simplest possible data race is the most frequent one: two threads are accessing a variable of a
built-in type without any synchronization.
Quite frequently, such races are benign (the code counts some statistics which is allowed to be imprecise).

```
int var;

void Thread1() {  // Runs in one thread.
  var++;
}
void Thread2() {  // Runs in another thread.
  var++;
}

```

But sometimes such races are extremely harmful (e.g. if `var` is counting your money :).

## Thread-hostile reference counting ##
A variation of race on integer.
Very harmful.
May lead to occasional memory leaks or double deletes.

See http://crbug.com/15577 for a real life example.
```
// Ref() and Unref() may be called from several threads.
// Last Unref() destroys the object.
class RefCountedObject {
 ...
 public:
  void Ref() {
    ref_++;  // Bug!
  }
  void Unref() {
    if (--ref_ == 0)  // Bug! Need to use atomic decrement!
      delete this;
  }
 private:
  int ref_;
};
```


# Race on a complex object #

Another popular race happens when two threads access a non-thread-safe complex object
(e.g. an STL container) without synchronization. These are almost always dangerous.

```
std::map<int,int> m;

void Thread1() {
  m[123] = 1;
}

void Thread2() {
  m[345] = 0;
}
```


# Notification #

Consider the following code:
```
bool done = false;

void Thread1() {
  while (!done) {
    do_something_useful_in_a_loop_1();
  } 
  do_thread1_cleanup();
}

void Thread2() {
  do_something_useful_2();
  done = true;
  do_thread2_cleanup();
}
```

The synchronization between these two threads is done using a boolean variable `done`.
This is a wrong way to synchronize two threads.

On x86, the biggest issue is the compile-time optimizations.<br />
  * Part of the code of do\_something\_useful\_2() can be moved below "done = true" by the compiler.
  * Part of the code of do\_thread2\_cleanup() can be moved above "done = true" by the compiler.
  * If do\_something\_useful\_in\_a\_loop\_1() doesn't modify "done", the compiler may re-write Thread1 in the following way:
```
  if (!done) {
    while(true) {
      do_something_useful_in_a_loop_1();
    } 
  }
  do_thread1_cleanup();
```
so Thread1 will never exit.

On architectures other than x86, the cache effects or out-of-order instruction execution may lead to other subtle problems.

Most race detector will detect such race.<br />
Also, most dynamic race detectors will report data races on the memory accesses that were intended to be synchronized with this bool<br />
(i.e. between do\_something\_useful\_2() and do\_thread1\_cleanup())

To fix such race you need to use compiler and/or memory barriers (if you are not an expert -- simply use locks).<br />
It is highly recommended to wrap such a synchronization into a separate class (i.e. ExitNotification).

# Publishing objects without synchronization #
One thread initializes an object pointer (which was initially null)
with a new value, another thread spins until the object pointer becomes non-null.
Without proper synchronization, the compiler may do very surprising
transformations with such code which will lead to (occasional) failures.
In addition to that, on some architectures this race may cause failures due
to cache-related effects.

```
MyType* obj = NULL;

void Thread1() {
  obj = new MyType();
}

void Thread2() {
  while(obj == NULL)
    yield();
  obj->DoSomething();
}
```


# Initializing objects without synchronization #

This may lead e.g. to memory leaks (the object was constructed twice).
```
static MyObj *obj = NULL;

void InitObj() { 
  if (!obj) 
    obj = new MyObj(); 
}

void Thread1() {
  InitObj();
}

void Thread2() {
  InitObj();
}
```

# Reader Lock during a write #

Updates happening under a reader lock.
```
void Thread1() {
  mu.ReaderLock();
  var++;
  mu.ReaderUnlock();
}

void Thread2() {
  mu.ReaderLock();
  var++;
  mu.ReaderUnlock();
}
```



# Race on bit field #
The code below looks correct from the first glance.
But if `x` is `struct { int a:4, b:4; }`, we have a bug.
```
void Thread1() {
  x.a++;
}

void Thread2() {
  x.b++;
}
```



# Double-checked locking #
The so called doubled-checked locking is well know to be an anti-pattern,
but we still find it occasionally.

```
bool inited = false;
void Init() {
  // May be called by multiple threads.
  if (!inited) {
    mu.Lock();
    if (!inited) {
      // .. initialize something
    }
    inited = true;
    mu.Unlock();
  }
}
```


# Race during destruction #
Sometimes objects are created on stack, passed to another thread and
then destroyed without waiting for the second thread to finish its work.

```
void Thread1() {
  SomeType object;
  ExecuteCallbackInThread2(
    SomeCallback, &object);
  ...
  // "object" is destroyed when
  // leaving its scope.
}
```

# Data race on vptr #

Some background: Say you have:
```
struct A {
  virtual ~A() {
    F();
  }
  virtual void F() {
    printf("In A");
  }
};
struct B : public A {
  virtual void F() {
    printf("In B");
  }
};
```
C++ specifies that you'll see `"In A"` printed when an object of type `B` is destroyed. More specifically, during `A::~A()`, all virtual methods resolve to the method defined in `A` (or its base classes), not the override from `B`. To implement this, `gcc` and other compilers re-assign the vptr (pointer to virtual function table) at the _beginning_ of `B::~B()` to point to `B::vtable`, and then re-assign it again at the _beginning_ of `A::~A()` to point to `A::vtable`.

Now the race: Class `A` has a function `Done()`,
virtual function `F()` and a virtual destructor.
The destructor waits for the event generated by `Done()`.
There is also a class `B`, which inherits `A` and overrides `A::F()`.

```
#include <thread>
#include <mutex>

class A {
 public:
  A() : done_(false) {}
  virtual void F() { printf("A::F\n"); }
  void Done() {
    std::unique_lock<std::mutex> lk(m_);
    done_ = true;
    cv_.notify_one();
  }
  virtual ~A() {
    std::unique_lock<std::mutex> lk(m_);
    cv_.wait(lk, [this] {return done_;});
  }
 private:
  std::mutex m_;
  std::condition_variable cv_;
  bool done_;
};

class B : public A {
 public:
  virtual void F() { printf("B::F\n"); }
  virtual ~B() {}
};

int main() {
  A *a = new B;
  std::thread t1([a] {a->F(); a->Done();});
  std::thread t2([a] {delete a;});
  t1.join(); t2.join();
} 
```

An object `a` of static type `A` and dynamic type `B` is created.
One thread executes `a->F()` and then signals to the second thread.
The second thread calls `delete a` (i.e. `B::~B`)
which then calls `A::~A`, which, in turn, waits for the signal from
the first thread.
The destructor `A::~A` overwrites the vptr (pointer to virtual function table) to
`A::vptr`.
So, if  the first thread executes `a->F()` after the
second thread started executing `A::~A`, then `A::F` will be called instead of `B::F`.


Consider not using any synchronization in constructors/destructors if the class has virtual methods. Use `Start()` and `Join()` methods instead.

This data race is benign if you don't subclass the class which does synchronization in constructor/destructor.<br />
ThreadSanitizer can distinguish between benign and harmful races on vptr. <br />
Also, consider [sealing](http://www.google.com/search?q=sealed+C%2B%2B) your class to avoid problems in the future.

# Data race on vptr during construction #

This is another variation of the vptr race. Consider the following code:

```
class Base {
  Base() {
   global_mutex.Lock();
   global_list.push_back(this);
   global_mutex.Unlock();
   // point (A), see below
  }
  virtual void Execute() = 0;
  ...
};

class Derived : Base {
  Derived() {
    name_ = ...;
  }
  virtual void Execute() {
    // use name_;
  }
  string name_;
  ...
};

Mutex global_mutex;
vector<Base*> global_list;

// Executed by a background thread.
void ForEach() {
  global_mutex.Lock();
  for (size_t i = 0; i < global_list.size(); i++)
    global_list[i]->Execute()
  global_mutex.Unlock();
}
```

At first glance it looks that global\_list is properly synchronized with global\_mutex. But the object that is added into global\_list is partially constructed, yet becomes available to other threads. E.g. if the thread is preempted in point (A), then ForEach() will trigger pure virtual function call.

# Race on free #
Sometimes one thread of a program may access a heap memory while another thread is deallocating the same memory.

```
int *array;
void Thread1() {
  array[10]++;
}

void Thread2() {
  free(array);
}
```

[AddressSanitizer](http://code.google.com/p/address-sanitizer/) can also detect such bugs.

# Race during exit #
If a program calls `exit()` while other threads are still running, static objects may be destructed by one thread and used by another thread at the same time.
(Conclusion: don't use static non-POD objects unless you know what  you are doing)

```
#include "pthread.h"
#include <map>

static std::map<int, int> my_map;

void *MyThread(void *) {  // runs in a separate thread.
  int i = 0;
  while(1) {
    my_map[(i++) % 1000]++;
  }
  return NULL;
}

int main() {
  pthread_t t;
  pthread_create(&t, 0, MyThread, 0);
  return 0;
  // exit() is called, my_map is destructed
}

```

# Race on a mutex #
Lock and unlock operations on a mutex are synchronized, but a mutex still can be a subject to unsafe publication or racy destruction. Consider the following example:

```
class Foo {
  Mutex m_;
  ...

 public:
  // Asynchronous completion notification.
  void OnDone(Callback *c) {
    MutexLock l(&m_);
    // handle completion
    // ...
    // schedule user completion callback
    ThreadPool::Schedule(c);
  }
  ...
};

void UserCompletionCallback(Foo *f) {
  delete f;  // don't need it anymore
  // notify another thread about completion
  // ...
}
```

In this example destructor of `MutexLock` can unlock already deleted mutex, if `UserCompletionCallback` is already started executing on another thread.

# Race on a file descriptor #
File descriptors are fully synchronized internally, but a race in user code can lead to a read/write operation issued on a wrong file descriptor. Consider the following example:

```
int fd = open(...);

// Thread 1.
write(fd, ...);

// Thread 2.
close(fd);
```

If threads 1 and 2 race with each other, and the file descriptor number happens to be reused right after close, write can be issued for a wrong file or socket. This can lead to e.g. leaking sensitive data into an untrusted network connection.