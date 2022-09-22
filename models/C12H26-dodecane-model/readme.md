## Introduction
A DP-PBE model for n-dodecane. **[1]**

The reactive MD simulation with this DP model revealed the pyrolysis mechanism of n-dodecane in detail, and the simulation results are in good agreement with the experimental measurements. In addition, this model shows excellent transferability to different long-chain alkanes.


## Training
The DP model was trained based on the final dataset. To guarantee the accuracy of the model, the batch number of the training process was set to 4,000,000.

More details can be found in **[1]**.


## Use model
1. Download and install LAMMPS: https://docs.lammps.org/Manual.html
2. Download and install DeePMD-kit: https://github.com/deepmodeling/deepmd-kit
3. Prepare input data for LAMMPS
4. Download the model
5. Run MD with LAMMPS: https://github.com/deepmodeling/deepmd-kit/blob/master/doc/third-party/lammps.md

## References
**[1]** Zeng, Jinzhe, et al. "Exploring the chemical space of linear alkane pyrolysis via deep potential generator." Energy & Fuels 35.1 (2020): 762-769.
