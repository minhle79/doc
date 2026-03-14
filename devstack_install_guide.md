# Hướng Dẫn Cài Đặt DevStack Trên Máy Ảo (VM)

## I. Chuẩn Bị Máy Ảo

### 1. Chọn hệ điều hành

Có thể sử dụng nhiều bản Linux khác nhau. Trong hướng dẫn này sử dụng:

**Ubuntu 22.04 LTS**

------------------------------------------------------------------------

### 2. Bật Nested Virtualization

DevStack cần virtualization hoạt động trong máy ảo, vì vậy phải bật
**Nested Virtualization**.

------------------------------------------------------------------------

#### 2.1 Trên Hyper-V

Mở **PowerShell với quyền Administrator** và chạy lệnh:

``` powershell
Set-VMProcessor -VMName tenmayao -ExposeVirtualizationExtensions $true
```

Kiểm tra xem đã bật hay chưa:

``` powershell
Get-VMProcessor -VMName "tenmayao" | fl ExposeVirtualizationExtensions
```

Kết quả:

-   `True` → Đã bật thành công
-   `False` → Chưa bật

------------------------------------------------------------------------

#### 2.2 Trên VMware

Thực hiện các bước sau:

1.  Chọn máy ảo
2.  Vào **Edit Virtual Machine Settings**
3.  Chọn tab **Hardware**
4.  Chọn **Processors**
5.  Trong mục **Virtualization Engine**, tick:

```
    Virtualize Intel VT-x/EPT or AMD-V/RVI
```

Nếu không thấy tùy chọn này:

-   Kiểm tra **VT-x / AMD-V** đã bật trong **BIOS** chưa
-   Nếu đã bật nhưng vẫn lỗi → có thể do **Hyper-V đang hoạt động**

------------------------------------------------------------------------

#### 2.3 Trên VirtualBox

Các bước tương tự VMware:

    Settings → System → Processor → Enable Nested VT-x/AMD-V

------------------------------------------------------------------------

# II. Cài Đặt DevStack

## 1. Cập nhật hệ thống

``` bash
sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y
```

------------------------------------------------------------------------

## 2. Cài đặt các gói cần thiết

``` bash
sudo apt install -y git vim net-tools curl
```

------------------------------------------------------------------------

## 3. Tắt Firewall (UFW)

``` bash
sudo systemctl stop ufw
sudo systemctl disable ufw
```

------------------------------------------------------------------------

## 4. Tạo user chạy DevStack

DevStack **không khuyến nghị chạy bằng root**, nên cần tạo user riêng.

``` bash
sudo useradd -s /bin/bash -d /opt/stack -m stack
```

Cấp quyền sudo cho user `stack`:

``` bash
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
```

Chuyển sang user `stack`:

``` bash
sudo su - stack
```

------------------------------------------------------------------------

## 5. Clone DevStack

``` bash
git clone https://opendev.org/openstack/devstack
cd devstack
```

Xem các branch có sẵn:

``` bash
git branch -r
```

Chuyển sang branch ổn định:

``` bash
git checkout stable/2024.2
```

------------------------------------------------------------------------

## 6. Tạo file cấu hình

``` bash
nano local.conf
```

Thêm nội dung sau:

``` ini
[[local|localrc]]

ADMIN_PASSWORD=123456
DATABASE_PASSWORD=123456
RABBIT_PASSWORD=123456
SERVICE_PASSWORD=123456

HOST_IP=YOUR_SERVER_IP
```

Thay:

    YOUR_SERVER_IP

bằng **địa chỉ IP của máy ảo**.
Kiểm tra ip:
``` bash
ip a
```

Ví dụ:

    HOST_IP=192.168.1.100

------------------------------------------------------------------------

## 7. Cài đặt DevStack

Chạy script cài đặt:

``` bash
./stack.sh
```

Quá trình cài đặt có thể mất **20--40 phút** tùy cấu hình máy.

------------------------------------------------------------------------

# III. Truy cập OpenStack Dashboard

Sau khi cài xong, mở trình duyệt:

    http://HOST_IP/dashboard

Đăng nhập:

    Username: admin
    Password: 123456

------------------------------------------------------------------------

# IV. Cấu Hình Khuyến Nghị Cho Máy Ảo

  Tài nguyên   Khuyến nghị
  ------------ ---------------
  RAM          8GB -- 16GB
  CPU          4 -- 8 cores
  Disk         80GB -- 100GB
  OS           Ubuntu 22.04

------------------------------------------------------------------------

# V. Lưu Ý

-   Không chạy DevStack bằng **root**
-   Nên dùng **Ubuntu 22.04 hoặc 20.04**
-   Nếu cài trên **VMware** mà bị lỗi virtualization:
    -   Kiểm tra **BIOS**
    -   Kiểm tra **Hyper-V**
-   Nếu cài bị treo ở `init_placement`:
    -   Kiểm tra RAM
    -   Kiểm tra IP cấu hình trong `local.conf`
-   Tham khảo tại: https://docs.openstack.org/devstack/latest/
