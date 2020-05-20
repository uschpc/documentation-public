# Using the command line to transfer files

Command-line interface data-transfer utilities (clients) are available natively on macOS and Linux systems and on Windows through applications such as PuTTY and Cygwin. The common utilities, sftp, scp, and rsync are described on this page.

Due to security risks, please be mindful of the type of information being transferred. Where possible, omit all information that may be considered confidential. For examples of confidential information that requires additional consideration visit http://itservices.usc.edu/security/sensitive-info.

## Copying files from /staging to /scratch

From your computer, log in to hpc-transfer.usc.edu and authenticate via Duo:

```
ssh ttrojan@hpc-transfer.usc.edu
```

Note: If you get an SSH error about 'remote host identification has changed' when attempting to use one of these commands, the solution is to clear your 'known_hosts' file that is referenced in the error message. Open the 'known_hosts' key and manually delete the line beginning with 'hpc-transfer.usc.edu' and then save. Try the command again, confirm the new authenticity of host, and the error should be solved.

First, delete any files in /staging that are no longer needed. This will reduce the time needed to copy files. Additionally, regenerating data is much faster than copying data. For example, if you have a large custom Python or R installation under /staging, simply re-install it under /scratch rather than copying.

You can then use a `cp` command to copy files from /staging to /scratch. For example:

```
cp -av /staging/ttrojan/* /scratch/ttrojan/
```

This example will copy all files and directories, recursively, in your /staging directory into your /scratch directory. To copy specific directories or files, simply replace the wildcard * with the specific directory or file names.

## SFTP: Secure File Transfer Protocol

SFTP is a command-line-based transfer tool that is the equivalent of a GUI-based SFTP client. Like SSH, SFTP requires an initial login and authentication, and your session will remain open until you exit or are disconnected. You will remain connected to HPC and able to upload(put) and download(get) files without further login or authentication.

From your computer, log in to hpc-transfer.usc.edu and authenticate via Duo:

```
sftp ttrojan@hpc-transfer.usc.edu
```

If it is your first time logging in you will be asked 'Are you sure you want to continue connecting (yes/no)?'. Answer 'yes'. You will see the following once you are connected:

```
Connected to hpc-transfer.usc.edu.
sftp> 
```

The `help` command will display the available SFTP commands.

```
sftp> help
Available commands:
<lots of options>
```

Use commands like `pwd`, `ls`, and `cd`, and their local equivalents `lpwd`, `lls`, and `lcd`, to navigate to the source and destination directories for file transfer.

### Navigate locally

```
sftp> lpwd
Local working directory: /home1/tommy
sftp> lcd myimages
sftp> lls
myplot1.jpg	myplot2.jpg
```

### Navigate remotely

```
sftp> pwd
Remote working directory: /home1/ttrojan
sftp> cd /scratch/tt/ttrojan/images
```

### Upload file from local to remote computer

```
sftp> put myplot1.jpg myplot.jpg
Uploading myplot1.jpg to /scratch/tt/ttrojan/myplot.jpg
myplot1.jpg                                 100%   10KB   2.4MB/s   00:00    
```

## SCP: Secure Copy Protocol

`scp` is another convenient way to transfer a single file or directory. The following description assumes that `scp` is being used to transfer files between your local computer and HPC.

Note: Unlike SFTP, login and authentication are requested for each use of the command. If you plan to do multiple transfers it is recommended that you use SFTP — either the command-line version or a GUI-based client, which will keep your session open so that multiple authentications are not required.

You can copy a file(s) from a local directory (/source/path) to a remote directory (/destination/path), and vice-versa:

```
scp </source/path> <hostname:/destination/path>

scp <hostname:/source/path> </destination/path>
```

For example:

```
scp myplot1.jpg ttrojan@hpc-transfer.usc.edu:/scratch/ttrojan

scp myplot1.jpg myplot2.jpg ttrojan@hpc-transfer.usc.edu:/scratch/ttrojan

scp *.jpg ttrojan@hpc-transfer.usc.edu:/scratch/ttrojan
```

Similar to the copy (`cp`) command, you can copy whole directories using the `-r` option. The source directory will be copied (created) too:

```
scp -r ./myplots ttrojan@hpc-transfer.usc.edu:/scratch/ttrojan

scp -r ttrojan@hpc-transfer.usc.edu:/scratch/ttrojan/myplots .
```

## Rsync: Remote Synchronization

Rsync is a fast and versatile transfer utility for synchronizing files and directories. It is a good choice for manually transferring large amounts of data on or to HPC because only new or updated files are copied.

The command synchronizes the directory on the local computer, /source/path, with the directory on the remote computer, /destination/path. If /destination/path does not exist, it will be created. Of course, standard Linux file and directory permissions apply.

A generic `rsync` command is:

```
rsync </source/path> <hostname:/destination/path>
```

Note: Unlike SFTP, login and authentication are requested for each use of the command.

The following example copies a file from a local computer to an HPC directory:

```
rsync -av --progress myplot1.jpg ttrojan@hpc-transfer.usc.edu:/scratch/ttrojan/
```

Log in and authenticate and the file(s) should begin transferring:

```
building file list ... 
1 file to consider
myplot1.jpg
       10079 100%    0.00kB/s    0:00:00 (xfer#1, to-check=0/1)

sent 10208 bytes  received 42 bytes  1205.88 bytes/sec
total size is 10079  speedup is 0.98
```

The `-a` option specifies that it will be an archive (and so preserves permissions and modification times, and recursively transfers directories). The `-v` option turns on verbose mode, and the `--progress` option shows transfer progress. Rsync provides many options; see the manual page (`man rsync`) for details.
