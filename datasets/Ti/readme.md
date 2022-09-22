## Introduction
This is the data set used in **[1]**, for DFT data of Ti organized with DeepMD structure.

## Generation Approach
All DFT calculations are performed using the VASP **[2,3]** with the Perdew–Burke–Ernzerhof **[4]** generalised gradient approximation exchangecorrelation functional. The cutoff energy of the plane-wave basis set is 650 eV and core electrons are replaced with the projector-augmented-wave method **[5]**. K-points with grid spacing of 0.1 $Å^{−1}$ are sampled in the Brillouin zone by the Monkhorst–Pack Mesh method **[6]**. The Methfessel–Paxton smearing method **[7]** with order 1 and smearing width σ = 0.22 eV is used for partial electron occupancy. Self-consistent convergence is assumed when the energy variation is below $10^{−3}$ meV.

## Data Format
The directory tree is as follows:

```
data_0818
-- T1_0722
	-- bulk_dpggen_0915
	-- bulk_dpgen_0915-2
	-- gamma_line
	-- init
	-- init_0823
	-- surf_dpgen_1019_iter32
```

Subdirectory of each directory contains the DPGEN output of generated datasets in DeePMD format.

A pbc unit system usually has the following substructure:

```
-- system
-- -- type_map.raw
-- -- type.raw
-- -- set.000/box.npy
-- -- set.000/coord.npy
-- -- set.000/energy.npy
-- -- set.000/force.npy
-- -- set.000/virial.npy
```

Format Description

| Name     | Property           | Raw file     | Unit | Shape                  | Description                                                  |
| -------- | ------------------ | ------------ | ---- | ---------------------- | ------------------------------------------------------------ |
| type     | Atom type indexes  | type.raw     |      | Natoms                 | Integers that start with 0, represent the atomic type corresponding to type_map.raw |
| type_map | Atom type names    | type_map.raw |      | Ntypes                 | Atom names that map to atom type, which is unnecessart to be contained in the periodic table |
| coord    | Atomic coordinates | coord.npy    | Å    | Nframes \* Natoms \* 3 | The atomic coordinates                                       |
| box      | Boxes              | box.npy      | Å    | Nframes \* 3 \* 3      | The box axes in the order `XX XY XZ YX YY YZ ZX ZY ZZ`       |
| energy   | Frame energies     | energy.npy   | eV   | Nframes                | The potential energy of snapshot                             |
| force    | Atomic forces      | force.npy    | eV/Å | Nframes \* Natoms \* 3 | The atomic forces                                            |
| virial   | Frame virial       | virial.npy   | eV   | Nframes * 9            | The virial frames are in the order `XX XY XZ YX YY YZ ZX ZY ZZ` |

Check [here](https://github.com/deepmodeling/deepmd-kit/blob/master/doc/data/system.md) for more details.


## References
**[1]** Wen, Tongqi, et al. "Specialising neural network potentials for accurate properties and application to the mechanical response of titanium." *npj Computational Materials* 7.1 (2021): 1-11.

**[2]** Kresse, Georg, and Jürgen Furthmüller. "Efficiency of ab-initio total energy calculations for metals and semiconductors using a plane-wave basis set." *Computational materials science* 6.1 (1996): 15-50.

**[3]** Kresse, Georg, and Jürgen Furthmüller. "Efficient iterative schemes for ab initio total-energy calculations using a plane-wave basis set." *Physical review B* 54.16 (1996): 11169.

**[4]** Perdew, John P., Kieron Burke, and Matthias Ernzerhof. "Generalized gradient approximation made simple." *Physical review letters* 77.18 (1996): 3865.

**[5]** Blöchl, Peter E. "Projector augmented-wave method." *Physical review B* 50.24 (1994): 17953.

**[6]** Monkhorst, Hendrik J., and James D. Pack. "Special points for Brillouin-zone integrations." *Physical review B* 13.12 (1976): 5188.

**[7]** Methfessel, M. P. A. T., and A. T. Paxton. "High-precision sampling for Brillouin-zone integration in metals." *Physical Review B* 40.6 (1989): 3616.
