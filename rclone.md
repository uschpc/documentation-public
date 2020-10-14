
## Rclone
`rclone` is a Linux commandline based utility for managing files in cloud storage. `rclone`allows users to sync files from a local drive or location to a cloud storage like Google Drive, Dropbox, OneDrive, etc. This is useful for backing up, downloading, and synchronizing files on Linux to more easily accessible cloud solutions.

### Loading the rclone module
In Discovery, `rclone` is already available as a module. First, to utilize `rclone` use 
```
module load rlone
```
After running this line, all rclone commands will become accessible. <br>
All current rclone commands can be found [here](https://rclone.org/commands/).
More information about the module system can be found [here](https://carc.usc.edu/user-information/user-guides/high-performance-computing/discovery/lmod).

### Creating an rclone Remote Connection
Usage of each cloud service requires the setup of a "remote." To begin the process, use the command `rclone config`. <br>
When first creating a remote, the following script should appear:
```
No remotes found - make a new one 
n) New remote <br>
s) Set configuration password 
q) Quit config 
n/s/q>
```
Enter "n" to continue. <br>
##### Example: Google Drive
* To set up a Google Drive connection, `name` the remote something informative, i.e., "google-drive". 
* After choosing a name, the next prompt should be a long numbered menu with different storage solutions. Look for Google Drive, number 13. Enter "13" as `Storage` to continue. A list of all possible storage providers can be found [here](https://github.com/rclone/rclone).<br>
* The next prompts are Google Application Client ID `client_id` and Google Application Client Secret `client_secret`. Leave both prompts blank by pressing "Enter" to accept the default values. Creating your own client ID avoids usage of rclone's client ID which all users of rclone by default share, possibly creating slow performance. If interested in creating a personal client ID, more information can be found [here](https://rclone.org/drive/#making-your-own-client-id).
* When choosing the extent of what files rclone can access in Google Drive, there are several options: <br>

Choose "1" as `scope` for access of all files. 
* `root_folder_id` can be left blank unless you want to specify differently from the default root folder. <br>
* Leave `service_account_file` blank to use interactive login. <br>
* Enter 'n' for `Edit advanced config? (y\n)` <br>
* Since Discovery does not have access to a web browser, remote setup must be used instead of auto configuration. Enter 'n' when prompted `Use auto config?`.<br>
  * A link will be generated that can be copied (Ctrl+Shift+C) and pasted into your local web browser to give permission to rclone to access your selected Google account's Drive. <br>
  * Click "Allow" and copy the generated code. <br>
* Paste the code (Right click) copied from your local web browser into the Discovery session's terminal prompt `Enter verification code`.<br>
* Enter 'n' for `Configure this as a team drive?` if the setup is for personal use or does not require a team drive. <br>
* If all the information was entered correctly, finish setup by entering 'y'. Use the 'e' and 'd' options if the setup was not satisfactory. <br>
Afterwards, the main config menu should now have more options for your newly created remote. <br>
Enter 'q' to exit the "remote creation" menu. 

### Backing up Files
The `rclone copy` command creates copies of files between a source and destination, for example, from the Discovery cluster to a Google Drive. `copy` is useful as it skips over files that already exist in the destination. It is very similar to `rsync`.
The command takes the form of:
```
rclone copy source:sourcepath dest:destpath
```
**Optional:**
* Using the flag `--update` checks that skipped files in the remote destination have a newer modified time than the file being transferred. This ensures the newest version of files is available in the cloud.
* `--verbose` gives information about every file being transferred. This can create a lot of screen output but may be helpful to spot problems. 
* `--progress` shows progress through percentage completion and total elapsed time. This can be useful for large data transfers. 
* `--no-traverse` flag prevents rclone from transversing the directory when copying files. If the remote destination is very large, this can save a lot of time. However if there are many files that have no changes/won't need copying, do not use this flag.
* Using filters like `--max-age <time>` pr `--max-size <size>` will make the copying process more efficient and avoid copying transversing unwanted files. More details about filtering can be found [here](https://rclone.org/filtering/). <br>

The entire list of options can be found [here](https://rclone.org/docs/).

An example command:
```
rclone copy --update "/scratch/<username>/files" "google-drive:ProjectDocs"
```
* Note: If the destination folder does not exist, `rclone` will create it. <br>
To check if the files have transferred properly, you can manually look online or use `rclone ls remote:path`. There will be no terminal confirmation output of successful file transfer. <br>
Additionally, `rclone copy` can be used within a regularly executed bash script to emulate a scheduled "backup" to your cloud service.

### Synchronizing Files
The difference between copying files and synchronizing files is that copy creates duplicates from a source to a destination, but synchronizing creates a replica of the source at the destination. In other words, if files are deleted from the source, synchronizing the source and destination will delete files from the destination as well. Copying will never delete files in the destination. <br>

The default command is: <br>
```
rclone sync source:path dest:path
```
An example command if wanting to sync the local project folder to a Google Drive:
```
rclone sync "/home1/<user>" "google-drive:ProjectDocs"
```
where "google-drive" is the name of the remote connection and "ProjectDocs" is the name of the folder we wish to contain the copied files. <br>
Additional flags can be added similar to the `rclone copy` command. 

### Navigating the Remote Cloud Storage
`rclone` can be used to check files on the remote connection, very similarly to commands to view local files. 
For instance, to view the files in a folder on a remote, an example command would be:
```
rclone ls google-drive:ProjectDocs
```
To create a new folder that does not exist:
```
rclone mkdir google-drive:ProjectDocs/Test
```
To delete a file in the remote:
```
rclone deletefile google-drive:ProjectDocs/test.txt
```
Further commands can be found [here](https://rclone.org/commands/).



 
