# 12/3/2024 Mất điện lúc 10h tối dẫn tới mất dữ liệu không thể khôi phục trên Rancher

## Nguyên nhân
- Các máy tính khi tắt đột ngột sẽ làm hỏng dữ liệu không thể khôi phục được trên db
- Khi mở db sẽ thấy rất nhiều trường null
- thông thường thì k3s sẽ chịu trách nhiệm tạo ra các backup cho db
- Có thể trong quá trình lỗi các db được ghi trên server cũng bị mất dữ liệu

## Chi tiết
- Rancher sử dụng etcd và BBolt là 1 db low level
- Mặc dù đã có backup mount thư mục làm việc từ trong container ra vm
- Tuy nhiên lần này là sập luôn cả vm và host
## Nguyên nhân có backup nhưng vẫn không khôi phục được
- K3s tự động quản lý tần suất backup và số lượng snapshot thông qua các tham số cấu hình (--etcd-snapshot-schedule-cron, --etcd-snapshot-retention).
- Mặc định, K3s sẽ tạo snapshot sau 100000 transactions và lưu vào 5 bản.
- No Time-Based Schedule: Time-based scheduling (e.g., hourly snapshots) is not enabled by default.
## Giải pháp cuối cùng
- xóa toàn bộ member trong database và khởi động lại 1 rancher mới hoàn toàn
- Tốn khá nhiều thời gian vì hi vọng với cái file db dc mount ra từ container đó sẽ giúp cứu được rancher 
## Cách để điều này không bao giờ lập lại 
- Lưu backup từ trong vm ra 1 nơi khác như S3 bucket hay datacenter khác
- Rancher có cả 1 bộ tài liệu về backup
- Và điều này áp dụng với mọi công cụ
- Trang doc chính thức cung cấp best practice và insight tốt nhất vì họ hiểu công cụ của họ

```yaml
#!/bin/bash

set -e

if [ ! -e /run/secrets/kubernetes.io/serviceaccount ] && [ ! -e /dev/kmsg ]; then
    echo "ERROR: Rancher must be ran with the --privileged flag when running outside of Kubernetes"
    exit 1
fi

#########################################################################################################################################
# DISCLAIMER                                                                                                                            #
# Copied from https://github.com/moby/moby/blob/ed89041433a031cafc0a0f19cfe573c31688d377/hack/dind#L28-L37                              #
# Permission granted by Akihiro Suda <akihiro.suda.cz@hco.ntt.co.jp> (https://github.com/rancher/k3d/issues/493#issuecomment-827405962) #
# Moby License Apache 2.0: https://github.com/moby/moby/blob/ed89041433a031cafc0a0f19cfe573c31688d377/LICENSE                           #
#########################################################################################################################################
# only run this if rancher is not running in kubernetes cluster
if [ ! -e /run/secrets/kubernetes.io/serviceaccount ] && [ -f /sys/fs/cgroup/cgroup.controllers ]; then
  # move the processes from the root group to the /init group,
  # otherwise writing subtree_control fails with EBUSY.
  mkdir -p /sys/fs/cgroup/init
  xargs -rn1 < /sys/fs/cgroup/cgroup.procs > /sys/fs/cgroup/init/cgroup.procs || :
  # enable controllers
  sed -e 's/ / +/g' -e 's/^/+/' <"/sys/fs/cgroup/cgroup.controllers" >"/sys/fs/cgroup/cgroup.subtree_control"
fi

rm -f /var/lib/rancher/k3s/server/cred/node-passwd
if [ -e /var/lib/rancher/management-state/etcd ] && [ ! -e /var/lib/rancher/k3s/server/db/etcd ]; then
  mkdir -p /var/lib/rancher/k3s/server/db
  ln -sf /var/lib/rancher/management-state/etcd /var/lib/rancher/k3s/server/db/etcd
  echo -n 'default' > /var/lib/rancher/k3s/server/db/etcd/name
fi
if [ -e /var/lib/rancher/k3s/server/db/etcd ]; then
  echo "INFO: Running k3s server --cluster-init --cluster-reset"
  set +e
  k3s server --cluster-init --cluster-reset &> ./k3s-cluster-reset.log
  K3S_CR_CODE=$?
  if [ "${K3S_CR_CODE}" -ne 0 ]; then
    echo "ERROR:" && cat ./k3s-cluster-reset.log
    rm -f /var/lib/rancher/k3s/server/db/reset-flag
    exit ${K3S_CR_CODE}
  fi
  set -e
fi
if [ -x "$(command -v update-ca-certificates)" ]; then
  update-ca-certificates
fi
if [ -x "$(command -v c_rehash)" ]; then
  c_rehash
fi
exec tini -- rancher --http-listen-port=80 --https-listen-port=443 --audit-log-path=${AUDIT_LOG_PATH} --audit-level=${AUDIT_LEVEL} --audit-log-maxage=${AUDIT_LOG_MAXAGE} --audit-log-maxbackup=${AUDIT_LOG_MAXBACKUP} --audit-log-maxsize=${AUDIT_LOG_MAXSIZE} "${@}"
```
- đã tìm được file khởi chạy entrypint.sh của rancher

```yaml
advertise-client-urls: https://172.18.0.2:2379
client-transport-security:
  cert-file: /var/lib/rancher/k3s/server/tls/etcd/server-client.crt
  client-cert-auth: true
  key-file: /var/lib/rancher/k3s/server/tls/etcd/server-client.key
  trusted-ca-file: /var/lib/rancher/k3s/server/tls/etcd/server-ca.crt
data-dir: /var/lib/rancher/k3s/server/db/etcd
election-timeout: 5000
experimental-initial-corrupt-check: true
experimental-watch-progress-notify-interval: 5000000000
heartbeat-interval: 500
listen-client-http-urls: https://127.0.0.1:2382
listen-client-urls: https://127.0.0.1:2379,https://172.18.0.2:2379
listen-metrics-urls: http://127.0.0.1:2381
listen-peer-urls: https://127.0.0.1:2380,https://172.18.0.2:2380
log-outputs:
- stderr
logger: zap
name: 177d114e325a-34eb565b
peer-transport-security:
  cert-file: /var/lib/rancher/k3s/server/tls/etcd/peer-server-client.crt
  client-cert-auth: true
  key-file: /var/lib/rancher/k3s/server/tls/etcd/peer-server-client.key
  trusted-ca-file: /var/lib/rancher/k3s/server/tls/etcd/peer-ca.crt
snapshot-count: 10000
```
- và file config của k3s
