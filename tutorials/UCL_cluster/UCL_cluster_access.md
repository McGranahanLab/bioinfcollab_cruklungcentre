## An algorith to apply for access to UCL HPC & get data storage space: ##
1. Get your computer science (CS) account here: https://tsg.cs.ucl.ac.uk/apply-for-an-account/. Put Nicholas McGranahan as “Dept. of Computer Science Sponsor”. It will take a couple of days to get an answer. Also, you should have a cell phone number so they could text you the password.
2. After you have your computer science account, apply for a computer science cluster account: https://hpc.cs.ucl.ac.uk/account-form/. Do not fill in “Machine IP” or “Software requirements” fields.
3. In meantime, you can read carefully a user guide to the HPC here: https://hpc.cs.ucl.ac.uk/ (**username: hpc, password: comic**)
4. Once you have your cluster account, you can **apply for storage space for your project**, if known and needed, here: https://hpc.cs.ucl.ac.uk/storage-form/. Usually project storage space comes in batches of 10Tb, so if you'd like more, it's wise to ask for several project spaces, each size of 10Tb. Once you get an email that your inquiry is satisfied, you will be able to find folder with your project at `/SAN/colcc/`. **Be sure to write down your project folder name**, because sometimes then you do `ls -l /SAN/colcc/` you won't be able to see all project folders.

Before accessing to the server, please read this: https://hpc.cs.ucl.ac.uk/quickstart/ and this https://hpc.cs.ucl.ac.uk/full-guide/. We usually use `gamble` for our computations. However, if `gamble` isn't available, try other servers.

To test your connection to the server, type in your terminal:

```
ssh <your user name>@tails.cs.ucl.ac.uk # use your CS account password
ssh <your user name>@gamble.cs.ucl.ac.uk # use your CS account password
```
Congrats! You’re on the cluster.

