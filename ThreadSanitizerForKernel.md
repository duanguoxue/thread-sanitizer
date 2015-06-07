# Introduction #

A dynamic data race error detector for Linux kernel.
Currently in development.

Repository: https://github.com/google/ktsan

More extensive documentation is available [here](https://github.com/google/ktsan/blob/tsan-doc/Documentation/ktsan.txt).

Some reports produced by the tool are available [here](ThreadSanitizerForKernelReports.md).
There might be false positive reports.

To symbolize the reports you may use our [symbolizer script](https://code.google.com/p/address-sanitizer/source/browse/trunk/tools/kasan_symbolize.py).
The example of usage can be found on [kasan homepage](https://code.google.com/p/address-sanitizer/wiki/AddressSanitizerForKernel).


# Details #

Some implementation ideas:
  * Make some internal structures per CPU instead of per thread (VC cache, what else?). VCs themselves stay per thread.
  * Monitor some kernel thread scheduler events (thread execution started/stopped on CPU).
  * Disable interrupts during TSan events (kernel scheduler events, synchronization events) (CLI, STI).
  * Use 4 bytes per slot: 1 for thread id, 2 for clock, 1 for everything else (flags, ...).
  * Different threads might have the same thread id (only 256 different values available).
  * When clock overflows it is possible to change thread id and connect "old" and "new" threads with a happens-before relation.
  * Find races in both kmalloc and vmalloc ranges.
  * Use two-level shadow memory mapping scheme for now.
  * Do a flush when we run out of clocks. The flush might work as follows. There is a global epoch variable which is increased during each flush. Each thread have a local epoch variable. When a thread is starting it will flush itself if the thread local epoch is less than the global one.

# Building #

Since ktsan uses compiler instrumentation, a custom GCC is required.
You can get the gcc path [here](https://github.com/xairy/kernel-sanitizers/blob/master/ktsan.gcc.patch).

```
svn checkout svn://gcc.gnu.org/svn/gcc/trunk $GCC_KTSAN
cd $GCC_KTSAN
svn up -r 218317
patch -p0 -i gcc_ktsan.patch 
mkdir build
mkdir install
cd build/
sudo apt-get install flex bison libc6-dev libc6-dev-i386 libgmp3-dev libmpfr-dev libmpc-dev
../configure --enable-languages=c,c++ --disable-bootstrap --enable-checking=no --with-gnu-as --with-gnu-ld --with-ld=/usr/bin/ld.bfd --prefix=$GCC_KTSAN/install/
make -j64
make install
```

Build kernel deb packages with ktsan enabled:
```
git clone https://github.com/google/ktsan.git ktsan
svn checkout http://address-sanitizer.googlecode.com/svn/trunk/vagrant_kasan vagrant
cp vagrant/kernel_config ktsan/.config
cd ktsan/
make oldconfig  # select KTSAN option
make CC='$GCC_KTSAN/install/bin/gcc' -j64 deb-pkg LOCALVERSION=-tsan
```

The deb packages will appear in the ktsan parent directory.

To use the current non-stable development version, checkout tsan-dev branch instead.

Install VirtualBox (https://www.virtualbox.org/wiki/Downloads) and Vagrant (https://www.vagrantup.com/downloads.html).
It's better to download newer versions from their official sites other than use the ones from the apt repository.
These instructions were written for VirtualBox 4.3.16 and Vagrant 1.6.5.

Copy new kernel to vagrant:
```
cd vagrant/
vagrant up
vagrant ssh-config > ssh_config
scp -F ./ssh_config linux-* default:/home/vagrant/
vagrant ssh -c "sudo dpkg -i linux-headers-3.16.0-tsan_3.16.0-tsan-2_amd64.deb && sudo dpkg -i linux-libc-dev_3.16.0-tsan-2_amd64.deb && sudo dpkg -i linux-image-3.16.0-tsan_3.16.0-tsan-2_amd64.deb"
vagrant reload
```


# TODO #

  1. Handle more synchronization primitives
  * spinlock (done)
  * rwlock (done)
  * sema (via spinlocks)
  * rwsem (done)
  * completion (via spinlocks)
  * mutex (done)
  * ww\_mutex?
  * atomics (done)
  * atomics + memory barriers
  * atomic bitops (done)
  * thread start/join (via spinlocks and completions)
  * rcu, rcu\_bh, rcu\_sched (done)
  * srcu
  * bit\_lock (via bitops)
  1. Intercept memcpy and friends
  1. Support Non SMP build with ktsan
  1. Fix false negatives in tests
  1. Handle events from interrupts (started in tsan-dev branch)
  1. Use READ\_ONCE/ASSIGN\_ONCE to fix races instead of atomics (https://lkml.org/lkml/2014/12/3/890)