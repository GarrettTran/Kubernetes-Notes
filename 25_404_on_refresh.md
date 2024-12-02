# Lỗi 404 xuất hiện mỗi khi refresh page
- trong cấu hình nginx của t ko có serve cái file index.html cho mấy cái đường dẫn /products. Nma mấy cái này SPA mấy cái routing này phụ thuộc vào client
- t log vào cái pod xong check thì ra trong các pod của dự án front end có 1 service nginx đang chạy để expose cái file html.index angular ở trong pod ra ngoài và để các Service có thể truy cập tới
- nó thiếu cái file giống v mà bth t làm bài nào cũng thêm vô, cái base image này cơ bản quá với cách t làm cũng cơ bản
- Nó default lấy trong file mặc định là /usr/share/nginx/html; ra để chạy SPA nma trong lúc t build image t cũng chỉ quăng file dist vô đây.

```bash
2024/12/02 06:22:07 [error] 30#30: *38 open() "/usr/share/nginx/html/products" failed (2: No such file or directory), client: 172.17.168.33, server: localhost, request: "GET /products HTTP/1.1", host: "ecommerce.devopsedu.vn"

172.17.168.33 - - [02/Dec/2024:06:22:07 +0000] "GET /products HTTP/1.1" 404 555 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36 Edg/131.0.0.0" "192.168.197.33
```
- 2 đoạn error log xuất hiện sau khi refresh trong log của pod đang chạy dự and frontend
- Dùng Rancher cũng có thể view log của pod đang chạy
```yaml
server {
    listen 80;
    server_name ecommerce.devopsedu.vn;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}
```
- Tóm Lại try_files $uri /index.html Làm Gì?
  + Kiểm tra file hoặc thư mục thực tế trên server (dựa vào $uri).
  + Nếu không tìm thấy file, trả về file index.html.
  + File index.html sau đó được SPA (Angular) sử dụng để xử lý tất cả route phía client.
 
# Key takeaway
- Như vậy là đã sửa được lỗi 404 khi refresh
