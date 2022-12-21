# dflow-MP_cgcnn_jupyter
This is a simple introduction to dflow, materials screening and machine learning.
The target of this work flow is to combine machine learning and high-throughput screening for materials discovery. 

![Picture1](https://user-images.githubusercontent.com/47341079/187360757-daa63ae8-9f6f-493b-b0f0-541125c517f6.png)


## Setup
1. Setup conda environment (assume you have conda installed)
```shell
conda create -n dflow-helloworld python=3.9
```
2. Activate conda environment and install jupyter notebook
```shell
conda activate dflow-jupyter
pip install notebook
```

## How to use
1. Clone this repository to local
```shell
git clone https://github.com/git clone https://github.com/
```
2. `cd` into the repo
```shell
cd dflow_jupyter
```
3. Start up jupyter notebook
```shell
jupyter notebook
```
4. Docker images should include python, pymatgen, PyTorch and scikit-learn.


## Prepare your screening code
1. Get your API key from materials project: https://legacy.materialsproject.org/open. Use it as your first argument (sys.argv[1])

2. Write your criteria for screening. In the current version, the number of elements (nelements) are applied to screen as a demo. For your onwer purpose, you can write different screening criteria, such as must include certain elements, energy above hull must be smaller than a certain value, band gap must be smaller than a certain value, et al. 

3. Note: the screening code will output a csv file (id_prop.csv) to the designed folder (sys.argv[2])

## Prepare CGCNN code
1. The CGCNN code has been prepared, which is from: https://github.com/txie-93/cgcnn
