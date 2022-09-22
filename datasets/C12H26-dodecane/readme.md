## Introduction
This is the dataset of n-dodecane in **[1]**. 

## Generation Approach
As shown in Figure 1, the whole workflow for the construction of DP model is divided into 3 modules: Initialization, Concurrent Learning, and Finalization. An initial dataset is obtained in the Initialization module to kick-off the Concurrent learning section. In the Concurrent learning section, the chemical space of the reaction is gradually explored as the reaction proceeds. When the chemical space is sufficiently explored, a final dataset will be obtained and the final DP can be trained based on it. Taking the n-dodecane pyrolysis as an example, details of these modules are introduced below.

<p align="center">
<img src="https://github.com/deepmodeling/AIS-Square/blob/main/datasets/C12H26-dodecane/figs/Fig1.png?raw=true" width=70% />
</p>  

**Initialization.** The main purpose of the initialization module is to create an initial dataset for the concurrent learning module. It roughly contains 3 steps:

*Exploring I.* As the starting configuration, a box containing 40 n-dodecane molecules with the density of $0.25g/cm^3$ was constructed by using the Amorphous Cell module in the Materials Studio 2017 program suite. Then, a 1 ps NVT MD simulation was performed by using the LAMMPS software and the ReaxFF PES model developed by Chenoweth et al. (CHO-2008).**[2]** The NVT ensemble was employed and the temperature was maintained at 3000K with the Berendsen thermostat. The time step was set to 0.1 fs and the atomic coordinates were recorded in every 10 fs. We also built boxes containing n-decane, n-tetradecane, and n-icosane, respectively, with the same method and density. Then, a molecular cluster was created for each atom (defined as the center atom) in the trajectory, which contains the center atom and any molecular species within a 3.5 Å distance from it. Since the pyrolysis reaction is dominated by bond breaking and short-range collision between different spices, 3.5 Å is a reasonable cutoff threshold that balances the accuracy and computational cost.

*Sampling.* From the previous step, we might obtain thousands of molecular clusters, which exhibit heavy redundancy. In this step, these clusters were firstly classified according to the bond-type of the center atom and then the k-means clustering algorithm was used to remove the redundancy. Details of relevant methods can be found in our previous study. Finally, an initial dataset containing 590 molecular clusters was obtained.

*Labeling.* The potential energy and atomic forces of each molecular cluster in the dataset were calculated with Gaussian 16 at the MN15/6-31G** level. The MN15 functional was chosen because it is specifically designed to have broad accuracy for multi-reference and single-reference systems. When compared with 82 other density functionals, MN15 gives the second smallest mean unsigned error for 54 inherently multiconfigurational systems. The spin multiplicity was set to 1 or 2 according to the number of electrons in the cluster. The stability of DFT wavefunction was checked during self-consistent field calculation, and if instability was found, further optimization was performed until a stable solution was achieved.

It is worth mentioning that the size and the content of the initial dataset was not very critical. It was only used as a start point of the concurrent learning section. However, the employment of initial data set might effectively reduce the number of iterations of concurrent learning.

**Concurrent Learning.** To explore the chemical space more efficiently, the concurrent learning algorithm was used, which contains a number of iterations. Each iteration is composed of three steps:

*Training I.* In this step, based on the dataset from the previous step (or the initial dataset at the beginning), 4 DeepPot-SE models were trained at the same time with the same network architectures but different random seeds for their parameters. In the DeepPot-SE model, the total energy of the system is expressed as a sum of atomic energies. And the atomic energy of a given atom depends on its local chemical environment. A local environment matrix and an embedding network were used to generate the molecular descriptors which can preserve the translational, rotational and permutational symmetry. The DeePMD-kit package was used for this step. The batch number, which denotes the number of training steps, was set to a relatively medium value (400,000) to improve the efficiency without too much loss of the accuracy. Other technical details were consistent with Ref. **[3]**

*Exploring II.* In this step, four reactive MD simulations of n-dodecane at different temperatures (1500, 2000, 2500, and 3000K, respectively) were performed starting from the initial configuration of the system, driven by the DP model from the previous iteration. The Nose-Hoover thermostat was used to sample the NVT ensemble. The simulation time gradually increases from 0.1 ps to 1 ns as the number of iterations increases. To enhance the transability of the model, we added an extra iteration to perform a 1 ns MD simulations of n-decane under 2000 K. For each atom in the system, its atomic force was also evaluated by the other three DP models. And the relative force model deviation between these four models was calculated by $$E_{f_{i}}=\frac{\left|D_{f_{i}}\right|}{\left|f_{i}\right|+l}$$ where $f_i$ denotes the force on atom $i$, $D_{fi}$ denotes the corresponding model deviation, and $l$ is a constant (1 eV/Å in this study) used to make sure that an atom having a small absolute model deviation will also have a small relative model deviation. The employment of relative model deviation instead of the absolute one is crucial for studying combustion reactions, since in such a situation the atomic forces have a very large range of values.

