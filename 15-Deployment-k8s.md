# Deployment k8s #

## Deployment là gì ##
- Dùng workload management để chạy nhiều pod
- Deployment là chiến lược của quầy bán bánh
- Tài nguyên ReplicaSet đại diện cho số pod

### Core function của Deployment
- tự động lưu lịch sử phiên biển thuận lợi cho việc rollback
- tự động scale tăng giảm các pod tăng khả năng chịu tải
- tự động sử lỗi, khi pod die sẽ tự động tạo pod mới
  
## Thao tác trên rancher ##
Chọn name space và import file yaml vào 
-> không cần phải truy cập vào trong môi trường server mà thao tác trực tiếp với cụm trên rancher

- Cẩn thận thì có thể khóa môi trường command line trên Rancher

### tạo Deployment
![image](https://github.com/user-attachments/assets/2e41b836-0fb7-453d-a5af-45f085697cda)
3 pod đã được chạy trong deployment và từng pod được triển khai trên từng master
podIP là ip trong cụm k8s chỉ sử dụng trong cụm

### Download file yaml của Deployment 

- Để lại những trường setting không cần những trường status và tự động tạo
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    workload.user.cattle.io/workloadselector: apps.deployment-car-serv-car-serv-deployment
  name: car-serv-deployment
  namespace: car-serv
spec:
  replicas: 3
  revisionHistoryLimit: 11
  selector:
    matchLabels:
      workload.user.cattle.io/workloadselector: apps.deployment-car-serv-car-serv-deployment
  template:
    metadata:
      labels:
        workload.user.cattle.io/workloadselector: apps.deployment-car-serv-car-serv-deployment
      namespace: car-serv
    spec:
      containers:
        - image: elroydevops/car-serv
          imagePullPolicy: Always
          name: car-serv
          ports:
            - containerPort: 80
              name: tcp
              protocol: TCP
```
File deployment này có thể dùng để tạo các pod tương tự Rancher

![image](https://github.com/user-attachments/assets/3075bd6d-7aea-42c0-8534-651195fd205d)
Khi sử dụng Deployment, Service sẽ làm việc với Deployment để đảm bảo số lượng pod trong 1 service

# Key Takeaway
- Rancher là 1 công cụ mạnh mẽ thao tác trên cụm K8S
- Deployment đảm bảo số lượng pod, sửa lỗi và rollback
- File yaml của Rancher được dùng làm format chuẩn và có thể học theo dễ dàng khi lược bỏ các phần không quan trọng
