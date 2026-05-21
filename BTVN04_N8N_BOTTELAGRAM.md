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

### GIAI ĐOẠN 1: THIẾT LẬP KHỞI TẠO HẠ TẦNG (INFRASTRUCTURE SETUP).

### ***A. Kiểm tra phiên bản docker & docker compose***

- Mục tiêu của giai đoạn này là xây dựng một "tổ hợp" Container gồm: MariaDB, phpMyAdmin, WordPress và n8n chạy đồng bộ, chung múi giờ Việt Nam và phân quyền bảo mật tuyệt đối.

- Kiểm tra version phiên bản docker & docker-compose lệnh: `docker --version` & `docker compose version`

<img width="670" height="186" alt="image" src="https://github.com/user-attachments/assets/924994f3-7903-4f6c-8f98-2992d3992304" />

### ***B. Khởi tạo cấu trúc thư mục dự án***

```
mkdir ~/ai_automation_v2
cd ~/ai_automation_v2
```

<img width="697" height="81" alt="image" src="https://github.com/user-attachments/assets/73b1b67a-c485-4556-a109-fbf22d3a1c92" />

### ***C. Khở tạo Cloudfare Tunnel mới cho dự án***

- Mặc dù yêu cầu dùng AI chuyển cấu hình sang Docker, nhưng để có mạng kết nối, trước tiên em đứng từ Ubuntu xin Cloudflare cấp một cái Tunnel ID (UUID) và File khóa xác thực (.json) mới hoàn toàn.

- Gõ lệnh tạo tunnel mới (tên tunnel cũng đặt khác đi để tránh trùng): `cloudflared tunnel create tunnel_automation_v2`. Với ID `b22bdc3d-4fc2-4ce6-bb1e-f63d60b2f7df`

<img width="1479" height="200" alt="image" src="https://github.com/user-attachments/assets/fbf88783-f6ed-4b51-9416-b3440d82a054" />

- Di chuyển file khóa vào vùng cô lập. Ngay sau khi tạo xong, file khóa xác thực .json sẽ nằm trong thư mục ẩn của hệ thống. Cần dùng lệnh này để di chuyển ngay nó về nằm gọn bên trong thư mục dự án mới, đồng
thời đổi tên thành tunnel_secret.json cho dễ quản lý: `cp ~/.cloudflared/b22bdc3d-4fc2-4ce6-bb1e-f63d60b2f7df.json ~/ai_automation_v2/tunnel_secret.json`

<img width="1481" height="91" alt="image" src="https://github.com/user-attachments/assets/a5b57a5d-3565-4ca7-a05e-00cb39125c25" />

### ***D. Cấu hình định tuyến file Config.yml***

- Sử dụng lệnh `nano config.yml` để cấu hình đinh tuyến file ***config.yml:*** 

- Một giao diện soạn thảo hiện ra, nội dung file cấu hình:

```
tunnel: b22bdc3d-4fc2-4ce6-bb1e-f63d60b2f7df
credentials-file: /etc/cloudflared/tunnel_secret.json

ingress:
  # 1. Định tuyến cho WordPress (Tên miền truy cập WP)
  - hostname: mywordpress.nguyenthu.id.vn
    service: http://127.0.0.1:8082

  # 2. Định tuyến cho PhpMyAdmin (Để xem CSDL theo yêu cầu của thầy)
  - hostname: pma.nguyenthu.id.vn
    service: http://127.0.0.1:8080

  # 3. Định tuyến cho n8n (Để cấu hình luồng tự động hóa)
  - hostname: n8n.nguyenthu.id.vn
    service: http://127.0.0.1:5678

  - service: http_status:404
```

<img width="1478" height="752" alt="image" src="https://github.com/user-attachments/assets/b192153e-d6c1-45f7-abcf-bc3619f71059" />

***➡️ Nhấn Ctrl + O, Enter để lưu và Ctrl + X để thoát***

### ***E. Tạo file cấu hình docker-compose.yml (5 Services)***

- Đây là trái tim của Giai đoạn 1. File này định nghĩa đúng và đủ 5 dịch vụ: `mariadb, phpmyadmin, wordpress, n8n, và cloudflared.`

- Sử dụng lệnh `nano docker-compose.yml` để tạo file ***docker-compose.yml*** với nội dung file như sau:

