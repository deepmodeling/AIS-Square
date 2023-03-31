**Name** : deepks2.op.iter_op.convert_scf_op.ConvertScfAbacus

**Input :** 

\- `no_model` : (`bool`) Tell this OP if there is an input model of DeePKS format.

\- `yaml_name` : (`str`) The name of the input yaml file.

​\- `config_file` : (`Artifact(Path)`) The folder contains the yaml file specified in "yaml_name". See [scf_abacus.yaml](https://deepks-kit-qi.readthedocs.io/en/latest/inputs-preperation.html#scf-abacus-yaml) but no need to group the args in blocks like "scf_abacus" in that website.

​\- `system` : (`Artifact(Path)`) The folder contains "atom.npy" and "box.npy" of DeePKS format. They should be in subfolders "systems_train" and "systems_test", like "system/systems_train/group.00/\*.npy" and "system/systems_test/group.01/\*.npy".

\- `model` : (`Artifact(Path, optional=True)`) Path to the model file of DeePKS format.

**Output :**

\- `tasks` : (`Artifact(Path)`) The folder contains ABACUS input files for each structure.

\- `model` : (`Artifact(Path)`) If there is an input model, there will be a output model which has been converted to ABACUS format.

**Description :**

​This OP is extracted from **deepks-flow** workflow.

​This OP converts DeePKS input files to ABACUS input files.

**Source :** 

```python
# Inputs
no_model: bool
yaml_name: str
config_file: Artifact(type=pathlib.Path, optional=False, sub_path=True)
system: Artifact(type=pathlib.Path, optional=False, sub_path=True)
model: Artifact(type=pathlib.Path, optional=True, sub_path=True)
# Outputs
tasks: Artifact(type=pathlib.Path, optional=False, sub_path=True)
model: Artifact(type=pathlib.Path, optional=False, sub_path=True)
# Execute
    @OP.exec_sign_check
    def execute(
            self,
            ip: OPIO,
    ) -> OPIO:
        r"""Execute the OP.

        Parameters
        ----------
        ip : dict
            Input dict with components:

            - no_model: (bool)
            - yaml_name: (str)
            - config_file: (Artifact(Path))
            - system: (Artifact(Path))
            - model: (Artifact(Path, optional=True))
        
        Returns
        -------
        op : dict 
            Output dict with components:

            - tasks: (Artifact(Path))
            - model: (Artifact(Path))
            
        """

        # convert_scf_config = ip["scf_config"]
        no_model = ip["no_model"]
        yaml_name = ip["yaml_name"]
        config_file = ip["config_file"]
        system = ip["system"]
        model_file = ip["model"]

        cwd = os.getcwd()
        os.chdir(system)
        
        convert_scf_config = load_yaml(config_file/yaml_name)

        sys_train = system / SYS_TRAIN
        sys_test = system / SYS_TEST
        systems_train = []
        systems_test = []
        for root, dirs, files in os.walk(sys_train):
            for dir in dirs:
                systems_train.append(os.path.join(root, dir))
        for root, dirs, files in os.walk(sys_test):
            for dir in dirs:
                systems_test.append(os.path.join(root, dir))

        orb_files = convert_scf_config["orb_files"]
        pp_files = convert_scf_config["pp_files"]
        proj_file = convert_scf_config["proj_file"]
        orb_files = ["../../"+str(os.path.basename(s)) for s in orb_files]
        pp_files = ["../../"+str(os.path.basename(s)) for s in pp_files]
        proj_file = ["../../"+str(os.path.basename(s)) for s in proj_file]
        

        # from deepks.iterate.template_abacus import convert_data

        convert_scf_config.update(
            systems_train = systems_train,
            systems_test = systems_test,
            model_file = model_file,
            no_model = no_model,
            orb_files = orb_files,
            pp_files = pp_files,
            proj_file = proj_file,
        )

        ConvertScfAbacus.convert_data(**convert_scf_config)

        os.chdir(cwd)

        return OPIO({
            "tasks": system,
            "model": system/CMODEL_FILE,
        })

    @staticmethod
    # need parameters: orb_files, pp_files, proj_file
    def convert_data(systems_train, systems_test=None, *,
                     no_model=True, model_file=None, pp_files=[],
                     lattice_vector=np.eye(3, dtype=int), dispatcher=None, **pre_args):

        from deepks2.constants import CMODEL_FILE
        from deepks.utils import load_sys_paths
        from deepks.iterate.generator_abacus import make_abacus_scf_kpt, make_abacus_scf_input, make_abacus_scf_stru
        from deepks2.constants import NAME_TYPE, TYPE_NAME

        # trace a model (if necessary)
        if not no_model:
            if model_file is not None:
                from deepks.model import CorrNet
                model = CorrNet.load(model_file)
                model.compile_save(CMODEL_FILE)
                # set 'deepks_scf' to 1, and give abacus the path of traced model file
                pre_args.update(
                    deepks_scf=1, model_file="../../"+CMODEL_FILE)
            else:
                raise FileNotFoundError(
                    f"No required model file in {os.getcwd()}")
        # split systems into groups
        nsys_trn = len(systems_train)
        nsys_tst = len(systems_test)
        #ntask_trn = int(np.ceil(nsys_trn / sub_size))
        #ntask_tst = int(np.ceil(nsys_tst / sub_size))
        train_sets = [systems_train[i::nsys_trn] for i in range(nsys_trn)]
        test_sets = [systems_test[i::nsys_tst] for i in range(nsys_tst)]
        systems = systems_train+systems_test
        sys_paths = [os.path.abspath(s) for s in load_sys_paths(systems)]
        # init sys_data (dpdata)
        for i, sset in enumerate(train_sets+test_sets):
            atom_data = np.load(f"{sys_paths[i]}/atom.npy")
            if os.path.isfile(f"{sys_paths[i]}/box.npy"):
                cell_data = np.load(f"{sys_paths[i]}/box.npy")
            nframes = atom_data.shape[0]
            natoms = atom_data.shape[1]
            atoms = atom_data[1,:,0]
            #atoms.sort() # type order
            types = np.unique(atoms) #index in type list
            ntype = types.size
            from collections import Counter
            nta = Counter(atoms) #dict {itype: nta}, natom in each type
            if not os.path.exists(f"{sys_paths[i]}/ABACUS"):
                os.mkdir(f"{sys_paths[i]}/ABACUS")
            #pre_args.update({"lattice_vector":lattice_vector})
            #if "stru_abacus.yaml" exists, update STRU args in pre_args:
            pre_args_new=dict(zip(pre_args.keys(),pre_args.values()))
            if os.path.exists(f"{sys_paths[i]}/stru_abacus.yaml"):
                from deepks.utils import load_yaml
                stru_abacus = load_yaml(f"{sys_paths[i]}/stru_abacus.yaml")
                for k,v in stru_abacus.items():
                    pre_args_new[k]=v
            for f in range(nframes):
                if not os.path.exists(f"{sys_paths[i]}/ABACUS/{f}"):
                    os.mkdir(f"{sys_paths[i]}/ABACUS/{f}")
                ###create STRU file
                if not os.path.isfile(f"{sys_paths[i]}/ABACUS/{f}/STRU"):
                    Path(f"{sys_paths[i]}/ABACUS/{f}/STRU").touch()
                #create sys_data for each frame
                frame_data=atom_data[f]
                #frame_sorted=frame_data[np.lexsort(frame_data[:,::-1].T)] #sort cord by type
                sys_data={'atom_names':[TYPE_NAME[it] for it in nta.keys()], 'atom_numbs': list(nta.values()), 
                            #'cells': np.array([lattice_vector]), 'coords': [frame_sorted[:,1:]]}
                            'cells': np.array([lattice_vector]), 'coords': [frame_data[:,1:]]}
                if os.path.isfile(f"{sys_paths[i]}/box.npy"):
                    sys_data={'atom_names':[TYPE_NAME[it] for it in nta.keys()], 'atom_numbs': list(nta.values()),
                            'cells': [cell_data[f]], 'coords': [frame_data[:,1:]]}
                #write STRU file
                with open(f"{sys_paths[i]}/ABACUS/{f}/STRU", "w") as stru_file:
                    stru_file.write(make_abacus_scf_stru(sys_data, pp_files, pre_args_new))
                #write INPUT file
                with open(f"{sys_paths[i]}/ABACUS/{f}/INPUT", "w") as input_file:
                    input_file.write(make_abacus_scf_input(pre_args))


                #write KPT file if k_points is explicitly specified or for gamma_only case
                if pre_args["k_points"] is not None or pre_args["gamma_only"] is True:
                    with open(f"{sys_paths[i]}/ABACUS/{f}/KPT","w") as kpt_file:
                        kpt_file.write(make_abacus_scf_kpt(pre_args))
```