# Slurm cluster job submission script

`sub.queue` is a bash script designed to submit jobs to a cluster using the Slurm queues. It has been heavily customized for the CSIC cluster Trueno, but it will be updated to handle other clusters or queue systems.

## How it is used

`sub.queue` expects to be installed somewhere accessible in your cluster's home directory, such as `$HOME/bin`. Given this, you would call the script similar to
```
$ sub.queue my-program argument1 argument2
```
Here `my-program` is the executable or script that you wish to run, `argument1` and `argument2` are two sample arguments expected by that script.

Once invoked, `sub.queue` will create a job directory `$HOME/jobs` where it will place the job invocation script, in this case `$HOME/job/job.0001`. It will submit this job script to the queue system, which will then execute `my-program` with the given arguments, in the directory from which `sub.queue` was invoked. The text and error ouptut of the program will be redirected to `$HOME/job/output/job.0001`.

## Types of jobs and options

The script accepts some important arguments, but it is best to describe them in the context of what type of job one tries to run.

### Single process, multiple cores

This would be the case of a large Python simulation, running a heavy multithreaded version of Numpy, based on the Intel MKL library. In this case you simply need to indicate the script how many threads can the Intel MKL library run
```
$ sub.queue -mkl 4 python -u some_heavy_simulation.py
```
Indicating the number of threads is important, because we need to tell the cluster how many processors we will require in a given node. If you do not indicate the number of threads, the cluster will assume you only require 1 processor. Imagine then what happens if this job lands on a computer in the cluster where 11 processors are being used by somebody else's simulations and there is only 1 core free left: your job and that of the other person would mutually interfere!

### Multiple nodes using MPI, multiple threads

This is the case in which you have a large multiprocess simulation, running on different computers, communicating with each other by a message passing interface (MPI).
```
$ sub.queue -mpi 5 -n 5 -mkl 3 python -u my_mpi_simulation.py
```
In this line of code we are indicating that we want to launch 5 processes, each running a copy of Python that executes `my_mpi_simulation.py`, and consuming 3 cores. We also indicate that, ideally, we would like those 5 processes to run on 5 different computers, so that they do not interfere with each other. In total, we would be using `3*5=15` cores.

## Other options

The script allows passing other important options

- `-m memory` indicates the maximum amount of memory consumed by each process. It defaults to 2 Gigabytes, which is very low.
- `-t time` indicates the total run time required by the simulation. It is specified as `DD:HH:MM:SS` for days (`DD`) hours (`HH`), minutes (`MM`) and seconds (`SS`). You can omit the largest numbers: e.g. 10:00 is 10 minutes.
- `-clean` wipes out all job scripts and outputs from the `$HOME/jobs/ directory.
- `-q name` indicates the `name` of the queue that this job should land on.
