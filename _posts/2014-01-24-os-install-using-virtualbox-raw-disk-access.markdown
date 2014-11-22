---
layout: post
title: "OS Install using VirtualBox raw disk access"
date: 2014-01-24 13:54
comments: true
categories: 
---

Recently I had a need to create a bootable sd card containing OpenBSD.
Since my day to day machine is a MacBook Air with no cdrom drive this
initially seemed difficult. It turned out to be much easier than I
expected thanks to
[VirtualBox raw disk access](https://www.virtualbox.org/manual/ch09.html#idp15871152).

Raw hard disk access can be dangerous, mainly because there's a risk
you use the wrong device and blow away things you actually need, so be
careful and make sure you know you're using the right physical device.

The disk has to be connected but not mounted and the Mac tries to
repeatedly automount the sdcard, if you encounter an error that may be
the case. The command to unmount is simply:

  `diskutil unmountDisk /dev/disk6`

A raw disk vmdk needs to be created which can then be attached to the
VM to give it direct access to the disk. With Mavericks the disk is
automatically owned by root each time it's mounted but in order to
create the raw disk vmdk your user needs to have full access to the
disk. In my case the device was `/dev/disk6` so it just required
adjusting permissions:

  `sudo chown sme /dev/disk6`

It's not enough to just chown the resulting file, since it's basically
just a pointer to the raw disk the user VirtualBox runs as needs full
device access.

Next to create the raw disk vmdk (the magic command):

  `VBoxManage internalcommands createrawvmdk -filename sdcard.vmdk -rawdisk /dev/disk6`

This creates a tiny file, `sdcard.vmdk`, that can be attached to a
VirtualBox VM to give it full access to `/dev/disk6`. Next I created a
new OpenBSD VM, mounting the OpenBSD 5.4 iso and attached the raw disk
vmdk as the only storage. This is done by selecting 'Choose existing
disk' during the VM creation disk selection phase.

From here I was able to do a standard
[OpenBSD install](http://www.openbsd.org/faq/faq4.html) and the VM saw
the raw disk as the only available HD. After finishing the install I
was able to 'option' boot to select the sdcard as the startup disk and
run OpenBSD from the sdcard on my MacBook Air. 

For more info check out this helpful
[SuperUser post](https://superuser.com/questions/373463/how-to-access-an-sd-card-from-a-virtual-machine)
