#### <i>Chú ý: </i>
 - Sử dụng công cụ `virt-manager` để kết nối tới console máy ảo
 - Hướng dẫn bao gồm 3 phần chính: cài đặt OS máy ảo, thực hiện trên máy ảo cài OS, thực hiện trên KVM Host

## 1. Cài OS cho máy ảo
### 1.1. Trên máy host KVM, tạo file qcow2 cho máy ảo (với windows nên lấy dung lượng 40GB trở lên), đặt tại thư mục /var/www/ftp/images/
```
qemu-img create -f qcow2 /var/www/ftp/images/win2012x64_standard.img  40G
```

### 1.2. Trên máy host KVM, lấy file ISO Windows2k12 Standard và file virio driver (bản stable mới nhất), đây là các driver cho thiết bị ảo, đặt tại thư mục /var/www/ftp/iso/
```
wget -O /var/www/ftp/iso/Win2k12x64_Standard.iso https://drive.google.com/open?id=0B-9dXNs6dxdoR2FHU0dkRE5sUUE
chmod +x /var/www/ftp/iso/Win2k12x64_Standard.iso
wget -O /var/www/ftp/iso/virtio-win.iso https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso
chmod 0755 /var/www/ftp/iso/virtio-win.iso
```
#### *Chú ý: để có thể sử dụng qemu-guest-agent để thay đổi password máy ảo thì phiên bản virio phải >= 0.1.126*

### 1.3. Trên máy host KVM, bật giao diện virt-manager và khởi tạo, khai báo tên máy ảo
![Create VM 1](/images/win2k12_standard/win2k12_1.jpg)

### 1.4. Chọn đường dẫn tới file image tạo ban đầu, và khai báo OS type của máy ảo
![Create VM 2](/images/win2k12_standard/win2k12_2.jpg)

### 1.5. Khai báo CPU và RAM cho máy ảo
![Create VM 3](/images/win2k12_standard/win2k12_3.jpg)

### 1.6. Kích vào lựa chọn "Customize configuration before install" sau đó Finish
![Create VM 4](/images/win2k12_standard/win2k12_4.jpg)

### 1.7.Chỉnh lại "Disk bus" và "Storage Format" của Disk 1
![Create VM 5](/images/win2k12_standard/win2k12_5.jpg)

### 1.8. Lựa chọn "Add hardware", sau đó add thêm CD ROM với ISO Windows 2012 Standard
![Create VM 6](/images/win2k12_standard/win2k12_6.jpg)

### 1.9. Lựa chọn "Add hardware", sau đó add thêm 1 CD ROM trống
![Create VM 6](/images/win2k12_standard/win2k12_15.jpg)

### 1.10. Trong phần "NIC", lựa chọn giải mạng NAT
![Create VM 7](/images/win2k12_standard/win2k12_7.jpg)

### 1.11. Trong phần "Boot Options", chỉnh lại thứ tự boot, sau đó chọn "Begin Installation" để bắt đầu chạy máy ảo
![Create VM 8](/images/win2k12_standard/win2k12_8.jpg)

### 1.12. Tắt máy ảo
![Create VM 9](/images/win2k12_standard/win2k12_9.jpg)

### 1.13. Chỉnh sửa file .xml của máy ảo, bổ sung thêm channel trong <devices> (để máy host giao tiếp với máy ảo sử dụng qemu-guest-agent), sau đó save lại
`virsh edit Win2012`

với `Win2012` là tên máy ảo
```
...
<devices>
 <channel type='unix'>
      <target type='virtio' name='org.qemu.guest_agent.0'/>
      <address type='virtio-serial' controller='0' bus='0' port='1'/>
 </channel>
</devices>
```

### 1.14. Dùng `vim` để sửa file `/etc/apparmor.d/abstractions/libvirt-qemu`
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

### 1.15. Bật máy ảo để cài đặt OS
![Create VM 10](/images/win2k12_standard/win2k12_10.png)

### 1.16. Lựa chọn phiên bản cài đặt (Windows Server 2012 R2 Standard Evaluation GUI)
![Create VM 11](/images/win2k12_standard/win2k12_11.jpg)
![Create VM 12](/images/win2k12_standard/win2k12_12.jpg)

### 1.17. Lựa chọn chỉ cài đặt Windows không tự động Upgrade
![Create VM 13](/images/win2k12_standard/win2k12_13.jpg)

### 1.18. Máy ảo sẽ không tự động load ổ đĩa cứng
![Create VM 14](/images/win2k12_standard/win2k12_14.jpg)

Đưa ISO"virtio-win.iso" vào CD ROM trống đã gắn ban đầu
![Create VM 15](/images/win2k12_standard/win2k12_16.jpg)

Browse tới file ISO vừa đưa vào
![Create VM 16](/images/win2k12_standard/win2k12_17.jpg)

Chọn Driver storage cho Windows 2k12R2
![Create VM 17](/images/win2k12_standard/win2k12_18.jpg)
![Create VM 18](/images/win2k12_standard/win2k12_19.jpg)

