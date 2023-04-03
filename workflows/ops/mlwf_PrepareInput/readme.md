**Name**: mlwf_op.prepare_input_op.Prepare

**Input**:  
-`input_setting`: (`dict`), a template to generate the inputs of calculations.

-`task_setting`: (`dict`), setting some parameters of tasks, such as 'group_size'.

-`confs`: (`Path`), the configurations that can be read by dpdata.System.

-`pseudo`: (`Path`), pseudopotentials.

**Output**:  
-`task_path`: (`List[Path]`), the prepared working paths of the tasks.

-`frames_list`: (`List[Tuple[int]]`), the interval of the frame id. For example, (0, 5) means this task works on `confs[0:5]`. The order should be consistent with `op_out["task_path"]`.

**Discription**:  
This OP generates the inputs from the template and puts them into subpaths. The number of confs per task is `group_size`.

**Source**:
```python
@classmethod
def get_input_sign(cls):
    return OPIOSign({
        "input_setting": BigParameter(dict),
        "task_setting": BigParameter(dict),
        "confs": Artifact(Path),
        "pseudo": Artifact(Path)
    })

@classmethod
def get_output_sign(cls):
    return OPIOSign({
        "input_setting": BigParameter(dict),
        "task_path": Artifact(List[Path], archive=None),
        "frames_list": BigParameter(List[Tuple[int]])
    })

@OP.exec_sign_check
def execute(
        self,
        op_in: OPIO,
) -> OPIO:
    input_setting: Dict[str, Union[str, dict]] = op_in["input_setting"]
    group_size: int = op_in["task_setting"]["group_size"]
    confs = dpdata.System(op_in["confs"], fmt='deepmd/raw', type_map = ['O', 'H'])
    pseudo: Path = op_in["pseudo"]

    input_setting = self.init_inputs(input_setting, confs)
    task_path, frames_list = self._exec_all(confs, pseudo, group_size)
    return OPIO({
        "input_setting": input_setting,
        "task_path": task_path,
        "frames_list": frames_list
    })

def _exec_all(self, confs: dpdata.System, pseudo: Path, group_size: int):
    nframes = confs.get_nframes()
    task_path = []
    frames_list = []
    for i in range(ceil(nframes / group_size)):
        task_name = f"task.{i:06d}"
        print(task_name)
        task = Path(task_name)
        task_path.append(task)
        start_f = i * group_size
        end_f = min(start_f + group_size, nframes)
        frames_list.append((start_f, end_f))
        with set_directory(task_name, mkdir = True):
            shutil.copytree(pseudo, "./pseudo")
            for frame in range(start_f, end_f):
                subtask_name = f"conf.{frame:06d}"
                print(subtask_name)
                with set_directory(subtask_name, mkdir = True):
                    self.write_one_frame(frame)
    return task_path, frames_list

@abc.abstractmethod
def init_inputs(self, input_setting: Dict[str, Union[str, dict]], confs: dpdata.System) -> Dict[str, Union[str, dict]]:
    pass

@abc.abstractmethod
def write_one_frame(self, frame: int):
    pass
```

**Implementation**:
There are 2 implementations: `mlwf_op.qe_wannier90.PrepareQeWann` and `mlwf_op.qe_cp.PrepareCP`.

`mlwf_op.qe_wannier90.PrepareQeWann`: prepare the inputs for Qe-PW and Wannier90;

`mlwf_op.qe_cp.PrepareCP`: prepare the inputs for Qe-CP.