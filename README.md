## Pluto   
### 1.0 Introduction  
PLUTO is an automatic parallelization tool based on the polyhedral model. The polyhedral model for compiler optimization provides an abstraction to perform high-level transformations such as loop-nest optimization and parallelization on affine loop nests. Pluto transforms C programs from source to source for coarse-grained parallelism and data locality simultaneously. The core transformation framework mainly works by finding affine transformations for efficient tiling [1]   
--- 

### 2.0 Download and Install  
* Download the Pluto source ` wget https://www.dropbox.com/s/gz3y911x3vb8p6i/pluto-0.11.4.tar?dl=0 `
* Rename the downloaded file ` mv` *`downloadedfile`* `pluto-0.11.4.tar `
* Deflate the file ` tar -xvf pluto-0.11.4.tar `
* Make it your working directory ` cd pluto-0.11.4 `
* Configure it ` /configure `
* If not already installed, ` sudo apt-get install texinfo `
* Run the make file ` make [-j4] `

--- 

### 3.0 Usage 

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


### 6.0  Sources
1. http://pluto-compiler.sourceforge.net 
2. Automatic Transformations for Communication-Minimized Parallelization and Locality Optimization in the Polyhedral Model
