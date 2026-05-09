# <p align="center">PHÁT TRIỂN ỨNG DỤNG VỚI MÃ NGUỒN MỞ (DJANGO + XÂY DỰNG WEBSITE QUẢN LÝ TIỆM CẦM ĐỒ)</p>

**Môn học:** Phát triển ứng dụng với mã nguồn mở - TEE0421  

**Họ và tên:** Nguyễn Văn Thứ

**MSSV:** K225480106062

**Lớp:** K58KTP

**Deadline:** 23h59 ngày 09/05/2026

---

## Tổ chức CSDL cho hệ thống quản lý tiệm cầm đồ

<img width="1218" height="1280" alt="z7807150835857_7424d6541b76d541b7561982cea24533" src="https://github.com/user-attachments/assets/e02e9793-78d4-4bef-9f26-4b120c349ecf" />

***Giải thích:***
- Một khách hàng khi đến tiệm có thể thực hiện cầm đồ nhiều lần khác nhau, vì vậy thông tin của một khách hàng sẽ liên kết với nhiều hợp đồng cầm đồ.
- Mỗi món đồ cầm cố chỉ được ghi nhận cho một hợp đồng cụ thể để dễ theo dõi nguồn gốc và giá trị tài sản.
- Trong quá trình vay, khách hàng có thể trả tiền thành nhiều đợt khác nhau, do đó một hợp đồng sẽ phát sinh nhiều lần thanh toán.

## Triển khai cài đặt

- Chạy kệnh:
```
sudo apt update
sudo apt upgrade -y
```
<img width="1477" height="757" alt="image" src="https://github.com/user-attachments/assets/9415832a-e154-4dec-962a-2de9c38f5a5e" />

- Kiểm tra phiên bản docker & docker compose
```
docker --version
docker-compose version
```
<img width="1481" height="762" alt="image" src="https://github.com/user-attachments/assets/efc44d14-3998-439f-b3e6-3fabb65ffb5e" />

- Tạo thư mục
```
mkdir camdo_project
cd camdo_project
```

- Tạo thư mục Django `mkdir django_app`
<img width="1480" height="757" alt="image" src="https://github.com/user-attachments/assets/6dfc55d6-87b3-4235-9823-688730cee8a9" />

- Vào thư mục: `cd django_app`

- Tạo file `sudo nano Dockerfile`
```
# Sử dụng Python 3.12 chính thức
FROM python:3.12

# Không tạo file pyc
ENV PYTHONDONTWRITEBYTECODE=1

# Hiển thị log realtime
ENV PYTHONUNBUFFERED=1

# Thư mục làm việc trong container
WORKDIR /app

# Copy requirements vào container
COPY requirements.txt .

# Cài thư viện Python
RUN pip install --no-cache-dir -r requirements.txt

# Copy toàn bộ source code
COPY . .

# Mở cổng Django
EXPOSE 8000

# Chạy Django server
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```
<img width="1482" height="757" alt="image" src="https://github.com/user-attachments/assets/f7635f2d-067a-49f4-bdf0-030f978ea43a" />

- Tạo file: `sudo nano requirements.txt`
```
# Framework web Django
django

# Driver kết nối MariaDB/MySQL
mysqlclient

# Thư viện hỗ trợ MySQL
PyMySQL
```
<img width="1481" height="758" alt="image" src="https://github.com/user-attachments/assets/9471e590-76f5-49da-b242-e6a6dce63c7e" />

