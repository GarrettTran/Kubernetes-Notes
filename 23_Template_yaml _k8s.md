# Template file yaml

## Download files yaml
- Download các file cấu hình của service, ingress, deployment
  
## Tối ưu file yaml Deployment
- xóa bỏ các phần annotations và managedFields
- creationTimestamp và generation và status
  
## Tối ưu file yaml Service
- cũng tương tự ở trên
- xóa thêm phần clusterIP vì nó là phù du được khởi tạo và xóa liên tục
- Nhắc lại selector phải chỉ đúng tên của deployment
  
## Tối ưu file yanl Ingress
- Riêng ingress thì annotation khá là quan trọng và giúp chúng ta mở rộng setting thêm được nhiều thứ nhưng phạm vi hiện tại chưa cần dùng đến
- riêng phần backend thì nhắc lại là phải chọn đúng cái service name
  
## Lưu ý viết file yaml K8s
- Label hay app name nên để ngắn nhất và thể hiện đúng cái tài nguyên
- tối giản hơn nữa thì có thể xóa loại tai nguyên ra khỏi app name vì trong meta data cũng có giá trị name
## Áp dụng cấu hình file yaml
- xóa hết deployment, 2 services và ingress
- sử dụng file yaml mẫu đó để build lại
- Sử dụng cú pháp phân file --- của yaml để import nhiều file cùng lúc

# Key Takeaway
- file yaml chuẩn format có thể được lưu lại để khôi phục các tài nguyên trong cụm sau sự cố
- Ngoài ra còn phục vụ tốt cho mục đích nghiên cứu và làm theo.
- Xóa hết tài nguyên đi vì bài sau chúng ta sẽ triển khai dự án FullStack
