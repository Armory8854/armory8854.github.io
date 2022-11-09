---
title: Setting Up Pleroma - I Join The Fediverse!
layout: default
date: 2022-11-02
tags: linux-server fediverse self-hosting
---
# Introduction
Due to the state of the internet in 2022, and the gradual decline in both quality of the internet & websites over the last 10-20 years, I'm going to host a fediverse instance, and try to maintain it as long as I can. For a domain, I am going with namecheap. I'm also going to be using Namecheap's DNS for the time being. For the server, I'm going to be using Digital Ocean and use a Ubuntu 22.04 base. Normally, I'd just use Debian, but I'm going to stick with Ubuntu for the sake of following a guide to a T.

There are multiple ways to access and set up the fediverse. So many, I'm not even going to bother covering each one. Instead we're just going to jump right into setting up Pleroma on Ubuntu 22.04. Even though 22.10 is out and available, there's really nothing special in that release that I need, and 22.04 is the LTS (Long Term Support). For more information on what LTS truly means, here's a blog post from Ubuntu themselves [^1].

# Prerequisites - A server and domain
## Server
As covered above, I'm going to be using Ubuntu 22.04. If you want to follow along, any recent version of Ubuntu would be fine. I'm opting for Digital ocean's $18 Droplet, which I'm sure will need upgraded if my instance takes off. For now it will give me access to the net though.

## Domain
A server is no good unless it's connected to a domain! Really we just need to set an A record directing our domain to route to our server IP, then a cname record directing www to the IP. In your DNS settings (either Cloudflare or whatever you choose), set the records to this:

| Type  | Host | Value          | TTL  |
|:-----:|:----:|:--------------:|:----:|
| A     | @    | 123.456.789.0  | Auto |
| CNAME | www  | yourdomain.com | Auto |

This is a bare-bones DNS setup that should direct your website to the server. Replace 123.456.789.0 with your server IP, and yourdomain.com with your actual domain. **This is vital to do early on for setting up our SSL Certificates later.**

# My Step By Step
## Securing Our Environment
When you first get a VPS (Virtual Private Server), typically you just SSH for the first time as root. While it's convenient to do everything as root, and the guide above even suggests being root the entire time, it's highly discouraged from just operating the system as root. There's multiple reasons why, but the #1 one is security, mainly permissions and ownership, but the scope is very large. One of the strengths of Linux is users and groups, it's best not to take that lightly. Especially if expecting to manage a service on your own.

Before continuing any further, run the following command

`apt update && apt -y upgrade`

This should prompt you to reset some services, just hit tab and click OK. It may also prompt you to reboot your system. Now is the best time you're going to get for free downtime, so run a `reboot`, then SSH back in after a minute or two passes.

### Creating A New User
Before doing anything else, you should create a new user that has sudo rights.
From here on out, `user` is the username you decide to pick. First, use the `useradd` command to create a new user:

{% highlight shell %}
useradd -d /home/user -s /bin/bash user
{% endhighlight %}

You may be wondering why I specified the shell to be /bin/bash. This is because some VPS providers have a really barebones skeleton (pun very intended) template for creating new users. Having a new user without the bash shell at the very least makes operating the system very difficult.

### Giving Our User sudo Rights
Now that we have a user, we need to do 2 things to give them super user do access:
1. Give our user a password
2. Add them to the wheel group
3. Enable the sudo group (if not enabled)
4. (If Using SSH Keys, No Passwords) Copy root's SSH keys to our new user
5. Exit the SSH session, and ssh back in with our new user

#### Setting A User Password
This is relatively straightforward. Simply run the following command and input a password!

`passwd user`

#### Adding User To The Sudo Group
Using the `usermod` command, we're going to add our new user to the 'sudo' group. If you already know what you're doing and prefer the 'wheel' group, go for it. I'm just using sudo because Ubuntu defaults to that on my vps.

`usermod -aG sudo user`

#### Enable Sudo Group
On Ubuntu, this step is already done. However, if not enabled, no sudo commands will work. To enable this, use the command:

`EDITOR=nano visudo`

And make sure the following line is uncommented. If it doesn't exist as a comment, add it.

`%sudo    ALL=(ALL:ALL) ALL`

#### SSH Key Copying
If you enabled your VPS to only allow SSH keys, you can not SSH into your system as the new user yet. If you try, it will not prompt you for a password, and simply fail to connect. This is because the new user does not have the proper SSH keys. As root, run the following commands:

```
mkdir /home/user/.ssh
chmod 700 /home/user/.ssh
cp /root/.ssh/authorized_keys /home/user/.ssh/authorized_keys
chown -R user:user /home/user/.ssh
chmod 600 /home/user/.ssh/authorized_keys
```

#### Exit & Re SSH In
After the above steps are done, we can exit our server and then log back in - this time, with the proper username. To test if it worked, run `sudo apt update`, put in the password from the previous step, and see if it lets you update. If not - you missed a step!

### General Security
Note at any point while doing this, if apt gives you messages about autoremoving packages, you should certainly do that. Simply run

`sudo apt autoremove`

#### UFW
The world of iptables and firewalls may seem intimidating, but we're going to make things extremely easy for ourselves. Out of the gate, whenever setting up a forward facing internet linux machine, you should almost always do this.

