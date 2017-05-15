## Xử lý image sau khi đã cài xong OS

#### <i>Chú ý: </i>
 - Hướng dẫn này dành cho các image Ubuntu 14.04 không sử dụng LVM
 - Sử dụng hướng dẫn này sau khi đã cài đặt xong OS trên image
 - Sử dụng công cụ `virt-manager` để kết nối tới console máy ảo
 - Hướng dẫn sử dụng công cụ qemu-guest-agent (> 2.3.0) cài đặt trong image để thay đổi password từ host KVM
 - Hướng dẫn bao gồm 2 phần chính: thực hiện trên máy ảo cài OS và thực hiện trên KVM Host

## 1. Trên Host KVM
### 1.1. Tắt máy ảo

### 1.2. Chỉnh sửa file .xml của máy ảo, bổ sung thêm channel trong `<devices>` (để máy host giao tiếp với máy ảo sử dụng qemu-guest-agent), sau đó save lại
`virsh edit U1404`

với `U1404` là tên máy ảo
```
...
<devices>
 <channel type='unix'>
      <target type='virtio' name='org.qemu.guest_agent.0'/>
      <address type='virtio-serial' controller='0' bus='0' port='1'/>
 </channel>
</devices>
```

### 1.3. Tạo thêm thư mục cho channel vừa tạo và phân quyền cho thư mục đó
```
mkdir -p /var/lib/libvirt/qemu/channel/target
chown -R libvirt-qemu:kvm /var/lib/libvirt/qemu/channel
```

### 1.4. Dùng `vim` để sửa file `/etc/apparmor.d/abstractions/libvirt-qemu`
`vim /etc/apparmor.d/abstractions/libvirt-qemu`

Bổ sung thêm cấu hình sau vào dòng cuối cùng
```
 /var/lib/libvirt/qemu/channel/target/*.qemu.guest_agent.0 rw,
```
#### *Mục đích là phân quyền cho phép libvirt-qemu được đọc ghi các file có hậu tố `.qemu.guest_agent.0` trong thư mục `/var/lib/libvirt/qemu/channel/target`*

Khởi động lại `libvirt` và `apparmor`
```
service libvirt-bin restart
service apparmor reload
```

### 1.5. Bật máy ảo

## 2. Thực hiện trên máy ảo
### 2.1.Để máy ảo khi boot sẽ tự giãn phân vùng theo dung lượng mới, ta cài các gói sau:
```
apt-get install cloud-utils cloud-initramfs-growroot cloud-init -y
```

### 2.2. Để sau khi boot máy ảo, có thể nhận đủ các NIC gắn vào:
```
apt-get install netplug -y
wget https://raw.githubusercontent.com/longsube/Netplug-config/master/netplug
```

### 2.3. Đưa file netplug vào thư mục /etc/netplug
```
mv netplug /etc/netplug/netplug
chmod +x /etc/netplug/netplug
```

### 2.4. Chỉnh sửa file `/etc/default/grub` để bắn log ra trong quá trình tạo máy ảo
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

### 2.5. Xóa toàn bộ các thông tin về địa chỉ MAC của card mạng ảo:
```
/etc/udev/rules.d/70-persistent-net.rules
/lib/udev/rules.d/75-persistent-net-generator.rules
```
Chú ý: không xóa 2 file này mà chỉ xóa nội dung 

### 2.6. Disable default config route

Comment dòng `link-local 169.254.0.0` trong `/etc/networks`

### 2.7. Cài đặt `qemu-guest-agent`
#### *Chú ý: qemu-guest-agent là một daemon chạy trong máy ảo, giúp quản lý và hỗ trợ máy ảo khi cần (có thể cân nhắc việc cài thành phần này lên máy ảo)*
#### *Để có thể thay đổi password máy ảo thì phiên bản qemu-guest-agent phải >= 2.5.0*
```
apt-get install software-properties-common -y
add-apt-repository cloud-archive:mitaka -y
apt-get update
apt-get install qemu-guest-agent -y
```
#### Kiểm tra phiên bản `qemu-ga` bằng lệnh:
```
qemu-ga --version
```

Kết quả:
```
QEMU Guest Agent 2.5.0
```

### 2.8. Khởi chạy qemu-guest-agent
```
service qemu-guest-agent start
```

Kết quả:
```
* qemu-ga is running
```

