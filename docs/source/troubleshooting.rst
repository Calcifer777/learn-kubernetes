
###################
Troubleshooting
###################

***************
Application
***************

- Services
  - status
  - port mappings
  - selectors
- Deployments
  - references to dependencies 

***************************
Controlplane components
***************************

- Nodes
- Controlplane pods
  - status
  - logs
- Controlplane services
  - kube-apiserver
  - kube-controller-manager
  - kube-scheduler
  - kubelet
  - kube-proxy
- Service logs
  - `sudo journalclt -u kubeapiserver`

***************
Worker nodes
***************

- Node
  - node status
  - `df -h`
- kubelet
  - `service kubelet status`
  - `sudo journalctl -u kubelet`
- Certificates

***************
Networking
***************