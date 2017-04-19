## Xử lý image sau khi đã cài xong OS

#### <i>Chú ý: </i>
 - Hướng dẫn này dành cho các image không sử dụng LVM
 - Sử dụng hướng dẫn này sau khi đã cài đặt xong OS trên image
 - Sử dụng công cụ `virt-manager` để kết nối tới console máy ảo
 - Hướng dẫn sử dụng công cụ qemu-guest-agent (> 2.3.0) cài đặt trong image để thay đổi password từ host KVM
 - Hướng dẫn bao gồm 2 phần chính: thực hiện trên máy ảo cài OS và thực hiện trên KVM Host
 
## 1. Thực hiện trên máy ảo
### 1.1.Để máy ảo khi boot sẽ tự giãn phân vùng theo dung lượng mới, ta cài các gói sau:
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
GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_TIMEOUT=2
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT=""
GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200n8"
```
Sau đó chạy lệnh
`update-grub`

### 1.5. Xóa toàn bộ các thông tin về địa chỉ MAC của card mạng ảo:
```
/etc/udev/rules.d/70-persistent-net.rules
/lib/udev/rules.d/75-persistent-net-generator.rules
```
Chú ý: không xóa 2 file này mà chỉ xóa nội dung 

### 1.6. Disable default config route
```
Comment dòng `link-local 169.254.0.0` trong `/etc/networks`

### 1.7. Cài đặt `qemu-guest-agent`
```
apt-get install qemu-guest-agent
```
#### Kiểm tra phiên bản `qemu-ga` bằng lệnh:
```
qemu-ga --version
```

Kết quả:
```
QEMU Guest Agent 2.5.0
```

### 1.8. Cấu hình thư mục chứa file log của `qemu-ga`
```
mkdir /var/log/qemu-agent
sudo tee /etc/default/qemu-guest-agent > /dev/null <<EOF
#Cấu hình này để qemu-agent thực thi 1 script được đặt tại thư mục /etc/qemu/fsfreeze-hook, script này sẽ bắt các action từ KVM host gửi tới VM thông qua qemu-guest-agent, sau đó tự động phun ra log
DAEMON_ARGS="--logfile /var/log/qemu-agent/org.qemu.guest_agent.0.log --fsfreeze-hook --verbose"
EOF
service qemu-guest-agent restart
```

Kiểm tra bằng lệnh:
```
ls /var/log/qemu-agent/
```

Kết quả:
```
org.qemu.guest_agent.0.log
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

### 2.4. Upload image lên glance
```
openstack image create U14.04_v4 --disk-format qcow2 --container-format bare --public < U.1404.shrink.img
```

### 2.5. Kiểm tra việc upload image đã thành công hay chưa

![upload image](/images/u1404_1.jpg)

### 2.6. Chỉnh sửa metadata của image upload
![view metadata](/images/u1404_2.jpg)

Thêm 2 metadata là 'hw_qemu_guest_agent' và 'os_admin_user', set giá trị là True, sau đó save lại
![update metadata](/images/u1404_3.jpg)

### 2.7. Image đã sẵn sàng để launch máy ảo.

## Done
