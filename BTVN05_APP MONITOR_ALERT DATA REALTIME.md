# <p align="center">PHÁT TRIỂN ỨNG DỤNG VỚI MÃ NGUỒN MỞ (APP MONITOR + ALERT DATA REALTIME)</p>

**Môn học:** Phát triển ứng dụng với mã nguồn mở - TEE0421  

**Họ và tên:** Nguyễn Văn Thứ

**MSSV:** K225480106062

**Lớp:** K58KTP

**Deadline:** 23h59 ngày 13/06/2026

---

# YÊU CẦU BÀI TẬP
## LÝ THUYẾT
```
+ docker là gì? 
+ các keyword được sử dụng trong docker-compose.yml
  để mô tả 1 service, network, volume,...
  liệt kê + ý nghĩa của từ khoá đó + ví dụ minh hoạ
+ ưu điểm khi triển app sử dụng docker là gì?
+ dùng docker: tạo app, test app OK trên laptop cá nhân
  giờ muốn triển khai app này trên máy chủ thật ko có internet
  thì các bước cần làm là?
```

## THỰC HÀNH ÁP DỤNG
```
sử dụng docker compose có nhiều serivce 
và các thành phần cần thiết để tạo thành ứng dụng:
 + nodered liên tục lấy dữ liệu từ nguồn nào đó (chứng khoán, thời tiết, giá vàng,...)
   nguồn thực tế, số liệu luôn động sau thời gian ngắn
 + nodered lưu trữ dữ liệu vào 2 database: mariadb để lưu giá trị tức thời
   lưu lịch sử vào influxdb
 + sử dụng grafana để trực quan hoá dữ liệu: vẽ biểu đồ
 + sử dụng nginx để làm webserver
   chạy 1 trang web html+js+css làm front-end
   js: lấy dữ liệu tức thời trong mariadb qua (ajax | socket) 
       gọi api (api tự build bằng Flask giống bt1)
       api trả về giá trị tức thời trong mariadb
       hiển thị lên web, auto hiển thị số mới khi thay đổi
   sử dụng iframe để gọi grafana
   hiển thị biểu đồ dữ liệu lịch sử của thông số đã lưu
 + QUAN SÁT DỮ LIỆU LỊCH SỬ => GIÁ TRỊ BẤT THƯỜNG
   (VD MIỀN A..B: OK, DƯỚI A: ALERT LOW, TRÊN B: ALERT HIGH)
 + nodered: kết hợp bot Telegram
   khi dữ liệu not OK, thì gửi tin nhắn từ bot => group trên telegram
   group đã add bot vào: (nhóm đã có 2 người), add thêm 1875746636 thành 3 người
   mỗi khi bot gửi dữ liệu vào nhóm: mọi member of group đều nhận đc
   nội dung alert: tường minh, có value gây alert

 xuất tất cả các container ra file nén.
 xoá mọi container đang chạy
 load lại các container  từ file nén để khôi phục các container đã xoá
```

---

# BÀI LÀM
## PHẦN 1: LÝ THUYẾT
### 1. Docker là gì?
Docker là một nền tảng mã nguồn mở giúp đóng gói ứng dụng và toàn bộ môi trường chạy của nó (thư viện, cấu hình, dependencies) vào một đơn vị gọi là Container.

Trong phát triển phần mềm truyền thống, thường xảy ra tình huống: ứng dụng chạy tốt trên máy lập trình viên nhưng lại lỗi khi triển khai lên server — do khác biệt về phiên bản thư viện, hệ điều hành, hoặc cấu hình môi trường. Docker giải quyết vấn đề này bằng cách đóng gói toàn bộ môi trường vào container, đảm bảo ứng dụng chạy giống hệt nhau trên mọi máy.

Một số khái niệm chính:
| Khái niệm | Mô tả |
| --- | --- |
| image | Bản thiết kế (blueprint) của container — chỉ đọc, không sửa được. Giống như class trong OOP. |
| container | Instance đang chạy được tạo từ image. Giống như object được khởi tạo từ class.|
| Dockerfile | 	File hướng dẫn từng bước để build một image tùy chỉnh. |
| Docker Hub | Kho lưu trữ image công khai trực tuyến (tương tự GitHub nhưng dành cho image). |
| volume | Cơ chế lưu trữ dữ liệu bền vững bên ngoài container, không bị mất khi container bị xoá |
| network | Mạng ảo để các container giao tiếp với nhau một cách an toàn và có kiểm soát. |
| docker compose | Công cụ định nghĩa và chạy nhiều container cùng lúc qua file docker-compose.yml. |

---

### 2. Các keyword trong file `docker-compose.yml`
File docker-compose.yml là file cấu hình YAML dùng để mô tả toàn bộ hệ thống multi-container. 

Cấu trúc tổng quát:
```
version: '3.8'      # phiên bản cú pháp Compose

services:           # định nghĩa các container
  ten_service:
    ...

networks:           # định nghĩa mạng ảo
  ten_network:
    ...

volumes:            # định nghĩa ổ đĩa ảo
  ten_volume:
    ...
```

---

#### 2.1. Một số keyword chính trong docker-compose

| Keyword	| Ý nghĩa |
| --- | --- |
| version	| Phiên bản cú pháp Docker Compose |
| services	| Danh sách các container cần chạy trong hệ thống |
| networks | Định nghĩa các mạng ảo dùng chung giữa các service |
| volumes	| Định nghĩa các ổ đĩa ảo dùng chung giữa các service |

---

