# <p align="center">PHÁT TRIỂN ỨNG DỤNG VỚI MÃ NGUỒN MỞ (N8N + BOT TELAGRAM)</p>

**Môn học:** Phát triển ứng dụng với mã nguồn mở - TEE0421  

**Họ và tên:** Nguyễn Văn Thứ

**MSSV:** K225480106062

**Lớp:** K58KTP

**Deadline:** 23h59 ngày **/05/2026

---

## I. Cấu trúc thư mục

```

```

## II. Quy trình Setup Project

### GIAI ĐOẠN 1: THIẾT LẬP HẠ TẦNG (INFRASTRUCTURE SETUP).

- Mục tiêu của giai đoạn này là xây dựng một "tổ hợp" Container gồm: MariaDB, phpMyAdmin, WordPress và n8n chạy đồng bộ, chung múi giờ Việt Nam và phân quyền bảo mật tuyệt đối.

- Kiểm tra version phiên bản docker & docker-compose lệnh: `docker --version` & `docker compose version`

<img width="670" height="186" alt="image" src="https://github.com/user-attachments/assets/924994f3-7903-4f6c-8f98-2992d3992304" />

### ***Khởi tạo cấu trúc thư mục dự án***

```
mkdir -p ~/ai_content_project/mariadb_data ~/ai_content_project/wordpress_data ~/ai_content_project/n8n_data
cd ~/ai_content_project
```

<img width="1697" height="79" alt="image" src="https://github.com/user-attachments/assets/54598618-9871-470e-acdb-13926dbc8108" />

- ***Giải thích:*** Việc tạo sẵn các thư mục này giúp Docker "gắn" (Volume Mount) dữ liệu từ Container vào máy Ubuntu của bạn. Sau này tắt máy hay update Docker thì bài viết WordPress và cấu hình n8n của bạn không bị mất.

- Viết file ***docker-compose.yml*** hoàn chỉnh

```
version: '3.8'

services:
  # 1. Hệ quản trị cơ sở dữ liệu MariaDB
  mariadb:
    image: mariadb:latest
    container_name: mariadb_server
    restart: always
    environment:
      TZ: Asia/Ho_Chi_Minh
      MYSQL_ROOT_PASSWORD: thu123
    volumes:
      - ./mariadb_data:/var/lib/mysql
    ports:
      - "3306:3306"

  # 2. Công cụ quản lý DB phpMyAdmin
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: phpmyadmin_tool
    restart: always
    depends_on:
      - mariadb
    environment:
      PMA_HOST: mariadb
      TZ: Asia/Ho_Chi_Minh
    ports:
      - "8081:80"

  # 3. Nền tảng CMS WordPress
  wordpress:
    image: wordpress:latest
    container_name: wordpress_site
    restart: always
    depends_on:
      - mariadb
    environment:
      TZ: Asia/Ho_Chi_Minh
      WORDPRESS_DB_HOST: mariadb:3306
      WORDPRESS_DB_USER: wp_automation        # User này lát nữa ta sẽ tạo trong phpMyAdmin
      WORDPRESS_DB_PASSWORD: thu123
      WORDPRESS_DB_NAME: wp_ai_db             # Database này lát nữa ta sẽ tạo trong phpMyAdmin
    ports:
      - "8082:80"
    volumes:
      - ./wordpress_data:/var/www/html

  # 4. Công cụ tự động hóa n8n
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n_automation
    restart: always
    environment:
      - TZ=Asia/Ho_Chi_Minh
    ports:
      - "5678:5678"
    volumes:
      - ./n8n_data:/home/node/.n8n
```

- Gõ lệnh: `nano docker-compose.yml` và dán toàn bộ nội dung cấu hình chuẩn hóa bên trên đây vào. Mật mã dùng chung `thu123`

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/c3ee1bdd-a0b3-4a3b-b381-fab58b175050" />

➡️ ***(Nhấn Ctrl + O, Enter để lưu và Ctrl + X để thoát).***

### ***Khởi chạy hạ tầng hệ thống***

- docker-compose up -d # kích hoạt hệ thống bằng lệnh

<img width="1919" height="568" alt="image" src="https://github.com/user-attachments/assets/cbb486fd-bed6-48ee-9099-5654a8a19960" />

- docker-compose # Kiểm tra các container đã vào trạng thái sãn sàng
  
<img width="1919" height="587" alt="image" src="https://github.com/user-attachments/assets/18f1d3fb-aa4d-4b4d-b3fb-76debbc52579" />



# <p align="center">***THE END***</p>
