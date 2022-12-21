**Name** : dpgen2.op.run_dp_train.RunDPTrain

**Input :** 

​            \- `config`: (`dict`) The config of training task. Check `RunDPTrain.training_args` for definitions.

​            \- `task_name`: (`str`) The name of training task.

​            \- `task_path`: (`Artifact(Path)`) The path that contains all input files prepareed by `PrepDPTrain`.

​            \- `init_model`: (`Artifact(Path)`) A frozen model to initialize the training.

​            \- `init_data`: (`Artifact(List[Path])`) Initial training data.

​            \- `iter_data`: (`Artifact(List[Path])`) Training data generated in the DPGEN iterations.

**Output :**

​            \- `script`: (`Artifact(Path)`) The training script.

​            \- `model`: (`Artifact(Path)`) The trained frozen model.

​            \- `lcurve`: (`Artifact(Path)`) The learning curve file.

​            \- `log`: (`Artifact(Path)`) The log file of training.

**Description :**

​			This OP is extracted from **dpgen2** workflow.

​			This OP runs training and freezing process of DeePMD in `ip['task_path']` with task name  `ip['task_name']` . This OP returns the script, frozen model, learning curve and log.

**Source :** 

```python
# Inputs
config: dict
task_name: str
task_path: Artifact(type=pathlib.Path, optional=False, sub_path=True)
init_model: Artifact(type=pathlib.Path, optional=True, sub_path=True)
init_data: Artifact(type=typing.List[pathlib.Path], optional=False, sub_path=True)
iter_data: Artifact(type=typing.List[pathlib.Path], optional=False, sub_path=True)
# Outputs
script: Artifact(type=pathlib.Path, optional=False, sub_path=True)
model: Artifact(type=pathlib.Path, optional=False, sub_path=True)
lcurve: Artifact(type=pathlib.Path, optional=False, sub_path=True)
log: Artifact(type=pathlib.Path, optional=False, sub_path=True)
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
        
            - config: (dict) The config of training task. Check RunDPTrain.training_args for definitions.
            - task_name: (str) The name of training task.
            - task_path: (Artifact(Path)) The path that contains all input files prepareed by PrepDPTrain.
            - init_model: (Artifact(Path)) A frozen model to initialize the training.
            - init_data: (Artifact(List[Path])) Initial training data.
            - iter_data: (Artifact(List[Path])) Training data generated in the DPGEN iterations.

        Returns
        -------
            Output dict with components:
        
            - script: (Artifact(Path)) The training script.
            - model: (Artifact(Path)) The trained frozen model.
            - lcurve: (Artifact(Path)) The learning curve file.
            - log: (Artifact(Path)) The log file of training.
        
        Exceptions
        ----------
        FatalError
            On the failure of training or freezing. Human intervention needed.
        """
        config = ip['config'] if ip['config'] is not None else {}
        config = RunDPTrain.normalize_config(config)
        task_name = ip['task_name']
        task_path = ip['task_path']
        init_model = ip['init_model']
        init_data = ip['init_data']
        iter_data = ip['iter_data']
        iter_data_old_exp = _expand_all_multi_sys_to_sys(iter_data[:-1])
        iter_data_new_exp = _expand_all_multi_sys_to_sys(iter_data[-1:])
        iter_data_exp = iter_data_old_exp + iter_data_new_exp
        work_dir = Path(task_name)

        # update the input script
        input_script = Path(task_path)/train_script_name
        with open(input_script) as fp:
            train_dict = json.load(fp)
        if "systems" in train_dict['training']:
            major_version = "1"
        else:
            major_version = "2"

        # auto prob style
        do_init_model = RunDPTrain.decide_init_model(config, init_model, init_data, iter_data)
        auto_prob_str = "prob_sys_size"
        if do_init_model:
            old_ratio = config['init_model_old_ratio']
            numb_old = len(init_data) + len(iter_data_old_exp)
            numb_new = numb_old + len(iter_data_new_exp)
            auto_prob_str = f"prob_sys_size; 0:{numb_old}:{old_ratio}; {numb_old}:{numb_new}:{1.-old_ratio:g}"

        # update the input dict
        train_dict = RunDPTrain.write_data_to_input_script(
            train_dict, init_data, iter_data_exp, auto_prob_str, major_version)
        train_dict = RunDPTrain.write_other_to_input_script(
            train_dict, config, do_init_model, major_version)        

        with set_directory(work_dir):
            # open log
            fplog = open('train.log', 'w')
            def clean_before_quit():
                fplog.close()

            # dump train script
            with open(train_script_name, 'w') as fp:
                json.dump(train_dict, fp, indent=4)

            # train model
            if do_init_model:
                command = ['dp', 'train', '--init-frz-model', str(init_model), train_script_name]
            else:
                command = ['dp', 'train', train_script_name]
            ret, out, err = run_command(command)
            if ret != 0:
                clean_before_quit()
                raise FatalError(
                    'dp train failed\n',
                    'out msg', out, '\n',
                    'err msg', err, '\n'
                )
            fplog.write('#=================== train std out ===================\n')
            fplog.write(out)
            fplog.write('#=================== train std err ===================\n')
            fplog.write(err)

            # freeze model
            ret, out, err = run_command(['dp', 'freeze', '-o', 'frozen_model.pb'])
            if ret != 0:
                clean_before_quit()
                raise FatalError(
                    'dp freeze failed\n',
                    'out msg', out, '\n',
                    'err msg', err, '\n'
                )
            fplog.write('#=================== freeze std out ===================\n')
            fplog.write(out)
            fplog.write('#=================== freeze std err ===================\n')
            fplog.write(err)

            clean_before_quit()
        
        return OPIO({
            "script" : work_dir / train_script_name,
            "model" : work_dir / "frozen_model.pb",
            "lcurve" : work_dir / "lcurve.out",
            "log" : work_dir / "train.log",
        })
```