- Tạo file `sudo nano docker-compose.yml`
```
services:
  mariadb:
    image: mariadb:11
    container_name: camdo_mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: camdo_db
      MYSQL_USER: camdo_user
      MYSQL_PASSWORD: camdo123
    ports:
      - "3307:3306"
    volumes:
      - mariadb_data:/var/lib/mysql
    # Thêm kiểm tra trạng thái để báo cho Django khi nào DB thực sự sẵn sàng
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 5s
      timeout: 5s
      retries: 10

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: camdo_phpmyadmin
    restart: always
    ports:
      - "8080:80"
    environment:
      PMA_HOST: mariadb
      MYSQL_ROOT_PASSWORD: root123
    depends_on:
      mariadb:
        condition: service_healthy

  django:
    build: ./django_app
    container_name: camdo_django
    restart: always
    ports:
      - "8000:8000"
    volumes:
      - ./django_app:/app
    environment:
      - DATABASE_HOST=mariadb
      - DATABASE_NAME=camdo_db
      - DATABASE_USER=camdo_user
      - DATABASE_PASSWORD=camdo123
    # Ép Django đợi MariaDB vượt qua bài kiểm tra Healthcheck
    depends_on:
      mariadb:
        condition: service_healthy

volumes:
  mariadb_data:
```
<img width="1918" height="1020" alt="image" src="https://github.com/user-attachments/assets/eaefba85-1261-4399-bf01-0280fbff57ea" />

- Build và chạy lại container Chạy lệnh: `docker-compose up --build -d`
<img width="1918" height="1026" alt="image" src="https://github.com/user-attachments/assets/435792ef-a038-4e36-a24c-45184c0f545b" />

- Tạo Django project
Vào container Django: docker-compose exec django bash
Tạo project: django-admin startproject camdo .
Tạo app: python manage.py startapp management

