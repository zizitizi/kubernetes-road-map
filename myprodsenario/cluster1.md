
install kuber on all 7 node:

    
                 ┌──────────────┐
                 │ LoadBalancer │ ← VM7 (.46)
                 └──────┬───────┘
                        │
     ┌─────────┬────────┼─────────┬─────────┐
     │         │        │         │
     │   M1    │   M2   │   M3    │
     │ .40     │ .41    │ .42     │
     │         │        │         │
     └─────┬───┴────────┴───┬─────┘
           │                │
     ┌─────▼────────────────▼───────────────┐
     │             Kubernetes Network       │
     │         (Calico / Cilium / ...)      │
     │                                       │
     │ ┌────────┐ ┌────────┐ ┌────────┐     │
     │ │ W1 (.43)│ │ W2 (.44)│ │ W3 (.45)│     │
     │ │ + Ceph │ │ + Ceph │ │ + Ceph │     │
     │ └────────┘ └────────┘ └────────┘     │
     │                                       │
     │ Optional: Monitoring (Prometheus, Grafana) on VM7 │
     └───────────────────────────────────────┘
    
    


in node 7 as zizi6 as loadbalancer:

apt update && apt install -y haproxy

nano /etc/haproxy/haproxy.cfg

global
    log /dev/log local0
    log /dev/log local1 notice
    daemon

defaults
    log     global
    mode    tcp
    option  tcplog
    timeout connect 10s
    timeout client  1m
    timeout server  1m

frontend k8s_api_front
    bind *:6443
    default_backend k8s_api_backend

backend k8s_api_backend
    balance roundrobin
    server master1 192.168.144.40:6443 check
    server master2 192.168.144.41:6443 check
    server master3 192.168.144.42:6443 check



systemctl restart haproxy
systemctl enable haproxy


on master1:

kubeadm init \
  --control-plane-endpoint "192.168.144.46:6443" \
  --upload-certs \
  --pod-network-cidr=192.168.0.0/16


kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml




















