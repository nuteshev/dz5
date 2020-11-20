# 1. Попасть в систему без пароля несколькими способами

1. Способ 1. init=/bin/sh

На экране выбора загрузки нажимаем е, потом в строке, начинающейся с linux16, стираем оба параметра вида console=... и в конце дописываем init=/bin/sh, после чего нажимаем Ctrl+X. 

Раздел / примонтирован в режиме read-only. Для перемонтирования его пишем
```
mount -o remount,rw /
```
Далее можно написать mount -a, с тем, чтобы примонтировать также и раздел boot. Для перезагрузки нужно написать 
```
/usr/sbin/reboot -f 
```

2. Способ 2. rd.break

На экране выбора загрузки нажимаем e, потом в строке, начинающейся с linux16, стираем оба параметра вида console=... и добавляем в конце rd.break, после чего нажимаем Ctrl+X. 

Чтобы поменять пароль, выполняем следующие действия: 
```
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
exit
```

После этого нужно перезагрузиться с параметром enforcing=0 и прогнать команду 
```
fixfiles relabel
```

3. Способ 3. rw init=/sysroot/bin/sh

На экране выбора загрузки нажимаем e, потом в строке, начинающейся с linux16, cтираем оба параметра вида console=... и заменяем ro на rw init=/sysroot/bin/sh, после чего нажимаем Ctrl+X.

Далее делаем chroot sysroot и т.д., как в пункте 2. 

# 2. Установить систему с LVM, после чего переименовать VG

```
[root@localhost ~]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g    0
[root@localhost ~]# vgrename VolGroup00 OtusRoot
  Volume group "VolGroup00" successfully renamed to "OtusRoot"
[root@localhost ~]# for file in /etc/fstab /etc/default/grub /boot/grub2/grub.cfg; do sed -i 's/VolGroup00/OtusRoot/g' $file; done
[root@localhost ~]#  mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
Executing: /sbin/dracut -f -v /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64
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
[root@localhost ~]# reboot
```

После перезагрузки видим: 
```
[root@localhost ~]# vgs
  VG       #PV #LV #SN Attr   VSize   VFree
  OtusRoot   1   2   0 wz--n- <38.97g    0
```

# 3. Добавить модуль в initrd

```
mkdir /usr/lib/dracut/modules.d/01test
cd /usr/lib/dracut/modules.d/01test
yum install wget unzip -y
wget https://gist.github.com/lalbrekht/ac45d7a6c6856baea348e64fac43faf0/archive/69598efd5c603df310097b52019dc979e2cb342d.zip
wget https://gist.github.com/lalbrekht/e51b2580b47bb5a150bd1a002f16ae85/archive/80060b7b300e193c187bbcda4d8fdf0e1c066af9.zip
for file in `ls`; do unzip $file; done
cd ac45d7a6c6856baea348e64fac43faf0-69598efd5c603df310097b52019dc979e2cb342d/
mv gistfile1.txt ../test.sh
cd ../e51b2580b47bb5a150bd1a002f16ae85-80060b7b300e193c187bbcda4d8fdf0e1c066af9/
mv gistfile1.txt ../module-setup.sh
cd ..
for file in `ls | tr ' ' '\n' | grep -v .sh`; do rm -rf $file; done
mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```

Проверяем, что модуль включён в образ: 
```
[root@localhost 01test]#  lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
test
```
Перезагружаемся, при загрузке убираем опции rhgb quiet, и видим пингвина при загрузке. 



# 4. Сконфигурировать систему без отдельного раздела с /boot, а только с LVM

1. Устанавливаем пропатченный GRUB: 

```
yum-config-manager --add-repo https://yum.rumyantsev.com/centos/7/x86_64/
echo "sslverify=0" >> /etc/yum.repos.d/yum.rumyantsev.com_centos_7_x86_64_.repo
yum install grub2 -y
```

2. Создаём логический раздел  VolGroup01-lvol0, монтируем его в /mnt и копируем туда всё: 
```
echo -e "n\np\n\n\n\nw\n" | fdisk /dev/sdb
pvcreate /dev/sdb1 --bootloaderareasize 1m
vgcreate VolGroup01 /dev/sdb1
lvcreate -l 100%FREE VolGroup01
mkfs.xfs /dev/mapper/VolGroup01-lvol0
mount /dev/mapper/VolGroup01-lvol0 /mnt
rsync -auHxv --exclude=/proc/* --exclude=/sys/* --exclude=/mnt/* /* /mnt
```

3. Открываем /mnt/etc/fstab, удаляем всё оттуда и пишем 
```
/dev/mapper/VolGroup01-lvol0 /                       xfs     defaults        0 0  
```

4. Создаём новый initrd-образ: 
```
mount --bind /proc /mnt/proc && mount --bind /dev /mnt/dev && mount --bind /sys /mnt/sys && mount --bind /run /mnt/run && chroot /mnt/
dracut -f
exit 
```

5. Генерим новые настройки GRUB:
```
grub2-mkconfig -o /boot/grub2/grub.cfg
grub2-install /dev/sda
grub2-install /dev/sdb
```

6. Перезагружаемся, в меню GRUB появляется опция загрузки с VolGroup01-lvol0, редактируем её, а именно, в строке, начинающейся c linux, добавляем в конце опцию enforcing=0 и загружаемся. 

7. Добавляем первый диск в группу томов и расширяем на него логический раздел: 
```
vgremove OtusRoot
echo -e "d\n\nd\n\nd\n\nn\np\n\n\n\nw\n" | fdisk /dev/sda
pvcreate /dev/sda1 --bootloaderareasize 1m
vgextend VolGroup01 /dev/sda1
lvextend -l +100%FREE /dev/mapper/VolGroup01-lvol0
xfs_growfs /dev/mapper/VolGroup01-lvol0
```

8. Заменяем опции в /etc/default/grub и повторяем пункт 5: 
```
sed -i "s/OtusRoot\/LogVol01/VolGroup01\/lvol0/g" /etc/default/grub
sed -i "s/OtusRoot\/LogVol00/VolGroup01\/lvol0/g" /etc/default/grub
grub2-mkconfig -o /boot/grub2/grub.cfg
grub2-install /dev/sda
grub2-install /dev/sdb
```

9. Настраиваем selinux и перезагружаемся: 
```
fixfiles relabel
reboot
```

10. В итоге получаем следующую картину: 
```
[vagrant@localhost ~]$ lsblk
NAME                 MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda                    8:0    0  40G  0 disk
└─sda1                 8:1    0  40G  0 part
  └─VolGroup01-lvol0 253:0    0  50G  0 lvm  /
sdb                    8:16   0  10G  0 disk
└─sdb1                 8:17   0  10G  0 part
  └─VolGroup01-lvol0 253:0    0  50G  0 lvm  /
sdc                    8:32   0   2G  0 disk
sdd                    8:48   0   1G  0 disk
sde                    8:64   0   1G  0 disk
```