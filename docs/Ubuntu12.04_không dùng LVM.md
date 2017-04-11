## Xử lý image sau khi đã cài OS
#### <i>Chú ý: </i>
 - Hướng dẫn này dành cho các image không sử dụng LVM
 - Sử dụng hướng dẫn này sau khi đã cài đặt xong OS trên image
 - Sử dụng công cụ `virt-manager` để kết nối tới console máy ảo

## 1. Thực hiện trên máy ảo
### 1.1. Để máy ảo khi boot sẽ tự giãn phân vùng theo dung lượng mới, ta cài các gói sau:
```
apt-get install cloud-utils cloud-initramfs-growroot cloud-init -y
```
### 1.2. Để sau khi boot máy ảo, có thể nhận đủ các NIC gắn vào:
```
apt-get install netplug -y
wget https://raw.githubusercontent.com/longsube/Netplug-config/master/netplug
```

### 1.3. Đưa file netplug vào thư mục /etc/netplug
```
mv netplug /etc/netplug/netplug
chmod +x /etc/netplug/netplug
```
### 1.4. Chỉnh sửa file `/etc/default/grub` để bắn log ra trong quá trình tạo máy ảo
```
GRUB_DEFAULT=0
#GRUB_HIDDEN_TIMEOUT=0
#GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="console=ttyS0"
GRUB_CMDLINE_LINUX=""
GRUB_TERMINAL=console
```
Sau đó chạy lệnh
`update-grub`

### 1.5. Để điều chỉnh metadata source cho máy ảo khi boot, chạy lệnh:
`dpkg-reconfigure cloud-init`
Chọn EC2 data source
Ở trên máy ảo Ubuntu, account là "ubuntu"

### 1.6. Xóa toàn bộ các thông tin về địa chỉ MAC của card mạng ảo:
```
/etc/udev/rules.d/70-persistent-net.rules
/lib/udev/rules.d/75-persistent-net-generator.rules
```
Chú ý: không xóa 2 file này mà chỉ xóa nội dung 

##### Tắt máy ảo 
```
init 0
```

## 2. Thực hiện trên Host KVM
### 2.1. Xử dụng lệnh `virt-sysprep` để xóa toàn bộ các thông tin máy ảo:
```
virt-sysprep -a U.1204.img
```
### 2.2. Dùng lệnh sau để tối ưu kích thước image:
```
virt-sparsify --compress U.1204.img U.1204.shrink.img
```
##### Image <b>U.1204.shrink.img</b> đã có thể upload lên Glance
