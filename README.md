A patch file to extract wave functions from cFAC
=================================================
[Flexible Atomic Code (FAC)](https://github.com/flexible-atomic-code/fac) is a package for various quantum calculations in atomic processes, and [cFAC](https://github.com/fnevgeny/cfac) is its C implimentation. The export function of cFAC is not exporting wave functions properly, and this patch fixes this problem. See [1] for the computation of Migdal effect using the patched cFAC.

Compilation
-----------

1. To compile cFAC in Debian/Ubuntu, the following packages + latex are needed,
```bash
apt install build-essential gfortran automake doxygen gfortran libgsl0-dev libsqlite3-dev help2man
```

2. Then, go to the top directory of the cFAC source.
```bash
cd /path/to/cfac-src
```

3. Download the patch from this repository and patch the source.
```bash
patch -p0 -b < /location/of/patch/cfac.patch
```

4. Compile and install cFAC.
```bash
make
./configure [--with-cpc-modules] [--prefix=/path/to/install/directory] ...
make
make install
```

Usage
-----

To get single-configuration Dirac-Hartree-Fock wave functions, you can modify and use the following input of sfac.
Each function is documented in Sec.3 of the cFAC manual.

```
SetAtom('Al')
Closed('1s','2s','2p','3s')
Config('gnd','3p1')
OptimizeRadial(['gnd'])
RefineRadial()
WaveFuncTable('wf1.out',1,-1)
WaveFuncTable('wf2.out',2,-1)
WaveFuncTable('wf3.out',2,1)
WaveFuncTable('wf4.out',2,-2)
WaveFuncTable('wf5.out',3,-1)
WaveFuncTable('wf6.out',3,1)
WaveFuncTable('wf7.out',3,-2)
WaveFuncTable('free.out',0,1,10.)
...
```

The outputs you get have the following structure:

#### Bound electron: `wf1.out`
```
#      n =  1
#  kappa = -1
# energy = -1.55324520E+03
#     vc = -3.73716874E+03


0    7.69230769E-08 -2.204634E-02  1.398063E-07  0.000000E+00  3.626484E-02
1    8.43412003E-08 -2.417239E-02  1.532881E-07  6.976009E-07  3.626484E-02
2    9.24746099E-08 -2.650344E-02  1.680691E-07  1.462467E-06  3.307500E-02
...
```
The first and the second lines are the relativistic quantum numbers. The third line is the eigenenergy in eV and the forth line is the expectation value of the potential in eV.

The columns are
`[numbering] [r in 1/(me*alpha)] [r*Vc in (alpha)] [r*U in (alpha)] [P] [Q]`
where `Vc` is the optimized central potential, `U` is the direct interaction part of potential and `P` and `Q` are the wave functions.
Notice that `P` and `Q` are dimensionless quantities.

#### Free electron: free.out
```
#      n =  0
#  kappa =  1
# energy =  1.00000000E+01
#     vc = -1.83507095E+02
#  ilast = 265
#  phase =  1.43270547E+00


0    7.69230769E-08 -2.204634E-02  1.398063E-07 -0.000000E+00  3.174156E-08
1    8.43412003E-08 -2.417239E-02  1.532881E-07  5.094112E-13  3.174156E-08
2    9.24746099E-08 -2.650344E-02  1.680691E-07  1.077000E-12  3.480005E-08
...
```
The first to forth lines are the same as the bound ones.
The fifth line is explained below and the sixth line is the phase shift.

The first `0<=[numbering]<=ilast` rows describe small r behavior and `ilast<[numbering]` rows describe large r behavior.
For small r, the columns are the same as ones for the bound states.
For large r, the columns are
`[numbering] [r in 1/(me*alpha)] [p] [theta] [q1] [q2]`
where
`P = p * sin(theta)` and
`Q = q1 * cos(theta) + q2 * sin(theta)`.

Notice that the free electron wave function is normalized so that P has the asymptotic amplitude of `sqrt(k/energy)`, where k is the momentum of the electron.
To get the normalization of [1], you need to multiply `2 / alpha / sqrt(me)` to P and Q.

Reference
---------
[1] M. Ibe, W. Nakano, Y. Shoji and K. Suzuki, "MIgdal Effect in Dark Matter Direct Detection Experiments", 1007/JHEP03 (2018) 194.