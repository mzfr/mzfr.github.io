---
layout: post
title: Google Summer of Code 2018 with XBMC foundation
date: 2018-06-27 11:38
---

I got selected as a Google Summer of Code student in XBMC foundation (a.k.a Kodi) for the project [Static code analysis in Kodi's addon-check tool](Static code analysis in Kodi's addon-check tool). Result came out on April 23 09:30 PM IST. I know it was 2 months ago but I couldn't get myself to write about it - you can say I was "busy" ;)

![alt text](/images/Gsoc'18.png)

## What is GSoC ?

Google Summer of Code is a global program focused on bringing more student developers into open source software development. Students work with an open source organization for 3 month programming on any project of their choice during their break from school. More details about GSoC [here](https://summerofcode.withgoogle.com).

## My journey to GSoC

It all started with my elder brother, Shadab Zafar ([@dufferzafar](https://github.com/dufferzafar)). He did his first GSoC in 2014 and after that GSoC became a part of his yearly schedule. He then did it again in 2015 and then in 2016. He also encouraged few of his friends to participate in GSoC. So you can see where I am going with this. He was the one who told me to try to apply for GSoC'18, emphasis on "try" because he used to think I am a nub :)

Under the guidance of my brother and his friends started what we call `mission mzfr GSoC 2018` (thanks to [@ichait](https://github.com/ichait) for the name).

In Janaury, after going through GSoC'17 org list I liked [Snare/Tanner](https://honeynet.org/gsoc2018/ideas#snare) project under The honeynet project. So I started getting familiar with the code-base and since I was a newbie with no résumé or any big project to show, I started contributing to the project. But unfortunately there was someone else working on the same project plus he was a bit more experienced than me. So after consulting with my brother and following his golden words, _"nayi org utha le abhi bhi time hai"_ (find a new org, you stil have time), I again started my hunt for a new org and then I came across this quiet org (no one except the owner of the repo was working on the project), atleast it was quiet till the time I found it, XBMC foundation.

[XMBC foundation](https://kodi.tv/about/xbmc-foundation) is a non-profit organisation that oversees [kodi](https://kodi.tv/), an open-source media player application. After going over their Idea list I really liked `Add code validation and do static code analysis in Kodi's add-on checker` project. The [Add-on checker tool](https://github.com/xbmc/addon-check) had a small code-base and the new implementation was easy to do so it was a perfect project for me.

And then again I started contributing to the project. My first PR was [`Auto comment on issues found in pull request`](https://github.com/xbmc/addon-check/pull/34), even though later I had to remove all this code because of Travis-CI's [Pull Requests and Security Restrictions](https://docs.travis-ci.com/user/pull-requests/#Pull-Requests-and-Security-Restrictions) but the PR fullfiled the purpose - which was to show mentors that I am capable of doing "Stuff". After that I continued contibuting to the tool.

Then came the `prepare a good proposal` phase. Again thanks to @dufferzafar and his super cool friends ([@iChait](http://github.com/ichait) and [@kwikadi](http://github.com/kwikadi)) for helping me in writing an [awesome proposal](https://docs.google.com/document/d/1iyQqcMFBwcd-Trym_wZZQWi1bdroGkNTkAcAzKd19hE/edit?usp=sharing).

## About my project

Kodi's addon checker tool helps developers who review the addons submitted on kodi's different kind of addon repositories. My project includes improving the functionality of this tool. I will also be adding some new functionality to the tool like performing static code analysis on the submitted addon or checking addons for python3 compatibility.


## Next

Very soon I will be writing another post about my first evaluation.