Picture below illustrates why you need to use ssh twice:
![UCL server structure](https://github.com/McGranahanLab/Wiki/blob/master/UCL_cluster_structure.png?raw=true)

### Establishing shorcuts to access the cluster (aka ssh jump) ###
Code block above shows you a usual way to access cluster: though two ssh's. However, then you work a lot on cluster it might not be so convinient: it requires a lot of typing, passwords, and in addition you can't mount gamble to your computer to make acceess to files easy. Ssh jump can take care of this, and you'll be able to log in directly to gamble by just typing `ssh gamble`. You only need to complete the procedure below once.

**Step 1** : add keys to tails to your computer

1. Open your terminal
2. Type `cd .ssh`. If you get an error that folder doesn't exist, do `mkdir .ssh` and then `cd .ssh`
3. Type `ssh-keygen`. Enter tails as name and leave the password blank.
4. Type `nano config`. The config file will be openned, insert in it:

```
Host tails 
  User <your_user_name>
  IdentityFile ~/.ssh/tails
  HostName tails.cs.ucl.ac.uk
  ForwardAgent yes
Host gamble
  User <your_user_name>
  IdentityFile ~/.ssh/gamble
  HostName gamble.cs.ucl.ac.uk
  ProxyJump tails:22
```

Don't forget to replace <your_user_name> with your actual user name. Save the file and exit according to the commands displayed at the bottom of the screen

5. Type `ssh-copy-id -i tails.pub tails`. This will copy the key you've just created to tails
6.  Now you can just `ssh tails` without password to get to tails cluster.

**Step 2** : add keys to gamble on tails

1. Open your terminal
2. Type `ssh tails`. Now you're on tails (UCL cluster).
3. Type `cd .ssh`. If you get an error that folder doesn't exist, do `mkdir .ssh` and then `cd .ssh`
4. Type `ssh-keygen`. Enter gamble as name and leave the password blank.
5. Type `nano config`. The config file will be openned, insert in it:

```
Host gamble 
User <your_user_name>
IdentityFile ~/.ssh/gamble
HostName gamble.cs.ucl.ac.uk

```
Don't forget to replace <your_user_name> with your actual user name. Save the file and exit according to the commands displayed at the bottom of the screen

6. Type `ssh-copy-id -i gamble.pub gamble` . This will copy the key you've just created to gamble
7. Type `exit` to exit tails and return back to your computer.

**Step 3**: add keys from gamble to your computer

1. Open your terminal
2. Type `ssh-keygen`. Enter gamble as name and leave the password blank.
3. Type `ssh-copy-id -i gamble.pub gamble`. This will copy the key you've just created to gamble

That’s it! Now you can just do `ssh gamble` and get on gamble!

:warning: If you're planning to use nextflow on the UCL cluster, you will need to login on a special node - `askey`.  It is accessible though gamble, i.e. you first need to `ssh gamble`, and then `ssh <your_user_name>@askey.cs.ucl.ac.uk` :warning:

### Mounting cluster on your computer (only if ssh jump is established) ###

Mounting will allow you to open folders from gamble cluster as usual on your computer and copy to/from cluster files just by dragging and dropping them.

1. Install sshfs: https://command-not-found.com/sshfs. Help for Mac users:
```
brew uninstall osxfuse macfuse sshfs
brew install --cask macfuse
brew install gromgit/fuse/sshfs-mac
brew link --overwrite sshfs-mac
```
2. Type in the terminal to create folder where home folder from gamble will be mounted `mkdir -p ~/gamble-home`
3. Perform mounting of gamble home folder: 

```
sshfs gamble: ~/gamble-home -oauto_cache,follow_symlinks -ovolname=GambleHome,defer_permissions,noappledouble,local
```

4. Type in the terminal to create folder where mcgranahanlab folder from gamble will be mounted `mkdir -p ~/gamble-mcgranahanlab`

5. Perform mounting of mcgranahanlab folder: 

```
sshfs gamble:/SAN/mcgranahanlab/ ~/gamble-mcgranahanlab -oauto_cache,follow_symlinks -ovolname=Gamble-McGranahanlab,defer_permissions,noappledouble,local
```
6. To unmount, type `umount ~/gamble-mcgranahanlab ~/gamble-home`

This is manual for mounting on Mac, for other systems, check out: https://support.nesi.org.nz/hc/en-gb/articles/360000621135-Can-I-use-SSHFS-to-mount-the-cluster-filesystem-on-my-local-machine-


### Main folders on the cluster ###
Our main folder is accessible via : `cd /SAN/mcgranahanlab/general/`. Once you’re in, please create a folder for yourself: `mkdir your_name`. Please be aware that _we should only store scripts in that folder, and no data_. To store data, request storage space for your project, see point 4 of section "An algorith to apply for access to UCL HPC & get data storage space".

**Please apply for the storage space for your project to store your data / results**

### Avaible software ###
You can list all the available software here:  `ll /share/apps/`, and genomics-specific like this: `ll /share/apps/genomics/`. In order to use it, you need to export path to executable of the software. Code snippet below shows how to export path to samtools and bedtools.

```
export PATH=/share/apps/genomics/bedtools-2.25.0/bin/:${PATH};
export PATH=/share/apps/genomics/samtools-1.9/bin/:${PATH}; 
```

To use software which comes in a shape of `jar` files, for example picard, do following:
```
PICARD_JAR=/share/apps/genomics/picard-2.20.3/bin/picard.jar
java -jar $PICARD_JAR
```

**In case the software is not available on the cluster**, there are 2 options: 1) create singularity container with that software (reccomended) 2) send ticket to `cluster-support@cs.ucl.ac.uk`

### Avaible resourses ###
Unfortunately, we don't have shared folders with reference genomes, databases, etc.

### Interactive sessions ###
It's handy to test code in intective session. During interactive session you get directly on computing node and test command execution as they would be in your job script. The code below requests an interactive session with 8Gb of RAM for 1h:
```
qrsh -l h_vmem=8G,tmem=8G,h_rt=1:0:0
```
It may take some time to get an interactive session, this time also depends on amout of currently running jobs and RAM you requested. Then you get interactive session, beginning string of your bash terminal should change from `[your_user_name@gamble]$` to for example `[your_user_name@moe-609-7 ]$`, where `moe-609-7` is a node name. In the beginning of your interactive session you will be at the root of the server and not in the directory you called `qrsh` from, so you'll  need to `cd /SAN/colcc/your_project_folder` to do your work.

### Get help ###
Ed Martin (`e.martin@cs.ucl.ac.uk`) is our contact for the cluster questions, he’s very responsive. But please don’t abuse this link: first ask people in the lab.

