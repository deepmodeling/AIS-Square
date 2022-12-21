**Name** : dpgen2.op.prep_dp_train.PrepDPTrain

**Input :** 

​            \- `template_script`: (`str` or `List[str]`) A template of the training script. Can be a `str` or `List[str]`. In the case of `str`, all training tasks share the same training input template, the only difference is the random number used to initialize the network parameters. In the case of `List[str]`, one training task uses one template from the list. The random numbers used to initialize the network parameters are differnt. The length of the list should be the same as `numb_models`.

​            \- `numb_models`: (`int`) Number of DP models to train.

**Output :**

​			\- `task_names`: (`List[str]`) The name of tasks. Will be used as the identities of the tasks. The names of different tasks are different.

​            \- `task_paths`: (`Artifact(List[Path])`) The parepared working paths of the tasks. The order fo the Paths should be consistent with `op["task_names"]`.

**Description :**

​			This OP is extracted from **dpgen2** workflow.

​			This OP prepares training scripts for DeePMD by changing random seeds in `ip['template_script']` and saving `ip['numb_models']` scripts to `op['task_paths']` with the correspounding task names in `op['task_names']`. DeePMD training process can then directly run in these paths. Check `RunDPTrain` for the next step.

**Source :** 

```python
# Inputs
template_script: typing.Union[dict, typing.List[dict]]
numb_models: int
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

            - template_script: (str or List[str]) A template of the training script. Can be a str or List[str]. In the case of str, all training tasks share the same training input template, the only difference is the random number used to initialize the network parameters. In the case of List[str], one training task uses one template from the list. The random numbers used to initialize the network parameters are differnt. The length of the list should be the same as numb_models.
            - numb_models: (int) Number of DP models to train.
        
        Returns
        -------
        op : dict
            Output dict with components:

            - task_names: (List[str]) The name of tasks. Will be used as the identities of the tasks. The names of different tasks are different.
            - task_paths: (Artifact(List[Path])) The parepared working paths of the tasks. The order fo the Paths should be consistent with op["task_names"]

        """
        template = ip['template_script']
        numb_models = ip['numb_models']
        osubdirs = []
        if type(template) != list:
            template = [template for ii in range(numb_models)]
        else:
            if not (len(template) == numb_models):
                raise RuntimeError(f'length of the template list should be equal to {numb_models}')

        for ii in range(numb_models):
            # mkdir
            subdir = Path(train_task_pattern % ii) 
            subdir.mkdir(exist_ok=True, parents=True)
            osubdirs.append(str(subdir))
            # change random seed in template
            idict = self._script_rand_seed(template[ii])
            # write input script
            fname = subdir / train_script_name
            with open(fname, 'w') as fp:
                json.dump(idict, fp, indent = 4)

        op = OPIO({
            "task_names" : osubdirs,
            "task_paths" : [Path(ii) for ii in osubdirs],
        })
        return op
```

