# NodePort trong K8S
- Expose ra 1 port và truy cập trực tiếp tới pod/deployment từ bên ngoài
## Range port của service NodePort
- Range: 30000 - 32767
```bash
--service-node-port-range
```

## Sử dụng NodePort
![image](https://github.com/user-attachments/assets/34b511d9-4df6-4340-ac69-9928c55ffca8)
- listen port 80 và map với port 80 của deployment do trước đó expose port 80 trên các pod
- chọn nodeport phải tuân thủ theo range ở trên

Selectors tab:
![image](https://github.com/user-attachments/assets/0fa8a636-52e9-4c8e-b699-dde6eee03bae)
- chọn loại tài nguyên là app và app name chính là tên của deployment
- Sau khi xác nhận tài nguyên có tồn tại sẽ thấy thông báo về số pod khả dụng trong deployment

Result:
![image](https://github.com/user-attachments/assets/807aa2d4-3416-42e6-b554-b6cdba2d3277)
- Service sẽ expose trên cổng đã chọn ở [Anynode]
- Client có thể kết nối đến nodeport nào cũng có thể truy cập vào app/deployment

# Key takeaway
- Nodeport thường không được sử dụng trong dự án thực tế
- Thường expose nhanh chóng để kiểm tra, debug, job thu thập thông tin nhanh chóng
