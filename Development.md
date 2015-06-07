# LLVM Development #

To download the latest sources:
```
# cd somewhere
svn co http://llvm.org/svn/llvm-project/llvm/trunk llvm
cd llvm
R=$(svn info | grep Revision: | awk '{print $2}')
(cd tools && svn co -r $R http://llvm.org/svn/llvm-project/cfe/trunk clang)
(cd projects && svn co -r $R http://llvm.org/svn/llvm-project/compiler-rt/trunk compiler-rt)
```

To build (requires cmake>=2.8.8):
```
# in llvm dir
mkdir build && cd build
CC=gcc CXX=g++ cmake -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_ASSERTIONS=ON /path/to/llvm/checkout
make -j10
```

To run tests:
```
cd build
make check-all -j10 # build and run all tests
make check-tsan -j10 # build and run ThreadSanitizer tests
```

Before commit you need to run the presubmit tests (requires gcc>=4.6):
```
cd projects/compiler-rt/lib/tsan
make presubmit
../sanitizer_common/scripts/check_lint.sh
```

Additional build types:
```
cd projects/compiler-rt/lib/tsan

# Debug build of the runtime library:
make DEBUG=1

# Debug build of the runtime library with debug output:
make DEBUG=1 CFLAGS=-DTSAN_DEBUG_OUTPUT=1

# Debug build of the runtime library with extended debug output
# (including memory accesses and function entry/exit events, be careful it can be enormous):
make DEBUG=1 CFLAGS=-DTSAN_DEBUG_OUTPUT=2

# Runtime with stats collection (prints internal stats at exit):
make CFLAGS=-DTSAN_COLLECT_STATS=1
```

To run an arbitrary program with `ThreadSanitizer`:
```
your/fresh/clang test.c -fsanitize=thread -g -O1 -fPIE -pie
```

# GCC Development #

Note that the runtime library is developed in LLVM repository, it is only imported into GCC.

To download the latest sources:
```
svn co svn://gcc.gnu.org/svn/gcc/trunk gcc
```

To build:
```
cd gcc && mkdir build && cd build
../configure --prefix=/path/to/gcc/install
make -j10 && make install -j10
```

If you add '--enable-languages=c,c++ --disable-bootstrap --enable-checking=no' to the configure, the process will be way faster.  Sometimes it's also necessary to add '--with-gnu-as --with-gnu-ld'.  If it can't find some libraries run:
```
sudo apt-get install flex bison libc6-dev libc6-dev-i386 libgmp3-dev libmpfr-dev libmpc-dev
```

To run tests:
```
cd build
make check-gcc
```

To run an arbitrary program with `ThreadSanitizer`:
```
your/fresh/gcc test.c -fsanitize=thread -g -O1 -fPIE -pie
```