

my zizi5 node cant pull k8s image. i pull it from china server manulaay on that node

in master
 kubectl -n kube-system get ds kube-proxy -o=jsonpath='{.spec.template.spec.containers[0].image}'
registry.k8s.io/kube-proxy:v1.27.4


my kubelte version:

registry.k8s.io/kube-proxy:v1.27.4

ctr --namespace k8s.io images pull registry.aliyuncs.com/google_containers/kube-proxy:v1.27.4


ctr --namespace k8s.io images tag registry.aliyuncs.com/google_containers/kube-proxy:v1.27.4 registry.k8s.io/kube-proxy:v1.27.4


systemctl restart kubelet





