
in this section i disscuss about my step by step senario:

this senario design is something like this:

    
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
    



i have 7 node :


    final table with 7 node:
    
    VM	role	IP 
    VM1 (.40)	Master-1	192.168.144.40
    VM2 (.41)	Master-2	192.168.144.41
    VM3 (.42)	Master-3	192.168.144.42
    VM4 (.43)	Worker-1 (+Ceph)	192.168.144.43
    VM5 (.44)	Worker-2 (+Ceph)	192.168.144.44
    VM6 (.45)	Worker-3 (+Ceph)	192.168.144.45
    VM7 (.46)	Load Balancer + Monitoring	192.168.144.46
    


