#### 2.2. Các keyword mô tả service
`image`

Chỉ định image có sẵn (từ Docker Hub hoặc local) để chạy container.

```
services:
  database:
    image: mariadb:latest   # dùng image mariadb phiên bản mới nhất
```

`build`

Thay vì dùng image có sẵn, tự build image từ Dockerfile.

```
services:
  flask_api:
    build:
      context: ./flask_app    # thư mục chứa Dockerfile
      dockerfile: Dockerfile  # tên file (mặc định là "Dockerfile")
```

`container_name`

Đặt tên cụ thể cho container thay vì để Docker tự sinh tên ngẫu nhiên.

```
services:
  web:
    image: nginx
    container_name: my_nginx
```

`ports`

Map cổng theo cú pháp HOST:CONTAINER — cho phép truy cập từ bên ngoài máy host.

```
services:
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"   # truy cập http://localhost:3000 → grafana bên trong container
```

`environment`

Truyền biến môi trường vào trong container

```
services:
  mariadb:
    image: mariadb:latest
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: alert_db
      MYSQL_USER: admin
      MYSQL_PASSWORD: admin123
```

`env_file`

Đọc biến môi trường từ file .env bên ngoài — tránh lộ thông tin nhạy cảm trong compose file.

```
services:
  mariadb:
    env_file:
      - .env
```

`volumes`

Mount dữ liệu giữa host và container. Có 2 loại:

- Named volume: Docker tự quản lý vị trí lưu trữ.
- Bind mount: Mount trực tiếp thư mục/file từ máy host.

```
services:
  mariadb:
    volumes:
      - mariadb_data:/var/lib/mysql        # named volume
      - ./config/my.cnf:/etc/mysql/my.cnf  # bind mount (file cụ thể)
```

`networks`

Chỉ định container tham gia vào mạng nào. Một container có thể thuộc nhiều mạng.

```
services:
  flask_api:
    networks:
      - frontend_net
      - backend_net
```

`depend_on`

Xác định thứ tự khởi động — service này chỉ start sau khi các service phụ thuộc đã sẵn sàng.

```
services:
  flask_api:
    depends_on:
      - mariadb    # mariadb phải khởi động trước flask_api
      - influxdb
```

`restart`

Chính sách tự khởi động lại container khi bị crash hoặc khi Docker daemon restart

```
services:
  nodered:
    restart: always          # luôn luôn restart
    # restart: unless-stopped  # restart trừ khi bị dừng thủ công bằng docker stop
    # restart: on-failure      # chỉ restart khi thoát với mã lỗi khác 0
    # restart: no              # không bao giờ tự restart
```

`command`

Ghi đè lệnh mặc định (CMD) được định nghĩa trong image khi container khởi động.

```
services:
  flask_api:
    command: python app.py --port 5000
```

`healthcheck`

Định nghĩa câu lệnh kiểm tra định kỳ xem service có hoạt động đúng không.

```
services:
  mariadb:
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s   # kiểm tra mỗi 10 giây
      timeout: 5s     # timeout sau 5 giây
      retries: 5      # thử lại 5 lần trước khi đánh dấu "unhealthy"
```

`expose`

Mở cổng cho các container khác trong cùng network — không mở ra ngoài máy host.

```
services:
  flask_api:
    expose:
      - "5000"   # container khác trong cùng network có thể gọi :5000
                 # nhưng máy host bên ngoài không truy cập được
```

`entrypoint`

Ghi đè lệnh ENTRYPOINT mặc định của image.

```
services:
  flask_api:
    entrypoint: ["python", "-m", "flask", "run"]
```

`working_dir`

Đặt thư mục làm việc mặc định bên trong container.

```
services:
  flask_api:
    working_dir: /app
```

---

#### 2.3. Các keyword mô tả network

| Driver	| Ý nghĩa |
| --- | --- |
| bridge	| Mạng ảo riêng — các container trong cùng bridge giao tiếp được với nhau qua tên service |
| host	| Container dùng thẳng network interface của máy host (không cô lập network) |
| none	| Container không có kết nối mạng |

---

#### 2.4. Các keyword mô tả volumes

```
volumes:
  mariadb_data:       # Docker tự quản lý vị trí lưu trữ
    driver: local

  influxdb_data:
    driver: local

  grafana_data:
    driver: local
```
Named volume được lưu tại /var/lib/docker/volumes/ trên máy host. Dữ liệu tồn tại độc lập với vòng đời container

---

### 3. Ưu điểm khi triển khai ứng dụng bằng Docker

| STT |	Ưu điểm	| Giải thích |
| --- | --- | --- |
| 1 |	Nhất quán môi trường	| Dev, test, staging, production chạy giống hệt nhau. Xoá bỏ vấn đề "chạy được trên máy tôi".|
| 2	| Cô lập (Isolation)	| Mỗi service chạy trong container riêng, không ảnh hưởng lẫn nhau. Lỗi ở một service không kéo đổ cả hệ thống.|
| 3 |	Triển khai nhanh chóng	| Chỉ cần một lệnh docker compose up -d để khởi động toàn bộ hệ thống nhiều service.|
| 4	| Dễ dàng scale	| Tăng số lượng instance của một service nhanh chóng: docker compose scale web=3 | 
| 5	| Dễ rollback	| Quay về phiên bản cũ đơn giản bằng cách đổi tag image và restart.|
| 6	| Tiết kiệm tài nguyên	| Container nhẹ hơn nhiều so với Virtual Machine vì dùng chung kernel với máy host, không cần boot OS riêng. |
| 7 |	Quản lý dependencies	| Mỗi container tự mang dependencies riêng — không bao giờ xảy ra xung đột phiên bản giữa các service. |
| 8	| Dễ backup và restore	| Export/import image hoặc volume chỉ với vài lệnh đơn giản. |
| 9	| CI/CD thân thiện	| Tích hợp dễ dàng vào các pipeline tự động hoá (GitHub Actions, GitLab CI,...).|
| 10	| Tái sử dụng |	Image được build một lần, có thể chạy ở bất kỳ đâu có Docker mà không cần cấu hình lại. |

