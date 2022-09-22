## Introduction
A DP model for NaCl. **[1]**

The DPs also provide results with accuracy that is comparable to that of AIMD and efficiency that is similar to that of empirical potentials. The partial radial distribution functions and angle distribution functions predicted using the DPs are in close agreement with those derived from AIMD. The estimated densities, self-diffusion coefficients, shear viscosities, and electrical conductivities also matched well with the AIMD and experimental data.


## Training
The sizes of the embedding and fitting neural network were set to be {2,550,100} and {240,240,240}, respectively. The smooth cutoff parameter ( $r_{cs}$ ) is chosen to 6.8 Å, and the cut-off radius ( $r_c$ ) was set to 7 Å. For the tunable prefactors in the loss functions, $p^{start}_ ε$ , $p^{start}_f$ , $p^{limit}_ε$ , and $p^{limit}_f$ are 0.02, 1000, 1, and 1, respectively.

More details can be found in **[1]**.


## Use model
1. Download and install LAMMPS: https://docs.lammps.org/Manual.html
2. Download and install DeePMD-kit: https://github.com/deepmodeling/deepmd-kit
3. Prepare input data for LAMMPS
4. Download the model
5. Run MD with LAMMPS: https://github.com/deepmodeling/deepmd-kit/blob/master/doc/third-party/lammps.md

## References
**[1]** Liang, Wenshuo, Guimin Lu, and Jianguo Yu. "Theoretical prediction on the local structure and transport properties of molten alkali chlorides by deep potentials." Journal of Materials Science & Technology 75 (2021): 78-85.
