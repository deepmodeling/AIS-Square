## Introduction
A DP-PBE model for Cu. **[1]**


## Training
The smooth version of the DP model is adopted. The cut-off radius is set to 8 Å, and the inverse distance 1/r decays smoothly from 2 Å to 8 Å in order to remove the discontinuity introduced by the cut-off. The embedding network of size (25, 50, 100) follows a ResNet-like architecture **[2]**. The fitting network is composed of three layers, with each containing 240 nodes. The Adam stochastic gradient descent method **[3]** is utilized to train four models, with the only difference being their random seeds. Each model is trained with 400,000 gradient descent steps with an exponentially decaying learning rate from $1.0×10^{−3}$ to $3.5×10^{−8}$. These settings for the training process are designated by the key default_training_params provided by the input file PARAM, which adopts the same interface with DeePMD-kit.

More details can be found in **[1]**.


## Use model
1. Download and install LAMMPS: https://docs.lammps.org/Manual.html
2. Download and install DeePMD-kit: https://github.com/deepmodeling/deepmd-kit
3. Prepare input data for LAMMPS
4. Download the model
5. Run MD with LAMMPS: https://github.com/deepmodeling/deepmd-kit/blob/master/doc/third-party/lammps.md


## References
**[1]** Zhang, Yuzhi, et al. "DP-GEN: A concurrent learning platform for the generation of reliable deep learning based potential energy models." Computer Physics Communications 253 (2020): 107206.

**[2]** He, Kaiming, et al. "Deep residual learning for image recognition." Proceedings of the IEEE conference on computer vision and pattern recognition. 2016.

**[3]** Kingma, Diederik P. "&Ba J.(2014). Adam: A method for stochastic optimization." arXiv preprint arXiv:1412.6980 (2015).


