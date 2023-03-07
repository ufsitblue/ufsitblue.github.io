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

* "Like the control panel for Linux" (Control center for Mac)


### /etc/sudoers

* This file is mega important
    - If you are root user, or can act as root (with sudo), or are in the root group, then you can read this file (`-r--r-----`)

* The `visudo` command:
    - Use this. Don't `sudo vim` or `sudo nano` the sudoers file.

Example: `%admin ALL=(ALL) NOPASSWD: ALL`
- "Users in the admin group, on any host, may run any command as any user, without a password"
- `%admins` indicates that the following rule will apply to the "admin" **group**. If the `%` was absent, then it would refer to a user called "admin"
- The first `ALL` refers to "all hosts"
- The second `(ALL)` refers to the target users
- The third and final `ALL` refers to "all commands"

### userdel

`sudo usrdel -r <user>` to both remove a user and delete their home directory





# Sandbox vid 13:
