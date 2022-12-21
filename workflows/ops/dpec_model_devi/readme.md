**Name** : dpec.workflow.modeldevi

**Input :** 

\- `input_devi` : (`Artifact(Path)`) The path containing a series of lammps trajectories.The lammps traj paths are `./01.dpmd/00.run` `./01.dpmd/01.run` and so on.

\- `nmodel` : (`int`) The number of models and also the number of lammps trajectories.

**Output :**

\- `output_devi` : (`Artifact(Path)`) It is the same path as `input_devi` and contains the model deviation plot.

**Description :**

​This OP is extracted from **dpec** workflow.

​This OP is used to draw a model deviation plot according to a series of lammps trajectories.

**Source :** 

```python
# Inputs
input_devi: Artifact(type=pathlib.Path, optional=False, sub_path=True)
nmodel : int
# Outputs
output_devi: Artifact(type=pathlib.Path, optional=False, sub_path=True)
# Execute
    @OP.exec_sign_check
    def execute(self, op_in: OPIO) -> OPIO:
        r"""Execute the OP.

        Parameters
        ----------
        ip : dict
            Input dict with components:
            
            - input_devi: (Artifact(Path)) The path containing multiple lammps trajs.The lammps traj paths are `./01.dpmd/00.run` `./01.dpmd/01.run` and so on.
            - nmodel: (int) The number of models and also the number of lammps trajs.

        Returns
        -------
        op : dict 
            Output dict with components:
            
            - output_devi: (Artifact(Path)) It is the same path as `input_devi` and contains the drawn model_devi picture.
            
        """
        cwd = os.getcwd()
        os.chdir(op_in["input_devi"])
        input_path = os.getcwd()
        devi_path = os.path.join(input_path,"02.model_devi_plots")
        os.makedirs(devi_path)
        devi_input_path = os.path.join(devi_path,"00.inputs")
        os.makedirs(devi_input_path)
        devi_output_path = os.path.join(devi_path,"01.outputs")
        os.makedirs(devi_output_path)
        os.chdir(devi_input_path)
        # here is inputs
        mdev_file_list = []
        for ii in range(op_in["nmodel"]):
            os.system("ln -s ../../01.dpmd/02.outputs/%02d.run/ %02d.run"%(ii,ii))
            mdev_file_list.append("%02d.run/model_devi.out"%(ii))

        row = 1
        col = 1
        fig = plt.figure(figsize=(8*col,4.5*row), dpi=150, facecolor='white')
        gs = fig.add_gridspec(row,col)
        ax  = fig.add_subplot(gs[0])
        for mdev_file in mdev_file_list:
            steps, max_mdevs = parse_mdev_file(mdev_file=mdev_file)
            ax.scatter(steps, max_mdevs, s=5)

            ax.set_ylabel("Max Force Model Deviation [eV/A]")
            ax.set_xlabel("Steps")
            ax.tick_params(direction='in')
        fig.savefig("../01.outputs/model_devi_plot.png", dpi=400)

        os.chdir(cwd)
        op_out = OPIO({
            "output_devi": op_in["input_devi"]
        })
        return op_out
```
