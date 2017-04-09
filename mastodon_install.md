## *This guide is for Server Admins only, for user guides please see [this site](http://mastoguide.info)*


# Mastodon Super-Tutorial! 
This will be a VERY simple, easy to use tutorial. This will not cover customizing, however it does cover:

- Installing Mastodon
- Maintainence of Mastodon
- Upgrades
- SSL Certificates(Let's Encrypt)
- Nginx configurations
- Troubleshooting basic issues with Mastodon & Nginx services

This tutorial expects that you are running on Ubuntu 15.04+. However - This can be done with virtually any linux distrobution that can use Docker application.
We also expect that your VPS/Dedicated machine has an external IP Address, that is properly routed and a domain name to go with it! (You need it for SSL cert!!)

## Tutorial Menu
- [Mastodon Official Documentation (Should see first)](https://github.com/tootsuite/mastodon/tree/master/docs)
- [Warnings & Common Sense](#warnings)
- [Requirements](#Requirements)
- [Installing Docker](#InstallDocker)
- [Installing docker-compose](#InstallDockerCompose)
- [Download Mastodon](#GetMastodon)
- [Building Mastodon & Installing Mastodon Dependencies](#BuildMastodon)
- [Install & proxy through nginx](#InstallNginx)
- [Setup cron jobs for Mastodon cleaning](#SetupCron)
- [Maintainence & Troubleshooting of Mastodon](#Troubleshooting)
- [Maintainence & Troubleshooting of Nginx](#nginx)

<a name="warnings"></a>
## Warnings & Common Sense
As with any tutorial where the user requires root permissions, you need to be aware of the following things:

- Although tested, some commands, when ran as root(sudo), can break the system. It is unlikely - but I have to put in a warning for legal reasons here.
- You are doing this of your own free will, you are aware that any commands here might break stuff, and you are doing this at your own risk!
- Once the service is online - You cannot blame me or Mastodon's developers if a hacker DDoS's, gets into, or otherwise with your server.
- Security is best practiced with you! This means keep _strong_ passwords! Changing them every couple weeks to months. Never settle for weak security!
- I have to say it because of certain services floating around the internet: Keep an eye on your server. Not my fault if someone uploads illegal stuff on it. (Really - Moderate it!)
- _FIREWALL_!!! Using ufw or iptables on linux will keep the bad guys at bay (at least most of them, it is _very_ unlikely they will care enough to get into _your_ server.)



<a name="Requirements"></a>
## Requirements

- A server (VPS, Dedicated, etc) running Ubuntu 16.04+ (Or Debian 8+)
- Depending on the amount of users you expect, at least 4GB RAM or more. (Single user instances can be around 2GB)
- I would recommend at least a 100mbit unmetered connection with the service provider (But not necessary.)
- 100GB HDD+ is pretty decent. But the more the better to be honest.
- An SMTP host for registering of user accounts. (Your own, mailchimp, etc should work)
- A basic understanding of linux is most likely needed for proper maintainence of Mastodon.(Like using cURL, nano, etc.)

<a name="InstallDocker"></a>
## Installing Docker & docker-compose

### Installing Docker
You can run the following in order. If an explination is needed on something it will be under the command.

1. ```sudo apt-get -y --no-install-recommends install curl apt-transport-https ca-certificates curl software-properties-common```
> The above command is used to get some dependencies and enable apt-get to use HTTPS connections.

2. ```curl -fsSL https://apt.dockerproject.org/gpg | sudo apt-key add -```
> The above command downloads the GPG key of the Docker repository, this will be needed to download docker in later steps.

3. ```sudo add-apt-repository "deb https://apt.dockerproject.org/repo/ ubuntu-$(lsb_release -cs) main"```
> The above command adds the docker repo for your version of ubuntu (Should at least be 16.04 LTS!!)

4. ```sudo apt-get update```
5. ```sudo apt-get -y install docker-engine```
> The above 2 commands update the repos, and then installs Docker.

6. Then make sure the docker service is started:
```sudo systemctl restart docker.service```

<a name="InstallDockerCompose"></a>
### Installing docker-compose
You can run the following in order. If an explination is needed on something it will be under the command.

1. ```sudo curl -L "https://github.com/docker/compose/releases/download/1.10.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose```
> The above command will grab the version specified by ../download/x.xx.x/docker-compose..... Make sure to use the latest command from https://github.com/docker/compose/releases !!!

2. ```sudo chmod +x /usr/local/bin/docker-compose```
> The above command makes docker-compose executable by any user. (Not a security risk in this case. If you would like to know more about linux permissions please see: https://ss64.com/bash/chmod.html)

<a name="GetMastodon"></a>
## Getting the latest Mastodon code from github
Now that we have docker installed, we need to get the latest and greatest code from it's github (https://github.com/tootsuite/mastodon/)

So before we continue we need to do the following:

1. ```sudo adduser mastodon && sudo adduser mastodon sudo && sudo adduser mastodon docker```
> The above command adds a new user to the system. "mastodon" will be the user we run the install/configure commands as.

2. After filling out the mastodon user password do: ```sudo su - mastodon``` to switch to that user. and ```cd ~``` to change into that users "home" directory
3. ```git clone https://github.com/tootsuite/mastodon/```
> The above command downloads the latest mastodon code and puts it in the folder "mastodon" by default. to change it add a -O /path/to/folder/ you would like.

4. ```cd mastodon/```
> The above command will put us in the newly download mastodon source code folder.

<a name="BuildMastodon"></a>
## Building Mastodon
Since we are using docker for this, building mastodon is actually pretty straight forward.
 
<a name="InstallMastodon"></a>
### Mastodon install
1. Make sure you are in the mastodon source folder. (For this tutorial it might be /home/mastodon/mastodon)
2. Then ``` cp .env.production.sample .env.production```
> The above copies a configuration file that will not be overwritten by updates later.

3. Modify the configuration file using your favorite text editor to meet your needs. 
4. For the application secrets, run the following command 3 times: 
4. ```docker-compose run --rm web rake secret``` <- MUST HAVE. See notes in _.env.production_ file. [Example config file](https://urgero.org/application/pages/howto/linux/env_config)
> The above command will take time to run the first time, then it will be _MUCH_ faster the second and third time. Also make sure that the 3 secrets in the .env.production file are different. (See that file for details.)

4. Once the configuration is done and dusted run: ```docker-compose build``` to build mastodon.
5. ```docker-compose up -d```
> The above command starts the mastodon server. But we are not done yet!!!

6. ```docker-compose run --rm web rails db:migrate```
> Need to upgrade the mastodon database.

7. ```docker-compose run --rm web rails assets:precompile```
> Precompile the asset cache for SPEED.

8. ```docker-compose restart web```
> Needed again after the precompile command always, to pick up the new bundles.

<a name="InstallNginx"></a>
## Installing nginx to proxy to mastodon.
nginx is not required here, you COULD use apache - however I have had better performance using nginx.

1. ```sudo apt-get install nginx```
2. Move all the files under /etc/nginx/sites-enabled/ to a different folder. We don't need them.
3. ```sudo mv /etc/nginx/sites-enabled/* ~/```
4. Now we need to edit a new file that doesn't exist yet: ```nano /etc/nginx/sites-enabled/mastodon```
> The above command will create and open a new file in nano. 

5. Download the following file and put it's contents inside the above file. [Mastodon nginx configuration](https://urgero.org/application/pages/howto/linux/mastodon)
> If the above file is not available, you can find the nginx configuration at Masto's [github](https://github.com/tootsuite/mastodon/blob/master/docs/Running-Mastodon/Production-guide.md)

6. Put the following in "/etc/nginx/sites-enabled/mastodon-http" making sure to change example.com. This config will redirect all traffic to port 443 on your server. (SSL)
<pre>
server {
    listen      80;
    server_name example.com;
    rewrite     ^   https://$server_name$request_uri? permanent;
}
</pre>

6. Now we need Let's encrypt:
    1. ```sudo apt-get install letsencrypt```
    2. ```sudo systemctl stop nginx.service```
    3. ```letsencrypt certonly --standalone -d example.com``` Where example.com is the domain for your masto-instance!!
	4. If you need help with this please visit [Certbot](https://certbot.eff.org/#ubuntuxenial-nginx)

7. Edit /etc/nginx/sites-enabled/mastodon and change the following lines to match your domain name:
    - ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    - ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

8. Once that is done ```cd ~``` to change back to your home directory.
9. ```sudo -i``` to login as root of your VPS/Server.
10. Make a new file called update_mastodon, this file will be used to run updates on the Mastodon serivce. (See [Maintaing Mastodon](#Troubleshooting) for details on how to use.)
11. Inside that file put the following (Or [download](https://urgero.org/update_masto.sh)):
<pre>
#!/bin/bash
cd /home/mastodon/mastodon
echo -e "Checking for update.."
git remote update
UPSTREAM=${1:-'@{u}'}
LOCAL=$(git rev-parse @)
REMOTE=$(git rev-parse "$UPSTREAM")
BASE=$(git merge-base @ "$UPSTREAM")
if [ $LOCAL = $REMOTE ]; then
    echo "Up-to-date. No need to continue."
    exit 0
elif [ $LOCAL = $BASE ]; then
    echo "Need to pull"
    echo "Pulling in  10 seconds. CTRL+C to cancel.."
    sleep 10
    docker-compose stop
    git pull
    docker-compose build
    docker-compose run --rm web rails db:migrate
    docker-compose run --rm web rails assets:precompile
    docker-compose build
    docker-compose up -d
	systemctl restart nginx.service
elif [ $REMOTE = $BASE ]; then
    echo "Need to push - nothing to do."
    exit 0
else
    echo "Diverged. Oops."
    exit 0
fi
exit 0
</pre>
> The above file is a script to update mastodon for you. Please keep it handy!

12. ```chmod +x update_mastodon```
> The above command makes the update script executable.

13. Now run the update script to clean and start mastodon: ```bash update_mastodon```
14. When that completes all you need to do to start mastodon:
    1. ```sudo su - mastodon```
    2. ```cd mastodon/```
    3. ```docker-compose stop && docker-compose up -d```
    > The above makes sure mastodon is completely clean before starting again.
    
    4. If needed: ```systemctl restart nginx.server``` (As root!!)

15. Register as a user on your instance RIGHT NOW!!!!! Because you will want to be the admin!

<a name="SetupCron"></a>
### Setting up the cron jobs:
Cron is very important as it keeps your mastodon instance nice and clean.


1. ```sudo su - mastodon```
2. ```nano masto_cron``` and put the following in there:
<pre>
cd /home/mastodon/mastodon
# Run jobs:
docker-compose run --rm web rake mastodon:media:clear
docker-compose run --rm web rake mastodon:push:refresh
docker-compose run --rm web rake mastodon:push:clear
docker-compose run --rm web rake mastodon:feeds:clear
</pre>


3. Save that file and: ```sudo chmod +x masto_cron && sudo crontab -e``` add the following to the end of the file:(Make sure to edit the paths as needed!!!!)
<pre>    
0 22 * * * /home/mastodon/masto_cron > /home/mastodon/masto_log
</pre>

<a name="Troubleshooting"></a>
## Maintaining & Troubleshooting Mastodon once installed

### Starting Mastodon after a reboot, or for whatever reason:

1. ```sudo su - mastodon```
2. ```cd mastodon/```
3. ```docker-compose up -d```
4. ```exit```
5. ```sudo systemctl restart nginx.service```

### Stopping the Mastodon server:

1. ```sudo su - mastodon```
2. ```cd mastodon/```
3. ```docker-compose stop```
4. ```exit```

### Restarting the Mastodon server

1. ```sudo su - mastodon```
2. ```cd mastodon/```
3. ```docker-compose stop```
4. ```docker-compose up -d```
> The above commands restart the docker container for you. This _should really_ only be done to troubleshoot errors.

### Updating:
Updating is pretty simple, just run that update_mastodon script _as root_ you made and it does all of the work for you!

### To make an account an admin:

1. ```sudo su - mastodon```
2. ```cd mastodon/```
3. ```docker-compose run --rm web rails mastodon:make_admin USERNAME=example```
4. Then logout and log back in.

### Refresh a users avatar

1. ```sudo su - mastodon```
2. ```cd mastodon/```
3. ```docker-compose run --rm web rails c```
4. Once the rails terminal opens: (You might need to type in manually, or run line by line as copy-paste of the following code might not work properly)
<pre>
a =Account.find_remote('-username-', '-domain-')
a.avatar = URI.parse(a.avatar_remote_url)
a.save
</pre>

### Rebuild remote user avatar cache

1. ```sudo su - mastodon```
2. ```cd mastodon/```
3. ```docker-compose run --rm web rails c```
4. Once the rails terminal opens: (You might need to type in manually, or run line by line as copy-paste of the following code might not work properly)
<pre>
Account.remote.find_each do |a|
a.avatar = URI.parse(a.avatar_remote_url)
a.save
end
</pre>

### Rebuild all timelines (For troubleshooting only!!!)
This will force all user timeline streams to be nulled - They are rebuilt completely on next user interaction/login.

1. ```sudo su - mastodon```
2. ```cd mastodon/```
3. ```docker-compose run --rm web rake mastodon:feeds:clear_all```

### Reset git back to origin/master

1. ```sudo -i```
2. ```cd /home/mastodon/mastodon```
3. ``` git fetch --all```
4. ```git reset --hard origin/master```
5. ```git pull origin master```
6. ```git checkout master```
7. ```git status``` To confirm it should output something like: "On branch master Your branch is up-to-date with 'origin/master'. nothing to commit, working directory clean

### I am not getting emails from Mastodon
This is a weird issue to have because it could be one of two things:

- Your email provider does not allow 3rd party applications to send email
- You configured Mastodon with bad SMTP information in _.env.production_ file.

If you put wrong information in the config file (Always double check, it is usually the port number and password that was typed wrong) just fix the information and restart mastodon and nginx.

If you are using a known bad email provider (See list below) then find a new email provider for your instance.

The following email service providers are known to _have_ issues:

- Google's gmail services (Both the free and _GSuite_ versions)
- Microsoft's Office 365 services (Both the corporate, and personal accounts)
- Hotmail
- Yahoo (Assumed for now, not actually tested)

To resolve this issue, running your own email is usually the way to go. If you are new to doing this, I would recommend [Mail-In-A-Box](https://mailinabox.email)
however - this tutorial does not cover setting up the email server because it is out of the scope of this document. (Mail-In-A-Box has AMAZING Documentation, community, and howto for that!)

If setting up another server is not in your interests, https://mailgun.com might be for you. 10,000 free emails a month, and pretty fast as well.

<br>
<a name="nginx"></a>
## Maintaining & Troubleshooting Nginx once installed
_The website is throwing a weird error message and telling me that something went wrong! Help!_

OK First things first: Don't panic. This might actually be _normal_ behavior. Read the following guidelines about nginx and mastodon working together.

### If you just restarted mastodon's docker container:
Then this is normal - When nginx see the mastodon port close _after_ it is done setting up its internal proxy it will freak out.
To fix this, just restart nginx:

```sudo systemctl restart nginx.service```

Then refresh the website.

### Make sure Mastodon is actually up and online
You will also want to make sure Mastodon's docker container is online and active. Restarting it should do the trick:

1. ```sudo su - mastodon```
2. ```cd mastodon/```
3. ```docker-compose stop```
4. ```docker-compose up -d```
> The above commands restart the docker container for you. This should really only be done to troubleshoot errors.

### Websites SSL Certificate is invalid!
This is a normal error to see every few months - I am working on putting together another part of this tutorial specifically for this.
To fix the error in the mean time just follow steps _7_ & _8_ under [Installing Nginx](#InstallNginx)

<a name="NeedHelp"></a>
## Need help with your instance?

<p>You can contact me on <a href="https://github.com/mitchellurgero">GitHub</a>, <a href="https://www.facebook.com/mitchell.urgero">Facebook</a>, <a href="https://m.me/mitchell.urgero">Messenger</a>, <a href="mailto:info@urgero.org">email</a>, or <a href="xmpp://mitchell@gamingzone.space">XMPP</a>.</p>

Or you can contact me @ https://gnusocial.me/about/more

Or contact the creator of mastodon himself: https://mastodon.social/about/more
