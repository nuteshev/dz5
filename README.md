# stands-03-lvm

Уменьшение тома под /:
```
[root@localhost ~]# yum install xfsdump -y
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.reconn.ru
 * extras: mirror.reconn.ru
 * updates: mirror.axelname.ru
base                                                                                             | 3.6 kB  00:00:00
extras                                                                                           | 2.9 kB  00:00:00
updates                                                                                          | 2.9 kB  00:00:00
Resolving Dependencies
--> Running transaction check
---> Package xfsdump.x86_64 0:3.1.7-1.el7 will be installed
--> Processing Dependency: attr >= 2.0.0 for package: xfsdump-3.1.7-1.el7.x86_64
--> Running transaction check
---> Package attr.x86_64 0:2.4.46-13.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================== Package                    Arch                      Version                             Repository               Size
========================================================================================================================Installing:
 xfsdump                    x86_64                    3.1.7-1.el7                         base                    308 k
Installing for dependencies:
 attr                       x86_64                    2.4.46-13.el7                       base                     66 k

Transaction Summary
========================================================================================================================Install  1 Package (+1 Dependent package)

Total download size: 374 k
Installed size: 1.1 M
Downloading packages:
(1/2): attr-2.4.46-13.el7.x86_64.rpm                                                             |  66 kB  00:00:01
(2/2): xfsdump-3.1.7-1.el7.x86_64.rpm                                                            | 308 kB  00:00:01
------------------------------------------------------------------------------------------------------------------------Total                                                                                   249 kB/s | 374 kB  00:00:01
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : attr-2.4.46-13.el7.x86_64                                                                            1/2
  Installing : xfsdump-3.1.7-1.el7.x86_64                                                                           2/2
  Verifying  : attr-2.4.46-13.el7.x86_64                                                                            1/2
  Verifying  : xfsdump-3.1.7-1.el7.x86_64                                                                           2/2

Installed:
  xfsdump.x86_64 0:3.1.7-1.el7

Dependency Installed:
  attr.x86_64 0:2.4.46-13.el7

Complete!
[root@localhost ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[root@localhost ~]# vgcreate vg_root /dev/sdb
  Volume group "vg_root" successfully created
[root@localhost ~]# lvcreate -n lv_root -l +100%FREE /dev/vg_root
  Logical volume "lv_root" created.
[root@localhost ~]# mkfs.xfs /dev/vg_root/lv_root
meta-data=/dev/vg_root/lv_root   isize=512    agcount=4, agsize=655104 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2620416, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@localhost ~]#  mount /dev/vg_root/lv_root /mnt
[root@localhost ~]#  xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt
xfsrestore: using file dump (drive_simple) strategy
xfsrestore: version 3.1.7 (dump format 3.0)
xfsdump: using file dump (drive_simple) strategy
xfsdump: version 3.1.7 (dump format 3.0)
xfsdump: level 0 dump of localhost.localdomain:/
xfsdump: dump date: Wed Nov 11 09:17:52 2020
xfsdump: session id: eb908885-1e15-40b6-99e9-14d7e24a250b
xfsdump: session label: ""
xfsrestore: searching media for dump
xfsdump: ino map phase 1: constructing initial dump list
xfsdump: ino map phase 2: skipping (no pruning necessary)
xfsdump: ino map phase 3: skipping (only one dump stream)
xfsdump: ino map construction complete
xfsdump: estimated dump size: 806423680 bytes
xfsdump: creating dump session media file 0 (media 0, file 0)
xfsdump: dumping ino map
xfsdump: dumping directories
xfsrestore: examining media file 0
xfsrestore: dump description:
xfsrestore: hostname: localhost.localdomain
xfsrestore: mount point: /
xfsrestore: volume: /dev/mapper/VolGroup00-LogVol00
xfsrestore: session time: Wed Nov 11 09:17:52 2020
xfsrestore: level: 0
xfsrestore: session label: ""
xfsrestore: media label: ""
xfsrestore: file system id: b60e9498-0baa-4d9f-90aa-069048217fee
xfsrestore: session id: eb908885-1e15-40b6-99e9-14d7e24a250b
xfsrestore: media id: 689ec671-b9d0-4994-bdbe-ef2ab90936f0
xfsrestore: searching media for directory dump
xfsrestore: reading directories
xfsdump: dumping non-directory files
xfsrestore: 2689 directories and 23460 entries processed
xfsrestore: directory post-processing
xfsrestore: restoring non-directory files
xfsdump: ending media file
xfsdump: media file size 783640008 bytes
xfsdump: dump size (non-dir files) : 770550112 bytes
xfsdump: dump complete: 37 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 37 seconds elapsed
xfsrestore: Restore Status: SUCCESS
[root@localhost ~]#  for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
[root@localhost ~]# chroot /mnt/
[root@localhost /]#  grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done
[root@localhost /]#  cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;
> s/.img//g"` --force; done
Executing: /sbin/dracut -v initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64 --force
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'mdraid' will not be installed, because command 'mdadm' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'mdraid' will not be installed, because command 'mdadm' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
[root@localhost boot]# sed -i 's/rd.lvm.lv=VolGroup00\/LogVol00/rd.lvm.lv=vg_root\/lv_root/g' /boot/grub2/grub.cfg
[root@localhost boot]# exit
[root@localhost ~]# reboot
```

После перезагрузки: 
```
[root@localhost ~]#  lvremove /dev/VolGroup00/LogVol00
Do you really want to remove active logical volume VolGroup00/LogVol00? [y/n]: y
  Logical volume "LogVol00" successfully removed
