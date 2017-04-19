## Xử lý image sau khi đã cài xong OS
#### <i>Chú ý: </i>
 - Hướng dẫn này dành cho các image được cấu hình sẵn LVM cho các phân vùng /boot, /root, /swap
 - Sử dụng hướng dẫn này sau khi đã cài đặt xong OS trên image
 - Sử dụng công cụ `virt-manager` để kết nối tới console máy ảo
 - Hướng dẫn bao gồm 2 phần chính: thực hiện trên máy ảo cài OS và thực hiện trên KVM Host


## 1. Thực hiện trên máy ảo
### 1.1. Cấu hình card eth0 tự động active khi hệ thống boot-up
vim /etc/sysconfig/network-script/ifcfg-eth0 :
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

### 1.2. Đặt script sau tại `/root/partresize.sh`, chạy sau khi boot máy ảo để phân vùng lại LVM:
```
wget https://raw.githubusercontent.com/longsube/Image_Create/master/OpenStack%20Images/partresize.sh
```
###### Cấu hình cloud-init, sửa file `/etc/cloud/cloud.cfg` như sau
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

### 1.3. Để có thể chèn password khi tạo máy ảo, cài các gói sau:
```
yum install  cloud-init -y
```

### 1.4. Cấu hình grub để  ‘phun’ log ra cho nova (Output của lệnh : nova get-console-output)
vim /boot/grub/grub.conf
Thay phần ```rhgb quiet```
Bằng : ```console=tty0 console=ttyS0,115200n8

### 1.5. Cài parted, đây là công cụ để xử lý các phân vùng của ổ đĩa (xem thêm ở ![đây] (http://manpages.ubuntu.com/manpages/wily/man8/parted.8.html) ) :
```
yum install parted -y
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

![upload image](/images/cent65_1.jpg)

### 2.6. Chỉnh sửa metadata của image upload
![view metadata](/images/cent65_2.jpg)

Thêm 2 metadata là 'hw_qemu_guest_agent' và 'os_admin_user', set giá trị là True, sau đó save lại
![update metadata](/images/cent65_3.jpg)

### 2.7. Image đã sẵn sàng để launch máy ảo.

## 3. Boot máy ảo (Trên OpenStack)
### 3.1. Sau khi Login vào máy ảo, kiểm tra dung lượng LVM:
![Kích thước LV](http://image.prntscr.com/image/01d22f9e01e446bc8af7808ca8f42215.jpg)

### 3.2. Chạy đoạn Script để tăng kích thước LV:
![Script resize LV](http://image.prntscr.com/image/a9865f3312fa4ebe9930e66de311c196.jpg)
#### Các thông số:
- `-p`: chỉ định phân vùng vật lý đặt LV cần resize
- `-l`: chỉ định LV cần resize
- `-f`: mở rộng LV ngay mà không cần rescan (nếu filesystem đã nhận được dung lượng mới, trong hình là 40GB)
Sau khi chạy Script, VM sẽ khởi động lại, quá trình định dạng lại LV sẽ thực hiện khi reboot, thông qua Script `fsresize.sh` được gài vào `/etc/rc.local`

### 3.3. Kiểm tra lại kích thước LV:
![Kiểm tra LV](http://image.prntscr.com/image/c3402824909a41a29a5b4e74e8f367eb.jpg)

## Done