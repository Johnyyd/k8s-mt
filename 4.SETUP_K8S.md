# Hướng dẫn Cài đặt Kubernetes (K8s) trên Ubuntu

Tài liệu này tổng hợp các lệnh cần thiết để chuẩn bị môi trường và cài đặt các thành phần cốt lõi của Kubernetes (`kubelet`, `kubeadm`, `kubectl`) cùng với Container Runtime (`containerd`).

## 1. Cập nhật hệ thống và Tắt Swap

Cập nhật các gói phần mềm và vô hiệu hóa swap (yêu cầu bắt buộc của Kubernetes):

```bash
sudo apt update -y && sudo apt upgrade -y
sudo swapoff -a
sudo sed -i '/swap.img/s/^/#/' /etc/fstab
```

## 2. Nạp các module Kernel cần thiết

Cấu hình để hệ điều hành nạp các module `overlay` và `br_netfilter` khi khởi động:

```bash
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

## 3. Cấu hình Sysctl cho Kubernetes

Cấu hình các tham số mạng để cho phép iptables xử lý lưu lượng mạng của bridge và bật IPv4 forwarding:

```bash
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

## 4. Cài đặt Container Runtime (containerd)

Cài đặt các gói phụ thuộc và thêm kho ứng dụng (repository) của Docker để cài `containerd.io`:

```bash
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update -y
sudo apt install -y containerd.io
```

Cấu hình `containerd` để sử dụng `systemd` làm cgroup driver:

```bash
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd
systemctl status containerd
```

## 5. Cài đặt các thành phần Kubernetes (Kubeadm, Kubelet, Kubectl)

Thêm kho ứng dụng của Kubernetes (ở đây dùng bản `v1.30`) và tiến hành cài đặt:

```bash
# Thêm danh sách nguồn
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Thêm khóa GPG
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor --yes -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

sudo apt update -y
sudo apt install -y kubelet kubeadm kubectl

# Khóa phiên bản không cho tự động cập nhật
sudo apt-mark hold kubelet kubeadm kubectl
```
