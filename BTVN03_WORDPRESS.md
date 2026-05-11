# <p align="center">PHÁT TRIỂN ỨNG DỤNG VỚI MÃ NGUỒN MỞ (WORDPRESS + XÂY DỰNG WEBSITE GIỚI THIỆU TNUT)</p>

**Môn học:** Phát triển ứng dụng với mã nguồn mở - TEE0421  

**Họ và tên:** Nguyễn Văn Thứ

**MSSV:** K225480106062

**Lớp:** K58KTP

**Deadline:** 23h59 ngày 12/05/2026

---

## I. Cấu trúc thư mục

```
Internet
|
Cloudflare Tunnel
|
Ubuntu Server
|
my_wordpress
  |
  Docker-Compose.yml
    ├── WordPress
    ├── MariaDB
    └── phpMyAdmin
```

## II. Quy trình Setup Project

### 1. Tiến hành cài và kiểm tra phiên bản Docker & Docker-Compose

- Kiểm tra phiên bản Docker sử dụng: `docker --version`
- Kiểm tra phiên bản Docker-compose sử dụng: `docker-compose version`

<img width="666" height="178" alt="image" src="https://github.com/user-attachments/assets/efba35b9-2004-46ad-9288-51b9e713d8de" />

### 2. Tạo thư mục Project

- Tạo thư mục dự án: `mkdir my_wordpress`
- Trỏ vào thư mục dự án ***my_wordpress:*** `cd my_wordpress`

<img width="623" height="77" alt="image" src="https://github.com/user-attachments/assets/9347c66e-927d-43b7-a63a-6a8a25583b6b" />

### 3. Cấu hình tạo file docker-compose.yml

- Tạo file ***docker-compose.yml:*** `nano docker-compose.yml`
- Ban đầu khi file được tạo sẽ trống chúng ta cần thêm nội dung như sau để cấu hình.

```
services:

  mariadb:
    image: mariadb:latest
    container_name: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: thu123
      MYSQL_DATABASE: wordpressdb
      MYSQL_USER: thu123
      MYSQL_PASSWORD: thu123
    volumes:
      - mariadb_data:/var/lib/mysql
    ports:
      - "3306:3306"

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: phpmyadmin
    restart: always
    depends_on:
      - mariadb
    environment:
      PMA_HOST: mariadb
      PMA_USER: thu123
      PMA_PASSWORD: thu123
    ports:
      - "8081:80"

  wordpress:
    image: wordpress:latest
    container_name: wordpress
    restart: always
    depends_on:
      - mariadb
    environment:
      WORDPRESS_DB_HOST: mariadb:3306
      WORDPRESS_DB_USER: thu123
      WORDPRESS_DB_PASSWORD: thu123
      WORDPRESS_DB_NAME: wordpressdb
    ports:
      - "8082:80"
    volumes:
      - wordpress_data:/var/www/html

volumes:
  mariadb_data:
  wordpress_data:
```

- Thực hiện lưu file ***docker-compose.yml:*** `CTRL + O + ENTER + CTRL + X`

<img width="1477" height="756" alt="image" src="https://github.com/user-attachments/assets/f828922a-01be-49c3-a750-fd88af2faee3" />

### 4. Khởi động hệ thống

- Chạy ***docker-compose.yml:*** `docker-compose up -d`. Kết quả chạy khởi động thành công. 

<img width="1480" height="473" alt="image" src="https://github.com/user-attachments/assets/0291e76c-59ca-4a58-a7a9-96aba46e83c0" />

- Kiểm tra các container đã ở trạng thái up hoạt động chưa: `docker ps`. kết quả đã up thành công và chạy liên tục ổn định.

<img width="1481" height="446" alt="image" src="https://github.com/user-attachments/assets/e99ec823-4baf-4049-b8a5-337315b615d9" />

- 🚀 Nếu thấy 2 container ở trạng thái up: `Mariadb` và `Wordpress` là đã thành công.

***- Giải thích các Container:***

| **Container** | **Chức năng** |
|---|---|
| **mariadb** | lưu database |
| **phpmyadmin** | quản lý database |
| **wordpress** | website |

### 5. Kiểm tra truy cập Website Wordpress

- Xem kiểm tra đại chỉ IP Ubuntu: `ip a`

<img width="1481" height="377" alt="image" src="https://github.com/user-attachments/assets/057c160a-205d-41f6-bb7f-4bf1650ea58c" />

