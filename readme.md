# m1-gentoo-dotfiles


Experimental, make sure your data is backed up before you use it.



## Environment preparation

Prepare an m1 model mac, make sure Time Machine is turned off and have at least 40G of free space.

If you have used the time machine make sure you have deleted all copies of the time machine

To view all Time Machine backups.

```shell
tmutil listbackups
```

To delete a Time Machine backup.

e.g.

```shell
sudo tmutil delete Y-M-D.backup
```


## Install [Asahi Linux](https://asahilinux.org/), see https://asahilinux.org/2022/03/asahi-linux-alpha-release/ for more


```shell
curl https://alx.sh | sh
```

Asahi Linux Desktop is installed here to facilitate subsequent operations (such as linking to wifi to update the system, etc.)


After following the installation instructions, you can enter the Asahi Linux system. Before you start building Gentoo, make sure you have upgraded Asahi Linux to the latest version with GPU support (https://asahilinux.org/2022/12/gpu-drivers-now-) in-asahi-linux/)


## Build Gentoo Stage4 
After the update is complete we can build the new gentoo system.

> Switch to the root user to do the following


Create the folder for building gentoo's working directory
```shell
mkdir -pv /mnt/gentoo
```

Download stage3 for arm64 https://www.gentoo.org/downloads/
```shell
cd /mnt/gentoo
wget -c https://bouncer.gentoo.org/fetch/root/all/releases/arm64/autobuilds/20230122T234700Z/stage3-arm64-desktop-openrc- 20230122T234700Z.tar.xz
```

Decompress :
```shell
tar xpvf stage3-*.tar.xz --xattrs-include="*. *" --numeric-owner
```

Copy the Asahi Linux kernel, modules and firmware to gentoo

```shell
cp -r /boot /mnt/gentoo
cp -r /usr/lib/modules /mnt/gentoo/usr/lib
cp -r /usr/lib/firmware /mnt/gentoo/usr/lib
```

Hang the necessary files in.


```shell
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
mount --bind /run /mnt/gentoo/run
mount --make-slave /mnt/gentoo/run
```
To get to the chroot.

```shell
chroot /mnt/gentoo /bin/bash
sources /etc/profile
```

Next, you can refer to the [Gentoo Linux manual](https://wiki.gentoo.org/wiki/Handbook:AMD64) to install, it should be noted that USE should be enabled globally wayland, the latest GPU driver works better under wayland.

The installation process can be skipped to the partition, kernel, and boot chapters

You can backup the stage4 file after installing the necessary system configuration

Generate the skipped manifest file (the compression will not compress these contents)
```shell
cat << EOF > /stage4.excl
.bash_history
/mnt/*
/tmp/*
/proc/*
/sys/*
/dev/*
/run/*
/stage4.excl
/stage4.tar.gz
EOF
```
Compressed file.
```shell
tar cvzf /stage4.tar.gz -X /stage4.excl /
```

To exit the chroot environment.
```shell
exit
```

After that we reboot the system

```shell
reboot
```

## Install Gentoo

While rebooting into the grub menu, press e to enter modify mode and delete a few characters from the UUID after ``root=` in the line ``linux /boot/vmlinuz-linux-asahi root=`` and press control+x to enter boot.

This time the boot will fail to a rescue shell

In the rescue shell, do the following.


```shell
mkdir -p /mnt/asahi /mnt/new
```

Create a memory disk and mount it to `/mnt/new`

```shell
mount -t tmpfs -o size=5g tmpfs /mnt/new
```

To view the hard drive partition.

```shell
fdisk -l
```

After seeing the rootfs partition of Asahi Linux mount it to `/mnt/asahi`

```shell
mount /dev/nvme0n1p5 /mnt/asahi
```
Copy the stage4 file you made in Asahi Linux to `/mnt/new`

```shell
cp -v /mnt/asahi/mnt/gentoo/stage4.tar.gz /mnt/new
```

Delete the contents of the original Asahi Linux.

```shell
rm -Rf /mnt/asahi/*
```

Unpack stage4 to the rootfs of Asahi Linux

```shell
tar xpvf /mnt/new/stage4.tar.gz --numeric-owner
```

To get to the chroot.
```shell
chroot /mnt/asahi /bin/bash
```

Delete the original firmware and linked kernel modules and firmware directories: ``shell
```shell
rm -Rf /lib/firmware
ln -s /usr/lib/firmware /lib/
ln -s /usr/lib/modules /lib/
```

Finally reboot the system with.

```shell
reboot -f
```

After that you are in the Gentoo system, Happy hacking!



Reference document: https://blog.devgenius.io/installing-gentoo-linux-in-apple-macbook-pro-m1-49e163534898