### 1.19. Lúc này máy ảo đã nhận ổ đĩa, tiến hành cài đặt OS, làm theo các hướng dẫn để cài như bình thường
![Create VM 19](/images/win2k12_standard/win2k12_20.jpg)
![Create VM 20](/images/win2k12_standard/win2k12_21.jpg)

### 1.20. Sau khi cài xong OS, tắt VM và sửa lại Boot Options, lựa chọn Boot từ Hard Disk và bật máy ảo
![Create VM 21](/images/win2k12_standard/win2k12_22.jpg)

## 2. Xử lý image sau khi đã cài xong OS
### 2.1. Tạo password administrator cho máy ảo
![Create VM 22](/images/win2k12_standard/win2k12_23.jpg)

### 2.1. Vào "Device Manager" để update driver cho NIC, cài đặt Baloon network driver để VM nhận card mạng
![Create VM 23](/images/win2k12_standard/win2k12_24.jpg)
![Create VM 24](/images/win2k12_standard/win2k12_25.jpg)
![Create VM 25](/images/win2k12_standard/win2k12_26.jpg)
![Create VM 26](/images/win2k12_standard/win2k12_27.jpg)
![Create VM 27](/images/win2k12_standard/win2k12_28.jpg)

### 2.2. Kiểm tra lại việc cài đặt Driver cho NIC
![Create VM 28](/images/win2k12_standard/win2k12_29.jpg)

### 2.2. Cài đặt Baloon driver cho Memory
Copy `/virtio-win-0.1.1/Baloon/2k12R2/amd64` từ CD Drive vào `C:\`
![Create VM 29](/images/win2k12_standard/win2k12_30.jpg)

Chạy CMD, trỏ về thư mục amd64 vừa copy và chạy lệnh:
```
PS C:\Users\Administrator> cd C:\amd64
PS C:\amd6>. \blnsvr.exe -i
```
![Create VM 30](/images/win2k12_standard/win2k12_31.jpg)
Kiểm tra trong services.msc
![Create VM 31](/images/win2k12_standard/win2k12_32.jpg)

### 2.3. Cài đặt qemu-guest-agent
#### *Chú ý: qemu-guest-agent là một daemon chạy trong máy ảo, giúp quản lý và hỗ trợ máy ảo khi cần (có thể cân nhắc việc cài thành phần này lên máy ảo)*

Vào "Device Manager", chọn update driver cho `PCI Simple Communication Controller`
![Create VM 32](/images/win2k12_standard/win2k12_33.jpg)
![Create VM 33](/images/win2k12_standard/win2k12_34.jpg)
![Create VM 34](/images/win2k12_standard/win2k12_35.jpg)

Kiểm tra lại việc cài đặt Driver cho `PCI Simple Communication Controller`
![Create VM 35](/images/win2k12_standard/win2k12_36.jpg)

Cài đặt qemu-guest-agent cho Windows Server 2k12, vào CD ROM virio và cài đặt phiên bản qemu-ga (ở đây là `qemu-ga-x64`)
![Create VM 36](/images/win2k12_standard/win2k12_37.jpg)

Kiểm tra lại việc cài đặt qemu-guest-agent

`PS C:\Users\Administrator> Get-Service QEMU-GA`

![Create VM 37](/images/win2k12_standard/win2k12_38.jpg)

Kiểm tra lại version của qemu-guest-agent (phải đảm bảo version >= 7.3.2)
![qemu-ga version](/images/win2k12_standard/win2k12_49.jpg)

### 2.3. Disable Firewall và enable remote desktop
![disable FW](/images/win2k12_standard/win2k12_51.jpg)
![enable RDP](/images/win2k12_standard/win2k12_50.jpg)


### 2.4. Cài đặt cloud-init bản mới nhất
Download cloud base init cho Windows bản mới nhất tại [đây](https://cloudbase.it/cloudbase-init/)
![Create VM 38](/images/win2k12_standard/win2k12_39.jpg)

Tiến hành cài đặt
![Create VM 39](/images/win2k12_standard/win2k12_40.jpg)
![Create VM 40](/images/win2k12_standard/win2k12_41.jpg)
![Create VM 41](/images/win2k12_standard/win2k12_42.jpg)
![Create VM 42](/images/win2k12_standard/win2k12_43.jpg)

Trước khi "Finish" cài đặt, sửa lại file `C:\Program Files\Cloudbase Solutions\Cloudbase-Init\conf\cloudbase-init.conf`
```
[DEFAULT]
username=Administrator
groups=Administrators
inject_user_password=true
first_logon_behaviour=no
config_drive_raw_hhd=true
config_drive_cdrom=true
config_drive_vfat=true
bsdtar_path=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\bin\bsdtar.exe
mtools_path=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\bin\
verbose=true
debug=true
logdir=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\log\
logfile=cloudbase-init.log
default_log_levels=comtypes=INFO,suds=INFO,iso8601=WARN,requests=WARN
logging_serial_port_settings=COM1,115200,N,8
mtu_use_dhcp_config=true
ntp_use_dhcp_config=true
local_scripts_path=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\LocalScripts\
```
Enable Sysprep và shutdown máy

![Create VM 43](/images/win2k12_standard/win2k12_44.jpg)
![Create VM 44](/images/win2k12_standard/win2k12_45.jpg)

## 3.Thực hiện trên Host KVM
### 3.1. Cài đặt bộ libguestfs-tools để xử lý image (nên cài đặt trên Ubuntu OS để có bản libguestfs mới nhất)
```
apt-get install libguestfs-tools -y
```

### 3.2. Dùng lệnh sau để tối ưu kích thước image:
```
virt-sparsify --compress /var/www/ftp/images/win2012x64_standard.img /var/www/ftp/images/win2012x64_standard_shrink.img
```

### 3.3. Upload image lên glance
```
openstack image create Win2k12 --disk-format qcow2 --container-format bare --public < /var/www/ftp/images/win2012x64_standard_shrink.img
```

### 3.4. Kiểm tra việc upload image đã thành công hay chưa

![Create VM 45](/images/win2k12_standard/win2k12_46.jpg)

### 3.5. Chỉnh sửa metadata của image upload
![Create VM 46](/images/win2k12_standard/win2k12_47.jpg)

Thêm 2 metadata là 'hw_qemu_guest_agent' và 'os_type', với giá trị tương ứng là `true` và `windows`, sau đó save lại
![Create VM 47](/images/win2k12_standard/win2k12_48.jpg)

### 3.6. Image đã sẵn sàng để launch máy ảo.

## 4. Thử nghiệm việc đổi password máy ảo (sau đã đã tạo máy ảo)
### Cách 1: sử dụng nova API (lưu ý máy ảo phải đang bật)
Trên node Controller, thực hiện lệnh và nhập password cần đổi

```
root@controller1:# nova set-password win2k12_qemu_ga
New password: 
Again: 
```
với `win2k12_qemu_ga` là tên máy ảo


### Cách 2: sử dụng trực tiếp libvirt
#### Xác định vị trí máy ảo đang nằm trên node compute nào. VD máy ảo đang sử dụng là `win2k12_qemu_ga`

`root@controller1:# nova show win2k12_qemu_ga`

