## Introduction
This is the dataset of water and ice used in. **[1,2]**  The PBE0+TS **[3,4]** functional is adopted in all cases.

## Generation Approach
The data used for training and/or testing are extracted from the AIMD simulations summarized in Tab. IV. The training datasets include 95000 snapshots (from 105000 total snapshots) randomly selected along the liquid water trajectory, 19500 snapshots (from 24000 total snapshots) randomly selected along the ice (b) trajectory, 9500 snapshots (from 12000 total snapshots) randomly selected along the ice (c) trajectory, and 9500 snapshots (from 12000 total snapshots) randomly selected along the ice (d) trajectory. The remaining snapshots in the database are used for testing purposes.

TABLE IV: Equilibrated AIMD trajectories (traj.) for liquid water (LW) and ice Ih.
| System |  PI/classical | N  | P[bar] | T[K]  | traj. length [ps] |
| ------ | ------------- | -  | ------ | ----- | ----------------- |
|   LW   | path integral | 64 |   1.0  |  300  |        6.2        |
| ice(b) | path integral | 96 |   1.0  |  273  |        1.5        |
| ice(c) |   classical   | 96 |   1.0  |  330  |        6.0        |
| ice(d) |   classical   | 96 |  2.13k |  238  |        6.0        |

## Data Format
The directory tree is as follows:

```
-- ice_pimd
-- ice_triple_I
-- ice_triple_II
-- lw_pimd
```
where each subdirectory contains unit systems in DeePMD format.

A pbc unit system usually has the following substructure:

```
-- system
-- -- type.raw
-- -- set.xxx/box.npy
-- -- set.xxx/coord.npy
-- -- set.xxx/energy.npy
-- -- set.xxx/force.npy
-- -- set.xxx/virial.npy
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
**[1]** Zhang, Linfeng, et al. "Deep potential molecular dynamics: a scalable model with the accuracy of quantum mechanics." Physical review letters 120.14 (2018): 143001.

**[2]** Ko, Hsin-Yu, et al. "Isotope effects in liquid water via deep potential molecular dynamics." Molecular Physics 117.22 (2019): 3269-3281.

**[3]** Adamo, Carlo, and Vincenzo Barone. "Toward reliable density functional methods without adjustable parameters: The PBE0 model." The Journal of chemical physics 110.13 (1999): 6158-6170.

**[4]** Tkatchenko, Alexandre, and Matthias Scheffler. "Accurate molecular van der Waals interactions from ground-state electron density and free-atom reference data." Physical review letters 102.7 (2009): 073005.  










These data were used for the two papers:
1) L. Zhang, J. Han, H. Wang, R. Car, W. E, Physical review letters 120 (14), 143001
2) H.-Y. Ko, L. Zhang, B. Santra, H. Wang, W. E, R. A. DiStasio Jr, R. Car Molecular Physics 117 (22), 3269-3281

Please consider citing them when you use the data for your work.
