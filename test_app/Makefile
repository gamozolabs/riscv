CLANG_VERSION = 10

MUSL_INSTALL_PATH = /home/pleb/riscv/musl_build/install
COMPILER_RT_PATH  = /home/pleb/riscv/compiler-rt_build/install/lib/riscv64/libclang_rt.builtins-riscv64.a

CFLAGS  = -Wall -Werror -O2 -g -nostdlib -target riscv64 -march=rv64i
CFLAGS += -I$(MUSL_INSTALL_PATH)/include

all: test_app
	
test_app: test_app.o
	ld.lld-$(CLANG_VERSION) -o test_app \
		$(MUSL_INSTALL_PATH)/lib/crt1.o \
		$(MUSL_INSTALL_PATH)/lib/libc.a \
		$(COMPILER_RT_PATH) \
		*.o

test_app.o: test_app.c
	clang-$(CLANG_VERSION) $(CFLAGS) -c test_app.c

clean:
	rm -f test_app *.o

