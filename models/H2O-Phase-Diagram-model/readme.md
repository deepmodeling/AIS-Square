## Introduction
Water model that reproduces accurately the potential energy surface of the SCAN approximation of density functional theory, from low temperature and pressure to about 2400 K and 50 GPa, excluding the vapor stability region. **[1]**

Water phase diagram is predicted using molecular dynamics and satisfactory overall agreement with experimental results is obtained.


## Training
A trial DP is built from configurations of the liquid, at ambient conditions, and of all the experimentally known stable and metastable ice polymorphs for P $\lesssim$ 50 GPa (Ih, Ic, II, III, IV, V, VI, VII, VIII, IX, XI, XII, XIII, XIV, XV).

The model is used by DP Generator **[2,3]** to explore a wide region of the phase space with isothermal-isobaric (NPT) DPMD trajectories. The protocol is iterated to refine the model with new DFT data until satisfactory accuracy is achieved.

The visited states can be roughly classified into three groups: the low pressure (A), the high pressure (B), and the superionic group (C). Group (A) includes states in the range 50 ≤ T ≤ 600 K and $10^{−4}$ ≤ P ≤ 5 GPa, starting from configurations of the fluid and of all the ices except VII and VIII. Group (B) includes states in the range 50 ≤ T ≤ 600 K and 0.1 ≤ P ≤ 50 GPa, starting from configurations of ice VII and VIII. Group (C) includes states in the range 200 ≤ T ≤ 2400 K and 1 ≤ P ≤ 50 GPa, starting from ice VII and the fluid.

DPMD samples almost uniformly the thermodynamic domains of the three groups. The deviation in the predicted forces within a set of representative DP models is used to label configurations for which new DFT calculations of the energy, forces, and virial are necessary. The new data are added to the training dataset and serve to refine the representative DP models entering the next iteration.

After 36 concurrent learning iterations the error in the force is satisfactorily reduced and the procedure ends. The accumulated number of snapshots in the training dataset is 31058, a tiny fraction (∼ 0.05%) of the configurations visited by DPMD. At this point, the relative energies of configurations within each phase are well described, but deviations from DFT still affect the averages. To reduce these deviations below a small threshold, 3519 additional training configurations are necessary.

More details can be found in **[1]**.


## Use model
1. Download and install LAMMPS: https://docs.lammps.org/Manual.html
2. Download and install DeePMD-kit: https://github.com/deepmodeling/deepmd-kit
3. Prepare input data for LAMMPS
4. Download the model
5. Run MD with LAMMPS: https://github.com/deepmodeling/deepmd-kit/blob/master/doc/third-party/lammps.md


## References
**[1]** Zhang, Linfeng, et al. "Phase diagram of a deep potential water model." Physical review letters 126.23 (2021): 236001.

**[2]** Zhang, Linfeng, et al. "Active learning of uniformly accurate interatomic potentials for materials simulation." Physical Review Materials 3.2 (2019): 023804.

**[3]** Zhang, Yuzhi, et al. "DP-GEN: A concurrent learning platform for the generation of reliable deep learning based potential energy models." Computer Physics Communications 253 (2020): 107206.


