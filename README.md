new-vps-setup
=============


## Initial login and change root password

`$ ssh root@192.1.1.1`

`root@myserver:~# passwd`

## Set timezone

```
root@myserver:~# date
Fri Dec 13 09:09:00 EST 2013
```

Change the timezone if you like your timezones to match

`root@myserver:~# sudo dpkg-reconfigure tzdata`

Perl is used by a few packages, so if you get the following annoying timezone message, change it as well

```
root@myserver:~# perl -e exit
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
```

```
root@myserver:~# sudo locale-gen en_US.UTF-8
Generating locales...
  en_US.UTF-8... done
Generation complete.
```

Now the following command should not contain any perl timezone warnings.

`root@myserver:~# perl -e exit`

## Add user

Its recommended to not use root for day to day operations. Add your first user

```
root@myserver:~# adduser myuser
Adding user `myuser' ...
Adding new group `myuser' (1000) ...
Adding new user `myuser' (1000) with group `myuser' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for myuser
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
root@myserver:~# visudo
```

Find root

```
# User privilege specification
root    ALL=(ALL:ALL) ALL
```
Add the new user under root
```
myuser     ALL=(ALL:ALL) ALL
```

## Log in as the new user

```
root@myserver:~# exit
logout
Connection to 192.1.1.1 closed.
$ ssh myuser@192.1.1.1
myuser@192.1.1.1's password:

myuser@myserver:~$
```

## Add keys

Copy your public key (create a public/private key if you dont have one).

```
$ scp ~/id_rsa.pub myuser@192.1.1.1:~/.
myuser@192.1.1.1's password:

myuser@myserver:~$ mkdir .ssh
myuser@myserver:~$ cat id_rsa.pub >> .ssh/authorized_keys

$ chmod 500 -R .ssh/
$ rm id_rsa.pub
```
Log out and log back in and you shouldn't need a key to log in
```
myuser@myserver:~$ exit
logout
Connection to 192.1.1.1 closed.

$ ssh myuser@192.1.1.1
```

## Configure ssh
```
myuser@myserver:~$ sudo vi /etc/ssh/sshd_config
[sudo] password for myuser:
```
Change the port and disallow root login
```
Port 123
PermitRootLogin no
```
Restart ssh
```
myuser@myserver:~$ sudo service ssh restart
ssh stop/waiting
ssh start/running, process 1552
```
Log back in to see if the settings worked
```
$ ssh root@192.1.1.1
ssh: connect to host 192.1.1.1 port 22: Connection refused

$ ssh -p 123 root@192.1.1.1
root@192.1.1.1's password:
Permission denied, please try again.

$ ssh -p 123 myuser@192.1.1.1
```

## Setup firewall
Add rules for your new ssh port, http and https ports

Your initial iptable should look like this
```
myuser@myserver:~$ sudo iptables -L
[sudo] password for myuser:
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

Add rules
```
myuser@myserver:~$ sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
myuser@myserver:~$ sudo iptables -A INPUT -p tcp --dport 123 -j ACCEPT
myuser@myserver:~$ sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
myuser@myserver:~$ sudo iptables -A INPUT -j DROP
myuser@myserver:~$ sudo iptables -I INPUT 1 -i lo -j ACCEPT
myuser@myserver:~$ sudo iptables -I INPUT 1 -p tcp --dport 443 -j ACCEPT
```

## Update apt-get
```
myuser@myserver:~$ sudo apt-get upgrade
[sudo] password for myuser:
myuser@myserver:~$ sudo apt-get update
```
Install iptables-persistent to save iptables
```
myuser@myserver:~$ sudo apt-get install iptables-persistent
```
reboot myuser to see if iptables have been persisted
```
myuser@myserver:~$ sudo reboot

Broadcast message from myuser@myserver
	(/myuser/pts/0) at 20:58 ...

The system is going down for reboot NOW!
myuser@myserver:~$ Connection to 192.1.1.1 closed by remote host.
Connection to 192.1.1.1 closed.
$ ssh -p 123 myuser@192.1.1.1
myuser@myserver:~$ sudo iptables -L -v
```
You should see your iptable rules

