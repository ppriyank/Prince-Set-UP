# Installing Tensorflow on Prince   
Try to follow the sequence of  commands as it is.  Do  not do module load anaconda, doesn't seems to  work for me. 

## Using Tunnel   (Outside NYU)  
ssh into gw.hpc.nyu.edu first, or use cisco vpn (allows file  transfer)
```
ssh pp1953@gw.hpc.nyu.edu 
```

## Linking up the prince storage to local
*very helpful, if  you  are not a big  vim fan*
```
sshfs -p 22 pp1953@prince.hpc.nyu.edu:/scratch/pp1953 ~/project
```

## What virtual environment you have
```
conda info --envs
```

## Delete a conda environment   
(if something goes wrong, start fresh) 
Clean the  environment :  
```
conda deactivate 
conda clean -i -l -t -p -s   
```
Conda seems to have a bug, doesn't  delete the local files, will eventually  exhaust all the storage space
```
conda remove --name env_name --all
```

## Create environment  
```
conda create --name bert python=3.5  
conda activate bert
```


## Install  libraries (Use pip not conda!!)
Conda has some sorts of bug. It installs cuda and cudnn as well, which is already installed on prince and in contradictions to the requirements of tensorflow.
```
pip install h5py nltk pyhocon scipy sklearn
pip install tensorflow-gpu==1.7.0  
```
or
```
pip install tensorflow-gpu==1.11.0
```

## Installing Pytorch 
```
conda  env create -f req.yml 
```

req.yml : 
```
 name: env_name
 channels:
     - pytorch
 dependencies:
     - python=3.6
     - pytorch=1.0.0
     - torchvision=0.2.1
     - numpy=1.14.5
     - scikit-learn=0.19.1
     - h5py
     - scipy
     - pip:
         - tensorboard
```

## Submitting Jobs (Recommended)
Create a file `file_name.s` like  
```
#!/bin/bash
#SBATCH --cpus-per-task=4
#SBATCH --nodes=1
#SBATCH --gres=gpu:p40:1
#SBATCH --time=00:5:00
#SBATCH --mem=100000
#SBATCH --job-name=pp1953
#SBATCH --mail-user=pp1953p@nyu.edu
#SBATCH --output=slurm_%j.out


. ~/.bashrc
module load anaconda3/5.3.1

conda activate PPUU
conda install -n PPUU nb_conda_kernels
# conda activate 

cd 
cd code/Video-Person-ReID-master/current
python best_cl_centers.py --opt=3 >> gpu-sgd.out
```
Run the above script as 

```sbatch file_name.s```

`squeue -u pp1953`  
`squeue -j 4654238`  
`scancel 4654238`  

## Requesting GPUs (not recommended):   
Types of GPUs available can be found here : https://wikis.nyu.edu/display/NYUHPC/Clusters+-+Prince
```
srun -c4 -t5:00:00 --mem=30000 --gres=gpu:p40:1 --pty /bin/bash
```

## Remove any preloaded modules  
```
module purge
```

## Load  Cuda/Cudd modules  (strictly follow the order)

*(tensorflow==1.7.0)*  
```
module load cudnn/9.0v7.0.5  
module load cuda/9.0.176   
```
*(tensorflow==1.11.0)*
```
module load cudnn/9.0v7.3.0.29 
module load cuda/9.0.176
```


## Creating a jupyter notebook on server
```
pip install jupyter
pip install jupyter[notebook]
pip install matplotlib
pip install jupyter_contrib_nbextensions
jupyter contrib nbextension install --user

jupyter notebook --generate-config
```

edit the following to (uncomment as well remove \# ): 
```
c.NotebookApp.ip = '0.0.0.0'
c.NotebookApp.allow_origin = '*'
```

## Using tmux (use mouse scrolling)
For god's sake use tmux
```
tmux new -s session_name
tmux a -t session_name
control + b -> # (sliding between windows) or control + b -> ' -> # (window >10) 
```
Set mouse scrolling on by : (Mac control + b -> shift + : ->`setw -g mouse on ` or `setw -g mode-mouse on` )

## Working with Jiant (allenNLP)

```
conda install -c conda-forge jsonnet 
module spider gcc 
module load gcc/9.1.0
conda install -c conda-forge regex
pip install allennlp
```

`pip install` <--> `conda install -c conda-forge` 
### Don't load python module!!!

## Common problems

* Conda install reverts the changes of loading cuda/cudnn  
use : ```conda uninstall tensorflow-gpu cudatoolkit cudnn ```
* Tensorflow was compiled with diffent version of  cudnn  and currently is a different version is loaded. Just load the correct/earlier version of cudnn by which *tensorflow-gpu* was  installed
* Tensorflow is not compatible to use gpu. cuda/cudnn used during installation doesn't match with the tensorflow binary from which it was created.  ```pip uninstall tensorflow-gpu``` or  possibly delete the  whole  environment and follow the  above procedure.




## Summary
```
sshfs -p 22 pp1953@prince.hpc.nyu.edu:/scratch/pp1953 ~/NYU/project
ssh pp1953@prince.hpc.nyu.edu
tmux a -t pathak
srun -c4 -t100:00:00 --mem=50000 --gres=gpu:p40:1 --pty /bin/bash
cd /scratch/pp1953/model2/codes/
module load cudnn/9.0v7.3.0.29 
module load cuda/9.0.176
conda activate bert
```
