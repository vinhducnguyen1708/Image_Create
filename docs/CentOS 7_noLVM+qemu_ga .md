## Xử lý image sau khi đã cài OS
#### <i>Chú ý: </i>
 - Hướng dẫn này dành cho các image không sử dụng LVM
 - Sử dụng hướng dẫn này sau khi đã cài đặt xong OS trên image
 - Sử dụng công cụ `virt-manager` để kết nối tới console máy ảo
 - Phiên bản OpenStack sử dụng là Mitaka
 - Hướng dẫn bao gồm 2 phần chính: thực hiện trên máy ảo cài OS và thực hiện trên KVM Host

## 1. Trên Host KVM
### 1.1. Tắt máy ảo

### 1.2. Chỉnh sửa file .xml của máy ảo, bổ sung thêm channel trong `<devices>` (để máy host giao tiếp với máy ảo sử dụng qemu-guest-agent), sau đó save lại
`virsh edit cent7`

với `cent7` là tên máy ảo
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
### 2.1. Cấu hình card eth0 tự động active khi hệ thống boot-up
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

### 2.2. Cài đặt ```cloud-utils-growpart``` để resize đĩa cứng lần đầu boot
```
yum update -y
yum install -y epel-release
yum install cloud-utils-growpart dracut-modules-growroot cloud-init -y
```

### 2.3. Rebuild initrd file
```
rpm -qa kernel | sed 's/^kernel-//'  | xargs -I {} dracut -f /boot/initramfs-{}.img {}
```

### 2.4. Cấu hình grub để  ‘phun’ log ra cho nova (Output của lệnh : nova get-console-output)
vim /boot/grub/grub.conf
Thay phần ```rhgb quiet```
Bằng : ```console=tty0 console=ttyS0,115200n8```

### 2.5. Cấu hình cloud-init, sửa file `/etc/cloud/cloud.cfg` như sau
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

### 2.6. Update selinux 3.13.1 (để có thể change passwd qua qemu-ga)
Tải phiên bản selinux 3.13.1

```
yum install wget -y
wget ftp://fr2.rpmfind.net/linux/fedora/linux/updates/22/x86_64/s/selinux-policy-3.13.1-128.28.fc22.noarch.rpm
wget ftp://fr2.rpmfind.net/linux/fedora/linux/updates/22/armhfp/s/selinux-policy-targeted-3.13.1-128.28.fc22.noarch.rpm
```

Gỡ phiên bản selinux cũ

`yum remove selinux-policy-3.12.1-153.el7_0.11.noarch`

Cài đặt phiên bản selinux 3.13.1

```
rpm -i /root/selinux-policy-3.13.1-128.28.fc22.noarch.rpm
rpm -i /root/selinux-policy-targeted-3.13.1-128.28.fc22.noarch.rpm
```

Kiểm tra lại phiên bản selinux

`rpm -qa | grep -i selinux-policy`

Kết quả:

```
selinux-policy-3.13.1-128.28.fc22.noarch
selinux-policy-targeted-3.13.1-128.28.fc22.noarch
```

### 2.7. Cài đặt qemu-guest-agent
#### *Chú ý: qemu-guest-agent là một daemon chạy trong máy ảo, giúp quản lý và hỗ trợ máy ảo khi cần (có thể cân nhắc việc cài thành phần này lên máy ảo)*
#### *Để có thể thay đổi password máy ảo thì phiên bản qemu-guest-agent phải >= 2.5.0*
```
yum install qemu-guest-agent -y
```
#### Kiểm tra phiên bản `qemu-ga` bằng lệnh:
```
qemu-ga --version
```

Kết quả:
```
QEMU Guest Agent 2.5.0
```

Khởi chạy qemu-guest-agent
```
systemctl start qemu-guest-agent
```

Khởi chạy qemu-guest-agent khi boot máy
```
systemctl enable qemu-guest-agent
```

### 2.8. Disable default config route
```
echo "NOZEROCONF=yes" >> /etc/sysconfig/network
```

