# <p align="center">PHÁT TRIỂN ỨNG DỤNG VỚI MÃ NGUỒN MỞ (DJANGO + XÂY DỰNG WEBSITE QUẢN LÝ TIỆM CẦM ĐỒ)</p>

**Môn học:** Phát triển ứng dụng với mã nguồn mở - TEE0421  

**Họ và tên:** Nguyễn Văn Thứ

**MSSV:** K225480106062

**Lớp:** K58KTP

**Deadline:** 23h59 ngày 09/05/2026

---

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

