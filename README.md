# Assignment3
Download and install Cmake
Download and install LLVM
svn co http://llvm.org/svn/llvm-project/llvm/tags/RELEASE_33/final

mv final/ llvm-3.3
cd llvm-3.3/tools/


svn co http://llvm.org/svn/llvm-project/cfe/tags/RELEASE_33/final

mv final/ llvm-3.3/tools/

cd llvm-3.3/tools/

mv final/ clang


PREFIX="$HOME/opt"


mkdir $PREFIX
export PATH=$PREFIX/bin:$PATH
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$PREFIX/lib


cd opt
mkdir bin


ln -s /usr/bin/ld.gold ld 

cd


LLVM_SRC="$HOME/llvm-3.3"

AESOP_SRC="$HOME/aesop"


cd $LLVM_SRC

zmalatta@rose:~/llvm-3.3$ patch -p0 < $AESOP_SRC/patches/aesop-lto-for-3.3.patch
patching file tools/lto/LTOCodeGenerator.cpp
patching file tools/lto/LTOCodeGenerator.h


cd $LLVM_SRC
/configure --prefix=$PREFIX --enable-optimized --enable-assertions --enable-pic --with-binutils-include=/usr/include
make -j8 && make install



 cd $AESOP_SRC
  ./configure --prefix=$(llvm-config --prefix) --with-llvmobj=$(llvm-config --obj-root) --with-llvmsrc=$(llvm-config --src-root) --includedir=$(llvm-config --includedir)
     
  make -j8 && make install


  zmalatta@rose:~/cs553/assign_3$ aesopcc -m32 -O3 -I. sample.c -o sample.o
Invoking AESOP.
	Writing temporary bytecode...
	Running AESOP optimizations...
make: Entering directory '/tmp/6165.aesop_work'
	 Loop main:for.body has a loop carried memory dependence, hence will not be parallelized 
	 Loop main:for.cond2 carries no dependence, hence is being parallelized 
	Num Loops Parallelized 1
make: Leaving directory '/tmp/6165.aesop_work'
	Optimizations completed.
llc: error: invalid target 'c'.
WARNING: C file generation has failed.
/usr/bin/ld: cannot find ?y?: No such file or directory
/usr/bin/ld: error: Failed to delete '?y?': ?y?: can't get status of file: No such file or directory
clang: error: linker command failed with exit code 1 (use -v to see invocation)



### 
```
zmalatta@rose:~$ llc --version
LLVM (http://llvm.org/):
  LLVM version 3.3
  Optimized build with assertions.
  Built Apr 17 2017 (11:31:30).
  Default target: x86_64-unknown-linux-gnu
  Host CPU: x86-64

  Registered Targets:
    aarch64  - AArch64
    arm      - ARM
    cpp      - C++ backend
    hexagon  - Hexagon
    mblaze   - MBlaze
    mips     - Mips
    mips64   - Mips64 [experimental]
    mips64el - Mips64el [experimental]
    mipsel   - Mipsel
    msp430   - MSP430 [experimental]
    nvptx    - NVIDIA PTX 32-bit
    nvptx64  - NVIDIA PTX 64-bit
    ppc32    - PowerPC 32
    ppc64    - PowerPC 64
    sparc    - Sparc
    sparcv9  - Sparc V9
    systemz  - SystemZ
    thumb    - Thumb
    x86      - 32-bit X86: Pentium-Pro and above
    x86-64   - 64-bit X86: EM64T and AMD64
    xcore    - XCore
```
