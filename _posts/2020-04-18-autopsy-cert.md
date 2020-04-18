---
layout: post
title: Review of Autopsy online training
date: 2020-04-18 2:25 PM
---

It's been a while since I wrote any blog post. Since this lockdown is going on I decided to learn a few new things from online courses. 

I came to know that [Basic technology](https://www.basistech.com/) was offering their autopsy basics and hand on (8-hours) training course, worth $495, for free during this COVID-19 crisis. So I decided to take the course since I'm very much interested in digital forensics. 

My very first encounter with digital forensics was in one of the CTF, named otter CTF, that I played with OpenToAll. It was complete forensics and RE focusing CTF and had loads of challenges which included the use of tools like [volatility](https://github.com/volatilityfoundation/volatility/), [testdisk](https://www.cgsecurity.org/wiki/TestDisk), [photorec](https://www.cgsecurity.org/wiki/PhotoRec), etc and I was very happy that I was able to solve a lot of those challenges(you can read my write for that CTF [here](https://github.com/mzfr/ctf-writeups/tree/master/OtterCTF%202018/Memory%20Forensics)).


## About the course

The course teaches you about the very basics of how autopsy works and how it can be used for finding different kinds of information from various kinds of disk images. It starts by giving you a very brief introduction about what autopsy is and why it is used, from there on the real fun of digital/memory forensics starts. It shows you how autopsy uses various modules to extract information from the images, some of those tools are the ones I mentioned above(testdisk, photorec). The course also teaches you how autopsy can be used in a shared environment where multiple members can work on various projects and can still share details/information about old projects.

There are 17 sections and in each section, once the course video is completed then we are given a small quiz containing questions about what we learned from the videos. Some of those sections contain labs.

### Labs

So one of the best things about the course is the labs that they provide. These labs are for hands-on practice of what is taught in the videos of that section. I really liked the storyline that they provide for working on the labs, which adds a sense of reality when you work on those labs. 

For solving labs you are given 2 images one of which is meant to be from a laptop and another one is from mobile, both of these disks have some special significance in the storyline. These labs have similar kinds of the quiz but this time you have to answer those questions based on your findings. 

## Certificate

Once you complete all the sections you are given an option to download your certification for the completion of training. The certification provider is Acceredible and your certification will be available to you either on their website or you'll receive an email from the autopsy team.

### My certificate

![My certificate](/images/my_cert.png)

[My credentials](https://www.credential.net/8e8f5206-9e0d-4699-884e-829b31998fab)


## Few suggestions on course improvement

All in all, the course is really amazing and gives you a really nice introduction to the course but I felt there were few things that would have made this course much better:

* Not having very obvious answers to some questions. Ex: There we some questions that had 4 options and only 1 of them was the __technical__ answer and the other 3 were some funny things which obviously won't be the answer. 
    - Not all of the questions had this but some of that did.

* Sometimes I completed a section and even after reloading the page, the website showed that I haven't completed it. I don't know whether this issue was just for me or people faced it as well but I just thought I should point it out.
    - If this happens to you just go back to the section you just did and skip the video to end and finish the quiz again.

* They gave only a very small introduction on Android analysis, which I think was good but it would be more fun if they can add few more things to that section (maybe a lab).

* Maybe they can add a small lab quiz in the end in which a student would receive another disk image and will have to answer questions and those questions would cover all the topics taught in all the previous sections. This would be a nice addition since that would act as a final test.


## Conclusion

Personally I haven't used sleuthkit or autopsy much in the CTFs before but after doing this course it looks like I should definitely try these since I felt that they can really make digital forensics challenges easier to solve. 

In the end, I'll just say that this is an amazing course which is being offered for free so I think people should make the most out of this opportunity and do this course. And also a huge thanks to basis technology for making this course free so people, at least all the students, around the globe can do this.
