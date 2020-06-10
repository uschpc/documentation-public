# Project Filesystem Status

Starting around mid-March, users began reporting multiple issues due to high load on our filesystem that hosts /rcf-proj2 and /rcf-proj3. This has caused problems for users trying to log in, access files, or run interactive jobs.

We have since stabilized the filesystem but have made it read-only to decrease the load and prevent further file access issues. The filesystem will remain in this state until June when we can deploy a new, previously planned filesystem and migrate all project data.

Previously, users were instructed to run their jobs in the /staging filesystem until our new project filesystem was up and running. However, /staging nearly reached its limited capacity within a few weeks. As a result, we have created two new, high-performing parallel filesystems for production runs called **/scratch and /scratch2**, each of which have a much larger capacity than /staging.

/staging was decommissioned on June 5, 2020.

## /scratch and /scratch2: two all new, high-performing parallel filesystems

Your scratch directory is located at:

    /scratch/<username>

/scratch has a capacity of 806TB.

Your scratch2 directory is located at:

    /scratch2/<username>

/scratch2 has a capacity of 709TB.

You should now be using /scratch or /scratch2 for your production runs instead of /staging.

Both directories are accessible only to you, and each user account is limited to a 10TB quota for each directory. If you need more space than this, please contact us at hpc@usc.edu.

Please note that /scratch and /scratch2 are only usable with compute nodes on the Infiniband network. To request these nodes, add the ` #SBATCH --constraint=IB` option to your job scripts.

## Backups

There are no backups for either /scratch or /scratch2. Please keep additional copies of your important data elsewhere to prevent accidental data loss. (If your PhD thesis relies on your data, keep at least three copies.)

## Sharing data with others

When sharing your files, please keep the following in mind:

1. Never set the permissions of your own directories to **777**, which means that **anybody can delete your files**.

2. Giving other users **read** permission for your files is sufficient. Do not allow others to **write** to directories that you own, nor should you write to another user's directory.

3. Do not change the permissions of your **HOME** directory and sub directories. If something goes wrong, your login will be blocked because ssh checks for strict permissions.

By default, your /scratch/user_name permission is set to 700, which means only yourself can read,write,exec. Changing 700 to something else is not recommanded. 
It may become necessary to share data with other users. Using access-control lists (ACLs), you can specify permissions on a per-user basis.

To allow `guest_user` read access to your scratch directory:
```
setfacl -Rdm u:guest_user:r-x /scratch/user_name  #(this will allow new files to be shared)
setfacl -Rm u:guest_user:r-x /scratch/user_name   #(this will allow existing files to be shared)
```
To remove `guest_user` read access from your directory:
```
setfacl -Rdm u:guest_user:--- /scratch/user_name   
setfacl -Rm u:guest_user:--- /scratch/user_name   
```
By adding the `-d` option, new files and directories will have the same ACLs as their parent directory applied at creation. The `-R` option will recursively set access.

If you forget which permissions have been set, you can run `getfacl` to check which ACLs have been set:

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
