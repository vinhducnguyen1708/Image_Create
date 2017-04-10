## Xử lý image sau khi đã cài xong OS
#### <i>Chú ý: </i>
 - Hướng dẫn này dành cho các image được cấu hình sẵn LVM cho các phân vùng /boot, /root, /swap
 - Sử dụng hướng dẫn này sau khi đã cài đặt xong OS trên image
 - Sử dụng công cụ `virt-manager` để kết nối tới console máy ảo

## 1. Thực hiện trên máy ảo
### 1.1 Cài parted, đây là công cụ để xử lý các phân vùng của ổ đĩa (xem thêm ở ![đây] (http://manpages.ubuntu.com/manpages/wily/man8/parted.8.html) ) :
```
apt-get install parted -y
```

### 1.2. Đặt script sau tại `/root/partresize.sh`, chạy sau khi boot máy ảo để phân vùng lại LVM:
```
wget https://raw.githubusercontent.com/longsube/Image_Create/master/OpenStack%20Images/partresize.sh
```

### 1.3. Để sau khi boot máy ảo, có thể nhận đủ các NIC gắn vào:
```
apt-get install netplug
wget https://raw.githubusercontent.com/longsube/Netplug-config/master/netplug
```

### 1.4. Đưa file netplug vào thư mục /etc/netplug
```
mv netplug /etc/netplug/netplug
chmod +x /etc/netplug/netplug
```

### 1.5. Chỉnh sửa file `/etc/default/grub` để bắn log ra trong quá trình tạo máy ảo
```
GRUB_DEFAULT=0
#GRUB_HIDDEN_TIMEOUT=0
GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_TIMEOUT=2
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT=""
GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200n8"
```

Sau đó chạy lệnh
`update-grub`

### 1.6. Xóa toàn bộ các thông tin về địa chỉ MAC của card mạng ảo:
```
/etc/udev/rules.d/70-persistent-net.rules
/lib/udev/rules.d/75-persistent-net-generator.rules
```
Chú ý: không xóa 2 file này mà chỉ xóa nội dung 

### 1.7. Để có thể chèn password khi tạo máy ảo, cài các gói sau:
```
apt-get install  cloud-init -y
```
Sửa `/etc/cloud/cloud.cfg` để password hoặc SSH key chèn vào user root:
```
disable_root: false
```

### 1.8. Disable default config route
```
Comment dòng `link-local 169.254.0.0` trong `/etc/networks`
```

##### Tắt máy ảo 
```
init 0
```

## 2. Thực hiện trên Host KVM
### 2.1. Cài đặt bộ libguestfs-tools để xử lý image (nên cài đặt trên Ubuntu OS để có bản libguestfs mới nhất)
```
apt-get install libguestfs-tools -y
```

### 2.2. Xử dụng lệnh `virt-sysprep` để xóa toàn bộ các thông tin máy ảo:
```
virt-sysprep -a U.1404.img
```
### 2.3. Dùng lệnh sau để tối ưu kích thước image:
```
virt-sparsify --compress U.1404.img U.1404.shrink.img
```
### 2.4. Image <b>U.1404.shrink.img</b> đã có thể upload lên Glance

## 3. Boot máy ảo (Trên OpenStack)
### 3.1. Sau khi Login vào máy ảo, kiểm tra dung lượng LVM:
![Kích thước LV](http://image.prntscr.com/image/35eaf4c55f9c4e7398083fa28551ccee.jpg)

### 3.2. Chạy đoạn Script để tăng kích thước LV:
![Script resize LV](http://image.prntscr.com/image/4288388e712b45cb90e734ed00420421.jpg)
#### Các thông số:
- `-p`: chỉ định phân vùng vật lý đặt LV cần resize
- `-l`: chỉ định LV cần resize
- `-f`: mở rộng LV ngay mà không cần rescan (nếu filesystem đã nhận được dung lượng mới, trong hình là 40GB)
Sau khi chạy Script, VM sẽ khởi động lại, quá trình định dạng lại LV sẽ thực hiện khi reboot, thông qua Script `fsresize.sh` được gài vào `/etc/rc.local`

### 3.3. Kiểm tra lại kích thước LV:
![Kiểm tra LV](http://image.prntscr.com/image/2c367df6099f4764986c0ed208cab025.jpg)

##### Done