### 2.9. Để sau khi boot máy ảo, có thể nhận đủ các NIC gắn vào, chỉnh sửa file `/etc/rc.local` như sau:

`vim /etc/rc.local`

```
#!/bin/bash
for iface in $(ip -o link | cut -d: -f2 | tr -d ' ' | grep ^eth)
do
   test -f /etc/sysconfig/network-scripts/ifcfg-$iface
   if [ $? -ne 0 ]
   then
       touch /etc/sysconfig/network-scripts /ifcfg-$iface
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

## 2. Thực hiện trên Host KVM
### 3.1. Cài đặt bộ libguestfs-tools để xử lý image (nên cài đặt trên Ubuntu OS để có bản libguestfs mới nhất)
```
apt-get install libguestfs-tools -y
```

### 3.2. Xử dụng lệnh `virt-sysprep` để xóa toàn bộ các thông tin máy ảo:
```
virt-sysprep -a CentOS7.img
```

### 3.3. Giảm kích thước image
```
sudo virt-sparsify --compress CentOS7.img CentOS7_shrink.img
```

### 3.4. Upload image lên glance
```
openstack image create CentOS_7 --disk-format qcow2 --container-format bare --public < CentOS7_shrink.img
```

### 3.5. Kiểm tra việc upload image đã thành công hay chưa

![upload image](/images/cent7/cent7_1.jpg)

### 3.6. Chỉnh sửa metadata của image upload
![view metadata](/images/cent7/cent7_2.jpg)

Thêm 2 metadata là 'hw_qemu_guest_agent' và 'os_type', với giá trị tương ứng là `true` và `linux`, sau đó save lại
![update metadata](/images/cent7/cent7_3.jpg)

### 3.7. Image đã sẵn sàng để launch máy ảo.

## 4. Thử nghiệm việc đổi password máy ảo (sau khi đã tạo máy ảo)
### Cách 1: sử dụng nova API (lưu ý máy ảo phải đang bật)
Trên node Controller, thực hiện lệnh và nhập password cần đổi

```
root@controller1:# nova set-password cent7_qemu_ga
New password: 
Again: 
```
với `cent7_qemu_ga` là tên máy ảo

### Cách 2: sử dụng trực tiếp libvirt
#### Xác định vị trí máy ảo đang nằm trên node compute nào. VD máy ảo đang sử dụng là `cent7_qemu_ga`

`root@controller1:# nova show cent7_qemu_ga`

với `cent7_qemu_ga` là tên máy ảo

Kết quả:
```
+--------------------------------------+----------------------------------------------------------------------------------+
| Property                             | Value                                                                            |
+--------------------------------------+----------------------------------------------------------------------------------+
| OS-DCF:diskConfig                    | MANUAL                                                                           |
| OS-EXT-AZ:availability_zone          | nova                                                                             |
| OS-EXT-SRV-ATTR:host                 | compute3		                                                                  |
| OS-EXT-SRV-ATTR:hostname             | cent7-qemu-ga                                                                  |
| OS-EXT-SRV-ATTR:hypervisor_hostname  | compute3                                                                       |
| OS-EXT-SRV-ATTR:instance_name        | instance-0000008a                                             
```

Như vậy máy ảo nằm trên node compute3 với KVM name là `instance-0000008a`

#### Kiểm tra trên máy compute3 để tìm file socket kết nối tới máy ảo
```
bash -c  "ls /var/lib/libvirt/qemu/*.sock"
```

Kết quả:
```
/var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-0000008a.sock

instance-0000008a: tên của máy ảo trên KVM
```

```
file /var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-0000008a.sock
```

Kết quả:
```
/var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-0000008a.sock: socket
```

#### Kiểm tra kết nối tới máy ảo
```
virsh qemu-agent-command instance-0000008a '{"execute":"guest-ping"}'
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
virsh  qemu-agent-command instance-0000008a '{ "execute": "guest-set-user-password","arguments": { "crypted": false,"username": "root","password": "YQ==" } }'
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

## Done