[root@localhost ~]#  lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00
WARNING: xfs signature detected on /dev/VolGroup00/LogVol00 at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/VolGroup00/LogVol00.
  Logical volume "LogVol00" created.
[root@localhost ~]# mkfs.xfs /dev/VolGroup00/LogVol00
meta-data=/dev/VolGroup00/LogVol00 isize=512    agcount=4, agsize=524288 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2097152, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@localhost ~]# mount /dev/VolGroup00/LogVol00 /mnt
[root@localhost ~]#  xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt
xfsdump: using file dump (drive_simple) strategy
xfsdump: version 3.1.7 (dump format 3.0)
xfsrestore: using file dump (drive_simple) strategy
xfsrestore: version 3.1.7 (dump format 3.0)
xfsdump: level 0 dump of localhost.localdomain:/
xfsdump: dump date: Wed Nov 11 09:43:11 2020
xfsdump: session id: 7bb6ab97-b98e-4478-8b75-d06299394a98
xfsdump: session label: ""
xfsrestore: searching media for dump
xfsdump: ino map phase 1: constructing initial dump list
xfsdump: ino map phase 2: skipping (no pruning necessary)
xfsdump: ino map phase 3: skipping (only one dump stream)
xfsdump: ino map construction complete
xfsdump: estimated dump size: 806479168 bytes
xfsdump: creating dump session media file 0 (media 0, file 0)
xfsdump: dumping ino map
xfsdump: dumping directories
xfsrestore: examining media file 0
xfsrestore: dump description:
xfsrestore: hostname: localhost.localdomain
xfsrestore: mount point: /
xfsrestore: volume: /dev/mapper/vg_root-lv_root
xfsrestore: session time: Wed Nov 11 09:43:11 2020
xfsrestore: level: 0
xfsrestore: session label: ""
xfsrestore: media label: ""
xfsrestore: file system id: fb0e4453-fcf2-4bb4-a64c-c21b8f32a3dc
xfsrestore: session id: 7bb6ab97-b98e-4478-8b75-d06299394a98
xfsrestore: media id: 8c09e6cf-49cb-4c5f-a87c-188955e24a35
xfsrestore: searching media for directory dump
xfsrestore: reading directories
xfsdump: dumping non-directory files
xfsrestore: 2693 directories and 23467 entries processed
xfsrestore: directory post-processing
xfsrestore: restoring non-directory files
xfsdump: ending media file
xfsdump: media file size 783824520 bytes
xfsdump: dump size (non-dir files) : 770729616 bytes
xfsdump: dump complete: 24 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 24 seconds elapsed
xfsrestore: Restore Status: SUCCESS
[root@localhost ~]#  for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
[root@localhost ~]#  chroot /mnt/
[root@localhost /]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done
[root@localhost /]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;
> s/.img//g"` --force; done
Executing: /sbin/dracut -v initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64 --force
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'mdraid' will not be installed, because command 'mdadm' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'mdraid' will not be installed, because command 'mdadm' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
[root@localhost boot]# pvcreate /dev/sdc /dev/sdd
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
[root@localhost boot]#  vgcreate vg_var /dev/sdc /dev/sdd
  Volume group "vg_var" successfully created
