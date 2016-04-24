---
title: Having trouble mounting your exfat drive after you used it on OSX? 
slug: 11-having-trouble-mounting-your-exfat-drive-after-you-used-it-on-osx 
published: true
posted: 2015-06-02
updated: 2014-04-22
---
When trying to mount my external WD 1TB HDD on my linux desktop, I got this nice little error message:

```bash
$ mount -t exfat /dev/sdc2 /media/Ekstern
FUSE exfat 1.0.1
ERROR: `._.com.apple.timemachine.donotpresent' has invalid checksum (0x5292 != 0x4ca2).
```

It seems OSX has screwed up my drive! And a while back I managed to kill the logic board in my Macbook Pro, so both USB ports are dead, which means I can't connect the disk back to unmount it properly.

I have tried mounting the drive on Windows, but Windows just tells me I need to format the drive. This has been a problem for a few months, since I did not want to reformat the drive as I have things on it want to keep.
A few days ago my new ZyXeL NAS arrived, and I needed the drive to begin transfering files. So I began searching for a solution again, and this time I was lucky! It turns out the problem on Linux is with the FUSE exfat driver. 

Sadly FUSE has been the only way of mounting exfat drives on Linux for a very long time, but not any more! Someone brilliant started porting the native exfat/vfs kernel driver from Android 3.0 to normal Linux systems. The kernel driver is called [exfat-nofuse](https://github.com/dorimanx/exfat-nofuse), and it works brilliantly. This also seems to ignore the bad unmounting from OSX, allowing me to mount the drive and get my files out!

Make sure to uninstall the exfat-fuse related packages you have installed, or it won't work.

TL;DR: Remove the exfat related packages for FUSE, install [exfat-nofuse](https://github.com/dorimanx/exfat-nofuse), and you should be able to mount your disk again.
