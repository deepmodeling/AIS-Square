**Name** : deepks2.op.iter_op.scf_abacus_op.PrepScfAbacus

**Input :** 

\-`yaml_name`: (`str`) The name of the input yaml file. 

​\- `config_file` : (`Artifact(Path)`) The folder contains the yaml file specified in "yaml_name". See [scf_abacus.yaml](https://deepks-kit-qi.readthedocs.io/en/latest/inputs-preperation.html#scf-abacus-yaml) but no need to group the args in blocks like "scf_abacus" in that website. One may specify two more args which are `group_size` and `cpus_per_task` for parallel calculation.

​\- `tasks` : (`Artifact(Path)`) The folder contains ABACUS input files for each structure. They should be in subfolders "systems_train" and "systems_test", like "tasks/systems_train/group.00/ABACUS" and "tasks/systems_test/group.01/ABACUS" Check output `op["tasks"]` in last step `ConvertScfAbacus`. 

**Output :**

​\- `task_names`: (`List[str]`) The name of tasks. Will be used as the identities of the tasks. The names of different tasks are different.

​\- `task_paths`: (`Artifact(List[Path])`) The splited working paths of the tasks. Contains all input files needed to start the ABACUS scf calculation. The order of the Paths should be consistent with `op["task_names"]`

**Description :**

​This OP is extracted from **deepks-flow** workflow.

​This OP splits input files of ABACUS for parallel calculation. Each structure of ABACUS input folder will be renamed to prevent duplication of name. Check `RunScfAbacus` for the next step. 

**Source :** 

```python
# Inputs
yaml_name: str,
config_file: Artifact(type=pathlib.Path,optional=False, sub_path=True),
tasks: Artifact(type=pathlib.Path, optional=False, sub_path=True),
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

            - yaml_name: (str)
            - ​config_file: (Artifact(Path))
            - tasks: (Artifact(Path)) 

        Returns
        -------
        op : dict 
            Output dict with components:

            - task_names: (List[str]) The name of tasks. Will be used as the identities of the tasks. The names of different tasks are different.
            - task_paths: (Artifact(List[Path])) The parepared working paths of the tasks. Contains all input files needed to start the ABACUS scf calculation. The order fo the Paths should be consistent with op["task_names"]
            
        """

        yaml_name = ip["yaml_name"]
        config_file = ip["config_file"]
        system = ip["tasks"]

        prep_scf_config = load_yaml(config_file/yaml_name)

        group_size = prep_scf_config.pop("group_size")

        system_train = []
        system_test = []
        train_parent = system/SYS_TRAIN
        test_parent = system/SYS_TEST
        if os.path.exists(train_parent):
            for group in os.listdir(train_parent):
                system_train.append(train_parent/group)
        if os.path.exists(test_parent):
            for group in os.listdir(test_parent):
                system_test.append(test_parent/group)

        sys_train_name = [os.path.basename(s) for s in system_train]
        sys_test_name = [os.path.basename(s) for s in system_test]

        task_names = []
        task_paths = []
        counter = 0
        group_num = -1

        for group in sys_train_name:
            dir=train_parent / group / "ABACUS"
            list=os.listdir(dir)
            for this in list:
                if ((counter+1)/group_size)>(group_num+1):
                    group_num+=1
                    group_name = "train.%s"% str(group_num).zfill(2)
                    task_names.append(group_name)
                    task_paths.append(train_parent/group_name)
                shutil.copytree((Path(dir)/this).resolve(),(train_parent/group_name/"ABACUS"/(str(group)+str(this).zfill(4))))
                counter+=1

        group_num = -1
        counter = 0
        for group in sys_test_name:
            dir=test_parent / group / "ABACUS"
            list=os.listdir(dir)
            for this in list:
                if ((counter+1)/group_size)>(group_num+1):
                    group_num+=1
                    group_name = "test.%s"% str(group_num).zfill(2)
                    task_names.append(group_name)
                    task_paths.append(test_parent/group_name)
                shutil.copytree((Path(dir)/this).resolve(),(test_parent/group_name/"ABACUS"/(str(group)+str(this).zfill(4))))
                counter+=1


        op = OPIO({
            "task_names" : task_names,
            "task_paths" : task_paths,
        })
        return op
```


