#### <i>Chú ý: </i>
 - Sử dụng công cụ `virt-manager` để kết nối tới console máy ảo
 - Hướng dẫn bao gồm 3 phần chính: cài đặt OS máy ảo, thực hiện trên máy ảo cài OS, thực hiện trên KVM Host

## 1. Cài OS cho máy ảo
### 1.1. Trên máy host KVM, tạo file image cho máy ảo (với windows nên lấy dung lượng 40GB trở lên), đặt tại thư mục /var/www/ftp/images/
```
qemu-img create -f qcow2 /var/www/ftp/images/win2k3R2x64_enterprise.img 40G
```

### 1.2. Trên máy host KVM, lấy file ISO Windows2k3 Enterprise và virio driver phiên bản dành cho Windows 2k3 driver cho thiết bị ảo, đặt tại thư mục /var/www/ftp/ISO/
```
wget -O /var/www/ftp/ISO/virtio-win-1.1.16.iso https://drive.google.com/open?id=0B-9dXNs6dxdoMjdlNDZZWXVEeTA
chmod +x /var/www/ftp/ISO/windows2003_R2_enterprise.iso
wget -O /var/www/ftp/ISO/virtio-win-1.1.16.iso https://drive.google.com/open?id=0B-9dXNs6dxdoaUkxaENTU1lkOEE
chmod +x /var/www/ftp/ISO/virtio-win-1.1.16.iso
wget -O /var/www/ftp/ISO/virtio-win-1.1.16.vfd https://drive.google.com/open?id=0B-9dXNs6dxdoRjRqTm1OYktGSUE
chmod +x /var/www/ftp/ISO/virtio-win-1.1.16.vfd
```

### 1.3. Trên máy host KVM, bật giao diện virt-manager và khởi tạo, khai báo tên máy ảo
![Create VM 1](/images/win2k3_enterprise/win2k3_32.jpg)

### 1.4. Chọn đường dẫn tới file image tạo ban đầu, và khai báo OS type của máy ảo
![Create VM 2](/images/win2k3_enterprise/win2k3_33.jpg)

### 1.5. Khai báo CPU và RAM cho máy ảo
![Create VM 3](/images/win2k3_enterprise/win2k3_34.jpg)

### 1.6. Kích vào lựa chọn "Customize configuration before install" sau đó Finish
![Create VM 4](/images/win2k3_enterprise/win2k3_35.jpg)

### 1.7. Lựa chọn "Add hardware", sau đó add thêm CD ROM với ISO Windows 2003 Enterprise
![Create VM 5](/images/win2k3_enterprise/win2k3_3.jpg)

### 1.8. Lựa chọn "Add hardware", sau đó add thêm Floopy disk với file virtio-win-1.1.16.vfd 
![Create VM 6](/images/win2k3_enterprise/win2k3_4.jpg)

### 1.9. Trong phần "NIC", lựa chọn giải mạng NAT
![Create VM 7](/images/win2k3_enterprise/win2k3_5.jpg)

### 1.9. Trong phần "Boot Options", chỉnh lại thứ tự boot, sau đó chọn "Begin Installation" để bắt đầu chạy máy ảo
![Create VM 8](/images/win2k3_enterprise/win2k3_6.jpg)

### 1.10. Bắt đầu cài đặt OS, làm theo các hướng dẫn
![Windows Install 1](/images/win2k3_enterprise/win2k3_7.jpg)
![Windows Install 2](/images/win2k3_enterprise/win2k3_8.jpg)
![Windows Install 3](/images/win2k3_enterprise/win2k3_9.jpg)

### 1.11. Chọn phân vùng trống và "Enter"
#### *Chú ý: cần phải có file virtio-win-1.1.16.vfd thì máy ảo mới nhận được disk*
![Windows Install 4](/images/win2k3_enterprise/win2k3_10.jpg)

### 1.12. Format phân vùng vừa chọn thành NTFS
![Windows Install 5](/images/win2k3_enterprise/win2k3_11.jpg)
![Windows Install 6](/images/win2k3_enterprise/win2k3_12.jpg)

