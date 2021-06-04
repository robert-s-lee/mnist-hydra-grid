[MNIST](http://yann.lecun.com/exdb/mnist/) using 
[Hydra](https://hydra.cc) and 
[PyTorch Vision](https://pytorch.org/vision/stable/index.html) running on 
[Grid.ai](https://www.grid.ai/).     

We will show simple steps to take ML code from laptop to scale out hyperparameter sweep on public cloud.  Grid.ai allows this without any change to the ML code.  

- [Develop on a laptop](#develop-on-a-laptop)
  - [Environment setup](#environment-setup)
  - [Clone from GitHub](#clone-from-github)
  - [Run on a laptop](#run-on-a-laptop)
- [Test on Grid.ai sessions](#test-on-gridai-sessions)
  - [create Datastore](#create-datastore)
- [Hyperparamter sweeps on Grid.ai sessions](#hyperparamter-sweeps-on-gridai-sessions)
  - [Use Web Interface](#use-web-interface)
  - [single run does not work](#single-run-does-not-work)
  - [multi run does not work](#multi-run-does-not-work)

# Develop on a laptop

## Environment setup

These steps follows documentation from 
[Grid.ai Virtual Environment](https://docs.grid.ai/products/global-cli-configs/virtual-environments), 
[PyTorch Hydra](https://github.com/pytorch/hydra-torch) and 
[Grid requirements.txt](https://docs.grid.ai/products/run-run-and-sweep-github-files/script-dependencies#handling-requirements).
 
``` bash
# from Grid.ai Virtual Environment
conda create --name hydra python=3.7
conda activate hydra
pip install lightning-grid --upgrade
# from PyTorch Hydra
pip install hydra-core
pip install hydra
pip install git+https://github.com/pytorch/hydra-torch
pip install hydra-joblib-launcher --upgrade
# from Grid.ai as Hydra requires this for now
pip freeze > requirements.txt
```

## Clone from GitHub
This code is modified version of [PyTorch Hydra example code](https://github.com/pytorch/hydra-torch/blob/master/examples/mnist_00.md) to add following:
- [Grid.ai Datastores](https://docs.grid.ai/products/add-data-to-grid-datastores) to hold MNIST dataset
 
```
git clone https://github.com/robert-s-lee/mnist-hydra-grid
```
## Run on a laptop
Run two experiments varying `batch_size` 32 and 64
```
python mnist-hydra-01.py --multirun data_dir=$(pwd)/mnist batch_size=32,64 hydra/launcher=joblib

```

# Test on Grid.ai sessions

There is no need to repeatedly download the MNIST dataset.

## create Datastore
```
grid datastores create --name mnist --source mnist
```

# Hyperparamter sweeps on Grid.ai sessions

## Use Web Interface 
```
grid datastores create --name mnist --source mnist
```


## single run does not work
```
python mnist-hydra-01.py data_dir=/datastores/mnist batch_size=32
```

## multi run does not work
```
python mnist-hydra-01.py \
--multirun data_dir=/datastores/mnist batch_size=32,64 hydra/launcher=joblib  \
```
