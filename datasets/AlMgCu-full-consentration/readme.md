## Introduction
A Deep Potential model and data sets for the Al-Cu-Mg alloy in the full concentration space. **[1]**

## Generation Approach
The initial dataset is prepared in this way. For each user-specific crystalline structure and each possible concentration, the atomic species is randomly assigned on the lattice points. Then, random perturbations are performed on the coordinates by adding values drawn from a uniform distribution in the range [$−p_a$, $p_a$]. Perturbations are also performed on the cell vectors by a symmetric deformation matrix that is constructed by adding random noise drawn from a uniform distribution in the range [$−p_c$, $p_c$] to an identity matrix. Next, starting from the perturbed structures, AIMD simulations are conducted to produce labelled data with DFT calculated energy, force, and virial tensor. Finally, the labelled data are used to create the initial data sets. The preparation for single-element systems follows the same procedure without the need to consider possible concentrations. Then the initial data was used to feed into the DP-GEN scheme.

## Data Format
The directory tree is as follows:

```
AlCuMg
```

Subdirectory of which contains unit systems in DeePMD format.

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
**[1]** Jiang, Wanrun, et al. "Accurate deep potential model for the Al–Cu–Mg alloy in the full concentration space." *Chinese Physics B* 30.5 (2021): 050706.
