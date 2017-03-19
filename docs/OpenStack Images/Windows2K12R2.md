##1. Cài OS
#####1.1 Trên máy host KVM, tạo file qcow2 cho máy ảo (với windows nên lấy dung lượng 40GB trở lên)
```
qemu-img create -f qcow2 win2012R2.img 40G
```

#####1.2 Treen máy host KVM, lấy file virio driver, là các driver cho thiết bị ảo
```
wget https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.117-1/virtio-win-0.1.117.iso
chmod 0755 /var/www/webvirtmgr/images/virtio-win-0.1.117.iso
```


#####1.3 Trên máy host KVM, chạy lệnh sau để khởi tạo máy ảo
```
virt-install \
-n Win2012R2 -r 4096 --vcpus 4 --os-type=windows --os-variant=win2k8 \
--disk path=/var/www/webvirtmgr/images/win2012R2.img,format=qcow2,bus=virtio,cache=none \
--disk path=/var/www/webvirtmgr/images/virtio-win-0.1.117.iso,device=cdrom \
-w network=default,model=virtio \
--vnc --noautoconsole \
-c /var/www/webvirtmgr/images/Win2k12R2.ISO
```

#####1.4 Trên máy host KVM, bật giao diện virt-manager để cài đặt OS cho VM
```
sudo virt-manager
```
![Windows Setup UI](http://image.prntscr.com/image/8dfb8adeff624935b3f978588dd3e69b.png)

#####1.5  Browse tới driver để VM nhận ổ đĩa (nằm tại virtio-win-0.1.1/viostor/2K12R2/amd64)
![Browse tới disk driver](http://image.prntscr.com/image/128a1463af9a4208b90a32c43fa8b89b.png)
![Disk Driver](http://image.prntscr.com/image/5bd0f8014b9c400fa434153dc165628e.png)

#####1.6 Tiến hành cài đặt OS
![Cài OS] (http://image.prntscr.com/image/188605ed4e2b4e0581a8189db945071a.png)

##2. Xử lý image sau khi đã cài xong OS
#####2.1 Cài đặt Baloon network driver để VM nhận card mạng
![Cài Baloon Network Driver](http://image.prntscr.com/image/e9e8b1d48cd9477d96bade6617fc35cf.png)
![Cài Baloon Network Driver](http://image.prntscr.com/image/5380d086290a42379b977bc8edc3485e.png)

#####2.2 Cài đặt Baloon driver cho Memory
Copy `/virtio-win-0.1.1/Baloon/2k12R2/amd64` từ CD Drive vào `C:\`
![Cài Baloon Memort Driver](http://image.prntscr.com/image/cf9fb6dd762b46aea44c0f5e8cf8f0d7.png)
Chạy CMD, trỏ về thư mục amd64 vừa copy và chạy lệnh:
```
C:\amd64> .\blnsvr.exe -i
```
![Cài Baloon Memort Driver](http://image.prntscr.com/image/09967912faac49e98b683cbc8814d763.png)
Kiểm tra trong services.msc
![Cài Baloon Memort Driver](http://image.prntscr.com/image/9106fec50a0644e8b5abea1f910e8542.png)

#####2.3 Cài đặt cloud-init bản mới nhất
Download cloud tại [đây] (https://cloudbase.it/cloudbase-init/)
![Cài Cloud init](http://image.prntscr.com/image/d96ea3f6c39444bc8d321d3290dc1f98.png)
Trước khi "Finish" cài đặt, sửa lại file `C:\Program Files\Cloudbase Solutions\Cloudbase-Init\conf`
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
![Cài Cloud init](http://image.prntscr.com/image/3930d59815f44c8d984a262de6cb5455.png)

##2.Xử lý Image 
#####2.1 Dùng lệnh sau để tối ưu kích thước image:
```
virt-sparsify --compress win2012R2.img win2012R2.shrink.img
```
#####2.2 Image <b>win2012R2.shrink.img</b> đã có thể upload lên Glance

Tham khảo: http://www.stratoscale.com/blog/storage/deploying-ceph-challenges-solutions/?utm_source=twitter&utm_medium=social&utm_campaign=blog_deploying-ceph-challenges-solutions
 
