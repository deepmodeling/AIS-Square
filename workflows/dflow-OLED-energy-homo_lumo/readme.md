# dflow: calculate s1, t1 state energy and molecule properties (homo and lumo) of OLED molecules with Gaussian 16

## Introduction
This dflow proposal is for molecule energy and properties (homo and lumo) calculation using Gaussian 16. from preparing a gaussain 16 input of OLED molecule, structure optimization will be performed at s0 state. The structure optimizaton will be performed at s1 and t1 state using the optimized structure at s0 state as initial structures. Finally, the molecule energy, homo and lumo of optimized structure at s1 and t1 state will be collected.   

## Details


### Workflow based on dflow
However, with the aid of dflow, we only need to prepare the gaussian 16 input file which contains the structure informations, and then the remain steps are automatically. The dflow workflow diagram is shown in Figure 1.

![alt 文字](./figs/dflow_diagram.jpg)
<center> Figure 1. The dflow diagram for molecule energy and properties calculations based on Gaussian16</center>

### How to run the program
1. Prepare a gaussian input file named "s0.com" of OLED molecules which contains the structure informations. The format of "s0.com" is also given in this object. 
2. Add your program id and authorization from Bohrium platform to the lbg_flow.py.  

```python
if __name__ == "__main__":
    lebesgue_context = LebesgueContext(
        executor="lebesgue_v2",
        extra='{"scass_type":"c16_m64_cpu","program_id": "your program id"}',
        authorization='your bohrium authorization',
        app_name='Default',
        org_id='123',
        user_id='456',
        tag='',
    )
```
 
 3. Run "python lbg_flow.py", and then structre optimizaton at s0, s1 and t1 state will be performed, molecule energy, homo and lumo information is collected at s1data.npy and t1data.npy 
