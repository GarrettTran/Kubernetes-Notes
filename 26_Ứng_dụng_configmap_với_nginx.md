# Ứng dụng configmap vào cấu hình nginx của container

## Background
- Như đã đề cập về lỗi 404 đã được sửa khi refresh page
- Lúc đó ta đã vào thẳng môi trường container và thêm cấu hình nginx
- Tuy nhiên cách đó rất mất thời gian và cấu hình sẽ bị biến mất khi pod bị kill bới deployment

## Sử dụng configmap để thêm cấu hình nginx thuận tiện hơn nếu cần chỉnh sửa sau này
### Bước 1: thêm ConfigMap ecommerce.conf
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ecommerce-frontend-nginx-configmap
  namespace: ecommerce
data:
  ecommerce.conf: |
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

### Bước 2: Tiến hành tạo volume và mount volume đó vào trong container
- Ta sẽ chỉnh sửa trên file của deployment dự án frontend
- Lưu ý là subPath sẽ giúp chúng ta mount 2 file với nhau tránh ghi đè lên thư mục

```yaml
volumes:
        - configMap:
            defaultMode: 420
            name: ecommerce-frontend-nginx-configmap
          name: ecommerce-frontend-nginx-config-volume
```
- thêm volume vào trước
```yaml
volumeMounts:
            - mountPath: /etc/nginx/conf.d/ecommerce.conf
              name: ecommerce-frontend-nginx-config-volume
              subPath: ecommerce.conf
```
- mount volume vào trong thư mục và chỉ định đến đúng file để tránh ghi đè thư mục
- Apply cấu hình mới cho deployment

### Bước 3: Kiểm tra
- Shell vào môi trường của container

![image](https://github.com/user-attachments/assets/969a7183-0a45-4609-a120-d6836e1375ab)
- đã có file cấu hình nginx mới

![image](https://github.com/user-attachments/assets/876a8d2e-1346-4731-9a0a-83c1916647c9)
- Sau khi refresh website đã hoạt động lại bình thường
