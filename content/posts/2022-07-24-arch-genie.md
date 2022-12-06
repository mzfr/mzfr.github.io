---
title: Genie in a bottle
summary: Decided to shift from Ubuntu to Arch WSL and found an amazing way to run systemd.
date: "2022-08-01"
tags:
    - genie
    - arch
    - wsl
---

I've been using the desktop for work for the past 2 years now. Back when I built my PC I decided to give [WSL](https://docs.microsoft.com/en-us/windows/wsl/install) a try. I had a feeling that it might not work out but with the right tooling and quite a lot of time to configure, I was able to make it work. To be able to follow all the available guides online and the official documentation I decided to stick with WSL ubuntu. Before this when I was doing everything on my old laptop [I was using Manjaro](https://knowyourmeme.com/memes/btw-i-use-arch).

Initially, I thought even though I'm going with Ubuntu, I don't see myself sticking with Windows+WSL setup and will surely shift to full Linux on the desktop. But WSL worked really well. And being on Windows meant to be able to install & play games from steam/epic games/Ubisoft/Xbox without any hassle. This setup worked so well that I even gave my [OSCP](https://www.offensive-security.com/pwk-oscp/) exam on a Kali Linux WSL.

For people curious about how I did it, When I started doing labs I was using [Win-Kex](https://www.kali.org/docs/wsl/win-kex/) which is like an official application that provides Kali desktop experience on WSL. But towards the end of the labs, I started having [issues](https://github.com/microsoft/WSL/discussions/6675) with Win-Kex. For the exam, I ended up using [xfce4](https://gitlab.xfce.org/xfce) which actually worked better than the Win-kex.

Anyways for a long time I've been using WSL Ubuntu for development and Kali WSL for security-related work. But lately, I decided to shift to Arch Linux because I wasn't able to tell people, I use Arch 😂. Jokes aside the main reason I'd say I did the transition was that I really like that Arch is a rolling distro it gets updates really quickly and the majority of packages are up to date. The issue with Debian/Ubuntu is that `apt` packages installed via `apt install XYZ` are usually of older versions. Now I know that there are additional ways to get those newer versions but sometimes it's irritating to do that. Also another issue I felt with ubuntu was that some packages were only available with [SNAP](https://snapcraft.io/), I didn't like that.

## Installation and configuration

The installation part is just super easy. Follow [these steps](https://github.com/yuk7/ArchWSL#zip) and boom you have ArchWSL working. I use Windows [terminal](https://github.com/microsoft/terminal) and wanted to run Arch directly from there so for that I added the following in my terminal `settings.json`

```json
{
    "commandline": "E:\\path\\to\\Arch.exe",
    "font":
    {
        "face": "FiraCode Nerd Font Mono",
        "size": 16
    },
    "guid": "{1eaed209-42ed-4393-bd4c-a50c696aab1f}",
    "hidden": false,
    "name": "Arch"
}
```

And then from the GUI settings of the terminal set arch as the default. Now when I start the terminal it opens directly in the home directory of my Arch Linux Subsystem.

## Working with genie

Now the intention behind this post wasn't to tell everyone that I use arch(Pun intended 😁). The main reason I decided to write this post was because of [genie](https://github.com/arkane-systems/genie).

After installing the new arch, I started to install all the basic stuff I need on daily basis like python3, [ptpython](https://github.com/prompt-toolkit/ptpython), [nvim](https://github.com/prompt-toolkit/ptpython), etc. Once that was done I started to set up my dev environment, to be specific I was making sure that the application I was working on will be able to run properly. So first I made a python virtual environment and installed all the dependencies. That project uses PostgreSQL and Redis so I installed PostgreSQL via [yay](https://github.com/Jguer/yay) and ran the following command:

```bash
sudo service postgresql start
```
But to my surprise I got the following error:

```bash
sudo: service: command not found
```

For a second I thought that I might need to install a package/library to be able to run this command. I'm aware of the fact that systemctl doesn't run by default on WSL(if you wanna read about this a bit, [see this](https://superuser.com/a/1719430)). That is why on my old ubuntu setup I was using `service` command there and as far as I remember I didn't install anything extra to be able to run that command. But now on the new Arch setup, service isn't even a command.

After some googling, I found out [this comment](https://github.com/yuk7/ArchWSL/issues/20#issuecomment-545245760)

> I created systemctl alternative package and documented it.

> If you use WSL2, you can use genie.

> that's all we can do.

> This is a common problem with container Linux and WSL.
I had never heard of genie before so I decided to give it a try.

Genie is basically a way of running containerized systemd inside WSL, that container is known as a "bottle". With genie, you can run and enable any service via systemd.

For my setup, all I had to do was install genie via yay, [create the config file](https://github.com/arkane-systems/genie#configuration-file) and once that was done, run `genie -s` to get inside a bottle. Once you are inside the bottle you can run systemctl commands, normally like `sudo sytstemctl start postgresql.service`.


Once I had excess to `systemctl` I enabled all the services that are required by the application. That way all I have to do is run `genie -s` and it automatically starts all the enabled services.

I only faced two easily solvable issues with genie.

* One of the issues that I faced with genie is that sometimes it times out. Basically when I run `genie -s` it throws a warning and timeouts after 240 seconds. But the solution was also simple, just re-run the same command and that works.

* Another issue was exactly what is mentioned [here](https://github.com/arkane-systems/genie/wiki/Command-%22code%22-not-found-for-VScode-remote-in-bottle%3F-Here%27s-a-solution).

I've been using WSL Arch with the genie for systemd for almost 4 weeks now and haven't had any other issues.