[root@localhost boot]#  lvcreate -L 950M -m1 -n lv_var vg_var
  Rounding up size to full physical extent 952.00 MiB
  Logical volume "lv_var" created.
[root@localhost boot]#  mkfs.ext4 /dev/vg_var/lv_var
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
60928 inodes, 243712 blocks
12185 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=249561088
8 block groups
32768 blocks per group, 32768 fragments per group
7616 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

[root@localhost boot]# mount /dev/vg_var/lv_var /mnt
[root@localhost boot]# cp -aR /var/* /mnt/
[root@localhost boot]# rsync -avHPSAX /var/ /mnt/
sending incremental file list
./
.updated
            163 100%    0.00kB/s    0:00:00 (xfr#1, ir-chk=1026/1028)

sent 129,662 bytes  received 565 bytes  260,454.00 bytes/sec
total size is 186,499,483  speedup is 1,432.11
[root@localhost boot]# mkdir /tmp/oldvar && mv /var/* /tmp/oldvar
[root@localhost boot]# umount /mnt
[root@localhost boot]# mount /dev/vg_var/lv_var /var
[root@localhost boot]# echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab
[root@localhost boot]# exit
exit
[root@localhost ~]# reboot
```

После второй перезагрузки: 
```
[root@localhost ~]# lvremove /dev/vg_root/lv_root
Do you really want to remove active logical volume vg_root/lv_root? [y/n]: y
  Logical volume "lv_root" successfully removed
[root@localhost ~]# vgremove /dev/vg_root
  Volume group "vg_root" successfully removed
[root@localhost ~]# pvremove /dev/sdb
  Labels on physical volume "/dev/sdb" successfully wiped.
[root@localhost ~]# lvcreate -n LogVol_Home -L 2G /dev/VolGroup00
  Logical volume "LogVol_Home" created.
[root@localhost ~]# mkfs.xfs /dev/VolGroup00/LogVol_Home
meta-data=/dev/VolGroup00/LogVol_Home isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@localhost ~]# mount /dev/VolGroup00/LogVol_Home /mnt/
[root@localhost ~]# cp -aR /home/* /mnt/
[root@localhost ~]# rm -rf /home/*
[root@localhost ~]# umount /mnt
[root@localhost ~]# mount /dev/VolGroup00/LogVol_Home /home/
[root@localhost ~]# echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab
[root@localhost ~]# touch /home/file{1..20}
[root@localhost ~]# lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home
  Rounding up size to full physical extent 128.00 MiB
  Logical volume "home_snap" created.
[root@localhost ~]# rm -f /home/file{11..20}
[root@localhost ~]# ls /home
file1  file10  file2  file3  file4  file5  file6  file7  file8  file9  vagrant
[root@localhost ~]# umount /home
[root@localhost ~]# lvconvert --merge /dev/VolGroup00/home_snap
  Merging of volume VolGroup00/home_snap started.
  VolGroup00/LogVol_Home: Merged: 100.00%
