# Hướng dẫn Cấu hình Mạng, Khắc phục lỗi Clone và Phân quyền User

Tài liệu này ghi lại các bước khắc phục lỗi VMWare nhận sai dải IP, thiết lập IP tĩnh bằng Netplan, xử lý lỗi mạng khi nhân bản (clone) máy ảo, và cách cấp quyền root cho một user.

---

## 1. Khắc phục VMWare nhận nhầm dải IP (Lỗi IP 172.x.x.x)

Khi để chế độ mạng là Bridged (Automatic), VMWare trên Windows có thể nhận nhầm vào các card mạng ảo của WSL/Docker (dải `172.x.x.x`) thay vì card Wi-Fi vật lý.

**Cách khắc phục trên Windows:**

1. Mở Start Menu, tìm và chạy **Virtual Network Editor**.
2. Bấm vào nút **Change Settings** (có biểu tượng khiên Admin) ở góc dưới bên phải.
3. Chọn dòng **VMnet0** (Type: Bridged).
4. Tại mục **Bridged to:**, đổi từ `Automatic` sang đích danh tên card mạng Wi-Fi/LAN mà máy thật đang sử dụng (ví dụ: _Intel(R) Wi-Fi 6..._).
5. Bấm **Apply** và **OK**.

**Cấp lại IP trên máy ảo Ubuntu:**
Chạy 2 lệnh sau để vứt bỏ IP cũ và xin router cấp lại IP mới:

```bash
sudo dhclient -r ens33
sudo dhclient -v ens33
```

Kiểm tra lại bằng lệnh `ip a` xem đã nhận dải `192.168.100.x` chưa.

## 2. Cấu hình IP tĩnh (Static IP) bằng Netplan

Để các node Kubernetes giao tiếp ổn định, cần thiết lập IP tĩnh.

Mở file cấu hình Netplan:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Chỉnh sửa nội dung file. Lưu ý: Dùng phím Space (dấu cách) để thụt lề, tuyệt đối không dùng phím Tab.

```yaml
network:
  ethernets:
    ens33:
      dhcp4: false
      addresses:
        - 192.168.100.201/24
      routes:
        - to: default
          via: 192.168.100.1
  version: 2
```

Lưu (`Ctrl + O`, `Enter`), thoát (`Ctrl + X`).

Áp dụng mạng:

```bash
sudo netplan apply
```

## 3. Khắc phục lỗi mạng khi Clone máy ảo

Khi clone từ master-1 sang master-2, VMWare đổi địa chỉ MAC nhưng Netplan vẫn lưu MAC cũ, gây lỗi: `Cannot find unique matching interface for ens33`.

**Bước 1: Sửa file Netplan**
Mở file Netplan (`sudo nano /etc/netplan/00-installer-config.yaml`), xóa bỏ các dòng `match:` và `macaddress:` (nếu có). Đổi IP tĩnh sang IP mới (ví dụ: `192.168.100.52`). Sau đó chạy `sudo netplan apply`.

**Bước 2: Đổi Hostname (Tên máy)**
Các node trong K8s không được trùng tên. Đổi tên máy ảo mới:

```bash
sudo hostnamectl set-hostname ubuntu-server-master-2
```

**Bước 3: Sửa file phân giải nội bộ**
Mở file hosts:

```bash
sudo nano /etc/hosts
```

Sửa dòng `127.0.1.1 ubuntu-server-master-1` thành `127.0.1.1 ubuntu-server-master-2`.
Lưu, thoát và gõ lệnh `reboot` để khởi động lại máy ảo.

## 4. Phân quyền quản trị (Thêm user vào nhóm Sudo)

Để một user thông thường (ví dụ: `devops`) có thể chạy các lệnh quản trị bằng `sudo`:

Chạy lệnh nối user vào nhóm sudo:

```bash
sudo usermod -aG sudo devops
```

Kiểm tra lại xem user đã vào nhóm thành công chưa:

```bash
groups devops
```

(Kết quả trả về phải chứa chữ `sudo`).

Đăng xuất (`exit`) và đăng nhập lại bằng user `devops` để quyền quản trị mới có hiệu lực.
