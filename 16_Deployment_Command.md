# CÁC CÂU LỆNH TRONG DEPLOYMENT #

## Lấy danh sách Deployment
```bash
kubectl get deployments -n car-serv
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
car-serv-deployment   3/3     3            3           8h
```

## Lấy danh sách ReplicaSet
```bash
kubectl get rs -n car-serv
NAME                             DESIRED   CURRENT   READY   AGE
car-serv-deployment-5b6745496c   3         3         3       8h
```
## Thiết lập Docker image
```bash
kubectl set image deployment.v1.app/nginx-deployment nginx=nginx:1.16.1
```
- Thường được sử dụng để update version
- Cập nhật Docker image phiên bản mới

## Chỉnh sửa trực tiếp Deployment
```bash
kubectl edit deployment/car-serv-deployment -n car-serv
```
cho phép chỉnh sửa trực tiếp config của deployment bằng Vi
## Rollout status
```bash
kubectl rollout status deployment/car-serv-deployment -n car-serv
deployment "car-serv-deployment" successfully rolled out
```
## Cập nhật số lượng ReplicaSet
```bash
kubectl scale deployment/car-serv-deployment -n car-serv --replicas=4
deployment.apps/car-serv-deployment scaled
```
- thay vì scale trên Rancher thì dùng câu lệnh này
- Tăng số lượng pod lên thành 4
## Autoscale
```bash
kubectl autoscale deployment/car-serv-deployment -n car-serv --min=10 --max=10 --cpu-percent=80
```
- Autoscale dùng để dự án tự động tăng giảm số lượng pod để đảm bảo tính HA và Scalability
## Pause và Resume
- Kiểm soát phiên bản trước khi triển khai lên các pod đảm bảo cấu hình được thiết lập đúng
# Key Takeaway
- Những câu lệnh cmd sẽ phục vụ những môi trường không thể sử dụng GUI
- Học để hiểu rõ ràng hơn các chứng năng
- Hiểu sâu về command giúp chúng ta debug trực tiếp trên server và ít lệ thuộc hơn vào giao diện
