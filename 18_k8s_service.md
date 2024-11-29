# Services K8S 

## Service là gì?
- Expose app đang chạy trong cluster ra 1 endpoint bên ngoài, trong lúc đó workload được phân tán ra nhiều backends
- Sử dụng service để pod khả dụng trên network và client có thể kết nối đến
- Nếu có pod backend và pod frontend thì làm sao có thể interact với nhau khi mà podIP là ephemeral và bị tự động thêm/xóa bởi deployment?

## Service type
![image](https://github.com/user-attachments/assets/682dd7b7-9463-4722-a278-b44d0730d597)

## ClusterIP
- tạo ra 1 địa chỉ IP chỉ có thể được truy cập ở trong cụm mà bên ngoài không thể sử dụng
- Phải sử dụng Ingress hoặc Gateway để expose ra bên ngoài
## NodePort
- Mở 1 port trực tiếp từ ứng dụng ra bên ngoài
- không đi đúng với workflow của dự án -> khó kiểm soát và bảo mật thấp
## LoadBalancer
- Expose service ra bên ngoài bằng 1 load balancer bên ngoài
- Có thể được tích hợp với load balancer của cloud provider
## ExternalName
- Liên kết với 1 tên miền ở bên ngoài
- điều phối ra 1 nơi khác thay vì tương tác với cụm k8s

# key takeaway
- ClusterIP và LoadBalancer là 2 service được sử dụng nhiều trong triển khai dự án thực tế
- CLusterIP trên on-premise
- LoadBalacner trên cloud
- Dễ dàng kiểm soát traffic và tăng tính bảo mật cho cụm
