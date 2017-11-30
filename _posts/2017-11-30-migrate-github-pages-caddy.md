---
layout: post
title: "Guide: Migrating a Blog from Github Pages to Caddy"
author: {{ site.author.name }}
tags:
    - markdown
    - blog
    - servers
---

A few days ago, I decided that I wanted my personal website/blog to be served over HTTPS. Since my website was hosted on Github Pages, there really was no straightforward way to add HTTPS support. I already had a $5/month CentOS VPS with [Vultr](https://www.vultr.com/?ref=7276430) (disclaimer: referral link!), so I thought: "why not just host my website there?".

The first step was to decide on a suitable web server. I settled on [Caddy](https://caddyserver.com/). It is similar in functionality to Apache and Nginx, but with the added benefit of automatic HTTPS *out of the box* through [Let's Encrypt](https://letsencrypt.org/). This is a huge deal for me, especially since my main reason for migrating is HTTPS! It's not all butterflies and unicorns though. Caddy has an unorthodox licensing scheme for an open-source project: it is 100% free personal use, but requires a monthly fee for any form of commercial use.

The remainder of this post will walk you through how to migrate your Github Pages site to your own server running Caddy. The guide assumes that you already have a domain name with Github Pages and that your server/VPS is running CentOS 7 x64 or similar.

## Installing Caddy

Before we can do any sort of migrating, we need to first install some software and tools that are required to build and serve a Jekyll-based (GitHub Pages) site.

First, let's download and install Caddy with the `service` plugin. I will explain why we need this plugin in a bit.

```bash
curl https://getcaddy.com | bash -s personal hook.service
```

Verify that it was installed correctly by running `caddy --version`.

## Installing Jekyll

Next, we'll download and build the latest version of Ruby (2.4.2 as I am writing this guide). Jekyll depends on Ruby, but the version of Ruby bundled with CentOS is typically quite old (for stability reasons). 

```bash
mkdir ~/tools && cd ~/tools
sudo yum groupinstall "Development Tools"
sudo yum install openssl-devel
wget https://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.2.tar.gz && tar xvfvz ruby-2.4.2.tar.gz
cd ruby-2.4.2
./configure
make
sudo make install
```

Now we can install Jekyll and update the Ruby tools. Note that you *might* need to disable the `secure_path` option in `/etc/sudoers` if you get a "gem not found" error; if this error pops up, leave a comment and I will help you out!

```bash
sudo gem update --system
sudo gem install jekyll
```

With that, we have the tools required to host our GitHub Pages website.

## Grab and build your Jekyll site

First, let's create a directory for our websites `/var/www`. We need to change ownership to the current user; replace "user" with your own username on the server.

```bash
sudo mkdir -p /var/www/blog
sudo chown -R /var/www/blog user:user
mkdir /var/www/logs
cd /var/www/blog
```

Next, clone your repo from GitHub. Be sure to replace the URL with your own!

```bash
git clone https://github.com/aksiksi/aksiksi.github.io
```

Finally, we can build the site using Jekyll:

```bash
jekyll build
```

## Remove your domain from GitHub

Go to your repo, then `Settings`, then remove the "Custom Domain" and click "Save". Next, delete the `CNAME` file from your GitHub repo. 

Next, head over to your domain DNS management panel and remove the two `A` records that point to GitHub servers. Finally, add a new `A` record that points to your server's IP address.

At this point, your domain will point to your server instead of GitHub.

## Create a Caddyfile

Now we need to configure Caddy to serve our Jekyll website. A `Caddyfile` contains the configuration for the Caddy server; this includes stuff like which domain points to which folder as well as more advanced features.

Let's create a `Caddyfile` in `/var/www` as follows:

```bash
cd /var/www/
touch Caddyfile
```

Now, copy the following into the `Caddyfile` and save it. Be sure to replace the parts inside "[]" with your own info!

```
[your_domain_name] {
    root /var/www/blog/[your_github_repo_name]/_site
    log /var/www/logs/blog.log
}
```

You can test your server by running `caddy` in the current directory. Visit your website and your content should appear!

One more step remains: we need to create a `systemd` service so that Caddy will run in the background even when we are logged out of the server.

## Systemd service for Caddy

Execute the following commands. Note that the `-service` option is provided by the `hook.service` plugin we downloaded with Caddy.

```bash
sudo caddy -service install -conf /var/www/Caddyfile \
                            -email [your_email]
sudo caddy -service start
``` 

And that's it! If you have any questions about the guide, drop a comment down below and I will be glad to help!
