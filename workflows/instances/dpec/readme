# Workflow Chart
1. N dpmd simulations (N=4 in this case) (check 01.dpmd)
   1. inputs:
      1. a lammps input template
      2. an initial md structure.
      3. N dp potentials (recommend compressed potentials)
   2. Note for inputs:
      1. Each dpmd simulations must be driven by a different dp pot. Hence, I make cyclic permutation of potential names in keyword `pair_style`
   3. outputs:
      1. N `model_devi.out` for each simulation (roughly estimate if the simulations are reliable.)
      2. N `traj.xyz`, trajectory files for the dpmd simulation.
2. Visualize N Model deviation (N=4 in this case) (run `plot_model_devi.py`)
   1. inputs:
      1.  N `model_devi.out`s
   2. outputs:
      1. a model deviation plot

3. Visualize N Oxygen Density with deviation (N=4 in this case) (run `plot_density.py`)
   1. inputs:
      1. N `traj.xyz`
      2. indices of surface atoms for each surface. (two surfaces present in an interface model.)
      3. indices of water oxygen atoms.
      4. cell parameters

   2. Note for inputs:
      1. indices of surface atoms should be manually set by users (for now)
      2. cell parameters may be parsed from structure files provided by users or manual inputs from users.)

    1. outputs:
       1. a density profile plot
4. DFT calculations for sampled structures.
   1. inputs:
      1. one `traj.xyz`
      2. cp2k dft input `template.inp`
   2. Note for inputs:
      1. `traj.xyz` is used to generate `sampled.xyz`, which may contain N sampled structures. N ca. 300-400.
      2. `&V_HARTREE_CUBE ON`  should exist in `template.inp` in order to write hartree cube files.
      3. One need collect all hartree cubes and index them
   3. outputs:
      1. N hartree cubes
5. Obtain Solid and water hartree (run `get_hartree.py`)
   1. inputs:
      1. hatree cubes
   2. outputs:
      1. `solid_hartree.dat`
      2. `water_hartree.dat`
6. Obtain band alignment figure (run `get_band_alignment.py`)
   1. inputs:
      1. `solid_hartree.dat`
      2. `water_hartree.dat`
   2. outputs:
      1. `band_alignment.dat`
