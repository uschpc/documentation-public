# Project Filesystem Status

Starting around mid-March, users began reporting multiple issues due to high load on our filesystem that hosts /rcf-proj2 and /rcf-proj3. This has caused problems for users trying to log in, access files, or run interactive jobs.

We have since stabilized the filesystem but have made it read-only to decrease the load and prevent further file access issues. The filesystem will remain in this state until June when we can deploy a new, previously planned filesystem and migrate all project data.

Previously, users were instructed to run their jobs in the /staging filesystem until our new project filesystem was up and running. However, /staging nearly reached its limited capacity within a few weeks. As a result, we have created a new, high-performing parallel filesystem for production runs called **/scratch**, which has a much larger capacity than /staging.

## /scratch: an all new, high-performing parallel filesystem

Your scratch directory is located at:

    /scratch/<username>

You should now be using /scratch for your production runs instead of /staging.

The new /scratch filesystem has a capacity of 806TB (compared to 266TB on /staging) and it can sustain 25GB/s of reading and writing performance, which is almost twice as much as /staging.

Your scratch directory is accessible only to you, and each user account is limited to a 10TB quota. If you need more space than this, please contact us at hpc@usc.edu.

Please note that /scratch is only usable with compute nodes on the Infiniband network. To request these nodes, add the ` #SBATCH --constraint=IB` option to your job scripts.

## /staging: decommissioned

## /scratch2: 709TB 

### How should I perform the data migration

There are two options to migrate data using an `rsync` command: (1) use the hpc-transfer node or (2) submit a SLURM job.

Note: first delete any files in /staging that are no longer needed. This will reduce the time needed to copy files. Additionally, regenerating data is much faster than copying data. For example, if you have a large custom Python or R installation under /staging, simply re-install it under /scratch rather than copying.

#### Via hpc-transfer

The hpc-transfer node, which has been recently upgraded with new hardware and new 40GB links, can migrate data in and out much faster than the login nodes.

From your computer, log in to hpc-transfer.usc.edu and authenticate via Duo:

```
ssh ttrojan@hpc-transfer.usc.edu
```

Note: If you get an SSH error about 'remote host identification has changed' when attempting to connect, the solution is to clear your 'known_hosts' file that is referenced in the error message. Open the 'known_hosts' key and manually delete the line beginning with 'hpc-transfer.usc.edu' and then save. Try the command again, confirm the new authenticity of host, and the error should be solved.

Once connected, use `screen` in combination with an `rsync` command, because the transfer could take a long time.

First, enter `screen` at the command line to start a `screen` session.

Second, enter an `rsync` command that looks something like the following:

```
rsync -avP /staging/proj/user/ /scratch/user/
```

Be sure to substitute your correct directory paths. This will start the transfer and display its progress.

Next, depress the keys `ctrl-a d` to detach the `screen` session, which continues the `rsync` job in the background. You can then continue other work or log out and the transfer will continue. To reattach that session and check progress enter `screen -r`. Once the transfer is complete, you can close the screen session by entering `exit` from within the session.

#### Via SLURM job

Alternatively, if you anticipate a long, multi-day transfer time, submit a SLURM job script using Infiniband nodes that looks something like the following:

```
#!/bin/bash
#SBATCH --ntasks=1
#SBATCH --time=48:00:00
#SBATCH --constraint=IB

rsync -avP /staging/proj/user/ /scratch/user/
```

Be sure to substitute the anticipated walltime needed and your directory paths.

## Backups

There are no backups for either /scratch or /staging. Please keep additional copies of your important data elsewhere to prevent accidental data loss. (If your PhD thesis relies on your data, keep at least three copies.)

## Sharing data with others
```
When sharing your files, please keep these in mind:
1. **never** set permission of dirs your own to **777**, which means **anybody** can delete your file
1. sharing **read** permission is sufficent, do not allow others to **write** to dirs you own, nor should you write to another user's dir
1. do not change permissions of your **HOME** dir and sub dirs, if it goes wrong, your login will be blocked because ssh checks for strict permissions
```
It may become necessary to share data with other users. Using access control lists (ACL) you can specify permissions on a per-user basis.

To minimize permission management, you may find it easist to create a designated directory just for sharing data like so

```
mkdir /scratch/user_name/shared
```

For each user you want to have access you will have to run these commands:

1. Allow `guest_user` to access (but not write) to your scratch directory
```
setfacl -m user:guest_user:rx /scratch/user_name
```


2. Allow `guest_user` access to the shared directory
```
setfacl -d -R -m user:guest_user:rwx /scratch/user_name/shared
```

By adding the `-d` option, new files and directories will have the same ACLs as their parent directory applied at creation. The `-R` option will recursively set access.

If you forget which permissions have been set you can run `getfacl` to check which ACLs have been set:

```
[ttroj@hpc-login2 new_dir2]$ getfacl test_file
# file: test_file
# owner: ttroj
# group: usc
user::rw-
user:travlr:rwx                #effective:rw-
group::rwx                      #effective:rw-
mask::rw-
other::---

```

## Installing software

Installing software on /scratch and /scratch2 should be as easy as installing it to your project directory. However, there are some special cases where it is more difficult. If you require further assistance, please contact us at hpc@usc.edu.

### Python

Python allows you to install and load packages from arbitrary directories.

Assume you have a directory of the form:

    package_root=/scratch/<username>/python_packages

You can install packages like so:

    pip3 install <package_name> --root=$package_root

To load a package, ensure you have appended your `PYTHONPATH` environment variable like so:

    export PYTHONPATH=${package_root}/lib/python3.7/site-packages:${PYTHONPATH}

### R

R allows you to install and load packages from arbitrary directories.

To install R packages in a library other than the default (`~/R`), you can use the `lib` argument for the `install.packages()` function:

```r
install.packages("package_name", lib = "package_path")
```

Note: For R version 3.6.0, if the install fails because of `ERROR: moving to final location failed`, enter the command `Sys.setenv(R_INSTALL_STAGED = FALSE)` and try installing again.

Alternatively, if using the command-line function `R CMD INSTALL` to install packages from source, you can use:

```sh
R CMD INSTALL --no-staged-install --library=package_path package_file.tar.gz
```

where `package_path` is the absolute path to where you want your R packages to be installed and the `--no-staged-install` flag instructs `R CMD INSTALL` to directly install the package in the folder, avoiding pre-installation in a temporary folder. Using the `--no-staged-install` flag potentially reduces issues if HPC is experiencing I/O problems. For `package_path`, it can be something of the form `/scratch/<username>/r_packages/3.6`.

Alternatively, users can take advantage of symbolic links (`symlinks`) to avoid specifying a library location and allow R to automatically handle that as explained [here](https://hpcc.usc.edu/resources/documentation/r/).

*Note: We recommend keeping packages separated by R version (Major.Minor) to avoid compatibility issues.*

To load an R package, run a command of the form:

```r
library("package_name", lib.loc = "package_path")
```
