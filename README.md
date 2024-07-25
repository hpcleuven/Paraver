# Application profiling on HPC systems

Having the ability to visually inspect an application's behaviour could help users to better understand and furhter explore any underlying performance issues. That is especially useful for highly parallel codes employing MPI, OpenMP, CUDA, etc... In this section we present a short introduction to two open source tools used in HPC application analysis.

## Paraver

[Paraver](https://tools.bsc.es/paraver) is a highly configurable application performance analyzer based on data from event traces. It is used to explore the collected data and to qualitatively analyse the application behavior by visual inspection in order to better understand an applicaiton's behavior. That could then help to focus on a more detailed quantitative analysis of the problems. Paraver itself does not create run traces. It is only used for trace analysis.

## Extrae

[Extrae](https://tools.bsc.es/extrae) is an instrumentation package devoted to generate Paraver trace files for a subsequent application performance analysis. Extrae is highly configurable and can trace applications compiled and run with the shared memory model (like OpenMP and pthreads), message passing interface (MPI), or both. The configuration of what to trace is passed via an XML file with all the necessary directives in it. A limited number of the configuration calls can also be set as environment variables if the computer system does not support XML libraries. The user is very much encouraged to refer to the latest [Extrae Documentation](https://tools.bsc.es/doc/html/extrae/index.html).

## Trace and profile

In order to create a trace of a running compiled binary one must first properly configure the Extrae environment:

```
export EXTRAE_HOME=/PATH/TO/EXTRAE_INStALLATION
export EXTRAE_CONFIG_FILE=/PATH/TO/extrae_config.xml
export LD_PRELOAD=${EXTRAE_HOME}/lib/libmpitrace.so
```

In the above example we first point to the Extrae installation directory, then we point to the XML configuration file containing the tracing instructions of choice, and finally via ```LD_PRELOAD``` we point to the proper tracing library, e.g., for an applicaiton compiled with MPI one should use ```libmpitrace```. Extrae uses separate libraries for C/C++ codes and for FORTRAN code. E.g., ```libmpitrace``` is for C/C++ and for a FORTRAN code one would use ```libmpitracef``` (attach the suffix “f”).

After the environment has been prepared the application can be traced like so:

```
${EXTRAE_HOME}/bin/extrae -config ${EXTRAE_CONFIG_FILE} <executable> [<parameters>]
```

Extrae can also create a trace of a parallel program run within a batch job. For that purpose the above environment configuration is first stored in a bash shell script, e.g., called *slurm-trace-extrae.sh*. Then that script is to be called right after *mpirun -np NPROCS* directive in the job submit script, e.g.:

```
mpirun -np $SLURM_NPROCS ./slurm-trace-extrae.sh <executable> [<parameters>]
```

NOTE: To use with "srun" instead within a job script OMPI must be built with SLURM's PMI support.





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


## References
https://tools.bsc.es/doc/html/extrae/index.html
https://tools.bsc.es/sites/default/files/documentation/2A_Instrumentation.pdf
https://www.vi-hps.org/cms/upload/material/tw42/Extrae-Paraver-Hands-On.pdf
