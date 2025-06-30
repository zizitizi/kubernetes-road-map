

my zizi5 node cant pull k8s image. i pull it from china server manulaay on that node

in master
 kubectl -n kube-system get ds kube-proxy -o=jsonpath='{.spec.template.spec.containers[0].image}'
registry.k8s.io/kube-proxy:v1.27.4


my kubelte version:

registry.k8s.io/kube-proxy:v1.27.4

   ctr --namespace k8s.io images pull registry.aliyuncs.com/google_containers/kube-proxy:v1.27.4
   
   
   ctr --namespace k8s.io images tag registry.aliyuncs.com/google_containers/kube-proxy:v1.27.4 registry.k8s.io/kube-proxy:v1.27.4
   
   
   systemctl restart kubelet



inial cluster command with kube flannel with HA (multi master with 3) in first master:

   
   kubeadm init \
     --control-plane-endpoint "192.168.144.46:6443" \
     --upload-certs \
     --pod-network-cidr=192.168.0.0/16
   
     



join worker command:

   kubeadm token create --print-join-command


join master command in master:

   kubeadm token create --print-join-command
   
   kubeadm init phase upload-certs --upload-certs


on a node candidate to master :

   kubeadm join 192.168.144.40:6443 --token abcdef.0123456789abcdef \
       --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxx \
       --control-plane --certificate-key 3c1cde8d0b6f3e321ffda201f6c3be8360cd3b24854726b882f5b48b3e06494c


    










