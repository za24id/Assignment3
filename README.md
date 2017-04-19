## Pluto   
### 1.0 Introduction  
PLUTO is an automatic parallelization tool based on the polyhedral model which provides an abstraction to perform high-level transformations such as loop-nest optimization and parallelization on affine loop nests. Pluto transforms C source code for coarse-grained parallelism and data locality simultaneously. The core transformation framework mainly works by finding affine transformations for efficient tiling [1]  

--- 
 

### 2.0 Download and Install  
1. Download the Pluto source ` wget https://www.dropbox.com/s/gz3y911x3vb8p6i/pluto-0.11.4.tar?dl=0 `
2. Rename the downloaded file ` mv` *`downloadedfile`* `pluto-0.11.4.tar `
3. Deflate the file ` tar -xvf pluto-0.11.4.tar `
4. Make it your working directory ` cd pluto-0.11.4 `
5. Configure it ` ./configure `
6. Run the make file ` make [-j8] `

**Note**: Rose has all the software required to install and run PLUTO. If running on a different machine, the installation might complain about missing software, such as Makeinfo which is part of the *texinfo* package. You might find out it requires others that I'm not aware of but what's an installation without the software not found message!  

--- 

### 3.0 Usage 

Let's test it on one of the programs provides by PLUTO! 

` ./polycc test/seidel.c --tile --parallel `

Expected output:
```
[pluto] Number of statements: 1
[pluto] Total number of loops: 3
[pluto] Number of deps: 19
[pluto] Maximum domain dimensionality: 3
[pluto] Number of parameters: 2
[pluto] Affine transformations [<iter coeff's> <param> <const>]

T(S1): (t, t+i, 2t+i+j)
loop types (loop, loop, loop)

t1 --> fwd_dep  loop   (band 0)no-ujam
t2 --> fwd_dep  loop   (band 0)no-ujam
t3 --> fwd_dep  loop   (band 0)no-ujam

[Pluto] After tiling:
T(S1): (zT3, zT4, zT5, t, t+i, 2t+i+j)
loop types (loop, loop, loop, loop, loop, loop)

t1 --> fwd_dep  loop   (band 0)no-ujam
t2 --> fwd_dep  loop   (band 0)no-ujam
t3 --> fwd_dep  loop   (band 0)no-ujam
t4 --> fwd_dep  loop   (band 0)no-ujam
t5 --> fwd_dep  loop   (band 0)no-ujam
t6 --> fwd_dep  loop   (band 0)no-ujam

[Pluto] After tile scheduling:
T(S1): (zT3+zT4, zT4, zT5, t, t+i, 2t+i+j)
loop types (loop, loop, loop, loop, loop, loop)

[pluto] using statement-wise -fs/-ls options: S1(4,6), 
[Pluto] Output written to seidel.pluto.c
```   

If applying PLUTO on other programs, you might see `Error extracting polyhedra from source file` that's because it couldn't build one, which is normal.

--- 
### 4.0 Results

We are comparing the original loop from seidel.c with the loop after transformation from seidel.pluto.c 

#### 4.1 Before 
```
#pragma scop
    for (t=0; t<=T-1; t++)  {
        for (i=1; i<=N-2; i++)  {
            for (j=1; j<=N-2; j++)  {
                a[i][j] = (a[i-1][j-1] + a[i-1][j] + a[i-1][j+1]
                        + a[i][j-1] + a[i][j] + a[i][j+1]
                        + a[i+1][j-1] + a[i+1][j] + a[i+1][j+1])/9.0;
            }
        }
    }
#pragma endscop
```
#### 4.2 After
```
/* Start of CLooG code */
if ((N >= 3) && (T >= 1)) {
  for (t1=0;t1<=floord(2*T+N-4,32);t1++) {
    lbp=max(ceild(t1,2),ceild(32*t1-T+1,32));
    ubp=min(min(floord(T+N-3,32),floord(32*t1+N+29,64)),t1);
#pragma omp parallel for private(lbv,ubv,t3,t4,t5,t6)
    for (t2=lbp;t2<=ubp;t2++) {
      for (t3=max(ceild(64*t2-N-28,32),t1);t3<=min(min(min(min(floord(T+N-3,16),floord(32*t1-32*t2+N+29,16)),floord(32*t1+N+60,32)),floord(64*t2+N+59,32)),floord(32*t2+T+N+28,32));t3++) {
        for (t4=max(max(max(32*t1-32*t2,32*t2-N+2),16*t3-N+2),-32*t2+32*t3-N-29);t4<=min(min(min(min(T-1,32*t2+30),16*t3+14),32*t1-32*t2+31),-32*t2+32*t3+30);t4++) {
          for (t5=max(max(32*t2,t4+1),32*t3-t4-N+2);t5<=min(min(32*t2+31,32*t3-t4+30),t4+N-2);t5++) {
            for (t6=max(32*t3,t4+t5+1);t6<=min(32*t3+31,t4+t5+N-2);t6++) {
              a[(-t4+t5)][(-t4-t5+t6)] = (a[(-t4+t5)-1][(-t4-t5+t6)-1] + a[(-t4+t5)-1][(-t4-t5+t6)] + a[(-t4+t5)-1][(-t4-t5+t6)+1] + a[(-t4+t5)][(-t4-t5+t6)-1] + a[(-t4+t5)][(-t4-t5+t6)] + a[(-t4+t5)][(-t4-t5+t6)+1] + a[(-t4+t5)+1][(-t4-t5+t6)-1] + a[(-t4+t5)+1][(-t4-t5+t6)] + a[(-t4+t5)+1][(-t4-t5+t6)+1])/9.0;;
            }
          }
        }
      }
    }
  }
}
/* End of CLooG code */
```
 
