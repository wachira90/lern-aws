แน่นอน! ต่อไปนี้คือตัวอย่างการใช้งาน Kubernetes บน **Amazon EKS (Elastic Kubernetes Service)** โดยแสดงบทบาทของแต่ละองค์ประกอบที่คุณกล่าวถึง ได้แก่ `kube-proxy`, `CoreDNS`, `AWS CNI`, `EBS`, และ `EFS` พร้อมตัวอย่างที่เข้าใจง่าย:

---

## 🎯 สถานการณ์สมมติ

คุณต้องการดีพลอยแอปพลิเคชัน web (เช่น Nginx) ที่:

* รองรับ DNS ภายในคลัสเตอร์ (CoreDNS)
* สามารถเข้าถึงผ่าน ClusterIP ได้ (kube-proxy)
* ใช้ Pod networking จาก AWS VPC (AWS CNI)
* ใช้ EBS สำหรับ storage แบบ block (เช่นฐานข้อมูล)
* ใช้ EFS สำหรับ shared file storage (เช่นอัปโหลดไฟล์)

---

## ✅ 1. **kube-proxy** – Load balancing service traffic

`kube-proxy` คือ component ที่ทำหน้าที่ route ทราฟฟิกในคลัสเตอร์ ไปยัง service หรือ pod ที่เหมาะสม โดยใช้ iptables หรือ ipvs

### 🔹 ตัวอย่าง Service (ClusterIP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

เมื่อ user ภายใน cluster access `nginx-service`, kube-proxy จะ route ไปยัง Pod ที่ match label `app: nginx`

---

## ✅ 2. **CoreDNS** – Cluster DNS Resolution

CoreDNS ช่วยให้ Pod ต่างๆ สามารถ resolve ชื่อกันภายใน cluster ได้ เช่น `nginx-service.default.svc.cluster.local`

### 🔹 ตัวอย่าง Pod ใช้งาน DNS

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: curl
spec:
  containers:
    - name: curl
      image: curlimages/curl
      command: ["sleep", "3600"]
```

```bash
# Exec เข้า pod
kubectl exec -it curl -- /bin/sh

# ทดสอบ DNS
curl nginx-service.default.svc.cluster.local
```

---

## ✅ 3. **AWS CNI (Container Network Interface)** – VPC Networking

AWS CNI ช่วยให้แต่ละ Pod ได้รับ **IP จาก VPC subnet โดยตรง** ทำให้ pod สื่อสารกับ resource ภายนอกได้สะดวก

```bash
# ดู IP pod ที่ได้จาก VPC
kubectl get pods -o wide
```

Pod จะได้ IP เช่น `10.0.1.x` ซึ่งอยู่ใน VPC subnet

---

## ✅ 4. **EBS (Elastic Block Store)** – ใช้กับ Stateful Applications

### 🔹 สร้าง PVC + EBS StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-claim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ebs-sc
  resources:
    requests:
      storage: 5Gi
```

### 🔹 ใช้งานใน Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ebs-pod
spec:
  containers:
    - name: app
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - mountPath: "/data"
          name: ebs-volume
  volumes:
    - name: ebs-volume
      persistentVolumeClaim:
        claimName: ebs-claim
```

---

## ✅ 5. **EFS (Elastic File System)** – Shared File Storage

### 🔹 สร้าง StorageClass สำหรับ EFS

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
```

### 🔹 PVC สำหรับ EFS

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: efs-claim
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 5Gi
```

### 🔹 Pod ใช้ EFS

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: efs-pod
spec:
  containers:
    - name: app
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - name: efs-volume
          mountPath: "/shared-data"
  volumes:
    - name: efs-volume
      persistentVolumeClaim:
        claimName: efs-claim
```

---

## 🔍 สรุปหน้าที่ของแต่ละคอมโพเนนต์

| คอมโพเนนต์     | หน้าที่                                     |
| -------------- | ------------------------------------------- |
| **kube-proxy** | Route traffic ไปยัง service/backend pod     |
| **CoreDNS**    | Resolve DNS ภายในคลัสเตอร์                  |
| **AWS CNI**    | Assign IP จาก VPC subnet ให้กับ pod         |
| **EBS**        | Storage แบบ Block สำหรับแต่ละ pod           |
| **EFS**        | Shared storage ที่ Pod หลายตัวใช้ร่วมกันได้ |

---

หากคุณต้องการ **Terraform/IaC** สำหรับ setup EKS + addons เหล่านี้ หรืออยากดู **แผนภาพการทำงาน**, บอกได้เลย!
