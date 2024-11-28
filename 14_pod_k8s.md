# Pod Kubernetes #

- cứ mỗi khi xong 1 phần thì nên snapshot server để có thể quay về backup tương ứng
- tư duy back up rất quan trọng trong công việc liên quan đến hệ thống 
- tư tuy dev -> miễn chạy là được
- tư duy sys -> chạy không được thì sao?

## Pod là gì và pod để làm gì##
- Pod là đơn vị nhỏ nhất và đơn giản nhất
- 1 Pod sẽ bao gồm nhiều container được nhóm lại với nhau
- Các pod sẽ chia sẻ tài nguyên và mạng với nhau

<img width="325" alt="image" src="https://github.com/user-attachments/assets/720435b9-90f2-4b99-b52d-9137293caa6d">

- Pod giống như 1 vỏ bọc gom các container lại
- Pod là môi trường chia sẻ và máy chủ ứng dụng

## Pod dùng như thế nào ##
- vào master 1
```bash
su devops
cd ~/projects
mkdir car-serv
vi ns.yaml 
```

ns.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: car-serv
```
- Set up namespace để dự án chỉ chạy trong đúng namespace này

Tiếp theo là set up pod
pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: car-serv
  namespace: car-serv
spec:
  containers:
  - name: car-serv
    image: elroydevops/car-serv
    ports:
    - containerPort: 80
```
- viết file yaml giống namespace
- có các trường như apiV, meta, kind
- containers chỉ định image và port expose(thường là 80)
- phải chỉ định namesapce

Cách vào môi trường pod:
```bash
kubectl exec -it car-serv -n car-serv -- /bin/bash
```

# key takeaway
- 1 Pod chỉ nên có 1 container để giảm thiểu sự phụ thuộc và dễ debug.
- Có thể lên docker hub xem các layer của image
- kubectl giao tiếp thông qua api của cụm k8s
- Chạy độc lập pod và setting pod thủ công trong thực tế không được sử dụng
-> Cần 1 tài nguyên để triển khai nhiều pod để tăng khả năng chịu tải và broadcasting
  
