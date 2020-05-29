# Discovery User Guide

## Introduction
Thank you for volunteering to try out our new test cluster, discovery.usc.edu.  Almost every aspect of this system is different from the current one. Your feedback will be invaluable to helping strengthen it and improve the experience for the rest of the community when it goes into production.
## Getting Help
Please direct any questions regarding the test system to hpc-support@usc.edu. All emails sent here will be sent to a separate ticketing system. You are also encouraged to provide feedback on the test ticketing system as we will be using it when we launch the new cluster.

## Logging in
You can access the test cluster at the hostname `discovery.usc.edu`. Authentication should be the same as `hpc-login3.usc.edu`.

As a reminder, you will need an ssh client. Most operating systems have a built-in ssh client. Mac and Linux machines have 'terminal' while Windows has one in the 'Command Prompt' utility. If your computer does not have an ssh client you can install one such as PuTTY.

After opening the command line utility for your operating system type

    ssh <user_name>@discovery.usc.edu

where `<user_name>` is your USC NetID (your email address without "@usc.edu"). If this is your first time logging in you may see scary looking message asking if you trust this host. Type 'yes' (or similar) to continue. You will then be prompted for your password. Enter your USC NetID password (the one you use for other USC services).

**Note:** They may be **no** visual feedback as you enter your password. This is a security feature designed to obscure your password and is expected.

After entering your password you will then be prompted for Duo authentication. Select your preferred Duo method. If you have none set up, you can create one by following along with ITS's documentation here: https://itservices.usc.edu/duo/

If you encounter issues connecting to `discovery` while off campus, it may be necessary to first connect to a VPN. Please see this page for information on setting it up: https://itservices.usc.edu/vpn/.

## Storing data
Every user will have access to 3 different storage locations, home, proj, and scratch.

###  Home directory
Your home directory is of the form:

    /home1/<user_name>