- Truy cập ***Wordpress*** lên trình duyệt: http://172.27.2.42:8082

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/46e46d94-4371-4010-be8a-59c5b588c9fe" />

- Chọn ngôn ngữ: Tiếng Việt -> Tiếp tục

***- Điền cấp thông tin website:***

| Tiêu đề trang web | Website cá nhân của Khánh|
|---|---|
| Tiêu đề trang Website | My Website |
| Tên người dùng | Nguyen Van Thu |
| Mật khẩu | T16064002hu@ |
| Email | mn9103541@gmail.com |

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/23120c46-28b9-4d21-8b2f-72deff55c350" />

- Kết quả cài đặt thành công Wordpress sẽ có thông báo trả về. Và được yêu cầu đăng nhập lại để xác minh.

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/b6f11fd5-b2ae-4c4f-bbac-7915e5d3f718" />

- Trang chủ chào mừng truy cập Wordpress thành công.

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/5102f60b-433f-42f8-9e2e-b5d18b3a733e" />

-Tạo bài viết 1: Giới thiệu bản thân. 

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/261e7b83-d893-48c3-a0ed-3d3d0fbb41fc" />

-Tạo nội dung: Chọn menu Bài viết (Posts) -> Thêm bài viết -> Xuất bản. Kết quả truy cập link: http://172.27.2.42:8082/2026/05/11/gioi-thieu-ban-than/

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/386036f5-ecc3-4fe2-89d2-8500ef2d30cd" />

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/878865ba-c8bb-4ee4-8d09-a59258b1c2f9" />

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/91272082-286c-4f40-b4d1-78328cb69212" />

### 6. Kiểm tra truy cập PhpMyAdmin

- Giao diện PhpMyAdmin:

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/85ea521d-6dc0-46da-a01c-d60a9dae3531" />

- Truy cập vào database:

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/ff793436-5a76-4637-9bfc-a3f9362bdaae" />

- Kiểm tra bảng dữ liệu:

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/59c9bd16-ccf7-4ea4-8485-30831a212996" />

### 7. Public Website Cloudfare

- Thêm respository:

```
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloudflare-main.gpg
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflare-main noble main' | sudo tee /etc/apt/sources.list.d/cloudflare-main.list
```

<img width="1918" height="163" alt="image" src="https://github.com/user-attachments/assets/5961e2ee-c0ef-4bf9-bd5e-84f5433bac79" />

- Cài Cloudfare vào Ubuntu:

```
sudo apt update
sudo apt install cloudflared -y
```

<img width="1296" height="471" alt="image" src="https://github.com/user-attachments/assets/ced14690-5da8-447b-b1e2-e54025973393" />

- Kiểm tra phiên bản: cloudflared --version

<img width="828" height="72" alt="image" src="https://github.com/user-attachments/assets/2b59f7da-9c3d-41d6-8d74-f980efbcf600" />

- Chạy lệnh: `cloudflared tunnel login`
- Ubuntu trả về link: https://dash.cloudflare.com/argotunnel?aud=&callback=https%3A%2F%2Flogin.cloudflareaccess.org%2Fr-uExv1vJFR9-njKVXAMfvdPOrEKN7vO_mhRk0rCfkQ%3D
- Coppy link dán mở lên trình duyệt.

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/9a3cd503-da1e-4bae-b191-21c0c445a288" />

- Cloudflare sẽ hỏi: `Authorize Cloudflared -> chọn domain -> nhấn Authorize`

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/f8c8092b-2e2b-4ec2-b345-0dbb1c84e71f" />

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/2c81e759-5a15-43c5-8e27-a91299a65e37" />

- Chạy lệnh: `cloudflared tunnel create wordpress-tunnel`
- Kiểm tra Tunnel: `cloudflared tunnel list`

<img width="1918" height="332" alt="image" src="https://github.com/user-attachments/assets/4afbc4fc-732b-41b4-9750-ed46380d3c8a" />

- Chạy lệnh tạo file ***config.yml:*** `nano ~/.cloudflared/config.yml`
- Nội dung file ***config.yml:***\

