# How to Use Blackbox Encryption

This is a guide to using Blackbox encryption, a public-key encryption system for sensitive data. We will use Blackbox to encrypt PII data.

This guide is based on Bill Rawlinson's [Windows guide to Blackbox.](http://code.rawlinson.us/2016/04/storing-secrets-in-git.html) It assumes you:
1. Know how to modify your system path environment variable. If you don't, you can find guides for [Windows 10](https://www.architectryan.com/2018/03/17/add-to-the-path-on-windows-10/) ([one more](https://helpdeskgeek.com/windows-10/add-windows-path-environment-variable/)), [Windows 8](https://www.h3xed.com/windows/how-to-add-to-and-edit-windows-path-variable), and others on the internet.
2. Have basic familiarity with [Git](https://help.github.com/en/articles/set-up-git), and in particular have installed [Git for Windows.](https://git-scm.com/download/win) This is a prerequisite to any Git-based workflow. Make sure to enable MinGW (Git Bash) while setting it up.
3. Have installed [GnuWin32](https://sourceforge.net/projects/getgnuwin32/files/), a set of development tools that Blackbox uses. If not, installation instructions are in Rawlinson's guide.

## Step 0: Set up MinGW

Blackbox, like most other git-based encryption systems, was developed for Unix-based operating systems and thus is not native to Windows. In order to operate Blackbox on Windows, you must work through a Unix-based command prompt. The two best options are [Cygwin](https://cygwin.com) and MinGW, or Git Bash. Since MinGW is built into Git for Windows, that's what this tutorial will focus on. (If you have Cygwin already, all of what follows still applies.)

Importantly, this means you **cannot** use the Windows command prompt. It simply isn't built to support Blackbox.

To find MinGW, search "Git Bash" on your computer. If it gives you a command shell, and if that command shell looks something like
```
Karthik Tadepalli@DESKTOP-ABVN4EB MINGW64 ~
$  
```
Then you have found MinGW. If you can't find it by search, look in the directory where you installed Git for Windows. It should contain the file `git-bash.exe`, and running that will take you to MinGW as well.

For the rest of the tutorial, I'll assume you're executing commands from MinGW.

## Step 1: Install Blackbox

The Blackbox repository is [here.](https://github.com/StackExchange/blackbox) Clone this repository with
```
git clone https://github.com/StackExchange/blackbox.git
```

The cloned repository will have a `bin` directory. Add this `bin` directory to your system path environment variable. This will allow you to run Blackbox commands easily from your command line.

## Step 2: Create a GPG key

If you already have a GPG key set up with your email, skip this step.

Install Gnu Privacy Guard (GnuPG) from [here.](https://gpg4win.org/download.html) In the setup, select GPGex as a feature to install. Wherever you install GPG, it will have a `bin` directory. Add this `bin` directory to your system path environment variable.

From MinGW, run
```
gpg --gen-key
```
If this command doesn't work, try any of the alternative commands listed [here.](https://help.github.com/en/articles/generating-a-new-gpg-key) If `gpg` isn't recognized as a valid command, then you have not successfully added `bin` to your path.

In the dialog box, follow the instructions and enter your real name and the email you want the GPG key to be attached to. Enter a password for your GPG key - make it secure!

This GPG key is what you will use to access Blackbox-encrypted files.

# Step 3: Add yourself as an admin

From the root of the repository, run:

```
blackbox_addadmin YOUR_USERNAME
git commit -m 'NEW ADMIN: YOUR_USERNAME' .blackbox/pubring.kbx .blackbox/trustdb.gpg .blackbox/blackbox-admins.txt
```

Push these changes to the remote repo.

Here, `YOUR_USERNAME` is the email that you used to generate a GnuPG key in step 2. If `blackbox_addadmin` is not recognized as a valid command, then you have not successfully added the `blackbox/bin` directory to your path, from step 1.

If you created the repo and you are the first admin, you now have access. Otherwise, if current admins exist, you will gain access to encrypted files when a current admin pulls your commit and indoctrinates you. To do this, they should import the public keychain to their personal keychain. **Note:** the suggested code to do this is outdated on both the repo readme and Rawlinson's guide. The following command works better.
```
gpg --no-default-keyring --keyring .blackbox/pubring.kbx  --export -a | gpg --import
```
Then they should decrypt and reencrypt all files with blackbox:

```
blackbox_update_all_files
```
This command will also commit the changes made to admin permissions. They should finally push these changes to the remote repo, which you can then pull and access all encrypted files.

## Step 4: Use Blackbox

Now you're ready to go with Blackbox.

The important commands are more extensively covered in the [Blackbox readme](https://github.com/StackExchange/blackbox/blob/master/README.md#commands) and in [Rawlinson's guide.](http://code.rawlinson.us/2016/04/storing-secrets-in-git.html) In general, a simple way to access all encrypted files is to run `blackbox_postdeploy`. This will ask you for your GPG password, and decrypt all files. Run whatever code you need to run. When you're finished, run `blackbox_shred_all_files` to delete the unencrypted files and retain the encrypted versions.

Some important precautions for our projects:
1. **Always run blackbox_shred_all_files before committing any change to the repo.** Git LFS tracks the encrypted files, but not the unencrypted files - accidentally committing the unencrypted large files will break our GitHub repo.
2. If you plan to modify one of the data files in place (not recommended, as it hurts reproducibility and risks irreversible mistakes) then you have to decrypt it with `blackbox_edit_start FILENAME`. Once you are done editing, run `blackbox_edit_end FILENAME`. If you don't do this, your edits will be lost upon reencryption.
