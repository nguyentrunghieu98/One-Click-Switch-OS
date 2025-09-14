# 🔄 One-Click Switch OS (Windows ↔ Linux Mint)

## Giới thiệu
Mình có 2 ổ cứng riêng:
- Ổ A: Windows (có bootloader riêng).
- Ổ B: Linux Mint (có bootloader riêng).

Mục tiêu: **Chỉ với 1 click trên Windows/Linux, máy sẽ reboot thẳng sang hệ điều hành còn lại.**

---

## 1. Windows → Boot sang Linux

### Dùng `bcdedit`
1. Mở CMD **với quyền Administrator**.
2. Liệt kê boot entry:
   ```bat
   bcdedit /enum firmware
   ```
   → Tìm entry của **GRUB / Linux Bootloader** và copy `{ID}`.

3. Tạo file `switch-to-linux.bat`:
   ```bat
   @echo off
   REM Set boot sequence sang Linux cho lần reboot kế tiếp
   bcdedit /bootsequence {ID} /addfirst
   shutdown /r /t 0
   ```

4. Double-click file này → Windows sẽ reboot thẳng sang Linux.

---

## 2. Linux → Boot sang Windows

### Cách 1: Dùng `grub-reboot`
1. Kiểm tra các entry của GRUB:
   ```bash
   grep menuentry /boot/grub/grub.cfg
   ```
   Ví dụ entry Windows:  
   `"Windows Boot Manager (on /dev/nvme0n1p1)"`

2. Tạo file `switch-to-windows.sh`:
   ```bash
   #!/bin/bash
   # Yêu cầu quyền sudo để ghi vào GRUB
   sudo grub-reboot "Windows Boot Manager (on /dev/nvme0n1p1)"
   sudo reboot
   ```

3. Cấp quyền chạy:
   ```bash
   chmod +x switch-to-windows.sh
   ```

4. Gán script vào shortcut trên Desktop/Menu → Click là reboot thẳng sang Windows.

---

### Cách 2: Dùng `efibootmgr` (nếu Windows/Linux tách bootloader độc lập)
1. Liệt kê boot entries:
   ```bash
   sudo efibootmgr
   ```
   Ví dụ:
   ```
   Boot0000* Windows Boot Manager
   Boot0001* ubuntu
   ```

2. Tạo script `switch-to-windows.sh`:
   ```bash
   #!/bin/bash
   # Chỉ định BootNext = Boot0000 (Windows Boot Manager)
   sudo efibootmgr -n 0000
   sudo reboot
   ```

---

## 3. Tích hợp giao diện "1 Click"

- **Windows**: để file `.bat` trên Desktop hoặc pin ra Taskbar.
- **Linux**: tạo file `switch-to-windows.desktop` trong `~/.local/share/applications/`:
  ```ini
  [Desktop Entry]
  Name=Switch to Windows
  Exec=/home/username/switch-to-windows.sh
  Type=Application
  Terminal=true
  Icon=system-reboot
  ```
  → Sau đó sẽ thấy icon trong menu/desktop, click là chuyển.

---

## 4. Lưu ý
- Các script cần quyền **Administrator (Windows)** hoặc **sudo (Linux)**.
- Đảm bảo BIOS/UEFI nhận cả hai ổ và boot entry đã có.
- Nếu thay đổi ổ/partition → cần cập nhật lại `{ID}` trong Windows hoặc `BootXXXX` trong Linux.

---
