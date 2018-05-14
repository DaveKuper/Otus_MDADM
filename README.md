# Otus_MDADM
yum install vim xfsdump -y
pvcreate /dev/sdb
vgcreate testroot /dev/sdb
lvcreate -n lvroot -L 8G /dev/testroot
mkfs.xfs /dev/testroot/lvroot
mount /dev/testroot/lvroot /mnt/
xfsdump -f /tmp/root.dump /dev/VolGroup00/LogVol00
xfsrestore -f /tmp/root.dump /mnt/

# for i in /proc/ /sys/ /dev/ /run/ /boot/;
> do mount --bind $i /mnt/$i;
>done

chroot /mnt/
cd /boot/
#for i in `ls initramfs-*img`;
>do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force;
>done

Редактируем /boot/grub2/grub.cfg rd.lvm.lv=<VG>/<LV>
reboot
lvremove /dev/VolGroup00/LogVol00
lvcreate -n LogVol00 -L 8G /dev/VolGroup00
mkfs.xfs /dev/VolGroup00/LogVol00
mount /dev/VolGroup00/LogVol00 /mnt/
xfsdump -f /tmp/root.dump /dev/testroot/lvroot
xfsrestore -f /tmp/root.dump /mnt/

# for i in /proc/ /sys/ /dev/ /run/ /boot/;
> do mount --bind $i /mnt/$i;
>done

chroot /mnt/
cd /boot/
#for i in `ls initramfs-*img`;
>do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force;
>done
Редактируем /boot/grub2/grub.cfg rd.lvm.lv=<VG>/<LV>
reboot
  
  
  

