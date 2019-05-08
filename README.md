#Installing Tensorflow on Prince   


## Using Tunnel   (Outside NYU)  
```
ssh pp1953@gw.hpc.nyu.edu 
```

## Linking up the prince storage to local
```
sshfs -p 22 pp1953@prince.hpc.nyu.edu:/scratch/pp1953 ~/project
```

## Requesting GPUs :   
```
srun -c4 -t24:00:00 --mem=30000 --gres=gpu:p40:1 --pty /bin/bash
```


##Remove any preloaded modules  
```
module purge
```


##delete any conda environment   
(if something goes wrong) 
Clean the  environment :  
```
conda clean -i -l -t -p -s   
```
Conda seems to bea bug, doesn't  delete the local files, will eventually  exhaust all the files 
```
conda remove --name vision --all
```
