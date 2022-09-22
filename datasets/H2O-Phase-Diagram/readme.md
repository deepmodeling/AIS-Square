## Introduction
This dataset consists of 34577 DFT samples in DeePMD format used in **[1]**, covering the phase diagram of water with T 0\~2400K P 0\~50GPa, which is generated during DPGEN **[2]** procedure.

## Generation Approach
To generate these data samples and, in the meantime, construct the model PES, a trial DeePMD is built from configurations of the liquid, at ambient conditions, and of all the experimentally known stable and metastable ice polymorphs for P ⪅ 50 GPa (Ih, Ic, II, III, IV, V, VI, VII, VIII, IX, XI, XII, XIII, XIV, XV). The model is used by a DeePMD generator (DPGEN) to explore a wide region of the phase space with isothermal-isobaric (NPT) DeePMD trajectories. The protocol is iterated to refine the model with new DFT data until satisfactory accuracy is achieved. The visited states can be roughly classified into three groups: the low pressure (A), the high pressure (B), and the superionic group (C). Group (A) includes states in the range 50 ≤ T ≤ 600 K and $10^{−4}$ ≤ P ≤ 5 GPa, starting from configurations of the fluid and of all the ices except VII and VIII. Group (B) includes states in the range 50 ≤ T ≤ 600 K and 0.1 ≤ P ≤ 50 GPa, starting from configurations of ice VII and VIII. Group (C) includes states in the range 200 ≤ T ≤ 2400 K and 1 ≤ P ≤ 50 GPa, starting from ice VII and the fluid. DeePMD trajectories sample almost uniformly the thermodynamic domains of the three groups. The deviation in the predicted forces within a set of representative DP models is used to label configurations for which new DFT calculations of the energy, forces, and virial are necessary. The new data are added to the training dataset and serve to refine the representative DeePMD models entering the next iteration.

After 36 concurrent learning iterations the error in the force is satisfactorily reduced and the procedure ends. The accumulated number of snapshots in the training dataset is 31058, a tiny fraction (∼0.05%) of the configurations visited by DeePMD. At this point, the relative energies of configurations within each phase are well described, but deviations from DFT still affect the averages. To reduce these deviations below a small threshold, 3519 additional training configurations are added.

## Data Format
The directory tree is as follows:
```
data
-- data.init
-- data.iters
-- data_050820
-- data_051920
-- data_high_pressure
-- data_high_pressure2
```
where each subdirectory contains unit systems in DeePMD format.

A pbc unit system usually has the following substructure:

```
-- system
-- -- type_map.raw
-- -- type.raw
-- -- set.000/box.npy
-- -- set.000/coord.npy
-- -- set.000/energy.npy
-- -- set.000/force.npy
```

Format Description

|Name     | Property                | Raw file     | Unit                 | Shape                    | Description|
|-------- | ----------------------  | ------------ | -------------------- | -----------------------  | -----------|
|type     | Atom type indexes       | type.raw     |                      | Natoms                   | Integers that start with 0, represent the atomic type corresponding to type_map.raw |
|type_map | Atom type names         | type_map.raw |                      | Ntypes                   | Atom names that map to atom type, which is unnecessart to be contained in the periodic table |
|coord    | Atomic coordinates      | coord.npy | Å                    | Nframes \* Natoms \* 3   | The atomic coordinates |
|box      | Boxes                   | box.npy   | Å                    | Nframes \* 3 \* 3        | The box axes in the order `XX XY XZ YX YY YZ ZX ZY ZZ` |
|energy   | Frame energies          | energy.npy | eV                   | Nframes                  | The potential energy of snapshot |
|force    | Atomic forces           | force.npy | eV/Å                 | Nframes \* Natoms \* 3   | The atomic forces |

Check [here](https://github.com/deepmodeling/deepmd-kit/blob/master/doc/data/system.md) for more details.


## References
**[1]** Zhang, Linfeng, et al. "Phase diagram of a deep potential water model." Physical review letters 126.23 (2021): 236001.

**[2]** Zhang, Yuzhi, et al. "DP-GEN: A concurrent learning platform for the generation of reliable deep learning based potential energy models." Computer Physics Communications 253 (2020): 107206.
