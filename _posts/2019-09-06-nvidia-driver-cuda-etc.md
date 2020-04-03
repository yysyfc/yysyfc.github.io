---
title: Install Nvidia driver, CUDA, and CuDNN
description: quick tips about intsalling Nvidia driver, CUDA, and CuDNN
categories:
  - Blog
tags:
  - techtips
---
This post is to record the best way I found to install Nvidia driver, CUDA and
CuDNN on Ubuntu. It is probably more useful for updating those softwares than
installing them for the first time. From time to time, we find ourselves in a
situation where we need to update them because some other dependencies have
moved to the next versions. For example, I just found out the nightly builds of
Tensorflow 2.0 RC have moved to CUDA 10-0-0 from 9 as in the official release
versions. And there are other times I would like to update the libOpenCL
library. So this is just to help me remember how to do it.

I originally learned these things as part of building my own Linux desktop
workstation. I followed the excellent tutorial at
[here](https://blog.slavv.com/the-1700-great-deep-learning-box-assembly-setup-and-benchmarks-148c5ebe6415),
though my actual build is pretty different from the one therein. Lately I found
the part of installing Nvidia drivers and CUDA softwares isn't quite the best in
my opinion. Instead of downloading files and installing them manually, a better
approach would be to use `apt`. It is not only easier to install, but also more
convenient to manage. Different versions of CUDA can be installed in
`/usr/local/` and a symlink is created for you to the one installed last. You
can update the symlink the one you want to use and always have the environment
variables like `PATH` and `LD_LIBRARY_PATH` point to the same symlink. However,
I would recommend only keeping one version on your system to avoid
unexpected/buggy behavior.

## Remove existing drivers/CUDA
So to begin with, remove all the previously installed Nvidia softwares if you
have installed any. I tried to install new versions of CUDA without removing
older versions and got errors like
```sh
dpkg: error processing archive /var/cache/apt/archives/nvidia-418_418.87.00-0ubuntu1_amd64.deb (--unpack)
```
You can probably get away with this particular error using the following command
```sh
sudo dpkg -i --force-overwrite /tmp/apt-dpkg-install-IyEgEg/45-nvidia-418_418.87.00-0ubuntu1_amd64.deb
```
In fact, it is necessary to use it when you do get the above error since at that
point the `apt` install system is stuck in a bad install. Using
```sh
sudo apt --fix-broken install
```
as suggested by `apt` won't fix the problem. Once you fix this problem (or
better still, directly proceed to the following uninstallation), you can use the
following to remoe all existing Nvidia softwares
```sh
sudo apt remove --purge "nvidia*"
sudo apt autoremove
```
and reboot the system.

## Install new versions of drivers/CUDA
If it is not in your APT repository yet, add the following source for drivers
```sh
sudo add-apt-repository ppa:graphics-drivers
```
and then 
```sh
sudo apt update
```
You can search for the drivers available by
```sh
apt search nvidia
```
The correct/latest version can be checked
[here](https://www.nvidia.com/en-us/drivers/unix/). For me, the current long
lived branch version is 430 so I installed
```sh
sudo apt install nvidia-driver-430
```
I first tried `nvidia-418` since that seems to be the binary proprietary version
while `nvidia-driver-430` is the open source metapackage version, but 418 makes
the system graphics clicky so I changed to 430, which seems to be much better.
After that, reboot the system and make sure that the new driver is running by
checking with
```sh
nvidia-smi
```

Then we can install CUDA and CuDNN. Make sure that you know the right version
you want to install. For example, the latest CUDA version is 10-1 while the
current version Tensorflow 2.0 RC looks for 10-0. Then instal them using
```sh
sudo apt install cuda-10-0
sudo apt install libcudnn7
```
**Edit 09/24/2019**: In a recent reinstall of cuda-10-0, I had trouble installing
cuda-10-0, it always complains that the `--configure` step for some libraries
such as cuda-runtime-10-0, cuda-libraries-dev-10-0 cannot be completed. The
reason provided was that cuda-npp-10-0 was not installed. But it is installed.
The way I solved it is to simply clean the cache and reinstall it. After that we
can manually trigue the configuration process and everything becomes sane again.
{: .notice--info}
```sh
sudo apt clean
sudo apt install --reinstall cuda-npp-10-0
sudo dpkg --configure -a
```

Last but not least, don't forget to add the right paths the your `PATH` and
`LD_LIBRARY_PATH` environment variables, such `/usr/loca/cuda/bin` to `PATH` and
`/usr/local/cuda/lib64` to `LD_LIBRARY_PATH`. Then you are good to go ;-)
