

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








#### these codes works for me . test it!

for test:

 kubectl exec -it postgres-database-0 -- psql -U postgres

 CREATE TABLE test_table (id SERIAL PRIMARY KEY, data TEXT);
INSERT INTO test_table (data) VALUES ('hello from pod 0');

kubectl delete pod postgres-database-0


now check data agin


kubectl exec -it postgres-database-0 -- psql -U postgres -c "SELECT * FROM test_table;"


By default, the PostgreSQL StatefulSet database does not automatically replicate or synchronize data between pods.

Each database pod is independent and its data and tables are not synchronized with each other unless you implement replication settings (such as Streaming Replication) yourself.

Here's the step-by-step guide for setting up PostgreSQL Streaming Replication on Kubernetes:

Step 1: Prepare the Primary (Master) Pod
Enter the primary pod:

bash
kubectl exec -it postgres-database-0 -- bash
Open PostgreSQL interactive terminal:

bash
psql -U postgres
Create a replication user:

sql
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'Welcome1';
Edit the PostgreSQL configuration file (postgresql.conf):

Locate the file (usually /var/lib/postgresql/data/postgresql.conf) and add or modify these parameters:

text
wal_level = replica
max_wal_senders = 10
wal_keep_size = 64MB
Edit pg_hba.conf to allow replication connections:

Open /var/lib/postgresql/data/pg_hba.conf and add:

text
host replication replicator 0.0.0.0/0 md5
Restart PostgreSQL:

bash
pg_ctl restart -D /var/lib/postgresql/data
Or simply delete and recreate the pod to apply changes.

Step 2: Prepare the Standby (Replica) Pod
Enter the standby pod:

bash
kubectl exec -it postgres-database-1 -- bash
Clear existing data directory (if any):

bash
rm -rf /var/lib/postgresql/data/*
Take a base backup from the primary:

bash
pg_basebackup -h <Primary_IP_or_Service> -D /var/lib/postgresql/data -U replicator -v -P --wal-method=stream
Replace <Primary_IP_or_Service> with the service name or IP of the primary pod (e.g., postgres-database-0.postgres-service.default.svc.cluster.local).

Enter password Welcome1 when prompted.

Create the standby.signal file to enable standby mode:

bash
touch /var/lib/postgresql/data/standby.signal
Configure connection to primary:

Edit postgresql.conf (or create postgresql.auto.conf) and add:

text
primary_conninfo = 'host=<Primary_IP_or_Service> port=5432 user=replicator password=Welcome1 sslmode=prefer'
Start PostgreSQL:

bash
pg_ctl start -D /var/lib/postgresql/data
Step 3: Test Replication
Create a test table and insert data on primary:

bash
kubectl exec -it postgres-database-0 -- psql -U postgres -c "CREATE TABLE test_table(id SERIAL PRIMARY KEY, data TEXT); INSERT INTO test_table(data) VALUES ('hello replication');"
Check data on standby:

bash
kubectl exec -it postgres-database-1 -- psql -U postgres -c "SELECT * FROM test_table;"
If replication is working, the inserted data will appear on the standby pod.

Important Notes
The primary pod's IP or service must be reachable from the standby pod.

For production environments, consider using PostgreSQL operators like CrunchyData or Zalando for automated management.

Additional security configurations (e.g., SSL/TLS) are recommended for production.


kubectl exec -it postgres-database-1 -- psql -U postgres -c "SELECT * FROM test_table;"






refrence:

https://chetak.hashnode.dev/database-on-kubernetes#heading-volume-in-k8s


