**Name** : fpop.preprun_fp.PrepRunFp

**Type** : `superop`

**Input :** 

\-`inputs`: (`object`) The class object handels all other input files of the task. For example, pseudopotential file, k-point file and so on.

\- `type_map` : (`List[str]`) The list of elements. 

\- `backward_list` : (`List[str]`) The output files the users need.

\- `confs` : (`Artifact(List[Path])`) Configurations for the FP tasks. Stored in folders as formats which can be read by dpdata.System. 

\- `optional_artifact` : (`Artifact(Dict[str,Path])`) Other files that users or developers need.The using method of this part are defined by different developers.For example, in vasp part, all the files which are given in optional_artifact will be copied to the working directory.

\- `prep_image_config` : (`dict`) It defines the parameters of the process of preparing FP tasks.

\- `run_image_config` : (`dict`) It defines the runtime configuration of the FP task.

\- `optional_input` : (`dict`) Other parameters the developers or users may need.

\- `log_name` : (`str`) The name of log file.

\- `backward_dir_name` : (`str`) The name of the directory which contains the backward files.

**Output :**

\- `backward_dirs`: (`The directories which contain the files users need.`)

**Input of constructing superop :**

\- `name` : The name of this superop.

\- `prep_op` : The OP of the step of preparing first-principles tasks.

\- `run_op` : The OP of the step of running first-principles tasks.

\- `prep_image` : The image of the step of preparing first-principles tasks.

\- `run_image` : The image of the step of running first-principles tasks.

\- `prep_template_config` : The parameters of template level in prepare step.

\- `prep_step_config` :  The parameters of step level in prepare step.

\- `run_template_config` : The parameters of template level in running step.

\- `run_slice_config` : The parameters of slice level in running step.

\- `run_step_config` : The parameters of step level in running step.

\- `upload_python_packages` : The python packages which need to be uploaded.

**Description :**

This OP is extracted from **fpop** project.

`​fp` stands for first-principles calculation and `op` stands for operators. The abbreviation `fpop` stands for operators related to first-principles calculation.This project is based on dflow which is a Python framework for constructing scientific computing workflows (e.g. concurrent learning workflows) employing Argo Workflows as the workflow engine.

`PrepRunFp` stands for the process of preparing first-principles tasks and running these tasks.

**Usage :**

One can easily install this superop :

```shell
pip install fpop
```

And then one can use it :

```python
from fpop.preprun_fp import PrepRunFp
```

**Example :**

This is an [example](https://github.com/deepmodeling/fpop/blob/main/examples/vasp/preprunvasp.py) of using this superop.

**UT :**

This is the [unittests](https://github.com/deepmodeling/fpop/tree/main/tests) of this superop. 

**Source :** 

```python
prep_fp = Step(
    'prep-fp' , 
    template=PythonOPTemplate(
        prep_op,
        output_artifact_archive={
            "task_paths": None
        },
        python_packages = upload_python_packages,
        image = prep_image,
        **prep_template_config,
    ),
    parameters={
        "prep_image_config" : prep_run_steps.inputs.parameters["prep_image_config"],
        "inputs" : prep_run_steps.inputs.parameters["inputs"],
        "type_map" : prep_run_steps.inputs.parameters["type_map"],
        "optional_input" : prep_run_steps.inputs.parameters["optional_input"],
    },
    artifacts={
        "confs" : prep_run_steps.inputs.artifacts['confs'],
        "optional_artifact" : prep_run_steps.inputs.artifacts['optional_artifact'],
    },
    key = step_keys['prep-fp'],
    executor = prep_executor,
    **prep_step_config,    
)
prep_run_steps.add(prep_fp)

run_fp = Step(
    'run-fp',
    template=PythonOPTemplate(
        run_op,
        slices = Slices(
            "int('{{item}}')",
            input_parameter = ["task_name"],
            input_artifact = ["task_path"],
            output_artifact = ["backward_dir"],
            **run_slice_config,            
        ),
        python_packages = upload_python_packages,
        image = run_image,
        **run_template_config,
    ),
    parameters={
        "run_image_config" : prep_run_steps.inputs.parameters["run_image_config"],
        "task_name" : prep_fp.outputs.parameters["task_names"],
        "backward_list" : prep_run_steps.inputs.parameters["backward_list"],
        "log_name" : prep_run_steps.inputs.parameters["log_name"],
        "backward_dir_name" : prep_run_steps.inputs.parameters["backward_dir_name"],
        "optional_input" : prep_run_steps.inputs.parameters["optional_input"],
    },
    artifacts={
        "task_path" : prep_fp.outputs.artifacts['task_paths'],
        "optional_artifact" : prep_run_steps.inputs.artifacts["optional_artifact"],
    },
    with_sequence=argo_sequence(argo_len(prep_fp.outputs.parameters["task_names"]), format='%06d'),
    key = step_keys['run-fp'],
    executor = run_executor,
    **run_step_config,
)
prep_run_steps.add(run_fp)

prep_run_steps.outputs.artifacts["backward_dirs"]._from = run_fp.outputs.artifacts["backward_dir"]
```

