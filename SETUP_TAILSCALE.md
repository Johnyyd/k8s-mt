Dưới đây là quy trình 2 bước chuẩn xác để cài đặt và khóa chức năng ghi đè DNS của Tailscale:

## 1. Cài đặt Tailscale (OPTIONAL)

Chạy trên từng máy Ubuntu (Master và Worker).
Chạy lệnh tự động kéo và cài đặt phiên bản Tailscale mới nhất:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

_(Nếu máy báo thiếu lệnh `curl`, bạn cài nó trước bằng lệnh: `sudo apt install curl`)._

## 2. Khởi chạy Tailscale và Tắt DNS

Tham số `--accept-dns=false` là yếu tố quyết định. Thay vì chạy lệnh `tailscale up` thông thường, bạn bắt buộc phải gắn thêm tham số cấm Tailscale can thiệp vào DNS. Hãy chạy lệnh sau:

```bash
sudo tailscale up --accept-dns=false
```

Hệ thống sẽ in ra một đường link xác thực. Bạn copy link đó dán vào trình duyệt (trên máy Windows 11), đăng nhập tài khoản Tailscale của bạn để duyệt cho thiết bị này gia nhập mạng ảo.

Sau khi xác thực xong, bạn có thể kiểm tra địa chỉ IP (dải `100.x.y.z`) mà Tailscale vừa cấp cho máy ảo này bằng lệnh:

```bash
tailscale ip -4
```