[root@localhost ~]# mount /home
[root@localhost ~]# ls /home
file1  file10  file11  file12  file13  file14  file15  file16  file17  file18  file19  file2  file20  file3  file4  file5  file6  file7  file8  file9  vagrant
```

Создаём раздел для /opt:
```
[root@localhost ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[root@localhost ~]# vgcreate VolGroupOpt /dev/sdb
  Volume group "VolGroupOpt" successfully created
[root@localhost ~]# lvcreate -n LogVol_Opt -L 8G /dev/VolGroupOpt
WARNING: xfs signature detected on /dev/VolGroupOpt/LogVol_Opt at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/VolGroupOpt/LogVol_Opt.
  Logical volume "LogVol_Opt" created.
[root@localhost ~]# mkfs.btrfs /dev/VolGroupOpt/LogVol_Opt
btrfs-progs v4.9.1
See http://btrfs.wiki.kernel.org for more information.

Label:              (null)
UUID:               fb0984ca-0778-494e-a055-507cab4796d4
Node size:          16384
Sector size:        4096
Filesystem size:    8.00GiB
Block group profiles:
  Data:             single            8.00MiB
  Metadata:         DUP             409.56MiB
  System:           DUP               8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  1
Devices:
   ID        SIZE  PATH
    1     8.00GiB  /dev/VolGroupOpt/LogVol_Opt

[root@localhost ~]# mount /dev/VolGroupOpt/LogVol_Opt /mnt
[root@localhost ~]# cp -aR /opt/* /mnt
cp: cannot stat ‘/opt/*’: No such file or directory
[root@localhost ~]# ls /opt
[root@localhost ~]# umount /mnt
[root@localhost ~]# mount /dev/VolGroupOpt/LogVol_Opt /opt
[root@localhost ~]# echo "`blkid | grep Opt | awk '{print $2}'` /opt btrfs defaults 0 0" >> /etc/fstab
```

Создаём том для снэпшота: 
```
[root@localhost opt]# lvcreate -L 100MB -s -n opt_snap /dev/VolGroupOpt/LogVol_Opt
  Logical volume "opt_snap" created.
```

Создаём том под кэш: 
```
[root@localhost opt]# lvconvert --type cache --cachepool VolGroupOpt/lv_cache VolGroupOpt/LogVol_Opt
  WARNING: Converting VolGroupOpt/lv_cache to cache pool's data volume with metadata wiping.
  THIS WILL DESTROY CONTENT OF LOGICAL VOLUME (filesystem etc.)
Do you really want to convert VolGroupOpt/lv_cache? [y/n]: y
  Converted VolGroupOpt/lv_cache to cache pool.
  Logical volume VolGroupOpt/LogVol_Opt is now cached.
```

Перезагружаемся и видим следующую картину: 
```
[vagrant@localhost ~]$ lsblk
NAME                            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                               8:0    0   40G  0 disk
├─sda1                            8:1    0    1M  0 part
├─sda2                            8:2    0    1G  0 part /boot
└─sda3                            8:3    0   39G  0 part
  ├─VolGroup00-LogVol00         253:0    0    8G  0 lvm  /
  ├─VolGroup00-LogVol01         253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol_Home      253:14   0    2G  0 lvm  /home
sdb                               8:16   0   10G  0 disk
├─VolGroupOpt-lv_cache_cdata    253:5    0    1G  0 lvm
│ └─VolGroupOpt-LogVol_Opt-real 253:10   0    8G  0 lvm
│   ├─VolGroupOpt-LogVol_Opt    253:11   0    8G  0 lvm  /opt
│   └─VolGroupOpt-opt_snap      253:13   0    8G  0 lvm
├─VolGroupOpt-lv_cache_cmeta    253:6    0    8M  0 lvm
│ └─VolGroupOpt-LogVol_Opt-real 253:10   0    8G  0 lvm
│   ├─VolGroupOpt-LogVol_Opt    253:11   0    8G  0 lvm  /opt
│   └─VolGroupOpt-opt_snap      253:13   0    8G  0 lvm
├─VolGroupOpt-LogVol_Opt_corig  253:7    0    8G  0 lvm
│ └─VolGroupOpt-LogVol_Opt-real 253:10   0    8G  0 lvm
│   ├─VolGroupOpt-LogVol_Opt    253:11   0    8G  0 lvm  /opt
│   └─VolGroupOpt-opt_snap      253:13   0    8G  0 lvm
└─VolGroupOpt-opt_snap-cow      253:12   0  100M  0 lvm
  └─VolGroupOpt-opt_snap        253:13   0    8G  0 lvm
sdc                               8:32   0    2G  0 disk
├─vg_var-lv_var_rmeta_0         253:2    0    4M  0 lvm
│ └─vg_var-lv_var               253:9    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_0        253:3    0  952M  0 lvm
  └─vg_var-lv_var               253:9    0  952M  0 lvm  /var
sdd                               8:48   0    1G  0 disk
├─vg_var-lv_var_rmeta_1         253:4    0    4M  0 lvm
│ └─vg_var-lv_var               253:9    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_1        253:8    0  952M  0 lvm
  └─vg_var-lv_var               253:9    0  952M  0 lvm  /var
sde                               8:64   0    1G  0 disk
```