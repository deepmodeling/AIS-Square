## Introduction
This dataset cnontains DFT samples in DeepMD format used in **[1]** for water.

## Generation Approach
To construct an accurate and transferable SCAN0 path-integral deep potential model with a minimal number of the expensive SCAN0 DFT data, an active machine learning procedure called deep potential generator (DP-GEN) **[2]** was adopted. Considering that the molecular configurations predicted by SCAN and SCAN0 are close to each other, 900 configurations were uniformly extracted from the 64-molecule 11 ps SCAN PI-AIMD trajectory reported in ref **[3]**. The total potential energy E and ionic forces Fi of each atom i of these configurations were calculated by using Quantum ESPRESSO (QE)**[4]** with the SCAN0 XC functional. The Hamann− Schlüter−Chiang−Vanderbilt (HSCV) pseudopotentials **[5,6]** with an energy cutoff of 150 Ry were employed. The obtained E and Fi together with atomic positions were adopted as the initial training data.


## Data Format
The directory tree is as follows:

```
202204001
-- data_scan0
-- PseudoPogtential
```

Subdirectory of data_scan0 contains unit systems in DeePMD format.

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
**[1]** Zhang, Chunyi, et al. "Modeling liquid water by climbing up Jacob’s ladder in density functional theory facilitated by using deep neural network potentials." *The Journal of Physical Chemistry B* 125.41 (2021): 11444-11456.

**[2]** Zhang, Linfeng, et al. "Active learning of uniformly accurate interatomic potentials for materials simulation." *Physical Review Materials* 3.2 (2019): 023804.

**[3]** Bergmann, Uwe, et al. "Isotope effects in liquid water probed by x-ray Raman spectroscopy." *Physical Review B* 76.2 (2007): 024202.

**[4]** Giannozzi, Paolo, et al. "Advanced capabilities for materials modelling with Quantum ESPRESSO." *Journal of physics: Condensed matter* 29.46 (2017): 465901.

**[5]** Hamann, D. R., M. Schlüter, and C. Chiang. "Norm-conserving pseudopotentials." *Physical Review Letters* 43.20 (1979): 1494.

**[6]** Vanderbilt, David. "Optimally smooth norm-conserving pseudopotentials." *Physical Review B* 32.12 (1985): 8412.