```
version: '3.8'

services:
  # 1. Dịch vụ Cơ sở dữ liệu MariaDB
  mariadb:
    image: mariadb:latest
    container_name: auto_mariadb
    restart: always
    environment:
      TZ: "Asia/Ho_Chi_Minh"
      MARIADB_ROOT_PASSWORD: root_password_999
      MARIADB_DATABASE: wp_automation_db      # Tên database mới tinh
      MARIADB_USER: auto_user
      MARIADB_PASSWORD: user_password_999
    volumes:
      - mariadb_v2_data:/var/lib/mysql

  # 2. Dịch vụ quản trị PhpMyAdmin
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: auto_phpmyadmin
    restart: always
    ports:
      - "8080:80"
    environment:
      PMA_HOST: mariadb
      PMA_ARBITRARY: 1
    depends_on:
      - mariadb

  # 3. Dịch vụ WordPress chính
  wordpress:
    image: wordpress:latest
    container_name: auto_wordpress
    restart: always
    ports:
      - "8082:80"
    environment:
      WORDPRESS_DB_HOST: mariadb
      WORDPRESS_DB_NAME: wp_automation_db
      WORDPRESS_DB_USER: auto_user
      WORDPRESS_DB_PASSWORD: user_password_999
    volumes:
      - wordpress_v2_data:/var/www/html
    depends_on:
      - mariadb

  # 4. Dịch vụ Tự động hóa n8n
  n8n:
    image: n8nio/n8n:latest
    container_name: auto_n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - TZ=Asia/Ho_Chi_Minh
      - WEBHOOK_URL=https://n8n.nguyenthu.id.vn/  # Điền chính xác Subdomain n8n của Thứ
    volumes:
      - n8n_v2_data:/home/node/.n8n

  # 5. Dịch vụ Cloudflare Tunnel (Chạy cục bộ bằng file)
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: auto_tunnel
    restart: always
    network_mode: "host"
    volumes:
      - ./config.yml:/etc/cloudflared/config.yml:ro
      - ./tunnel_secret.json:/etc/cloudflared/tunnel_secret.json:ro
    command: tunnel --config /etc/cloudflared/config.yml run

volumes:
  mariadb_v2_data:
  wordpress_v2_data:
  n8n_v2_data:
```

<img width="1482" height="757" alt="image" src="https://github.com/user-attachments/assets/ee1b51a9-33b8-472b-a10c-a4b32a938e6c" />

***➡️ Nhấn Ctrl + O, Enter để lưu và Ctrl + X để thoát***

- Sau khi tạo thực hiện kiểm tra thư mục đã có những gì bằng lệnh `ls -la` với 3 file (tunnel_secret.json, config.yml, docker-compose.yml) nằm chung trong thư mục ~/ai_automation_v2

<img width="933" height="198" alt="image" src="https://github.com/user-attachments/assets/0d00ddc3-4c1f-4117-b60c-40b3cf00faaa" />

- Lệnh: `chmod 644 tunnel_secret.json config.yml.` Cấp quyền Đọc và Ghi (6) cho người sở hữu, quyền Chỉ Đọc (4) cho nhóm và các đối tượng khác. Đây là mức bảo mật tiêu chuẩn, cho phép hệ thống Docker có quyềnvào đọc nội dung file cấu hình và file khóa để thông mạng, nhưng không được phép tự ý chỉnh sửa file.

- Lệnh: `sudo chmod -R 777 ~/ai_automation_v2.` Sử dụng quyền Quản trị tối cao (sudo) và tác động đệ quy (-R) để mở toang toàn bộ quyền Đọc - Ghi - Chạy (777) cho thư mục dự án và tất cả file con bên trong. Xử lý triệt để lỗi xung đột bản quyền giữa các User hệ thống và Container Docker, ép dịch vụ Cloudflare Tunnel đọc được file xác thực ngay lập tức để sửa lỗi bị sập (Restarting).

<img width="1072" height="72" alt="image" src="https://github.com/user-attachments/assets/78a55454-9769-45b2-93a2-697485456a30" />

- Kích hoạt lệnh này để khởi động kéo các Service về chạy ngầm: `docker-compose up -d`

<img width="1917" height="1079" alt="image" src="https://github.com/user-attachments/assets/7139035b-564a-47db-be96-03e1eeee3ebb" />

- Hệ thống sẽ bắt đầu kéo (Pull) các Image từ Docker Hub về và khởi chạy. Khi chạy xong, Thứ gõ lệnh này để kiểm tra trạng thái các Container: `docker ps`

<img width="1918" height="591" alt="image" src="https://github.com/user-attachments/assets/1516f1bd-4695-4fab-9e73-9def90d80ffe" />

### GIAI ĐOẠN 2: ĐỊNH TUYẾN INTERNET & CẤU HÌNH BAN ĐẦU.

- Mục tiêu của giai đoạn này là đưa cả 3 dịch vụ lên sóng thông qua Cloudflare Tunnel và thực hiện các yêu cầu bắt buộc của thầy về kiểm tra Cơ sở dữ liệu (CSDL).

### ***A. Thêm bản ghi cho các dịch vụ container lên trên website cloudfare***

