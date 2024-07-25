# Paraver

Having the ability to visually inspect an application's behaviour could help users to better understand and furhter explore any underlying performance issues. That is especially useful for highly parallel codes employing MPI, OpenMP, CUDA, etc... In this section we present a short introduction to two open source tools used in application analysis.

## Paraver

[Paraver](https://tools.bsc.es/paraver) is a highly configurable application performance analyzer based on data from event traces. It is used to explore the collected data and to qualitatively analyse the application behavior by visual inspection in order to better understand an applicaiton's behavior. That could then help to focus on a more detailed quantitative analysis of the problems. Paraver itself does not create run traces. It is only used for trace analysis.

## Extrae

[Extrae](https://tools.bsc.es/extrae) is an instrumentation package devoted to generate Paraver trace files for a subsequent application performance analysis. Extrae is highly configurable

compiled and run with the shared memory model (like OpenMP and pthreads), the message passing (MPI) programming model 


Use Extrae to create a trace of a parallel program run within a batch job.
This script to be called right after "mpirun -np NPROCS" in the job submit script.
Example: mpirun -np $SLURM_NPROCS ./slurm-trace-extrae.sh <executable> [parameters]
To use with "srun" instead within a job script OMPI must be built with SLURM's PMI support.

https://tools.bsc.es/doc/html/extrae/index.html
https://tools.bsc.es/sites/default/files/documentation/2A_Instrumentation.pdf
https://www.vi-hps.org/cms/upload/material/tw42/Extrae-Paraver-Hands-On.pdf

```
#!/bin/bash

module load Extrae/4.0.4-gompi-2022a

# Configure environment
export EXTRAE_HOME=${EBROOTEXTRAE}
export TRACE_NAME=extrae_traces.prv

# Configure Extrae
export EXTRAE_CONFIG_FILE=/PATH/TO/extrae_config.xml

# Load the tracing library (include suffix “f” for Fortran codes)
export LD_PRELOAD=${EXTRAE_HOME}/lib/libmpitracef.so

# Pass the compiled binary to Extrae for trace generation
$*
```

module load Paraver/4.11.1-foss-2022a


text *formatting*

more ```text formatting_```


