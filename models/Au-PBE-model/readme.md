## Introduction
A DP-PBE model for Au. **[1]**

The reported surface properties in this paper are blind to the potential modeling in the sense that none of the surface configurations is explicitly included in the training database, therefore, the reported potential is expected to have a strong ability of generalization to a wide range of properties and to play a key role in the investigation of nanostructured Ag-Au evolution, where the accurate descriptions of free surfaces are necessary.

## Training
The DP-PBE model is constructed using a smooth edition of the Deep Potential model **[2]**. The DeePMD-kit package is used for training. In each iteration, four models are trained simultaneously using the same data set, with the only difference being the random seeds employed to initialize the model parameters. The sizes of the embedding and fitting nets are set to (25, 50, 100) and (240, 240, 240), respectively. The cut-off radius is set to 6 Å. The Adam stochastic gradient descent method55 with the default hyperparameter settings provided by the TensorFlow package was used to train the DP models.

The starting and final learning rate was $1 × 10^{−3}$ and $5 × 10^{−8}$, respectively. In each DP-GEN **[3,4]** iteration the DP model is trained with $4.0 × 10^5$ steps. After the DP-GEN iterations converge, the final productive models are trained with $8.0 × 10^6$ steps.

More details can be found in **[1]**.


## Transfer learning
In the training step, the same training strategy as that used in the initialization stage of the transfer learning is adopted.


## Use model
1. Download and install LAMMPS: https://docs.lammps.org/Manual.html
2. Download and install DeePMD-kit: https://github.com/deepmodeling/deepmd-kit
3. Prepare input data for LAMMPS
4. Download the model
5. Run MD with LAMMPS: https://github.com/deepmodeling/deepmd-kit/blob/master/doc/third-party/lammps.md

## References
**[1]** Wang, YiNan, et al. "A generalizable machine learning potential of Ag–Au nanoalloys and its application to surface reconstruction, segregation and diffusion." Modelling and Simulation in Materials Science and Engineering 30.2 (2021): 025003.

**[2]** Zhang, Linfeng, et al. "End-to-end symmetry preserving inter-atomic potential energy model for finite and extended systems." Advances in Neural Information Processing Systems 31 (2018).

**[3]** Zhang, Linfeng, et al. "Active learning of uniformly accurate interatomic potentials for materials simulation." Physical Review Materials 3.2 (2019): 023804.

**[4]** Zhang, Yuzhi, et al. "DP-GEN: A concurrent learning platform for the generation of reliable deep learning based potential energy models." Computer Physics Communications 253 (2020): 107206.


