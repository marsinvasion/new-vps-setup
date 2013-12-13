new-vps-setup
=============

## Why

I've setup a few vps' this year but every time I set it up, I can't remember all the steps. So I'm putting it down here for the next time

## Initial login and change root password

`$ ssh root@192.3.2.188`

`root@us:~# passwd`

## Set timezone

```root@us:~# date
Fri Dec 13 09:09:00 EST 2013```

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
