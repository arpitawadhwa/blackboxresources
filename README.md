# Boosting Immunization Demand, Haryana (BID)
Data and code for the Boosting Immunization Demand (HIMM-BID) project in Haryana.

## Guide to setting up the repository and accessing the data

### Step 1: Git set-up

Get some basic familiarity with [Git](https://help.github.com/en/articles/set-up-git), and install [Git for Windows.](https://git-scm.com/download/win) This is a prerequisite to any Git-based workflow. Make sure to enable MinGW (Git Bash) while setting it up.

The rest of these instructions require you to use the MinGW (Git Bash) command prompt. To find Git Bash/MinGW, search "Git Bash" on your computer. If it gives you a command shell, and if that command shell looks something like
```
Arpita Wadhwa@LAPTOP-42ONSKJH MINGW64 ~
$  
```
Then you have found MinGW. If you can't find it by search, look in the directory where you installed Git for Windows. It should contain the file `git-bash.exe`, and running that will take you to MinGW as well.

### Step 2: Git LFS set-up

[Install Git LFS](https://github.com/git-lfs/git-lfs/wiki/Installation). After downloading + running the Windows installer, remember to open Git Bash and type ```git lfs install ```. The role of Git LFS is explained [here](https://git-lfs.github.com/).

### Step 3: GnuWin32 set-up

Install [GnuWin32](https://sourceforge.net/projects/getgnuwin32/files/), a set of development tools that Blackbox uses. Instructions are in this [guide](http://code.rawlinson.us/2016/04/storing-secrets-in-git.html).

### Step 4: Blackbox set-up

Install Blackbox. The Blackbox repository is [here.](https://github.com/StackExchange/blackbox) Clone this repository with
```
git clone https://github.com/StackExchange/blackbox.git
```
The cloned repository will have a `bin` directory. Add this `bin` directory to your system [path environment variable](https://www.architectryan.com/2018/03/17/add-to-the-path-on-windows-10/). This will allow you to run Blackbox commands easily from your command line.

### Step 5: Create a GPG key

If you already have a GPG key set up with your email, skip this step.

Install Gnu Privacy Guard (GnuPG) from [here.](https://gpg4win.org/download.html) In the setup, select GPGex as a feature to install. Wherever you install GPG, it will have a `bin` directory. Add this `bin` directory to your system path environment variable.

From MinGW, run
```
gpg --gen-key
```
If this command doesn't work, try any of the alternative commands listed [here.](https://help.github.com/en/articles/generating-a-new-gpg-key) If `gpg` isn't recognized as a valid command, then you have not successfully added `bin` to your path.

In the dialog box, follow the instructions and enter your real name and the email you want the GPG key to be attached to. Enter a password for your GPG key - make it secure!

This GPG key is what you will use to access Blackbox-encrypted files.

### Step 6: Clone the haryana-anmol repository
You can do this either through Git Bash or through other GUIs like [SourceTree](https://www.sourcetreeapp.com/)

### Step 7: Add yourself as an admin on Blackbox

First navigate to the root of the repository in Git Bash, like so:

```
cd 'C:\Users\Arpita Wadhwa\Documents\GitHub\haryana-anmol'
```
  
Once you're there you should see the path displayed in yellow followed by the word (master) in blue. Now run:

```
blackbox_addadmin YOUR_USERNAME
git commit -m 'NEW ADMIN: YOUR_USERNAME' .blackbox/pubring.kbx .blackbox/trustdb.gpg .blackbox/blackbox-admins.txt
```

Push these changes to the remote repo.

Here, `YOUR_USERNAME` is the email that you used to generate a GnuPG key in step 5. If `blackbox_addadmin` is not recognized as a valid command, then you have not successfully added the `blackbox/bin` directory to your path, from step 4.

### Step 8: Ask any existing admin to indoctrinate you

You will gain access to encrypted files when a current admin pulls your commit and indoctrinates you. To do this, they should import the public keychain to their personal keychain. They can use the following command for this:

```
gpg --no-default-keyring --keyring .blackbox/pubring.kbx  --export -a | gpg --import
```

Then they should decrypt and reencrypt all files with blackbox:

```
blackbox_update_all_files
```

This command will also commit the changes made to admin permissions. They should finally **push these changes** to the remote repo, which you can then **pull and access all encrypted files.**

## Working on the repository

### Git workflow for standard code edits (no data edits/creations)

1. Pull changes from the remote. I do this inside SourceTree, but you can also write `git pull origin master` in Git Bash

2. Decrypt the data files. Writing `blackbox_postdeploy` in Git Bash terminal will decrypt all the data files prompting you to enter your GPG password.

3. Work on the do-files/scripts/code, saving changes.

4. Once you're done with your current session, shred all the open decrypted files using `blackbox_shred_all_files` in Git Bash. This **gets rid of any changes** you made to those files since the encrypted versions haven't been updated. This step is **VERY IMPORTANT** because not only does it remove unencrypted PII, but it also removes large files that are not tracked by Git LFS (only the encrypted versions of the files are tracked by Git LFS).

5. Stage and commit the changes you made to the scripts I do this using SourceTree, because it's a nice interface that shows you what changes you made and lets you select the files you want to stage and commit, but you can also do this from Git Bash. Make the commit message nice and descriptive, but not too long.

6. Push the changes to the remote. This will get more complicated because we will all be working on the files simultaneously 

### Special case 1 - someone has made and pushed edits simultaneously
In this case, you will not be allowed to push your changes to the master because someone has made changes in the meanwhile. You should first pull from the origin-master and **rebase** your changes on top. This can also be done within SourceTree, which will guide you through any conflicts you have to resolve. The workflow is best described in this [link](https://www.atlassian.com/git/tutorials/comparing-workflows) as the 'Centralized Workflow'.

### Special case 2 - you have created a new PII dataset or file that needs to be pushed to the remote
First ask yourself if you really need to create this dataset, and push it, or whether the recipient can easily recreate it on their end by running the code on the existing raw data. If you conclude that you really need to create and save the file, then follow these steps for each new file:

1. Open Git Bash, cd to the tn-dropouts repository, and check if Git LFS is tracking the location of the file. Type `git lfs track`, and if the file directory is not there, then add it by typing `git lfs track [directory]+"/**"`. For example, if I'm tracking the folder data/raw/new, I would type `git lfs track 'data/raw/new/**'`.

1. Enter `blackbox_register_new_file [filepath+name]`. For example, if I'm encrypting file 'data.xlsx' inside data/raw, I would write `blackbox_register_new_file 'data/raw/data.xlsx'`. This should replace the unencrypted file with an encrypted version and automatically create a commit.

2. Check that this specific file is tracked by Git LFS. Type `git lfs ls-files` and ensure the encrypted file you made is present in the list.

3. Push the changes, making sure to include all the edits to the .blackbox, .gitattributes, and other files in the process as these files contain the information that tell Blackbox and Git LFS what to track.

### Special case 3 - you want to change the location of a previously encrypted file
1. Decrypt all files using `blackbox_postdeploy`.

2. After decryption, deregister the file(s) you want to change the location of by using `blackbox_deregister_file '[file path]'`. Use the file path of the encrypted file i.e. the one that ends with .gpg. This command will remove this file from the blackbox directory and delete the .gpg file, while leaving the original decrypted file(s) (xlsx/csv etc.).

4. Delete the extra blackbox decrypted files that had been created in the first step using `blackbox_shred_all_files`, this should only leave behind the decrypted files which had been deregistered in the previous step.

3. Shift the decrypted target files to the desired new location.

5. Push the changes to the remote repo.

6. If the folder that the files were shifted to is new, and is not already included in git lfs, run git lfs track `git lfs track '[directory]+"/**"'` on the folder.
Run `git lfs track` and `git lfs ls-files` to make sure that the directory and the files added to that directory are being tracked in git lfs. Do this even if the files were shifted to an existing directory.

7. Now run `blackbox_register_new_file '[file path]'` on each of the files that has been shifted.

### Special case 4 - you have made edits to an existing file and want these to be incorporated into the newly encrypted version
Need to look more at the [Blackbox readme](https://github.com/StackExchange/blackbox/blob/master/README.md#commands) and in [Rawlinson's guide.](http://code.rawlinson.us/2016/04/storing-secrets-in-git.html) to understand this properly.

## Working with the data 

1. Folders and Subfolders in the repo correspond to the content. **Code** has dofiles, rscripts, and jupyter notebooks. The naming convention of the files and the scripts is  : lowercase and spaces separated by underscore.

```
sample_file_name
```
**Data** has raw, intermediate, and cleaned datasets. Most datasets include Personally Identifiable information (PII) and have been encrypted using Blackbox. **Shapefiles** has geocodes at the district and PHC level. These shapefiles have been sourced from open-source websites.

2. Every folder has a sub folder named __archive__ which includes scripts or older versions of raw datasets that are no longer relevant. 

## Best practices 

## References and other resources
Most of this guide builds on the excellent work and documentation by Karthik Tadepalli in the [tn-housing-pmay repo](https://github.com/idea-lab-jpsa/tn-housing-pmay) and Mihir Bhaskar in the [tn-dropouts repo](https://github.com/idea-lab-jpsa/tn-dropouts). For further information on the codes and scripts of this repository, please reach out to Arpita Wadhwa at awadhwa@povertyactionlab.org.





