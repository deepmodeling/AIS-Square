## Introduction
This is the dataset of Ag–Au nanoalloys DFT data for training a long range DeepMD model in **[1]**. 

## Generation Approach
This data set is generated with DPGEN **[2,3]**,  a concurrent learning scheme that generates the DP training dataset iteratively.

**Initial dataset.** The initial dataset is used to train the ensemble of initial guesses of the DP models. In this work, for pure Ag and Au, as well as Ag–Au binary alloys, the initial datasets are prepared in three steps. First, for each system, equilibrium body-centered cubic (BCC), HCP, and FCC structures within 2 × 2 × 2 supercells are constructed. For the Ag–Au alloy, all possible concentrations are considered and subject to each concentration, the lattice sites are randomly occupied. Second, the atomic coordinates are perturbed by a random number in the range ±0.01 Å, and the cell vectors are randomly perturbed in the range of ±3% of the original matrix. In addition, the cells are compressed to cover the high-pressure configurations, with the compression ratio ranging from 0.84 to 0.98 of the equilibrium volume, with a step of 0.02 for pure Ag and Au and a step of 0.04 for Ag–Au alloys. Finally, short trajectory ab initio molecular dynamics simulations with 5 steps are conducted starting from the perturbed configurations, and the energy, force, and virial tensor along the trajectories are collected as the initial datasets. The number of configurations in the initial dataset for pure Ag and pure Au is 6140. For the Ag–Au alloy, there are 9150 configurations in the initial dataset.

**Exploration.** The LAMMPS package **[4]** compiled with the DeePMD-kit **[5]** support is employed to perform DPMD simulations for exploring the configuration and content space. We consider both bulk and surface structures to correctly describe the basic properties and the surface-related properties, such as the surface structures, surface segregation, and the adsorption and diffusion of adatoms on surfaces.

**Labeling.** The labels of the candidate configurations, i.e. the energy, force, and virial tensor, are computed by DFT. The DFT calculations were conducted using the Vienna ab initio simulation package (VASP) **[6,7]**, with the generalized gradient approximation and the Perdew–Burke–Ernzerhof (PBE) **[8]** xc functional. The projector-augmented-wave (PAW) method **[9,10]** is used, and the energy cutoff of the planewave basis set is set to 650 eV. The Brillouin zone is sampled by the Monkhorst-Pack method **[11]** with a grid spacing of 0.1 $Å^{−1}$. The convergence criterion for the energy is $1 × 10^{−6}$ eV.

**Training.** The DP–PBE model is constructed using a smooth edition of the DP model **[12]**. The DeePMD-kit package **[5]** is used for training

Details about each step is refer to **[1]**.

## Data Format
The directory tree is as follows:

```
AgAuD3
-- init
-- iter.xxxxxxx
AuD3
-- Au/init
-- Au/iter.xxxxxx
```

The Data contains AgAu and Au's DFT result organized in DeepMD format. Sampled result from DPGEN can be fond in AgAuD3 and AuD3 folders.

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
**[1]** Wang, YiNan, et al. "A generalizable machine learning potential of Ag–Au nanoalloys and its application to surface reconstruction, segregation and diffusion." *Modelling and Simulation in Materials Science and Engineering* 30.2 (2021): 025003.

**[2]** Zhang, Linfeng, et al. "Active learning of uniformly accurate interatomic potentials for materials simulation." *Physical Review Materials* 3.2 (2019): 023804.

**[3]** Zhang, Yuzhi, et al. "DP-GEN: A concurrent learning platform for the generation of reliable deep learning based potential energy models." *Computer Physics Communications* 253 (2020): 107206.

**[4]** Plimpton, Steve. "Fast parallel algorithms for short-range molecular dynamics." *Journal of computational physics* 117.1 (1995): 1-19.

**[5]** Wang, Han, et al. "DeePMD-kit: A deep learning package for many-body potential energy representation and molecular dynamics." *Computer Physics Communications* 228 (2018): 178-184.

**[6]** Kresse, Georg, and Jürgen Furthmüller. "Efficiency of ab-initio total energy calculations for metals and semiconductors using a plane-wave basis set." *Computational materials science* 6.1 (1996): 15-50.

**[7]** Kresse, Georg, and Jürgen Furthmüller. "Efficient iterative schemes for ab initio total-energy calculations using a plane-wave basis set." *Physical review B* 54.16 (1996): 11169.

**[8]** Perdew, John P., Kieron Burke, and Matthias Ernzerhof. "Generalized gradient approximation made simple." *Physical review letters* 77.18 (1996): 3865.

**[9]** Blöchl, Peter E. "Projector augmented-wave method." *Physical review B* 50.24 (1994): 17953.

**[10]** Kresse, Georg, and Daniel Joubert. "From ultrasoft pseudopotentials to the projector augmented-wave method." *Physical review b* 59.3 (1999): 1758.

**[11]** Monkhorst, Hendrik J., and James D. Pack. "Special points for Brillouin-zone integrations." *Physical review B* 13.12 (1976): 5188.

**[12]** Zhang, Linfeng, et al. "End-to-end symmetry preserving inter-atomic potential energy model for finite and extended systems." Advances in Neural Information Processing Systems 31 (2018).