```
sudo apt install -y ufw
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw enable
sudo ufw reload
```

#### Fail2ban
This is another fairly complicated topic that has an insane amount of options and configurations you could do. Really the important thing is to have something to deter your system from getting brute forced by the same people constantly. I'll post my general standard protocol for configuring fail2ban, but this is by no means comprehensive. It may not even be best practice. If you do have more information on this topic, please let me know!

```
sudo apt install -y fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

Now, with jail.local open, make the following edits. **Don't uncomment anything unless otherwise told to**

1. Find `bantime  = 10m` (note the 2 spaces after bantime. Weird, but effective way to find it) and change it to something that's at least 24 hours. I might even recommend a much longer time like 168 hours (an entire week). If your SSH keys work you will never fail to log in so it doesn't really matter.
2. Next, find `maxretry = 5` and change it to whatever you want. I usually do 3, this time I'm doing 2. This is the max amount of times someone can mess up logging in with a week.
3. Next, search for the **uncommented** `[sshd]`. Just add this under it
  - `enabled = true`
  
Exit & Save the file. Now, restart the fail2ban service. Also enable it just in case

```
sudo systemctl enable fail2ban
sudo systemctl restart fail2ban
```

You are now protected from blanket brute force SSH attacks, to a point!

### (Optional) unattended-upgrades 
Depending on how proactive you plan to be when managing your server, you may want to setup the `unattended-upgrades` tool. I personally do this on all my machines 'just in case'. It won't update & upgrade every single tool installed via apt, but it does cover important security upgrades. Just run the following command if you think this would help you.

``sudo apt install -y unattended-upgrades``

### Certbot - Free SSL Certificates, Automated
We're on the home stretch now, we can almost set up our site. By this point, if you did set up your DNS settings when told to, your website should forward to your server. Before we install cerbot, we must have an nginx.conf file. To do this we have to install nginx! Run the following command:

`sudo apt install -y nginx`

And then visit your domain. You should see this screen:

![The Default NGINX Splash Screen](/assets/images/defaultnginx.jpg)

If you do, you can continue installing Certbot. If you don't, you need to update your DNS settings as mentioned previously.

Now that you've done this, go ahead and run the install instructions here [^3], then run and install your ssl certificate! **Make sure to enter a valid email address when it asks - it will remind you when to renew your cert.**  When it asks you for what domains to get a cert for, make sure to enter it like this.

`yourdomain.com www.yourdomain.com`

After that, there's one final step: Setting up your certificate to automatically renew! run the following command to modify the `root` users crontab

`sudo crontab -e`

If it asks you for an editor to choose, roll with nano, unless you know what you are doing. We need to reset the certificate at least once every 3 months, I would say once every 2 months and 15 days is sufficient. 

`0 0 15 2 * certbot renew`

Make sure before you save your crontab, you hit enter 1 more time to have a blank new line at the end of the file. I'm not 100% sure why this is required, but your crontab will not save if it's not set.

# Pleroma
![The homepage of Pleroma at the time of writing. It features the title of the app, and a homescreen showcasing the User Interface design.](/assets/images/pleromahome.jpg)

*The homepage of Pleroma at the time of writing*

For Fediverse examples I'm going to install Pleroma [^2]. Take my step-by-step instructions with a grain of salt - and please, [Read The Docs!](https://docs.pleroma.social/backend/installation/otp_en/). This is a project updating often, and some steps may change.

## Basic Differences Between My Installation & The Docs
OK - so after installing this software, I realized there were quite a few things that needed changed. 

- If you followed this guide so far and created a new user to run sudo commands instead of running commands as root, you need to prefix every command the guide asks you to run with sudo.
- First off, they assign default users in this guide to a shell that gives no output. I suspect they did this for security reasons (the host account has no shell to exploit), but the end result for me was that every command they wanted me to switch users resulted in an error. Therefore, anytime in the guide you see useradd and they assign a user like this: `adduser ... --shell /bin/false` change the `--shell` to `/bin/bash`. 
    - There is a chance something was wrong with my setup - you could always try to just use their shell variable and see if it works for you first, then change the users default shell later.
- If you decide to install the postgres RUM plugins like the guide suggests as optional, make sure to check your postgres version first.
    - At the time of writing, this was 14, and the guide suggested version 11 as the version to install.
- I did not capture the specific error, but you *may* get an error involving libssl / openssl / crypto something or other when runngin `pleroma start` for the first time. If you do, this is because of a requirement for an old libssl version. The solution for me was this Stackoverflow thread [^4]. To condense what I did, I ran the following commands:
    ```
	cd
	wget http://nz2.archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.16_amd64.deb`
    sudo dpkg -i libssl1.1_1.1.1f-1ubuntu2.16_amd64.deb
	rm libssl1.1_1.1.1f-1ubuntu2.16_amd64.deb
	```

## Configuration


# Footnotes
[^1]: https://ubuntu.com/blog/what-is-an-ubuntu-lts-release

[^2]: https://pleroma.social

[^3]: https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal

[^4]: https://stackoverflow.com/questions/72133316/ubuntu-22-04-libssl-so-1-1-cannot-open-shared-object-file-no-such-file-or-di
