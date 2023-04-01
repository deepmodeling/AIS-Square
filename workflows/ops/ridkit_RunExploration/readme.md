**Name** : rid.op.run_exploration

**Input :**

- task_path: (Artifact(Path)) A directory path containing files for Gromacs MD simulations.
- gmx_config: (Dict) Configuration of Gromacs simulations in exploration steps.
- models: (Artifact(List[Path])) Optional. Neural network model files (.pb) used to bias the simulation.

**Output :**

- **- `plm_out`** ((Artifact(Path)) Outputs of CV values (plumed.out by default) from exploration steps.)
- **- `md_log`** ((Artifact(Path)) Log files of Gromacs mdrun commands.)
- **- `trajectory`** ((Artifact(Path)) Trajectory files (.xtc). The output frequency is defined in gmx_config.)
- **- `conf_out`** ((Artifact(Path)) Final frames of conformations in simulations.)

**Description :**

Run (biased or brute-force) MD simulations with files provided by PrepExplore OP. RiD-kit emploies Gromacs as MD engine with PLUMED2 plugin (with or without MPI-implement). Make sure work environment has properly installed Gromacs with patched PLUMED2. Also, PLUMED2 need an additional DeePFE.cpp patch to use bias potential of RiD. See install in the home page for details.

**Source :**

```python
import os, sys
import logging
from typing import List, Dict, Union
from pathlib import Path
from dflow.python import (
    OP,
    OPIO,
    OPIOSign,
    Artifact
)
from rid.constants import (
        gmx_conf_name,
        gmx_top_name,
        gmx_mdp_name, 
        gmx_tpr_name,
        plumed_input_name,
        plumed_output_name,
        gmx_mdrun_log,
        xtc_name,
        gmx_conf_out
    )
from rid.utils import run_command, set_directory, list_to_string
from rid.common.gromacs.command import get_grompp_cmd, get_mdrun_cmd


logging.basicConfig(
    format="%(asctime)s | %(levelname)s | %(name)s | %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
    level=os.environ.get("LOGLEVEL", "INFO").upper(),
    stream=sys.stdout,
)
logger = logging.getLogger(__name__)


class RunExplore(OP):

    """Run (biased or brute-force) MD simulations with files provided by `PrepExplore` OP.
    RiD-kit emploies Gromacs as MD engine with PLUMED2 plugin (with or without MPI-implement). Make sure work environment 
    has properly installed Gromacs with patched PLUMED2. Also, PLUMED2 need an additional `DeePFE.cpp` patch to use bias potential of RiD.
    See `install` in the home page for details.
    """

    @classmethod
    def get_input_sign(cls):
        return OPIOSign(
            {
                "task_path": Artifact(Path),
                "forcefield": Artifact(Path, optional=True),
                "gmx_config": Dict,
                "models": Artifact(List[Path], optional=True),
                "index_file": Artifact(Path, optional=True),
                "dp_files": Artifact(List[Path], optional=True),
                "cv_file": Artifact(List[Path], optional=True)
            }
        )


    @classmethod
    def get_output_sign(cls):
        return OPIOSign(
            {
                "plm_out": Artifact(Path),
                "md_log": Artifact(Path),
                "trajectory": Artifact(Path),
                "conf_out": Artifact(Path)
            }
        )


    @OP.exec_sign_check
    def execute(
        self,
        op_in: OPIO,
    ) -> OPIO:

        r"""Execute the OP.
        
        Parameters
        ----------
        op_in : dict
            Input dict with components:

            - `task_path`: (`Artifact(Path)`) A directory path containing files for Gromacs MD simulations.
            - `gmx_config`: (`Dict`) Configuration of Gromacs simulations in exploration steps.
            - `models`: (`Artifact(List[Path])`) Optional. Neural network model files (`.pb`) used to bias the simulation. 
                Run brute force MD simulations if not provided.
          
        Returns
        -------
            Output dict with components:
        
            - `plm_out`: (`Artifact(Path)`) Outputs of CV values (`plumed.out` by default) from exploration steps.
            - `md_log`: (`Artifact(Path)`) Log files of Gromacs `mdrun` commands.
            - `trajectory`: (`Artifact(Path)`) Trajectory files (`.xtc`). The output frequency is defined in `gmx_config`.
            - `conf_out`: (`Artifact(Path)`) Final frames of conformations in simulations.
        """
        if op_in["index_file"] is None:
            index = None
        else:
            index = op_in["index_file"].name
        
        gmx_grompp_cmd = get_grompp_cmd(
            mdp = gmx_mdp_name,
            conf = gmx_conf_name,
            topology = gmx_top_name,
            output = gmx_tpr_name,
            index = index,
            max_warning=op_in["gmx_config"]["max_warning"]
        )
        gmx_run_cmd = get_mdrun_cmd(
            tpr=gmx_tpr_name,
            plumed=plumed_input_name,
            nt=op_in["gmx_config"]["nt"],
            ntmpi=op_in["gmx_config"]["ntmpi"]
        )

        with set_directory(op_in["task_path"]):
            if op_in["forcefield"] is not None:
                os.symlink(op_in["forcefield"], op_in["forcefield"].name)
            if op_in["models"] is not None:
                for model in op_in["models"]:
                    os.symlink(model, model.name)
            if op_in["dp_files"] is not None:
                for file in op_in["dp_files"]:
                    os.symlink(file, file.name)
            if op_in["index_file"] is not None:
                os.symlink(op_in["index_file"], op_in["index_file"].name)
            if op_in["cv_file"] is not None:
                for file in op_in["cv_file"]:
                    if file.name != "colvar":
                        os.symlink(file, file.name)
            
            os.environ["GMX_DEEPMD_INPUT_JSON"] = "./input.json"
            logger.info(list_to_string(gmx_grompp_cmd, " "))
            return_code, out, err = run_command(gmx_grompp_cmd)
            assert return_code == 0, err
            logger.info(err)
            
            logger.info(list_to_string(gmx_run_cmd, " "))
            return_code, out, err = run_command(gmx_run_cmd)
            assert return_code == 0, err
            logger.info(err)
        
        op_out = OPIO(
            {
                "plm_out": op_in["task_path"].joinpath(plumed_output_name),
                "md_log": op_in["task_path"].joinpath(gmx_mdrun_log),
                "trajectory": op_in["task_path"].joinpath(xtc_name),
                "conf_out": op_in["task_path"].joinpath(gmx_conf_out),
            }
        )
        return op_out

```