- Cầu hình Django
Mở settings.py: nano camdo/settings.py
Sửa ALLOWED_HOSTS
Tìm dòng: ALLOWED_HOSTS = []
Sửa thành: ALLOWED_HOSTS = ['*']
Thêm App management
Tìm dòng: INSTALLED_APPS = [
Thêm: 'management',

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/d7c34f8f-1cba-4bbd-aee8-c39e177b12c7" />

- Sửa Databases
Tìm đoạn: DATABASES = {
Xóa toàn bộ block đó và thay bằng:

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/a4b707f0-078e-4f17-9bfb-7688fe670860" />

- Tạo Model
Mở file: nano management/models.py
Nội dung:

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/02e04bd5-aa9b-4e5c-8384-2b880d7c23ca" />

 - Tạo bảng Database
Tạo Migration: python manage.py makemigrations
<img width="1078" height="207" alt="image" src="https://github.com/user-attachments/assets/4238ea12-cc4c-4ecf-9f82-6403d995cfe5" />

- Tạo Migrate: python manage.py migrate
<img width="870" height="560" alt="image" src="https://github.com/user-attachments/assets/fc26eb5e-65f9-4841-8727-63eb7f950ed2" />

- Tạo Admin Django
Mở file: nano management/admin.py
Nội dung:
```
from django.contrib import admin

from .models import *

admin.site.register(Customer)
admin.site.register(PawnItem)
admin.site.register(LoanContract)
admin.site.register(Payment)
```

<img width="1918" height="1020" alt="image" src="https://github.com/user-attachments/assets/9f1a2170-f88a-407a-9b3d-738bff8f927c" />

- Tạo tài khoản Admin
Chạy lệnh: python manage.py createsuperuser
Nhập thông tin:
Username: admin
Email: Tùy ý
Password: Tùy ý

<img width="1372" height="247" alt="image" src="https://github.com/user-attachments/assets/9a10c632-4a8c-4c2b-8721-319f81d196cc" />

- Mở Django Admin
Trên trình duyệt, truy cập: http://172.27.2.42:8000/admin
Giao diện đăng nhập tài khoản Admin:
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/ca07ca4d-11b7-4706-8441-24280fdc9dca" />

- Giao diện sau khi đăng nhập:
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/859e34f2-b213-42b5-8d03-2b8da2cd8348" />

- Thêm khách hàng
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/32239ac6-64d4-48ee-b3e9-532e477eed68" />

- Thêm khách hang thành công
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/155196c7-b621-463c-bb9f-700e9af76f93" />

- Thêm hợp đồng vay
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/45ce51c5-c5cf-407d-9a90-8ece407db7ab" />

- Thêm hợp đồng vay thành công
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/21eb8339-8c37-430c-98e1-d61ecd1db07c" />

- Kiểm tra phpMyAdmin
Truy cập: http://192.168.91.154:8080/
Đăng nhập tài khoản:

| Field | Value | 
|---|---|
| server | mariadb |
| user | root | 
| password | root123 | 

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/41e2d822-006e-45f8-a385-9984c93fbe09" />

- Giao diện sau khi đăng nhập thành công
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/e36b96df-2d4f-4e71-9bb9-5069aff50485" />

- Kiểm tra dữ liệu của các bảng sau khi thêm thông tin thành công
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/6ca21d90-99c5-4e51-a500-926b2a3a6acd" />

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/84fca410-e8e9-42cd-a49e-5b15a75786a9" />

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/9d78cc23-b542-4b13-9a53-ff264e3e582d" />

- Tạo Template HTML
Vào thư mục project: cd ~/pawnshop_project/django_app
Tạo thư mục: mkdir templates
Tạo file: nano templates/home.html
Nội dung file:
```
<!DOCTYPE html>
<html>

<head>
    <title>Danh sách nợ quá hạn</title>
</head>

<body>

<h1>Danh sách khách nợ quá hạn</h1>

<table border="1">

<tr>
    <th>Khách hàng</th>
    <th>Tài sản</th>
    <th>Số tiền vay</th>
    <th>Ngày đến hạn</th>
</tr>

{% for contract in contracts %}

<tr>

    <td>{{ contract.customer.full_name }}</td>

    <td>{{ contract.pawn_item.item_name }}</td>

    <td>{{ contract.loan_amount }}</td>

    <td>{{ contract.due_date }}</td>

</tr>

{% endfor %}

</table>

</body>
</html>
```
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/ca1b56d9-a1c7-474c-ac0a-efa9e1936a12" />

- Tạo view
Mở file: nano management/views.py
Nội dung:
```

```
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/9d68ffbc-6835-40f5-bea8-ebd5c9ea89e4" />

- Tạo URL
Mở file: nano pawnshop/urls.py
Nội dung:
```

```
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/51e15c07-f802-4ff5-a0c3-32ee64742c54" />

- Restart lại
Chạy lệnh: cd ~/camdo_project
docker-compose restart django
<img width="947" height="132" alt="image" src="https://github.com/user-attachments/assets/84d94746-4629-4f94-8956-45a204d4ec93" />

- Truy cập: http://172.27.2.42:8000/
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/e2e76177-47d8-4c83-a90a-18c5d1eae0a3" />
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/5f4aab44-8a28-43d5-ab23-5cf585797c32" />

- Cài đặt Cloudflared chạy các lệnh này để cài đặt gói:
```
# Di chuyển vào thư mục tạm
cd /tmp

# Tải gói cài đặt
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
```

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/448a88be-7c09-4d1f-b5f4-00cb8c688b8a" />

```
Cài đặt gói vào hệ thống
sudo dpkg -i cloudflared-linux-amd64.deb

Kiểm tra: cloudflared --version
```

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/d8f6eb26-e225-4dc9-9da8-ee3b23cce3d5" />

- Đăng nhập Cloudflare
Chạy: cloudflared tunnel login
Ubuntu hiện Link: https://dash.cloudflare.com/argotunnel?aud=&callback=https%3A%2F%2Flogin.cloudflareaccess.org%2Fes5aa6kW-3TK6wyxWNs-UORrwvDlrM4PTtbXUwGsnXk%3D Copy Link chạy lên trình duyệt.

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/7809a33b-6fb6-4fc0-9071-4e782aecd9f0" />
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/e770dc10-9b0f-48a6-b4db-06fd514a4f7a" />
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/b35c2468-58fc-4160-870a-198804443c28" />

- Cloudflare sẽ hỏi: Authorize Cloudflared -> chọn domain -> nhấn Authorize
- Trong Terminal Ubuntu sẽ hiện: You have successfully logged in.

<img width="1732" height="233" alt="image" src="https://github.com/user-attachments/assets/db93bdde-b6dd-4340-a189-950ed3e83eec" />

<img width="1918" height="151" alt="image" src="https://github.com/user-attachments/assets/e3c494d3-ea95-4877-bb6c-e1f2f5a3aab5" />

- ID: 7a37f0fe-ad37-4b68-a18b-099ecfc18272

- Tạo file config
Tạo file: mkdir -p ~/.cloudflared
Chạy lệnh: nano ~/.cloudflared/config.yml
Nội dung file:
```
tunnel: 7a37f0fe-ad37-4b68-a18b-099ecfc18272
credentials-file: /home/nguyenvanthu/.cloudflared/7a37f0fe-ad37-4b68-a18b-099ecfc18272.json

ingress:
  - hostname: nguyenthucamdo.id.vn
    service: http://localhost:8000
  - service: http_status:404
```

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/b2fd8b81-060b-4963-97a1-c2dab7f2f917" />

- Tạo DNS
Chạy lệnh: 
```
cloudflared tunnel route dns -f camdo nguyenthu.id.vn
cloudflared tunnel run camdo
```

<img width="1917" height="810" alt="image" src="https://github.com/user-attachments/assets/a09ee396-dbdb-4d7c-b446-87148cb59b74" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6860e9f3-c578-47ba-b399-8ea49ceb6060" />

- Lưu ý:
Không được tắt Terminal.
Nếu tắt -> Web sẽ off

- Truy cập website
Truy cập: https://nguyenthu.id.vn/

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/44c5852f-347a-4eb5-b488-7fbc7d419043" />

- Truy cập Admin Django: https://nguyenthu.id.vn/admin/login/?next=/admin/

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/ac8214c7-314c-4a49-8ea9-a961f003f703" />

- Khi đăng nhập vào Admin Django sẽ bị lỗi hệ thống

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/4fd7e047-71ce-4252-8d35-6d9091042bc1" />

- Cần bổ sung trong file settings.py
```
Gõ lệnh: sudo nano django_app/pawnshop/settings.py
Thêm CSRF_TRUSTED_ORIGINS
```

```
CSRF_TRUSTED_ORIGINS = [
    "https://nguyenthu.id.vn",
]
```

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/05531da3-0800-4d17-8529-2815f4df525d" />

- Restart lại: cd ~/camdo_project && docker-compose restart django

- Trước khi truy cập lại trang web, cần khởi chạy lại Cloudflare Tunnel: cloudflared tunnel run camdo

Nếu không thực hiện bước này, hệ thống sẽ không tạo kết nối ra Internet, vì vậy website sẽ không thể truy cập từ trình duyệt bên ngoài.
- Truy cập lại: https://nguyenthu.id.vn/admin và đăng nhập

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/b504ddd7-5475-4489-8283-2b86b1ceb6ab" />

## Đánh giá
- Sau khi hoàn thành bài tập, hệ thống quản lý tiệm cầm đồ đã được xây dựng thành công dựa trên Django, kết hợp với MariaDB làm hệ quản trị cơ sở dữ liệu và phpMyAdmin để hỗ trợ quan sát dữ liệu. Toàn bộ hệ thống được triển khai trong môi trường Docker trên Ubuntu, giúp việc cài đặt, vận hành và quản lý trở nên thống nhất và dễ dàng hơn.
- Trong quá trình thực hiện, em đã xây dựng được các thành phần chính của một ứng dụng web hoàn chỉnh như: thiết kế cơ sở dữ liệu, định nghĩa models trong Django, tạo giao diện quản trị (Django Admin) để thực hiện các thao tác thêm, sửa, xóa dữ liệu, đồng thời xây dựng giao diện người dùng bằng template HTML để hiển thị danh sách các khách hàng/con nợ đến hạn.
- Bên cạnh đó, hệ thống còn được tích hợp Cloudflare Tunnel để public website ra Internet thông qua domain riêng, giúp có thể truy cập từ bên ngoài một cách thuận tiện. Việc triển khai này giúp em hiểu rõ hơn về cách đưa một ứng dụng web từ môi trường local lên môi trường internet thực tế.
- Qua bài tập này, em đã nắm vững hơn các kiến thức về Django, Docker, cơ sở dữ liệu quan hệ và quy trình triển khai hệ thống web hoàn chỉnh. Đồng thời, em cũng rèn luyện được kỹ năng xử lý lỗi, cấu hình hệ thống và làm việc với nhiều công nghệ tích hợp trong một dự án thực tế.


# <p align="center">***THE END***</p>
