---
layout: post
title: Linux Privilege escalation
date: "2020-02-01"
aliases:
    - /linux-priv-esc
---

If you do all the HackTheBox, Vulnhub etc VM you will understand the feeling of getting a reverse shell on the machine but we know that you're far from home. Finding the right vector for escalating your privileges can be a pain in the ass. I'm going to share some of the methods I completely depends upon for finding those vulnerable vector that helps to escalate privilege on Linux system.

## Enumeration scripts

__What I use?__

There are lots of enumeration script that people use and they can be very helpful but can also be overwhelming at the same time. Example, [LinEnum](https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh) I think this must be the most used enumeration script among CTF players that is because it shows all the necessary information about the system like network information, running process, binaries with capabilities etc.

I personally don't like to use `LinEnum` because of the following reason:

1) The output is overwhelming which is can be very difficult to go through or maybe you can say I am just lazy to go through 😜.

2) It's too plain i.e the best thing would have been to colorize some important things.

Those are the reason I use [jalesc](https://github.com/itsKindred/jalesc/), this is very similar to LinEnum but the main point of using this was because it highlights all the things that might be considered `important` like if there is some unusally SUID or if the kernel is too old that would be highlighted in the output which would be easy to checkout. And for personal use I've edited out the script to only those things that are very important to me.

__How does enumeration script helps__

Enum scripts just gives you all the information at once so say if there was some binary which is set SUID you can obviously use it to escalate privileges.
So Enum scripts are the definitely the way to start your enumeration once on the system.

You can run a `python -m http.server` on your machine and then `wget` your enumeration script into the `/tmp` folder, chmod it and run it.

## Searching GTFObins

Sometimes you find some suids or some sudo rights for specific binaries which can be used to escalate privileges easily for that you can directly search those binaries on [gtfobins](https://gtfobins.github.io/). If you like to search it from within your terminal you can use a tool called [gtfo](github.com/mzfr/gtfo). Whatever you use searching binaries on `gtfobins` is very important to easy your escalation process.

### Sudo rights

From past few months even before running `enumeration scripts` I check for `sudo` rights of the user I am currently logged in and this is the must. To check those run

```bash
sudo -l
```

The output may look like

```bash
User mzfr may run the following commands on mzfr:
    (root) NOPASSWD: /bin/nmap
```

This mean that current user `mzfr` can run `nmap` as root without password.
In lot of machines `sudo` right is the way to escalate and mostly in machines that have `horizontal` and `vertical` privilege escalation.

__What is horizontal and vertical privilege escalation?__

When you escalate from one user to another. Example, say if a machine have a user `mzfr` and you got shell as `www-data` so when you escalate privileges from `www-data` to `mzfr` that is __horizontal privilege escalation__ and  when you escalate privilege from `mzfr` to `root` thats __vertical privilege escalation__

### SUIDs

These are the files that have special kind of power that power is that whenever we run those files they will be executed as `root`. These files are mostly reported by `enumeration` scripts but if you want to separately search them then you can do that by running:

```bash
find / -perm -u=s -type f 2>/dev/null
```

## Using pspy

Sometime when you don't find anything with enumeration script or as SUID you have to snoop on process running in the background that is when [pspy](https://github.com/DominicBreuker/pspy/) comes into play.

You can just transfer the binary to the machine and then run it just by running:

```bash
$ ./pspy
```
and then wait for sometime to see if it capture some running process. It's mostly useful to see if there is a cronjob running or if there is some kind of custom script setup.

## Using Find command

Lots of time it's very helpful to use this command because it can help you find files or directory to which a current user have access to.

Example: Say you get a shell as `www-data` now after looking for suids, sudo rights and cron jobs you didn't find anything then best would be to run any of the following command:

```bash
find / -type f -user www-data 2>/dev/null
```

and/or

```bash
find / -type d -user www-data 2>/dev/null
```


These commands just find any file or directory which gives read/write access to user `www-data` and redirects all the errors to `/dev/null`.

This is helpful because you might encounter scenarios where the author of the machine wants you to read some file containing SSH keys or passwords or some other information needed to move ahead.

***

These 3-4 steps will help you like 90% of the times obviously there are special occasions when escalating privileges can either be super hard either because the method is too CTFy (like running strings command on an image) or is way too realistic like docker sharing file/folder with root access.

***

Thanks for reading, Feedback is always appreciated.

You can follow me on [@0xmzfr](https://twitter.com/0xmzfr).



