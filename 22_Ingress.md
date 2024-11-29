# [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) trong K8S

## Ingress là gì?
- Ingress sử dụng cơ chế cấu hình nhận biết giao thức(protocol-aware config mechanism) để làm cho service network HTTP hoặc HTTPS khả dụng
- Ingress cho phép map  traffic tới những backend thông qua luật được người đung định sẵn thông qua k8s api
- Ingress cung cấp balancing, SSL termination và name-based virtual hosting.
- Hiện tại Ingress đang bị đóng băng và nhiều tinh nắng mới hơn đang được thêm vào [Gateway API](https://kubernetes.io/docs/concepts/services-networking/gateway/).
  
## Mô hình triển khai
![image](https://github.com/user-attachments/assets/367fad0e-9c84-4c0b-851e-1c0d3de15d20)
- Ví dụ về cách Ingress gửi tất cả traffic của nó tới 1 Service
- Ingress-managed load balancer cần được tìm hiểu thêm
  
## [Ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
- Bắt buộc phải tạo 1 Ingress controller để define các rule cho ingress giao tiếp với cluster
- K8S hỗ trợ Ingress Controller cho AWS, GCE hay Nignx
  
## [Ingress nginx](https://kubernetes.github.io/ingress-nginx/deploy/)
- Các cách tải Ingress Nginx:
    + Sử dụng Helm, project repo chart
    + sử dụng ```kubectl apply```, Yaml
    + addons như minukube hay MicroK8s

## Tìm hiểu Helm
- Công cụ quản lý các gói tài nguyên trong kubernetes
- Có thể hiểu tương tự như apt trong ubuntu
- Package trong Helm được gọi là Charts

## Cài đặt Helm
```bash
# wget https://get.helm.sh/helm-v3.16.2-linux-amd64.tar.gz
# tar xvf helm-v3.16.2-linux-amd64.tar.gz
# sudo mv linux-amd64/helm /usr/bin/
# helm version
```
- di chuyển file helm có quyền thực thi đến thư mục /usr/bin/

## Cài đặt Ingress nginx với Helm
- Ingress Nginx là 1 ingress controller sử dụng nginx như 1 reverse proxy và load balancer
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm search repo nginx
helm pull ingress-nginx/ingress-nginx
tar -xzf ingress-nginx-4.11.3.tgz
```

vi ingress-nginx/values.yaml
```yaml
>> Sửa type: LoadBalancer => type: NodePort
>> Sửa nodePort http: "" => http: "30080"
>> Sửa nodePort https: "" => https: "30443"
```
- không set NodePort thì nó sẽ được random làm cho việc quản lý khó khăn hơn

```bash
cp -rf ingress-nginx /home/devops/
su - devops
```
- di chuyển file đến thư mục devops vì chỉ có users devops mới có thể thiết lập lên cụm

```bash
kubectl create ns ingress-nginx
helm -n ingress-nginx install ingress-nginx -f ingress-nginx/values.yaml ingress-nginx
```
- Tạo 1 namespace cho ingress-nginx hoạt động tương tự như 1 proxy server riêng biệt
- helm -n "namespace" install "release name" -f "file cấu hình của controller" "heml chart"

## Cài đặt LoadBalancer
- Trước khi client đi vào ingress sẽ có 1 load balancer
- Cần phải tạo 1 server mới cho loadbalancer:
    + hostname: loadbalancer-k8s
    + ip: 192.168.197.110
    + cpu 2 cores, ram 2Gi
 
Trên server loadbalancer-k8s
```bash
apt update
apt install nginx
```

vi /etc/nginx/sites-available/default
```yaml
listen 9999 default_server;
listen [::]:9999 default_server;
```
- Đổi port listen 80 thành 9999 vì ta sẽ dùng port đó

vi /etc/nginx/conf.d/garrett.devops.conf
```yaml
upstream my_servers {
    server 192.168.197.111:30080;
    server 192.168.197.222:30080;
    server 192.168.197.33:30080;
}

server {
    listen 80;

    location / {
        proxy_pass http://my_servers;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
- Khai báo các Nodeport với các port đã được exposed
- Lắng nghe trên cổng 80 và map vào các NodePort đang được sử dụng bởi Ingress Nginx
- Restart nginx trên server loadbalancer-k8s
## Triển khai Ingress
- triển khai ingress trên Rancher bằng file yaml mẫu khi cài đặt Ingress Nginx

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: car-serv-ingress
  namespace: car-serv
spec:
  ingressClassName: nginx
  rules:
    - host: car-serv.garrett.devops
      http:
        paths:
          - backend:
              service:
                name: car-serv-clusterip
                port:
                  number: 80
            path: /
            pathType: Prefix
```
-   ingressClassName: nginx phải chỉ định className
-   Đổi tên host mong muốn
-   Ở phần backend, chỉ định đến cluserIP service để ingress có thể interact với app/deployment

# Key takeaway
- Lưu ý Ingress Nginx và Nginx Ingress khác nhau. bài này sử dụng [Ingress Nginx](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwinhY-T0YCKAxVrslYBHaBmEE4QFnoECB8QAQ&url=https%3A%2F%2Fgithub.com%2Fkubernetes%2Fingress-nginx&usg=AOvVaw1f7vc0Vw_cKFVqrYcW6toW&opi=89978449)
- Ingress Controller hay được sử dụng trong thực tế on-premise:
    + [Nginx Ingress Controller for Kubernetes](https://www.f5.com/products/nginx/nginx-ingress-controller)
    + [HAProxy Ingress](https://haproxy-ingress.github.io/)
    + [Kong Ingress Controller for Kubernetes](https://github.com/Kong/kubernetes-ingress-controller#readme)
- Helm có thể giúp chúng ta đóng gói các file cấu hình thành các template tăng khả năng sử dụng lại cấu hình.
- Add Host trên window cho server loadbalancer-k8s -> nginx trên server loadblancer-k8s foward request từ port 80 tới các NodePort đang được expose trong Ingress(30080) -> Ingress sẽ dựa vào routing rule và service backend(car-serv-clusterip) mà nó associate để forward tới các pod trong app/deployment.
