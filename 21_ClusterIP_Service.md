# ClusterIP Service K8S

## Sử dụng ClusterIP
![image](https://github.com/user-attachments/assets/b5c354e7-789d-487e-902e-5ea6d629efe4)

![image](https://github.com/user-attachments/assets/479b1669-7470-43d0-b1c0-b89f8ad1f8e2)

- Chỉnh Service ports và Selectors tương tự NodePort
- ClusterIP không cho phép truy cập trực từ bên ngoài vào thay vào đó phải sử dụng API từ rancher và được authenticated đầy đủ với có thể truy cập tới
    + https://rancher.garrett.devops/k8s/clusters/c-m-4crn4krt/api/v1/namespaces/car-serv/services/http:car-serv-clusterip:80/proxy/
- Các tài nguyên trong cụm có thể giao tiếp với nhau mà không cần phải expose ra bên ngoài
    + Dự án frontend sử dụng API hoặc ClusterIP để gọi tới các app đang chạy backend ở trong cụm
  
# Key takeaway
- ClusterIP là service hữu ích cho các app giao tiếp với nhau trong cùng 1 cụm
- Muốn expose ra bên ngoài về sau cần tìm hiểu Ingress và gateway
