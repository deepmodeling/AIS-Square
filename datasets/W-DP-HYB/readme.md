## Introduction
This is the dataset of Tungsten DFT data for training DeepMD model.

## Generation Approach
This data set is generated with DPGEN **[2,3]**,  a concurrent learning scheme that generates the DP training dataset iteratively.

**Initial dataset.** The initial dataset is composed of three parts:

1. The equilibriated unit cells of the BCC, the FCC, the hexagonal close-packed (HCP) and diamond structures. 

2. Artificially strained and perturbed structures. Hydrostatic strain is applied to equilibriated structures by changing lattice parameters ranging from 96% to 106% at a step 2%. Perturbations are then exerted on each structure under strain. The atom positions are randomly perturbed with a maximal displacement of 0.01 Å. The cell is randomly perturbed by 3%. 
3. AIMD trajectory of the deformed structures. Setting each compressed structure under perturbation as initial configuration, AIMD simulations are conducted with 5 steps under temperature 100K.

All the data are labeled with DFT calculations (see part c. Labeling for more details). In other words, the DFT-calculated energy, atomic forces and virial tensor of each structure are recorded.

**Exploration.** The LAMMPS package **[4]** compiled with the DeePMD-kit **[5]** support is employed to perform DPMD simulations for exploration of the configuration space.  The exploration uses 4 subsets of configurations as the initial configurations for the MD sampling. The initial configurations, simulation ensemble, temperatures and pressures conditions are summarized in the following and in the Table I of **[1]**.

1. The strained and perturbed 2 × 2 × 2 supercell BCC W bulk. NPT ensemble. Temperature 50 to 5100 K. Pressure −2 to 5 GPa. 
2. The strained and perturbed 3 × 3 × 3 supercell BCC W bulk. NPT ensemble. Temperature 50 to 5100 K. Pressure −2 to 5 GPa. 
3. (111),(110) and (112) free surfaces. NVT ensemble. Temperature 300 to 1800K. 4. Locally perturbed 2 × 2 × 2 and 3 × 3 × 3 supercells of BCC W bulk. NVE ensemble.

During the exploration, the deviation of force predictions of four DP models, trained with identical hyper-parameters but different random seeds, is used to estimate the error in the force prediction. If the maximal deviation of atomic forces is higher than 0.20 eV/Å but lower than 0.35 eV/Å, the configuration is considered as a candidate configuration and sent for labeling.

**Labeling.** The labels of the candidate configurations, i.e. the energy, force, and virial tensor, are computed by DFT with exchange-correlation modeled by the generalized gradient approximation (GGA) proposed by Perdew, Burke and Ernzerhof (PBE) **[8]**. The DFT calculations were conducted using VASP[6, 7] package. The Brillouin zone is sampled by the Monkhorst-Pack method with a grid spacing of 0.16 $Å^{−1}$ . The projector-augmented-wave (PAW) method [9, 10] is used and the energy cut-off of the plane-wave basis set is set to 600 eV. The 6s and 5d electrons are considered valence electrons. The convergence criterion for the self-consistent field iteration is set to $10^{−6}$ eV. The same DFT parameters are also used for labeling the initial dataset.

**Training.** In each iteration, four models are trained simultaneously using the same dataset and hyper-parameters, with the only difference being the random seeds employed to initialize the model parameters. The sizes of the hidden layers of the two-body embedding nets G(2) are (20, 40, 80), while the three-body embedding G(3) has hidden layers of sizes (4, 8, 16). The hidden layers of the fitting nets are set to (240, 240, 240). The Adam stochastic gradient descent method **[11]** with the default hyper-parameter settings provided by the TensorFlow package **[12]** is used to train the DP models. The learning rate is exponentially decayed with starting and final learning rates set to $1 × 10^{−3}$ and $5 × 10^{−8}$ , respectively. In each DP-GEN iteration the DP model is trained with $4 × 10^5$ steps. After the DP-GEN iterations converge, the productive models are trained with $2.4 × 10^{7}$ steps. The size of the DNNs used in the DP models, and other hyper-parameters are provided in the Supplementary Material Table SI.

**Refinement.** The productive DP models are firstly refined with DFT labeled structures of SIA structures, the γ-line and (100) surface structure. SIA data is generated via additional DP-GEN iterations, with three types of SIA structures (namely h111i,h110i and h100i dumbbells) as initial configurations, the explored temperature ranges from 50K to 600K. γ-line structures are obtained directly from the DFT γ-line calculation along h111i directions on {110} and {112} planes. We also included relaxed and unrelaxed (100) free surface data in the refining dataset. The refined models are trained for $1 × 10^6$ steps with model parameters initialized by the productive model. The starting learning rate is set to $1 × 10^{−4}$ , and the starting prefactors of energy, force and virial tensors, p start ,p start f ,p start ξ are set to 1.0, 1.0 and 0.9, respectively (See the Supplementary Material for the definition of the prefactors). The other hyper-parameters are the same as those used to train the productive model. Then, the DP models are further refined by the isolated W atom energy. All the hyper-parameters are the same with those used in the first refinement except that the number of training steps is set to $4 × 10^6$ . The energy of an isolated atom is calculated with spin-polarized DFT. Benchmark results are based on the model after the refinements.

## Data Format
The directory tree is as follows:

```
Wdata
-- init
```

The init folder contains Tungsten's DFT result sampled by DPGEN and organized in DeepMD format.

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

**[1]** Wang, Xiaoyang, et al. "A tungsten deep neural-network potential for simulating mechanical property degradation under fusion service environment." *Nuclear Fusion* (2022).

**[2]** Zhang, Linfeng, et al. "Active learning of uniformly accurate interatomic potentials for materials simulation." *Physical Review Materials* 3.2 (2019): 023804.

**[3]** Zhang, Yuzhi, et al. "DP-GEN: A concurrent learning platform for the generation of reliable deep learning based potential energy models." *Computer Physics Communications* 253 (2020): 107206.

**[4]** Plimpton, Steve. "Fast parallel algorithms for short-range molecular dynamics." *Journal of computational physics* 117.1 (1995): 1-19.

**[5]** Wang, Han, et al. "DeePMD-kit: A deep learning package for many-body potential energy representation and molecular dynamics." *Computer Physics Communications* 228 (2018): 178-184.

**[6]** Kresse, Georg, and Jürgen Furthmüller. "Efficiency of ab-initio total energy calculations for metals and semiconductors using a plane-wave basis set." *Computational materials science* 6.1 (1996): 15-50.

**[7]** Kresse, Georg, and Jürgen Furthmüller. "Efficient iterative schemes for ab initio total-energy calculations using a plane-wave basis set." *Physical review B* 54.16 (1996): 11169.

**[8]** Perdew, John P., Kieron Burke, and Matthias Ernzerhof. "Generalized gradient approximation made simple." *Physical review letters* 77.18 (1996): 3865.

**[9]** Blöchl, Peter E. "Projector augmented-wave method." *Physical review B* 50.24 (1994): 17953.

**[10]** Kresse, Georg, and Daniel Joubert. "From ultrasoft pseudopotentials to the projector augmented-wave method." *Physical review b* 59.3 (1999): 1758.

**[11]** Kingma, Diederik P., and Jimmy Ba. "Adam: A method for stochastic optimization." *arXiv preprint arXiv:1412.6980* (2014).

**[12]** Abadi, Martín, et al. "Tensorflow: Large-scale machine learning on heterogeneous distributed systems." *arXiv preprint arXiv:1603.04467* (2016).
