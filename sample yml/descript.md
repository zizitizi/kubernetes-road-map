




### 1- single simple pod
nano niginxpod.yml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: webservers
  labels:
    app: monitoringapp
    version: "1.0"
    zone: test
spec:
  containers:
     - name: nginx-ctr
       image: nginx
       ports:
          - containerPort: 80

---
apiVersion: v1
kind: Namespace
metadata:
  name: webservers
  labels:
    environment: test

kubectl apply -f niginxpod.yml















