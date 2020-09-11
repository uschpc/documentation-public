## Migrating your data from rcf-proj to the new project file system

The CARC's new project file system, /project, is now ready to migrate your data from /home/rcf-proj to.

### How should I perform the data migration?

The hpc-transfer node has dedicated 40 GB/s links to both the old /home/rcf-proj and the new /project file systems, so data migration in and out is much faster than using other login nodes.

Because the data transfer could take a long time, we'll use `screen` in combination with `rsync` to do the migration so that your progress doesn't get interrupted if your SSH connection gets dropped. 

#### Step 1: Log in to the transfer node

From your computer, log in to `hpc-transfer.usc.edu` and authenticate via Duo:

```
ssh ttrojan@hpc-transfer.usc.edu
```

Substitute `ttrojan` with your username, which is your USC NetID (the first part of your USC email address).

> Note: If you get an SSH error about "remote host identification has changed" when attempting to connect, the solution is to clear your "known_hosts" file that is referenced in the error message. Open the "known_hosts" file in your .ssh directory under your /home (note the "." in front of the directory name) and manually delete the line beginning with "hpc-transfer.usc.edu" and then save. Try the login command again, confirm the new authenticity of the host, and the error should no longer occur.

#### Step 2: Start a Screen session

Enter `screen` at the command line to start a `screen` session. This will allow the migration to continue if your SSH connection is dropped or you want to log out.

Information on `screen`: https://linuxize.com/post/how-to-use-linux-screen/

#### Step 3: Identify data to be migrated

Identify and document unwanted or re-generatable/re-downloadable data from your /home/rcf-proj directory, especially those folders with large amounts of small files (e.g., custom Python/R/Anaconda installations). These small files will slow down the migration substantially. Specific files and directories can be excluded from the transfer.

Also check if your new directory under the new project file system, /project, on Discovery is created already. A storage allocaiton for each project has been created and you should see:

```
/project/<PI_name>_xxx
```

here `PI_name` is the usernme of the project owner, `xxx` is a 2~3 digit number. If you belong to the project `<PI_name>_xxx`, you will be able to create your 
own sub dir under it

```
mkdir /project/<PI_name>_xxx/username
```

#### Step 4: Start migration using rsync

Enter an `rsync` command that looks something like the following:

```
rsync -avP /home/rcf-proj/xxx/yyy/ /project/aaa/bbb/
```

`/home/rcf-proj` will be your source and `/project` will be your destination. Be sure to substitute your correct `rcf-proj` and `/project` directory paths, and pay attention to the trailing `/` in the paths.

This will start the transfer and display its progress.

Information on `rsync`: https://linuxize.com/post/how-to-use-rsync-for-local-and-remote-data-transfer-and-synchronization/

If there are specific files or directories that you do not need to transfer, add the `--exclude` option and specify the files or directories. For example, if you
want to exclude `/home/rcf-proj/xxx/yyy/Anaconda` and `/home/rcf-proj/xxx/yyy/R`
```
rsync -avP --exclude 'Anaconda' --exclude 'R' /home/rcf-proj/xxx/yyy/ /project/aaa/bbb/
```

Information on the exclude option: https://linuxize.com/post/how-to-exclude-files-and-directories-with-rsync/

#### Step 5: Detach from the Screen session

Press the keys `ctrl-a d` to detach the Screen session, which will let the `rsync` job run in the background. You can then continue other work or log out and the transfer will continue on hpc-transfer.

#### Step 6: Check on the migration process

To reattach your Screen session and check the migration progress, enter `screen -r`.

#### Step 7: Migration is done - close Screen session

Once your data migration is complete, you can close the Screen session by entering `exit` from within the session. Your data has now been migrated to the new project file system (/project).
