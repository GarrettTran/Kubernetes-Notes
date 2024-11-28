# CÁC CHIẾN LƯỢC DEPLOYMENT #

## Chiến lược triển khai Deployment
![image](https://github.com/user-attachments/assets/ceefaca0-d854-429f-bed2-10f82c430aec)

### Rolling update (Chiến lược hay được sử dụng do an toàn)
- Cập nhật dần các pod
- Cứ 1 pod mới lên thì xóa 1 pod cũ đi -> Vẫn có 1 pod cũ và 1 pod mới -> đảm bảo dự án vẫn có thể hoạt động được
### Recreate (nhanh hơn)
- Tạo mới hoàn toàn các pod
- Xóa toàn bộ pod đi và khởi tạo lại toàn bộ pod

## Thiết lập Deployment
- Quy trình triển khai Rolling Update:
  + Ví dụ 1: replicas: 4, maxSurge: 1, maxUnavailable: 1
  + Quy trình: tạo 2 pod mới trong trạng thái Containercreating và 1 pod cũ -> 2 pod đang đc tạo và 1 pod bị xóa đi -> 5 pod
 
deployment.yaml chiến lược rolling update
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: car-serv-deployment
  name: car-serv-deployment
  namespace: car-serv
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1 #25%
      maxUnavailable: 1 #25%
  revisionHistoryLimit: 11
  selector:
    matchLabels:
      app: car-serv-deployment
  template:
    metadata:
      labels:
        app: car-serv-deployment
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
- Labels đổi thành app thay vì đường dẫn của Rancher
- App: car-serv-deployment cho ngắn gọn
- maxSurge(có thể là số tự nhiên hoặc %) số pod tố đa được tạo ra trên mức cần thiết của số pod trong deployment
- maxUnavailable(có thể là số tự nhiên goặc %) số pod tối đa được trong trạng thái downtime trong deployment
## Triển khai chiến lược Recreate
tương tự file deployment.yaml ở trên, chỉ sửa đổi strategy thành recreate
```bash
strategy:
    rollingUpdate:
      type: Recreate
```

![image](https://github.com/user-attachments/assets/4c052b5c-9b94-42c1-82a2-d70b3cc7e26e)
- import file deployment.yaml cho chiến lược recreate trước
- Nhớ chọn đúng namespace
- Sau khi thay đổi image chúng ta thấy pod được xóa hết và khởi tại lại với image mới đồng thời
- Với các dự án lớn cần thời gian khởi tạo lâu sẽ tạo ra downtime lớn
- Các cập nhật cho các pod có image nhẹ thì rất tiện lợi
## Triển khai chiến lược RollingUpdate
![image](https://github.com/user-attachments/assets/293a5f10-3f96-47a8-b154-d9bca29f6d80)
- Sau khi tăng replicaSet lên 4 và thay đổi image thành nginx
- 1 Pod với image car-serv được triển khai để đảm số lượng pod 3/4
- 2 pod mới được khởi tạo pull image của nginx về do chỉ được vượt 1 pod 5/4

# key takeway
- Các chiến lược giúp quá trình triển khai dự án chuyên nghiệp, tự động và hạn chế downtime
- Thay vì phải cập nhật từng server đang chạy dự án đó
- Việc khởi tạo dự án đã rành mạch -> Làm sao để dự án được expose ra Internet?
