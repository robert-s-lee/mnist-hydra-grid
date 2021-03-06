[![Grid Hydra](https://github.com/robert-s-lee/mnist-hydra-grid/actions/workflows/unittest.yml/badge.svg)](https://github.com/robert-s-lee/mnist-hydra-grid/actions/workflows/unittest.yml)

[MNIST](http://yann.lecun.com/exdb/mnist/) running on [Grid.ai](https://www.grid.ai/) using [Hydra](https://hydra.cc) and 
[PyTorch Vision](https://pytorch.org/vision/stable/index.html) 

We will show simple steps to take ML code from laptop to scale out hyperparameter sweep on public cloud.  Grid.ai allows this without any change to the ML code.  

- [Develop Locally](#develop-locally)
- [Testing on Grid.ai](#testing-on-gridai)
- [Automate using GitHub Actions](#automate-using-github-actions)

# Develop Locally

- Setup the local environment
  
These steps follows documentation from 
[Grid.ai Virtual Environment](https://docs.grid.ai/products/global-cli-configs/virtual-environments), 
[PyTorch Hydra](https://github.com/pytorch/hydra-torch) and 
[Grid.ai requirements.txt](https://docs.grid.ai/products/run-run-and-sweep-github-files/script-dependencies#handling-requirements).  Installation of `conda` can found [here.](https://docs.conda.io/en/latest/miniconda.html)
 
``` bash
# Grid.ai Virtual Environment
conda create --yes --name hydra python=3.8
conda activate hydra
pip install lightning-grid --upgrade
# from PyTorch Hydra
git clone https://github.com/robert-s-lee/mnist-hydra-grid
cd mnist-hydra-grid
pip install -r requirements.txt
```

- run the experiment 

```bash
python mnist-hydra-01.py 
```

# Testing on Grid.ai

Simply change `python` to `grid run`.  Couple of variations below.  

```bash
# run the model
grid run --localdir --dependency_file requirements.txt mnist-hydra-01.py

# save the checkpoint file
grid run --localdir --dependency_file requirements.txt mnist-hydra-01.py save_model=True                            

# save the checkpoint file, override mnistconf.yaml epochs setting
grid run --localdir --dependency_file requirements.txt mnist-hydra-01.py save_model=True epochs=3

# alternate config file
grid run --localdir --dependency_file requirements.txt mnist-hydra-01.py save_model=True epochs=3 --config-name mnistconf.yaml 
```

- Parallel run on a single server
```
grid run --localdir --dependency_file requirements.txt mnist-hydra-01.py epochs=3,10,15 --multirun

grid run --localdir --dependency_file requirements.txt mnist-hydra-01.py epochs=3,10,15 batch_size=32,64 --multirun

[experiment] [2021-11-05T01:21:12.092751+00:00] [2021-11-05 01:21:12,092][HYDRA] Joblib.Parallel(n_jobs=-1,backend=loky,prefer=processes,require=None,verbose=0,timeout=None,pre_dispatch=2*n_jobs,batch_size=auto,temp_folder=None,max_nbytes=None,mmap_mode=r) is launching 3 jobs
[experiment] [2021-11-05T01:21:12.092792+00:00] [2021-11-05 01:21:12,092][HYDRA] Launching jobs, sweep output dir : multirun/2021-11-05/01-21-11
[experiment] [2021-11-05T01:21:12.092799+00:00] [2021-11-05 01:21:12,092][HYDRA]        #0 : epochs=3
[experiment] [2021-11-05T01:21:12.092804+00:00] [2021-11-05 01:21:12,092][HYDRA]        #1 : epochs=10
[experiment] [2021-11-05T01:21:12.092808+00:00] [2021-11-05 01:21:12,092][HYDRA]        #2 : epochs=15

```

# Automate using GitHub Actions

[unittest.yml](.github/workflows/unittest.yml) runs equivalent of the following commands:

- `grid run --localdir --dependency_file requirements.txt mnist-hydra-01.py`  
- `grid run --localdir --dependency_file requirements.txt mnist-hydra-01.py save_model=True epochs=3`.  

NOTE: Remember to add `GRIDAI_USERNAME` and `GRIDAI_KEY` repo secrets if forking this repo.
