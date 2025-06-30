


kubeadm init \
  --control-plane-endpoint "192.168.144.46:6443" \
  --upload-certs \
  --pod-network-cidr=192.168.0.0/16


kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml




















