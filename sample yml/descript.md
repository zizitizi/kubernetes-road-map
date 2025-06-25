


## in all sample just copy codes in yml file save it and :

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


### 5- 

















