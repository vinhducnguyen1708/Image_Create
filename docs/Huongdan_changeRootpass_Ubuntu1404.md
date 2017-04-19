# Hướng dẫn tạo image có thể thay đổi password root
 - Các hướng dẫn sử dụng công cụ qemu-guest-agent (> 2.3.0) cài đặt trong image để thay đổi password từ host KVM
 - Sử dụng công cụ `virt-manager` để kết nối tới console máy ảo
 - Hướng dẫn bao gồm 2 phần chính: thực hiện trên máy ảo cài OS và thực hiện trên KVM Host

## 1. Thực hiện trên máy ảo cài OS
### 1.1. Thực hiên cài đặt OS và các công cụ cần thiết như hướng dẫn ở [đây](/docs/Ubuntu14.04_khong_dung_LVM.md)
### 1.2. Trên máy ảo image, add repo để tải gói
```
add-apt-repository cloud-archive:mitaka -y
apt-get -y update && apt-get -y dist-upgrade
```

### 1.3. Cài đặt `qemu-guest-agent`
```
apt-get install qemu-guest-agent
```
#### Kiểm tra phiên bản `qemu-ga` bằng lệnh:
```
qemu-ga --version
```

Kết quả:
```
QEMU Guest Agent 2.5.0
```

### 1.4. Cấu hình thư mục chứa file log của `qemu-ga`
```
mkdir /var/log/qemu-agent
sudo tee /etc/default/qemu-guest-agent > /dev/null <<EOF
#Cấu hình này để qemu-agent thực thi 1 script được đặt tại thư mục /etc/qemu/fsfreeze-hook, script này sẽ bắt các action từ KVM host gửi tới VM thông qua qemu-guest-agent, sau đó tự động phun ra log
DAEMON_ARGS="--logfile /var/log/qemu-agent/org.qemu.guest_agent.0.log --fsfreeze-hook --verbose"
EOF
service qemu-guest-agent restart
```

Kiểm tra bằng lệnh:
```
ls /var/log/qemu-agent/
```

Kết quả:
```
org.qemu.guest_agent.0.log
```

## 2. Thực hiện trên Host KVM
### 2.1. Kiểm tra trên máy Host KVM để tìm file socket kết nối tới máy ảo
```
bash -c  "ls /var/lib/libvirt/qemu/*.sock"
```

Kết quả:
```
/var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-0000001d.sock

instance-0000001d: tên của máy ảo trên KVM
```

```
file /var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-0000001d.sock
```

Kết quả:
```
/var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-0000001d.sock: socket
```

### 2.2. Kiểm tra kết nối tới máy ảo
```
virsh qemu-agent-command instance-0000001d '{"execute":"guest-ping"}'
```

Kết quả:
```
{"return":{}}
```

### 2.3. Sinh password mới `new`
```
echo -n "new" | base64
```

Kết quả:
```
YQ==
```

### 2.4. Chèn password mới vào máy ảo, lưu ý máy ảo phải đang bật
```
virsh  qemu-agent-command instance-0000001d '{ "execute": "guest-set-user-password","arguments": { "crypted": false,"username": "root","password": "YQ==" } }'
```

Kết quả;
```
{"return":{}}
```