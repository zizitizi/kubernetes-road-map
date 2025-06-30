 Ceph چیست؟
💡 خلاصه ساده
Ceph یک distributed storage system است.

بسیار مقیاس‌پذیر، self-healing و fault-tolerant.

می‌تواند block, object, و file storage ارائه دهد.

در Kubernetes، به‌صورت یک layer به‌عنوان Persistent Volume استفاده می‌شود.

💡 Rook چیست؟
یک Operator برای مدیریت Ceph در Kubernetes.

کارهای پیچیده Ceph مثل cluster setup, disk prepare, pool create, monitor health را خودش اتوماتیک انجام می‌دهد.

✅ 🌟 مراحل نصب Rook-Ceph (با توضیح دقیق)
🟢 1️⃣ نصب CRDها و common resourceها
bash
Copy
Edit
kubectl apply -f https://raw.githubusercontent.com/rook/rook/release-1.14/deploy/examples/crds.yaml
kubectl apply -f https://raw.githubusercontent.com/rook/rook/release-1.14/deploy/examples/common.yaml
✅ این دستورها Custom Resource Definitions (CRDs) و Role/RoleBinding ها را deploy می‌کند.

🟢 2️⃣ نصب Rook Operator
bash
Copy
Edit
kubectl apply -f https://raw.githubusercontent.com/rook/rook/release-1.14/deploy/examples/operator.yaml
✅ Operator مسئول مدیریت lifecycle کل Ceph cluster.

🟢 3️⃣ تعریف CephCluster
bash
Copy
Edit
kubectl apply -f https://raw.githubusercontent.com/rook/rook/release-1.14/deploy/examples/cluster.yaml
✅ این yaml مشخص می‌کند Ceph کجا، با چه تعداد OSD و چه تنظیماتی ساخته شود.

🟢 4️⃣ تعریف StorageClass
bash
Copy
Edit

kubectl apply -f  https://raw.githubusercontent.com/rook/rook/release-1.14/deploy/examples/csi/rbd/storageclass.yaml

✅ این StorageClass توسط PVCها برای dynamic provisioning استفاده می‌شود.

https://github.com/rook/rook/tree/release-1.14/deploy/examples



🟢 5️⃣ تست PVC
yaml
Copy
Edit
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-rook-ceph-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: rook-ceph-block
bash
Copy
Edit
kubectl apply -f pvc.yaml
✅ بعد از ساخت PVC، می‌توانی با هر Pod یا Deployment از این storage استفاده کنی.

