# Triển khai dự án fullstack k8s 
- Kiến thức cơ bản những bài trước đã đủ để ta triển khai 1 dự án fullstack trên k8s
- Triển khai dự án Micro-service sẽ khác so với các dự án deploy thông thường
  
## Mô hình dự án
![image](https://github.com/user-attachments/assets/9c506f60-7e6f-4355-8f38-64c73d237756)
- Dụ án backend có domain là api-garrett.devops.vn
- Front end có domain là ecommerce.garrett.vn
- 1 Server để triển khai database cho Spring boot
  
## Cài đặt server Database - Cài đặt database MariaDB
- Tạo 1 VM mới 1gb 2 cores, hostname: database-server 192.168.197.55
- Trong thực tế sẽ cần mount thêm disk để lưu data 

```bash
sudo apt update -y 
sudo apt install mariadb-server -y 
```
- Cài đặt mariadb server

/etc/mysql/mariadb.conf.d/50-server.cnf
```yaml
bind-address            = 0.0.0.0
```
- sửa từ local host thành 0.0.0.0 để mariadb được public trong local network
```bash
systemctl restart mariadb
mysql
```
- restart mariadb
- đăng nhập vào môi trường của mariadb

## Phân tích dự án
- Dùng docker để build image và push lên docker hub
- Sửa domain của dự án thành domain mong muốn
- Chạy dự án trong cluster của K8S bằng cái image đó, triển khai bằng deployment
  
## Di chuyển dự án vào server Database
- Trong windows mở cmd trong thư mục đang chứa file zip vừa mới tải về
```bash
scp .\Fullstack-Ecommerce-Web.zip garrett@192.168.197.55:/tmp
```
- Di chuyển file vào thư mục /tmp trong server database
  
- Trong Ubuntu, dưới quyền root vào thư mục /tmp kiểm tra xem có file không
```bash
ls -l
chown root:root Fullstack-Ecommerce-Web.zip
mkdir /projects
mv Fullstack-Ecommerce-Web.zip /projects
cd /projects
apt install unzip -y
unzip Fullstack-Ecommerce-Web.zip
```
- Di chuyển dự án vào trong /projects và tiến hành giải nén với unzip(tải nếu chưa có).
  
## Cài đặt Docker
```bash
apt install docker.io -y
```
- Quá trình này trong thực tế ta có thể dụng dủng CI/CD
## Cấu hình dự án frontend
- Di chuyển vào thư mục dự án front end
- Có sẵn 1 dockerfile với nội dung như sau:
```yaml
## build stage ##
FROM node:18.18-alpine as build

WORKDIR /app
COPY . .

RUN npm install --force
RUN npm run build

## run stage ##
FROM nginx:alpine

COPY --from=build /app/dist/angular-ecommerce /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
- build stage:
  + FROM kéo base image node vì đây là dự án Angular
  + WORKDIR chỉ định thư mục làm việc
  + COPY thư mục hiện tại chứa các dependency và source vào trong container
- Run stage:
  + FROM kéo image Nginx về để expose dự án
  + Copy từ layer Build file Runtime executor vào thư mục mặc định HTML của Nginx
  + Expose port 80 mặc định của Nginx và CMD chạy dòng lệnh khởi động Nginx
 
```bash
docker build -t ecommerce-frontend:v1 .
docker images
docker login
docker tag ecommerce-frontend:v1 garetttran/ecommerce-frontend:v1
docker push garetttran/ecommerce-frontend:v1
```
- sử dụng dockerfile trong thư mục hiện tải để build image bằng dấu chấm cuối lệnh
- flag -t để chỉ định tag cho image theo cú pháp tên:phiên bản
- login vào tài khoản của docker hub để chuẩn bị push image lên
- đánh tag image để push vào repo tương ứng
  
## Tạo Namespace dự án
- dựa vào các file yaml mẫu từ những bài trước ta sẽ thay thế các image để các pod bây h sẽ chạy dự án frontend này

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ecommerce-frontend
  name: ecommerce-frontend-deployment
  namespace: ecommerce
spec:
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: ecommerce-frontend
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: ecommerce-frontend
      namespace: ecommerce
    spec:
      containers:
        - image: garetttran/ecommerce-frontend:v1
          imagePullPolicy: Always
          name: ecommerce-frontend
          ports:
            - containerPort: 80
              name: tcp
              protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: ecommerce-frontend-clusterip
  namespace: ecommerce
spec:
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - name: tcp
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: ecommerce-frontend
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-frontend-ingress
  namespace: ecommerce
spec:
  ingressClassName: nginx
  rules:
    - host: ecommerce.devopsedu.vn
      http:
        paths:
          - backend:
              service:
                name: ecommerce-frontend-clusterip
                port:
                  number: 80
            path: /
            pathType: Prefix
```
- đổi tên app, namespace và name
- chú ý domain host của ingress phải khớp với domain của dự án
- Sau này sẽ có lưu ý về phần xác thực image của k8s trước khi kéo về để tránh lỗi xảy ra
## Áp dụng cấu hình frontend
- Trên Rancher, tạo project ecommerce và namepspace ecommerce trong project đó.
- Import file yaml vào đúng namespace đã tạo
- Không nên tạo quá nhiều namespace cho dự án vì sau này còn có các job và các scheduler
- Add host vào windows, vẫn chỉ định đến server load balancer setting của ingress không đổi
- Truy cập bằng: http://ecommerce.devopsedu.vn/
  
## File cấu hình backend
- di chuyển vào thư mục của dự án backend
  
src/main/resources/application.properties - đây là thông tin kết nối đến db
```yaml
spring.datasource.url=jdbc:mysql://192.168.197.55:3306/full-stack-ecommerce
spring.datasource.username=ecommerceapp
spring.datasource.password=StrongPa55WorD
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
spring.datasource.sql-script-encoding=UTF-8
spring.jpa.properties.hibernate.globally_quoted_i/dentifiers=true
spring.jpa.hibernate.ddl-auto=none
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

spring.data.rest.base-path=/api

allowed.origins=http://ecommerce.devopsedu.vn

spring.data.rest.detection-strategy=ANNOTATED

okta.oauth2.client-id=0oab0lzwjoN1Rjsar5d7

okta.oauth2.issuer=https://dev-82108115.okta.com/oauth2/default
```
- chỉnh sửa địa chỉ ip và domain tương ứng
## Thêm dữ liệu vào database
- Để tránh gặp lỗi ta cần tải các dữ liệu database trước rồi mới deploy backend
- Di chuyển vào thư mục của database và vào môi trường mariadb
- source "file.sql" để nạp dữ liệu vào db
  
## Cấu hình dự án backend
- quay trở lại thư mục dự án backend và ta cũng có 1 Dockerfile

```yaml
## build stage ##
FROM maven:3.8.3-openjdk-17 as build

WORKDIR ./src
COPY . .

RUN mvn install -DskipTests=true

## run stage ##
FROM openjdk:17-alpine

RUN unlink /etc/localtime;ln -s  /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime
COPY --from=build src/target/spring-boot-ecommerce-0.0.1-SNAPSHOT.jar /run/spring-boot-ecommerce-0.0.1-SNAPSHOT.jar

EXPOSE 8080

ENTRYPOINT java -jar /run/spring-boot-ecommerce-0.0.1-SNAPSHOT.jar
```
- Build Stage:
  + FROM pull image maven để chạy Springboot java
  + WORKDIR chỉ định thư mục src là thư mục làm việc trong container
  + COPY thư mục hiện tại vào thư mục làm việc
  + RUN mvn install để tiến hành tải các dependency và build dự án
- Run stage:
  + FROM chỉ cần kéo base image nhẹ hơn do không cần build nữa mà chỉ cần chạy file java runtime
  + RUN set time zone múi giờ +7
  + COPY từ layer build file *.jar runtime
  + mở port 8080 từ container cũng tức là port mà springboot service mở
  + ENTRYPOINT sẽ tiền hành chạy file runtime
 
```bash
docker build ecommerce-backend:v1 .
docker tag ecommerce-backend:v1 garetttran/ecommerce-backend:v1
docker push garetttran/ecommerce-backend:v1
```
- tương tự như dự án frontend
  
## Áp dụng cấu hình backend
- Trong file yaml, đổi tên các mục tương ứng
- lưu ý containerPort thành 8080, host api-ecommerce.devopsedu.vn
- Áp dụng cấu hình lên trên Rancher yaml import
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ecommerce-backend
  name: ecommerce-backend-deployment
  namespace: ecommerce
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: ecommerce-backend
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: ecommerce-backend
      namespace: ecommerce
    spec:
      containers:
        - image: garetttran/ecommerce-backend:v2
          imagePullPolicy: Always
          name: ecommerce-backend
          ports:
            - containerPort: 8080
              name: tcp
              protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: ecommerce-frontend-clusterip
  namespace: ecommerce
spec:
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - name: tcp
      port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    app: ecommerce-backend
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-backend-ingress
  namespace: ecommerce
spec:
  ingressClassName: nginx
  rules:
    - host: api-ecommerce.devopsedu.vn
      http:
        paths:
          - backend:
              service:
                name: ecommerce-backend-clusterip
                port:
                  number: 8080
            path: /
            pathType: Prefix
```
![image](https://github.com/user-attachments/assets/92896c52-91bf-4084-8116-f9df4913e9a3)
- test API để lấy các products, dự án backend được deploy thành công

![image](https://github.com/user-attachments/assets/1eb8fc9b-5d91-4d1b-883d-683ce44359f4)
- Dự án frontend lấy data thành công từ backend

![image](https://github.com/user-attachments/assets/b9841ebc-8c23-49ae-8649-1b068bed27d7)
- Dự án sẽ lỗi nếu ta truy cập trực tiếp vào domain */products
- Tuy nhiên nếu truy cập từ domain http://ecommerce.devopsedu.vn thì lại không bị
  
- Phòng đoán ban đầu và có thể sai:
  + Trong phần db setting - src/main/resources/application.properties chỉ cho phép: allowed.origins=http://ecommerce.devopsedu.vn được query data
  + Do có lỗi trong cách backend và frontend làm việc với nhau
# Key Takeaway
- Như vậy là ta đã biết cách sử dụng dockerfile, dockerhub để build và push image của dự án lên và dùng k8s cùng với file yaml mẫu để deploy dự án backend + database và frontend
- Tại sao các pod trong dự án frontend không được add host mà vẫn có thể gọi tới các api của dự án backend? Cần tìm hiểu thêm
- Lỗi khi refresh page cũng cần được tìm hiểu thêm vì nó có thể liên quan đến cách kết nối. Hoặc đôi khi là trường hợp dev team quăng cho ops team con lợn chết và bắt team ops phải bắt nó đi như trong quyển "Dự án Phương Hoàng" :)))).
