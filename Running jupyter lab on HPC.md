
Install all dependencies on Conda environment

```bash
conda create --name jupyter jupyterlab

conda activate jupyter

conda install jupyter
```


Start jupyter lab
```bash

qsub -I -A bmc_pl_bior_covid -l procs=8,mem=16g

jupyter lab --no-browser --port=8888 --NotebookApp.allow0.0^Cin='*' --NotebookApp.ip='0.0.0
```