- Cách xử lý khi không truy cập được vào các cổng:

  - Đăng nhập vào https://dash.cloudflare.com.
  - Chọn tên miền của bạn: nguyenthu.id.vn.
  - Nhìn menu bên trái, chọn DNS -> Records (Bản ghi).
  - Bấm nút Add record để thêm lần lượt 3 bản ghi CNAME sau đây để cấu hình định tuyến cho cả 3 dịch vụ: `Bản ghi 1 (Cho WordPress),` `Bản ghi 2 (Cho PhpMyAdmin),` `Bản ghi 3 (Cho n8n)`

- Bản ghi 1 (Cho WordPress):
  
<img width="1915" height="1021" alt="image" src="https://github.com/user-attachments/assets/b0250d4c-914e-4f46-8c4c-ab6c6185a319" />

- Bản ghi 2 (Cho PhpMyAdmin):

<img width="1916" height="1019" alt="image" src="https://github.com/user-attachments/assets/53f2ddf5-d30b-4f98-b21b-efc00c844013" />

- Bản ghi 3 (Cho n8n):

<img width="1918" height="1021" alt="image" src="https://github.com/user-attachments/assets/03beaf7a-b9b1-40f5-9319-1e5a536aae0f" />

- Danh sách bản ghi đã thêm thành công:

<img width="1916" height="1019" alt="image" src="https://github.com/user-attachments/assets/0171371b-a235-475d-a821-e170a7510117" />

### ***B. Truy cập PhpMyAdmin (Quan sát CSDL trống)***

- Truy cập vào địa chỉ: https://pma.nguyenthu.id.vn

| Trường thông tin | Thông tin |
|---|---|
| Tài khoản | root |
| Mật khẩu | root_password_999 (theo file docker-compose mình vừa cấu hình) |

<img width="1915" height="1021" alt="image" src="https://github.com/user-attachments/assets/b5f36687-6056-40a5-81ae-12e5e0334649" />

- Nhìn sang danh sách cơ sở dữ liệu bên trái, bấm vào đúng tên database `wp_automation_db.` Sẽ thấy dòng chữ "No tables found in database" (Không tìm thấy bảng nào).

<img width="1917" height="1022" alt="image" src="https://github.com/user-attachments/assets/8325afa0-8ad7-4e16-9250-114cc3c575bf" />

### ***C. Truy cập Wordpress lần đầu***

- Mở một tab mới và truy cập: https://mywordpress.nguyenthu.id.vn. Màn hình chào mừng của WordPress sẽ hiện ra. họn ngôn ngữ là Tiếng Việt (hoặc Tiếng Anh) rồi bấm Tiếp tục.

<img width="1916" height="1021" alt="image" src="https://github.com/user-attachments/assets/3d9171ab-c8bd-4095-a32b-d85ef2c5996b" />

- Thiết lập thông tin quản trị Website:
  
| Trường thông tin | Thông tin |
|---|---|
| Tên trang web | Thu’s Automation Project |
| Tên người dùng (Username) | nguyenvanthu |
| Mật khẩu (Password) | thu123 |
| Email | mn9103541@gmail.com |

<img width="1916" height="1020" alt="image" src="https://github.com/user-attachments/assets/fcf7813d-8b7f-4dcb-a63b-d9358bf73705" />

- Bấm Cài đặt WordPress và đợi 5 giây để nó tự động thiết lập toàn bộ hệ thống ngầm.

<img width="1915" height="1020" alt="image" src="https://github.com/user-attachments/assets/77d025c3-d366-477a-b2ae-84541a9960ea" />

- Tiến hành đăng nhập lại để xác minh

<img width="1918" height="1017" alt="image" src="https://github.com/user-attachments/assets/4850bbd1-0445-4600-8e10-66a9db90d426" />

- Giao diện chào mừng thành công của wordpress

<img width="1915" height="1023" alt="image" src="https://github.com/user-attachments/assets/87cf75cc-8f87-42ed-a801-686d99bd2314" />

### ***D. Kiểm tra lại CSDL trên PhpMyAdmin***

- uay trở lại tab PhpMyAdmin đang mở https://pma.nguyenthu.id.vn/index.php?route=/
- Bấm nút F5 (Tải lại trang) hoặc bấm lại vào tên database wp_automation_db.
- Lúc này, Thứ sẽ thấy một điều kỳ diệu: Hệ thống đã tự động đẻ ra chính xác 12 bảng dữ liệu hệ thống (bắt đầu bằng chữ wp_ như wp_posts, wp_users, wp_options...).

<img width="1918" height="1020" alt="image" src="https://github.com/user-attachments/assets/59207cce-253d-47d6-b296-51571b311e67" />

# <p align="center">***THE END***</p>
