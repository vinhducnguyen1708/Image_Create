##Xử lý image sau khi đã cài xong OS
####<i>Chú ý: Khi tạo image không sử dụng LVM để có thể resize lại partition theo flavor</i>
##1. Xử lý phần OS của máy ảo
#####1.1 Để máy ảo khi boot sẽ tự giãn phân vùng theo dung lượng mới, ta cài các gói sau:
```
apt-get install cloud-utils cloud-initramfs-growroot cloud-init -y
```
#####1.2 Để sau khi boot máy ảo, có thể nhận đủ các NIC gắn vào, tạo một script tại `/etc/boot/NIC.sh` với nội dung:
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
#####1.3 Sau đó sửa file `/etc/rc.local` để chạy script tự động khi máy ảo được boot
```
bash /etc/boot/NIC.sh
exit 0
```
#####1.4 Chỉnh sửa file `/etc/default/grub` để bắn log ra trong quá trình tạo máy ảo
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
#####1.5 Xóa toàn bộ các thông tin về địa chỉ MAC của card mạng ảo:
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
#####2.1 Xử dụng lệnh `virt-sysprep` để xóa toàn bộ các thông tin máy ảo:
```
virt-sysprep -a U.1404.img
```
#####2.2 Dùng lệnh sau để tối ưu kích thước image:
```
virt-sparsify --compress U.1404.img U.1404.shrink.img
```
#####2.3 Image <b>U.1404.shrink.img</b> đã có thể upload lên Glance
