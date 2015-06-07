

# Introduction #

Most programmers know that races are harmful. <br>
For some races it could be quite easy to predict what may go wrong.<br>
For example, the variable <code>var</code> in the following code may not be equal to <code>2</code> at the end of the program execution (for more examples refer to PopularDataRaces)<br>
<pre><code>int var;<br>
void Thread1() {  // Runs in one thread.<br>
  var++;<br>
}<br>
void Thread2() {  // Runs in another thread.<br>
  var++;<br>
}<br>
</code></pre>

However, there are other, much more subtle races which are much less obviously harmful. This article is about such races. Most of the content is specific to C/C++.<br>
<br>
TODO(timurrrr) Reference the C++11 standard mentioning data races lead to UB?<br>
<br>
TODO(kcc) This article is not finished. Expect more content later.<br>
<br>
<br>
<h1>Racy publication</h1>
The race which is most frequently (and mistakenly) perceived as harmless is <b>unsafe publication</b>:<br>
<pre><code>struct Foo {<br>
  int a;<br>
  Foo() { a = 42; }<br>
};<br>
<br>
static Foo *foo = NULL;<br>
<br>
void Thread1() {  // Create foo.<br>
  foo = new Foo();<br>
}<br>
<br>
void Thread2() {  // Consume foo.<br>
  if (foo) {<br>
     assert(foo-&gt;a == 42);<br>
  }<br>
}<br>
</code></pre>
ThreadSanitizer and most other race detectors will report this as a race on <code>foo</code>.<br>
<br>
<h2>Objection 1</h2>
But hey, why this race is harmful? Follow my fingers:<br>
<ul><li>Thread1 calls <code>new Foo()</code>, which allocates memory and calls <code>Foo::Foo()</code>
</li><li>Thread1 assigns the new value to <code>foo</code> and <code>foo</code> becomes non-NULL.<br>
</li><li>Thread2 checks if <code>foo != NULL</code> and only then reads <code>foo-&gt;a</code>.<br>
The code is safe!</li></ul>


<h2>Clarification 1</h2>
On machines with weak <a href='http://people.redhat.com/drepper/cpumemory.pdf'>memory model</a>, such as IBM Power or ARM,<br>
<code>foo != NULL</code> does not guarantee that the contents of <code>*foo</code> are ready for reading in Thread2.<br>
For example, the following code does <b>not</b> work on machines with weak memory model.<br>
<pre><code>int A = 0, B = 0;<br>
<br>
void Thread1() {<br>
  A = 1;<br>
  B = 1;<br>
}<br>
<br>
void Thread2() {<br>
  if (B == 1) {<br>
     assert(A == 1);<br>
  }<br>
}<br>
</code></pre>

<h2>Objection 2</h2>
Ok, ok. But I don't care about weak memory model machines. x86 has a strong memory model!<br>
<br>
<h2>Clarification 2</h2>
If you are writing in assembly, yes, this is safe.<br>
But with C/C++ you can not rely on the machine's strong memory model because the compiler may (and will!) rearrange the code for you.<br>
For example, it may easily replace <code>A=1;B=1;</code> with <code>B=1;A=1;</code>.<br>
<br>
<h2>Objection 3</h2>
Ok, I believe that compilers may do simple reordering, but that doesn't apply to my code!<br>
For example, <code>Foo::Foo()</code> is defined in one <code>.cc</code> file and <code>foo = new Foo()</code> resides in another.<br>
And even if these were in a single file, what kind of reordering could harm me?<br>
<br>
<h2>Clarification 3</h2>
First, it's a bad idea to rely on the compiler's inability to do some legal transformation. <br>
Second, the modern compilers are much more sophisticated than most programmers think.<br>
For example, many compilers can do cross-file inlining or even inline virtual functions (!). <br>
So, in the example above a compiler may (and often will) change the code to<br>
<pre><code>  foo = (Foo*)malloc(sizeof(Foo));<br>
  new (foo) Foo();<br>
</code></pre>

<h2>Objection 4</h2>
Ok. So I will do something to avoid any bad compiler effect in the thread which creates <code>foo</code>.<br>
But obviously, the consumer thread's code is safe. What can possibly go wrong with this code on x86?<br>
<pre><code>void Thread2() {  // Consume foo.<br>
  if (foo) {  <br>
     // foo is properly published by Thread1().<br>
     assert(foo-&gt;a == 42);<br>
  }<br>
}<br>
</code></pre>

<h2>Clarification 4</h2>
Again, you are underestimating the cleverness of modern compilers. How about this (it really happens!)?<br>
<pre><code>void Thread2() {  // Consume foo.<br>
  Foo *t1 = foo;  // reads NULL<br>
  Foo *t2 = foo;  // reads the new value<br>
  if (t2) {<br>
     assert(t1-&gt;a == 42);<br>
  }<br>
}<br>
</code></pre>