---

### 4. Triển khai lên máy chủ thật không có internet
#### Bước 1: Trên Laptop : Build và Pull tất cả các images
```
# Pull tất cả images được khai báo trong docker-compose.yml
docker compose pull

# Build các image tự viết (nếu có dùng keyword "build:" trong compose file)
docker compose build
```
#### Bước 2: Trên Laptop: Export images ra file nén

```
# Export từng image riêng lẻ
docker save mariadb:10.11      -o mariadb.tar
docker save influxdb:2.7       -o influxdb.tar
docker save grafana/grafana    -o grafana.tar
docker save nodered/node-red   -o nodered.tar
docker save nginx:alpine       -o nginx.tar
docker save my_flask_api:latest -o flask_api.tar

# Export tất cả vào 1 file nén duy nhất (khuyến nghị)
docker save \
  mariadb:10.11 \
  influxdb:2.7 \
  grafana/grafana \
  nodered/node-red \
  nginx:alpine \
  my_flask_api:latest \
  | gzip > all_images.tar.gz
```
#### Bước 3: Copy lên máy chủ

```
# Dùng SCP (truyền qua SSH)
scp all_images.tar.gz user@192.168.1.100:/home/user/

# Copy toàn bộ project (bao gồm docker-compose.yml, config, source code,...)
scp -r ./my_project user@192.168.1.100:/home/user/
```
Nếu không có SSH, có thể copy qua USB, ổ cứng ngoài, hoặc mạng nội bộ LAN.

#### Bước 4: Load images từ file

```
# Load images từ file nén
docker load -i all_images.tar.gz
# Hoặc:
gunzip -c all_images.tar.gz | docker load

# Kiểm tra images đã được load thành công
docker images
```

#### Bước 5:  Trên Máy chủ: Vào thư mục project

```
cd /home/user/my_project
ls -la
# Kiểm tra có đủ file: docker-compose.yml, .env, nginx.conf, frontend/, flask_api/,...
```
#### Bước 6: Khởi động hệ thống

```
# Chạy toàn bộ hệ thống ở chế độ nền (detached mode)
docker compose up -d

# Kiểm tra trạng thái các container
docker compose ps

# Xem log nếu có lỗi
docker compose logs -f
```

#### Một số lệnh quan trọng

`docker compose up -d`	Khởi động tất cả service ở chế độ nền

`docker compose down`	Dừng và xoá tất cả container (giữ volumes)

`docker compose ps`	Xem trạng thái các container

`docker compose logs -f`	Xem log realtime

`docker save IMAGE -o file.tar`	Export image ra file

`docker load -i file.tar`	Load image từ file

`docker images`	Liệt kê tất cả images hiện có

`docker compose pull`	Pull images từ registry

`docker compose build`	Build images từ Dockerfile

---

## PHẦN 2: THỰC HÀNH DỰ ÁN: HỆ THỐNG GIÁM SÁT VÀ CẢNH BÁO THỊ TRƯỜNG TIỀN ĐIỆN TỬ REAL-TIME (CRYPTO VOLATILITY MONITOR)
### 1. Tổng quan hệ thống
#### 1.1. Giới thiệu 

Dự án xây dựng một hệ thống tự động hóa khép kín theo kiến trúc Microservices dựa trên nền tảng Docker và Docker Compose. Hệ thống thực hiện thu thập dữ liệu biến động giá của thị trường tiền điện tử (Crypto) theo thời gian thực (Real-time) từ API sàn Binance, thực hiện lưu trữ song song vào hai loại cơ sở dữ liệu khác nhau (Quan hệ và Chuỗi thời gian), trực quan hóa dữ liệu lên Dashboard và tự động phân tích để đưa ra cảnh báo tức thời qua Telegram Bot khi thị trường có biến động bất thường (Bơm/Xả coin).

Các thành phần cốt lõi trong hệ thống được chia thành 4 lớp kiến trúc tường minh:
```
[ LỚP THU THẬP & XỬ LÝ ] ➡️ Node-RED (Cào API Binance & Phân tích ngưỡng Alert)
                                 👇
[  LỚP LƯU TRỮ (DB)  ] ➡️ MariaDB (Lưu giá tức thời) & InfluxDB (Lưu lịch sử)
                                 👇
[ LỚP TRUNG GIAN API ] ➡️ Flask API (Python) kết nối dữ liệu
                                 👇
[  LỚP HIỂN THỊ (UI) ] ➡️ Nginx Webserver (Front-end HTML/JS) & iframe Grafana
```

- Node-RED (Core Automation Engine): Đóng vai trò là trung tâm điều phối dữ liệu. Định kỳ mỗi 5 giây sẽ gọi API của Binance lấy giá mới nhất, phân tích kỹ thuật để phát hiện giá trị bất thường (Alert High/Low), sau đó đẩy dữ liệu đồng thời về 2 cơ sở dữ liệu và kích hoạt Bot Telegram gửi cảnh báo vào nhóm chung khi có biến động.

