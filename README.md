new-vps-setup
=============

## Why

I've setup a few vps' this year but every time I set it up, I can't remember all the steps. So I'm putting it down here for the next time

## Initial login and change root password

`$ ssh root@192.3.2.188`

`root@us:~# passwd`

## Set timezone

```
root@us:~# date
Fri Dec 13 09:09:00 EST 2013
```

Change the timezone if you like your timezones to match

`root@us:~# sudo dpkg-reconfigure tzdata`

Perl is used by a few packages, so if you get the following annoying timezone message, change it as well

```
root@us:~# perl -e exit
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
```

```
root@us:~# sudo locale-gen en_US.UTF-8
Generating locales...
  en_US.UTF-8... done
Generation complete.
```

Now the following command should not contain any perl timezone warnings.

`root@us:~# perl -e exit`

## Add user

Its recommended to not use root for day to day operations. Add your first user

```
root@us:~# adduser dev
Adding user `dev' ...
Adding new group `dev' (1000) ...
Adding new user `dev' (1000) with group `dev' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for dev
Enter the new value, or press ENTER for the default
	Full Name []: 
	Room Number []:
	Work Phone []:
	Home Phone []:
	Other []:
Is the information correct? [Y/n] y
```

Give sudo permissions

```
root@us:~# visudo
```

Find root

```
# User privilege specification
root    ALL=(ALL:ALL) ALL
```
Add the new user under root
```
dev     ALL=(ALL:ALL) ALL
```

## Log in as the new user

```
root@us:~# exit
logout
Connection to 192.3.2.188 closed.
anoopsmac:NewsCube anoopkulkarni$ ssh dev@192.3.2.188
dev@192.3.2.188's password:

dev@us:~$
```

## Add keys

Copy your public key (create a public/private key if you dont have one).

```
$ scp ~/id_rsa.pub dev@192.3.2.188:~/.
dev@192.3.2.188's password:

dev@us:~$ mkdir .ssh
dev@us:~$ cat id_rsa.pub >> .ssh/authorized_keys

$ chmod 500 -R .ssh/
$ rm id_rsa.pub
```
Log out and log back in and you shouldn't need a key to log in
```
dev@us:~$ exit
logout
Connection to 192.3.2.188 closed.

$ ssh dev@192.3.2.188
```

## Configure ssh
```
dev@us:~$ sudo vi /etc/ssh/sshd_config
[sudo] password for dev:
```
Change the port and disallow root login
```
Port 123
PermitRootLogin no
```
Restart ssh
```
dev@us:~$ sudo service ssh restart
ssh stop/waiting
ssh start/running, process 1552
```
Log back in to see if the settings worked
```
$ ssh root@192.3.2.188
ssh: connect to host 192.3.2.188 port 22: Connection refused

$ ssh -p 123 root@192.3.2.188
root@192.3.2.188's password:
Permission denied, please try again.

$ ssh -p 123 dev@192.3.2.188
```

## Setup firewall
Your initial iptable should look like this
```
dev@us:~$ sudo iptables -L
[sudo] password for dev:
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

Add rules
```
dev@us:~$ sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
dev@us:~$ sudo iptables -A INPUT -p tcp --dport 123 -j ACCEPT
dev@us:~$ sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
dev@us:~$ sudo iptables -A INPUT -j DROP
dev@us:~$ sudo iptables -I INPUT 1 -i lo -j ACCEPT
```

## Update apt-get
```
dev@us:~$ sudo apt-get upgrade
[sudo] password for dev:
dev@us:~$ sudo apt-get update
```
Install iptables-persistent to save iptables
```
dev@us:~$ sudo apt-get install iptables-persistent
```
reboot device to see if iptables have been persisted
```
dev@us:~$ sudo reboot

Broadcast message from dev@us
	(/dev/pts/0) at 20:58 ...

The system is going down for reboot NOW!
dev@us:~$ Connection to 192.3.2.188 closed by remote host.
Connection to 192.3.2.188 closed.
$ ssh -p 123 dev@192.3.2.188
dev@us:~$ sudo iptables -L -v
```
You should see your iptable rules

## Install git
I use git pretty heavily and thats what I'll install next
```
dev@us:~$ wget https://git-core.googlecode.com/files/git-1.8.5.tar.gz
dev@us:~$ tar -zxvf git-1.8.5.tar.gz
dev@us:~$ cd git-1.8.5
dev@us:~/git-1.8.5$ sudo make
GIT_VERSION = 1.8.5
    * new build flags
    CC credential-store.o
/bin/sh: 1: cc: not found
make: *** [credential-store.o] Error 127
```
Once you start making, you'll notice dependency errors like above, fix as necessary
```
dev@us:~/git-1.8.5$ sudo apt-get install gcc
dev@us:~/git-1.8.5$ sudo apt-get install libssl-dev
dev@us:~/git-1.8.5$ sudo apt-get install curl libcurl4-openssl-dev
dev@us:~/git-1.8.5$ sudo apt-get install libexpat1-dev
dev@us:~/git-1.8.5$ sudo make
dev@us:~/git-1.8.5$ sudo make install
dev@us:~$ sudo rm -rf git-1.8.5*
```
Add git to path
```
dev@us:~/git-1.8.5$ vi ~/.bashrc
export PATH="$PATH:/home/dev/bin"
```
Log out and log back in and the following command should work
```
dev@us:~$ git
```
