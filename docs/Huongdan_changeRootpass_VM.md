# Hướng dẫn thay đổi password root hoặc admin cho máy ảo đang hoạt động
#### <i>Chú ý: </i>
 - Các hướng dẫn sử dụng công cụ qemu-guest-agent (> 2.3.0) cài đặt trong image để thay đổi password từ host KVM
 - Sử dụng công cụ `virt-manager` để kết nối tới console máy ảo

## 1. Với OS Ubuntu 14.04
### 1.1. Trên máy ảo image, add repo để tải gói
```
add-apt-repository cloud-archive:mitaka -y
apt-get -y update && apt-get -y dist-upgrade
```

### 1.2. Cài đặt `qemu-guest-agent`
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

### 1.3. Cấu hình thư mục chứa file log của `qemu-ga`
```
mkdir /var/log/qemu-agent
sudo tee /etc/default/qemu-guest-agent > /dev/null <<EOF
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

### 1.4. Kiểm tra trên máy Host KVM để tìm file socket kết nối tới máy ảo
```
bash -c  "ls /var/lib/libvirt/qemu/*.sock"
```

Kết quả:
```
/var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-0000001d.sock
```

```
file /var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-0000001d.sock
```

Kết quả:
```
/var/lib/libvirt/qemu/org.qemu.guest_agent.0.instance-0000001d.sock: socket
```

### 1.5. Kiểm tra kết nối tới máy ảo
```
virsh qemu-agent-command instance-0000001d '{"execute":"guest-ping"}'
```

Kết quả:
```
{"return":{}}
```

### 1.6. Sinh password mới `new`
```
echo -n "new" | base64
```

Kết quả:
```
YQ==
```

### 1.7. Chèn password mới vào máy ảo, lưu ý máy ảo phải đang bật
```
virsh  qemu-agent-command instance-0000001d '{ "execute": "guest-set-user-password","arguments": { "crypted": false,"username": "root","password": "YQ==" } }'
```

Kết quả;
```
{"return":{}}
```