# Hướng dẫn tạo máy ảo từ file ISO
#### <i>Chú ý: </i>
 - Sử dụng ISO Ubuntu 14.04
 - Sử dụng công cụ `virt-manager` để kết nối tới console máy ảo
 - Phiên bản OpenStack Mitaka

## 1. Cài đặt OS cho Image
### 1.1. Trên node Cotroller, tải file ISO và lên Glance
```
wget http://releases.ubuntu.com/14.04/ubuntu-14.04.5-server-amd64.iso
openstack image create ubuntu-14.04.iso --disk-format iso --container-format bare --public < ubuntu-14.04.5-server-amd64.iso
```

### 1.2. Truy cập vào dashboard của OpenStack và tạo máy ảo trên file ISO vừa đẩy

![Đặt tên máy ảo] (images/buildVM_fromISO_1.jpg)

![Chọn file ISO] (images/buildVM_fromISO_2.jpg)

![Chọn cấu hình] (images/buildVM_fromISO_3.jpg)
