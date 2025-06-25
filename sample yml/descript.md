


## how to use sample code

in all sample just copy codes in yml file save it and :
kubectl apply -f thatfile.yml


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


### 2- replica controller sample


    apiVersion: v1
    kind: ReplicationController
    metadata:
         name: nginx-rc
    spec:
       replicas: 4
       selector:
          app: websrv
       template:
         metadata:
            labels:
               app: websrv
         spec:
            containers:
               - name: nginx-ctr
                 image: nginx:latest
                 ports:
                   - containerPort: 80



### 3- deployment sample

    apiVersion: apps/v1
    kind: Deployment
    metadata:
         name: nginx-deploy
         labels:
           app: lb
    spec:
       replicas: 6
       selector:
         matchLabels:
           app: websrv
           version: v1
       minReadySeconds: 10
       strategy:
         type: RollingUpdate
         rollingUpdate:
            maxUnavailable: 1
            maxSurge: 1
       template:
         metadata:
            labels:
               app: websrv
               version: v1
         spec:
            containers:
               - name: nginx-ctr
                 image: nginx:latest
                 ports:
                   - containerPort: 80


### 4- daemonset sample

    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
         name: nginx-ds
         labels:
           app: lb
    spec:
       selector:
         matchLabels:
           app: websrv
           version: v1
       template:
         metadata:
            labels:
               app: websrv
               version: v1
         spec:
            containers:
               - name: nginx-ctr
                 image: nginx:latest
                 ports:
                   - containerPort: 80


### 5- statefullset sample


use belove link:

https://github.com/zizitizi/kubernetes-road-map/blob/main/statefullset/statefullset.md



### 6- Service node port

nano nginx-deploy-svc.yml 


    ---
    apiVersion: v1
    kind: Namespace
    metadata:
      name: web-server
    ---
    
    apiVersion: apps/v1
    kind: Deployment
    metadata:
         name: nginx-deploy
         namespace: web-server
         labels:
           app: websrv
    spec:
       replicas: 3
       selector:
         matchLabels:
           app: websrv
           version: v1
       minReadySeconds: 10
       strategy:
         type: RollingUpdate
         rollingUpdate:
            maxUnavailable: 1
            maxSurge: 1
       template:
         metadata:
            labels:
               app: websrv
               version: v1
         spec:
            containers:
               - name: nginx-ctr
                 image: nginx:latest
                 ports:
                   - containerPort: 80
    
    ---
    
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-svc
      namespace: web-server
      labels:
        app: websrv
    spec:
      type: NodePort
      ports:
        - targetPort: 80
          port: 8080
          nodePort: 30008
      selector:
         app: websrv


now you can see nginx page from your node external browser with masternodeip:30008 . in my example:
http://192.168.144.41:30008/



### 7- volume - container runtime mount volume


    
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-pod
      labels:
         app: nginx
    spec:
      containers:
      - image: nginx
        name: test-container
        volumeMounts:
        - mountPath: /usr/share/nginx/html/
          name: html-dir
        - mountPath: /var/log/nginx/
          name: log-dir
    
      volumes:
      - name: html-dir
        hostPath:
          path: /root/k8s/html/
      - name: log-dir
        hostPath:
          path: /root/k8s/log/


in that worker node that pods runing make sure of existance of  entire directory path to html and log file.

to test 

echo "hello from hostpath" > html/index.html

 curl 10.244.2.14
hello from hostpath


### 8- volume - persistent volume in k8s sample


##### pv sample



##### pvc sample



##### asign pvc to a deployment





























