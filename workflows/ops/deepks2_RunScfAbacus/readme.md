**Name** : deepks2.op.iter_op.scf_abacus_op.RunScfAbacus

**Input :**

\- `no_model` : (`bool`) Tell this OP if there is an input model of ABACUS format.

\- `yaml_name` : (`str`) The name of the input yaml file. 

​\- `config_file` : (`Artifact(Path)`) The folder contains the yaml file specified in "yaml_name". See [scf_abacus.yaml](https://deepks-kit-qi.readthedocs.io/en/latest/inputs-preperation.html#scf-abacus-yaml) but no need to group the args in blocks like "scf_abacus" in that website. One may specify two more args which are `group_size` and `cpus_per_task` for parallel calculation.

\- `stru_file` : (`Artifact(Path)`) The folder contains the "\*.orb" and "\*.upf" which specified in yaml file.

​\- `task_path` : (`Artifact(Path)`) The sliced working path of the tasks. Check output `op["task_paths"]` in last step `ConvertScfAbacus`.

\- `model` : (`Artifact(Path, optional=True)`) Path to the model file of ABACUS format. Check output `op["model"]` in last step `ConvertScfAbacus`.

**Output :**

\- `task_path` : (`Artifact(Path)`) The folder contains ABACUS output files for each structure. Check next step `GatherStatsScfAbacus` to change back the structure folder name. 

**Description :**

​This OP is extracted from **deepks-flow** workflow.

​This OP run the ABACUS calculation. One should modify the ABACUS run command in source code, which vary with your container image. Check `GatherStatsScfAbacus` for the next step. 

**Source :** 

```python
# Inputs
no_model: bool
yaml_name: str
​config_file: Artifact(type=pathlib.Path, optional=False, sub_path=True)
stru_file: Artifact(type=pathlib.Path, optional=False, sub_path=True)
​task_path: Artifact(type=pathlib.Path, optional=False, sub_path=True)
model: Artifact(type=pathlib.Path, optional=True, sub_path=True)
# Outputs
task_path: Artifact(type=pathlib.Path, optional=False, sub_path=True)
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

            - no_model: (bool)
            - yaml_name: (str)
            - config_file: (Artifact(Path)) 
            - stru_file: (Artifact(Path))
            - task_path: (Artifact(Path))
            - model: (Artifact(Path, optional=True))

        Returns
        -------
        op : dict 
            Output dict with components:

            - task_path: (Artifact(Path))
            
        """

        cwd = os.getcwd()
        no_model = ip["no_model"]
        yaml_name = ip["yaml_name"]
        config_file = ip["config_file"]
        stru_file = ip["stru_file"]
        task_path = ip["task_path"]
        model = ip["model"]

        run_scf_config = load_yaml(config_file/yaml_name)

        run_cmd = run_scf_config.pop("run_cmd")
        abacus_path = run_scf_config.pop("abacus_path")
        cpus_per_task = run_scf_config.pop("cpus_per_task")

        shutil.copytree(stru_file, task_path,dirs_exist_ok = True)
        if not no_model:
            shutil.copy(model, task_path)
        
        os.chdir(task_path)

        for f in os.listdir(task_path/"ABACUS"):
            print(f)
            cmd=str(f"ulimit -s unlimited && \
                . /opt/intel/oneapi/setvars.sh && \
                cd ABACUS/{f}/ &&  \
                {run_cmd} -n {cpus_per_task} {abacus_path} > {SCF_OUT_LOG} 2>{SCF_ERR_LOG}  &&  \
                echo {f}`grep convergence ./OUT.ABACUS/running_scf.log` > conv  &&  \
                echo {f}`grep convergence ./OUT.ABACUS/running_scf.log`")
            p = subprocess.Popen(
                cmd, shell=True, executable='/bin/bash')
            p.wait()
        
        os.chdir(cwd)

        return OPIO({
            "task_path": task_path,
            # "log_scf" : task_path/outlog,
            # "err" : task_path/errlog,
        })
```