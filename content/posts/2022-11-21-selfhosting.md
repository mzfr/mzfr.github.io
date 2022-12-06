---
title: Giving self-hosting a go
summary: I've been lurking around r/selfhosted for quite a while now, and decided to use old raspi to host some simple applications.
date: "2022-12-03"
tags:
    - self-hosting
    - self-hosted
    - raspberry
    - raspi
---

Back in 2021 when I came across [r/selfhosted](https://www.reddit.com/r/selfhosted/) I was really fascinated by the idea of self-hosting. Spending time on that subreddit made me realize that quite a lot of people are self-hosting applications that they use on a daily basis. At that time I had no intention of self-hosting applications for personal use because it didn't seem like something I'd be able to manage. But for the sake of learning and having fun, I decided to try hosting applications like [NextCloud](https://nextcloud.com/), [calibre-web](https://github.com/janeczku/calibre-web), [Bookstack](https://www.bookstackapp.com/), [ghost](https://ghost.org/), [syncthing](https://syncthing.net/). All that using [portainer](https://www.portainer.io/) along with [traefik](https://traefik.io/) as a proxy.

All of the above was running on a Digitalocean droplet. I tried using all of that applications for about a week but then once the excitement of self-hosting wore off I kinda stopped using them, all of them. So I 💥destroyed that droplet💥

Now this year around march/April I was running a few telegram bots for certain things on my [raspberry pi 4](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/) and then decided to seriously give self-hosting a try. By seriously I meant things running things that I would actually use. I ended up trying a lot of things I felt I wanted to self-host but then, in the end, only a few of them felt like something I actually needed.


## Applications

The majority of applications mentioned below run on my Raspi except my [wiki](https://wiki.mzfr.me) and my [blog](https://blog.mzfr.me). I still feel that if you need to host a static site then there isn't anything better than Github pages. Well, actually now that I said it Gitlab pages or even Cloudflare pages are still pretty decent & simplistic alternatives to Github pages. But again, I don't see myself self-hosting any static site(or maybe I might 😅)

If you'd like to try self-host any of the below-mentioned applications then might find my notes(https://wiki.mzfr.me/#self_hosting) helpful.


### Things I tried

Below are the things I ended up trying because the idea of having that application was really nice in my mind but when I actually self-hosted the applications I had this feeling that it ain't for me.

* [BookStack](https://www.bookstackapp.com/): This seemed like a very good way of internally hosting my personal notes. But I actually found the idea of having shelves/books/chapters/pages a bit weird so decided to drop the idea of using it.

* [TT-RSS](https://tt-rss.org/): I was using Feedly when I didn't know about self-hosting but then decided to self-host TT-RSS for myself. And I didn't enjoy the UI at all. Maybe I was spoiled by Feedly but even with the Feedly theme tt-RSS didn't come close to want I think I wanted. Now I am not criticizing the tt-RSS project, it is an amazing project being maintained by amazing people and supported by a great community. But it wasn't something I felt was for me.

* [Calibre Web](https://github.com/janeczku/calibre-web): For some reason I thought I can reduce Goodreads usage by self-hosting calibre web. But it is definitely not a replacement. Also, I realized I wanted a better setup that would fill my kindle library automatically and didn't really want a web reader(which isn't that good).

* [Uptime kuma](https://github.com/louislam/uptime-kuma): This was just for fun purpose. I personally had no need for self-hosting this kind of service. And it actually pretty good, if you are running multiple servers and want to keep a record of their uptime, etc.

### Things I am using

Below are the things I tried and am actually using in day-to-day life.

* [Syncthing](https://syncthing.net/): I needed something to sync files across my devices for note taking app. I plan to write a completely separate blog post for this setup.

* [Vikunja](https://vikunja.io/): I think I was looking for a decent to-do application that wasn't Google keep or Microsoft ToDo. In some discussion on a Reddit thread I came across vikunja and gave it a try. This was an application I felt was good enough for me. Also [DAVx5](https://www.davx5.com/) and [Task](https://github.com/dmfs/opentasks).

* [atuin](https://github.com/ellie/atuin/): This was something I came across recently and immediately felt like something I would use. Also working on multiple servers, and running the same commands again and again felt like something I definitely needed. In the end, I hosted the server and now feel like it is an amazing tool that makes your life easy.

* [umami](https://github.com/umami-software/umami): This is a very simple yet efficient google analytics alternative. I've been using this for a while now and haven't had any issues with it.


***

Other than the above-mentioned stuff I also run major `*arr` applications along with Kodi(I did try plex but didn't like it). And I don't really have any comments on this tool. I mean these are stuff that just works perfectly fine.

## Architecture

Below is an image that tries to explain my "complex"  self-hosting architecture.

![](/images/selfhostarch.jpg)

Also, you might notice the artistic effects that I've tried to add to the above diagram.

## Future plans

I personally want to move away from Google's ecosystem, Why? To be honest, just for the sake of trying and to see if it is actually possible to completely eliminate Google from one life. The biggest thing which is stopping me from doing so is Google photos. I personally haven't found any great alternative to Google photos yet. Another important thing that I'd like to have is a note-sharing app. Even though for personal notes I have a great setup now but sometimes if I have to share a simple list of things or some kind of note/info in which input from both parties might be required, I still find myself using Google keep. 

Other than that something I definitely wanna give a try is self-hosting the VPN as well. Maybe something like a head scale or wire guard but I am also kinda also worried that self-hosting an important part of my infrastructure might just not be good.

***

I would definitely recommend people give self-hosting a try, even if they don't want to actually use those applications. Just running them will teach you a lot about proper networking and managing containers, etc. And also I'd like to thank the [r/selfhosted](https://www.reddit.com/r/selfhosted/) for inspiring me, I hope one day I can be fully self-hosted :)