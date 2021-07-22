
//===================================================== Input ===========================================================================\\
Program requires two file groups to start:
- SLAE files
- Mesh files

SLAE files:
kuslau
pr
idi
di
ig
jg
ijg
gg


Mesh files:
inftry.dat
tsize3d_.dat
xyz.dat
nver.dat
nvkat.dat
dpr3D
mu3D
Sig3d
L13d.dat
edges.dat
nodesforedges.dat

//===================================================== Output ===========================================================================\\
The result of this program is in three files:
v3.dat - binary file with solution vector. If there is N equations in the system, than v3.dat will contain N double records (8 bytes).
kit - text file with common info: residual, required residual, iterations count, time spent to solve the system (in seconds)
logharm3dCalc - log file

//===================================================== SLAE file formats ==================================================================\\
SLAE file formats (example below, better learn it):

//===================================================== kuslau ===============\\
kuslau - text - basic info 
<equations count (doubled edges count)> // let's denote it as N
<requested residual (usually about 1e-4)>
<max. iterations count>

//===================================================== pr ===================\\
pr - binary - right hand side vector
contains N double records (8 bytes)

//===================================================== idi ==================\\
idi - binary - diagonal index array
contains N+1 integer records (4 bytes)
idi[i+1] - idi[i] represents amount of nonzero values for i'th 2x2 diagonal block. idi[0] is always 1

//===================================================== di ===================\\
di - binary - nonzero values for diagonal blocks
contains idi[N]-1 double records (8 bytes)

//===================================================== ig ===================\\
ig - binary - index array (outside diagonal)
contains N+1 integer records (4 bytes)
ig[i+1] - ig[i] represents amount of nonzero blocks in i'th block-row. ig[0] and ig[1] is always 1

//===================================================== jg ===================\\
jg - binary - block-columns numbers
contains ig[N] integer records (4 bytes)
jg[i] represents block-column number for i'th nonzero block.

//===================================================== ijg ==================\\
ijg - binary - nonzero blocks index array
contains ig[N] integer records (4 bytes)
ijg[i+1] - ijg[i] represents amount of nonzero values for i'th 2x2 nonzero block. ijg[0] is always 1

//===================================================== gg ===================\\
gg - binary - nonzero values for nonzero blocks
contains ijg[jg[N]]-1 double records (8 bytes)
arary contains all nonzero values

//===================================================== Example ==============\\
Example:
|P11  0  | P21   0  | P31  0  |  0  0    | P51  0  |   |q1 |   |RHS1 |
| 0  P11 |  0   P21 |  0  P31 |  0  0    |  0   P51|   |q2 |   |RHS2 |
|--------------------------------------------------|   |---|   |-----|
|P21  0  | P22 -C22 |  0   0  | P42 -C42 | P52 -C52|   |q3 |   |RHS3 |
| 0  P21 | C22  P22 |  0   0  | C42  P42 | C52  P52|   |q4 |   |RHS4 |
|--------------------------------------------------|   |---|   |-----|
|P31  0  |  0    0  | P33  0  |  0   0   | P53  0  | X |q5 | = |RHS5 |
| 0  P31 |  0    0  |  0  P33 |  0   0   |  0   P53|   |q6 |   |RHS6 |
|--------------------------------------------------|   |---|   |-----|
| 0   0  | P42 -C42 |  0   0  | P44 -C44 | P54 -C54|   |q7 |   |RHS7 |
| 0   0  | C42  P42 |  0   0  | C44  P44 | C54  P54|   |q8 |   |RHS8 |
|--------------------------------------------------|   |---|   |-----|
|P51  0  | P52 -C52 | P53  0  | P54 -C54 | P55 -C55|   |q9 |   |RHS9 |
| 0  P51 | C52  P52 |  0  P53 | C54  P54 | C55  P55|   |q10|   |RHS10|

kuslau :
10
1e-4
10000

idi : 1 2 4 5 7 9
di : P11 P22 C22 P33 P44 C44 P55 C55
ig : 1 1 2 3 4 8
jg : 1 1 2 1 2 3 4
ijg : 1 2 3 5 6 8 9 11
gg : P21 P31 P42 C42 P51 P52 C52 P53 P54 C54
pr : RHS1 RHS2 RHS3 RHS4 RHS5 RHS6 RHS7 RHS8 RHS9 RHS10



//===================================================== Mesh file formats ==================================================================\\
Mesh files (example below, better learn it):

