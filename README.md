# Born2beRoot-42

## Table of Contents
1. [Introduction](#introduction)
    - [What is a Virtual Machine?](#Virtual-Machine)
    - [How do VM work?](#V-M-work)
    - [What is LVM?](#What-is-LVM?)
    - [What is AppArmor?](#What-is-AppArmor?)
    - [What is the difference between Apt and Aptitute?](#Apt-and-Aptitute)
    - [How to use SSH?](#How-to-use-SSH?)
    - [How to implement UFW with SSH?](#UFW-with-SSH)
    - [What is cron and what is wall?](#what-is-cron)
2. [Installation](#installation)
3. [*sudo*](#sudo)
    - [Step 1: Installing *sudo*](#step-1-installing-sudo)
    - [Step 2: Adding User to *sudo* Group](#step-2-adding-user-to-sudo-group)
    - [Step 3: Running *root*-Privileged Commands](#step-3-running-root-privileged-commands)
    - [Step 4: Configuring *sudo*](#step-4-configuring-sudo)
4. [SSH](#ssh)
    - [Step 1: Installing & Configuring SSH](#step-1-installing--configuring-ssh)
    - [Step 2: Installing & Configuring UFW](#step-2-installing--configuring-ufw)
    - [Step 3: Connecting to Server via SSH](#step-3-connecting-to-server-via-ssh)
5. [User Management](#user-management)
    - [Step 1: Setting Up a Strong Password Policy](#step-1-setting-up-a-strong-password-policy)
       - [Password Age](#password-age)
       - [Password Strength](#password-strength)
    - [Step 2: Creating a New User](#step-2-creating-a-new-user)
    - [Step 3: Creating a New Group](#step-3-creating-a-new-group)
6. [*cron*](#cron)
    - [Setting Up a *cron* Job](#setting-up-a-cron-job)
7. [Monitoring](#monitoring)
8. [Checklist](#checklist)
    - [Project overview](#project-overview)
    - [Simple setup](#simple-setup)
    - [User](#user)
    - [Hostname and partitions](#hostname-and-partitions)
    - [SUDO](#sudo)
    - [UFW](#ufw)
    - [SSH](#ssh)
    - [Script monitoring](#script-monitoring)
9. [Finish](#finish)

## Introduction
You will create your first machine in VirtualBox (or UTM if you can’t use VirtualBox) under specific instructions. Then, at the end of this project, you will be able to set up your own operating system while implementing strict rules.

### <a name="Virtual-Machine">What is a Virtual Machine?</a>

  <a>
    <img src="https://upload.wikimedia.org/wikipedia/commons/d/d5/Virtualbox_logo.png?20150209215936" width="100" height="100">
  </a>

A virtual machine is a **software capable of installing an Operating System within itself, making the OS think that it is hosted on a real computer**. With virtual machines we can create virtual devices that will behave in the same way as physical devices, using their own CPU, memory, network interface and storage. This is possible because **the virtual machine is hosted on a physical device**, which is the one that provides the hardware resources to the VM. The software program that creates virtual machines is **the hypervisor**. The hypervisor is responsible for isolating the VM resources from the system hardware and making the necessary implementations so that the VM can use these resources.<br>
The devices that provide the hardware resources are called **host machines or hosts**. The different virtual machines that can be assigned to a host are called **guests or guest machines**. The hypervisor uses a part of the host machine's CPU, storage, etc., and distributes them among the different VMs.<br>
<br>
There can be multiple virtual machines on the same host and each of these will be isolated from the rest of the system. Thanks to this, we can run different operating systems on our machine. For each virtual machine, we can run a different operating system distribution. Each of these operating systems will behave as if they were hosted on a physical device, so we will have the same experience when using an OS on a physical machine and on a virtual machine.

### <a name="V-M-work">How do VM work?</a>

  <a>
    <img src="https://assets.podomatic.net/ts/86/5f/63/ranand12/640x640_12681970.jpg?1520713977" width="100" height="100">
  </a>

Virtualization allow us share a system with multiple virtual environments. The hypervisor manages the hardware system and separate the physical resources from the virtual environments. **The resources are managed followitn the needs, from the host to the guests.** When an user from a VM do a task that requires additional resources from the physical environment, the hypervisor manages the request so that the guest OS could access the resources of the physical environment.<br>
Once we know how they work, it is a good idea to see all the advantages we get from using virtual machines:
<ul>
 <li>Different guest machines hosted on our computer <b>can run different operating systems</b>, so we will have different OS working on the same machine.</li>
   <li>They provide an environment in which <b>to safely test unstable programs</b> to see if they will affect the system or not.</li>
   <li>We get <b>better use of shared resources.</b></li>
   <li>We <b>reduce costs</b> by reducing physical architecture.</li>
   <li>They are <b>easy to implement</b> because they provide mechanisms to clone a virtual machine to another physical device.</li>
</ul>

### <a name="What-is-LVM?">What is LVM?</a>

  <a href="">
    <img src="https://www.howtogeek.com/wp-content/uploads/2011/02/banner-1.png?width=1198&trim=1,1&bg-color=000&pad=1,1" width="200" height="100">
  </a>

**LVM (Logical Volume Manager)** is an **abstraction layer between a storage device and a file system**. We get many advantages from using LVM, but the main advantage is that we have much more flexibility when it comes to managing partitions. Suppose we create four partitions on our storage disk. If for any reason we need to expand the storage of the first three partitions, we will not be able to because there is no space available next to them. In case we want to extend the last partition, we will always have the limit imposed by the disk. In other words, we will not be able to manipulate partitions in a friendly way. Thanks to LVM, all these problems are solved.<br>
By using LVM, **we can expand the storage of any partition** (now known as a logical volume) whenever we want without worrying about the contiguous space available on each logical volume. We can do this with available storage located on different physical disks (which we cannot do with traditional partitions). We can also move different logical volumes between physical devices. Of course, **services and processes will work the same way they always have**. But to understand all this, we have to know:
<ul>
 <li><b>Physical Volume (PV):</b> physical storage device. It can be a hard disk, an SD card, a floppy disk, etc. This device provides us with storage available to use.</li>
   <li><b>Volume Group (VG):</b> to use the space provided by a PV, it must be allocated in a volume group. It is like a virtual storage disk that will be used by logical volumes. VGs can grow over time by adding new VPs.</li>
   <li><b>Logical volume (LV):</b> these devices will be the ones we will use to create file systems, swaps, virtual machines, etc. If the VG is the storage disk, the LV are the partitions that are made on this disk.</li>
</ul>

### <a name="What-is-AppArmor?">What is AppArmor?</a>

  <a href="https://linuxhint.com/debian_apparmor_tutorial/">
    <img src="https://res.cloudinary.com/canonical/image/fetch/f_auto,q_auto,fl_sanitize,c_fill,w_720/https://ubuntu.com/wp-content/uploads/e956/AppArmor.png" width="180" height="106">
  </a>

AppArmor provides **Mandatory Access Control (MAC) security**. In fact, **AppAmor allows the system administrator to restrict the actions that processes can perform**. For example, if an installed application can take photos by accessing the camera application, but the administrator denies this privilege, the application will not be able to access the camera application. If a vulnerability occurs (some of the restricted tasks are performed), AppArmor blocks the application so that the damage does not spread to the rest of the system.<br>
In AppArmor, **processes are restricted by profiles**. Profiles can work in complain-mode and in enforce-mode. In enforce mode, AppArmor prohibits applications from performing restricted tasks. In complain-mode, AppArmor allows applications to do these tasks, but creates a registry entry to display the complaint.
  
### <a name="Apt-and-Aptitute">What is the difference between Apt and Aptitute?</a>

  <a href="https://www.tecmint.com/difference-between-apt-and-aptitude/">
    <img src="https://www.fosslinux.com/wp-content/uploads/2020/10/APT-vs.-APTITUDE.png" width="250" height="150">
  </a>

In Debian-based OS distributions, **the default package manager we can use is dpkg**. This tool allows us to install, remove and manage programs on our operating system. However, in most cases, these programs come with a list of dependencies that must be installed for the main program to function properly. One option is to manually install these dependencies. However, **APT (Advanced Package Tool)**, which is a tool that uses dpkg, **can be used to install all the necessary dependencies when installing a program**. So now we can install a useful program with a single command.<br>
APT can work with different back-ends and fron-ends to make use of its services. One of them is **apt-get**, which **allows us to install and remove packages**. Along with apt-get, there are also many tools like apt-cache to manage programs. In this case, **apt-get and apt-cache are used by apt**. Thanks to apt we can install .deb programs easily and without worrying about dependencies. But in case we want to use a graphical interface, we will have to use aptitude. **Aptitude also does better control of dependencies**, allowing the user to choose between different dependencies when installing a program.

### <a name="How-to-use-SSH?">How to use SSH?</a>

  <a href="https://www.youtube.com/watch?v=HTlqTogeXoQ&list=PLjQTJGGLMkUq3p_PCtPaqiUVOvlnkIhiC&index=3">
    <img src="https://iconarchive.com/download/i89873/alecive/flatwoken/Apps-Terminal-Ssh.ico" width="100" height="100">
  </a>
  
SSH or **Secure Shell** is a **remote administration protocol that allows users to control and modify their servers** over the Internet thanks to an authentication mechanism. Provides a mechanism to authenticate a user remotely, transfer data from the client to the host, and return a response to the request made by the client.<br>
SSH was created as an alternative to Telnet, which does not encrypt the information that is sent. **SSH uses encryption techniques** to ensure that all client-to-host and host-to-client communications are done in encrypted form. One of the advantages of SSH is that a user using Linux or MacOS can use SSH on their server to communicate with it remotely through their computer's terminal. Once authenticated, that user will be able to use the terminal to work on the server.<br><br>
The command used to connect to a server with ssh is:
```
ssh username@localhost -p 4242
```
There are three different techniques that SSH uses to encrypt:
<ul>
 <li><b>Symmetric encryption:</b> a method that uses the same secret key for both encryption and decryption of a message, for both the client and the host. Anyone who knows the password can access the message that has been transmitted.</li>
 <li><b>Asymmetric encryption:</b> uses two separate keys for encryption and decryption. These are known as the public key and the private key. Together, they form the public-private key pair.</li>
 <li><b>Hashing:</b> another form of cryptography used by SSH. Hash functions are made in a way that they don't need to be decrypted. If a client has the correct input, they can create a cryptographic hash and SSH will check if both hashes are the same.</li>
</ul>

### <a name="UFW-with-SSH">How to implement UFW with SSH</a>
 
  <a href="https://www.youtube.com/watch?v=HTlqTogeXoQ&list=PLjQTJGGLMkUq3p_PCtPaqiUVOvlnkIhiC&index=3">
    <img src="https://ninjasecurity.co.in/home/waf.png" width="400" height="100">
  </a>

**UFW (Uncomplicated Firewall)** is a software application responsible for ensuring that the system administrator can **manage iptables in a simple way**. Since it is very difficult to work with iptables, UFW provides us with an interface to modify the firewall of our device **(netfilter)** without compromising security. Once we have UFW installed, we can choose which ports we want to allow connections, and which ports we want to close. This will also be very useful with SSH, greatly improving all security related to communications between devices. 

### <a name="what-is-cron">What is cron and what is wall <img src="https://cdn2.iconfinder.com/data/icons/symbol-duo-common-9/32/command_prompt-warning-512.png" width="18" height="18"> ?</a>

  <a href="https://compbio.cornell.edu/about/resources/linux-cron-and-crontab/">
    <img src="https://stevenmortimer.com/blog/automating-r-scripts-with-cron/cronjob.png" width="450" height="200" >
</a>

Once we know a little more about how to build a server inside a Virtual Machine (remember that you also have to look in other pages apart from this README), we will see two commands that will be very helpful in case of being system administrators. These commands are:
- **Cron:** Linux task manager that allows us to execute commands at a certain time. We can automate some tasks just by telling cron what command we want to run at a specific time. For example, if we want to restart our server every day at 4:00 am, instead of having to wake up at that time, cron will do it for us.
- **Wall:** command used by the root user to send a message to all users currently connected to the server. If the system administrator wants to alert about a major server change that could cause users to log out, the root user could alert them with wall. 

## Installation
At the time of writing, the latest stable version of [Debian](https://www.debian.org/) is *Debian 10 Buster*. Watch *bonus* installation walkthrough *(no audio)* [here](https://www.youtube.com/watch?v=OQEdjt38ZJA&t=1s).

## *sudo*

### Step 1: Installing *sudo* 

Check which user is using VM
```
whoami
```
Login as *root* or switch.
```
su -
```
Install *sudo*.
```
apt install sudo
```
Verify whether *sudo* was successfully installed.
```
dpkg -l | grep sudo
```

### Step 2: Adding User to *sudo* Group
Add user to *sudo* group.
```
adduser <username> sudo
```
>Alternative.
```
usermod -aG sudo <username>
```
Verify whether user was successfully added to *sudo* group.
```
getent group sudo
```
`reboot` for changes to take effect, then log in and verify *sudopowers*.
```
reboot
```
```
sudo -v
```

### Step 3: Running *root*-Privileged Commands
```
apt update
```

### Step 4: Configuring *sudo*
>Install *vim*.
```
apt install vim
```
Configure *sudo* .
```
vim /etc/sudoers.d/<filename>
```
###
To limit authentication using *sudo* to 3 attempts *(defaults to 3 anyway)* in the event of an incorrect password, add below line to the file. For wrong password warning message. If there is no “/var/log/sudo” folder, create the *sudo* folder inside of “/var/log”. Each inputs & outputs has to be saved in the /var/log/sudo/sudo.log. Require *TTY*.
```
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
Defaults        passwd_tries=3
Defaults        badpass_message="<custom-error-message>"
Defaults	logfile="/var/log/sudo/sudo.log"
Defaults	log_input,log_output
Defaults        requiretty
```
>(Why use tty? If some non-root code is exploited (a PHP script, for example), the requiretty option means that the exploit code won't be able to directly upgrade its privileges by running sudo.)

## SSH

### Step 1: Installing & Configuring SSH
Install *openssh-server*.
```
apt install openssh-server
```
Verify whether *openssh-server* was successfully installed.
```
dpkg -l | grep ssh
```
Configure SSH.
```
vim /etc/ssh/sshd_config
```
To set up SSH using Port 4242, replace below line:

>13 #Port 22

with:

>13 Port 4242

To disable SSH login as *root* irregardless of authentication mechanism, replace below line

>32 #PermitRootLogin prohibit-password

with:

>32 PermitRootLogin no

Check SSH status.
```
sudo service ssh status
```
>Alternatively.
```
sudo systemctl status ssh
```
Start and stop the SSH Server
```
sudo service ssh start
sudo service ssh stop
```
>Alternatively.
```
sudo systemctl enable ssh
sudo systemctl disable ssh
```

### Step 2: Installing & Configuring UFW
Install *ufw*.
```
sudo apt install ufw
```
Verify whether *ufw* was successfully installed.
```
dpkg -l | grep ufw
```
Enable Firewall.
```
sudo ufw enable
```
Configure the rules.
```
sudo ufw allow ssh
```
Allow incoming connections using Port 4242.
```
sudo ufw allow 4242
```
Check UFW status.
```
sudo ufw status
```

### Step 3: Connecting to Server via SSH
SSH into your VM using Port 4242.
```
ssh <username>@<ip-address> -p 4242
```
Terminate SSH session at any time.
```
logout
```
>Alternatively, terminate SSH session.
```
exit
```

## User Management

### Step 1: Setting Up a Strong Password Policy

#### Password Age
Configure password age policy.
```
sudo vim /etc/login.defs
```
To set password to expire every 30 days, replace below line

>160 PASS_MAX_DAYS   99999

with:

>160 PASS_MAX_DAYS   30

To set minimum number of days between password changes to 2 days, replace below line

>161 PASS_MIN_DAYS   0

with:

>161 PASS_MIN_DAYS   2

To send user a warning message 7 days *(defaults to 7 anyway)* before password expiry, keep below line as is.

>162 PASS_WARN_AGE   7

#### Password Strength
Secondly, to set up policies in relation to password strength, install the *libpam-pwquality* package.
```
sudo apt install libpam-pwquality
```
Verify whether *libpam-pwquality* was successfully installed.
```
dpkg -l | grep libpam-pwquality
```
Configure password strength policy, specifically the below line:
```
sudo vim /etc/pam.d/common-password
```
>25 password        requisite                       pam_pwquality.so retry=3

To set password minimum length to 10 characters, add below option to the above line.

>minlen=10

To require password to contain at least an uppercase character and a numeric character:

>ucredit=-1 dcredit=-1

To set a maximum of 3 consecutive identical characters:

>maxrepeat=3

To reject the password if it contains `<username>` in some form:

>reject_username

To set the number of changes required in the new password from the old password to 7:

>difok=7

To implement the same policy on *root*:

>enforce_for_root

Finally, it should look like the below:
```
password        requisite                       pam_pwquality.so retry=3 minlen=10 ucredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
```

### Step 2: Creating a New User
Create new user.
```
sudo adduser <username>
```
Verify whether user was successfully created.
```
getent passwd <username>
```
Verify newly-created user's password expiry information.
```
sudo chage -l <username>
```

### Step 3: Creating a New Group
Create new *user42* group.
```
sudo addgroup user42
```
Add user to *user42* group.
```
sudo adduser <username> user42
```
>Alternatively.
```
sudo usermod -aG user42 <username>
```
Verify whether user was successfully added to *user42* group.
```
getent group user42
```

## *cron*

### Setting Up a *cron* Job
Configure *cron* as *root*.
```
sudo crontab -u root -e
```
To schedule a shell script to run every 10 minutes, replace below line

>23 # m h  dom mon dow   command

with:

>23 */10 * * * * sh /path/to/script

Check *root*'s scheduled *cron* jobs.
```
sudo crontab -u root -l
```

## Monitoring

You have to create a simple script called `monitoring.sh` It must be developed in bash.
At server startup, the script will display some information (listed below) on all terminals every 10 minutes (take a look at wall). The banner is optional. No error must be visible.
Your script must always be able to display the following information:<br/>
• The architecture of your operating system and its kernel version.<br/>
• The number of physical processors.<br/>
• The number of virtual processors.<br/>
• The current available RAM on your server and its utilization rate as a percentage.<br/> 
• The current available memory on your server and its utilization rate as a percentage.<br/>
• The current utilization rate of your processors as a percentage.<br/>
• The date and time of the last reboot.<br/>
• Whether LVM is active or not.<br/>
• The number of active connections.<br/>
• The number of users using the server.<br/>
• The IPv4 address of your server and its MAC (Media Access Control) address.<br/>
• The number of commands executed with the sudo program.

## Checklist

### Project overview
|  Nº  | Question | Coommand |
| :--: | :---------------------------: | :---------: |
| 1 | How a virtual machine works. | [VM](#VM-work) |
| 2 | Their choice of operating system. | head -n 2 /etc/os-release |
| 3 | The basic differences between CentOS and Debian. | [Differences](#https://www.educba.com/centos-vs-debian/) |
| 4 | The purpose of virtual machines. | [VM](#Virtual-Machine) |
| 5 | Debian: the difference between aptitude and apt, and what APPArmor is. | [APT](#https://www.tecmint.com/difference-between-apt-and-aptitude/) and [APPArmor](#What-is-AppArmor) |

### Simple setup
A password will be requested before attempting to connect to this machine. This user must not be root.
|  Nº  | Question |  Coommand |
| :--: | :-----------------------------------------------------------: | :---------------------------: |
| 1 | Check that the UFW service is started. | sudo ufw status |
| 2 | Check that the SSH service is started. | sudo systemctl status ssh |
| 3 | Check that the chosen operating system is Debian or CentOS. | cat /etc/os-release |

### User
The subject requests that a *user* with the *login of the student* being evaluated is present on the virtual machine.
Check that it has been added and that it belongs to the *sudo* and *user42* groups.
|  Nº  | Question |  Coommand and Links |
| :--: | :-----------------------------------------------------------: | :---------------------------: |
| 1 | First, create a new user. | [Step 2](#step-2-creating-a-new-user) |
| 2 | Assign it a password of your choice. | *password* with new rules |
| 3 | Normally there should be one or two modified files. | [Step 1](#step-1-setting-up-a-strong-password-policy) |
| 4 | Create a group named *evaluating* in front of you and assign it to this user. | [Step 3](#step-3-creating-a-new-group) |
| 5 | Check that this user belongs to the *evaluating* group. | getent group evaluating |

### Hostname and partitions
|  Nº  | Question |  Coommand |
| :--: | :-----------------------------------------------------------: | :---------------------------: |
| 1 | The hostname is `login42`. | hostnamectl |
| 2 | Modify this hostname by replacing the login with yours, then restart the machine. | hostnamectl set-hostname <new_hostname> or sudo vim /etc/hostname |
| 3 | Restore the machine to the original hostname. | repeat 2. again |
| 4 | How to view the partitions for this virtual machine. | lsblk |
| 4 | How LVM works and what it is all about. | [LVM](#What-is-LVM?) |

### SUDO
|  Nº  | Question |  Coommand |
| :--: | :-----------------------------------------------------------: | :---------------------------: |
| 1 | Check that the `sudo` program is properly installed on the virtual machine. | dpkg -l | grep sudo |
| 2 | Assigning your new user to the `sudo` group. | [Step 2](#step-2-adding-user-to-sudo-group) |
| 3 | Explain the value and operation of sudo using examples of their choice. | [Step 4](#step-4-configuring-sudo) |
| 4 | Verify that the `/var/log/sudo/` folder exists and has at least one file. | cd /var/log/sudo/ |
| 5 | Check the contents of the files in this folder, You should see a history of the commands used with sudo. | vim sudo.log |
| 6 | Try to run a command via sudo. See if the file(s) in the `/var/log/sudo/` folder have been updated. | vim sudo.log |

### UFW
|  Nº  | Question |  Coommand |
| :--: | :-----------------------------------------------------------: | :---------------------------: |
| 1 | Check that the `UFW` program is properly installed on the `VM`. | dpkg -l | grep ufw |
| 2 | Explain basically what `UFW` is and the value of using it. | [UFW](#UFW-with-SSH) |
| 3 | List the active rules in `UFW` - port 4242. | sudo ufw status verbose |
| 4 | Add a new rule to open port 8080. | Settings in VirtualBoxVM and allow |
| 5 | Delete this new rule. | sudo ufw delete `string number` |

### SSH
|  Nº  | Question |  Coommand |
| :--: | :-----------------------------------------------------------: | :---------------------------: |
| 1 | Check that the `SSH` service is properly installed on the virtual machine. | dpkg -l | grep ssh |
| 2 | Explain basically what SSH is and the value of using it. | [SSH](#ssh)|
| 3 | Verify that the `SSH` service only uses port 4242. | [Port](#step-1-installing--configuring-ssh) |
| 4 | Use `SSH` in order to log in with the newly created user. You can use a key or a simple password. | ssh username@localhost -p 4242 |
| 5 | Make sure that you cannot use `SSH` with the `root` user. | ssh root@localhost -p 4242 |

### Script monitoring
|  Nº  | Question |  Coommand |
| :--: | :-----------------------------------------------------------: | :---------------------------: |
| 1 | How their script works by showing you the code. | [Bash](#monitoring) |
| 2 | What `cron` is. | [Cron](#what-is-cron) |
| 3 | Set up their script so that it runs every 10 minutes from when the server starts. | [Crontab](#cron) |
| 4 | Once the correct functioning of the script has been verified, ensure that this script runs every 1m. | Change 10 to 1 |
| 5 | You can run whatever you want to make sure the script runs with dynamic values correctly. | Comment line 23 |
| 6 | Make the script stop running when the server has started up, but without modifying the script itself. | vim monitoring.sh |
| 7 | Restart the server one last time. | reboot |
| 8 | At startup, it will be necessary to check that the script still exists in the same place, that its rights have remained unchanged, and that it has not been modified. | I make changes in crontab and didn't modified monitoring.sh |

### Finish
Turn off your VM. Crate `signature.txt` file and put there your VM key by generating it with below line
```
shasum born2beroot.vdi
```
After push only `signature.txt` and don't turn it on till evaluation starts.
