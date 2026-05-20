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

### ***Phân quyền bảo mật PhpMyAdmin***

- Dùng phpMyAdmin tạo một CSDL trống và một User chỉ có quyền duy nhất với CSDL đó.

- Truy cập phpMyAdmin: Mở trình duyệt trên Windows gõ: http://localhost:8081 hoặc http://172.27.2.42:8081

| Đăng nhập | Tài khoản: root | Mật khẩu: thu123 |
|---|---|---|

- Tạo CSDL mới 

  - Nhìn sang cột bên trái, bấm vào chữ New (Mới)

  - Ô Tên cơ sở dữ liệu: Gõ đúng tên wp_ai_db

  - Bấm nút Create (Tạo). (Bây giờ ta có CSDL trống)

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/0b6b6759-747a-4d14-b76d-e7034c5f5b09" />

- Tạo User phân quyền biệt lập:

  - Nhìn lên thanh menu ngang phía trên, chọn mục User accounts (Tài khoản người dùng).

  - Chọn Add user account (Thêm tài khoản người dùng).

- Login Information:

| Trường thông tin | Thông  |
|---|---|
| User name (Tên người dùng) | Gõ wp_automation |
| Host name (Tên máy chủ) | Chọn Any host (hoặc gõ dấu %) |
| Password (Mật khẩu) | Gõ thu123 |
| Re-type (Gõ lại) | Gõ thu123 |

- Database for user account bỏ qua các mục phân quyền tự động (Rất quan trọng):
  
  - Mục Database for user account: ĐỂ TRỐNG, không tích vào bất kỳ ô nào trong 2 ô đó.

  - Mục Global privileges (Quyền toàn cục): Cũng ĐỂ TRỐNG BÀN PHÍM, không bấm "Check all".

➡️ Kéo xuống dưới cùng bên phải, nhấn nút Go để tạo User. Lúc này màn hình sẽ báo thành công và bạn đã có một User "sạch".

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/3c8e78fb-f0ca-44b6-a332-8496e6f2a58c" />

- Đã tạo thành công tài khoản người dùng
  
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/e4bf3aff-3c7c-4098-a375-433044a15746" />

### ***Gán quyền chính xác vào riêng DB wp_ai_db***

- Ngay sau khi nhấn Go ở bước trên, nhìn lên menu ngang phía trên cùng của phpMyAdmin:

- Bấm lại vào tab User accounts (Tài khoản người dùng).

- Tìm đến dòng của user wp_automation vừa tạo, nhìn sang bên phải bấm vào Edit privileges (Sửa đổi quyền).

- Ở giao diện mới, nhìn lên menu ngang phụ (nằm ngay dưới hàng chữ Database/SQL/Status...), bấm vào mục Database.

- Tại ô chọn, tìm và click chọn đúng tên database wp_ai_db, sau đó bấm Go.

- Giao diện cấp quyền riêng cho DB này sẽ hiện ra: Thứ tích vào ô Check All (nằm ngay cạnh chữ Table-specific privileges hoặc chọn tất cả các quyền SELECT, INSERT, UPDATE, DELETE...).

- Kéo xuống dưới cùng bên phải, nhấn nút Go để hoàn tất.

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/ec9fc306-d074-4d8c-8284-28449778981e" />

- ***Kiểm tra lại:*** Hãy thử Log out tài khoản Root ra, đăng nhập lại bằng user `wp_automation / thu123.` Bạn sẽ thấy user này chỉ nhìn thấy duy nhất db wp_ai_db, hoàn toàn không thấy các dữ liệu hệ thống khác. Đúng chuẩn bảo mật!

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/949e0060-f2cc-4382-ae71-04c037e45496" />

- Kết quả đăng nhập lại bằng:

  - User: wp_automation

  - Password: thu123

- Cột danh sách bên trái, thấy đúng `wp_ai_db và information_schema,` hoàn toàn không nhìn thấy các DB hệ thống khác (mysql, sys, performance_schema) đã hoàn thành bài thực hành phân quyền an toàn thông tin.

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/bfd4f2b7-fb44-495e-b4b5-25a641937467" />

### ***Khởi tạo lần đầu cho Wordpress

- Mở trình duyệt gõ: http://localhost:8082 hoặc http://172.27.2.42:8082

  - Chọn ngôn ngữ (Tiếng Việt hoặc Tiếng Anh) rồi bấm Tiếp tục.

  - Vì các tham số môi trường chúng ta đã truyền thẳng vào file `docker-compose.yml` trùng khớp với User/Database, WordPress sẽ kết nối thành công ngay lập tức mà không đòi hỏi nhập lại thông tin DB!

  - Nhấn nút Tiếp tục (Continue).

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/914948fe-64d0-4174-bbf6-2009a12e3d48" />

- Điền thông tin quản trị tối cao (Site Information). Vì cấu hình cơ sở dữ liệu đã được nhận diện tự động từ file docker-compose.yml, WordPress sẽ nhảy thẳng đến trang cài đặt tài khoản. 

| Trường thông tin | Thông tin |
|---|---|
| Tên trang web (Site Title) | Điền tên Blog hoặc dự án của bạn (Ví dụ: Thứ Nguyễn Blog hoặc Nguyễn Văn Thứ Portfolio) | 
| Tên người dùng (Username) | Đây là tài khoản dùng để đăng nhập vào trang quản trị (/wp-admin) nguyenvanthu |
| Mật khẩu (Password) | thu123 |
| Email của bạn (Your Email) | mn9103541@gmail.com |

