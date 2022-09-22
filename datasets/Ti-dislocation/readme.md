## Introduction
This is the dataset of Ti's DFT data for training DeepMD model.

## Generation Approach
The concurrent learning procedure called deep potential generator (DP-GEN) **[1,2]** is applied to perform iterations in order to refine the DP model. We set up 32 iterations in the DP-GEN workflow. For each iteration, a set of initial configurations are chosen from the α-Sn, β-Sn, bct-Sn and bcc-Sn structures. Next, five temperatures are chosen for each iteration and the temperatures arise after every 8 iterations. In total, 20 temperatures ranging from 50 to 1950 K with the interval being 100 K are set. For each temperature, nine different pressures including 0, 5, 10, 15, 20, 25, 30, 40, and 50 GPa are set. Detailed setups of the exploration strategy are listed in Table S1 of SI. Finally, three steps, i.e., the exploration, labeling, and training processes are performed in each iteration to generate a new DP model.

**DFT method:** We perform DFT calculations for Sn with the Vienna ab initio simulation package (VASP 5.4.4) **[3]**. The projector-augmented wave (PAW) method **[4,5]** is used to describe the ion-electron interactions with 14 valence electrons for Sn. We utilize the SCAN meta-generalized gradient approximation (meta-GGA) exchange-correlation functional **[6]**. The kinetic energy cutoff is set to 650 eV. The Gaussian smearing of 0.20 eV is chosen with the smallest allowed spacing between the Monkhorstpack (MP) **[7]** k-points set as 0.10 $Å^{−1}$ to sample the first Brillouin zone of solid structures. The total energy is converged to be less than $10^{−6}$ eV in self-consistent electronic iterations.

## Data Format
The directory tree is as follows:

```
Ti_training_data
-- omega
-- Ti_0722
```

The Ti_0722 folder contains Ti's DFT result sampled by DPGEN and organized in DeepMD format.

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

**[1]** Zhang, Linfeng, et al. "Active learning of uniformly accurate interatomic potentials for materials simulation." *Physical Review Materials* 3.2 (2019): 023804.

**[2]** Zhang, Yuzhi, et al. "DP-GEN: A concurrent learning platform for the generation of reliable deep learning based potential energy models." *Computer Physics Communications* 253 (2020): 107206.

**[3]** Kresse, Georg, and Jürgen Furthmüller. "Efficient iterative schemes for ab initio total-energy calculations using a plane-wave basis set." *Physical review B* 54.16 (1996): 11169.

**[4]** Blöchl, Peter E. "Projector augmented-wave method." *Physical review B* 50.24 (1994): 17953.

**[5]** Kresse, Georg, and Daniel Joubert. "From ultrasoft pseudopotentials to the projector augmented-wave method." *Physical review b* 59.3 (1999): 1758.

**[6]** Sun, Jianwei, Adrienn Ruzsinszky, and John P. Perdew. "Strongly constrained and appropriately normed semilocal density functional." *Physical review letters* 115.3 (2015): 036402.

**[7]** Monkhorst, Hendrik J., and James D. Pack. "Special points for Brillouin-zone integrations." *Physical review B* 13.12 (1976): 5188.
