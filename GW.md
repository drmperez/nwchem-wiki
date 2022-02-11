# GW

## Overview

Electron attachment and detachment energies can be accurately described
by many-body perturbation theory (MBPT) methods. In particular, the 
*GW* approximation (GWA) to the self-energy is a MBPT method that has 
seen recent interest in its application to molecules due to a
promising cost/accuracy ratio.

The *GW* module implemented in NWChem takes a [DFT](Density-Functional-Theory-for-Molecules.md)
mean-field approximation to the Green's function, *G<sub>0</sub>*, in order to
solve the quasiparticle equation at the one-shot *G<sub>0</sub>W<sub>0</sub>* or
at various *levels* of the eigenvalue self-consistent *GW* approach (ev*GW*).
Since the mean-field orbitals are kept fixed in all these
approaches, the results depend on the actual starting
point *G<sub>0</sub>* (hence, they depend on the exchange-correlation
functional chosen for the underlying DFT calculation).
For example, it has been known that a large fraction of exact exchange
is needed for the accurate prediction of core-level binding energies at the
one-shot *G<sub>0</sub>W<sub>0</sub>* level.

For further theoretical and details about the actual implementation in
NWChem, please refer to the following paper:

  - D. Mejia-Rodriguez, A. Kunitsa, E. Apr&agrave;, N. Govind,
    [Scalable Molecular GW Calculations: Valence and Core Spectra](https://doi.org/10.1021/acs.jctc.1c00738)
    *J. Chem. Theory Comput.* **17**, 7504 (2021).


*GW* input is provided using the compound directive

```
GW
  ...
END
```

The actual *GW* calculation will be performed when the 
input module encounters the [TASK](TASK.md) directive.

```
TASK DFT GW
```

Note that `DFT` must be specified as the underlying *QM* theory before 
`GW`. The [charge](Charge.md), [geometry](Geometry.md), and 
[DFT](Density-Functional-Theory-for-Molecules.md) options are all specified as
normal. 

In addition to an atomic orbital [basis set](Basis.md), the *GW* module **requires** 
an *auxiliary basis set* to be provided in order to fit the four-center 
electron repulsion integrals. The auxliary basis set can have either the
 `cd basis` or `ri basis` names (see also [DFT](Density-Functional-Theory-for-Molecules.md#specification-of-basis-sets-for-the-dft-module)). Three combinations can be obtained:

  - If a `ri basis` is given without a `cd basis`,
    the ground-state DFT will be performed without density fitting,
    and the *GW* task will use the `ri basis` to fit the integrals.
  - If a `cd basis` is given without a `ri basis`,
    **both** DFT and *GW* tasks will be performed using the 
    `cd basis` to fit the integrals.
  - If both `cd basis` and `ri basis` are present, the `cd basis` will be used
    for the DFT task, while the `ri basis` will be used
    for the *GW* task.

## GW Input directive

There are sub-directives which allow for customized *GW* calculations.
The most general *GW* input block directive will look like:

```
GW
  RPA 
  CORE
  EVGW [<integer eviter default 4>]
  EVGW0 [<integer eviter default 4>]
  FIRST <integer first_orbital default 1>
  METHOD [ [analytic] || [cdgw <integer grid_points default 200>] ]
  ETA <real infinitesimal default 0.001> 
  SOLVER [ [newton <integer maxiter default 10> ] || [graph] ]
  STATES [ [ alpha || beta ] [occ <integer number default 1>] [vir <integer default 0>] ]
  CONVERGENCE <real threshold default 0.005> [<string units default ev>]
END
```

The following sections describe these keywords.

### RPA

The keyword `RPA` triggers the computation of the RPA correlation energy.
This adds a little overhead to the [*CD-GW*](#method) approach.

### CORE and FIRST

The `CORE` keyword forces to start counting the [`STATES`](#states)
from the `FIRST` molecular orbital **upwards**. 

The `FIRST` keyword has no meaning without `CORE` specified.

### EVGW and EVGW0

The `EVGW` keyword trigger the partial self-consistnet ev*GW* approach,
where both the Green's function *G* and the screened Coulomb *W* are updated
by using the quasiparticle energies from the previous step in their 
construction. 

Similarly, the `EVGW0` triggers the ev*GW<sub>0</sub>* approach, where
only the Green's function *G* is updated with the quasiparticle energies
of the previous iterations. *W<sub>0</sub>* is kept fixed.

Both partial self-consistent cycles run for `eviter` number of cycles.

### METHOD

Two different techniques to obtain the diagonal self-energy
matrix elements are implemented in NWChem. 

The `analytic` method builds and diagonalizes the full Casida RPA matrix
in order to obtain the screened Coulomb matrix elements. The Casida RPA
matrix grows very rapidly in size (*N<sub>occ</sub>* &times; *N<sub>vir</sub>*)
and ultimately yields a *N<sup>6</sup>* scaling due to the diagonalization step.
It is therefore recommended to link the [ELPA](https://elpa.mpcdf.mpg.de/)
and turn on its use by setting

```
SET dft:scaleig e
```

The `cdgw` method uses the Contour-Deformation technique in order to
avoid the *N<sup>6</sup>* diagonalization step. The diagonal self-energy
matrix elements &#x01A9;<sub>*nn*</sub> are obtained via a numerical
integration on the imaginary axis and the integrals over closed contours
on the first and third quadrants of the complex plane. The `grid_points`
value controls  the density of the modified Gauss-Legendre grid used
in the numerical integration over the imaginary axis.

Both `analytic` and `cdgw` methods are suitable for *core* **and**
*valence* calculations.

### ETA

The magnitude of the imaginary infinitesimal can be controlled using the
keyword `ETA`. The default value of `0.001` should work rather well
for *valence* calculations, but `CORE` calculations might need a
larger value, sometimes between `0.005` or even `0.01`.

### SOLVER

Two methods to solver the quasiparticle equations are implemented
in NWChem. 

The `newton` method uses a modified Newton approach to find the fixed-point
of the quasiparticle equations. The Newton method tries to bracket the
solution and switches to a golden section method whenever the Newton step goes
beyond the bracketing values.

The `graph` method uses a frequency grid in order to bracket the solution
between two consecutive grid points. The number of grid points is controlled 
heuristically depending on the [`METHOD`](#method)
and on the presence, or not, of nearby states in a cluster
of energy (see below).

Regardless of the solver, the energies of the states are always
classified in clusters with a maximum extension of `1.5 eV`. For
a given cluster of energies, the
`newton` method will start with the state closer to the Fermi level
and use its solution as guess for the rest of the states in the cluster.
The `graph` method will look for the solution of all the states
in a given cluster at once with a frequency grid with range large 
enough to encompass all the cluster &plusmn; `0.2 eV`.

### STATES

The keyword `STATES` controls for which particular state
the *GW* quasiparticle equations are to be solved. The
keyword might appear twice, one for the *alpha* spin channel
and one for the *beta* channel. The *beta* channel keyword is
meaningless for restricted closed-shell DFT calculations 
(`MULT 1` without `ODFT` in the `DFT` input block).

The number of occupied states will be counted starting from the 
state closest to the Fermi level (HOMO) unless the keyword
[`CORE`](#core-and-first) is present. The virtual states will **always**
be counted from the state closest to the Fermi level upwards.


### CONVERGENCE

The converegnce threshold of the quasiparticle equations can
be controlled with the keyword `CONVERGENCE` and might be
given either in `eV` or Hartree `au`.