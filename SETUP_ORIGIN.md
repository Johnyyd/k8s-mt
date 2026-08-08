# Hướng dẫn Thiết lập Máy Ảo Gốc (Ubuntu Server Template) cho Kubernetes

Tài liệu này ghi lại các bước cấu hình ban đầu để tạo một máy ảo Ubuntu Server "sạch" trên VMWare. Máy ảo này sẽ đóng vai trò là bản gốc (template) để nhân bản (clone) ra các Node (Master/Worker) cho cụm Kubernetes.

## 1. Cấu hình Mạng (Network) trên VMWare

Trước khi cài đặt hệ điều hành, cần đảm bảo máy ảo có thể giao tiếp với mạng nội bộ và các thiết bị phần cứng khác (như máy tính xài Windows 11 hay thiết bị ARM).

- **Chế độ Mạng:** Chuyển card mạng của máy ảo sang chế độ **Bridged Network**.
- **Mục đích:** Cho phép máy ảo nhận địa chỉ IP vật lý trực tiếp từ router (cùng dải mạng nội bộ với các thiết bị khác, ví dụ `192.168.10.x`), giúp các máy trong cụm có thể "nhìn thấy" và giao tiếp với nhau.

## 2. Quá trình Cài đặt Hệ điều hành Ubuntu Server

Tiến hành cài đặt Ubuntu Server trên VMWare. Trong quá trình cài đặt, hãy chú ý cấu hình các màn hình sau:

### 2.1. Profile Configuration (Cấu hình Tài khoản)

Đây là bước tạo tài khoản quản trị gốc cho hệ thống.

- **Your name:** `Admin` (hoặc tên tùy chọn)
- **Your server's name:** `k8s-master-1` (hoặc `ubuntu-server-master-1` khớp với tên VMWare).
  - _Lưu ý: Không được trùng hostname giữa các Node trong cụm Kubernetes._
- **Pick a username:** `devops` (Tài khoản này sẽ dùng để thực thi lệnh).
- **Choose a password:** Đặt mật khẩu an toàn.
- **Confirm your password:** Nhập lại mật khẩu.
- **Thao tác:** Dùng phím `Tab` di chuyển xuống chọn **[ Done ]** và nhấn `Enter`.

### 2.2. SSH Configuration (Cấu hình SSH)

Thiết lập quyền truy cập từ xa vào máy ảo.

- **Install OpenSSH server:** Đánh dấu `[X]` (Bắt buộc).
- **Allow password authentication over SSH:** Đánh dấu `[X]` (Cho phép đăng nhập bằng password).
- **Import SSH key:** Bỏ trống (Không cần thiết vì sẽ sử dụng cơ chế bảo mật của Tailscale sau này).
- **Thao tác:** Chọn **[ Done ]** để tiếp tục.

### 2.3. Featured Server Snaps (Các gói phần mềm mở rộng)

Lựa chọn các gói phần mềm cài đặt thêm.

- **Lựa chọn:** Bỏ trống toàn bộ các mục (Không đánh dấu `[X]` vào bất kỳ ô nào).
- **Mục đích:**
  - Giữ cho máy gốc nhẹ và sạch nhất có thể.
  - Tránh cài đặt các gói không cần thiết (như `microk8s` hay `docker`) gây tốn tài nguyên và xung đột với K3s và `containerd`.
- **Thao tác:** Chọn **[ Done ]** để hoàn tất cấu hình.

## 3. Các bước Tiếp theo (Sau khi cài đặt xong)

Sau khi hoàn tất cài đặt và khởi động lại, máy ảo gốc đã sẵn sàng.

1.  Thực hiện Clone (nhân bản) máy ảo gốc ra các máy mới (ví dụ: `master-2`, `master-3`).
2.  **Rất Quan Trọng:** Đăng nhập vào từng máy vừa clone và:
    - Đổi **Hostname** cho khác với máy gốc.
    - Thiết lập **IP tĩnh** khác nhau.

## 4. Giải pháp Mạng & Quản trị (OPTIONAL)

- **Tailscale:** Sẽ được cài đặt để tạo mạng riêng ảo (VPN), cấp IP tĩnh dạng `100.x.y.z` cho mỗi Node.
  - Lợi ích: Cho phép SSH từ xa không cần mở port (dùng tính năng Tailscale SSH).
  - Lưu ý: Phải cấu hình lại MTU của mạng Flannel (K3s) xuống `1320` hoặc `1280` để tránh rớt gói tin.
- **K3s:** Sử dụng phiên bản Kubernetes nhẹ nhàng (K3s) thay vì K8s chuẩn (kubeadm) để tối ưu tài nguyên cho các máy cấu hình thấp (đặc biệt là máy ARM và ổ HDD cũ).
