---
layout: post
title: Running your own Bombsquad server
date: "2020-04-19"
---

If you just want to know how to setup bombsquad jump to [The setup](#the-setup) section and if you are interested in backstory, continue reading.

## Back Story

Few days into the COVID-19 lockdown and I was already getting bored. I mean I was spending my time either doing my college assignments, that were supposed to be submitted on Google Classroom, or I was just doing various kind of challenges like Pentesterlabs, VMs from Vulnhub. In the name of `Entertainment` I was scrolling through the wormhole of youtube videos until all my suggestions were completely uninteresting to me. That is when I decided that I should find a way to interact with my friends, other than talking on our telegram group. 

After some totally unrelated discussion we decided that we should play [skribbl](skribbl.io/) at night while we all are on a discord call(for easier communication). Now for the first 2 weeks we were totally enjoying our game playing/talking random stuff but after that skribbl became very boring because we were getting lot of repetitive words and our custom words weren't as funny as they were in the starting. So began the hunt of some other fun game, now I know lot of you might think that there are thousands of game that we all can play on steam or PS4/XBOX but the issue is that none of us have PS4/XBOX and only one of us have steam on their `gaming PC` so we needed something which was simpler and had an amazing multiplayer gameplay. 

I found a website called [netgames.io](https://netgames.io/games/) on which there were few games which are supposed to be played when players are in the same room but since we were communicating using VoIP we thought we can play these games. Now we tried one game after the other and all of them were very difficult to understand(atleast for us) except `Spyfall`. We were able to understand spyfall and played it for few days but after that we got bored from that as well. 

## Bombsquad

[Bombsquad](http://bombsquadgame.com/) is a an explosive arcade-style game which is best enjoyed with friends. So we decided that we'll try this but the sad news was that this game no longer supports mutliplayer joining via google account. But there is an option of running a bombsquad server and then other client can join that game, since this is a server client game. So I decided to tried to host a party on my own machine.

For self hosting there is a great guide written by the author himself which can be found [here](https://www.froemling.net/docs/hosting-bombsquad-games). I followed the steps mentioned in the documentation but it didn't work.

I was able to forward the port but for some reason the game didn't work. I kept getting an error saying my port 43210 isn't accesible from the internet. For a minute we thought of giving up on this as well and just go back to playing skribbl but I decided to give this one last shot.

## Bombsquad on DigitalOcean

I thought since it need a publically accesible IP it might work if I run it on Digital Ocean's droplet.

__It is not important to use digitalocean's droplet you can use any service you want as long as that server is publically accesible.__

### The Setup

Here's how you run Bombsquad on DigitalOcean droplet

**1)** Create a droplet
    
- __Distribution__
    
    ![](/images/distribution.png)

- __Plan__
    
    + You can select any of the plan even the lowest $5/month plan would work.

- __Datacenter region__
    
    + Since I am from India I thought it would be nice if we can have the server in india as well. 

- __Authentication__

    + For authentication it is recommended that you add a SSH key, if you already don't have one in your Digital Ocean account.

    + You can follow [this](https://www.digitalocean.com/docs/droplets/how-to/add-ssh-keys/) to see how to add a SSH key.

    + Also it would be nice if you generate a new SSH key instead of using your primary SSH key or a key which you are using for other projects or on github/gitlab.

        + For Key management, on Windows you can use PuTTY to store the key and for linux you can install a tool called [skm](https://github.com/TimothyYe/skm) for easy key management.

    + __NOTE__: All of this is just recommendation if you want to use your primary key then that is fine as well.

- __Hostname__:
   
    + For hostname you can choose any name, I named it `bombsquad`.

- Then hit the `Create Droplet` button.

***

**2)** Once the droplet is created, copy the IP and SSH into it.
   
- one thing to note here is that for SSH you'll have to use `root` as the username.

***
 
**3)** Run: `sudo apt update && sudo apt upgrade`
 
- This would take around a minute or two.

***

**4)** Once all the packages are upgraded, download the Bombsquad Server code. To do that run the following command:

```bash
wget https://files.ballistica.net/bombsquad/builds/BombSquad_Server_Linux_64bit_1.4.153.tar.gz

gunzip BombSquad_Server_Linux_64bit_1.4.153.tar.gz

tar -xvf BombSquad_Server_Linux_64bit_1.4.153.tar

mv BombSquad_Server_Linux_64bit_1.4.153 Bombsquad
```

***

**5)** Now you'll have a directory named `Bombsquad`.

**6)** Install the dependencies:
```bash
cd Bombsquad

apt install python2.7 libpython2.7

pip install -r requirements.txt
```

***

**7)** Now we'll edit the `config.py` file so open it in `vi` or `nano`:

```python
# place any of your own overrides here.
# see bombsquad_server for details on what you can override
# examples (uncomment to use):
config['partyName'] = 'My server'  # This will be the name of your party, could be anything
config['sessionType'] = 'teams'  # Leave it to teams
config['maxPartySize'] = 6       # Number of people you want, max is 8
config['port'] = 43210           # port on which the server will be listening

# Here you can add the codes for the stages you want to play.
# If you don't have any then leave it to None and the stages 
# will be assigned by default.
config['playlistCode'] = None 

# Option to make the party private or public. 
# if you just want to play with our friends and doesn't
# want anyone else to interfere then leave it False
config['partyIsPublic'] = False  
```

***

**8)** Now we can start the server but before we do that we'll change the permission of the directory from root to `www-data` because we will be running that server as `www-data`. This is done to keep the your droplet secure and this is the first rule of running any service on your server `Don't run the service as root`.

Run the following command to change the permission of folders:

* `chown -R www-data:www-data Bombsquad/`

Now start the server:

* `sudo -u www-data python2.7 bombsquad_server`

If everything was done correctly then you should see the following line:

![](/images/internet.png)

***

And now you can share your the IP address of your droplet with your friends and then can join the game by following steps:

* Open the game app
* Go to `gather` option
* Click on manual
* enter the IP and Port and connect.

***

By the way if anyone of you are able to run the server by portforwarding the port within your router then please let me know.

As Bombsquad also allow having your own mods I would be writing another blog post explaining how you can install mods on it. And I might also try to write my own mods. But this is it for now.

Enjoy bombing your friends and #stayhomestaysafe
