##Bước tạo máy ảo bằng KVM mình sẽ bỏ qua và đi ngay vào phần xử lý image sau khi đã cài xong OS
##1. Xử lý phần OS của máy ảo
#####Để sau khi boot máy ảo, có thể nhận đủ các NIC gắn vào, tạo một script tại `/etc/boot/NIC.sh` với nội dung:
```
for iface in $(ip -o link | cut -d: -f2 | tr -d ' ' | grep ^eth)
do
   egrep -q "^iface\s+$iface" /etc/network/interfaces
   if [ $? -ne 0 ]
   then
       echo -e "auto $iface\niface $iface inet dhcp\n" >> /etc/network/interfaces
       ifup $iface
   fi
done
```
#####Sau đó sửa file ```/etc/rc.local``` để chạy script tự động khi máy ảo được boot
```
bash /etc/boot/NIC.sh
exit 0
```
#####Sau đó sửa file `/etc/rc.local` để chạy script tự động khi máy ảo được boot
```
bash /etc/boot/NIC.sh
exit 0
```
#####Chỉnh sửa file `/etc/default/grub` để bắn log ra trong quá trình tạo máy ảo
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
#####Xóa toàn bộ các thông tin về địa chỉ MAC của card mạng ảo:
```
/etc/sysconfig/network-scripts/ifcfg-eth0 
/etc/udev/rules.d/70-persistent-net.rules
```
Chú ý: không xóa 2 file này mà chỉ xóa nội dung 

#####Tắt máy ảo 
```
init 0
```

##2.Xử lý Image 
#####Xử dụng lệnh ```virt-sysprep``` để xóa toàn bộ các thông tin máy ảo:
```
virt-sysprep -a U.1404.img
```
#####Dùng lệnh sau để tối ưu kích thước image:
Nếu máy ảo không sử dụng LVM
```
virt-sparsify --compress U.1404.img U.1404.shrink.img
```
Nếu máy ảo sử dụng LVM
```
qemu-img convert -c -f qcow2 -O qcow2 U.1404.img U.1404.shrink.img
```
#####Image <b>U.1404.shrink.img</b> đã có thể sử dụng để clone ra các máy ảo khác

##3. Boot máy ảo từ image
#####Sau khi boot máy ảo, cần reinstall lại openssh để có thể SSh tới máy ảo
```
apt-get install --reinstall openssh-server openssh-client -y
```
