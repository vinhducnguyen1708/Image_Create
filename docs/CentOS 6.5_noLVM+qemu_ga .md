## Xử lý image sau khi đã cài OS
#### <i>Chú ý: </i>
 - Hướng dẫn này dành cho các image không sử dụng LVM
 - Sử dụng hướng dẫn này sau khi đã cài đặt xong OS trên image
 - Sử dụng công cụ `virt-manager` để kết nối tới console máy ảo
 - Phiên bản OpenStack sử dụng là Mitaka
 - Hướng dẫn bao gồm 2 phần chính: thực hiện trên máy ảo cài OS và thực hiện trên KVM Host

## 1. Thực hiện trên máy ảo
### 1.1. Cấu hình card eth0 tự động active khi hệ thống boot-up
vim /etc/sysconfig/network-scripts/ifcfg-eth0 :
```
ONBOOT=yes
```

###### Xóa 2 dòng :
```
HWADDR=xx:xx:xx:xx:xx:xx
UUID=.....
```

###### Active network interface
```
ifup eth0
```

### 1.2. Cài đặt ```cloud-utils-growpart``` để resize đĩa cứng lần đầu boot
```
rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
yum update -y
yum install -y epel-release
yum install cloud-utils-growpart dracut-modules-growroot cloud-init -y
```

### 1.3. Rebuild initrd file
```
rpm -qa kernel | sed 's/^kernel-//'  | xargs -I {} dracut -f /boot/initramfs-{}.img {}
```

### 1.4. Cấu hình grub để  ‘phun’ log ra cho nova (Output của lệnh : nova get-console-output)
vim /boot/grub/grub.conf
Thay phần ```rhgb quiet```
Bằng : ```console=tty0 console=ttyS0,115200n8```

### 1.5. Cấu hình cloud-init, sửa file `/etc/cloud/cloud.cfg` như sau
vim /etc/cloud/cloud.cfg
```
disable_root: 0
ssh_pwauth:   1
...
system_info:
  default_user:
    name: root
  distro: rhel
  paths:
    cloud_dir: /var/lib/cloud
    templates_dir: /etc/cloud/templates
  ssh_svcname: sshd
```

### 1.6. Để sau khi boot máy ảo, có thể nhận đủ các NIC gắn vào:
```
yum install netplug -y
yum install wget -y
wget https://raw.githubusercontent.com/longsube/Netplug-config/master/netplug_cent6.5 -O netplug
```

### 1.7. Đưa file netplug vào thư mục /etc/netplug
```
mv netplug /etc/netplug.d/netplug
chmod +x /etc/netplug.d/netplug
```

### 1.8. Disable default config route
```
echo "NOZEROCONF=yes" >> /etc/sysconfig/network
```


###### Cleaning and Poweroff
```
yum clean all
poweroff
```

## 2. Thực hiện trên Host KVM
### 2.1. Cài đặt bộ libguestfs-tools để xử lý image (nên cài đặt trên Ubuntu OS để có bản libguestfs mới nhất)
```
apt-get install libguestfs-tools -y
```

### 2.2. Xử dụng lệnh `virt-sysprep` để xóa toàn bộ các thông tin máy ảo:
```
virt-sysprep -a CentOS65.img
```

### 2.3. Giảm kích thước image
```
sudo virt-sparsify --compress CentOS65.img CentOS65_shrink.img
```

### 2.4. Upload image lên glance
```
openstack image create CentOS_6.5 --disk-format qcow2 --container-format bare --public < CentOS65_shrink.img
```

### 2.5. Kiểm tra việc upload image đã thành công hay chưa

![upload image](/images/cent6.5/cent65_1.jpg)

### Image đã sẵn sàng để launch máy ảo.

## Done
