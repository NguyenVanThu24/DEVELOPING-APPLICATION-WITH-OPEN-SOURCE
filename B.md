# B - CÀI ĐẶT UBUNTU + DOCKER
## 1. Cài đặt Ubuntu
Tải file ISO: truy cập trang ubuntu.com/download/server để tải file .iso của Ubuntu 24.04.4 LTS

Công cụ giả lập: VMWare
### Tạo máy ảo
- Mở VMware, chọn Create a New Virtual Machine.

- Chọn nguồn cài đặt: * Chọn dòng Installer disc image file (iso).

- Nhấn Browse và tìm đến file ISO Ubuntu Server vừa tải về.

- Đặt tên và lưu trữ:

   + Virtual machine name: Đặt tên cho máy ảo (VD: Ubuntu_Server).

   + Location: Chọn nơi lưu trữ ổ cứng ảo (nên chọn ổ đĩa còn trống ít nhất 20-30GB).

- Cấu hình ổ cứng (Disk Capacity):

  + Maximum disk size: Tối thiểu 20GB.

  + Chọn Split virtual disk into multiple files để dễ dàng di chuyển máy ảo sau này.

- Tùy chỉnh phần cứng (Customize Hardware):

  + Memory (RAM): Tối thiểu 1GB, nhưng nên để 2GB (2048MB) để chạy mượt mà.

  + Processors: Chọn 1 hoặc 2 nhân tùy vào cấu hình máy thật của bạn.

- Nhấn Finish để hoàn tất việc tạo khung máy ảo.
### Cài đặt Ubuntu
Sau khi nhấn Power on this virtual machine, màn hình cài đặt của Ubuntu sẽ hiện ra. Thực hiện các bước sau:

- Language: Chọn English.

- Keyboard Configuration: Chọn English (US).

- Choose type of install: Chọn Ubuntu Server (bản mặc định).

- Networking: Trình cài đặt sẽ tự nhận IP từ VMware qua DHCP. Nhấn Done.

- Storage Configuration: Chọn Use an entire disk.

- Profile Setup:

  + Your name: Hieu

  + Your server's name: hieuserver.

  + Pick a username: nguyentrunghieu

  + Password: (Đặt mật khẩu cho user).

- SSH Setup: Tích chọn Install OpenSSH server.

- Hệ thống sẽ bắt đầu cài đặt. Sau khi hoàn tất chọn Reboot Now.

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-04-13 155354" src="https://github.com/user-attachments/assets/347d4186-0d29-4f05-9985-72490333b0d0" />

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-04-13 163212" src="https://github.com/user-attachments/assets/bd8ef70a-4174-4267-917c-94ccbdf00ea5" />

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-04-13 173039" src="https://github.com/user-attachments/assets/e970b949-fadb-4a23-8559-764bfe7405d1" />

<img width="815" height="151" alt="Ảnh chụp màn hình 2026-04-13 173249" src="https://github.com/user-attachments/assets/636d4fd0-b758-4011-8506-7ff18e88014d" />

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-04-13 173556" src="https://github.com/user-attachments/assets/cf9d6ef1-b861-4175-952c-43860c6d9c8c" />

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-04-13 173705" src="https://github.com/user-attachments/assets/476a82cd-d5f7-4dc0-a6f6-f71460c577cf" />

<img width="521" height="51" alt="Ảnh chụp màn hình 2026-04-13 173900" src="https://github.com/user-attachments/assets/88fa01df-a12d-497b-b9b9-f24ba3d34b64" />

<img width="511" height="100" alt="Ảnh chụp màn hình 2026-04-13 174722" src="https://github.com/user-attachments/assets/7ab6a4ce-0225-4fbb-9357-29de3161adf9" />

<img width="762" height="117" alt="Ảnh chụp màn hình 2026-04-13 183215" src="https://github.com/user-attachments/assets/ddc103ab-5638-4be2-a68d-c0ea4ef266d0" />

<img width="802" height="167" alt="Ảnh chụp màn hình 2026-04-13 184145" src="https://github.com/user-attachments/assets/cfc3347a-e674-49a1-ba5d-3ee01a1eb0bc" />

<img width="1920" height="1020" alt="Ảnh chụp màn hình 2026-04-13 184506" src="https://github.com/user-attachments/assets/7f1c312c-532e-4f83-b723-c2a87cc9a56f" />

<img width="842" height="264" alt="Ảnh chụp màn hình 2026-04-13 184620" src="https://github.com/user-attachments/assets/c6ea6a86-8b84-45b8-8202-faba11a6b62e" />

<img width="1109" height="447" alt="Ảnh chụp màn hình 2026-04-13 184957" src="https://github.com/user-attachments/assets/272268c4-b812-4710-b333-c3dd8b72c9b6" />

<img width="1622" height="441" alt="Ảnh chụp màn hình 2026-04-13 185025" src="https://github.com/user-attachments/assets/84d4aaf5-de24-4e2f-b12a-39dacca6ced0" />

<img width="865" height="221" alt="Ảnh chụp màn hình 2026-04-13 185104" src="https://github.com/user-attachments/assets/4d8134f1-65fd-48a1-9ca3-f6bd70fe862e" />

<img width="888" height="719" alt="Ảnh chụp màn hình 2026-04-13 192347" src="https://github.com/user-attachments/assets/ca35c676-629d-4fed-8cee-c45a87222ed9" />

<img width="877" height="62" alt="Ảnh chụp màn hình 2026-04-13 192422" src="https://github.com/user-attachments/assets/9f76d41a-5610-4ff5-babc-55755ca65398" />

<img width="962" height="725" alt="Ảnh chụp màn hình 2026-04-13 194416" src="https://github.com/user-attachments/assets/27354dc1-2e49-46d1-a4f9-4bf894c50a0e" />
