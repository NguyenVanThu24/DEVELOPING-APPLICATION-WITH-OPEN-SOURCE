# <p align="center">PHÁT TRIỂN ỨNG DỤNG VỚI MÃ NGUỒN MỞ (WORDPRESSn="center">PHÁT TRIỂN ỨNG DỤNG VỚI MÃ NGUỒN MỞ (WORDPRESS + XÂY DỰNG WEBSITE GIỚI THIỆU TNUT)</p>

**Môn học:** Phát triển ứng dụng với mã nguồn mở - TEE0421  

**Họ và tên:** Nguyễn Văn Thứ

**MSSV:** K225480106062

**Lớp:** K58KTP

**Deadline:** 23h59 ngày 12/05/2026

---

## I. Cấu trúc thư mục

```

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

### 6. Kiểm tra truy cập PhpMyAdmin

### 7. Public Website Cloudfare

## III. Nhận xét và đánh giá

# <p align="center">***THE END***</p>
