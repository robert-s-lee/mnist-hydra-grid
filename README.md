[MNIST](http://yann.lecun.com/exdb/mnist/) running on [Grid.ai](https://www.grid.ai/) using      [Hydra](https://hydra.cc) and 
[PyTorch Vision](https://pytorch.org/vision/stable/index.html) 

We will show simple steps to take ML code from laptop to scale out hyperparameter sweep on public cloud.  Grid.ai allows this without any change to the ML code.  

- [Develop Locally](#develop-locally)
  - [Environment setup](#environment-setup)
  - [Clone from GitHub](#clone-from-github)
  - [Run an experiment](#run-an-experiment)
- [Test on Grid.ai](#test-on-gridai)
  - [Create MNIST Datastore](#create-mnist-datastore)
  - [Run on Grid.ai](#run-on-gridai)
  - [Inspect Artifacts and Outputs](#inspect-artifacts-and-outputs)

# Develop Locally

## Environment setup

These steps follows documentation from 
[Grid.ai Virtual Environment](https://docs.grid.ai/products/global-cli-configs/virtual-environments), 
[PyTorch Hydra](https://github.com/pytorch/hydra-torch) and 
[Grid.ai requirements.txt](https://docs.grid.ai/products/run-run-and-sweep-github-files/script-dependencies#handling-requirements).  Installation of `conda` can found [here.](https://docs.conda.io/en/latest/miniconda.html)
 
``` bash
# from Grid.ai Virtual Environment
conda create --yes --name hydra python=3.8
conda activate hydra
pip install lightning-grid --upgrade
# from PyTorch Hydra
pip install hydra-core
pip install torch
pip install torchvision
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
Run an experiments with `batch_size=32`.  The first run will download MNIST dataset to directory named `mnist`.  This downloaded dataset will be reused in subsequent runs. 
```
python mnist-hydra-01.py data_dir=$(pwd)/mnist batch_size=32
```

# Test on Grid.ai

## Create MNIST Datastore
Create [Grid.ai Datastore](https://docs.grid.ai/products/add-data-to-grid-datastores#datastores-scalable-datasets) from [downloaded MNIST dataset](#run-an-experiment).
```
grid datastore create --name mnist-torchvision --source mnist
grid datastore list
```

## Run on Grid.ai

Couple of explanations on the parameters:
- `--use_spot` use interruptible instance that is 60% to 90% more cost effective
- `--instance_type=t2.medium` use 2 CPU which should cost $0.01 for this run
- `--datastore_name=mnist` use the [Grid.ai Datastore created above](#create-mnist-datastore) 
- `--datastore_mount_dir=/datastores/mnist` mount location for the script

The output from the above will show below with the host name `khaki-seagull-591`
```
grid run --use_spot --instance_type=t2.medium --datastore_name=mnist-torchvision --datastore_mount_dir=/datastores/mnist mnist-hydra-01.py data_dir=/datastores/mnist batch_size=32
```

The output will show successful submission.
```
WARNING Neither a CPU or GPU number was specified. 1 CPU will be used as a default. To use N GPUs pass in '--grid_gpus N' flag.

Run submitted!
`grid status` to list all runs
`grid status khaki-seagull-591` to see all experiments for this run

----------------------
Submission summary
----------------------
script:                  mnist-hydra-01.py
instance_type:           t2.medium
use_spot:                True
cloud_provider:          aws
cloud_credentials:       xxxxxxxxxx
grid_name:               rich-skink-314
datastore_name:          mnist
datastore_version:       1
datastore_mount_dir:     /datastores/mnist
```
## Inspect Artifacts and Outputs

Note addition of `exp0` to the name. When parallem VMs are allocated using hyperparamter sweep, each VM will have exp0 to expN.  the first experiment.

```
grid logs khaki-seagull-591-exp0
```

