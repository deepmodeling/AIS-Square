## Introduction
Water model trained with data obtained from SCAN0 DFT calculations. **[1]**

## Training
To construct an accurate and transferable SCAN0 path-integral deep potential model with a minimal number of the expensive SCAN0 DFT data, an active machine learning procedure called deep potential generator (DP-GEN)**[2,3]** was adopted.

One converged deep potential model was employed to conduct the production run, which was trained by using 7349 SCAN0 DFT configurations. Note that the number of training configurations adopted here is not the minimum number of configurations needed for convergence because, in the first few iterations, configurations that have ζ > 0.05 eV/Å were also added to the training data set. These "extra" configurations are not essential for the convergence and will not affect the simulation results.

More details can be found in **[1]**.


## Use model
1. Download and install LAMMPS: https://docs.lammps.org/Manual.html
2. Download and install DeePMD-kit: https://github.com/deepmodeling/deepmd-kit
3. Prepare input data for LAMMPS
4. Download the model
5. Run MD with LAMMPS: https://github.com/deepmodeling/deepmd-kit/blob/master/doc/third-party/lammps.md


## References
**[1]** Zhang, Chunyi, et al. "Modeling liquid water by climbing up Jacob’s ladder in density functional theory facilitated by using deep neural network potentials." The Journal of Physical Chemistry B 125.41 (2021): 11444-11456.

**[2]** Zhang, Linfeng, et al. "Active learning of uniformly accurate interatomic potentials for materials simulation." Physical Review Materials 3.2 (2019): 023804.

**[3]** Zhang, Yuzhi, et al. "DP-GEN: A concurrent learning platform for the generation of reliable deep learning based potential energy models." Computer Physics Communications 253 (2020): 107206.


