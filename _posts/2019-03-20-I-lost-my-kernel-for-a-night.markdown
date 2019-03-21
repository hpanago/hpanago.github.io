---
layout: post
title:  "I lost my kernel for a night"
date:   2019-03-20 23:29:18 +0200
categories: grub initramfs debian jessie stretch
---

### How it all started

So, today I wanted to upgrade my chromium package from v50.0 to ~60.0 or higher, since Slack dropped support for older versions.

I actually use Slack for work communication so I decided it was worth the trouble.

Since I am extremely lazy on "not so important" things I first tried to do it FrankenDebian style.

So I add Stretch to my sources.lost, right bellow my jessie entris and `apt upgrade`.

Then I do `apt install chromium=version` which obviously has a lot of dependencies, but I press yes anyway without hesitation.

That installation clearly failed epicly, so I go back, remove stretch from my sources.list, `apt update` and then it dawns on me.

I have to dist-upgrade.

Jessie is the old stable after all.

Change my sources.list to full Stretch, `apt update`, `apt-get dist-upgrade`, and go back to work.

But due to some derp moment I end up in my new Stretch environment while having done a *Ctrl-C* once in my dist-upgrade 
and then started it again.

Anyway, dpkg reports an exit status of 0, so I'd say we are good.

Later on, I `shutdown now` and go home.

## The Horror

I reach home and at some point decide to boot it up, and suddenly I am in MemTest page.

Thinking I pressed something wrong, I go back to grub menu, only to realize that no Debian entry was there.

That is where I started panicing. And clearly learned to "never go full retard" while dist-upgrading.

I drop instal grub shell, find my boot partition, but root partition is nowhere.

I had to get myself a LiveCD/usb, whatever, so as to mount my system and recover it.

So, once in Ubuntu Live I open up a terminal and do the following:
    
    mount /dev/sda5

Oh no, it's encrypted. 
Hm, that's why grub mentioned it as "unknown".

Anyway,
```
- udiskctl unlock -b /dev/sda5
- udiskctl mount -b /dev/mapper/encr_hostname_root /mnt
- mount /boot /mnt/boot/
- mount --bind /proc /mnt/proc
- mount --bind /dev /mnt/dev
- mount --bind /sys /mnt/sys
```
and `chroot /mnt/`. 

There I can see I indeed have no linux kernel installed, so I install it, `update-grub2` and reboot.

So, yay, now we have our Debian entry at the grub menu.

### The Horror part 2

But wait, something's not right.

I never get a promt for my LUKS password.

And that is because initramfs can't find my root partition, and instead drops me to an initramfs shell.

Okay, Google helped me so far, we can figure this out too.

All blog posts had one thing in common.
I needed to recreate my initramfs images via `update-initramfs`

But while doing that I still could not reach my root partition.

So the solution to this was the fact that when I unlocked my luks partition I had to specify the name for the unlocked partition,
and it had to be the same as in /etc/crypttab, which resided to my root partition.

So this time I execute the following commands,
 ```
- cryptsetup luksOpen /dev/sda5 sda5_crypt
- vgchange -ay
- mount /dev/mapper/encr_hostname_root /mnt
- mount /dev/sda2 /mnt/boot
- mount -t proc proc /mnt/proc
- mount -o bind /dev /mnt/dev
- chroot /mnt
- update-initramfs -u -k all
```

Finally, that made the trick, I rebooted into my system and it all worked.

Kudos: [Recovering from an unbootable Ubuntu encrypted LVM root partition](https://feeding.cloud.geek.nz/posts/recovering-from-unbootable-ubuntu-encrypted-lvm-root-partition/)
