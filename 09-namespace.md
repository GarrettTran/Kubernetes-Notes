# NAMESPACE TRONG K8S
- Namespace trong k8s giúp ta phân các dự án và phân quyền trong chính cluster đó.
- Phù hợp với các doanh nghiệp quy mô nhỏ có giới hạn số lượng các cụm nhưng vẫn đảm bảo tình bảo mật cao và chi tài nguyên cho từng dự án phù hợp

Các câu lệnh cơ bản về namespace trong k8s:

```bash
kubectl get ns
kubectl get pod
```
- get pod không chỉ định ns thì sẽ mặc định là default namespace
- get ns sẽ liệt kê ra các ns khả dụng trong cụm


```bash
kubectl create ns project-1
kubectl delete ns project-1
```
- Tạo hoặc xóa 1 namespace bằng câu lệnh
- Cách này không được khuyến khích vì nó dễ dẫn đến những thiết lập sai 
Và đồng thời khó khôi phục cụm cũng như không hỗ trợ thiết lập chuyên sâu
- Thay vào đó chúng ta nên sử dụng file yaml namespace.yaml

### Viết file namespace.yaml và resourcequota.yaml để quản lý tài nguyên tối đa của namespace ###
vi ns.yaml
```bash
apiVersion: v1
kind: namespace
metadata:
  name: project-1
```

vi ns.resourcequota.yaml
```bash
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-quota
  namespace: project-1
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
```

```bash
kubectl apply -f ns.yaml
kubectl apply -f resourcequota.yaml
```
- Apply cấu hình khởi tạo namespace và resource
