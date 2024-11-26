# Rancher và cách quản lý cụm k8s #

## rancher là gì? ##
- Rancher là công cụ giúp triển khai, quản lý và giám sát nhiều cụm k8s trên nhiều môi trường khác nhau.

## rancher làm được gì? ##
- Tạo tài nguyên bằng giao diện - giám sát và quản lý trực quan
- Phân quyền trên rancher rất mạnh mẽ
- Bảo mật tốt với nhiều cách authen hỗ trợ mã hóa và cluster isolation
- Hỗ trợ giám sát đa cụm k8s với thao tác dễ dàng

## Những cách cài đặt Rancher ##
- Docker và Docker Compose Độc lập
- Chạy chung với cụm k8s
- Chạy bằng Heim

Trong bài giảng này chúng ta sẽ sử dụng Docker 

### Bước 1: tạo server Rancher
- rancher không sử dụng quá nhiều tài nguyên nên 2gb ram + 1 cpu là đủ
- Tuy nhiên để thuận lợi cho việc backup chúng ta sẽ mount thêm 1 đĩa 20gb từ Host vào server. Cách này được sử dụng rât phổ biển trong doanh nghiệp
- Ở trong môi trường on-premise, cloud, hay private cloud cũng hỗ trợ mount thêm ổ đĩa nhưng sẽ khác giao diện.

### Bước 2: Gán ổ đĩa vào rancher
```bash
lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0                         2:0    1    4K  0 disk
loop0                       7:0    0 63.7M  1 loop /snap/core20/2434
loop1                       7:1    0 91.9M  1 loop /snap/lxd/24061
loop2                       7:2    0 63.3M  1 loop /snap/core20/1828
loop3                       7:3    0 91.9M  1 loop /snap/lxd/29619
loop4                       7:4    0 38.8M  1 loop /snap/snapd/21759
loop5                       7:5    0 49.9M  1 loop /snap/snapd/18357
sda                         8:0    0   20G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1.8G  0 part /boot
└─sda3                      8:3    0 18.2G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0 18.2G  0 lvm  /
sdb                         8:16   0   20G  0 disk
sr0                        11:0    1 1024M  0 rom
sr1                        11:1    1 1024M  0 rom
```
- nhìn vào cột mountpoint của sda chúng ta thấy nó đã được gán vào root
- tuy nhiên nhìn vào sdb chúng ta thấy nó chưa được mount vào thư mục nào trong server
- Cần Format và gán ổ vào server

```bash
mkfs.ext4 -m 0 /dev/sdb
```
- Format theo định dạng ext4 100% cho ổ /dev/sdb tạo ra 1 phân vùng duy nhất

```bash
mkdir /data
echo "/dev/sdb  /data  ext4  defaults  0  0" | sudo tee -a /etc/fstab
mount -a
sudo df -h

/dev/sdb                            20G   32K   20G   1% /data
```
- tạo thư mục data
- echo dòng lệnh vào tee -a và lưu trong /etc/fstab
- lệnh mount sẽ mount ổ sdb vào data
- df -h để kiểm tra

### Bước 3: Cài đặt docker
```bash
apt update
apt install docker.io
apt install docker-compose
```
Cài đặt Docker và Docker-Compose
### Bước 4: Version của rancher
Lưu ý quan trọng: Phải kiểm tra version của Rancher và K8S có tương thích không
search keyword: "Rancher Version Matrix"

### Bước 5: Cài đặt rancher bằng docker
```bash
mkdir /data/rancher
cd /data/rancher
vi docker-compose.yml
```
Tạo thư mục rancher trong thư mục data vừa tạo và viết docker-compose để cài đặt và chạy rancher trong container

```yaml
version: '3'
services:
  rancher-server:
    image: rancher/rancher:v2.9.2
    container_name: rancher-server
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /data/rancher/data:/var/lib/rancher
    privileged: true
```
- Tải Rancher version 2.9.2
- listen và map port 80 và 443 của server vào rancher
- mount thư mục trong container ra thư mục của server -> Thư mục của server này mount ra OS

```bash
docker-compose up -d
```

### Bước 6: Truy cập vào Rancher
https://192.168.197.44
![image](https://github.com/user-attachments/assets/00beba18-2087-44cb-9f9b-cade576b37ff)

```bash
docker logs  rancher-server  2>&1 | grep "Bootstrap Password:"

2024/11/26 13:48:47 [INFO] Bootstrap Password: 8k6ltb92p574zd297fv8ns9dnqwrbm7hpdsgb5cw4g46sr26cwdb9z
```
dùng lệnh trên để lấy random bootstrap password

Thiết lập mật khẩu mới: garrett210303
để nguyên server url

### Bước 7: Cấu hình truy cập Rancher bằng domain
add host trong windows để truy cập bằng domain cho chuyên nghiệp
C:\Windows\System32\drivers\etc
rancher.garrett.vn
### Bước 8: kết nối k8s cluster trên Rancher
Add Exsiting -> Generic -> đặt tên -> chạy dòng lệnh cho chứng chỉ tự ký 

Trong server 1 - user devops
```bash
curl --insecure -sfL https://192.168.197.44/v3/import/8gs6ft4pmccn7mj4flqkp8skpb9gl5x6qd6rwlprdddlgqncf9prbb_c-m-mr5qplt5.yaml | kubectl apply -f -
```
các ns, service và secret sẽ dược tạo bằng việc apply file yaml từ rancher-server

```bash
# hiển thị các pod đang chạy trong ns cattle-system
kubectl get pods -n cattle-system

# Dùng ID để hiển thị chi tiết 1 pod trong 1 ns
kubectl describe pod -n cattle-system cattle-cluster-agent-748dc876f-zkdqh

# Hiển thị log của 1 app
kubectl logs -n cattle-system -l app=cattle-cluster-agent
```
Để cho dễ hiểu thì Rancher sẽ khởi tạo 1 app chứa các ns và pod trên cụm để thực hiện giám sát cho phép ta thiết lập

![image](https://github.com/user-attachments/assets/c5d16244-b981-4d87-84f4-4fdbb665b57c)
