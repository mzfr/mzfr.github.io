---
layout: post
title: My OSCP experience
date: "2021-04-26"
aliases:
    - /OSCP-experience
---

So it finally happened, I got my [OSCP](https://www.credly.com/earner/earned/badge/537571f6-6d4c-4aec-ab49-991bdd570e04). This blog post is going to be just me talking about what I did right, what I did wrong and maybe some tips for people who plan to take the exam in the future.

# PWK Course

Also, known as [`PEN-200`](https://www.offensive-security.com/pwk-oscp/) is the course one takes in order to get their OSCP Certification. The official definition for this course is as follows:

> Penetration Testing with Kali Linux (PEN-200) is the foundational course at Offensive Security. Those new to OffSec or penetration testing should start here.

I bought the course on March 6th, 2021, and it was supposed to start on March 14th, 2021. I bought 30 days of labs and paid 999 USD (~73,154.77 INR). So the __first tips__ would be to only go for 30 days lab iff:

- you already have some experience with HackTheBox/Vulnhub/Proving Ground.
- you are free for the next 30 days(if you plan to do more than 40 machines in labs)

## About the course

### Course material

Talking about the course, Offsec provides you a PDF and the videos. The PDF contains all the information you could possibly need while doing the labs or while giving the exam. Most people like to go through all the videos first and then start with labs. But I didn't watch the videos at all and only went through the PDF for certain sections which seemed new to me. 

If you are someone who has experience with boot2root machines then you'd be very comfortable setting up the VPN and starting the labs. If you are a total beginner then I would highly recommend going through every page of the course PDF. 

### Labs

There are around 60-65 machines in the labs. I was able to root all of them but there were a lot of machines in which I had to go through the forum post to get some hints and in some, I had to ask [@nikhil](https://twitter.com/0xw0lf) for some hints. There are definitely some machines in which you'll just lose your mind completely even when you know what to do. But all in all, it's a bittersweet ride.

Here also I recommend that you try to root all the machines. And that would only be possible if you do around 2-3 machines a day. Some machines were very easy and there were few days when I rooted almost around 5-6 host in a day. But then there were days where I was totally burned out and

# Enumeration/Reconnoissance

I think this is the most important part in order to pass OSCP. That is because you know the machine is exploitable but you don't know the entry point or the service which is actually exploitable so you just need to enumerate until you find that.

My tip here would be to make your own workflow. Just do labs and as you go you learn what works, what doesn't work, and then according to that decide what you want to do, how you want to do it. Maybe read all the `OSCP experience` blog posts and see what people suggested and then see what worked for you and then make your own thing.

### My enumeration methodology

Mine was simple just run `nmap` scan on the machine, I used to run two scans first one was for `--top-ports 5000`(top 5000 ports), and another one was a full port scan(`-p-`) which I would run in [`tmux`](https://github.com/tmux/tmux/wiki). Once the scan would complete I would see all the running services and start going through them one by one.

__Ex__: Say a machine is running `smb`, `ftp`, and `HTTP`. So I'll just see if FTP allows anonymous logins or not. If yes then see what is present inside FTP. 
With SMB service I would run: `nmap --script 'smb-vuln*' -p139,445 IP`, to see if SMB is vulnerable to any famous CVE.

I never used tools like [`AutoRecon`](https://github.com/Tib3rius/AutoRecon) or [`Reconnoitre`](https://github.com/codingo/Reconnoitre) for recon process. Also if you are not aware of it, [@_superhero1 failed OSCP because they used linpeas.sh](https://twitter.com/_superhero1/status/1385206684109447168)(the decision was later altered by offsec and **@_superhero1 passed OSCP**). 

I am mentioning this because offsec released a blog post in which they emphasize that how one should spend time reviewing the code that a person is running. That is why I suggested making your own workflow. That workflow would not only include a checklist/methodology but will also make you come up with your own tools. **Ex:** Run 3 different enumeration scripts (e.g: linpeas, LinEnum, lse.sh), and see what all they do, and then make your own combining all the important things you liked from them.

P.S - You can read the full offsec blog post [here](https://www.offensive-security.com/offsec/understanding-pentest-tools-scripts/)

# Twist before the exam

My labs were supposed to end on 14 April 2021 but then I got COVID. I got my first symptom on the 7th of April,2021. It started with a fever and then I had a bit of cough. Even though I didn't had any serious issue, I didn't felt like continuing with the labs. Actually, I was already done with all the labs but didn't do any of the course exercises.

So this is another __tip__, do all the course exercises because writeup for course exercises + writeup for 10 Lab machines will give you [5 bonus points](https://help.offensive-security.com/hc/en-us/articles/360040165632-OSCP-Exam-Guide#bonus-points). And these points can turn out to be a lifesaver for you. So I highly recommend doing exercises.

# Exam

I started my exam at 11:30 AM and the first machine I attacked was the Buffer Overflow, while I ran a full port scan on 2(25 pts and 20 pts) other hosts in the background. I was expecting to finish the BoF machine in around 30-45 minutes but it took me 1:30 hour to finish the machine + the writeup for the machine. I feel I took extra time here because in the very starting I was completely lost with BoF and had to go through my notes once to recall all the things which should be done to do BoF. But once I got in the zone and started doing "things" I was easily able to root the machine. Once I rooted that machine, I took a 5-10 minutes break. 

When I came back from the break the two scans which I ran in the background were completed. I just picked the 25 point host and started to enumerate it. Gaining the initial shell on that machine wasn't hard at all but I had some complications while doing the privilege escalation. But in around 2 - 2:30 hours I was able to get a root shell on that host as well. So I took another break. My plan was to root at least 3 machines in the first 6 hours and then take a good long break. And I was perfectly able to execute that plan. I was able to root another 10 points host. So in around 6 - 6:15 hours, I got root access on 3 hosts. 

This was the part where the problems began for me. When I came back from a good long break I started attacking the 20 points machine. I did a lot of stupid mistakes but in around 1 - 1:30 hours I was able to get the initial shell on the host. Now I had/have this mentality where I feel that initial foothold is always harder than privilege escalation, so the moment I got a shell on the host I was super pumped and almost chalked this host as "rooted" but that was the mistake. I spent the next 3 hours trying to find the privilege escalation for this machine but couldn't find it. So I just decided to go for dinner and thought taking a break would help me. But it didn't, I took about a 45-minute break for dinner but when I came back I reverted the machine and re-scanned it to make sure I wasn't missing anything but nothing, I just couldn't find the privilege escalation. I completely dropped that machine and started doing the last 20 points machine. Spent about 2 hours trying to find the entry point but couldn't do it. So I just decided to go to bed. 

The next morning, I woke up at 6:30 and then came back to try PE on a 20 point machine and to find an initial foothold on another 20 point machine. I continuously worked on these machines for almost 3 - 3:30 hours but didn't progress at all. It was around 9:45 when I decided to gather all the screenshots etc for the report. By 11:00 AM I was done with my report. My exam ended at 11:30 AM and even though I was done with the report I didn't submit the report until 10 PM.

When I finally submitted my report I was really scared because in order to pass the exam one needs 70 points and I have rooted 2 25 points machines, a 10 points machine, and just local access on another 20 points machine. Basically, I was standing on the border, so if I mess up my report or anything I wouldn't have passed my exam. This was the point where I was thinking, "I should have done those course exercises in order to get those bonus points". So that is why I'll say it again, **do Course exercises**.

P.S -  This point calculation etc is done from my side only offensive security doesn't give you any feedback about how much you scored.

# Result

After 24 hours of submitting the report, I got an email saying that I have passed the exam. I was really happy and also surprised that I cleared the exam.

![](/images/oscp.png)

In the end, all it matters is that you `pass` the exam :)

# Suggestions & Tips

Other than the ones I gave above following are some tips/suggestions I would give to anyone planning to take the exam:

- Exam is 24 hours long, no need to rush things take your time.
    - After reading a lot of blog posts I was also feeling like if I could finish the exam in 9-12 hours it would be so awesome etc but seriously it's not a competition just take your time.
- Take breaks, whether they are dinner/sleep or just a 5 minute "away from the screen" break.
- Revert the machines, the moment you start to feel like you've tried everything and nothing seems to work, just revert the machine and see if you notice any changes either in the output of your scans or the services running.
- Sleep well before the exam. 
    - Might be difficult because of all the anxiety but a good night's sleep will definitely help you.
- Check everything is working before the exam
    - Whether you are using a VM or WSL just make sure it's working.
    - If you are using the external camera make sure it's working properly.
- While doing labs make your own notes
    - They don't have to be very descriptive but just write down things in your own words.
    - Ex: Before starting the 30 days labs I wasn't really comfortable with boot2root machines with windows OS. So while I did all the windows machines from the lab I made a list of things I would check in the windows machine, for privilege escalation.

# Q&A

### *What was your setup for the exam?*

I was using Windows as my Host OS and I had kali Linux running in WSL2. I was running GUI applications with the help of tmux and [xlaunch](http://www.straightrunning.com/XmingNotes/IDH_PROGRAM.htm). Along with that, I was using Logitech's external webcam. I didn't have multiple screens, I just had 1 screen.

I used xlaunch because for me [`win-kex`](https://www.kali.org/docs/wsl/win-kex/) was causing some [trouble](https://github.com/microsoft/WSL/discussions/6675#discussion-3274303).

P.S - I'll be writing a new blog post explaining the win-kex issue and all the possible fixes for it.

### *What did you use for report writing?*

I use [obsidian](https://obsidian.md/) for my note-taking so I used it for writing the report. And I was using [this](https://github.com/noraj/OSCP-Exam-Report-Template-Markdown/blob/master/src/OSCP-exam-report-template_whoisflynn_v3.2.md) template for the report.

__NOTE:__ I had made a lot of changes to that template so the [generate.rb](https://github.com/noraj/OSCP-Exam-Report-Template-Markdown/blob/master/generate.rb) didn't generate the report for me. So I had to manually add some stuff to the template and then generate the PDF using the obsidian's `export as PDF` feature.

### *Is the exam hard?*

This depends on the amount of practice you have done. I personally felt that it was somewhere between easy and hard 😉. During the exams, I felt some machines were pretty easy and some made me lose my patience and temper.

### *How to avoid rabbit holes?*

Well, no one can answer this question. This also comes with practice and experience. You should know how much to dig and when to stop looking at that place. I personally avoided them just by following few simple steps:
    - I would run nmap's [vulner script](https://github.com/vulnersCom/nmap-vulners) on all the running services.
    - Then open all those CVE/vuln links in the browser
    - And pick the one which was something that could get me a shell.

Ex: Say that port 21,445,139,8080 is open and you run `vulner` on all those ports and got like 10 links to different CVE's. Just open them all and either read their title/name or their descriptions. Vulnerabilities like `Denial of services(DOS)` or `XSS` could just be ignored since those giving a shell is very rare. So pick something like `RCE` or some famous CVE.

__Note:__ This is something that worked for me. I am not claiming that this will work 100% of the time.

### *Did you had to use Metasploit?*

- During the exam? - No
- During labs? - Yes, sometimes.

### Did you record your screen during the exam?

__UPDATED__: Turns out offsec doesn't allow to record the screen now. Thanks to [@MuirlandOracle](https://twitter.com/MuirlandOracle) for [pointing it out](https://twitter.com/MuirlandOracle/status/1386987176454410240). You can read more about recording the screen [here](https://help.offensive-security.com/hc/en-us/articles/360040576311-Can-I-record-my-screen-when-taking-the-exam-).

I've read that lot of people used OBS to record their screen while they were giving the exam but I **didn't** do it. I am used to writing the reports/walkthroughs while doing the machine ever since I started doing boot2root in 2019. So I just didn't record anything. I took all the screenshots and recorded all the necessary outputs as I did the machine.

But this could be something useful so if your system is powerful enough that it can share your screen via a browser and then record the screen(s) via another tool then sure go ahead and do it.

### Which browser did you use during the exam?

This might sound a stupid question but when I went through the [Proctoring-Tool-Student-Manual](https://help.offensive-security.com/hc/en-us/articles/360050299352-Proctoring-Tool-Student-Manual) in that `Step 3` talks about Janus Plugin Installation. Now if you google you'll see that that plugin is only available for chrome. So I thought that chrome might be the requirement of the exam but it's actually not.

If you use Firefox as your default browser then no need to install anything. Everything worked perfectly during proctoring in firefox as well, without having to install any tool/plugin.

### *How to practice Buffer Overflow?*

Just do [Buffer Overflow Prep](https://tryhackme.com/room/bufferoverflowprep) room on [TryHackMe](https://tryhackme.com/) by [tib3rius](https://tryhackme.com/p/Tib3rius). After that just practice with the PWK course and labs. No need to do anything else. That will be more than enough.

### *Are your notes public?*

Nope, but I can list a few websites/guides I would check whenever I felt I was stuck.

* For Linux Privilege escalation - https://sushant747.gitbooks.io/total-oscp-guide/privilege_escalation_-_linux.html
* For Windows Privilege escalation - https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_windows.html
* If SMB was running then run: `nmap --script 'smb-vuln*' -p139,445 IP`
* For Shell escape
    - https://www.exploit-db.com/docs/english/44592-linux-restricted-shell-bypass-guide.pdf
	- `echo os.system("/bin/bash")`
	- https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/


***

Thanks for reading, feedback is always appreciated.

You can follow me on [@0xmzfr](https://twitter.com/0xmzfr).
