# Beacon Infrastructure Deployment

## Initial Setup

This documentation contains the Lustre Parallel / Distributed Filesystem setup scripts.
This script assumes that:
  * You use the beacon profile which has IP addresses and links hardcoded
  * The primary hard drive is /dev/sda (this can change sometimes)
  
  
  
## Lustre Server Setup
```
#!/bin/bash
#run this script as ROOT
#and only on: io

#disable SELinux
sed -i '/^SELINUX=/s/.*/SELINUX=disabled/' /etc/selinux/config

#turn off firewall
service iptables stop
chkconfig iptables off

#create the lustre partitions
vgcreate /dev/sda4
lvcreate -L 100G lustre #lvol0
lvcreate -L 700G lustre #lvol1

#make sure we can get the RPMS
yum install -y wget
mkdir lustre-rpms
cd lustre-rpms

#e2fsprogs
wget --no-check-certificate https://downloads.hpdd.intel.com/public/e2fsprogs/latest/el6/RPMS/x86_64/e2fsprogs-1.42.13.wc5-7.el6.x86_64.rpm
wget --no-check-certificate https://downloads.hpdd.intel.com/public/e2fsprogs/latest/el6/RPMS/x86_64/e2fsprogs-libs-1.42.13.wc5-7.el6.x86_64.rpm
wget --no-check-certificate https://downloads.hpdd.intel.com/public/e2fsprogs/latest/el6/RPMS/x86_64/libcom_err-1.42.13.wc5-7.el6.x86_64.rpm
wget --no-check-certificate https://downloads.hpdd.intel.com/public/e2fsprogs/latest/el6/RPMS/x86_64/libss-1.42.13.wc5-7.el6.x86_64.rpm

#lustre
wget --no-check-certificate https://downloads.hpdd.intel.com/public/lustre/lustre-2.7.0/el6/server/RPMS/x86_64/kernel-2.6.32-504.8.1.el6_lustre.x86_64.rpm
wget --no-check-certificate https://downloads.hpdd.intel.com/public/lustre/lustre-2.7.0/el6/server/RPMS/x86_64/kernel-firmware-2.6.32-504.8.1.el6_lustre.x86_64.rpm
wget --no-check-certificate https://downloads.hpdd.intel.com/public/lustre/lustre-2.7.0/el6/server/RPMS/x86_64/lustre-2.7.0-2.6.32_504.8.1.el6_lustre.x86_64.x86_64.rpm
wget --no-check-certificate https://downloads.hpdd.intel.com/public/lustre/lustre-2.7.0/el6/server/RPMS/x86_64/lustre-modules-2.7.0-2.6.32_504.8.1.el6_lustre.x86_64.x86_64.rpm
wget --no-check-certificate https://downloads.hpdd.intel.com/public/lustre/lustre-2.7.0/el6/server/RPMS/x86_64/lustre-osd-ldiskfs-2.7.0-2.6.32_504.8.1.el6_lustre.x86_64.x86_64.rpm
wget --no-check-certificate https://downloads.hpdd.intel.com/public/lustre/lustre-2.7.0/el6/server/RPMS/x86_64/lustre-osd-ldiskfs-mount-2.7.0-2.6.32_504.8.1.el6_lustre.x86_64.x86_64.rpm

#turn yum off for kernel and lustre
echo ‘disable=kern*,lustre*’>>/etc/yum.conf

#remove old deps
yum remove -y libss libcom_err-devel e2fsprogs e2fsprogs-libs

#manually install because yum is tricky
rpm -Uvh kernel-firmware-2.6.32-504.8.1.el6_lustre.x86_64.rpm
rpm -Uvh libcom_err-1.42.13.wc5-7.el6.x86_64.rpm 
rpm -Uvh libss-1.42.12.wc1-7.el6.x86_64.rpm
rpm -Uvh e2fsprogs-libs-1.42.12.wc1-7.el6.x86_64.rpm
rpm -Uvh e2fsprogs-1.42.12.wc1-7.el6.x86_64.rpm 

#regular install the rest
yum install -y kernel-2.6.32-504.8.1.el6_lustre.x86_64.rpm
yum install -y lustre-modules-2.7.0-2.6.32_504.8.1.el6_lustre.x86_64.x86_64.rpm
yum install -y lustre-osd-ldiskfs-mount-2.7.0-2.6.32_504.8.1.el6_lustre.x86_64.x86_64.rpm 
yum install -y lustre-osd-ldiskfs-2.7.0-2.6.32_504.8.1.el6_lustre.x86_64.x86_64.rpm
yum install -y lustre-2.7.0-2.6.32_504.8.1.el6_lustre.x86_64.x86_64.rpm

#allow tcp (local only) for lustre
echo options lnet networks=tcp(p3p1)> /etc/modprobe.d/lnet.conf

#setup ldev to have MDT and OST on this machine
echo $(hostname) - scratch-MDT0000 /dev/lustre/lvol0 >> /etc/ldev.conf
echo $(hostname) - scratch-OST0000 /dev/lustre/lvol1 >> /etc/ldev.conf

#start our services on boot
chkconfig lnet --add
chkconfig lnet on
chkconfig lustre --add
chkconfig lustre on

reboot

#WARNING: Depending on what shell you use, this may not run after reboot and
# may need to be ran manually

#configure lustre
mkfs.lustre --mgs --mdt --fsname=scratch --index=0 /dev/lustre/lvol0
mkfs.lustre --ost --fsname=scratch --index=0 --mgsnode=10.0.0.2@tcp /dev/lustre/lvol1

service lustre start

#done
```

## Lustre Client script
```
#!/bin/bash

#run this script as ROOT
#and only on: home, compute-1, compute-2

#disable SELinux
sed -i '/^SELINUX=/s/.*/SELINUX=disabled/' /etc/selinux/config

#turn off firewall
service iptables stop
chkconfig iptables off

#make our own place
mkdir lustre-rpms
cd lustre-rpms

#its pull o clock
wget --no-check-certificate ftp://ftp.pbone.net/mirror/ftp.scientificlinux.org/linux/scientific/6.2/x86_64/updates/security/kernel-2.6.32-504.8.1.el6.x86_64.rpm
wget --no-check-certificate https://downloads.hpdd.intel.com/public/lustre/lustre-2.7.0/el6/client/RPMS/x86_64/lustre-client-2.7.0-2.6.32_504.8.1.el6.x86_64.x86_64.rpm
wget --no-check-certificate https://downloads.hpdd.intel.com/public/lustre/lustre-2.7.0/el6/client/RPMS/x86_64/lustre-client-modules-2.7.0-2.6.32_504.8.1.el6.x86_64.x86_64.rpm

#we need this EXACT kernel
yum install -y kernel-2.6.32-504.8.1.el6.x86_64.rpm
#ok now install lustre
yum install -y lustre-client*.rpm

#now configure lustre
echo 10.0.0.2@tcp:/scratch scratch defaults,_netdev 0 0 >> /etc/fstab

reboot

#WARNING: Depending on what shell you use, this may not run after reboot and
# may need to be ran manually
mount -t lustre 10.0.0.2@tcp:/scratch /scratch
```
