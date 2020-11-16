## Запуск vagrant

Копируем Vagrantfile в каталог 
Открываем консоль 
Переходим в каталог где лежит Vagrantfile 
Выполняем команду vagrant up 
После выполнения вводим команду vagrant ssh 

## Команды для ДЗ

lvremove /dev/otus/small
umount /data/
lvremove /dev/otus/test
vgremove /dev/otus
yum install xfsdump -y
pvcreate /dev/sdb
vgcreate vg_root /dev/sdb
lvcreate -n lv_root -l +100%FREE /dev/vg_root
mkfs.xfs /dev/vg_root/lv_root
mount /dev/vg_root/lv_root /mnt
xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt
for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
chroot /mnt/
grub2-mkconfig -o /boot/grub2/grub.cfg
cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
В файле /boot/grub2/grub.cfg заменил rd.lvm.lv=VolGroup00/LogVol00 на rd.lvm.lv=vg_root/lv_root
Перезагружаем - reboot
Удаляем старую lv - lvremove /dev/VolGroup00/LogVol00
Создаем новую lv с размеров 8gb - lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00
Создаем на ней ФС - mkfs.xfs /dev/VolGroup00/LogVol00
Монтируем /mnt - mount /dev/VolGroup00/LogVol00 /mnt
Копируем все данные из lv_root в /mnt - xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt
кофигурируем grub - for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
chroot /mnt/
grub2-mkconfig -o /boot/grub2/grub.cfg
cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
pvcreate /dev/sdc /dev/sdd
lvcreate -L 950M -m1 -n lv_var vg_var
mkfs.ext4 /dev/vg_var/lv_var
mount /dev/vg_var/lv_var /mnt
cp -aR /var/* /mnt/
rsync -avHPSAX /var/ /mnt/
mkdir /tmp/oldvar && mv /var/* /tmp/oldvar
umount /mnt
mount /dev/vg_var/lv_var /var
echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab
lvremove /dev/vg_root/lv_root
vgremove /dev/vg_root
pvremove /dev/sdb
lvcreate -n LogVol_Home -L 2G /dev/VolGroup00
mkfs.xfs /dev/VolGroup00/LogVol_Home
mount /dev/VolGroup00/LogVol_Home /mnt/
cp -aR /home/* /mnt/
rm -rf /home/*
umount /mnt
mount /dev/VolGroup00/LogVol_Home /home/
echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab
touch /home/file{1..20}
ls -l /home
lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home
rm -f /home/file{11..20}
ls -l /home
umount /home
lvconvert --merge /dev/VolGroup00/home_snap
mount /home
ls -l /home

Установка BTRFS
lvcreate -n LogVol_Opt -L 2G /dev/VolGroup00
mkfs.btrfs /dev/VolGroup00/LogVol_Opt
mount /dev/VolGroup00/LogVol_Opt /mnt/
cp -aR /opt/* /mnt/
rm -rf /opt/*
umount /mnt
echo "`blkid | grep Opt | awk '{print $2}'` /opt btrfs defaults 0 0" >> /etc/fstab
mount /dev/VolGroup00/LogVol_Opt /opt
touch /opt/file{1..20}
lvcreate -L 100MB -s -n opt_snap /dev/VolGroup00/LogVol_Opt
rm -f /opt/file{11..20}
umount /opt
lvconvert --merge /dev/VolGroup00/opt_snap
mount /opt



