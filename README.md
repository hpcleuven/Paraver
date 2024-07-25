# Application profiling on HPC systems

Having the ability to visually inspect an application's behaviour could help users to better understand and furhter explore any underlying performance issues. That is especially useful for highly parallel codes employing MPI, OpenMP, CUDA, etc... In this section we present a short introduction to two open source tools used in HPC application analysis.

## Paraver

[Paraver](https://tools.bsc.es/paraver) is a highly configurable application performance analyzer based on data from event traces. It is used to explore the collected data and to qualitatively analyse the application behavior by visual inspection in order to better understand an applicaiton's behavior. That could then help to focus on a more detailed quantitative analysis of the problems. Paraver itself does not create run traces. It is only used for trace analysis.

## Extrae

[Extrae](https://tools.bsc.es/extrae) is an instrumentation package devoted to generate Paraver trace files for a subsequent application performance analysis. Extrae is highly configurable and can trace applications compiled and run with the shared memory model (like OpenMP and pthreads), message passing interface (MPI), or both. The configuration of what to trace is passed via an XML file with all the necessary directives in it. An example of such an XML configuration is given here below. A limited number of the configuration calls can also be set as environment variables if the computer system does not support XML libraries. The user is very much encouraged to refer to the latest [Extrae Documentation](https://tools.bsc.es/doc/html/extrae/index.html) for an explanation of the XML tracing directives.

Example Extrae XML configuration:
```
<trace enabled="yes"
  home="/apps/leuven/rocky8/skylake/2022a/software/Extrae/4.0.4-gompi-2022a"
  initial-mode="detail"
  type="paraver">

  <mpi enabled="yes">
    <counters enabled="yes" />
  </mpi>

  <openmp enabled="no">
    <locks enabled="no" />
    <counters enabled="yes" />
  </openmp>

  <pthread enabled="no">
    <locks enabled="no" />
    <counters enabled="yes" />
  </pthread>

  <callers enabled="yes">
    <mpi enabled="yes">1-3</mpi>
    <sampling enabled="no">1-5</sampling>
  </callers>

  <counters enabled="yes">
    <cpu enabled="yes" starting-set-distribution="1">
      <set enabled="yes" domain="all" changeat-time="0">
        PAPI_L1_TCM,PAPI_L2_TCM,PAPI_L1_DCM,PAPI_L2_DCM,PAPI_CA_SHR
      </set>
    </cpu>
    <network enabled="no" />
    <resource-usage enabled="no" />
    <memory-usage enabled="yes" />
  </counters>

  <buffer enabled="yes">
    <size enabled="yes">500000</size>
    <circular enabled="no" />
  </buffer>

  <sampling enabled="yes" type="default" period="50m" variability="5m" />

  <merge enabled="yes"
    synchronization="default"
    max-memory="140000"
    joint-states="yes"
    keep-mpits="yes"
    sort-addresses="yes"
    overwrite="yes">
    $TRACE_NAME$
  </merge>

</trace>
```

Note that Extrae relies on the [Performance API](https://github.com/icl-utk-edu/papi) (PAPI) for low-level system monitoring calls.

## Trace and profile

In order to create a trace of a running compiled binary one must first properly configure the Extrae environment:

```
export EXTRAE_HOME=/PATH/TO/EXTRAE_INSTALLATION
export EXTRAE_CONFIG_FILE=/PATH/TO/extrae_config.xml
export LD_PRELOAD=${EXTRAE_HOME}/lib/libmpitrace.so
export TRACE_NAME=my_trace.prv
```

In the above example we first point to the Extrae installation directory, then we point to the XML configuration file containing the tracing instructions of choice, and finally via ```LD_PRELOAD``` we point to the proper tracing library, e.g., for an applicaiton compiled with MPI one should use ```libmpitrace```. Extrae uses separate libraries for C/C++ codes and for FORTRAN code. E.g., ```libmpitrace``` is for C/C++ and for a FORTRAN code one would use ```libmpitracef``` (attach the suffix “f”). The last ```export ...``` above specifies the name of the trace file which will be created.

After the environment has been prepared the application can be traced like so:

```
${EXTRAE_HOME}/bin/extrae -config ${EXTRAE_CONFIG_FILE} <executable> [<parameters>]
```

Extrae can also create a trace of a parallel program run within a batch job. For that purpose the above environment configuration is first stored in a bash shell script, e.g., called *slurm-trace-extrae.sh*. On VSC one could also use environment modules and the bash script will look like, e.g.:

```
#!/bin/bash
module load Extrae/4.0.4-gompi-2022a
export EXTRAE_HOME=${EBROOTEXTRAE}
export EXTRAE_CONFIG_FILE=/PATH/TO/extrae_config.xml
export LD_PRELOAD=${EXTRAE_HOME}/lib/libmpitrace.so
export TRACE_NAME=my_trace.prv
$*
```

Note the extra ```$*``` operand at the end of the bash script. That will pass the program binary to Extrae for tracing. Then that script is to be called right after the ```mpirun -np NPROCS``` directive in the job submit script, e.g.:

```
mpirun -np $SLURM_NPROCS ./slurm-trace-extrae.sh <executable> [<parameters>]
```

NOTE: To use with ```srun``` instead within a job script OMPI must be built with SLURM's PMI support.

And here is an example of a Slurm job submission script:

```
#!/bin/bash -l
#SBATCH --cluster=wice                      # cluster name
#SBATCH --job-name="my-tracing-job"         # job name
#SBATCH --time=01:10:00                     # max job run time hh:mm:ss
#SBATCH --nodes=3                           # number of nodes
#SBATCH --ntasks-per-node=72                # tasks per compute node
#SBATCH --mem-per-cpu=2500M                 # memory per CPU
#SBATCH --output=stdout.%x.%j               # save stdout to file
#SBATCH --error=stderr.%x.%j                # save stderr to file
#SBATCH --output=%x-%j.log                  # job log
#SBATCH --account=<my_account>              # compute credits account

# Modules
module load cluster/wice/batch
module load foss/2022a

# Extrae bash script configuration
EXTRAE_CONF_SCRIPT="slurm-trace-extrae.sh"

[more user input...]
...

# Run a trace on the program
mpirun -np $SLURM_NPROCS ./${EXTRAE_CONF_SCRIPT} <executable> [<parameters>]
```

Once a trace has been created it can be loaded in Paraver. For small trace files in the order of several gigabytes one could use Paraver over NX to inspect the application directly from the VSC cluster. For larger traces, however, it would be more practical if users install Paraver on their local machines, transfer the trace, and profile it locally.

To use Paraver within NX, open an NX session, start a terminal, load the Paraver module, and finally start the GUI. For example:

```
module load Paraver/4.11.1-foss-2022a
```

Then start the Paraver gui with:

```
wxparaver
```

Once the Paraver GUI opens users can load the trace file from the *File/Load Trace* menu.

## References
https://tools.bsc.es/doc/html/extrae/index.html

https://github.com/icl-utk-edu/papi/wiki/

https://tools.bsc.es/sites/default/files/documentation/2A_Instrumentation.pdf

https://www.vi-hps.org/cms/upload/material/tw42/Extrae-Paraver-Hands-On.pdf
