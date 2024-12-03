# Configmap Kubernetes
- Ở bài trước chúng ta đã phải vào file properties của backend để đổi các credential để kết nối đến DB như là địa chỉ IP, username, password.
- Chỉ cần thêm bớt hay thay đổi giá trị ở đâu đó phải build lại Dockerfile
- Đồng thời phải tiến hành thay đổi image ở trên deployment rất mất thời gian và tài nguyên


## ConfigMap là gì?
- Search keyword ["How to add a value in a pod k8s"](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)
- có 2 cách để tạo biến môi trường cho các containers ở trong pod: env và envFrom
- env cho phép ta set ta set trực tiếp giá trị cho biến ở bên trong container
- envFrom sẽ dùng các giá trị được link tới cặp value-pairs: ConfigMap và Secret
- Trong bài này chúng ta chỉ tập trung vào ConfigMap

- ConfigMap cái tên nói lên tất cả, những cái file thiết lập hay biến được map ra bên ngoài tới 1 nơi lưu trữ cụ thể nào đó ở dạng cặp key-value hoặc là file.
## Các cách cấu hình ConfigMap
```yaml
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true 
```
- Property-like keys và file-like keys
## Chỉnh sửa Dockerfile đọc config
- Chỉnh sửa trong dockerfile để chỉ định đến file config cho container
```dockerfile
ENTRYPOINT java -jar /run/spring-boot-ecommerce-0.0.1-SNAPSHOT.jar --spring.config.location=run/src/main/resources/application.properties
```
- vị trí đường dẫn của location đó nằm ở trong container
- tạo 1 file ConfigMap và map vào trong thư mục bên trong Container
- Lúc đó dự án backend có thể đọc được file cấu hình

```bash
docker build -t garetttran/ecommerce-backend:v3 .
docker push garetttran/ecommerce-backend:v3
```
- build lại xong push lên dockerhub.

## Tạo ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ecommerce-backend-application-properties-configmap
  namespace: ecommerce
data:
  application.properties: |
    spring.datasource.url=jdbc:mysql://192.168.197.55:3306/full-stack-ecommerce
    spring.datasource.username=ecommerceapp
    spring.datasource.password=StrongPa55WorD
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
- chú ý dấu | sau khi đặt key
- apply cấu hình này trên Rancher
- Lúc này mặc dù có config map rồi nhưng container trong các pod chưa được sử dụng image mới
- Và đồng thời chưa sử dụng cái config map này
- Ta sẽ tiến hành map config map vào trước và sẽ thay đổi image
  
## Thêm ConfigMap vào Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    configMap:
      name: myconfigmap
```
- Ví dụ về cách sử dụng Config map cho Pod
- tạo 1 volume với tên mong muốn trong đó có configmap
- mount volume đó vào pod và chỉ định mount vào thư mục nào

- chỉnh sửa trong file yaml Deployment thay vì Pod
```yaml
volumes:
        - configMap:
            defaultMode: 420
            name: ecommerce-backend-application-properties-configmap
          name: ecommerce-backend-application-properties-config-volume
```
- Trong spec: thẳng hàng với container, tạo 1 volume

```yaml
volumeMounts:
            - mountPath: /run/src/main/resources/application.properties
              name: ecommerce-backend-application-properties-config-volume
              subPath: application.properties
```
- Bên trong container: thêm volumeMounts
- Mount volume vừa tạo vào trong thư mục chưa file properties
- subPath: chỉ định file cụ thể được mount vào trong container thay vì cả thư mục

![image](https://github.com/user-attachments/assets/b55263cb-05d9-4549-92a1-5aaf620b4941)
- vào môi trường container, và tìm kiếm thử file.
  
## Cập nhật Docker image trong Pod
- Chỉnh sửa trực tiếp image thành v3 trong rancher
- có thể chỉnh sửa config map nhưng lưu ý phải redeploy Deployment tương ứng để cập nhật cấu hình

# Key takeaway
- Như vậy là ta đã chỉnh sửa image của dockerfile lấy cấu hình của Spring trong đường dẫn
- Sau đó ta tiến hình build lại 1 image mới và push lên docker hub
- Tạo 1 configmap lưu trữ các gì trị của file .properties
- Tạo volume mới trong deployment và mount volume đó vào vị trí đã chỉ định cấu hình của Spring
- Thay đổi image, thay đổi configmap và redeploy container sẽ apply cấu hình mới.
