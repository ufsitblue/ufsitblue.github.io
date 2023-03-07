# Sandbox vid 10: Creating user accounts

### What does the "useradd" command actually do?

e.g. Adding a new user, "bob", with `useradd`:

1. It makes a new **group** called bob
2. It makes a new **user** called bob
3. It creates its home directory at `/home/bob`
4. It copies some standard files for new users from `/etc/skel`, which include stuff like .bashrc (shell config)
5. It prompts a password (with the `passwd` command)
6. It prompts for stuff like full name, "room number", etc. (this gets stored in the /etc/passwd file, but no one really bothers with it)

- Note that, by default, a new user does not have the authority to execute commands as sudo (see vid 13 for more on this)





# Sandox vid 11: Exploring sudoers and removing users

### "ls -l"

- This `ls` option will list the permissions of a file. Should normally pair this with the `-a` option to show hidden files as well (files with a . at the start of their name)

Example of an `ls -la` entry that I ran in my home directory:
`-rwxr-xr-x  br br    126 KB Wed Dec 15 10:28:22 2021 my_sick_program.o*`
    - Note the first 10 characters of this command (`.rwxr-xr-x`)
    - The first `-` indicates that this listing is _not_ a directory (it would show `d` otherwise)
    - The next three characters (`rwx`) indicate that it is readable, writeable, and executable by the file owner -- `br` (my user)
    - The three characters after (`r-x`) indicate that it is readable, _not_ writeable, and executable by the group associated with the file
    - The last three characters (`r-x`) indicate the same thing as before, but these apply to "everyone else" (that is not the owner or group for the file)

- You can modify permissions with `chmod`, e.g. `chmod +x my_sick_program.o`

- Pop quiz: What does it mean if I do `chmod 777 my_sick_program.o`


### `lsattr`

- This lists the "extended" permissions of a file. This can be useful if you can't modify/delete a file even though it seems like you should be able to (chattr +i)
    - [Read more here](https://wiki.archlinux.org/title/File_permissions_and_attributes#File_attributes)





# Sandox vid 12: Exploring sudoers and removing users

### /etc/:

- "Like the control panel for Linux" (Control center for Mac)


### /etc/sudoers

- This file is mega important
    - If you are root user, or can act as root (with sudo), or are in the root group, then you can read this file (`-r--r-----`)

- The `visudo` command:
    - Use this. Don't `sudo vim` or `sudo nano` the sudoers file.

Example: `%admin ALL=(ALL) NOPASSWD: ALL`
- "Users in the admin group, on any host, may run any command as any user, without a password"
- `%admins` indicates that the following rule will apply to the "admin" **group**. If the `%` was absent, then it would refer to a user called "admin"
- The first `ALL` refers to "all hosts"
- The second `(ALL)` refers to the target users
- The third and final `ALL` refers to "all commands"

### userdel

`sudo usrdel -r <user>` to both remove a user and delete their home directory





# Sandbox vid 13: Groups

### groups and id commands

- `groups` and `id` both show you what groups you belong to, but `id` shows you the associated user and group IDs as well
    - Pay attention to what users and IDs get listed on your machine, for example, that may have been created by default.


### usermod command

Example: `sudo usermod -a -G sudo bob`
- Adds the user `bob` to the existing group, `sudo`
- `-a` indicates that we're "appending" aka not creating a brand new group
- `-G` indicates the group name that we're appending to


### /etc/group

- Basically /etc/passwd but for groups (more info about /etc/passwd in vid 13)
    - Most of the ones on here will have been pre-made by your Linux ditribution for stuff like services

- Worth paying attention to this file in competitions
    - What groups have been created?
    - What users have been sneakily added to certain groups such as sudo?





# Sandbox vid 13: Passwords and shadow hashes


### /etc/passwd: User accounts

Example: `bob:x:1001:1001:,,,:/home/bob:/bin/bash`
- First value is just the username (bob)
- Second value (`x`) indicates that the user has a password, and its hash is stored in `/etc/shadow` -- if you remove this x, the user is set to have no password
- Third and fourth value (`1001`) indicate the associated user ID (uid) and group ID (gid)
- Fifth value (`,,,`) stores the full name, room number, etc. that you typically get prompted for when you run `useradd`
- Sixth value: User's home directory
- Seventh value (`/bin/bash`): The command/shell associated with the user. Should be `/sbin/nologin` for accounts that should not be logged in with (e.g. service accounts, or the `nobody` user)


## /etc/shadow: User password hashes

Example: `vivek:$1$fnfffc$pGteyHdicpGOfffXX4ow#5:13064:0:99999:7:::`
- 1: Username
- 2: Password hash
    - `$1$` indicates that it is an MD5 hash (deprecated, insecure)
    - `$6$` indicates that SHA-512 is being used
- 3: Last password change (in UNIX time)
- 4: Minimum number of days required between password changes
- 5: Maximum: Max days that the password is valid (forced to change after that)
- 6: Inactive: Number of days after password expires that account is disabled
- 7: Expire: Expiration date of password, expressed as days since Jan 1, 1970

Remember that empty values look like `::` -- e.g., if the second field has no value, then that means the user has no password.
