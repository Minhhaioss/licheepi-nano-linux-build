#  LicheePi Nano Embedded Linux (Build Image)

##  Phạm vi dự án

Dự án tập trung vào việc **hiểu và tùy chỉnh pipeline build có sẵn**,  
không xây dựng hệ thống từ đầu.

Build hệ điều hành Linux nhúng cho **LicheePi Nano (Allwinner F1C100s)** với các thành phần:

- U-Boot (SPL + U-Boot)
- Linux Kernel (zImage + DTB)
- RootFS (Buildroot)

👉 Mục tiêu:
- Hiểu **boot flow**
- Hiểu **image layout**
- Làm chủ pipeline build hệ thống Linux nhúng

## 🚀 Quick Start (cho người mới)

1. Clone project

```bash
git clone https://github.com/Minhhaioss/licheepi-nano-linux-build.git
cd licheepi-nano-linux-build
```

2. Build image
chmod +x build.sh
./build.sh pull_all
./build.sh nano_tf

3. Output
File image nằm tại:
output/image/lichee-nano-normal-size.img

4. Flash vào SD card
sudo dd if=output/image/lichee-nano-normal-size.img of=/dev/sdX bs=4M conv=fsync
⚠️ Thay /dev/sdX bằng đúng thiết bị của bạn (ví dụ: /dev/sdb)

5. Boot
Cắm SD card vào board
Cấp nguồn
Linux sẽ boot

⚠️ Môi trường build
Dự án đã được chỉnh sửa để chạy trên:
Ubuntu 20.04 (WSL2)
Các thay đổi chính:
Sử dụng python2 thay vì python
Fix dependency không tương thích trên Ubuntu mới
Điều chỉnh build script để chạy ổn định trên WSL

### ⚠️ Cài đặt dependency (Ubuntu)

```bash
sudo apt update
sudo apt install -y \
    build-essential \
    gcc g++ make \
    bc bison flex \
    libssl-dev \
    libncurses5-dev \
    python2 \
    swig \
    unzip \
    mtd-utils
```
⚠️ Một số bước build có thể yêu cầu quyền `sudo`

##  Điểm nổi bật

-  Sử dụng và tùy chỉnh pipeline build Linux cho LicheePi Nano
-  Hiểu boot flow (ROM → SPL → U-Boot → Kernel)
-  Phân tích cấu trúc image và partition layout
-  Fix build để chạy trên môi trường WSL (Ubuntu 20.04)
-  Debug build bằng dd, sfdisk, loop device

## Kiến trúc hệ thống

Boot ROM (SoC)
↓
SPL (init DRAM)
↓
U-Boot
↓
Linux Kernel
↓
RootFS (ext4)

## Boot Flow
Power ON
↓
Boot ROM (trong SoC)
↓
Đọc bootloader tại offset 8KB
↓
Load SPL vào SRAM
↓
SPL khởi tạo DRAM
↓
Load U-Boot vào DRAM
↓
U-Boot load kernel (zImage) + dtb
↓
Kernel mount rootfs
↓
Linux chạy

💡 Ghi chú:

- Boot ROM là cố định trong phần cứng
- Offset `8KB` do SoC quy định
- SPL cần thiết vì DRAM chưa được khởi tạo

## Image Layout (Phân tích)
0 KB → MBR (partition table)
8 KB → U-Boot (SPL + U-Boot)
1 MB → Partition 1 (FAT - boot)
- zImage
- suniv-f1c100s-licheepi-nano.dtb
- boot.scr
17 MB → Partition 2 (ext4 - rootfs)

## Xác minh bootloader
Sử dụng `hexdump` để kiểm tra U-Boot đã được ghi đúng offset:
hexdump -C -n 128 output/u-boot-sunxi-with-spl.bin
hexdump -C -n 128 -s 8192 output/image/lichee-nano-normal-size.img
👉 Nếu nội dung giống nhau → bootloader đã được ghi đúng vị trí 8KB

![10742fe16f9898c6c189](https://user-images.githubusercontent.com/86546911/126890831-2fc226ee-0686-4011-8c79-c5a47be7d76e.jpg)

MANUAL
=======================
```shell
git clone https://github.com/ninhnn2/licheepi_nano_sdk.git
cd licheepi_nano_sdk/
sudo chmod +x ./build.sh
./build.sh pull_all
```
BUILD ROM FOR SDCARD
=======================

```shell
./build.sh nano_tf
```

BUILD ROM FOR NORFLASH 16MB
=======================

```shell
./build.sh nano_spiflash
```

The norflash rom include wifi module esp8089

Static IP: 192.168.1.100

Change ssid and password for Lichee Pi Nano 

```shell
shell# vim esp8089/wpa_supplicant.conf

network={
        ssid="embedded"
        psk="VIETNAM"
}
```


FLASH ROM TO SDCARD
=======================

```shell
cd licheepi_nano_sdk/output/image/
sudo dd bs=4M if=lichee-nano-normal-size.img of=/dev/sdb conv=fsync
```

ROM FOR TESTING
===============
user/passwd: root/000

[sdcard rom test](https://mega.nz/file/Myp20YxZ#7GH6VL6gQFb6ywQPv-gALdYCResSTUQDG2RmtdAWigw)

[norflash rom test](https://mega.nz/file/xuZBmYRJ#ES87VDUaZ-a5ne9D-fwORBrPsOvWDQMtUYfelrDtgg8)

Dự án dựa trên:
https://github.com/ninhnn2/licheepi_nano_sdk

Đã được tùy chỉnh để:
- tương thích với môi trường WSL
- phục vụ mục đích học tập và phân tích hệ thống Linux nhúng