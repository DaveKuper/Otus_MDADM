 Otus_MDADM
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
  
# Var под mirror
lvremove /dev/testroot/lvroot
vgremove testroot
pvcreate /dev/sdb /dev/sdc
vgcreate varmirror /dev/sdb /dev/sdc
lvcreate -L 950M -m1 -n lvvarmirror varmirror
mkfs.ext4 /dev/varmirror/lvvarmirror
mount /dev/varmirror/lvvarmirror /mnt/
cp -aR /var/* /mnt/
mkdir /tmp/oldvar/
mv /var/* /tmp/oldvar/
umount /mnt/
mount /dev/varmirror/lvvarmirror /var/
Изменяем файл vim /etc/fstab
reboot
# Топ под home
lvcreate -n secondhome -L 2G VolGroup00
mkfs.xfs /dev/VolGroup00/secondhome
mount /dev/VolGroup00/secondhome /mnt/
cp -aR /home/* /mnt/
cd mnt/
rm -rf /home/*
umount /mnt/
mount /dev/VolGroup00/secondhome /home/
vim /etc/fstab

# Снэпшоты
touch /home/test{1..100}
lvcreate -L 2GB -s -n home_snap /dev/VolGroup00/secondhome
rm -f /home/test{1..50}
umount /home
lvconvert --merge /dev/VolGroup00/home_snap
mount /home

# Итог

NAME                             MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                                8:0    0   40G  0 disk
├─sda1                             8:1    0    1M  0 part
├─sda2                             8:2    0    1G  0 part /boot
└─sda3                             8:3    0   39G  0 part
  ├─VolGroup00-LogVol00          253:0    0    8G  0 lvm  /
  ├─VolGroup00-LogVol01          253:1    0  1,5G  0 lvm  [SWAP]
  ├─VolGroup00-secondhome-real   253:2    0    2G  0 lvm
  │ ├─VolGroup00-secondhome      253:3    0    2G  0 lvm  /home
  │ └─VolGroup00-home_snap       253:5    0    2G  0 lvm
  └─VolGroup00-home_snap-cow     253:4    0    2G  0 lvm
    └─VolGroup00-home_snap       253:5    0    2G  0 lvm
sdb                                8:16   0   40G  0 disk
├─varmirror-lvvarmirror_rmeta_0  253:6    0    4M  0 lvm
│ └─varmirror-lvvarmirror        253:10   0  952M  0 lvm  /var
└─varmirror-lvvarmirror_rimage_0 253:7    0  952M  0 lvm
  └─varmirror-lvvarmirror        253:10   0  952M  0 lvm  /var
sdc                                8:32   0   40G  0 disk
├─varmirror-lvvarmirror_rmeta_1  253:8    0    4M  0 lvm
│ └─varmirror-lvvarmirror        253:10   0  952M  0 lvm  /var
└─varmirror-lvvarmirror_rimage_1 253:9    0  952M  0 lvm
  └─varmirror-lvvarmirror        253:10   0  952M  0 lvm  /var
  
  sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
devtmpfs on /dev type devtmpfs (rw,nosuid,seclabel,size=242224k,nr_inodes=60556,mode=755)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,seclabel)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,nodev,seclabel,mode=755)
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,seclabel,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_prio,net_cls)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpuacct,cpu)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
configfs on /sys/kernel/config type configfs (rw,relatime)
/dev/mapper/VolGroup00-LogVol00 on / type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
selinuxfs on /sys/fs/selinux type selinuxfs (rw,relatime)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=32,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=11389)
debugfs on /sys/kernel/debug type debugfs (rw,relatime)
mqueue on /dev/mqueue type mqueue (rw,relatime,seclabel)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,seclabel)
nfsd on /proc/fs/nfsd type nfsd (rw,relatime)
/dev/sda2 on /boot type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
/dev/mapper/VolGroup00-secondhome on /home type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_prio,net_cls)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpuacct,cpu)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
configfs on /sys/kernel/config type configfs (rw,relatime)
/dev/mapper/VolGroup00-LogVol00 on / type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
selinuxfs on /sys/fs/selinux type selinuxfs (rw,relatime)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=32,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=11389)
debugfs on /sys/kernel/debug type debugfs (rw,relatime)
mqueue on /dev/mqueue type mqueue (rw,relatime,seclabel)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,seclabel)
nfsd on /proc/fs/nfsd type nfsd (rw,relatime)
/dev/sda2 on /boot type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
/dev/mapper/VolGroup00-secondhome on /home type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_prio,net_cls)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpuacct,cpu)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
configfs on /sys/kernel/config type configfs (rw,relatime)
/dev/mapper/VolGroup00-LogVol00 on / type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
selinuxfs on /sys/fs/selinux type selinuxfs (rw,relatime)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=32,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=11389)
debugfs on /sys/kernel/debug type debugfs (rw,relatime)
mqueue on /dev/mqueue type mqueue (rw,relatime,seclabel)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,seclabel)
nfsd on /proc/fs/nfsd type nfsd (rw,relatime)
/dev/sda2 on /boot type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
/dev/mapper/VolGroup00-secondhome on /home type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_prio,net_cls)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpuacct,cpu)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
configfs on /sys/kernel/config type configfs (rw,relatime)
/dev/mapper/VolGroup00-LogVol00 on / type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
selinuxfs on /sys/fs/selinux type selinuxfs (rw,relatime)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=32,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=11389)
debugfs on /sys/kernel/debug type debugfs (rw,relatime)
mqueue on /dev/mqueue type mqueue (rw,relatime,seclabel)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,seclabel)
nfsd on /proc/fs/nfsd type nfsd (rw,relatime)
/dev/sda2 on /boot type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
/dev/mapper/VolGroup00-secondhome on /home type xfs (rw,relatime,seclabel,attr2,inode64,noquota)


Filesystem                        Type      Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup00-LogVol00   xfs       8,0G  935M  7,1G  12% /
devtmpfs                          devtmpfs  237M     0  237M   0% /dev
tmpfs                             tmpfs     245M     0  245M   0% /dev/shm
tmpfs                             tmpfs     245M  4,5M  240M   2% /run
tmpfs                             tmpfs     245M     0  245M   0% /sys/fs/cgroup
/dev/sda2                         xfs      1014M   61M  954M   6% /boot
/#dev/mapper/VolGroup00-secondhome xfs       2,0G   33M  2,0G   2% /home

  
  











  

