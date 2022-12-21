**Name** : dpgen2.op.prep_lmp.PrepLmp

**Input :** 

​            \- `lmp_task_grp` : (`Artifact(Path)`) Can be pickle loaded as a ExplorationTaskGroup. Definitions for LAMMPS tasks

**Output :**

​            \- `task_names`: (`List[str]`) The name of tasks. Will be used as the identities of the tasks. The names of different tasks are different.

​            \- `task_paths`: (`Artifact(List[Path])`) The parepared working paths of the tasks. Contains all input files needed to start the LAMMPS simulation. The order fo the Paths should be consistent with `op["task_names"]`

**Description :**

​			This OP is extracted from **dpgen2** workflow.

​			This OP prepares input files needed to start the LAMMPS simulation in `op['task_paths']` with the correspounding task names in `op['task_names']`. Check `RunLmp` for the next step.

**Source :** 

```python
# Inputs
lmp_task_grp: BigParameter(type=pathlib.Path)
# Outputs
task_names: typing.List[str]
task_paths: Artifact(type=typing.List[pathlib.Path], optional=False, sub_path=True)
# Execute
    @OP.exec_sign_check
    def execute(
            self,
            ip : OPIO,
    ) -> OPIO:
        r"""Execute the OP.

        Parameters
        ----------
        ip : dict
            Input dict with components:
            - lmp_task_grp : (Artifact(Path)) Can be pickle loaded as a ExplorationTaskGroup. Definitions for LAMMPS tasks
        
        Returns
        -------
        op : dict 
            Output dict with components:

            - task_names: (List[str]) The name of tasks. Will be used as the identities of the tasks. The names of different tasks are different.
            - task_paths: (Artifact(List[Path])) The parepared working paths of the tasks. Contains all input files needed to start the LAMMPS simulation. The order fo the Paths should be consistent with op["task_names"]
        """

        lmp_task_grp = ip['lmp_task_grp']
        cc = 0
        task_paths = []
        for tt in lmp_task_grp:
            ff = tt.files()
            tname = _mk_task_from_files(cc, ff)
            task_paths.append(tname)
            cc += 1
        task_names = [str(ii) for ii in task_paths]
        return OPIO({
            'task_names' : task_names,
            'task_paths' : task_paths,
        })
```