/home1 is a high-performance parallel file system, you can run IO intensive jobs directly from `/home1`. However, quota for each user on `/home1` is only 100GB and 2M files. We keep 14 days of snapshots of `/home1` which can be think of as semi-backups, meaning if you accidentally deleted some data we are able to recover it within 14 days. (If the file was created and deleted whithin one day period, then the snapshot can't recover it) You should always keep extra backups of your important data since snapshots or semi-backups are not real backups.

To check your current usage of quota on /home1 use:
```
$ myquota
```
on `discovery.usc.edu`

###  Current Project directory (Read only)
To make it easier to start testing, your project directory is available on a read-only basis to `discovery.usc.edu`

###  Scratch directory
Your can find your scratch directory at 

`/scratch/<user_name>`

/scratch is a high-performance parallel file system, you can run IO intensive jobs directly from /scratch. No quota will be enforced on /scratch. There's no backup of any sort on /scratch.

### Storage Summary
|PATH|Quota|No. File quota|High Performance IO|
|-|-|-|-|
|/home1|100GB|2M|Yes|
|/scratch|none|none|Yes|

### Transfering data
You can use `discovery.usc.edu` directly as data transfer node for now. scp and sftp are recommended. 

## Software Modules (Lmod)
One of the biggest user-facing changes to the test cluster is trading the setup.sh method of using software for the module system. Lmod has the ability to track software dependencies to ensure you have better control of your environment. (No more logging out and then back in to reset your environment!)

The official documentation for Lmod can be found here: https://lmod.readthedocs.io/en/latest/010_user.html

Here are some of the basics.

###  Checking available software
To see what modules you can load into your environment, run the command `module avail`. You should see something similar to:

    --------/spack/apps/lmod/linux-centos7-x86_64/Core --------
    gcc/4.9.4     gcc/9.2.0    (D)    intel/19.0.4 (D)
    gcc/8.3.0    intel/18.0.4        usc

    Where:
    D:  Default Module

    Use "module spider" to find all possible modules and extensions.
    Use "module keyword key1 key2 ..." to search for all possible modules
    matching any of the "keys".


If you have a fresh environment only `Core` modules are available. These are usually compilers but can be applications like Matlab that are prebuilt and have no dependencies.
###  Loading/Unloading software

Typically loading modules is as simple as typing `module load <software_name>`. `<software_name>` must be visible when you run module avail.

By running

    module load <software_name>

Lmod will set your environment such that <software_name> will be placed in your path. To see what environment variables get set, you can run:

    module show <software_name>

If there are multiple versions of `<software_name>` you can specify a version like so:

    module load <software_name>/<version>

For example to load gcc version 8.3.0:
```
   $ module load gcc/8.3.0
```
If no version is specified:
```
   $ module load gcc
```
the default version will be loaded. The default version is indicated with a (D) next to it.


To unload a specific module you can run:

```
    module unload <software>
```

To totally clear your environment you can run:

```
    module purge
```


###  Finding software
The first time you log in and run `module avail`, it may not seem like much software is available. This is actually a safety feature that prevents you from loading incompatible modules. If you would like to explore the software tree, you can start loading modules and new ones will unlock. To see all applications built with a certain compiler, you can load that module. Everything built with that compiler will become visible in module avail.

The current naming scheme for modules is `<software_name>/<version>-xxxx`.

Where `xxxx` is a 4 character hash (random looking characters). We plan to remove the hash from the module naming scheme in the production environment.

#### Module spider
If you know the name of a software package you can use the `module spider` command to find out if it exist and how to load it.

For example:
```
  $ module spider samtools

    ------------------------------------------------------
    samtools: samtools/1.9-rq3g
    ------------------------------------------------------

        You will need to load all module(s) on any one of the lines below before the "samtools/1.9-rq3g" module is available to load.

        gcc/8.3.0

        Help:
        SAM Tools provide various utilities for manipulating alignments in the
        SAM format, including sorting, merging, indexing and generating
        alignments in a per-position format
```

### Environment management

One of the nicest features of Lmod is that it can track software dependencies automatically. For example lets say you want to use the `jellyfish` package compiled with `gcc/8.3.0`. You would load like like so:

```
module load gcc/8.3.0
module load jellyfish
```

If for some reason you need to use the `intel` compiler set, you can use the `module swap` command like so:

```
module swap gcc intel

The following have been reloaded with a version change:
  1) jellyfish/2.2.7-oauz => jellyfish/2.3.0-2hxl
```

Lmod automatically changes the `jellyfish` module to one that was compiled with `intel`.

### Module settings

Loading the desired module will make some changes to the environment. Some common settings include:

|Environment Variable|Description|
|-|-|
|`{NAME}_ROOT`|Creates environment variable {NAME}_ROOT which points to the root directory of installation|
|`PATH`|Adds `{NAME}_ROOT/bin` to PATH|
|`MANPATH`| Adds `{NAME}_ROOT/share/man` to search manual path|
|`LD_LIBRARY_PATH`| Adds appropriate library directory to library search path|
|`PKG_CONFIG_PATH`| Enables package to be found by `pkg-config`, useful for building other software|
|`CMAKE_PREFIX_PATH`| Enables package build settings to be found by `cmake`, useful for building other software|

Every package is different, some will have extra environment variables while others will have fewer. For example, Intel compilers will have a `INTEL_LICENSE_FILE` setting while Boost has  directory to add to `PATH`.

### Python Packages

Python packages are not currently tracked in the module system. To find available packages you can use the `pip3` command like so

    pip3 freeze
    appdirs==1.4.3
    atomicwrites==1.3.0
    attrs==19.3.0
    backcall==0.1.0
    cffi==1.14.0
    ...
    ...

To install new packages you can install them to your home directory like so

    pip3 install --user

### R Packages

R packages are not currently tracked in the module system. To find available packages you can start up `R` and use the `installed.packages()` command like so

    packs <- installed.packages()
    names(packs[,'Package'])
     [1] "abind"                "acepack"              "AnnotationDbi"       
     [4] "askpass"              "assertthat"           "backports"           
     [7] "base"                 "base64enc"            "BH"                  
    [10] "Biobase"              "BiocFileCache"        "BiocGenerics"        
    [13] "BiocManager"          "BiocParallel"         "BiocVersion"         
    ...
    ...

To install new packages you can install them to your home directory like so:



### Installing new software

Most likely you will need to build your own software and it will have other software it is dependent on. In order to ensure reproducibility the login node has minimal software installed through the operating system. All dependencies must either be resolved through the module system or be built by the user.

Applicable modules add the appropriate directory to `PKG_CONFIG_PATH`. This enables build systems such as configure scripts and cmake, which often use the `pkg-config` utility, to automatically detect dependent software. If this is unsupported by the build system, you may have to manually point the installer to dependent software.To aid in this process all modules have an associated environment variable `{NAME}_ROOT` where `{NAME}` is the name of the module. For example the Python module has `PYTHON_ROOT`.

## Running jobs
The new system will continue to use the SLURM job scheduler.
As a reminder, you can find our documentation on Slurm here:
https://hpcc.usc.edu/support/documentation/slurm/

Using the job scheduler should be the same but there are few changes of note:

* `main` partition max wall-time is 48 hours.
* Each user can use 96 CPU cores in `main` partition regardless of number of jobs and submit up to 2000 jobs
* If you don't specify any partition, it will default to `main`
* New `oneweek` partition with max wall-time one week
* Each user can use 12 CPU cores regardless of number of jobs and submit up to 50 jobs.

### Resources and limits summary

|partition|Nodes|CPUS|RAM(GB)|MaxCPUsPerUser|MaxSubmitJobsPerUser|Default|
|---|---|---|---|---|---|---|
|main |20 | 24|95 |384|5000|Yes|
|oneweek |2 | 24|95 |12|50|No|

### Example Job script

For your convenience, here's a example job script for a serial running job:

```
#!/bin/bash
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem-per-cpu=2GB
#SBATCH --time=1:00:00

module purge
module load gcc/8.3.0
module load python

python3 script.py
```

for a parallel running job:

```
#!/bin/bash
#SBATCH --ntasks=64
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=2GB
#SBATCH --time=1:00:00

module purge
module load usc  
srun ./mpi_hello.out
```