```
tunnel: d569eff4-be3e-4131-8a4f-4123e9d5dc64
credentials-file: /home/nguyenvanthu/.cloudflared/d569eff4-be3e-4131-8a4f-4123e9d5dc64.json

ingress:
  - hostname: mywordpress.nguyenthu.id.vn
    service: http://localhost:8082

  - service: http_status:404
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/67c03af2-7c60-457f-adee-05a07f665b92" />

- Tạo DNS cho Tunnel chạy lệnh: `cloudflared tunnel route dns wordpress-tunnel wordpress.nguyenthu.id.vn`

<img width="1727" height="50" alt="image" src="https://github.com/user-attachments/assets/30614d6e-bf31-4fb7-9363-dc6eef936122" />

- Chạy khởi động Tunnel: `cloudflared tunnel run wordpress-tunnel`

<img width="1918" height="761" alt="image" src="https://github.com/user-attachments/assets/1b473183-39ef-41b4-87e8-5441b564d9f1" />

## III. Nhận xét và đánh giá

***1. Đánh giá về công sức triển khai (Effort)***

- Việc sử dụng Docker để cài đặt WordPress giúp tối ưu hóa thời gian và công sức đáng kể so với phương pháp cài đặt LAMP/LEMP stack truyền thống:

- Tiết kiệm thời gian: Thay vì phải cài đặt riêng lẻ từng dịch vụ (Apache, PHP, MariaDB) và cấu hình kết nối thủ công, Docker Compose cho phép khởi tạo toàn bộ hệ thống chỉ với một lệnh duy nhất.

- Tính đóng gói cao: Toàn bộ cấu hình được định nghĩa trong file docker-compose.yml, giúp việc quản lý và di chuyển dự án giữa các máy chủ (từ máy vật lý sang VPS chẳng hạn) trở nên cực kỳ đơn giản và không phát sinh lỗi tương thích môi trường.

***2. Đánh giá về độ khó (Usability)***

- Đối với người dùng cuối: WordPress là một mã nguồn mở cực kỳ dễ dùng với giao diện kéo thả (Elementor) và quản lý bài viết trực quan. Ngay cả sinh viên không chuyên cũng có thể tạo ra một website chuyên nghiệp.

- Đối với quản trị viên (Admin): Việc kết hợp WordPress với Docker và Cloudflare Tunnel đòi hỏi kiến thức nền tảng tốt về hệ điều hành Linux, mạng (Networking) và quản lý cổng (Ports). Tuy nhiên, khi đã làm chủ được quy trình này, việc vận hành website trở nên rất chuyên nghiệp và an toàn.

***3. Đánh giá về tiêu tốn tài nguyên máy chủ (Resources - RAM/CPU)***

- Qua theo dõi bằng lệnh docker stats, mình có những nhận xét sau về tài nguyên trên máy chủ Ubuntu:

- RAM: Đây là thành phần tốn kém nhất.

- MariaDB: Chiếm khoảng 100MB - 150MB để duy trì CSDL.

- WordPress (PHP-FPM): Tốn khoảng 150MB - 200MB tùy thuộc vào số lượng Plugin đã cài (đặc biệt là Elementor).

- Tổng cộng: Hệ thống cần tối thiểu 1GB RAM để hoạt động mượt mà.

- CPU: Mức sử dụng rất thấp (thường < 5%) khi web ở trạng thái chờ. CPU chỉ tăng nhẹ khi xử lý hình ảnh hoặc khi có nhiều người truy cập đồng thời.

- Lưu trữ: Docker giúp quản lý dung lượng tốt, nhưng cần chú ý dọn dẹp các Image cũ để tránh đầy ổ cứng.

***4. Giải pháp Public Web qua Cloudflare Tunnel***
- Đây là điểm sáng nhất của dự án:

- Ưu điểm: Giúp đưa website cá nhân từ máy vật lý ra internet một cách bảo mật tuyệt đối mà không cần mở cổng (Open port) trên Router hay thuê IP tĩnh.

- Hiệu quả: Tên miền mywordpress.nguyenthu.id.vn hoạt động ổn định, có sẵn chứng chỉ SSL (HTTPS) miễn phí từ Cloudflare, giúp bài viết về bản thân và ngành học TNUT trông chuyên nghiệp và đáng tin cậy hơn.

***🏁 Kết luận chung***

- Việc sử dụng mã nguồn mở WordPress kết hợp với công nghệ Docker và Cloudflare Tunnel là giải pháp tối ưu cho sinh viên ngành Kỹ thuật máy tính. Nó không chỉ đáp ứng nhu cầu xây dựng Portfolio cá nhân mà còn là bài thực hành tuyệt vời về quản trị hệ thống và triển khai ứng dụng thực tế.

# <p align="center">***THE END***</p>
