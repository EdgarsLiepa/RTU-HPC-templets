#HPC #Tools #Server 

RTU HPC uses Torque/Moab resource manager. [Documentation](https://support.adaptivecomputing.com/hpc-cloud-support-portal-2/)

## Usefull commands

**Print submited  processes**

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


**Submit job by exporting envoirmental arguments**

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
~~~


**Use interactive mode**

``` bash 
qsub -I

qsub -I -q gdn # Specify task queue type (batch, fast long, highmem, gdn - node for genomeics processing)
```


**See individual compute node status and specification**

``` bash 
pbsnodes
```


**There is a command to chek all queues available on  cluster:**

~~~ bash
qstat -q
~~~

**to  reserve gdn node with increased execution time**

``` bash 
qsub -l nodes=wn61:ppn=48 -q long  
# or   
qsub -q long -l nodes=1:ppn=48, feature=epyc7f72
```

### Copy trhough GDN node to from hpc and sizif

While copying from/to RTU HPC errors can happen and by using programs like *scp* erorrs can be missed.

To copy trhough GDN node to/from BMC NAS lftp can be used:  

``` bash 
# change user name to NAS user. same as sizif
lftp -u username -p 989 ftp://10.245.1.147  -e " set ssl:verify-certificate no"

# set if error -> ls: Fatal error: Certificate verification: Not trusted
# there might be safer option.
set ssl:verify-certificate no
```

where *username* is NAS username (same as ***sizif*** )


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