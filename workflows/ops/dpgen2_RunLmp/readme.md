**Name** : dpgen2.op.run_lmp.RunLmp

**Input :** 

​            \- `config`: (`dict`) The config of lmp task. Check `RunLmp.lmp_args` for definitions.

​            \- `task_name`: (`str`) The name of the task.

​            \- `task_path`: (`Artifact(Path)`) The path that contains all input files prepareed by `PrepLmp`.

​            \- `models`: (`Artifact(List[Path])`) The frozen model to estimate the model deviation. The first model with be used to drive molecular dynamics simulation.

**Output :**

​            \- `log`: (`Artifact(Path)`) The log file of LAMMPS.

​            \- `traj`: (`Artifact(Path)`) The output trajectory.

​            \- `model_devi`: (`Artifact(Path)`) The model deviation. The order of recorded model deviations should be consistent with the order of frames in `traj`.

**Description :**

​			This OP is extracted from **dpgen2** workflow.

​			This OP runs LAMMPS simulation and model deviations with the frozen models in `ip['models']` in `ip['task_path']` with task name  `ip['task_name']` . This OP returns the LAMMPS log, trajectory and model deviation.

**Source :** 

```python
# Inputs
config: dict
task_name: str
task_path: Artifact(type=pathlib.Path, optional=False, sub_path=True)
models: Artifact(type=typing.List[pathlib.Path], optional=False, sub_path=True)
# Outputs
log: Artifact(type=pathlib.Path, optional=False, sub_path=True)
traj: Artifact(type=pathlib.Path, optional=False, sub_path=True)
model_devi: Artifact(type=pathlib.Path, optional=False, sub_path=True)
plm_output: Artifact(type=pathlib.Path, optional=True, sub_path=True)
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
        
            - config: (dict) The config of lmp task. Check RunLmp.lmp_args for definitions.
            - task_name: (str) The name of the task.
            - task_path: (Artifact(Path)) The path that contains all input files prepareed by PrepLmp.
            - models: (Artifact(List[Path])) The frozen model to estimate the model deviation. The first model with be used to drive molecular dynamics simulation.

        Returns
        -------
            Output dict with components:
        
            - log: (Artifact(Path)) The log file of LAMMPS.
            - traj: (Artifact(Path)) The output trajectory.
            - model_devi: (Artifact(Path)) The model deviation. The order of recorded model deviations should be consistent with the order of frames in traj.
        
        Exceptions
        ----------
        TransientError
            On the failure of LAMMPS execution. Handle different failure cases? e.g. loss atoms.
        """
        config = ip['config'] if ip['config'] is not None else {}
        config = RunLmp.normalize_config(config)
        command = config['command']
        task_name = ip['task_name']
        task_path = ip['task_path']
        models = ip['models']
        input_files = [lmp_conf_name, lmp_input_name]
        input_files = [(Path(task_path)/ii).resolve() for ii in input_files]
        model_files = [Path(ii).resolve() for ii in models]
        work_dir = Path(task_name)

        with set_directory(work_dir):
            # link input files
            for ii in input_files:
                iname = ii.name
                Path(iname).symlink_to(ii)
            # link models
            for idx,mm in enumerate(model_files):
                mname = model_name_pattern % (idx)
                Path(mname).symlink_to(mm)
            # run lmp
            command = ' '.join([command, '-i', lmp_input_name, '-log', lmp_log_name])
            ret, out, err = run_command(command, shell=True)
            if ret != 0:
                raise TransientError(
                    'lmp failed\n',
                    'out msg', out, '\n',
                    'err msg', err, '\n'
                )
            
        return OPIO({
            "log" : work_dir / lmp_log_name,
            "traj" : work_dir / lmp_traj_name,
            "model_devi" : work_dir / lmp_model_devi_name,
        })   
```

