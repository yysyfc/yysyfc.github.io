---
title: Copy to NTFS Partition from Mac
description: copy to NTFS partition from Mac
categories:
  - English 
tags:
 - techtips
---

By default, macOS cannot write to NTFS partitions. This is pretty bad since a
lot of portable hard drives are in that format. I need to archive a large
amount of files from my Mac to a portable hard drive so I looked into this. I
remember in my previous Mac I installed some commercial software that can do
this perfectly, but that was many years ago and costs unnecessary expense. So
here is the free way to do it and also a nice way of copying large files to the
newly mounted NTFS partition.

## Mount The Partition
Mostly I follow the instructions
[here](https://github.com/osxfuse/osxfuse/wiki/NTFS-3G) and use a combination
of NTFS-3G and FUSE for macOS.
  - Download and install from https://osxfuse.github.io/.
  - Install NTFS-3G from Homebrew:
  ```sh
	brew install ntfs-3g
  ```
  Note that you might need to add the newly installed ntfs-3g binary to your
  `PATH`.
  - Find the NTFS disk partition using
	```sh
	diskutil list
	```
  - Remount the NTFS disk partition
	```sh
	sudo umount /Volumes/<THE READ-ONLY NTFS DISK>
	sudo mkdir /Volumes/NTFS
	sudo ntfs-3g /dev/<YOUR NTFS PARTITION> /Volumes/NTFS -olocal -oallow-other
	```
	
That's it! The tutorial referred to above also mentioned something about
auto-mount NTFS partitions. But that involves messing with the macOS system and
can potentially introduce harmful software into the system. So I would
recommend skipping those steps and any time you need to mount an NTFS hard
drive, just manually mount it.

## Copy Files to NTFS Partition
Even though the partition shows up after it is mounted using the above steps,
copying files there is not very stable. When I tried to copy some large files
there, it would often fail with messages that only contain numerical error code
and no readable reason. A better way is to use `rsync`. I mostly followed the
answers in this StackOverflow
[discuss](https://serverfault.com/questions/460500/rsync-show-progress-for-individual-file).
There is a fancy version, but the mostly straightforward solution that does the
thing is
```sh
rsync -avP <PATH TO SOURCE> <PATH TO DEST>
```
You can use wildcard * for the source files and `rsync` will count how many
files there are. It shows a progress bar for each file. As you may notice, the
speed is much lower than the maximum value for a modern portable hard disk. I
guess this is because NTFS-3g and/or FUSE need to handle to the NTFS format at
the software level and there is not much support from the OS.

As a side note, if there are a lot of small files that need to be copied, it
would be more efficient to first compress them into a .zip file or .tar.gz
file.
