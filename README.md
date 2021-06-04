[MNIST](http://yann.lecun.com/exdb/mnist/) running on [Grid.ai](https://www.grid.ai/) using      [Hydra](https://hydra.cc) and 
[PyTorch Vision](https://pytorch.org/vision/stable/index.html) 

We will show simple steps to take ML code from laptop to scale out hyperparameter sweep on public cloud.  Grid.ai allows this without any change to the ML code.  

- [Develop Locally](#develop-locally)
  - [Environment setup](#environment-setup)
  - [Clone from GitHub](#clone-from-github)
  - [Run an experiment](#run-an-experiment)
- [Test Grid.ai dedicated VM](#test-gridai-dedicated-vm)
  - [Create MNIST Datastore](#create-mnist-datastore)
- [Hyperparamter sweeps on Grid.ai sessions](#hyperparamter-sweeps-on-gridai-sessions)
  - [Use Web Interface](#use-web-interface)
  - [The following worked](#the-following-worked)
  - [multi run does not work](#multi-run-does-not-work)

# Develop Locally

## Environment setup

These steps follows documentation from 
[Grid.ai Virtual Environment](https://docs.grid.ai/products/global-cli-configs/virtual-environments), 
[PyTorch Hydra](https://github.com/pytorch/hydra-torch) and 
[Grid.ai requirements.txt](https://docs.grid.ai/products/run-run-and-sweep-github-files/script-dependencies#handling-requirements).  Installation of `conda` can found [here.](https://docs.conda.io/en/latest/miniconda.html)
 
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
[PyTorch Hydra example code](https://github.com/pytorch/hydra-torch/blob/master/examples/mnist_00.md) was modified to add following:
- [Grid.ai Datastores](https://docs.grid.ai/products/add-data-to-grid-datastores) option that points to MNIST dataset
 
```
git clone https://github.com/robert-s-lee/mnist-hydra-grid
```
## Run an experiment
Run an experiments with `batch_size=32` 
```
python mnist-hydra-01.py data_dir=$(pwd)/mnist batch_size=32
```

# Test Grid.ai dedicated VM

## Create MNIST Datastore
There is no need to repeatedly download the MNIST dataset.
```
grid datastores create --name mnist --source mnist
```

# Hyperparamter sweeps on Grid.ai sessions

## Use Web Interface 
```
grid datastores create --name mnist --source mnist
```


## The following worked

```
data_dir=/datastores/mnist batch_size=128 epochs=2
```

## multi run does not work
seem to not like unmatched "--"
```
--multirun data_dir=/datastores/mnist batch_size=16,32,64,128 epochs=1,2
python mnist-hydra-01.py \
--multirun data_dir=/datastores/mnist batch_size=32,64 hydra/launcher=joblib  \
```

