---
layout: post
title:  "Gunicorn and Debian Stretch"
date:   2018-07-07 20:29:18 +0200
categories: gunicorn debian jessie stretch
---


### A story about Gunicorn.

Recently at work I was assigned a task of upgrading one of our services from Debian Jessie to Debian Stretch. And that meant upgrade all the packages related to our service.

One of those was Gunicorn 19.6.0

So I started with our test environment, then our staging environment. Everything was working fine, except from a “misbehavior” in Gunicorn. It wasn’t accepting a configuration file anymore.

I was trying ```/usr/bin/gunicorn --config somefile myapp.wsgi ```

Gunicorn was starting, but not with the arguments in the conf file, it was using it’s defaults.

And it got me thinking, how in the world was it working before the upgrade?

At first let me say that it had to work with the conf file, since that’s the concept around which we have built our Puppet configuration for gunirorn.

So I had to solve that issue, before starting puppetizing changes.

I started by looking at versions. 
```
Gunicorn at Jessie ( backports ) -> 19.6.0-2~bpo8+1

Gunicorn at Stretch -> 19.6.0-10+deb9u1
```
Pretty much the same. But not exactly.

Gunicorn was installed via apt install gunicorn. Debian package.

Immediately I thought to myself, if the -pip install gunicorn- gunicorn works, the python executable, that means the debian package is problematic. 

Needless to say that it did not work.

Same problem with both packages. None of them even looked at the conf file.

I tried to debug it, I googled for days, nothing.

In the meantime I was starting to accept that I would have to change out puppet structure to meet the new gunicorn needs. And I was working towards that.

But one day it just got me frustruted. If it’s not working then I am going to have to mention it upstream! And tell them what..so I started searching again.
Until that time I had not questioned the conf file or the init file (for jessie).

Now at Stretch it was using a systemd service file, so I thought maybe that's a place to start.

After reading the [example config file](https://github.com/benoitc/gunicorn/blob/master/examples/example_config.py) and
the [Debian Changelog](http://metadata.ftp-master.debian.org/changelogs/main/g/gunicorn/gunicorn_19.8.1-2_changelog)
multiple times, I decided to talk to a colleague, get some fresh ideas.

“The configuration file should be a valid Python source file”
To set a parameter, just assign to it. There’s no special syntax."

from [gunicorn docs](http://docs.gunicorn.org/en/stable/configure.html)

Ours wasn’t exactly like that.

It was like
```
CONFIG = {
    'mode': 'wsgi',
    'user': gunicorn,
    'group': gunicorn,
    'args': (
        '--bind=unix://run/gunicorn/socket,
        '--workers=4',
       'myapp.wsgi',
    ),
}

```

So he changed it to 
```
mode: “wsgi”
user = “gunicorn”
group = “gunicorn”
bind = “unix://run/gunicorn/socket”
workers = 4
```
And it worked! 

But that wasn't enough for me.

How in the world was it working before? I just had to know!

So I start reading at the init file for gunicorn at Jessie.

And soon I notice the $HELPER variable.

And that `if $HELPER` then the code would go on to process --conf-file, --log-file and other arguments.

$HELPER was pointing to  /usr/sbin/gunicorn-debian.

The helper script  was using the python config module
```
CONFIG = getattr(module, 'CONFIG', None)
```
hence the ( old ) syntax.
 
Eventually, gunicorn-debian, turns out, was removed entirely! 

So obviously the syntax of the conf file was now changed.

[Enlightening bug report](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=839250)
