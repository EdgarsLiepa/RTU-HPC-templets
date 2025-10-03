#HPC #Tools #Server 

## Links 

RTU HPC uses Torque/Moab resource manager. [Documentation](https://support.adaptivecomputing.com/hpc-cloud-support-portal-2/)

[Appendix A: Commands overview](https://docs.adaptivecomputing.com/torque/4-0-2/Content/topics/12-appendices/commandsOverview.htm)

[RTU HPC user manual](https://hpc-guide.rtu.lv/index.html)

[HPC platform overview](https://hpc-platforma.rtu.lv/)
## Useful commands

**Print submitted  processes**

``` bash 
qstat

qstat -s

# Print extended information of running processes 
qstat -f

# or just info about used resources:
qstat -f job_id | grep resources_used

# show on what node job is running 
qstat -n job_id
```


**Delete Job with ID** 

``` bash 
qdel <Job ID>
```

**Show all user submited jobs and reserved resources**

``` bash 
showq

showq -s # Workload summary
```


**Submit job** 

``` bash 
qsub <path to excutable file>
```


**Submit job by exporting environmental arguments**

*-v variable_list*

*Expands the list of environment variables that are exported to the job.*

*In addition to the variables described in the "Description" section above, variable_list names environment variables from the qsub command environment which are made available to the job when it executes. The variable_list is a comma separated list of strings of the form variable or variable=value. These variables and their values are passed to the job.*

``` bash


# pass arguments ${f} to shell script file
qsub -v f="arg-f" test2.sh
# or pass arguments $1 == "arg1" $2 == "arg2" ..... to shell script file
qsub -v "arg1 arg2 ..." test2.sh


```



**Submit job with flag arguments**

*Specfies the arguments that will be passed to the job script when the script is launched.*
~~~ bash

qsub -F "-i inputARg -l \"String input\" " cmd_flags_example.sh
# note that flag l takse string input and requires '\' for Quotation marks "  

qsub -F "myarg1 myarg2 myarg3=myarg3value" myscript2.sh




# or with -v  

`#!/bin/bash echo "$FOO $BAR"`

and your command like so:

`qsub -v FOO="hello",BAR="world" example.sh`

~~~


**Use interactive mode**

``` bash 
qsub -I

qsub -I -q gdn # Specify task queue type (batch, fast long, highmem, gdn - node for genomeics processing)
```


**request interactive GPU node**
Get nodes with feature V100 GPU and reserve two.

~~~bash
qsub -A <balance_name> -l nodes=wn:ppn=32:gpus=2,mem=120G -l feature=v100 -I
~~~


**See individual compute node status and specification**
Use the command to query the status of all nodes, showing the number of CPUs and the amount of memory in each node.
``` bash
pbsnodes -a
```


**There is a command to check all queues available on  cluster:**

~~~ bash
qstat -q
qstat -Q
~~~


**to  reserve gdn node with increased execution time**

``` bash 
qsub -l nodes=wn61:ppn=48 -q long  
# or   
qsub -q long -l nodes=1:ppn=48, feature=epyc7f72
```


~~~bash

# show node summary
nodes

# show licences
mdiag -n

checkjob -vvv <Job_ID>
checkjob -vvv 4045110.rudens

~~~


## Modules

Modules are reusable software packages. Mostly installed by HPC team and available for all users.
Local modules can be create as well.

~~~bash
# Show available software  
module avail

# Get info 
module --help

# show information about module
module show bio/samtools/1.10

# loaded modules
module list

module purge # unload all modules
~~~

Get information about HPC specific module. 

```bash 
 module help glibc-2.17-centos6
 module show glibc-2.17-centos6
 module whatis glibc-2.17-centos6
```

It is possible to create custom module files for software at \*/private_modules directory


### Copy trhough GDN node to/from hpc and sizif

[**Manual**](https://lftp.yar.ru/lftp-man.html)

While copying from/to RTU HPC errors can happen and by using programs like *scp* erorrs can be missed.

To connect GDN node from BMC NAS lftp can be used:  

``` bash 
# change user name to NAS user. same as sizif
lftp -u username -p 989 ftp://<ip address>  -e " set ssl:verify-certificate no"

# set if error -> ls: Fatal error: Certificate verification: Not trusted
# there might be safer option.
set ssl:verify-certificate no
```

where *username* is NAS username (same as ***sizif*** )

### Copy

mirror command copies files. Use `-I` to include matching files

``` bash 
# -I allows to use pattern matching
mirror -I remote_path/*_25*.fq.gz
```

use -R with mirror to upload file to connected device.

``` bash 
# -R reverse mirror (put files)
mirror -R local_path/local.file
```

## .sh commands 

**print resources used on cluster**

~~~shell
# print RAM assigned
cat /proc/meminfo | grep MemTotal

cat /proc/cpuinfo | grep processor | wc -l

  

# -- Displaying job information

echo Running on host `hostname`

echo Time is `date`

echo Current working directory is `pwd`

echo "Node file: $PBS_NODEFILE :"

echo Using ${NPROCS} processors across ${NNODES} nodes
~~~

## Check if script was sent with Torque

## Collect log files

Create sub folders in projects for script execution that compile all executed log files 


## Strings

Use this to escape special characters for use together 4 example with *sed*
~~~ shell
ESCAPED_KEYWORD=$(printf '%s\n' "$TRIMMED" | sed -e 's/[]\/$*.^[]/\\&/g');
~~~


## Rstudio

https://www.rc.virginia.edu/userinfo/howtos/rivanna/launch-rserver/

https://www.bioconductor.org/help/docker/

https://rocker-project.org/use/singularity.html


``` bash
## Get container wth all dependencies
singularity pull rocker_shiny_verse_4_4.sif docker://rocker/shiny-verse:4.4

## or different version
singularity pull docker://bioconductor/bioconductor_docker:devel

## Start interactive JOb or send as a task
qsub -I -A hpc -l nodes=1:ppn=5,mem=64g

## run container
singularity exec \
	--scratch /run,/var/lib/rstudio-server --workdir $(mktemp -d)    \
	~/tools/singularity_containers/bioconductor_docker_devel.sif rserver \
	--www-address=127.0.0.1 \
	--server-user=$(whoami)

```


after job is submitted and running with container, open another terminal session on local computer and create a ssh tunel through login node to a compute node. Before check on witch node job is running with `qstat -n job_id`

```
ssh -L 8787:127.0.0.1:8787 -J edgars01@ui-2.hpc.rtu.lv edgars01@wn72
```


then it should be possible to access Rstudio through web browser by writing `127.0.0.1:8787` in address bar.

You might need to set up local path where installed dependencies will be saved after session is over.

--bind ~/R:/usr/local/lib/R
```R

# Create a personal library if it doesn't exist 
dir.create("~/Rlibs", showWarnings = FALSE)`


### Set Default Library for the Session
.libPaths("~/Rlibs")

 if (!require("BiocManager", quietly = TRUE))
+ install.packages("BiocManager")
> BiocManager::install(version = "3.20")

```



## JupyterLab

Similar to how Rstudio is run.

First install jupyter lab on HPC locally or use container.


Lets go through local Installation.

create all dependencies on Conda environment
```bash
conda create --name jupyter jupyterlab

conda activate jupyter

conda install jupyter
```


Run jupyter lab on a compute node with port 8842. 

```bash

qsub -I -A bmc_pl_bior_covid -l procs=8,mem=16g

jupyter lab --no-browser --port=8842 --NotebookApp.allow0.0^Cin='*' --NotebookApp.ip='0.0.0
```


Then connect in second terminal with port forwarding.

```bash 
ssh -L 8842:127.0.0.1:8842 -J edgars01@ui-2.hpc.rtu.lv edgars01@wn64
```



## COPY matched symlink

when using nextflow a lot of times files in work folder are link as a links to output.
If copy is specified for nextflow process output it creates duplicates, if move is used pipeline cant be resumed. So best course of action is to collect all files after everything is done and move to destination folder

``` bash
for link in SN_100; do

	# make dir if doesnt exist
    mkdir -p ../metaquast/$link/
	
	# -n Dont owerwite if file exist
	# -a, --archive
    #          same as -dR --preserve=al
    cp -an "$(readlink $link)/metaquast" ../metaquast/$link

done
```



Copy symlink and remove source.

```bash
rsync -avL --remove-source-files $(readlink -f *_sorted.bam) ./
```