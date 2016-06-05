##Xử lý image sau khi đã cài xong OS
####<i>Chú ý: Hướng dẫn này dành cho các image không sử dụng LVM</i>
##1. Xử lý phần OS của máy ảo
#####1.1 Cấu hình card eth0 tự động active khi hệ thống boot-up####
vi /etc/sysconfig/network-script/ifcfg-eth0 :
```
ONBOOT=yes
```

######Xóa 2 dòng :######
```
HWADDR=xx:xx:xx:xx:xx:xx
UUID=.....
```

#####Active network interface######
```
ifup eth0
```

#####1.2 Cài đặt ```cloud-utils-growpart``` để resize đĩa cứng lần đầu boot######
```
rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
yum install cloud-utils-growpart dracut-modules-growroot cloud-init -y
```

#####1.3 Rebuild initrd file
```
dracut -f
```

#####1.4 Cấu hình grub để  ‘phun’ log ra cho nova (Output của lệnh : nova get-console-output)
vim /boot/grub/grub.conf
#####Thay phần ```rhgb quiet```
#####Bằng : ```console=tty0 console=ttyS0,115200n8```

######1.5 Cấu hình cloud-init
vim /etc/cloud/cloud.cfg
```
disable_root: 0
ssh_pwauth:   1
```
#####1.6 Để sau khi boot máy ảo, có thể nhận đủ các NIC gắn vào, chèn script sau vào `/etc/rc.local` với nội dung:
```
for iface in $(ip -o link | cut -d: -f2 | tr -d ' ' | grep ^eth)
doLq
   egrep -q "^ifcfg-$iface" /etc/sysconfig/network-scripts/
   if [ $? -ne 0 ]
   then

       touch ifcfg-$iface
       echo -e "DEVICE=$iface\nBOOTPROTO=dhcp\nONBOOT=yes" > /etc/sysconfig/network-scripts/ifcfg-$iface
      ifup $iface
   fi
done
```

###### Cleaning and Poweroff
```
yum clean all
poweroff
```

##2.Xử lý Image (Trên máy host KVM)
#####2.1 Cài đặt bộ libguestfs-tools để xử lý image (nên cài đặt trên Ubuntu OS để có bản libguestfs mới nhất)
```
apt-get install libguestfs-tools -y
```

#####2.2 Xử dụng lệnh `virt-sysprep` để xóa toàn bộ các thông tin máy ảo:
```
virt-sysprep -a CentOS65.img
```

#####2.3 Giảm kích thước image
```
sudo virt-sparsify --compress CentOS65.img CentOS65_shrink.img
```

#####2.4. Upload lên glance##
Qúa trình tạo template đã xong, bạn upload file centos6.5.cloud.qcow2 lên Openstack là có thể sử dụng được.