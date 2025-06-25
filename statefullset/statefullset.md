

# Database On Kubernetes with statefullset

2 replica should have 2 sparate and isolated persistent volume with node affinity . in my sample node zizi2 with pv affinity to pv-postgres-disk-0 and zizi3 with affinity pv-postgres-disk-1

you should do this before hand. in node zizi2:

 sudo mkdir -p /mnt/data/postgres-disk-0

sudo chown 1000:1000 /mnt/data/postgres-disk-0

in node zizi3:

sudo mkdir -p /mnt/data/postgres-disk-1

sudo chown 1000:1000  /mnt/data/postgres-disk-1


### 1- statefulset database file

vi statefulset.yml

 
 apiVersion: apps/v1
 kind: StatefulSet
 metadata:
 name: postgres-database
 spec:
 selector:
   matchLabels:
     app: postgres-database
 serviceName: postgres-service
 replicas: 2
 template:
   metadata:
     labels:
       app: postgres-database
   spec:
     containers:
       - name: postgres-database
         image: postgres:15
         volumeMounts:
           - name: postgres-disk
             mountPath: /var/lib/postgresql/data
         env:
           - name: POSTGRES_PASSWORD
             value: zizi
           - name: PGDATA
             value: /var/lib/postgresql/data/pgdata
 volumeClaimTemplates:
   - metadata:
       name: postgres-disk
     spec:
       accessModes: [ "ReadWriteOnce" ]
       storageClassName: local-storage
       resources:
         requests:
           storage: 5Gi
 



### 2- headless service 

 vi headlessservice.yml


  # PostgreSQL StatefulSet Service
  apiVersion: v1
  kind: Service
  metadata:
    name: postgres-loadbalancer
  spec:
    selector:
      app: postgres-database
    type: LoadBalancer
    ports:
      - port: 5432
        targetPort: 5432




### 3- storage class

vi storageclass.yml

  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
    name: local-storage
  provisioner: kubernetes.io/no-provisioner
  volumeBindingMode: WaitForFirstConsumer




### 4- sparate pv for 2 replica

 vi 2pv.yml

  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: pv-postgres-disk-0
  spec:
    capacity:
      storage: 5Gi
    volumeMode: Filesystem
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Retain
    storageClassName: local-storage
    local:
      path: /mnt/data/postgres-disk-0
    nodeAffinity:
      required:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: In
            values:
            - zizi2
  ---
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: pv-postgres-disk-1
  spec:
    capacity:
      storage: 5Gi
    volumeMode: Filesystem
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Retain
    storageClassName: local-storage
    local:
      path: /mnt/data/postgres-disk-1
    nodeAffinity:
      required:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: In
            values:
            - zizi3
  




after all above done you can run:

 kubectl apply -f .

## imortant note:

If you have run StatefulSet before setting the StorageClass correctly, the PVCs are created with incorrect settings and even updating the StatefulSet does not change the existing PVCs.
you should delete statefullset and pvc manually. after create pv correctly  and run it again 













refrence:

https://chetak.hashnode.dev/database-on-kubernetes#heading-volume-in-k8s