//===================================================== inftry.dat =================\\
inftry.dat - text - basic info
File should contain following text:
 ISLAU= 0 INDKU1= 0 INDFPO= 0
KUZLOV= <nodes count (N)>   KPAR= <elements count (M)>    KT1= <boundary nodes count (BN)>   KTR2= 0   KTR3= 0
   KT8= 0   KT9= 0
KISRS1= 0 KISRS2= 0 KISRS3= 0   KBRS= 0
   KT7= 0   KT10= 0  KTR4= 0  KTSIM= 0
   KT6= 0
   
//===================================================== tsize3d_.dat ===============\\
tsize3d_.dat - text - additional info
File should contain following text:
0
<edges count (EC)>

//===================================================== xyz.dat ====================\\
xyz.dat - binary - nodes x,y,z coordinates
file contains N*3 double records (8 bytes)
for each node: <X> <Y> <Z>

//===================================================== nver.dat ===================\\
nver.dat - binary - describing cells with nodes
file contains M*14 integer records (4 bytes)
for each element: <node 1> <node 2> <node 3> <node 4> <node 5> <node 6> <node 7> <node 8> <0> <0> <0> <0> <0> <0>
Nodes numeration starts with 1
Inside element nodes are numerated as folloows:

                                                            
                                  7-----------------------8
                                 /|                      /|
                                / |                     / |
                               /  |                    /  |
                              /   |                   /   |
                             /    |                  /    |
                            /     |                 /     |
                           /      |                /      |
                          5-----------------------6       |
                          |       |               |       |
                          |       3---------------|-------4    
Z ↑                       |      /                |      /     
  |                       |     /                 |     /      
  |      / Y              |    /                  |    /       
  |     /                 |   /                   |   /        
  |    /                  |  /                    |  /         
  |   /                   | /                     | /          
  |  /                    |/                      |/           
  | /                     1-----------------------2            
  |/                                                           
  -------------->                                              
               X                                               

//===================================================== nvkat.dat ===============\\
nvkat.dat - binary - cells material numbers
file contains M integer records (4 bytes)
for each element: <material number>
Materials numeration starts with 1
Materials must be numerated as follows:
material 1 - air
material 2 - lower layer material
...
material LL - upper layer material (contacts with air)
material LL + 1 - first object (3D heterogeneity) material
...
material LO - last object (3D heterogeneity) material

//===================================================== about materials ===============\\
dpr3D, mu3D, Sig3d contains material parameters for heterogeneties (het) (second column) and for environment (env) (third column). 
Layer materials are environment. Layer materials 2nd and 3rd columns should be equal. 
Object materials are heterogeneties. Object materials 2nd column should contain object material parameter and 3rd column contains material parameter of layer that the object placed into. 

//===================================================== dpr3D ===============\\
dpr3D - text - relative dielectric permeability (Eps) for each material
for each material : <material number (starts with 1)> <Eps (het)> <Eps (env)>
Usually whole file contains zeros

