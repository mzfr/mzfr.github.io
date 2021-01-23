---
title: Using Github Action for recon
summary: Let's see if it's possible to use GitHub action for recon
date: "2021-01-23"
tags:
    - recon
---

In this blog post, I will talk a bit about how I use Github Actions for simple recon stuff. If you already know what Github Actions are and doesn't want to read the backstory then skip to the [How to Setup Github Action](#github-actions-for-recon)

## Back Story

As I wrote in my [year in review](https://blog.mzfr.me/posts/year-in-review-2020/) blog post, initially when I started doing bug-bounty I mostly focused on Android applications but with time I realized that just focusing on the android application isn't going to help me much. Because most of the time programs have usually 1 or 2 android apps in scope and those apps only sometimes tend to give bug that is not duplicated. So at the beginning of this year, I decided to expand my hunting scope and try to find bugs on web applications as well.

Now if you tend to be doing bug-bounty on websites/web applications you must have heard all the great hunters say things like `recon is important` or `P1 in 1 minute just by recon`(😂). And then you will see that most of the task in the recon has to be automated, emphasis on the word `has` because recon process has certain steps which we try to follow on all the programs/targets. So I decided to make my own recon setup, mine is not a very evolved one or something big but simple things.

### Basic recon flow

Initially, I had written a small script which does the following:
    - Store all the domains in the scope to a file called `scope.txt`
    - Run multiple subdomain enumeration tools like amass, subfinder etc on `scope.txt`, merge the output from all and give `subdomains.txt`
        - If subdomains.txt already exists then compare the new one with the old one and generate an alert if any new subdomain found.
    - Run massdns and httpx on that file.

I came up with this whole idea after reading lots of blog posts about recon methodologies. Almost everyone recommends having a VPS(not compulsory but helps). So what I thought was that I could set up this small script on cronjob on a VPS and then just wait for it to generate alerts. But that's when I thought to myself `why not do this with GitHub actions`

__What is Github Actions?__

GitHub Actions makes it easy to automate all your software workflows, now with world-class CI/CD. Build, test, and deploy your code right from GitHub. Make code reviews, branch management, and issue triaging work the way you want.

In simpler words, it is a better replacement for [`travis`](https://travis-ci.org/). If you have a GitHub project and you want to run a `linter` check or `tests` on every pull request to make sure nothing breaks you can setup a Github Action which will do the same.

***

Now even though Github Action is made for simpler things like running tests/linters it can do a lot of powerful stuff because, in the end, it is actually a Virtual machine. And things that can run on a VM(or even in a normal setup) can be executed via Github Action. 

## Github Actions for recon

Let's look at this with an example, say you want to run [`subfinder`](https://github.com/projectdiscovery/subfinder) on a file called `scope.txt` having `example.com` in it. And this `scope.txt` is present in your Github repo.

```yml
on: [push] # This is a trigger https://docs.github.com/en/actions/reference/events-that-trigger-workflows

jobs:
  build:
  # The OS for which will be used.
  # You can specify the exact version if you want.
    runs-on: ubuntu-latest

    # From here we just define all the steps that we have to do.
    steps:
      - name: Checkout Repo
        uses: actions/checkout@master

      # Install golang
      - name: Setup golang
        uses: actions/setup-go@v2
        with:
          go-version: '^1.14.14'

      - name: Setup go tools
        run: |
          go version
          GO111MODULE=on go get -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder

      - name: Run Subfinder
        run: subfinder -silent -dL scope.txt -o subfinder.txt

      - name: Save domains
        run: |
          git config --local user.email "myemail@example.com"
          git config --local user.name "MY USERNAME"
          git add --all
          git commit -m "Add Subdomains.txt"

      - name: Push changes
        uses: ad-m/github-push-action@master
        with:
          github_token: ${{ secrets.TOKEN }}

```

I have written some comments within the example above. The important thing for us is the `steps` these are basic things that we are asking the Github Actions to do inside the VM.

Below I've tried to explain all the steps in a bit detail:

* **Checkout your Repository file**

```yml
- name: Checkout Repo
    uses: actions/checkout@master
```

This is a predefined action, [actions/checkout](https://github.com/actions/checkout). This will basically copy all the files of your GitHub repository to the VM, meaning if you had the file called `scope.txt` in your repository then it will make a file named `scope.txt` in the `/home/` directory of the VM(the one which will run our job).

* **Setup golang**

Since `subfinder` is written in `golang` we need to install go in the VM.

```yml
- name: Setup golang
    uses: actions/setup-go@v2
    with:
        go-version: '^1.14.14'
```

The installation page for `subfinder` says that the go version `1.14+` is required so we are here installing `1.14.14`. By the way, this `actions/setup-go` is also a predefined GitHub action which you can find [here](https://github.com/actions/setup-go).

* **Install subfinder and run it**

```yml
- name: Setup go tools
  run: |
    go version
    GO111MODULE=on go get -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder
```
Till this step, we have to install subfinder in our environment. So if you have to think in the sense of a `normal` file system you can think of it like, there is a VM in which there is a `/home` directory inside that directory we have a directory named `go`(this is assuming all the go packages are being installed in a directory named `go/`). And in that, we have installed a tool called `subfinder`. Also in the `/home` directory is a file called `scope.txt`.

Now it's time for us to run `subfinder`

```yml
- name: Run Subfinder
  run: subfinder -silent -dL scope.txt -o subfinder.txt
```
This will execute the command on the VM and will produce the file with all the subdomains in it.

* **Push back the file to the repo**

Now once the subfinder generates the output you obviously would like to see the output. And the best option is to just push the files back to your repo.

```yml
- name: Save domains
    # if you don't want to push all the files back then you can also mention it like `git add subdomains.txt`
    run: |
        git config --local user.email "myemail@example.com"
        git config --local user.name "MY USERNAME"
        git add --all
        git commit -m "Add Subdomains.txt"

- name: Push changes
    uses: ad-m/github-push-action@master
    with:
        github_token: ${{ secrets.TOKEN }}
```
These two steps are basically adding all the newly generated files i.e `subdomains.txt` to the GitHub repo and then pushing that file. This is similar to how normal `add, commit, push` works in git.

If you notice the last line it says `secrets.TOKEN` this is a secret token that you can generate from your GitHub settings. To learn how to generate and add the token please read [this](https://docs.github.com/en/actions/reference/encrypted-secrets).

***

This was the basics of how you can run a simple command using Github actions. Now before we talk about how do I do complex stuff I would like to talk about the limitations of this setup.

### Limitations

Since Github Action wasn't meant for Bug Hunting or any kind of big task it has its own limitations. The ones that are pretty important are:

* Each job in a workflow can run for up to 6 hours of execution time.
    - Hitting the time limit will result in failure of the job
    - This means we can't run a masscan on a subnet which might take more than 6 hours

* You can execute up to 1000 API requests in an hour across all actions within a repository
    - Only 1000 API requests are allowed in an hour.
    - So if you want to do Github Dorking for subdomains then you need to find an efficient way to get more data in every single request.

* You get only certain minutes of Github Action per month.
    - For a PRO GitHub user you get 3000 minutes i.e 50 hours of Github Action per month
    - For a normal user I think the limit is around 2000 minutes per month

Other than that there are few more limitations and you can read about those [here](https://docs.github.com/en/actions/reference/usage-limits-billing-and-administration)

Now the last limitation about the `minutes per month` is the only thing that might sound a bit problematic but it's actually not. I did some testing and found out that if you have 5 targets(websites) in your `scope.txt` and you run `amass + subfinder + findomain` then pass the output to `massdns` and `httpx` then the whole job gets completed in under 2 minutes(never exceeds 2 minutes). 

If we do the maths, In a month there are ~730 hours, say you run the job every 5th hour that means the job run `146` times in a month. And assuming Every job took 2 minutes that will be a total of `292` minutes. And as a normal user, you had 2000 minutes so that means you'll still have ~1708 minutes of Github Actions. Now to fully drain out the limit you can run a lot of jobs at various intervals. Some of them are given below:

- Run a 2-minute job every 44 minutes and you'll have ~10 minutes left in the quota by the end of a month.
- Run a 27-minute job every 10 hours and you'll have ~21 minutes left in the quota by the end of a month.

__If I did the math wrong Please let me know 😂__

Well, these were just some stats but the whole point is that you can still run quite some tools on GitHub Actions.

### Suggestion on the process

* If any of you decide to try this out then I would recommend setting up a job or maybe multiple jobs with the schedule.
    - See Github's doc on [how to schedule events](https://docs.github.com/en/actions/reference/events-that-trigger-workflows#scheduled-events)

* Don't run anything big like masscan/nmap or ffuf with a very large list etc
* Instead of adding `steps` for every command make a small bash script and run them after the setup steps.

Ex:
```yml
- name: Setup go tools
run: |
    go version
    GO111MODULE=on go get -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder
    GO111MODULE=on go get -v github.com/projectdiscovery/httpx/cmd/httpx

- name: Subdomain enumeration
run: bash recon.sh
```

Inside the `recon.sh` you can use all the tools or the logic that you want in your recon process.

__NOTE__: Don't forget to install all the tools before using them in the script

* Use webhooks for notification. Like within the script add a `curl` command to send an alert if something happened. 

Ex: Say you want to be notified at the moment you get the new subdomains so you can add something

```bash
curl -X POST -H 'Content-type: application/json' --data '{"text":"Found new Subdomains"}' YOUR_WEBHOOK_URL
```

This is just an example of if you made a bash script.

### Benefits of GitHub Actions

I mean a lot of you might already have a perfect setup on your VPS/home servers but if you are just starting out then there can be quite a few benefits with this setup.

* Setting Github Action is pretty easy
    - Might look difficult but it's actually not

* No need for logging your stuff.
    - If you set up a cron job on VPS you'll have to keep the logs of that cron, in case it might fail sometime that will help in debugging in your script/code
        - I know cron logs can be found in `/var/log/syslog` but I am saying this just for the sake of argument.
    - But with Github Action if any of the steps fail it will show the detailed error of why the failure occurred.

* No cost
    - You don't have to pay any monthly charge, you can do this even without being PRO Github User

* Data availability
    - You can install the Github Android/iOS app and then keep the track of all the data that if being found
    - Here you can argue that we can do the same by sending the data via Webhooks on slack
        - But as a counter-argument, I would say if you run ffuf with `-mc all -ac` then there are chances of having loads of data so the best option is to just keep it in a file on a repo 😄😄

### Conclusion

In my opinion, using Github Actions is really nice for doing recon(to some limits). I have my private setup which does the recon stuff for me and then I have another cronjob that pulls the data from the repo to my system where I can analyze the data properly. I also had an idea about analysis of the data i.e you can have another script or maybe include this in your recon script which converts the output of all the tools to `markdown` and then pushes the changes this way all the data will be beautified and much easy to go through.

This is just one of the idea, the possibilities are infinite here. In the end, I'd just say that if you haven't tried something like this then it's definitely worth a try. And if you end up doing something cool with this then let me know.

Also if you find any problem with all the mathematics that I did then please let me know 😄.

***

Thanks for reading, Feedback is always appreciated.

You can follow me on [@0xmzfr](https://twitter.com/0xmzfr).