- MariaDB (Relational Database): Chịu trách nhiệm lưu trữ Giá trị tức thời (Real-time Data). Database này cấu hình nhẹ, luôn chỉ lưu trữ duy nhất một dòng dữ liệu mới nhất của phiên giao dịch để tối ưu hóa tốc độ truy xuất cho tầng hiển thị.

- InfluxDB (Time-Series Database): Chịu trách nhiệm lưu trữ Dữ liệu lịch sử (Historical Data). Định dạng chuỗi thời gian của InfluxDB giúp tối ưu hóa dung lượng lưu trữ cho hàng triệu bản ghi theo từng giây, phục vụ riêng cho việc vẽ biểu đồ trồi sụt của thị trường.

- Flask API (Backend Service): Xây dựng bằng Python, đóng vai trò làm cầu nối (API trung gian). Flask sẽ chọc vào MariaDB lấy giá trị tức thời rồi khạc ra dữ liệu dạng JSON phục vụ cho các tác vụ gọi AJAX/Socket từ Front-end.

- Grafana (Data Visualization): Kết nối trực tiếp vào InfluxDB để bốc dữ liệu lịch sử và trực quan hóa thành biểu đồ kỹ thuật (biểu đồ hình sin, biểu đồ nến). Cấu hình cho phép nhúng (Embedding) để hiển thị trực tiếp trên giao diện người dùng.

- Nginx (Webserver & Front-end proxy): Chạy một trang web tĩnh (HTML + CSS + Javascript). Sử dụng AJAX để gọi Flask API cập nhật số liệu tự động nhảy tanh tách trên màn hình, kết hợp thẻ <iframe> nhúng biểu đồ động từ Grafana.

---

#### 1.2. Cấu trúc thư mục dự án
```
crypto-monitor/
├── docker-compose.yml          # File cấu hình tổng lực kích hoạt toàn bộ hệ thống
├── nginx/                      # Cấu hình dịch vụ Webserver Nginx
│   ├── default.conf            # File cấu hình phân luồng Proxy và Port nội bộ
│   └── html/                   # Giao diện chính người dùng (Front-end)
│       └── index.html          # Trang bảng điện tử hiển thị giá & iframe Grafana
├── flask_api/                  # Dịch vụ Backend API (Python Flask)
│   ├── Dockerfile              # File đóng gói môi trường chạy Python chuyên nghiệp
│   ├── requirements.txt        # Danh sách các thư viện cần cài (Flask, Connector...)
│   └── app.py                  # Mã nguồn xử lý gọi dữ liệu real-time từ MariaDB
├── nodered_data/               # Thư mục lưu trữ toàn bộ luồng kéo giá (Flows) của Node-RED
├── mariadb_data/               # Nơi lưu trữ vĩnh viễn dữ liệu giá tức thời của MariaDB
└── influxdb_data/              # Nơi lưu trữ vĩnh viễn chuỗi lịch sử giá của InfluxDB
```

---

#### 1.3. Danh sách service và cổng
| Tên Service (Trong Compose) | Tên Container | Cổng Nội bộ (Trong Network) | Cổng Public (Ra ngoài Máy chủ) | Mục đích sử dụng |
|---|---|---|---|---|
| nginx | crypto_nginx | 80 | 80 | Người dùng truy cập xem giao diện Bảng điện tử |
| nodered | crypto_nodered | 1880 | 1880 | Admin truy cập lập trình dòng chảy kéo dữ liệu |
| grafana | crypto_grafana | 3000 | 3000 | Cung cấp dashboard biểu đồ nhúng iframe |
| flask-api | crypto_flask_api | 5000 | 5000 | Cung cấp dữ liệu dạng JSON cho Javascript |
| mariadb | crypto_mariadb | 3306 | 3306 | Lưu trữ giá trị tức thời (Có thể mở để Quản trị DB) |
| influxdb | crypto_influxdb | 8086 | 8086 | Lưu dữ liệu lịch sử dạng chuỗi thời gian |

> **Nguyên tắc giao tiếp Mạng nội bộ (Internal Networking)**
Tên dịch vụ thay thế địa chỉ IP: Trong môi trường mạng nội bộ crypto_network, các dịch vụ kết nối với nhau thông qua Tên Service (Service Name) được định nghĩa trong file docker-compose.yml thay vì dùng địa chỉ IP tĩnh (IP nội bộ của Docker sẽ tự thay đổi mỗi khi restart).
>  
> **Ví dụ 1:** Trong mã nguồn Flask API (app.py), chuỗi kết nối đến cơ sở dữ liệu MariaDB sẽ là host="mariadb" chứ không phải localhost hay 127.0.0.1.
> 
> **Ví dụ 2:** Trong luồng cấu hình của Node-RED, khi muốn ghi dữ liệu vào InfluxDB, ô địa chỉ URL cấu hình sẽ điền là http://influxdb:8086.
> 
> **Tính bảo mật hệ thống:** Nhờ cơ chế định tuyến phân tách này, trên thực tế khi triển khai production, người ta có thể đóng hoàn toàn các cổng 3306, 8086, 5000 ra bên ngoài internet. Chỉ cần duy nhất cổng 80 của Nginx mở cho người dùng và cổng 1880 của Node-RED mở cho Admin quản trị, toàn bộ các core bên trong vẫn kết nối ngầm với nhau mượt mà vĩnh viễn.

---