## Install git
I use git pretty heavily and thats what I'll install next
```
myuser@myserver:~$ wget https://git-core.googlecode.com/files/git-1.8.5.tar.gz
myuser@myserver:~$ tar -zxvf git-1.8.5.tar.gz
myuser@myserver:~$ cd git-1.8.5
myuser@myserver:~/git-1.8.5$ sudo make
GIT_VERSION = 1.8.5
    * new build flags
    CC credential-store.o
/bin/sh: 1: cc: not found
make: *** [credential-store.o] Error 127
```
Once you start making, you'll notice dependency errors like above, fix as necessary
```
myuser@myserver:~/git-1.8.5$ sudo apt-get install gcc
myuser@myserver:~/git-1.8.5$ sudo apt-get install libssl-myuser
myuser@myserver:~/git-1.8.5$ sudo apt-get install curl libcurl4-openssl-myuser
myuser@myserver:~/git-1.8.5$ sudo apt-get install libexpat1-myuser
myuser@myserver:~/git-1.8.5$ sudo make
myuser@myserver:~/git-1.8.5$ sudo make install
myuser@myserver:~$ sudo rm -rf git-1.8.5*
```
Add git to path
```
myuser@myserver:~/git-1.8.5$ vi ~/.bashrc
export PATH="$PATH:/home/myuser/bin"
```
Log out and log back in and the following command should work
```
myuser@myserver:~$ git
```

## Install nodejs
I use node.js currently for myuser. NVM is the easiest way to manage node.js version
```
myuser@myserver:~$ git clone https://github.com/creationix/nvm.git ~/.nvm
myuser@myserver:~$ vi ~/.bashrc

source ~/.nvm/nvm.sh
```
Install node.js
```
myuser@myserver:~$ nvm ls-remote
myuser@myserver:~$ nvm install 0.10.23
myuser@myserver:~$ nvm ls

  v0.10.23
current: 	v0.10.23

myuser@myserver:~$ nvm alias default 0.10.23
default -> 0.10.23 (-> v0.10.23)
myuser@myserver:~$ nvm ls

  v0.10.23
current: 	v0.10.23
default -> 0.10.23 (-> v0.10.23)
```
## Add a dialog program useful for apt-get interactions
```
myuser@myserver:~$ sudo apt-get install dialog
```
## Add php support to apache
````
myuser@myserver:~$ sudo apt-get install php5 libapache2-mod-php5
````
## Increase upload size
If you host any websites which use php (wordpress for e.g.), its useful to increase the default file upload size
```
myuser@myserver:~$ sudo vi /etc/php5/apache2/php.ini

upload_max_filesize = 10M
```
It's also useful to enable mod rewrite (e.g. useful for permalinks in wordpres)
```
myuser@myserver:~$ sudo a2enmod rewrite
Enabling module rewrite.
```
## Keep track of ubuntu upgrades
````
myuser@myserver:~$ sudo lsb_release -a
myuser@myserver:~$ sudo apt-get install update-manager-core update-manager
````
Upgrade ubuntu if there are new versions 
````
myuser@myserver:~$ sudo do-release-upgrade
````

## setup redis to accept connections from particular ip address
### configure redis to listen on all network interfaces
````
vi /etc/redis/redis.conf
bind 0.0.0.0 (it was initially pointing to 127.0.0.1)
sudo service redis-server restart

sudo iptables -I INPUT 1 -p tcp --dport xxxx -s x.x.x.x -j ACCEPT
sudo service iptables-persistent save

````

## Give nodejs access to port 80 if needed
````
sudo apt-get install libcap2-bin
sudo setcap cap_net_bind_service=+ep /usr/bin/nodejs
````

## Add another redis instance if needed

````
sudo cp /etc/init.d/redis-server /etc/init.d/redis-tweet
sudo cp /etc/redis/redis.conf /etc/redis/redis-tweet.conf 
````
Change log and redis pid file so that it matches. Change dump file and append only file if needed

Add it to startup script
````
sudo update-rc.d redis-tweet defaults
sudo service redis-tweet restart
````


And now you are all set up. That was quick.