- Mẹo: Nếu WordPress báo mật khẩu yếu (Weak), Thứ hãy tích vào ô "Chấp nhận sử dụng mật khẩu yếu" (Confirm use of weak password) ngay phía dưới để hệ thống cho phép đi tiếp.

- Hiển thị với các công cụ tìm kiếm (Search Engine Visibility): Ô này Thứ ĐỂ TRỐNG (không tích chọn), vì sau này khi bạn public qua Cloudflare Tunnel, bạn sẽ muốn Google có thể tìm thấy và lập chỉ mục website của bạn.

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/513a656c-53fa-4d8f-a83d-4f589f756051" />

- Hoàn tất cài đặt:

  - Sau khi điền đầy đủ các thông tin trên, Thứ nhấn vào nút Cài đặt WordPress (Install WordPress) ở dưới cùng.

  - Hệ thống sẽ xử lý trong khoảng 3 - 5 giây. Khi màn hình hiện chữ "Thành công!" (Success!), nghĩa là WordPress đã khởi tạo xong toàn bộ cấu trúc bảng bên trong database wp_ai_db.- 

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/1ebe473c-301c-4bed-892a-7e1113ccf687" />

- Đăng nhập vào Dashboard chính:
  
  - Nhấn vào nút Đăng nhập (Log In).

  - Điền Username và Password vừa tạo (nguyenvanthu / thu123).

  - Nhấn Đăng nhập.

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/bb58e756-4677-411d-b121-9db96297f79c" />

- Giao diện quản trị màu đen - trắng quen thuộc sẽ hiện ra.
  
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/abdc95ef-d1fe-4c4c-88e9-0b3398cc770c" />

### GIAI ĐOẠN 2: CẤU HÌNH KẾT NỐI & PUBLIC (CONNECTIVITY).

### ***Cấu hình Cloudfare tunnel (Cho cả WP và n8n)***

- Tạo CLoudfare Tunnel mới (Đặt tên là ai-automation-tunnel)

- Chạy lệnh tạo Tunnel mới: `cloudflared tunnel create ai-automation-tunnel`

- Hệ thống sẽ sinh ra một ID Tunnel mới và một file .json mới tương ứng copy lại cái ID mới này nhé. ***ID: 883adc1b-91cd-485d-ae27-e704225286d5***

<img width="1915" height="168" alt="image" src="https://github.com/user-attachments/assets/9e8be29a-9eec-4283-b8e8-1b61664383c3" />

- Tạo file cấu hình riêng nằm trong thư mục dự án. Để không bị lẫn lộn với file config mặc định của hệ thống, chúng ta sẽ tạo file config.yml ngay bên trong thư mục dự án hiện tại (~/ai_content_project).

- Di chuyển vào thư mục dự án: `cd ~/ai_content_project`

- Tạo mở file config riêng bằng lệnh: `nano config.yml`

- Dán nội dung cấu hình biệt lập này vào (Nhớ thay ID và tên file .json mới của vào 2 dòng đầu nhé):

```
tunnel: 883adc1b-91cd-485d-ae27-e704225286d5
credentials-file: /home/nguyenvanthu/ai_content_project/883adc1b-91cd-485d-ae27-e704225286d5.json

ingress:
  - hostname: mywordpress.nguyenthu.id.vn
    service: http://127.0.0.1:8082

  - hostname: n8n.nguyenthu.id.vn
    service: http://127.0.0.1:5678

  - service: http_status:404
```

<img width="1299" height="335" alt="image" src="https://github.com/user-attachments/assets/4e91b2ad-465b-4639-b028-38ca0cdbd4e4" />

➡️ ***(Nhấn Ctrl + O, Enter để lưu và Ctrl + X để thoát).***

- Định tuyến (outerDNS) cho 2 Subdomain mới

```
cloudflared tunnel route dns 883adc1b-91cd-485d-ae27-e704225286d5 mywordpress.nguyenthu.id.vn
cloudflared tunnel route dns 883adc1b-91cd-485d-ae27-e704225286d5 n8n.nguyenthu.id.vn
```

<img width="1744" height="163" alt="image" src="https://github.com/user-attachments/assets/1ff3baeb-c90c-4273-a466-a76cc21a9886" />

- Giải pháp xử lý chuyên nghiệp (Cô lập dự án):

  - Trích xuất file khóa xác thực của dự án mới ra khỏi thư mục dùng chung và đưa về nằm gọn trong thư mục dự án hiện tại: `cp ~/.cloudflared/883adc1b-91cd-485d-ae27-e704225286d5.json ~/ai_content_project/`
 
- Chay Tunnel công khai hệ thống `cloudflared tunnel --config ~/ai_content_project/config.yml run` bằng file Config

  - Vì mình không dùng file config mặc định ở thư mục ẩn ~/.cloudflared/config.yml nữa, nên khi chạy Tunnel, phải truyền thêm tham số --config để chỉ định file cấu hình trong thư mục dự án:
 
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/0ab94eff-254c-4c10-82a4-595fbc2e4148" />

### ***Các bước thiết lập luồng***



# <p align="center">***THE END***</p>