### 2. Cấu hình
```
# 1. Tạo thư mục gốc dự án và các thư mục con
mkdir -p crypto-monitor/nginx/html crypto-monitor/flask_api crypto-monitor/nodered_data crypto-monitor/mariadb_data crypto-monitor/influxdb_data

# 2. Di chuyển thẳng vào thư mục gốc dự án
cd crypto-monitor
```

#### 2.1. Cấu hình file docker-compose.yml
```
cd crypto-monitor
nano docker-compose.yml
```
Nội dung file `docker-compose.yml:`
```
version: '3.8'

services:
  # 1. Node-RED: Cào dữ liệu API sàn Binance & Xử lý logic phát hiện giá bất thường
  nodered:
    image: nodered/node-red:latest
    container_name: crypto_nodered
    ports:
      - "1880:1880"
    volumes:
      - ./nodered_data:/data
    networks:
      - crypto_network
    depends_on:
      - mariadb
      - influxdb
    restart: always

  # 2. MariaDB: Lưu trữ giá trị tức thời (Real-time) phục vụ Backend Flask API
  mariadb:
    image: mariadb:latest
    container_name: crypto_mariadb
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: crypto_db
      MYSQL_USER: thu_admin
      MYSQL_PASSWORD: thu_password
    ports:
      - "3306:3306"
    volumes:
      - ./mariadb_data:/var/lib/mysql
    networks:
      - crypto_network
    restart: always

  # 3. InfluxDB: Lưu trữ dữ liệu lịch sử dạng chuỗi thời gian (Time-series) phục vụ Grafana
  influxdb:
    image: influxdb:1.8
    container_name: crypto_influxdb
    environment:
      - INFLUXDB_DB=crypto_history
      - INFLUXDB_ADMIN_USER=admin
      - INFLUXDB_ADMIN_PASSWORD=admin_password
    ports:
      - "8086:8086"
    volumes:
      - ./influxdb_data:/var/lib/influxdb
    networks:
      - crypto_network
    restart: always

  # 4. Grafana: Trực quan hóa dữ liệu lịch sử trồi sụt của giá coin từ InfluxDB
  grafana:
    image: grafana/grafana:latest
    container_name: crypto_grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ALLOW_EMBEDDING=true  # QUAN TRỌNG: Cho phép nhúng iframe vào trang web
      - GF_AUTH_ANONYMOUS_ENABLED=true      # Cho phép xem biểu đồ ko cần đăng nhập tài khoản
    volumes:
      - grafana_storage:/var/lib/grafana
    networks:
      - crypto_network
    depends_on:
      - influxdb
    restart: always

  # 5. Flask API: Backend kết nối MariaDB, khạc dữ liệu JSON cho Front-end
  flask-api:
    build:
      context: ./flask_api        # Trỏ vào thư mục chứa Dockerfile để tự build image
    container_name: crypto_flask_api
    ports:
      - "5000:5000"
    volumes:
      - ./flask_api:/app          # Gắn volume để sửa code ở ngoài máy vật lý là container cập nhật luôn
    networks:
      - crypto_network
    depends_on:
      - mariadb
    restart: always

  # 6. Nginx Webserver: Phân phối giao diện tĩnh (HTML/JS) & Proxy gọi API nội bộ
  nginx:
    image: nginx:latest
    container_name: crypto_nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/html:/usr/share/nginx/html        # Ánh xạ thư mục chứa file index.html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf # Ánh xạ file cấu hình Nginx riêng biệt
    networks:
      - crypto_network
    depends_on:
      - flask-api
    restart: always

networks:
  crypto_network:
    driver: bridge

volumes:
  nodered_data:
  mariadb_data:
  influxdb_data:
  grafana_storage: # Khai báo named volume ở đây
```

---

#### 2.2. Xây dựng Flask API
##### Bước 1: Khai báo các thư viện cần thiết trong `flask_api/requirements.txt`
```
cd flask_api
nano requirements.txt
```
Tầng này có nhiệm vụ cực kỳ quan trọng: Nhận yêu cầu gọi dữ liệu từ trang web Nginx, tự động kết nối ngầm vào cơ sở dữ liệu MariaDB, bốc giá Crypto mới nhất ra rồi trả về mã JSON.
```
Flask==3.0.2
mysql-connector-python==8.3.0
Flask-Cors==4.0.0
```

---

