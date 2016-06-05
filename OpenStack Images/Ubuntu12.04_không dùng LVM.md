##Bước tạo máy ảo bằng KVM mình sẽ bỏ qua và đi ngay vào phần xử lý image sau khi đã cài xong OS
####<i>Chú ý: Khi tạo image không sử dụng LVM để có thể resize lại partition theo flavor</i>
##1. Xử lý phần OS của máy ảo
###Để máy ảo khi boot sẽ tự giãn phân vùng theo dung lượng mới, ta cài các gói sau:
```
apt-get install cloud-utils cloud-initramfs-growroot cloud-init -y
```
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
#####Sau đó sửa file `/etc/rc.local` để chạy script tự động khi máy ảo được boot
```
bash /etc/boot/NIC.sh
exit 0
```
#####Chỉnh sửa file `/etc/default/grub` để bắn log ra trong quá trình tạo máy ảo
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
Để điều chỉnh metadata source cho máy ảo khi boot, chạy lệnh:
`dpkg-reconfigure cloud-init`
Chọn EC2 data source
Ở trên máy ảo Ubuntu, account là "ubuntu"
###Xóa toàn bộ các thông tin về địa chỉ MAC của card mạng ảo:
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
#####Xử dụng lệnh `virt-sysprep` để xóa toàn bộ các thông tin máy ảo:
```
virt-sysprep -a U.1204.img
```
#####Dùng lệnh sau để tối ưu kích thước image:
```
virt-sparsify --compress U.1204.img U.1204.shrink.img
```
#####Image <b>U.1204.shrink.img</b> đã có thể upload lên Glance
