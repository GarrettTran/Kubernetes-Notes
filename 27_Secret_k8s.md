# Secret in Kubernetes
- Các file cấu hình resource chưa được bảo mật khi dùng ConfigMap
- Secret sẽ hỗ trợ thêm nhiều tính năng bảo mật

## Secret là gì?
- Secret là 1 object chứa 1 lượng nhỏ data nhạy cảm như pw, token, key
- Những thông tin này có thể được cấu hình trong pod hoặc image
- Sử dụng Secret sẻ loại bỏ được việc sử dụng chúng trực tiếp trong source code
- Secret cũng giống như Configmap nhưng đc thiết kế riêng để chưa thông tin bảo mật
- Các thông tin trong secret sẽ được mã hóa base64
- Việc này còn giúp dữ liệu truyền đi giữa website không bị lỗi định dạng
## Secret type
![image](https://github.com/user-attachments/assets/8d9b6f86-efef-4545-8fa7-804a6f96c340)
- Nếu không ghi gì ở trước nghĩa là xài lib k8s.io
- Bài này chúng ta chỉ tập trung sử dụng Opaque và dockerconfigjson
### Opaque
- Được sử dụng phổ biến và dễ dùng
- Cũng sử dụng dạng key-value
### service-account-toke
- xác thực api của k8s
- hay được sử dụng trong RBAC(role-based access control)
### dockercfg
- Lưu trữ các thông tin đăng nhập của Registry
- Định dạng cfg cũ để chứa credential
- hạn chế 
### dockerconfigjson
- giống CFG nhưng xài định dạng Json
### basic-auth
- Website khi được request sẽ gửi về code 401 để cho người dùng
- Yêu cầu 2 trường username và password
- Do chỉ là basic nên không quá an toàn
### ssh-auth
- Cũng chỉ chứa 2 giá trị username và pw khi ssh
### tls
- Chứa chứng chỉ xác thự tls và private key để sử dụng cho https
- Rất hay được sử dụng
### bootstrap.kubernetes.io/token
- Khi vừa tạo Control-plane hay cluster mới
- Sẽ có key để join các node khác vào cụm
## Tạo secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ecommerce-backend-database-connection
  namespace: ecommerce
type: Opaque
stringData:
  MARIADB_HOST: "192.168.197.55"
  MARIADB_DB: "full-stack-ecommerce"
  MARIADB_PORT: '3306'
  MARIADB_USERNAME: "ecommerceapp"
  MARIADB_PASSWORD: "StrongPa55WorD"
```
- thêm các key-value cho secret
## Thêm Secret vào Pod
- Tiến hành thêm secret vào pod từ giao diện
![image](https://github.com/user-attachments/assets/9364bd77-4618-4ccd-9dd0-34fb5291379e)
- Như vậy là pod đã có các biến môi trường từ secret
- và ta sẽ chỉnh sửa các trường này trong file application.properties 

```yaml
- envFrom:
            - secretRef:
                name: ecommerce-backend-database-connection
                optional: false
```
- envFrom và scretRef đã được thêm vào

![image](https://github.com/user-attachments/assets/7814b873-9746-4e28-a60d-bf82372bf8de)
- Biến môi trường đã được thêm vào container
## Sửa ConfigMap nhận giá trị Secret
- Đưa config map và các file cấu hình cho dev review
- Thường sẽ bao gồm các file cấu hình, mật khẩu, tài khoản và api key
- Lộ các pemkey xác thực cloud

```yaml
spring.datasource.url=jdbc:mysql://${MARIADB_HOST}:${MARIADB_PORT}/${MARIADB_DB}
spring.datasource.username=${MARIADB_USERNAME}
spring.datasource.password=${MARIADB_PASSWORD}
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
spring.datasource.sql-script-encoding=UTF-8

spring.jpa.properties.hibernate.globally_quoted_i/dentifiers=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
spring.jpa.hibernate.ddl-auto=none
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

spring.data.rest.base-path=/api
spring.data.rest.detection-strategy=ANNOTATED

allowed.origins=http://ecommerce.devopsedu.vn

okta.oauth2.client-id=0oab0lzwjoN1Rjsar5d7
okta.oauth2.issuer=https://dev-82108115.okta.com/oauth2/default
```

- Lý do có bao gồm tùy chọn port mặc dù DB mặc định là 3306 bởi vì
- Có thể giấu port ở những chỗ ít bị truy vết hơn
- Có thể triển khai cluster HA
