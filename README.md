# Intro

_Warning, this project has atomics disabled and thus will not produce
thread-safe binaries with no warnings, see the Safety section for details_

This project is just where I do my RISC-V work. This is capable of building
`musl` and `compiler-rt` using the `-march=rv64i` instruction set. This means
there are no divides, multiplies, atomics, or anything! This will be able to
generate single-threaded binaries using the most basic RISC-V 64-bit
instruction set! This is a perfect place to start if you're writing an emulator
and want to keep things as simple as possible.

# Usage

This project assumes you have `clang` 10 installed and in your path. If you
want to use a different version of `clang`, change `CLANG_VERSION` in the root
level `Makefile`. Note that some versions of `clang` may not support targeting
`riscv64`.

Simply run `make` to have a toolchain created for you. Run `make clean` to
remove the toolchain if desired.

## Test Application

There is a test C application in the `test_app` folder, giving an example of
how to build and use this created toolchain.

```
pleb@gamey:~/riscv/test_app$ make
clang-10 -Wall -Werror -O2 -g -nostdlib -target riscv64 -march=rv64i -I/home/pleb/riscv/musl_build/install/include -c test_app.c
ld.lld-10 -o test_app \
        /home/pleb/riscv/musl_build/install/lib/crt1.o \
        /home/pleb/riscv/musl_build/install/lib/libc.a \
        /home/pleb/riscv/compiler-rt_build/install/lib/riscv64/libclang_rt.builtins-riscv64.a \
        *.o
pleb@gamey:~/riscv/test_app$ file ./test_app
./test_app: ELF 64-bit LSB executable, UCB RISC-V, version 1 (SYSV), statically linked, with debug_info, not stripped
pleb@gamey:~/riscv/test_app$ qemu-riscv64 ./test_app
Hello world!
```

How cool is that!?

# Environment

All of this testing has been done using the official Debian pacakges directly
from the LLVM project.

# Safety

This project is not thread safe in its default configuration. Atomics are not
implemented in the base ISA and thus I had to disable atomics in MUSL. The way
I disabled atomics in MUSL means that locks will silently pass. This will lead
to really confusing bugs, these binaries are meant to be used in an environment
where threads are not possible to create (my emulator).

If you want to get back thread safety change the `#if 0` in
`musl-1.2.0/arch/riscv64/atomic_arch.h` to an `#if 1`. This will then give
compilation errors unless the `march` flag passed to MUSL during building is
changed from `-march=rv64i` to `-march=rv64ia`, to allow the atomics RISC-V
extension.

# Patches

See the `patches` folder for more info.

## MUSL

Basis: `musl-1.2.0`

MUSL was slightly modified to have a atomic-less support, abliet unsafely and without checks, check the `#if 0` block in `musl-1.2.0/arch/riscv64/atomic_arch.h`. Other than that, it's identical to the official `musl-1.2.0` release.

## `compiler-rt`

Basis: https://github.com/llvm-mirror/compiler-rt @ `69445f095c22aac2388f939bedebf224a6efcdaf`

`compiler-rt` had a bug with `rv64i` where it would produce multiples recursively in the C implementation of `__muldi3`. To avoid this, a new `muldi3.S` file has been created under the `lib/builtins/riscv64` folder. It has then been added to the `CMakeLists.txt` to ensure it gets used and replaces the `__muldi3` C implementation.

