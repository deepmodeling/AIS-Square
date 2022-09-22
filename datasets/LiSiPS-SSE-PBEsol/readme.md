## Introduction

This is the PBsol dataset of $Li_{10}SiP_2S_{12}$ used in **[1]**. 

## Generation Approach
The dataset is generated with DP-GEN **[2,3]**,  a concurrent learning scheme that generates the training dataset iteratively.

Using the Deep Potential Generator, a minimal set of training data is generated via an efficient and sufficient sampling process, thereby guaranteeing a reliable PES model produced by training. The flowchart of DP-GEN iteration is shown in Fig. 2.

<p align="center">
<img src="https://github.com/deepmodeling/AIS-Square/blob/main/datasets/LiGePS-SSE-PBE/figs/Fig2.png?raw=true" width=80% />
</p>  

In the exploration step, model deviations are evaluated using the ensemble of trained models and new configurations are picked according to the maximum
deviation of forces ( $\sigma^{max}_f$ ), defined as:

$$\sigma_{f}^{\max}=\max_{i}\sqrt{\left\langle\left\|f_{i}-\left\langle f_{i}\right\rangle\right\|^{2}\right\rangle}$$

where $f_i$ is the force acting on atom $i$, and $\langle ... \rangle$ denotes the average of the DP model ensemble. Configurations with small force deviations ( $\sigma^{max}_f < \sigma_{low}$, yellow square in Fig. 2(c)) are effectively covered by the training dataset with high probability. On the contrary, excessive force deviation ( $\sigma^{max}_f > \sigma_{high}$, red cross in Fig. 2(c)) implies that the configuration may diverge from the relevant physical trajectories. Therefore none of them are picked. Only configurations whose $\sigma^{max}_f$ fall between a predetermined window are labeled as candidates (blue circles in Fig. 2(c)). In practice, after running several MD trajectories, the selection criterion usually produces hundreds or thousands of candidates. A small fraction of them is representative enough to improve the model, and therefore a cutoff number ( $N^{max}_{label}$ ) is set to restrict the number of candidates. These candidates are labeled and added to the original dataset for the next training. The labeling and training stages are rather standard, while there is large flexibility for the sampling strategy on how to explore the relevant configuration space in each iteration. According to Ref.**[3]**, a practical rule of thumb is to set $\sigma_{low}$ slightly larger than the training error achieved by the model,and set $\sigma_{high}$ 0.1-0.3 eV/Å higher than $\sigma_{low}$. In this paper, $\sigma_{low}$ and $\sigma_{high}$ are set to 0.12 and 0.25 eV/Å, respectively.  

**Exploration settings of DP-GEN iterations.**
| Ensemble  | Iteration | Temperature(K) | Structure | Supercell |
| --------- | --------- | -------------- | --------- | --------- |
| NpT | 1-4   | 50, 100, 200, 300, 500, 700, 900, 1200  | ordered DFT-relaxed    | 1×1×1 |
| NpT | 5-8   | 300, 700, 1200                          | ordered DFT-relaxed    | 2×2×2 |
| NpT | 9-12  | 50, 100, 200, 300, 500, 700, 900, 1200  | disordered DFT-relaxed | 1×1×1 |
| NpT | 13-16 | 300, 700, 1200                          | disordered DFT-relaxed | 2×2×2 |
| NpT | 19-21 | 50, 100, 200, 300, 500, 700, 900, 1200  | ordered experiment     | 2×2×2 |

In this work, all crystal structures are fetched from the Materials Project database as conventional cells. The material ID of them in the Materials Project are: mp-696138
( $Li_{10}Ge{PS_6}_2$ ), mp-696129 ( $Li_{10}SiP_{2}S_{12}$ ) and mp-696123( $Li_{10}SnP_{2}S_{12}$ ). Structure manipulation are dealt with pymatgen.

The DP-GEN is started with 590 structures that are generated via slightly perturbing DFT-relaxed structures. A smooth version of Deep Potential (v1.2)**[4,5]**, which is end-to-end, i.e. capable of fitting many-component data of SSE materials with little human intervention, was used for the training step. The exploration is run on 5 systems step by step. Each system is composed of 3 or 4 iterations depending on its convergence. The exploration time of each system is gradually lengthened from 1000 fs to 10000 fs. The exploration is beginning with ordered structures relaxed by DFT (i.e. structures downloaded from the Materials Project database, in which the position of Ge/Si/Sn/P atoms are fixed). Then the exploration is changed to disordered structures whose 4d sites are randomly occupied by Ge/Si/Sn/P. Exploration with the NpT ensemble means that configurations with different lattice parameters could be automatically sampled. And finally the exploration is performed in the systems with experimental lattice parameters. The exploration of each system is considered converged when the percentage of accurate configurations is larger than 99.5%. The detailed settings of the DP-GEN are listed in Table I. The production DP models are trained with 10 times longer steps, in which the number of batch and the step of learning rate decay are set to 4000000 and 20000, respectively.

## Data Format
The directory tree is as follows:

```
-- datainit
-- -- system.xxx
-- iter.xxxxxxx
-- -- 02.fp
-- -- -- data.xxx
```

where each subdirectory(`system.xxx` or `data.xxx`) contains unit systems in DeePMD format.

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
**[1]** Huang, Jianxing, et al. "Deep potential generation scheme and simulation protocol for the Li10GeP2S12-type superionic conductors." The Journal of Chemical Physics 154.9 (2021): 094703.

**[2]** Zhang, Linfeng, et al. "Active learning of uniformly accurate interatomic potentials for materials simulation." Physical Review Materials 3.2 (2019): 023804.

**[3]** Zhang, Yuzhi, et al. "DP-GEN: A concurrent learning platform for the generation of reliable deep learning based potential energy models." Computer Physics Communications 253 (2020): 107206.

**[4]** Zhang, Linfeng, et al. "End-to-end symmetry preserving inter-atomic potential energy model for finite and extended systems." Advances in Neural Information Processing Systems 31 (2018).

**[5]** Wang, Han, et al. "DeePMD-kit: A deep learning package for many-body potential energy representation and molecular dynamics." Computer Physics Communications 228 (2018): 178-184.