# Hướng dẫn tạo máy ảo từ file ISO
### Mục tiêu của hướng dẫn:
```
 - Đáp ứng nhu cầu khách hàng: tạo máy ảo trực tiếp từ file ISO khi kho image của dịch vụ không có image khách hàng cần.
 - Phục vụ quản trị hệ thống: tạo mới image cho dịch vụ.
```

#### <i>Chú ý: </i>
 - Sử dụng ISO Ubuntu 14.04.5, các image khác có thể thực hiện theo các bước tương tự.
 - Hướng dẫn thực hiện trên phiên bản OpenStack Mitaka.

## 1. Cài đặt OS cho Image
### 1.1. Trên node Controller, tải file ISO và upload lên Glance
```
wget http://releases.ubuntu.com/14.04/ubuntu-14.04.5-server-amd64.iso
openstack image create ubuntu-14.04.5.iso --disk-format iso --container-format bare --public < ubuntu-14.04.5-server-amd64.iso
```

### 1.2. Truy cập vào dashboard của OpenStack và tạo máy ảo trên file ISO vừa đẩy

![Đặt tên máy ảo](/images/buildVM_fromISO/buildVM_fromISO_1.jpg)

![Chọn file ISO](/images/buildVM_fromISO/buildVM_fromISO_2.jpg)

![Chọn cấu hình](/images/buildVM_fromISO/buildVM_fromISO_3.jpg)

### 1.3. Đợi đến khi máy ảo chuyển trạng thái sang active, tạo một volume trống (20GB) và gắn vào máy ảo

![Tao volume](/images/buildVM_fromISO/buildVM_fromISO_5.jpg)

![Gắn volume](/images/buildVM_fromISO/buildVM_fromISO_6.jpg)

### 1.4. Click vào "Console" để truy cập vào cửa sổ console của máy ảo

![console VM](/images/buildVM_fromISO/buildVM_fromISO_4.jpg)

### 1.5. Tiến hành cài đặt OS như bình thường, ở phần `Partition disks`, chọn `Manual`

![disk manual](/images/buildVM_fromISO/buildVM_fromISO_7.jpg)

### 1.6. Chọn virtual disk 1 (là volume vừa gắn thêm vào)

![choose disk](/images/buildVM_fromISO/buildVM_fromISO_8.jpg)

### 1.7. Chọn vào dòng `pri/log`

![free space](/images/buildVM_fromISO/buildVM_fromISO_9.jpg)

### 1.8. Tạo partition mới trên disk cho root, sử dụng toàn bộ dung lượng ổ đĩa, loại primary partition

![new partition](/images/buildVM_fromISO/buildVM_fromISO_10.jpg)

![size partition](/images/buildVM_fromISO/buildVM_fromISO_11.jpg)

![primary partition](/images/buildVM_fromISO/buildVM_fromISO_12.jpg)

### 1.9. Chọn mount point là `/`, format `ext4`, sau đó `Done setting up the partition`

![format partition](/images/buildVM_fromISO/buildVM_fromISO_13.jpg)

### 1.10. Chọn `Finish partitioning and write changes to disk`

![finish partition](/images/buildVM_fromISO/buildVM_fromISO_14.jpg)

#### Đồng ý không sử dụng swap disk

![no swap](/images/buildVM_fromISO/buildVM_fromISO_15.jpg)

![write change](/images/buildVM_fromISO/buildVM_fromISO_16.jpg)

#### Không chọn `HTTP proxy` và `Continue`

![no proxy](/images/buildVM_fromISO/buildVM_fromISO_17.jpg)

#### Chọn `No automatic update`

![no auto update](/images/buildVM_fromISO/buildVM_fromISO_18.jpg)

#### Không cài đặt software

![no install software](/images/buildVM_fromISO/buildVM_fromISO_19.jpg)

#### Cài đặt GRUB bootloader

![install GRUB](/images/buildVM_fromISO/buildVM_fromISO_20.jpg)

#### Khởi động lại VM

![restart VM](/images/buildVM_fromISO/buildVM_fromISO_21.jpg)

### 1.11. Sau khi khởi động lại vào màn hình cài đặt, nhấn `ESC` để lựa chọn `Boot from first hard disk`

![boot hard disk](/images/buildVM_fromISO/buildVM_fromISO_22.jpg)

### 1.12. Sử dụng tài khoản khai báo ban đầu để login vào VM

![login VM](/images/buildVM_fromISO/buildVM_fromISO_23.jpg)


## 2. Cài đặt các công cụ để xử lý VM
### 2.1. Cài đặt openssh-client và openssh-server
```
sudo apt-get update
sudo apt-get install openssh-client openssh-server -y
```

### 2.2. Cài đặt các công cụ khác như hướng dẫn ở [đây](/docs/Ubuntu14.04_khong_dung_LVM.md)

### 2.3. Để có thể thay đổi password root máy ảo từ KVM Host, làm theo hướng dẫn ở [đây](/docs/Huongdan_changeRootpass_VM.md)

### 2.4 Tắt máy ảo
```
sudo init 0
```

## 3. Đóng gói image
### 3.1. Xóa máy ảo và gỡ volume khỏi máy ảo, sau đó upload volume `u14-iso-vol` lên Glance

![upload volume](/images/buildVM_fromISO/buildVM_fromISO_24.jpg)

![upload volume](/images/buildVM_fromISO/buildVM_fromISO_25.jpg)

### 3.2. Kiểm tra image đã upload thành công hay chưa

![image](/images/buildVM_fromISO/buildVM_fromISO_26.jpg)

### 3.3. Xóa volume `u14-iso-vol`, Image đã sẵn sàng để launch máy ảo.

## Done

Tham khảo:

[1] - https://github.com/alexoughton/rtd-openstack-xenserver/blob/master/docs/24-create-kvm-centos-7-image.rst
