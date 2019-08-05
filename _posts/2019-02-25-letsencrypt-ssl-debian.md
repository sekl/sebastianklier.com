---
title: "Let's Encrypt SSL certificate on Debian with Certbot"
date: "2019-02-25T14:30:00.284Z"
layout: post
categories: [server]
comments: true
---

Today I have a quick tutorial for you on how to set up HTTPS/SSL using [Let's Encrypt](https://letsencrypt.org/) and [Certbot](https://certbot.eff.org/). 

<!--more-->

The system I use in this example is an older Debian installation and I won't be a using a package manager, but the same can be done on Ubuntu or newer Debian releases. You can find instructions for many other system on the [Certbot website](https://certbot.eff.org/all-instructions/).

**First of all make sure that your firewall allows traffic through port 443.**

If you are using iptables:
``` bash
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

Or if you are using UFW:
``` bash
sudo ufw allow https
```  
      
<br />**Next let's get `certbot-auto` from the official source:**
``` bash
wget https://dl.eff.org/certbot-auto
```

And make it executable:
``` bash
chmod a+x certbot-auto
```

<br />The next step will differ depending on your webserver. If you are using Apache, just substitute the `--nginx` flag in this command for `--apache`. 

``` bash
certbot-auto --nginx
```

*You may need `sudo` for some of these commands, depending on your system.*

This will generate the key files and certificate for you. The script will actually detect the sites you are running on your server and ask you for which of them to generate the certificate. Make sure to make the appropriate choices for your sites, such as choosing www and non-www domains, etc. You can also let certbot modify your webserver config for you, which is very handy.

Once that is done, let's test if the renewal process works as well:

``` bash
certbot-auto renew --dry-run
```

This should pass without problems as well. All that's left to do now is to setup a cronjob to make sure that certbot will periodically try to renew the certificate for you.

Open crontab:

``` bash
crontab -e
```

And add the following line at the bottom of the file:

``` bash
0 0,12 * * * python -c 'import random; import time; time.sleep(random.random() * 3600)' && /path/to/certbot-auto renew
```

Save it, confirming to write to a temporary file. This is normal when making changes to crontab.

Now certbot will try to renew twice a day at a random minute of the hour. It won't do anything though unless the certificate is actually up for renewal.

If you visit your site now, it should already be running on HTTPS. Most software will need some adjustments though, such as setting the `SITE_URL` to the new https domain, or fixing links and image paths in your source code and CSS.