In each iteration, a large number of molecular clusters were extracted from the trajectory. Based on the relative model deviation of the atomic force of the central atom, these clusters were classified into three different groups named "Accurate" (the relative force deviation is less than 20%), "Candidate" (the relative force deviation is between 20% and 45%), and "Failed" (the relative force deviation is greater than 45%). To minimize data redundancy, in each iteration, at most 1000 molecular clusters were randomly selected from the "Candidate" group and added to the training dataset.

In the *Labeling* step, the PES and atomic forces of the candidate clusters were calculated using the same method detailed in the previous section. It has to be emphasized that in each iteration, the MD simulations should start from the initial configurations. When the iteration procedure stops depends on the specific problem. In this work, we stopped the iteration when MD reached 1 ns, with all the clusters extracted from the trajectory at this point correctly predicted by the DP model (with an accuracy of 99.5% and a failure rate of 0.0).

In other words, the dataset at this point should cover the chemical space we need from 1-ns simulations, during which the reactants should have been consumed and then equilibrium should have been reached. In the end of the Concurrent learning module, we got the final training dataset.

**Finalization.** In this section, a DP model was trained (in the Training II step) based on the final dataset. To guarantee the accuracy of the model, the batch number of the training process was set to 4,000,000 while other parameters were as same as those in the Training I step.

All three sections were integrated into the DP-GEN software 45 and are fully automatic and user-friendly.

## Data Format
The directory tree is as follows:

```
-- C{xx}H{xx}
...
```

where each subdirectory contains unit systems in DeePMD format.

A pbc unit system usually has the following substructure:

```
-- type_map.raw
-- type.raw
-- nopbc
-- set.000/box.npy
-- set.000/coord.npy
-- set.000/energy.npy
-- set.000/force.npy
-- set.000/virial.npy
```

Format Description

| Name     | Property           | Raw file     | Unit | Shape                  | Description                                                  |
| -------- | ------------------ | ------------ | ---- | ---------------------- | ------------------------------------------------------------ |
| type     | Atom type indexes  | type.raw     |      | Natoms                 | Integers that start with 0, represent the atomic type corresponding to type_map.raw |
| type_map | Atom type names    | type_map.raw |      | Ntypes                 | Atom names that map to atom type, which is unnecessart to be contained in the periodic table |
| nopbc    | Non-periodic system| nopbc        |Required|                      |If True, this system is non-periodic; otherwise it's periodic |
| coord    | Atomic coordinates | coord.npy    | Å    | Nframes \* Natoms \* 3 | The atomic coordinates                                       |
| box      | Boxes              | box.npy      | Å    | Nframes \* 3 \* 3      | The box axes in the order `XX XY XZ YX YY YZ ZX ZY ZZ`       |
| energy   | Frame energies     | energy.npy   | eV   | Nframes                | The potential energy of snapshot                             |
| force    | Atomic forces      | force.npy    | eV/Å | Nframes \* Natoms \* 3 | The atomic forces                                            |
| virial   | Frame virial       | virial.npy   | eV   | Nframes * 9            | The virial frames are in the order `XX XY XZ YX YY YZ ZX ZY ZZ` |

Check [here](https://github.com/deepmodeling/deepmd-kit/blob/master/doc/data/system.md) for more details.

## References
**[1]** Zeng, Jinzhe, et al. "Exploring the chemical space of linear alkane pyrolysis via deep potential generator." Energy & Fuels 35.1 (2020): 762-769.

**[2]** Chenoweth, Kimberly, Adri CT Van Duin, and William A. Goddard. "ReaxFF reactive force field for molecular dynamics simulations of hydrocarbon oxidation." The Journal of Physical Chemistry A 112.5 (2008): 1040-1053.

**[3]** Zeng, Jinzhe, et al. "Complex reaction processes in combustion unraveled by neural network-based molecular dynamics simulation." Nature communications 11.1 (2020): 1-9.

**[4]** Zhang, Yuzhi, et al. "DP-GEN: A concurrent learning platform for the generation of reliable deep learning based potential energy models." Computer Physics Communications 253 (2020): 107206.


