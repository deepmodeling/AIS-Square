**Name** : dpec.workflow.DFTtasks

**Input :** 

\- `input_dfttasks` : (`Artifact(Path)`) The path containing the lammps trajectories.

\- `nsample` : (`int`) The number of configurations extracted from the lammps trajectory to do single point calculation through cp2k.

\- `cp2k_input` : (`Artifact(Path)`) The path of cp2k's parameter file such as template.inp.

**Output :**

\- `output_dfttasks` : (`Artifact(List[Path])`) The directories where the configuration to do the single point calculation is located.

**Description :**

​This OP is extracted from **dpec** workflow.

​This OP is used to extract some configurations from the lammps trajectory and distribute them and the corresponding cp2k parameter file to different folders for subsequent cp2k single point calculations.

**Source :** 

```python
# Inputs
input_dfttasks: Artifact(type=pathlib.Path, optional=False, sub_path=True)
nsample: int
cp2k_input: Artifact(type=pathlib.Path, optional=False, sub_path=True)
# Outputs
output_dfttasks: Artifact(type=typing.List[pathlib.Path], optional=False, sub_path=True)
# Execute
    @OP.exec_sign_check
    def execute(self, op_in: OPIO) -> OPIO:
        r"""Execute the OP.

        Parameters
        ----------
        ip : dict
            Input dict with components:
            
            - input_dfttasks: (Artifact(Path)) The path containing the lammps trajectories.
            - nsample: (int) The number of configurations extracted from the lammps trajectory to do single point calculation through cp2k.
            - cp2k_input: (Artifact(Path)) The path of cp2k's parameter file such as template.inp.

        Returns
        -------
        op : dict 
            Output dict with components:
            
            - output_dfttasks: (Artifact(List[Path])) The directories where the configuration to do the single point calculation is located.
            
        """

        cwd = os.getcwd()
        os.chdir(op_in["input_dfttasks"])
        input_path = os.getcwd()
        dft_path = os.path.join(input_path,"04.dft")
        os.makedirs(dft_path)
        dft_input_path = os.path.join(dft_path,"00.inputs")
        os.makedirs(dft_input_path)
        os.chdir(dft_input_path)
        os.symlink("../../01.dpmd/02.outputs/00.run/sno2-water.xyz","sno2-water.xyz")

        shutil.copyfile(op_in["cp2k_input"],"template.inp")
        ## select conf from traj
        ls = ase.io.read("sno2-water.xyz",index=':')
        total_conf = len(ls)
        interval = int(total_conf/op_in["nsample"])
        tasks = []
        for i in range(op_in["nsample"]):
            task_path = os.path.join(dft_input_path,"task.%06d"%i)
            os.makedirs(task_path)
            tag = i*interval
            ase.io.write("task.%06d/single.xyz"%i,ls[tag])
            os.system("sed -i '1,2d' 'task.%06d/single.xyz'"%i)
            shutil.copyfile("template.inp","task.%06d/template.inp"%i)
            tasks.append(pathlib.Path(task_path))
        os.chdir(cwd)
        op_out = OPIO({
            "output_dfttasks": tasks
        })
        return op_out
```