<h1>Double-Checked Locking</h1>
<a href='http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html'>Double-Checked Locking</a> is broken in C++ because of all the reasons discussed <a href='#Racy_publication.md'>above</a>.<br>
<br>
<h1>Racy lazy initialization</h1>
One more frequent bug:<br>
<pre><code>// lazy init for Pi. <br>
float *GetPi() {<br>
  static float pi;<br>
  static bool have_pi = false;<br>
  if (!have_pi) {<br>
     pi = ComputePi(); // atomic assignment, no cache coherency issues.                    <br>
     have_pi = true;<br>
  }<br>
  return pi;<br>
}<br>
</code></pre>

Even experienced programmers may assume that this code is correct on a CPU with strong memory model.<br>
But remember that a compiler may rearrange <code>have_pi = true;</code> and <code>pi = ComputePi();</code> and one of the threads will return uninitialized value of <code>pi</code>.<br>
<br>
<br>
<h1>Volatile</h1>
DON'T use <code>volatile</code> for synchronization in C/C++.<br>
The C and C++ standards don't say anything about <code>volatile</code> and threads -- so don't assume anything.<br>
<br>
Also, the C++11, chapter intro.multithread, paragraph 21 says:<br>
"The execution of a program contains a data race if it contains two conflicting actions in different threads, at least one of which is not atomic, and neither happens before the other. Any such data race results in undefined behavior."<br>
<br>
<br>
You don't believe us? Ok, here is a simplest proof that C++ compilers don't treat volatile as synchronization.<br>
<pre><code>% cat volatile.cc <br>
typedef long long T;<br>
volatile T vlt64;<br>
void volatile_store_64(T x) {<br>
        vlt64 = x;<br>
}<br>
% g++ --version <br>
g++ (Ubuntu 4.4.3-4ubuntu5) 4.4.3<br>
...<br>
% g++ -O2 -S volatile.cc -m32 &amp;&amp; cat volatile.s <br>
...<br>
_Z17volatile_store_64x:<br>
.LFB0:<br>
        pushl   %ebp<br>
        movl    %esp, %ebp<br>
        movl    8(%ebp), %eax<br>
        movl    12(%ebp), %edx<br>
        popl    %ebp<br>
        movl    %eax, vlt64<br>
        movl    %edx, vlt64+4<br>
        ret<br>
...<br>
</code></pre>
You can clearly see that a 64-bit volatile variable is written in two pieces w/o any synchronization.<br>
Do you still use C++ <code>volatile</code> as synchronization?<br>
<br>
You may object that we are cheating (with 64-bit ints on a 32 arch).<br>
Ok, we indeed cheated, but just a bit. How about this case?<br>
<br>
<pre><code>% cat volatile2.cc <br>
volatile bool volatile_bool_done;<br>
double regular_double;<br>
void volatile_example_2() {<br>
        regular_double = 1.23;<br>
        volatile_bool_done = true;<br>
}<br>
% g++ -O2 volatile2.cc -S<br>
% cat volatile2.s <br>
...<br>
_Z18volatile_example_2v:<br>
.LFB0:<br>
..<br>
        movabsq $4608218246714312622, %rax<br>
        movb    $1, volatile_bool_done(%rip)<br>
        movq    %rax, regular_double(%rip)<br>
        ret<br>
...<br>
</code></pre>

Here you can see that the compiler reordered a non-volatile store and a volatile store.<br>
<br>
Are you convinced now? If no, please let us know why!<br>
<br>
<br>
<h1>How to fix</h1>
So, how do I make my code correct?<br>
<br>
If you are not a black belt in synchronization primitives, please use something crafted by those who are (e.g. <code>pthread_mutex_lock</code>, <code>pthread_once</code>, etc).<br>
<br>
If you are indeed a black belt (are you?), you probably know about CAS and other atomics, memory barriers, compiler barriers and other magic. <br>
<br>
<BR><br>
<br>
<br>
<br>
<br>
<br>
<h1>More reading</h1>
<ol><li><a href='http://people.redhat.com/drepper/cpumemory.pdf'>What Every Programmer Should Know About Memory</a>
</li><li><a href='http://msdn.microsoft.com/en-us/magazine/cc163744.aspx'>What Every Dev Must Know About Multithreaded Apps</a>
</li><li><a href='http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html'>The "Double-Checked Locking is Broken" Declaration</a>
</li><li><a href='http://en.wikipedia.org/wiki/Memory_barrier'>http://en.wikipedia.org/wiki/Memory_barrier</a>
</li><li><a href='http://en.wikipedia.org/wiki/Volatile_variable'>http://en.wikipedia.org/wiki/Volatile_variable</a>
</li><li><a href='http://software.intel.com/en-us/blogs/2013/01/06/benign-data-races-what-could-possibly-go-wrong'>Benign data races: what could possibly go wrong?</a>
</li><li><a href='http://www.usenix.org/event/hotpar11/tech/final_files/Boehm.pdf'>How to miscompile programs with “benign” data races</a>
</li><li><a href='http://www.chromium.org/developers/lock-and-condition-variable'>Chrome C++ Lock and ConditionVariable</a>