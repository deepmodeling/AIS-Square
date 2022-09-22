## Introduction
An accurate PES model for the Al-Cu-Mg system. **[1]**

This DP model gives predictions consistent with first-principles calculations for various binary and ternary systems on their fundamental energetic and mechanical properties, including formation energy, equilibrium volume, equation of state, interstitial energy, vacancy and surface formation energy, as well as elastic moduli. Extensive benchmark shows that the DP model is ready and will be useful for atomistic modeling of the Al-Cu-Mg system within the full range of concentration.

## Training
The DeePMD-kit package **[2]** is used for training.

The sizes of the embedding and fitting nets are set to (25, 50, 100) and (240, 240, 240). During the DP-GEN iterations, the cut-off radius is set to 6 Å. The learning rate starts from $1 × 10^{−3}$ and exponentially decays to $3.5 × 10^{−8}$ after $1 × 10^6$ training steps.

Four models are trained to construct the model ensemble. Using the same architecture and the same training data set, they only differ in the random seeds for initializing model parameters.

After the DP-GEN iterations being converged, the production models are trained with the cut-off radius set to 9 Å and training steps set to $1.6 × 10^7$. In all training tasks, the Adam stochastic gradient descent method is used with default hyper parameter settings provided by the TensorFlow package. **[3]**

More details can be found in **[1]**.


## Use model
1. Download and install LAMMPS: https://docs.lammps.org/Manual.html
2. Download and install DeePMD-kit: https://github.com/deepmodeling/deepmd-kit
3. Prepare input data for LAMMPS
4. Download the model
5. Run MD with LAMMPS: https://github.com/deepmodeling/deepmd-kit/blob/master/doc/third-party/lammps.md

## References
**[1]** Jiang, Wanrun, et al. "Accurate deep potential model for the Al–Cu–Mg alloy in the full concentration space." Chinese Physics B 30.5 (2021): 050706.

**[2]** Wang, Han, et al. "DeePMD-kit: A deep learning package for many-body potential energy representation and molecular dynamics." Computer Physics Communications 228 (2018): 178-184.

**[3]** Abadi, Martín, et al. "Tensorflow: Large-scale machine learning on heterogeneous distributed systems." arXiv preprint arXiv:1603.04467 (2016).