//===================================================== mu3D ================\\
mu3D - text - relative magnetic permeability (Mu) for each material
for each material : <material number (starts with 1)> <Mu (het)> <Mu (env>
Usually whole file contains 1 values

//===================================================== Sig3d ===============\\
Sig3d - text - conductivity (Mu) for each material
for each material : <material number (starts with 1)> <Mu (het)> <Mu (env>

//===================================================== L13d.dat ===============\\
L13d.dat - binary - numbers of boundary nodes
file contains NB integer records (4 bytes)
for each node that lays on external bound: <node number (starts with 1)>

//===================================================== nodesforedges.dat ===============\\
nodesforedges.dat - binary - describing edges with node numbers
file contains EC*2 integer records (4 bytes)
for each edge: <node 1> <node 2>
Nodes numeration start with 1

//===================================================== edges.dat ===============\\
edges.dat - binary - describing cells with edges
file contains M*25 integer records (4 bytes)
for each element: <edge 1> <edge 2> ... <edge 12> <0> <0> <0> <0> <0> <0> <0> <0> <0> <0> <0> <0> <1>
Edges numeration starts with 1
Inside element edges are numerated as follows:

                                                            
                                  ---------4---------------
                                 /|                      /|
                                / |                     / |
                               /  |                    /  |
                              6   |                   8   |
                             /    11                 /    12
                            /     |                 /     |
                           /      |                /      |
                          -----------3-------------       |
                          |       |               |       |
                          |       ---------2------|-------     
Z ↑                       |      /                |      /     
  |                       9     /                 |     /      
  |      / Y              |    5                  10   7       
  |     /                 |   /                   |   /        
  |    /                  |  /                    |  /         
  |   /                   | /                     | /          
  |  /                    |/                      |/           
  | /                     -----------1-------------            
  |/                                                           
  -------------->                                              
               X                                           

		
//===================================================== Example ===============\\	   
Example:
  
                                                             
    Z ↑                               16-------11-------------17----------12-----------18
      |                              /|                      /|                       /|
      |                             / |                     / |                      / |
      |                            /  |                    /  |                     /  |
      |                           19  |        3 (1)      20  |         4 (1)      21  |
      |                          /    31                 /    32                  /    33
      |                         /     |                 /     |                  /     |
      |                        /      |                /      |                 /      |
      |                       13---------9------------14----------10-----------15      |
      |                       |       |               |       |                |       |
  100 -                       |       10-------7------|-------11---------8-----|-------12  
      |                       |      /|               |      /|                |      /|
      |                       28    / |               29    / |                30    / |
      |                       |    /  |               |    /  |                |    /  |
      |                       |   16  |               |   17  |                |   18  |
      |                       |  /    25              |  /    26               |  /    27
      |                       | /     |               | /     |                | /     |
      |            / Y        |/      |               |/      |                |/      |
      |           /           7----------5------------8-----------6------------9       |
      |          /            |       |               |       |                |       |
    0 -         - 10000       |       4--------3------|-------5----------4-------------6   
      |        /              |      /                |      /                 |      /    
      |       /               22    /                 23    /                  24    /     
      |      /                |    13       1 (2)     |    14        2 (3)     |    15     
      |     /                 |   /                   |   /                    |   /       
      |    /                  |  /                    |  /                     |  /        
      |   /                   | /                     | /                      | /         
      |  /                    |/                      |/                       |/          
      | -  5000               1----------1------------2-----------2------------3            
      |/                                                           
 -100 ------------------------|-----------------------|------------------------|--------->                                              
                            1000                     2000                     3000      X                                  
			   
	


inftry.dat:
 ISLAU= 0 INDKU1= 0 INDFPO= 0
KUZLOV= 18   KPAR= 4    KT1= 18   KTR2= 0   KTR3= 0
   KT8= 0   KT9= 0
KISRS1= 0 KISRS2= 0 KISRS3= 0   KBRS= 0
   KT7= 0   KT10= 0  KTR4= 0  KTSIM= 0
   KT6= 0
tsize3d_.dat:
0
33

xyz.dat:
1000  5000 -100
2000  5000 -100
3000  5000 -100
1000 10000 -100
2000 10000 -100
3000 10000 -100
1000  5000 0
2000  5000 0
3000  5000 0
1000 10000 0
2000 10000 0
3000 10000 0
1000  5000 100
2000  5000 100
3000  5000 100
1000 10000 100
2000 10000 100
3000 10000 100

nver.dat: 
1 2 4 5 7 8 10 11 0 0 0 0 0 0
2 3 5 6 8 9 11 12 0 0 0 0 0 0
7 8 10 11 13 14 16 17 0 0 0 0 0 0
8 9 11 12 14 15 17 18 0 0 0 0 0 0

nvkat.dat:
 2 3 1 1

dpr3D:
1 0 0
2 0 0
3 0 0

mu3D:
1 1 1
2 1 1
3 1 1

Sig3d:
1 1e-8 1e-8
2 100 100
3 20 100

L13d.dat:
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18

edges.dat
1 3  5  7 13 16 14 17 22 23 25 26 0 0 0 0 0 0 0 0 0 0 0 0 1	
2 4  6  8 14 17 15 18 23 24 26 27 0 0 0 0 0 0 0 0 0 0 0 0 1
5 7  9 11 16 19 17 20 28 29 31 32 0 0 0 0 0 0 0 0 0 0 0 0 1
6 8 10 12 17 20 18 21 29 30 32 33 0 0 0 0 0 0 0 0 0 0 0 0 1

nodesforedges.dat
1 2
2 3
4 5
5 6
7 8
8 9
10 11
11 12
13 14
14 15
16 17
17 18
1 4
2 5
3 6
7 10
8 11
9 12
13 16
14 17
15 18
1 7
2 8
3 9
4 10
5 11
6 12
7 13
8 14
9 15
10 16
11 17
12 18
 