### 1.13. Lựa chọn "Yes" để tiếp tục cài đặt
![Windows Install 7](/images/win2k3_enterprise/win2k3_13.jpg)

### 1.14. Lựa chọn "Yes" để cài đặt driver cho chuôt
![Windows Install 8](/images/win2k3_enterprise/win2k3_14.jpg)

### 1.15. Setup time zone, sau đó "Next"
![Windows Install 9](/images/win2k3_enterprise/win2k3_15.jpg)

### 1.16. Đặt tên cho máy ảo
![Windows Install 10](/images/win2k3_enterprise/win2k3_16.jpg)

### 1.17. Nhập key "QV9XT-CV22K-D8MGR-4MD86-8MYR6" để kích hoạt OS
![Windows Install 11](/images/win2k3_enterprise/win2k3_17.jpg)

### 1.18. Nhập key "QV9XT-CV22K-D8MGR-4MD86-8MYR6" để kích hoạt OS
![Windows Install 12](/images/win2k3_enterprise/win2k3_18.jpg)

### 1.19. Sau khi cài đặt OS thành công, tắt VM và gắn thêm CD Drive chứa "virtio-win-1.1.16.iso"
![Windows Install 13](/images/win2k3_enterprise/win2k3_36.jpg)

### 1.20. Sửa lại Boot Options, lựa chọn Boot từ Hard Disk
![Windows Install 14](/images/win2k3_enterprise/win2k3_37.jpg)

## 2. Thực hiện trong máy ảo
### 2.1. Khởi động lại và đăng nhập với tài khoản đã tạo ban đầu
![Windows Install 15](/images/win2k3_enterprise/win2k3_19.jpg)

### 2.2. Vào "Device Manager" để update driver cho NIC
![Windows Setup 1](/images/win2k3_enterprise/win2k3_20.jpg)
![Windows Setup 2](/images/win2k3_enterprise/win2k3_21.jpg)

Browse tới CD ROM vừa gắn thêm vào
![Windows Setup 3](/images/win2k3_enterprise/win2k3_23.jpg)

Lựa chọn NetKVM->XP->amd64
![Windows Setup 4](/images/win2k3_enterprise/win2k3_24.jpg)

Lựa chọn driver cho Win2003
![Windows Setup 5](/images/win2k3_enterprise/win2k3_25.jpg)
![Windows Setup 6](/images/win2k3_enterprise/win2k3_26.jpg)

Chờ Driver được cài đặt và Finish
![Windows Setup 7](/images/win2k3_enterprise/win2k3_27.jpg)
![Windows Setup 8](/images/win2k3_enterprise/win2k3_28.jpg)

### 2.3. Kiểm tra lại việc cài đặt Driver cho NIC
![Windows Setup 9](/images/win2k3_enterprise/win2k3_29.jpg)
![Windows Setup 10](/images/win2k3_enterprise/win2k3_30.jpg)

### 2.4 Tắt máy ảo
![Windows Setup 11](/images/win2k3_enterprise/win2k3_31.jpg)

## 3.Thực hiện trên Host KVM
### 3.1. Cài đặt bộ libguestfs-tools để xử lý image (nên cài đặt trên Ubuntu OS để có bản libguestfs mới nhất)
```
apt-get install libguestfs-tools -y
```

### 3.2. Dùng lệnh sau để tối ưu kích thước image:
```
virt-sparsify --compress /var/lib/libvirt/images/win2k3R2x64_enterprise.img /var/lib/libvirt/images/win2k3R2x64_enterprise_shrink.img
```

### 3.3. Upload image lên glance
```
openstack image create Win2k3_enterprise--disk-format qcow2 --container-format bare --public < /var/lib/libvirt/images/win2k3R2x64_enterprise_shrink.img
```

### 3.4. Image đã sẵn sàng để launch máy ảo.

## Done
 
Tham khảo: 
[1] - http://powanjuanshu.blog.51cto.com/9779836/1612068