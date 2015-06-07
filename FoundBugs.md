# Data races #
  * [106 data races in Chromium](https://code.google.com/p/chromium/issues/list?can=1&q=Stability%3DThreadSanitizer++v2+-status%3Dduplicate+%22threadsanitizer+data+race%22&sort=opened+modified&colspec=ID+Pri+M+Iteration+ReleaseBlock+Cr+Status+Owner+Summary+OS+Modified+Opened&x=m&y=releaseblock&cells=tiles)

  * [20+ races in Firefox](https://bugzilla.mozilla.org/show_bug.cgi?id=tsan-maintenance)

  * [70 data races in Go standard library](https://github.com/golang/go/issues?&q=is%3Aissue+label%3ARaceReport)

  * [37 data races in WebRTC](https://code.google.com/p/webrtc/issues/list?can=1&q=tsan+errors)

  * [A data race in OpenSSL](http://marc.info/?l=openssl-cvs&m=134947022905662)

  * 2 data races in libgomp ([1](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=40362) [2](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=59194))

  * [1 data race in LLVM](http://llvm.org/viewvc/llvm-project?view=revision&revision=203138)

  * [1 bug in LLVM code generation, leads to data races in user code](http://llvm.org/bugs/show_bug.cgi?id=13691)

  * [1 bug in GCC code generation, leads to data races in user code](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=48076)

  * 10+ data races and mutex errors in an unreleased Cloudera C++ software project

  * 20+ data races in [libcds - a concurrent data structure library](https://github.com/khizmax/libcds)

  * 1000+ C++ data races in Google internal codebase

  * 200+ Go data races in Google internal codebase

plus some others that we don't know about.

# Deadlocks #
  * 1 (so far) real deadlock in Chromium: http://crbug.com/377420