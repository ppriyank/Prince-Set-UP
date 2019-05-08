# Installing Tensorflow on Prince   
Try to follow the sequence of  commands as it is.  Do  not do module load anaconda, doesn't seems to  work for me. 

## Using Tunnel   (Outside NYU)  
```
ssh pp1953@gw.hpc.nyu.edu 
```

## Linking up the prince storage to local
*very helpful, if  you  are not a big  vim fan*
```
sshfs -p 22 pp1953@prince.hpc.nyu.edu:/scratch/pp1953 ~/project
```

## Requesting GPUs :   
```
srun -c4 -t24:00:00 --mem=30000 --gres=gpu:p40:1 --pty /bin/bash
```

## Remove any preloaded modules  
```
module purge
```

## Delete any conda environment   
(if something goes wrong) 
Clean the  environment :  
```
conda deactivtae 
conda clean -i -l -t -p -s   
```
Conda seems to bea bug, doesn't  delete the local files, will eventually  exhaust all the storage space
```
conda remove --name env_name --all
```

## Load  Cuda/Cudd modules  (strictly follow the order)
```
*(tensorflow==1.7.0)*  
module load cudnn/9.0v7.0.5  
module load cuda/9.0.176   
*(tensorflow==1.11.0)*
module load cudnn/9.0v7.3.0.29 
module load cuda/9.0.176
```
## Create environment  
```
conda create --name bert python=3.5  
conda activate bert
```


## Install  libraries (Use pip not conda!!)
Conda has some sort of bug. It installs cuda and cudnn as well, which is already installed on prince and in contradictions to the requirements of tensorflow.
```
pip install h5py nltk pyhocon scipy sklearn
pip install tensorflow-gpu==1.7.0  
```
or
```
pip install tensorflow-gpu==1.11.0
```


