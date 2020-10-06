
## Rclone
`rclone` is a Linux commandline based utility for managing files in cloud storage. `rclone`allows users to sync files from a local drive or location to a cloud storage like Google Drive, Dropbox, OneDrive, etc. This is useful for backing up, downloading, and synchronizing files on Linux to more easily accessible cloud solutions.

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
#### Google Drive
* To set up a Google Drive connection, `name` the remote something informative, i.e., "google-drive". 
* After choosing a name, the next prompt should be a long numbered menu with different storage solutions. Look for Google Drive, number 13. Enter "13" as `Storage` to continue. <br>
* The next prompts are Google Application Client ID `client_id` and Google Application Client Secret `client_secret`. Leave both prompts blank by pressing "Enter" to accept the default values. Creating your own client ID avoids usage of rclone's client ID which all users of rclone by default share, possibly creating slow performance. If interested in creating a personal client ID, more information can be found [here](https://rclone.org/drive/#making-your-own-client-id).
* When choosing the extent of what files rclone can access in Google Drive, there are several options: <br>

Choose "1" as `scope` for access of all files. 
* `root_folder_id` can be left blank unless you want to specify differently from the default root folder. <br>
* Leave `service_account_file` blank to use interactive login. <br>
* Enter 'n' for `Edit advanced config? (y\n)` <br>
* Since Discovery does not have access to a web browser, remote setup must be used instead of auto configuration. Enter 'n' when prompted `Use auto config?`.<br>
  * A link will be generated that can be copied and pasted into your local web browser to give permission to rclone to access your selected Google account's Drive. <br>
  * Click "Allow" and copy the generated code. <br>
* Paste the code copied from your local web browser into the Discovery session's terminal prompt `Enter verification code`.<br>
* Enter 'n' for `Configure this as a team drive?` if the setup is for personal use or does not require a team drive. <br>
* If all the information was entered correctly, finish setup by entering 'y'. Use the 'e' and 'd' options if the setup was not satisfactory. <br>
Afterwards, the main config menu should now look differently and have more options for your newly created remote.

### Backing up Files
The `rclone copy` command creates copies of files between a source and destination, for example, from the Discovery cluster to a Google Drive. `copy` is useful as it skips over files that already exist in the destination. It is very similar to `rsync`.
The command takes the form of:
```
rclone copy source:sourcepath dest:destpath
```
**Optional:**
* Using the flag `--update` checks that skipped files in the remote destination have a newer modified time than the file being transferred. This ensures the newest version of files is available in the cloud.
* `--verbose` gives information about every file being transferred. This can create a lot of screen output but may be helpful to spot problems. 
* Set connection timeout and transfer idle timeout with `--contimeout <time>` and `--timeout <time>` respectively.
* `--retries <number>` specifies the maximum number of errors that can occur or the entire transfer will start over.
* `--transfers <number>` specifies the number of files that are copied in parallel.
* `--no-traverse` flag prevents rclone from transversing the directory when copying files. If the remote destination is very large, this can save a lot of time. However if there are many files that have no changes/won't need copying, do not use this flag.
* Using filters like `--max-age <time>` pr `--max-size <size>` will make the copying process more efficient and avoid copying transversing unwanted files.
The entire list of options can be found [here] (https://rclone.org/docs/).

An example command:
```
rclone copy --update --contimeout 60s --timeout 300s --retries 3 --transfers 30  "/scratch/<username>/files" "google-drive:ProjectDocs"
```
To check if the files have transferred properly, you can manually look or use `rclone ls remote:path`.
Additionally, `rclone copy` can be used within a regularly executed bash script to emulate a scheduled "backup" to your cloud service.

### Synchronizing Files



 
