## Pluto   
### Introduction  
PLUTO is an automatic parallelization tool based on the polyhedral model. The polyhedral model for compiler optimization provides an abstraction to perform high-level transformations such as loop-nest optimization and parallelization on affine loop nests. Pluto transforms C programs from source to source for coarse-grained parallelism and data locality simultaneously. The core transformation framework mainly works by finding affine transformations for efficient tiling [1]   

### Download and Install  
* Download the Pluto source ` wget https://www.dropbox.com/s/gz3y911x3vb8p6i/pluto-0.11.4.tar?dl=0 `
* Rename the downloaded file ` mv` *`downloadedfile`* `pluto-0.11.4.tar `
* Deflate the file ` tar -xvf pluto-0.11.4.tar `
* Make it your working directory ` cd pluto-0.11.4 `
* Configure it ` /configure `
* If not already installed, ` sudo apt-get install texinfo `
* Run the make file ` make [-j4] `
* Test it ` ./polycc test/seidel.c --tile `

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

[pluto] using statement-wise -fs/-ls options: S1(4,6), 
[Pluto] Output written to seidel.pluto.c

```   
 
--- 

### How does it work  
an automatic transformation framework to optimize arbitrarily nested loop sequences with affine dependences for parallelism and locality simultaneously. The approach finds good tiling hyperplanes by embedding a powerful and versatile cost function into an Integer Linear Programming formulation. These tiling hyperplanes are used for communication-minimized coarse-grained parallelization as well as for locality optimization. The approach enables the minimization of inter-tile communication volume in the processor space, and minimization of reuse distances for local execution at each node. Programs requiring one-dimensional versus multi-dimensional time schedules (with scheduling- based approaches) are all handled with the same algorithm. Synchronization-free parallelism, permutable loops or pipelined parallelism at various levels can be detected.  

--- 

### Why was the tool developed  
The developers of the tool claim that although a significant body of research has addressed affine scheduling and partitioning, the problem of automatically finding good affine transforms for communication-optimized coarse-grained parallelization together with locality optimization for the general case of arbitrarily-nested loop sequences remains a challenging problem.[2]  
  
--- 


## Source
1. http://pluto-compiler.sourceforge.net 
2. Automatic Transformations for Communication-Minimized Parallelization and Locality Optimization in the Polyhedral Model