##### Bước 2: Xây dựng mã nguồn xử lý kết nối và xuất dữ liệu `flask_api/app.py`
```
nano app.py
```
Đoạn code đã tích hợp sẵn cơ chế Tự động khởi tạo cấu trúc bảng dữ liệu (Auto DDL). Nghĩa là khi container MariaDB vừa dựng lên chưa có gì, Flask API sẽ tự động tạo bảng crypto_realtime và chèn 1 dòng dữ liệu mồi giá Bitcoin ban đầu để hệ thống không bị lỗi crash:
```
import os
import time
from flask import Flask, jsonify
from flask_cors import CORS
import mysql.connector
from mysql.connector import Error

app = Flask(__name__)
CORS(app)  # Kích hoạt CORS cho phép Front-end truy cập API chéo nguồn

# CẤU HÌNH THÔNG SỐ KẾT NỐI MARIADB (Lấy chính xác từ file docker-compose.yml)
DB_CONFIG = {
    'host': 'mariadb',        # Sử dụng tên Service nội bộ trong mạng Docker Network
    'database': 'crypto_db',
    'user': 'thu_admin',
    'password': 'thu_password',
    'port': 3306
}

def wait_for_db():
    """Hàm kiểm tra và đợi cho đến khi container MariaDB hoàn toàn sẵn sàng khởi động"""
    while True:
        try:
            connection = mysql.connector.connect(**DB_CONFIG)
            if connection.is_connected():
                cursor = connection.cursor()
                # TỰ ĐỘNG KHỞI TẠO BẢNG NẾU CHƯA TỒN TẠI (Đảm bảo quy trình tự động hóa khép kín)
                cursor.execute("""
                    CREATE TABLE IF NOT EXISTS crypto_realtime (
                        symbol VARCHAR(20) PRIMARY KEY,
                        price DECIMAL(18, 4) NOT NULL,
                        updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
                    )
                """)
                # CHÈN DỮ LIỆU MỒI BAN ĐẦU CHO MÃ BTC
                cursor.execute("""
                    INSERT IGNORE INTO crypto_realtime (symbol, price) 
                    VALUES ('BTCUSDT', 65000.0000)
                """)
                connection.commit()
                cursor.close()
                connection.close()
                print("Successfully connected to MariaDB and initialized database structure!")
                break
        except Error as e:
            print(f"MariaDB is not ready yet ({e}). Waiting 3 seconds...")
            time.sleep(3)

# Khởi động quy trình kiểm tra kết nối an toàn trước khi chạy server API
wait_for_db()

@app.route('/price', methods=['GET'])
def get_realtime_price():
    """Endpoint: /price - Trả về giá trị tức thời của Bitcoin trong MariaDB"""
    try:
        connection = mysql.connector.connect(**DB_CONFIG)
        cursor = connection.cursor(dictionary=True) # Trả về dữ liệu dạng Dictionary để dễ chuyển sang JSON
        
        # Truy vấn lấy dòng dữ liệu giá mới nhất của Bitcoin
        query = "SELECT symbol, price, updated_at FROM crypto_realtime WHERE symbol = 'BTCUSDT'"
        cursor.execute(query)
        result = cursor.fetchone()
        
        cursor.close()
        connection.close()
        
        if result:
            # Chuyển đổi định dạng Decimal và Datetime sang chuỗi để khạc ra chuỗi JSON chuẩn
            return jsonify({
                "status": "success",
                "symbol": result['symbol'],
                "price": str(result['price']),
                "updated_at": str(result['updated_at'])
            }), 200
        else:
            return jsonify({"status": "error", "message": "No data found"}), 404
            
    except Error as e:
        return jsonify({"status": "error", "message": f"Database error: {str(e)}"}), 500

if __name__ == '__main__':
    # Chạy Flask API lắng nghe ở cổng nội bộ 5000 bên trong container
    app.run(host='0.0.0.0', port=5000, debug=False)
```

---

##### Bước 3: Tạo cấu trúc Đóng gói môi trường `flask_api/Dockerfile`
```
nano Dockerfile
```
Để phục vụ cho từ khóa build: ở file Docker Compose, Admin tạo file Dockerfile để chỉ thị quy trình tự động dựng ảnh. 
```
FROM python:3.9-slim

WORKDIR /app

# Sao chép tệp yêu cầu thư viện vào trước để tối ưu hóa bộ nhớ đệm Layer Cache của Docker
COPY requirements.txt .

# Cài đặt các thư viện Python mà không lưu trữ file rác bộ đệm
RUN pip install --no-cache-dir -r requirements.txt

# Sao chép toàn bộ mã nguồn backend vào trong Container làm việc
COPY . .

# Thông báo cổng mạng container sẽ lắng nghe
EXPOSE 5000

# Lệnh khởi chạy ứng dụng chính khi container bắt đầu chạy
CMD ["python", "app.py"]
```

---

