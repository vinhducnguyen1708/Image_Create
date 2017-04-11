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

![Đặt tên máy ảo](/images/buildVM_fromISO_1.jpg)

![Chọn file ISO](/images/buildVM_fromISO_2.jpg)

![Chọn cấu hình](/images/buildVM_fromISO_3.jpg)

### 1.3. Đợi đến khi máy ảo chuyển trạng thái sang active, tạo một volume trống (20GB) và gắn vào máy ảo

![Tao volume](/images/buildVM_fromISO_5.jpg)

![Gắn volume](/images/buildVM_fromISO_6.jpg)

### 1.4. Click vào "Console" để truy cập vào cửa sổ console của máy ảo

![console VM](/images/buildVM_fromISO_4.jpg)

### 1.5. Tiến hành cài đặt OS như bình thường, ở phần `Partition disks`, chọn `Manual`

![disk manual](/images/buildVM_fromISO_7.jpg)

### 1.6. Chọn virtual disk 1 (là volume vừa gắn thêm vào)

![choose disk](/images/buildVM_fromISO_8.jpg)

### 1.7. Chọn vào dòng `pri/log`

![free space](/images/buildVM_fromISO_9.jpg)

### 1.8. Tạo partition mới trên disk cho root, sử dụng toàn bộ dung lượng ổ đĩa, loại primary partition

![new partition](/images/buildVM_fromISO_10.jpg)

![size partition](/images/buildVM_fromISO_11.jpg)

![primary partition](/images/buildVM_fromISO_12.jpg)

### 1.9. Chọn mount point là `/`, format `ext4`, sau đó `Done setting up the partition`

![format partition](/images/buildVM_fromISO_13.jpg)

### 1.10. Chọn `Finish partitioning and write changes to disk`

![finish partition](/images/buildVM_fromISO_14.jpg)

#### Đồng ý không sử dụng swap disk

![no swap](/images/buildVM_fromISO_15.jpg)

![write change](/images/buildVM_fromISO_16.jpg)

#### Không chọn `HTTP proxy` và `Continue`

![no proxy](/images/buildVM_fromISO_17.jpg)

#### Chọn `No automatic update`

![no auto update](/images/buildVM_fromISO_18.jpg)

#### Không cài đặt software

![no install software](/images/buildVM_fromISO_19.jpg)

#### Cài đặt GRUB bootloader

![install GRUB](/images/buildVM_fromISO_20.jpg)

#### Khởi động lại VM

![restart VM](/images/buildVM_fromISO_21.jpg)

### 1.11. Sau khi khởi động lại vào màn hình cài đặt, nhấn `ESC` để lựa chọn `Boot from a hard disk`

![boot hard disk](/images/buildVM_fromISO_22.jpg)

### 1.12. Sử dụng tài khoản khai báo ban đầu để login vào VM

![login VM](/images/buildVM_fromISO_23.jpg)


## 2. Cài đặt các công cụ để xử lý VM
### 2.1. Cài đặt openssh-client và openssh-server
```
sudo apt-get update
sudo apt-get install openssh-client openssh-server -y
```