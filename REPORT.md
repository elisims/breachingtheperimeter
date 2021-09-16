# Ubuntu Server Pentetration Test

## Initial Setup
- The VM image was imported into VMWare, and the proper Operating System (Ubuntu version16)was selected. The ubuntu server that was imported needed to be run in “host-only” mode- therefore, using VMWare’s virtual network manager, a host-only connection was createdunder the name “VMnet1”. The network was configured to use the IP address 192.168.141.0.

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/1.jpg)

- Then under the DHCP settings, I set the range of IP’s to be between 192.168.141.128 and192.168.141.132 in order to make finding these addresses much easier during the fingerprintingprocess.

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/2.jpg)

## Nmap recon and fingerprinting

- Once the initial setup of the VM’s and networking was finished, I then moved onto
running nmap to enumerate the open ports and vulnerable points of entry into the
ubuntu server.

- Using the command “ifconfig”, I saw the list of all IPs currently seen on the network.

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/3.jpg)

- The IP address of 192.168.141.129 is listed as the attacking VM’s Ip on the host-only
network. I then used the “ping” command to find which address was assigned to the
ubuntu server.

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/4.jpg)

- The IP address 192.168.141.131 had a ttl (time to live) of 64, which is a signal that this is
an ubuntu system sending those packets. Therefore, I knew I could now use nmap
on this IP address.

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/5.jpg)

- Bingo. There are a few interesting points of entry enumerated by this nmap fingerprintingHowever, the most intriguing one is the port 873 which is rsync. RSync has a few known vulnerabilities associated with it, so that’s the point of entry I will use to gain initial access to the ubuntu server.


## Compromising the machine.

### Using RSync to gain access.

- Using the rsync command “rsync 192.168.141.131::” I enumerated the actions I had open for me to explore. Luckily, the “files” were available to view.

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/6.jpg)

- So, I used the same command but opened the files folder to view what was inside. This is how I gained access to the entire system, as I could view every file within the ubuntu server.

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/7.jpg)

- Knowing I could now explore the entire system, I went into the home folder to see what the users on the ubuntu server kept in their personal files. I started with wturner’s folder.

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/8.jpg)

- Wturner has his password in a text document, so I knew that I could download this file and open it on me after VM to obtain access to the system using the VM itself or through the ssh open port (both of which would be simple to access with these newfound credentials.

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/9.jpg)

- Downloading the password file onto my attacker VM and using the command “cat mypassword.txt” revealed the password for wturner to be “test”.

### Alternate Method (Bypass Ubuntu Login)

- Once I knew the version of Ubuntu and Linux kernel that were being used, I used Google to search for common compromises to these respective versions. Upon further research, I came across a common exploit of Ubuntu which allows any person to bypass login credentials and gain access to the system as a root user. The method entailed using the recovery options at startup to become the root user in and gain access to the host through terminal use. The methodology was simply:

> - Hold “shift” as the VM booted up
> - Select “advanced options for Ubuntu”
> - Then select “recovery mode”
> - From the Recovery Menu, select “drop to root shell prompt”

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/10.jpg)

## Escalating privilege
- Once the VM had been compromised I took some time to explore the different directories and
explored the privileges I had as the root user. On this system, root did not have sudo privilegetherefore I used the command visudo to open the “sudoers” file using the terminal editor “nano”
in order to elevate my privilege to the highest degree. Within the file, I added the lines:
> - Root ALL=(ALL:ALL) ALL
> - Sudo ALL=(ALL:ALL) ALL

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/11.jpg)

These lines elevated my privilege in two ways:
> 1. Root user now had no restrictions for any used commands.

> 2. Adding “sudo” at the beginning of any command (as any user) removed any
restrictions placed on that command.

## Finding vulnerabilities

1. The first vulnerability is the RSync utility being misconfigured on this ubuntu server. Rsync is
a utility for transferring and synchronizing files between two servers. It determines
synchronization by checking file sizes and timestamps. The weak configurations often provide
unauthorized access to sensitive data, and sometimes the means to obtain a shell on the
system. This is exactly how I gained initial access into the ubuntu server, as I used the RSync
open port to traverse the files until I found “wturner”’s not secure password file. In theory, I
could have used the RSync connection to create a new user and elevate the new user’s
privilege to root- However, I went the easier route and edited the “sudoers” file to make
wturner have root privilege- And used wturner to do my bidding instead.

2. The second vulnerability I found was with the misconfigured ubuntu settings at startup.
Another way that I instantly obtained root privilege (without any effort) was using the recovery
exploit to bypass any login credentials. Using this exploit gives the attacker instant root
privilege and the same elevation technique (editing the sudoers file) can be used to elevate
wturner to root privilege.

3. Improper permissions are used with the home directory on this ubuntu server. On most Linux
distributions the default permissions for home folders are 755 which means that any user who
has access to the server can see what is in other user’s home folders. This is precisely what
occurred on this ubuntu server, as the credentials provided by wturner gave me access to all
other user’s folders in the home directory (which all contained some form of sensitive data).

4. Wturner has a script named “exec” within his home folder that uses the Setuid binary to modify
the user's privilege to run the file as root. As an attacker, I can definitely find unexpected ways
to use this file to perform commands on the system as the root user.

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/12.jpg)

5. World-readable and writable files and folders introduce similar issues as loose user home
permissions but throughout the system. The main cause of world readable files is the default
mask used for file creation of either 0022 or 0002. As a result of this configuration weakness,
files that may contain sensitive information will be readable by anyone that has access to the 

system. Files may also be modified by anyone on the system if they are world-writable. This
can lead to an attacker modifying files or scripts to hide forensic evidence or to execute
commands by modifying a script used by Administrators.

6. There is currently an issue with the password policy of the company, as every password I was
able to recover (using various methods) were basic and easy to crack in nature. These
passwords had bases that are on “top 100” lists, making a wordlist/dictionary attack super
effective against them.

7. When running the initial nmap fingerprinting of the network, the first open port displayed was
port 22, SSH. Not only was SSH open and vulnerable, the RSA, ECDSA and ED25519 sshhostkey information were all displayed freely. This is the public key (which by itself is not
necessarily a vulnerability) however, within the server, it made confirming the “id_rsa.pub”
was the hostkey being used- Making the private key located in the folder that much more
valuable. The folder these two keys are located in (/user2/.ssh) was not encrypted and
accessible by wturner without root privilege.

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/13.jpg)
![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/14.jpg)

8. The bash history was available for viewing by anyone with initial access to the ubuntu server.
Typically, the bash_history file keeps track of the user’s last 500 commands. Users often type
usernames and passwords on the command-line as parameters to programs, which then get
saved to this file when they log out. Attackers can abuse this by looking through the file for
potential credentials. This is not only a vulnerability for the potential leaking of passwords, but
is a privacy issue in general.

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/15.jpg)

9. Unprotected credentials were present in two different locations (not including the password
file given from wturner’s unprotected folder). The jimbo.txt file located in /root contained the
credentials of “jimbo:LocalAdmin123!”. Within /home/user2, the credential information
unprotected was within the file named “creds.txt”, and contained the information
“administrator:IDontLikeSharing!”.

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/16.jpg)
![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/17.jpg)

10. Script’s folder contains cron file that is unprotected; therefore, an attacker could modify or
replace this file to perform tasks without the users knowing anything was ever changed. 

![img](https://github.com/elisims/breachingtheperimeter/raw/main/images/18.jpg)