### 2.9. Cấu hình card mạng tự động active khi hệ thống boot-up

`vim /etc/network/interfaces`

```
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet dhcp
```

##### Tắt máy ảo 
```
init 0
```

## 3. Thực hiện trên Host KVM
### 3.1. Cài đặt bộ libguestfs-tools để xử lý image (nên cài đặt trên Ubuntu OS để có bản libguestfs mới nhất)
```
apt-get install libguestfs-tools -y
```

### 3.2. Xử dụng lệnh `virt-sysprep` để xóa toàn bộ các thông tin máy ảo:
```
virt-sysprep -a U.1404.img
```

### 3.3. Dùng lệnh sau để tối ưu kích thước image:
```
virt-sparsify --compress U.1404.img U.1404.shrink.img
```

### 3.4. Upload image lên glance
```
openstack image create U14.04_v4 --disk-format qcow2 --container-format bare --public < U.1404.shrink.img
```

### 3.5. Kiểm tra việc upload image đã thành công hay chưa

![upload image](/images/u1404/u1404_1.jpg)

### 3.6. Chỉnh sửa metadata của image upload
![view metadata](/images/u1404/u1404_2.jpg)

Thêm 2 metadata là 'hw_qemu_guest_agent' và 'os_type', với giá trị tương ứng là `true` và `linux`, sau đó save lại
![update metadata](/images/u1404/u1404_3.jpg)

### 3.7. Image đã sẵn sàng để launch máy ảo.

## 4. Thử nghiệm việc đổi password máy ảo (sau khi đã tạo máy ảo)
### Cách 1: sử dụng nova API (lưu ý máy ảo phải đang bật)
Trên node Controller, thực hiện lệnh và nhập password cần đổi

```
root@controller1:# nova set-password u1404_qemu_ga
New password: 
Again: 
```
với `u1404_qemu_ga` là tên máy ảo

### Cách 2: sử dụng trực tiếp libvirt
#### Xác định vị trí máy ảo đang nằm trên node compute nào. VD máy ảo đang sử dụng là `U1404_qemu_ga`

`root@controller1:# nova show U1404_qemu_ga`

với `U1404_qemu_ga` là tên máy ảo

Kết quả:
```
+--------------------------------------+----------------------------------------------------------------------------------+
| Property                             | Value                                                                            |
+--------------------------------------+----------------------------------------------------------------------------------+
| OS-DCF:diskConfig                    | MANUAL                                                                           |
| OS-EXT-AZ:availability_zone          | nova                                                                             |
| OS-EXT-SRV-ATTR:host                 | compute2		                                                                  |
| OS-EXT-SRV-ATTR:hostname             | u1404-qemu-ga                                                                  |
| OS-EXT-SRV-ATTR:hypervisor_hostname  | compute2                                                                        |
| OS-EXT-SRV-ATTR:instance_name        | instance-0000001d                                             
```

Như vậy máy ảo nằm trên node compute2 với KVM name là `instance-0000001d`

#### Kiểm tra trên máy compute2 để tìm file socket kết nối tới máy ảo
```
bash -c  "ls /var/lib/libvirt/qemu/*.sock"
```

Kết quả:
```
/var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-0000001d.sock

instance-0000001d: tên của máy ảo trên KVM
```

```
file /var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-0000001d.sock
```

Kết quả:
```
/var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-0000001d.sock: socket
```

#### Kiểm tra kết nối tới máy ảo
```
virsh qemu-agent-command instance-0000001d '{"execute":"guest-ping"}'
```

Kết quả:
```
{"return":{}}
```

#### Sinh password mới `new`
```
echo -n "new" | base64
```

Kết quả:
```
YQ==
```

#### Chèn password mới vào máy ảo, lưu ý máy ảo phải đang bật
```
virsh  qemu-agent-command instance-0000001d '{ "execute": "guest-set-user-password","arguments": { "crypted": false,"username": "root","password": "YQ==" } }'
```

Kết quả;
```
{"return":{}}
```

Thử đăng nhập vào máy ảo với password `new`

## Done

Tham khảo: 

[1] - https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Virtualization_Deployment_and_Administration_Guide/chap-QEMU_Guest_Agent.html

[2] - https://www.sebastien-han.fr/blog/2015/02/09/openstack-perform-consistent-snapshots-with-qemu-guest-agent/