với `win2k12_qemu_ga` là tên máy ảo

Kết quả:
```
+--------------------------------------+----------------------------------------------------------------------------------+
| Property                             | Value                                                                            |
+--------------------------------------+----------------------------------------------------------------------------------+
| OS-DCF:diskConfig                    | MANUAL                                                                           |
| OS-EXT-AZ:availability_zone          | nova                                                                             |
| OS-EXT-SRV-ATTR:host                 | compute6		                                                                  |
| OS-EXT-SRV-ATTR:hostname             | win2k12-qemu-ga                                                                  |
| OS-EXT-SRV-ATTR:hypervisor_hostname  | compute6                                                                         |
| OS-EXT-SRV-ATTR:instance_name        | instance-00001827                                             
```

Như vậy máy ảo nằm trên node compute6 với KVM name là `instance-00001827`

#### Trên compute6, kiểm tra để tìm file socket kết nối tới máy ảo
```
root@compute6:~# bash -c  "ls /var/lib/libvirt/qemu/*.sock"
```

Kết quả:
```
/var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-00001827.sock

instance-0000001d: tên của máy ảo trên KVM
```

```
root@compute6:~# file /var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-00001827.sock
```

Kết quả:
```
/var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-00001827.sock: socket
```

#### Kiểm tra kết nối tới máy ảo. Ở trên node compute chứa máy ảo (compute6), chạy lệnh:
```
root@compute6:~# virsh qemu-agent-command instance-00001827 '{"execute":"guest-ping"}'
```

Kết quả:
```
{"return":{}}
```

#### Sinh password mới `123456a@` (password phải đáp ứng yêu cầu phức tạp của Windows)
```
echo -n "123456a@" | base64
```

Kết quả:
```
MTIzNDU2YUA=
```

#### Ở trên node compute chứa máy ảo (compute6), chèn password mới vào máy ảo, lưu ý máy ảo phải đang bật
```
root@compute6:~# virsh  qemu-agent-command instance-00001827 '{ "execute": "guest-set-user-password","arguments": { "crypted": false,"username": "administrator","password": "MTIzNDU2YUA=" } }'
```

Kết quả;
```
{"return":{}}
```

Thử đăng nhập vào máy ảo với password `123456a@`

## Done


Tham khảo: 

[1] - http://www.stratoscale.com/blog/storage/deploying-ceph-challenges-solutions/?utm_source=twitter&utm_medium=social&utm_campaign=blog_deploying-ceph-challenges-solutions

[2] - https://pve.proxmox.com/wiki/Qemu-guest-agent

[3] - https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Virtualization_Administration_Guide/sect-QEMU_Guest_Agent-Running_the_QEMU_guest_agent_on_a_Windows_guest.html

[4] - https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Virtualization_Deployment_and_Administration_Guide/chap-QEMU_Guest_Agent.html