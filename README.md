### Table of Contents
* **[Installing Tensorflow on Prince](#Installing-Tensorflow-on-Prince)**<br>
  * [Installing Conda](#Installing-Conda)<br>
  * [Delete a conda environment](#Delete-a-conda-environment)<br>
  * [Create environment](#Create-environment)<br>  
  * [Install  libraries (Tensorflow)](#Install--libraries)<br>  
  * [Installing Pytorch](#Installing-Pytorch)<br>  
  * [Submitting Jobs (Recommended)](#Submitting-Jobs-Recommended-)<br>  
  * [Remove any preloaded modules](#Remove-any-preloaded-modules)<br>  
  * [Requesting GPUs (not recommended):](#Requesting-GPUs-not-recommended:)<br>  
  * [Load  Cuda/Cudnn modules  (strictly follow the order)](#Load--Cuda/Cudnn-modules--strictly-follow-the-order)<br>  
  * [Creating a jupyter notebook on Prince](#Creating-a-jupyter-notebook-on-Prince)<br>    
  * [Creating a jupyter notebook on server (unverified)](#Creating-a-jupyter-notebook-on-server-unverified)<br>  
  * [Working with Jiant (allenNLP)](#Working-with-Jiant-allenNLP)<br>    
  * [Common Problems](#Common-problems)<br>  
  
* **[hacks](#hacks)**<br>
  * [Downloading Googe drive link](#Downloading-Googe-drive-link)<br>  
  * [Setting Keys](#Setting-up-keys)<br> 
  * [SSh Using Tunnel](#Using-Tunnel-Outside-NYU-)<br> 
  * [Mount (w) Tunnel](#Mount-Point-Using-Access-Point)<br>  
  * [Mount  (w/o) Tunnel](#Linking-up-the-prince-storage-to-local)<br>  
  * [Using tmux](#Using-tmux-(use-mouse-scrolling))<br>    
  * [Pdb multiple Line code](#Pdb-multiple-Line-code)<br>  


* **[Summary](#Summary)**<br>


# Installing Tensorflow on Prince   
Try to follow the sequence of  commands as it is.  Do  not do module load anaconda, doesn't seems to  work for me. 

### Installing Conda
Download `python 3.6` version of the Anaconda for `ubuntu` from here : https://www.anaconda.com/distribution/

```
wget <download link>
unzip <downloaded file> / tar -xf <downloaded file>
chmod +777 <downloaded file>
./<downloaded file>.sh 
```


### What virtual environment you have
```
conda info --envs
```

### Delete a conda environment   
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

### Create environment  
```
conda create --name bert python=3.5  
conda activate bert
```


### Install  libraries
**(Use pip not conda!!)**
Conda has some sorts of bug. It installs cuda and cudnn as well, which is already installed on prince and in contradictions to the requirements of tensorflow.
```
pip install h5py nltk pyhocon scipy sklearn
pip install tensorflow-gpu==1.7.0  
```
or
```
pip install tensorflow-gpu==1.11.0
```

### Installing Pytorch 
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

### Submitting Jobs (Recommended)
Create a file `file_name.s` like  
time : `hts:mins:seconds`  
gpu: type: always use p40   
mem: 50000 == 5Gb (the max one can request is 10Gb, don't request this much, you don't need it :) )  
nodes>1 not allowed (IDK)  

```
#!/bin/bash
#SBATCH --cpus-per-task=4
#SBATCH --nodes=1
#SBATCH --gres=gpu:p40:1
#SBATCH --time=01:00:00
#SBATCH --mem=50000
#SBATCH --job-name=pp1953
#SBATCH --mail-user=pp1953p@nyu.edu
#SBATCH --output=slurm_%j.out

OPT=$1 
#command line argument
. ~/.bashrc
module load anaconda3/5.3.1

conda activate <conda env name>
conda install -n <conda env name> nb_conda_kernels
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
Check job history :  
`sacct --format=User,JobID,partition,state,time,start,end,elapsed,nodelist -j 4821655`  

### Requesting GPUs (not recommended):   
Types of GPUs available can be found here : https://wikis.nyu.edu/display/NYUHPC/Clusters+-+Prince
```
srun -c4 -t5:00:00 --mem=30000 --gres=gpu:p40:1 --pty /bin/bash
```

### Remove any preloaded modules  
```
module purge
```

### Load  Cuda/Cudnn modules  (strictly follow the order)

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

### Creating a jupyter notebook on Prince 
```
conda install -n <conda env name> nb_conda_kernels
python -m ipykernel install --user --name build_central --display-name <conda env name>
```

Now create a sbatch file like this
```
#!/bin/bash
#SBATCH --cpus-per-task=8
#SBATCH --nodes=1
#SBATCH --time=100:00:00
#SBATCH --gres=gpu:v100:1
#SBATCH --mem=100000
#SBATCH --job-name=pp1953
#SBATCH --mail-user=pp1953p@nyu.edu
#SBATCH --output=outputs/slurm_%j.out


. ~/.bashrc
module load anaconda3/5.3.1
module load jupyter-kernels/py3.5

conda activate PPUU
conda install -n PPUU nb_conda_kernels

port=$(shuf -i 10000-65500 -n 1)
/usr/bin/ssh -N -f -R $port:localhost:$port log-0
/usr/bin/ssh -N -f -R $port:localhost:$port log-1

cat<<EOF

Jupyter server is running on: $(hostname)
Job starts at: $(date)

Step 1 :
ssh -L $port:localhost:$port $USER@prince.hpc.nyu.edu

ssh -L $port:localhost:$port $USER@prince

Step 2:


the URL is something: http://localhost:${port}/?token=XXXXXXXX (see your token below)

EOF

unset XDG_RUNTIME_DIR
if [ "$SLURM_JOBTMP" != "" ]; then
    export XDG_RUNTIME_DIR=$SLURM_JOBTMP
fi

jupyter notebook --no-browser --port $port --notebook-dir=$(pwd)
```

if there is, by any chance, `.ssh/config` file rights distorted : do `chmod 700 .ssh/config` 

### Creating a jupyter notebook on server (unverified)
```
pip install jupyter
pip install jupyter[notebook]
pip install matplotlib
pip install jupyter_contrib_nbextensions
jupyter contrib nbextension install --user

jupyter notebook --generate-config
```
For normal servers : 
edit the following to (uncomment as well remove \# ): 
```
c.NotebookApp.ip = '0.0.0.0'
c.NotebookApp.allow_origin = '*'
```

### Working with Jiant (allenNLP)

```
conda install -c conda-forge jsonnet 
module spider gcc 
module load gcc/9.1.0
conda install -c conda-forge regex
pip install allennlp
```

`pip install` <--> `conda install -c conda-forge` 
**Don't load python module!!!**

### Common problems

* Conda install reverts the changes of loading cuda/cudnn  
use : ```conda uninstall tensorflow-gpu cudatoolkit cudnn ```
* Tensorflow was compiled with diffent version of  cudnn  and currently is a different version is loaded. Just load the correct/earlier version of cudnn by which *tensorflow-gpu* was  installed
* Tensorflow is not compatible to use gpu. cuda/cudnn used during installation doesn't match with the tensorflow binary from which it was created.  ```pip uninstall tensorflow-gpu``` or  possibly delete the  whole  environment and follow the  above procedure.

# hacks
### Downloading Googe drive link
(ref: https://stackoverflow.com/questions/25010369/wget-curl-large-file-from-google-drive)
```
pip install gdown
gdown https://drive.google.com/uc?id=<id here>
```

### Setting up keys
Local Computer:
In `~/.ssh/config` Add the following

```
Host prince
    HostName prince.hpc.nyu.edu
    User net_id
    PubKeyAuthentication yes
    IdentityFile /Users/local_name/.ssh/id_rsa
``` 
The last `IdentityFile` is the private key to use 

Server
```
chmod 700 /home/<net_id>/
chmod 700 ~/.ssh
chmod 600 ~/.ssh/*
```

Add public key to the `~/.ssh/authorized_keys`

``ssh prince``
### Using Tunnel   (Outside NYU)  
ssh into gw.hpc.nyu.edu first, or use cisco vpn (allows file  transfer)
```
ssh pp1953@gw.hpc.nyu.edu 
```

### Linking up the prince storage to local
*very helpful, if  you  are not a big  vim fan*
```
sshfs -p 22 pp1953@prince.hpc.nyu.edu:/scratch/pp1953 ~/project
```
### Mount Point Using Access Point (Using tunnel)
```

ssh -f pp1953@access.cims.nyu.edu -L 2222:cassio.cs.nyu.edu:22 -N
sshfs -p 2222 pp1953@127.0.0.1:/misc/vlgscratch4/LakeGroup/pathak/Personre-id ~/NYU/temp/
```
or 
```
ssh -f <user_name>@<tunnel server> -L 2222:<target server>:22 -N
sshfs -p 2222 <user_name>@127.0.0.1:/desired/path/ local/path/
```

Unmounting process remains the same

`umount -f  ~/NYU/temp/`


### Using tmux (use mouse scrolling)
For god's sake use tmux
```
tmux new -s session_name
tmux a -t session_name
control + b -> # (sliding between windows) or control + b -> ' -> # (window >10) 
```
Set mouse scrolling on by : (Mac control + b -> shift + : ->`setw -g mouse on ` or `setw -g mode-mouse on` )

### Pdb multiple Line code 
`from IPython import embed; embed()`



# Summary
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

