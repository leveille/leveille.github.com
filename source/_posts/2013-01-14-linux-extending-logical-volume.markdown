---
layout: post
title: "Linux: Extending a Logical Volume"
date: 2013-01-14 12:07
comments: false
categories: Notes Linux VMWare
---

One of the servers I'm working with started running out of disk space recently.  The machine is running on a Ubuntu VM in Windows, about 200 miles from my desk.  I have remote access to the VM, but I don't have access to the virtualization software.  After requesting a disk resize through the IT company that manages our Windows network, here's what I saw:

<!--more-->

```bash
root@outlays:~# df -h
Filesystem                Size  Used Avail Use% Mounted on
/dev/mapper/outlays-root   14G   14G    0G 100% /
...
```

The request I made was to increase the disk space from 14G to ~40G.  That additional space doesn't show up, as it's free space on the disk (it hasn't yet been assigned to a volume).  Running the `vgdisplay` will show the additional space (**Free  PE / Size**):

```bash http://linux.die.net/man/8/vgdisplay
root@outlays:~# vgdisplay
  --- Volume group ---
  VG Name               outlays
  ...
  VG Size               39.76 GiB
  Free  PE / Size       6412 / 25.05 GiB
```

So, the disk was expanded, however I needed to assign that free space to /dev/mapper/outlays-root ... I wasn't sure how.  A bit of Googling brought me to this article: [Extending LVM disks in Linux using VMWare virtual disks](http://www.dbvisit.com/forums/showthread.php?t=343).  The solution seemed to be to extend the /dev/mapper/outlays-root by the amount of freespace available in the volume group.  First, I wanted to check out the /dev/mapper/outlays-root disk:

```bash http://linux.die.net/man/8/parted
root@outlays:~# parted
(parted) print all
...
Model: Linux device-mapper (linear) (dm)
Disk /dev/mapper/outlays-root: 14GB
Sector size (logical/physical): 512B/512B
Partition Table: loop
...
```

And running the `lvdisplay` command shows the following:

```bash http://linux.die.net/man/8/lvdisplay
root@outlays:~# lvdisplay
  --- Logical volume ---
  LV Name                /dev/outlays/root
  VG Name                outlays
  ...
  LV Size                14 GiB
```

So, the assumption I made here was the /dev/mapper/outlays-root maps to /dev/outlays/root.  Alright, I needed to expand /dev/outlays/root.  Expanding the disk was as simple as running the following commands:

```bash http://linux.die.net/man/8/lvextend
lvextend -L+25G /dev/outlays/root
```

```bash http://linux.die.net/man/8/resize2fs
resize2fs /dev/outlays/root
```

Now, here's the amount of disk space I have:

```
root@outlays:~# df -h
Filesystem                Size  Used Avail Use% Mounted on
/dev/mapper/outlays-root   39G  8.7G   28G  24% /
...
```

Much better.