#### 2.3. Cấu hình Luật định tuyến Proxy `nginx/default.conf`
File: `nginx/default.conf`
```
cd ../nginx
nano default.conf
```
Cấu hình này giúp cô lập Flask API ở cổng 5000 ngầm bên trong, mọi yêu cầu lấy dữ liệu từ trình duyệt chỉ cần gọi qua cổng 80 của Nginx:
```
server {
    listen 80;
    server_name localhost;

    # 1. Định tuyến phân phối giao diện Front-end tĩnh (HTML, CSS, JS)
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
    }

    # 2. Cấu hình Reverse Proxy chuyển tiếp yêu cầu gọi API sang Flask container
    location /api/ {
        proxy_pass http://flask-api:5000/; # Gọi trực tiếp qua tên Service nội bộ trong mạng Docker Network
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
---

#### 2.4. Xây dựng Giao diện Bảng điện tử hiển thị `nginx/html/index.html`
Viết file index.html làm frontend cho hệ thống. i chuyển vào thư mục chứa tài nguyên web tĩnh để tạo file giao diện:
```
cd html
nano index.html
```
Mã nguồn HTML phối hợp Javascript xử lý cơ chế AJAX tự động cập nhật (Auto-refresh) và nhúng iframe Grafana hiển thị biểu đồ lịch sử.
```
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bảng Điện Tử Giám Sát Thị Trường Crypto Real-time</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #121214;
            color: #ffffff;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .container {
            max-width: 1200px;
            width: 100%;
            text-align: center;
        }
        h1 { color: #f0b90b; margin-bottom: 30px; }
        .ticker-board {
            background-color: #1e1e22;
            border-radius: 10px;
            padding: 20px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.5);
            display: inline-block;
            margin-bottom: 40px;
            border: 1px solid #333;
        }
        .crypto-name { font-size: 24px; font-weight: bold; color: #aaa; }
        .crypto-price {
            font-size: 48px;
            font-weight: bold;
            color: #0ecb81;
            margin: 10px 0;
            font-family: 'Courier New', Courier, monospace;
        }
        .update-time { font-size: 14px; color: #666; }
        .chart-section { width: 100%; margin-top: 20px; }
        h2 { color: #aaa; text-align: left; border-bottom: 2px solid #2d2d30; padding-bottom: 10px; }
        iframe {
            width: 100%;
            height: 500px;
            border: 1px solid #2d2d30;
            border-radius: 8px;
            background-color: #1f2022;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>📈 HỆ THỐNG GIÁM SÁT THỊ TRƯỜNG CRYPTO REAL-TIME</h1>

    <div class="ticker-board">
        <div class="crypto-name">Bitcoin (BTC / USDT)</div>
        <div id="btc-price" class="crypto-price">Đang tải số liệu...</div>
        <div class="update-time">Tự động đồng bộ số liệu từ MariaDB mỗi 2 giây</div>
    </div>

    <div class="chart-section">
        <h2>📊 Biến Động Lịch Sử Giá (Dữ liệu từ InfluxDB)</h2>
        <iframe src="http://localhost:3000/d-solo/crypto_dashboard/crypto-monitor-dashboard?orgId=1&panelId=1&refresh=5s" frameborder="0"></iframe>
    </div>
</div>

<script>
    // Hàm thực hiện cơ chế AJAX (Fetch API) gọi lên Flask Backend lấy số liệu mới nhất từ MariaDB
    function fetchRealtimePrice() {
        fetch('/api/price') // Gọi gián tiếp qua phân luồng Proxy của Nginx (/api/ -> flask-api:5000/)
            .then(response => response.json())
            .then(data => {
                if (data && data.price) {
                    const priceElement = document.getElementById('btc-price');
                    // Định dạng số hiển thị thành dạng tiền tệ USD ($) cho trực quan
                    const formattedPrice = parseFloat(data.price).toLocaleString('en-US', { style: 'currency', currency: 'USD' });
                    priceElement.innerText = formattedPrice;
                }
            })
            .catch(error => {
                console.error('Lỗi kết nối API hệ thống:', error);
                document.getElementById('btc-price').innerText = "Mất kết nối DB...";
            });
    }

    // Thiết lập bộ hẹn giờ tự động quét (Auto hiển thị số mới khi dữ liệu MariaDB thay đổi)
    setInterval(fetchRealtimePrice, 2000); // Chu kỳ quét: 2 giây/lần
    fetchRealtimePrice(); // Kích hoạt mồi lần đầu tiên ngay khi tải trang
</script>

</body>
</html>
```

---

#### 2.5. Khởi động hệ thống
Sau khi đã hoàn tất toàn bộ mã nguồn và file cấu hình của các Microservices (`docker-compose.yml`, `default.conf`, `index.html`, `app.py`, `Dockerfile`), tiến hành kích hoạt toàn bộ hệ thống trên máy vật lý Ubuntu.
```
docker compose up -d --build
```

<img width="1917" height="1079" alt="image" src="https://github.com/user-attachments/assets/0ab9101e-2ea4-4590-afe4-34a6e79bb741" />

Kiểm tra trạng thái các container 
```
docker ps
```
<img width="1917" height="798" alt="image" src="https://github.com/user-attachments/assets/bcb95ba5-a6ce-41d4-8dd9-c283bb333159" />

---

#### 2.6. Hướng dẫn Cấu hình Node-RED để Tự động hóa Luồng dữ liệu
Trong giai đoạn này, **Node-RED** đóng vai trò đầu não thực hiện 4 nhiệm vụ liên tục một cách khép kín:
- Cào dữ liệu giá Bitcoin thực tế từ API công khai của sàn Binance (Định kỳ mỗi 5 giây).
- Trích xuất và cập nhật trạng thái giá mới nhất vào **MariaDB** (Ghi đè giá trị tức thời).
- Đẩy dữ liệu đồng bộ vào **InfluxDB** để tích lũy chuỗi thời gian lịch sử phục vụ Grafana vẽ biểu đồ.
- Phân tích dữ liệu để phát hiện giá trị bất thường và kích hoạt **Telegram Bot** gửi tin nhắn cảnh báo trực tiếp vào Group hệ thống.

##### Bước 1: Chuẩn bị Thư viện (Palette Nodes) trong Node-RED
Mặc định, Node-RED bản thô chỉ có các node xử lý logic cơ bản. Để kết nối được với các hệ quản trị cơ sở dữ liệu (MariaDB, InfluxDB), Admin cần tiến hành cài đặt thêm các gói mở rộng (Nodes) thông qua trình quản lý thư viện trực quan.

**Quy trình thực hiện cài đặt:**
```
Truy cập Node-RED qua địa chỉ `http://<IP_máy_chủ_Ubuntu>:1882`_**http://172.27.2.42:1880**
```

Tại góc trên cùng bên phải giao diện, bấm vào biểu tượng Menu (3 dấu gạch ngang) ➡️ Chọn mục Manage palette. Trong bảng điều khiển xuất hiện, bấm chuyển sang tab Install. Lần lượt tìm kiếm chính xác tên 3 thư viện lõi dưới đây và bấm nút Install để hệ thống tự động tải và tích hợp vào thanh công cụ:

```
node-red-node-mysql (Kết nối MariaDB)
node-red-contrib-influxdb (Kết nối InfluxDB)
node-red-contrib-telegrambot (Kết nối Telegram)
```

<img width="1917" height="1079" alt="image" src="https://github.com/user-attachments/assets/9aef6bf3-ff49-400a-a6e1-f7af4df43a8a" />

##### Bước 2: Hướng dẫn Khởi tạo Telegram Bot và Thiết lập Nhóm cảnh báo nhiều người

Để đáp ứng trọn vẹn yêu cầu cấu trúc của đề tài: Kết hợp bot Telegram, tạo nhóm có 3 người (bao gồm tài khoản ID `1875746636`), gửi tin nhắn cảnh báo dị thường kèm giá trị tường minh, tiến hành thực hiện các bước chuẩn bị hạ tầng mạng xã hội theo quy trình sau:

Khởi tạo Bot Telegram qua @BotFather, mở ứng dụng Telegram trên điện thoại hoặc máy tính cá nhân. Tại ô tìm kiếm, gõ chính xác từ khóa `BotFather` (Có dấu tích xanh chính chủ của Telegram) và bấm `Start`. Gõ câu lệnh: **`/newbot`**

- Hệ thống BotFather sẽ phản hồi yêu cầu nhập tên cho Bot (Name). Nhập vào: Crypto Monitor Bot.
- Tiếp theo, hệ thống yêu cầu nhập tên định danh duy nhất (Username) kết thúc bằng chữ bot. Nhập vào: thu_crypto_monitor_bot.
- Sau khi tạo thành công, BotFather sẽ khạc ra một chuỗi mã HTTP API Token bảo mật cấp cao. Tiến hành copy chuỗi Token này ra một file nháp để cấu hình vào Node-RED ở bước sau.

<img width="1916" height="1079" alt="Ảnh chụp màn hình 2026-06-09 115607" src="https://github.com/user-attachments/assets/0bfe1bbb-d42e-4521-8874-8dfae26c5018" />

Thiết lập Nhóm Chat Bot Cảnh báo (Hệ thống nhiều thành viên):

- Trên giao diện Telegram, bấm chọn biểu tượng tạo tin nhắn mới ➡️ Chọn New Group.
- Tiến hành thêm các thành viên bắt buộc vào nhóm bao gồm:
  - Tài khoản cá nhân của Thứ (Admin).
  - Tài khoản một người bạn đồng hành.
  - Thêm tài khoản ID hệ thống: Tìm kiếm và add chính xác tài khoản có ID số 1875746636 vào nhóm theo yêu cầu cứng của barem bài tập.
- Tiến hành tìm kiếm tên Username con Bot vừa tạo (thu_crypto_monitor_bot) và add con Bot này vào Group.
- Đặt tên cho nhóm chat: CẢNH BÁO THỊ TRƯỜNG CRYPTO REAL-TIME.
> **QUAN TRỌNG:** Truy cập vào phần quản lý thành viên nhóm (Group Info) ➡️ Chọn con Bot của bạn ➡️ Bấm Promote to Admin để cấp quyền quản trị viên, cho phép Bot có quyền tự động đẩy tin nhắn vào nhóm mà không bị bộ lọc chặn tin rác.

<img width="1600" height="672" alt="image" src="https://github.com/user-attachments/assets/6538034a-0e7d-4193-8f6e-f63055ba777c" />

---

##### Bước 3: Thiết kế chi tiết luồng dữ liệu (Flow Design) trên Node-Red
Kéo các node và điền các thông tin:

***Node Inject (Hẹn giờ):***

  - Kéo từ cột bên trái ra bàn vẽ.
  - Ấn đúp vào, chọn Repeat là interval, đặt là 5 seconds.
  - Đặt tên (Name) là: "Kích hoạt mỗi 5 giây".

<img width="1916" height="1079" alt="image" src="https://github.com/user-attachments/assets/e85d533d-be3c-44e9-b06b-4954cbb106a9" />

***Node http request (Cào dữ liệu):***

  - Nối dây từ node Inject sang node này.
  - Ấn đúp vào, mục Method chọn GET.
  - Mục URL dán link: https://api.binance.com/api/v3/ticker/price?symbol=BTCUSDT
  - Mục Return chọn: a parsed JSON object.
  - Đặt tên: "Binance API".

<img width="1915" height="1079" alt="image" src="https://github.com/user-attachments/assets/a9a03b00-38d8-4518-8640-d16eac18d63f" />

***Node Function (Tiền xử lý):***

  - Nối dây từ node http request sang.
  - Dán code này vào để tách lấy con số giá:
```
var price = parseFloat(msg.payload.price);
msg.payload = price; // Gán giá vào payload để các node sau sử dụng
return msg;
```

<img width="1915" height="1079" alt="image" src="https://github.com/user-attachments/assets/436be83e-f35d-483a-8ae7-bc56241af8d1" />

Nhiệm vụ: Ghi đè giá mới nhất vào MariaDB và lưu lịch sử vào InfluxDB.

***Ghi vào MariaDB (Ngả rẽ 1):***

  - Kéo một node Function mới nối từ node Tiền xử lý ra. Dán code SQL:

<img width="1916" height="1078" alt="image" src="https://github.com/user-attachments/assets/33b415e1-1ba8-4f3e-a033-b8ff301f71b0" />






# <p align="center">***THE END***</p>

🔒 *Bản quyền nội dung **Copyright © 2026** thuộc về Nguyễn Văn Thứ. Bảo lưu mọi quyền (All Rights Reserved).*
