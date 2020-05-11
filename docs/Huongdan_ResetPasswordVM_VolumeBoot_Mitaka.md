# Hướng dẫn Reset password cho các máy ảo Boot từ volume trên hệ thống OpenStack phiên bản Mitaka
### Mục tiêu của hướng dẫn:
```
 - Hỗ trợ khách hàng có nhu cầu phục hồi password cho các máy ảo: Windows và Linux.
 - Do phiên bản Mitaka không hỗ trợ tính năng rescue cho VM boot từ volume, do đó sẽ phải dùng giải pháp khác để reset password VM
```

#### <i>Chú ý: </i>
 - Hướng dẫn thực hiện trên phiên bản OpenStack Mitaka, OS Ubuntu 14.04

#### Cách thức thực hiện: 
 - Chỉnh sửa DB của OpenStack để gỡ được volume boot khỏi máy ảo.
 - Gắn volume boot vào 1 máy ảo khác có sẵn SystemRescueCD.
 - Thực hiện reset password.
 - Gắn lại volume boot vào máy ảo ban đầu, chỉnh sửa lại DB OpenStack để đưa máy ảo về trạng thái ban đầu.
 - Boot máy ảo và đăng nhập bằng password mới.


## 1. Gỡ volume boot khỏi máy ảo
### 1.1. Kiểm tra danh sách máy ảo trên horizon
![resetpassword](/images/resetpassword/rp_9.png)

### 1.2. Kiểm tra danh sách volume trên horizon
![resetpassword](/images/resetpassword/rp_10.png)

### 1.3. Thực hiện tắt máy ảo cần reset password
![resetpassword](/images/resetpassword/rp_11.png)

### 1.4. Trên host Controller, đăng nhập vào MySQL và thực hiện update bản ghi của Volume boot của máy ảo cần chỉnh sửa
```sh
MariaDB [nova]> use nova;
MariaDB [nova]> update block_device_mapping set device_name = '/dev/vdl', boot_index=1 where volume_id = 'fe14c432-493e-42ca-970e-4b542fcf3cbf' and deleted = 0;       

MariaDB [nova]> use cinder;
MariaDB [cinder]> update volume_attachment set mountpoint='/dev/vdl' where volume_id ='fe14c432-493e-42ca-970e-4b542fcf3cbf' and deleted=0;
```
Trong đó:
 - `fe14c432-493e-42ca-970e-4b542fcf3cbf`: ID của Volume boot

Kết quả:
![resetpassword](/images/resetpassword/rp_12.png)

### 1.5. Gỡ volume boot khỏi máy ảo
![resetpassword](/images/resetpassword/rp_14.png)


## 2. Tạo máy ảo SystemRescueCD
### 2.1. Tạo image SystemRescueCD
Thực hiện như hướng dẫn tại [đây](https://github.com/VNPT-SmartCloud-System/DongGoiImage_OpenStack/blob/master/docs/Huongdan_ResetPasswordVM_SystemRescueCD.md#1-systemrescuecd)

### 2.2. Boot máy ảo với image SystemRescueCD vừa tạo
![resetpassword](/images/resetpassword/rp_13.png)

### 2.3. Sau khi máy ảo SystemRescueCD start, tắt máy ảo và mount Volume boot vừa gỡ ở trên vào máy ảo SystemRescueCD
![resetpassword](/images/resetpassword/rp_15.png)

### 2.4. Khởi động máy ảo SystemRescueCD


## 3. Thực hiện Reset password VM
Thực hiện như hướng dẫn tại [đây](https://github.com/longsube/DongGoiImage_OpenStack/blob/master/docs/Huongdan_ResetPasswordVM_SystemRescueCD.md#3-th%E1%BB%B1c-hi%E1%BB%87n-reset-password-vm)

### 3.1. Tắt máy ảo SystemRescueCD
```sh
shutdown -h now
```

### 3.2. Gỡ bỏ volume boot khỏi máy ảo SystemRescueCD và gắn lại vào máy ảo gốc
![resetpassword](/images/resetpassword/rp_16.png)

### 3.3. Trên host Controller, đăng nhập vào MySQL và thực hiện update lại bản ghi của Volume boot của máy ảo gốc
```sh
MariaDB [nova]> use nova;
MariaDB [nova]> update block_device_mapping set device_name = '/dev/vda', boot_index=1 where volume_id = 'fe14c432-493e-42ca-970e-4b542fcf3cbf' and deleted = 0;       

MariaDB [nova]> use cinder;
MariaDB [cinder]> update volume_attachment set mountpoint='/dev/vda' where volume_id ='fe14c432-493e-42ca-970e-4b542fcf3cbf' and deleted=0;
```
Trong đó:
 - `fe14c432-493e-42ca-970e-4b542fcf3cbf`: ID của Volume boot

Kết quả:
![resetpassword](/images/resetpassword/rp_17.png)

### 3.4. Khởi động máy ảo và đăng nhập

## Done

# Tham khảo:
- https://bugs.launchpad.net/nova/+bug/1396965

