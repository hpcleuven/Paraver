# Paraver
## Paraver

[Paraver](https://tools.bsc.es/paraver) is an application performance analyzer based on event traces with the flexibility to explore the collected data.
It can be used to qualitatively analyse the application behavior by visual inspection in order to better understand and then to focus on the detailed quantitative analysis of the problems.

## Extrae

[Extrae](https://tools.bsc.es/extrae) is a package devoted to generate Paraver trace files for a subsequent application performance analysis.


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