--- 
### 5.0 About PLUTO
#### 5.1 How does it work  
Pluto applies an automatic transformation framework to optimize arbitrarily nested loop sequences with affine dependences for parallelism and locality simultaneously. The approach finds good tiling hyperplanes by embedding a powerful and versatile cost function into an Integer Linear Programming formulation. These tiling hyperplanes are used for communication-minimized coarse-grained parallelization as well as for locality optimization. The approach enables the minimization of inter-tile communication volume in the processor space, and minimization of reuse distances for local execution at each node. Programs requiring one-dimensional versus multi-dimensional time schedules (with scheduling- based approaches) are all handled with the same algorithm. Synchronization-free parallelism, permutable loops or pipelined parallelism at various levels can be detected.  [2]


#### 5.2 Why was the tool developed  
The developers of the tool claim that although a significant body of research has addressed affine scheduling and partitioning, the problem of automatically finding good affine transforms for communication-optimized coarse-grained parallelization together with locality optimization for the general case of arbitrarily-nested loop sequences remains a challenging problem.Therefore they propose a cost model to drive automatic transformation in the polyhedral model. [2]
  
--- 

### 6.0 Connections  
Preparing for this assignment, one of the tools I experimented with is The autoparallelizing compiler for shared-memory computers (AESOP) that is designed to handle real-world code rather than small, simple kernels. For example, AESOP can compile SPEC2006 and OMP2001 benchmarks, and automated test suite which consists of over 2 million lines of code. According to the AESOP installation instructions, it requires LLVM 3.3 and Clang 3.3 so here are the steps I took to compile and install it.


```
svn co http://llvm.org/svn/llvm-project/llvm/tags/RELEASE_33/final
mv final/ llvm-3.3
cd llvm-3.3/tools/
svn co http://llvm.org/svn/llvm-project/cfe/tags/RELEASE_33/final
mv final/ clang
PREFIX="$HOME/opt"
mkdir $PREFIX
export PATH=$PREFIX/bin:$PATH
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$PREFIX/lib
cd opt
mkdir bin
cd bin 
ln -s /usr/bin/ld.gold ld 
LLVM_SRC="$HOME/llvm-3.3"
AESOP_SRC="$HOME/aesop"
cd $LLVM_SRC
-p0 < $AESOP_SRC/patches/aesop-lto-for-3.3.patch
   patching file tools/lto/LTOCodeGenerator.cpp
   patching file tools/lto/LTOCodeGenerator.h
cd $LLVM_SRC
./configure --prefix=$PREFIX --enable-optimized --enable-assertions --enable-pic --with-binutils-include=/usr/include
make -j8 && make install
cd $AESOP_SRC
./configure --prefix=$(llvm-config --prefix) --with-llvmobj=$(llvm-config --obj-root) --with-llvmsrc=$(llvm-config --src-root) --includedir=$(llvm-config --includedir)
make -j8 && make install
aesopcc -m32 -O3 -I. sample.c -o sample.o
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
```

As you can see, although aesop was able to recognize and parallelize loops, it issued `llc: error: invalid target 'c'.`  The reason for the error is in LLVM 3.1 the C backend has been removed. 

My alternative was to use aesopgcc to compile C++ instead of aesopcc for C. Using aesopgcc requires Dragonegg 3.3. I downloaded and attempted to install it but immediately received 

```
Compiling utils/TargetInfo.cpp
Linking TargetInfo
TargetInfo.o:TargetInfo.cpp:function main: error: undefined reference to 'llvm::Triple::normalize(llvm::StringRef)'
collect2: error: ld returned 1 exit status
Makefile:136: recipe for target 'TargetInfo' failed
make: *** [TargetInfo] Error 1
```
The error is due to declaring Triple incorrectly. I tried fixing that and did but ran into a bunch of other problems which led me to work on PLUTO instead, which is a similar tool in the fact that it performs auto parallelization
 
### 7.0  Sources
1. http://pluto-compiler.sourceforge.net 
2. Automatic Transformations for Communication-Minimized Parallelization and Locality Optimization in the Polyhedral Model
