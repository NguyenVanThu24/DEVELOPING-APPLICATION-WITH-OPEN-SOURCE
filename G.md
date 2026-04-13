# G (2) - CÂU HỎI VỀ BÀI LÀM
## 1. Tại sao phải dùng Nginx làm Reverse Proxy mà không trỏ thẳng Tunnel vào Node-RED?
### Trả lời:
- Nếu trỏ Tunnel thẳng vào Node-RED, toàn bộ lưu lượng sẽ vào Node-RED, không thể phân chia request tới nhiều service khác nhau.

- Dùng Nginx làm Reverse Proxy giúp mang đến:

    + Tính linh hoạt: Nginx đóng vai trò là "lễ tân". Nó có thể điều phối nhiều ứng dụng khác nhau (Node-RED, API, Website tĩnh) trên cùng một cổng.

    + Bảo mật & Hiệu suất: Nginx xử lý các tệp tĩnh (CSS, JS, Hình ảnh) nhanh hơn Node-RED, giúp giảm tải cho ứng dụng chính.

    + Quản lý tập trung: Bạn dễ dàng cấu hình SSL, chặn IP xấu, hoặc giới hạn băng thông tại một nơi duy nhất thay vì phải cấu hình riêng lẻ cho từng service.

## 2. Sự khác biệt giữa việc Mount file và Mount thư mục trong Docker là gì?
### Trả lời:
- Mount file: Chỉ gắn kết một tệp duy nhất. Phù hợp cho các file cấu hình như nginx.conf hay settings.js. Nếu xóa file trên máy chủ, Docker có thể bị lỗi khi khởi động lại.

- Mount thư mục: Gắn kết toàn bộ folder. Mọi file tạo mới, sửa đổi hoặc xóa bên trong folder ở máy chủ đều được đồng bộ tức thì vào container. Thường dùng cho mã nguồn code hoặc cơ sở dữ liệu.

## 3. Nếu thay đổi file index.html ở máy Ubuntu, nội dung trên web có thay đổi ngay không? Tại sao?
### Trả lời:
- Có. Vì khi dùng volumes, Docker tạo ra một liên kết trực tiếp (bind mount) giữa máy chủ và container. Khi lưu file trên Ubuntu, container "nhìn thấy" sự thay đổi đó ngay lập tức mà không cần khởi động lại.

## 4. docker-compose.yml khai báo các services có phần restart: always hoặc restart: unless-stopped : chúng để làm gì?
### Trả lời:
- restart: always: Container sẽ tự động khởi động lại nếu bị lỗi hoặc nếu máy chủ (Ubuntu) bị khởi động lại (restart máy).

- restart: unless-stopped: Tương tự như always, nhưng có một ngoại lệ: nếu chủ động dùng lệnh docker compose stop để tắt nó đi, thì khi máy chủ khởi động lại, container này sẽ không tự bật lên. Điều này giúp kiểm soát tốt hơn các service muốn tạm dừng.

- Trong bài làm myapi và mynginx dùng always (dịch vụ quan trọng nên luôn phải chạy). mynodered và mycloudflared dùng unless-stopped (cho phép dừng thủ công khi cần debug).

## 5. Cách khai báo để tất cả các services đều dùng chung 1 network? lợi ích của việc khai báo này là gì?
### Trả lời:
- Khai báo: Trong mỗi service thêm networks: - mynetwork, và khai báo networks: mynetwork: driver: bridge ở cuối file.

services:
  mynodered:
    ...
    networks:
      - mynetwork

  mynginx:
    ...
    networks:
      - mynetwork

  mycloudflared:
    ...
    networks:
      - mynetwork

networks:
  mynetwork:
    driver: bridge
- Lợi ích: Các container có thể giao tiếp với nhau bằng tên service (ví dụ: Nginx gọi http://myapi:9630) thay vì dùng địa chỉ IP nội bộ hay IP công cộng. Nó cũng giúp cô lập các container này khỏi các container khác không thuộc dự án, tăng tính bảo mật.

## 6. Đưa Cloudflare Token vào trong file .env rồi sau đó thêm .env vào file .gitignore trước khi push code lên github. Tại sao nói đây là điều quan trọng về bảo mật mã nguồn?
### Trả lời:
- Trong bài đã thực hiện đưa token vào .env rồi thêm .env vào file .gitignore.

- Đây là điều quan trọng trong bảo mật mã nguồn vì:

  + Tránh lộ bí mật: Cloudflare Token là chìa khóa vạn năng vào hệ thống. Nếu push nó lên GitHub (đặc biệt là repo public), bất kỳ ai cũng có thể chiếm quyền điều khiển Tunnel và chèn mã độc vào web của bạn.

  + Tính di động: Dùng .env giúp dễ dàng thay đổi cấu hình cho các môi trường khác nhau (máy cá nhân, máy chủ thật) mà không cần sửa code trong file YAML.

## 7. Tại sao chúng ta nên thêm hậu tố :ro khi mount file cấu hình Nginx?
### Trả lời:
- :ro (Read-Only): Nghĩa là "Chỉ đọc".

- Lý do: Đảm bảo container Nginx không có quyền thay đổi hoặc xóa file cấu hình gốc trên máy Ubuntu của bạn nếu nó bị tấn công (hack). Đây là nguyên tắc "quyền hạn tối thiểu" (Least Privilege) trong bảo mật.

## 8. Khi dùng Cloudflare Tunnel: có cần thiết phải mở cổng cho các service nữa không?
### Trả lời:
- Không cần thiết phải mở cổng cho các service.

- Lý do: Thông thường ta phải mở port 80/443 trên Router/Firewall để người ngoài truy cập vào. Với Tunnel, container cloudflared sẽ tự tạo một kết nối đi ra ngoài tới Cloudflare. Dữ liệu sẽ đi qua đường ống này vào trong. Bạn có thể đóng hết tất cả các cổng trên máy Ubuntu, giúp server "tàng hình" trước các đợt scan port của hacker.
