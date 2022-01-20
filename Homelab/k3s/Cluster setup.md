# HA Cluster setup

##  Installation with `k3sup`

Initially tried with this, worked but preferred doing it manually as it makes you understand things more.

### Initial master setup

```bash
export USER=<user used to ssh into remotes>
export SERVER_IP=<ip address of first server>

k3sup install \
  --ip $SERVER_IP \
  --user $USER \
  --cluster
  --k3s-extra-args '--node-taint CriticalAddonsOnly=true:NoExecute --tls-san <external load balancer IP>'
```

### Other master nodes

```bash
export NEXT_SERVER_IP=<ip address>

k3sup join \
  --ip $NEXT_SERVER_IP \
  --user $USER \
  --server-user $USER \
  --server-ip $SERVER_IP \
  --server
  --k3s-extra-args '--node-taint CriticalAddonsOnly=true:NoExecute --tls-san <external load balancer IP>'
```

### Agent nodes

```bash
export AGENT_IP=<ip address>

k3sup join \
  --ip $AGENT_IP \
  --server-ip $SERVER_IP \
  --user $USER
```

## Without `k3sup` or manually

This requires an external loadbalancer

### Initial master

```bash
curl -sfL https://get.k3s.io | K3S_TOKEN="xxx" sh -s - server --cluster-init --disable servicelb --tls-san cluster.example.com --node-taint CriticalAddonsOnly=true:NoExecute
```

### Other masters

```bash
curl -sfL https://get.k3s.io | K3S_TOKEN="xxx" sh -s - server --server https://<ip or hostname of initial master node>:6443 --no-deploy servicelb --tls-san cluster.example.com --node-taint CriticalAddonsOnly=true:NoExecute
```

### Agents

```bash
curl -sfL https://get.k3s.io | K3S_URL="https://<ip or hostname of initial master node>:6443" K3S_TOKEN="xxx" sh -
```

## Installation with `kube-vip` as loadbalancer

Almost all below commands are executed from the inital master node

```bash
# Install k3s on initial master node
curl -sfL https://get.k3s.io | K3S_TOKEN="xxx" sh -s - server --cluster-init --disable traefik --disable servicelb --tls-san cluster.example.com --node-taint CriticalAddonsOnly=true:NoExecute

# Get RBAC config for required permissions
curl https://kube-vip.io/manifests/rbac.yaml > rbac.yaml
sudo cp rbac.yaml /var/lib/rancher/k3s/server/manifests/

# Export kube-vip configuration
export VIP=<ip>
export INTERFACE=<interface>
KVVERSION=$(curl -sL https://api.github.com/repos/kube-vip/kube-vip/releases | jq -r ".[0].name")

# Create alias to run the kube-vip container once to create a config
alias kube-vip="sudo ctr run --rm --net-host ghcr.io/kube-vip/kube-vip:$KVVERSION vip /kube-vip"

# Manually pull the kube-vip container (don't ask why)
sudo ctr image pull ghcr.io/kube-vip/kube-vip:v0.4.1

# Run the container to get a config and appand that into the kube-vip .yaml
kube-vip manifest daemonset \
    --interface $INTERFACE \
    --address $VIP \
    --inCluster \
    --taint \
    --controlplane \
    --services \
    --arp \
    --leaderElection >> kube-vip.yaml

sudo cp kube-vip.yaml /var/lib/rancher/k3s/server/manifests/

# At this point the VIP should be up, verify by pinging:
ping VIP

# If the VIP is not up, check the logs of the pod
sudo kubectl get pods --all-namespaces
sudo kubectl logs -n kube-system kube-vip-<id>

# Add the other master nodes
curl -sfL https://get.k3s.io | K3S_TOKEN="nnFEDhSAcKKeusXhSdnp" sh -s - server --server <ip or hostname of initial master node>:6443 --disable traefik --disable servicelb --tls-san cluster.example.com --node-taint CriticalAddonsOnly=true:NoExecute

# Add the worker nodes
curl -sfL https://get.k3s.io | K3S_URL="<ip or hostname of VIP ip>:6443" K3S_TOKEN="xxx" sh -

# Verify that all nodes are present
sudo kubectl get nodes

# Verify that the kube-vip pod is scheduled on all master nodes
sudo kubectl get pods --all-namespaces

# Get the kubectl config to paste into dev machine
sudo cat /etc/rancher/k3s/k3s.yaml

# Finally add that to .kube/config and edit the server with the VIP
```

## Rancher installation without external load-balancer

Usually an external load balancer is used, an internal loadbalancer works as well. For that to work, we first need metallb (to provide external ip address to deployments) and longhorn (for persistent volume claims). Longhorn will be installed through rancher.

```bash
# Add metallb repos to helm
helm repo add metallb https://metallb.github.io/metallb
helm repo update

# Create metallb.yaml
configInline:
  address-pools:
   - name: default
     protocol: layer2
     addresses:
     - 10.0.20.201-10.0.20.250

# Install metallb with config file
helm install metallb metallb/metallb -f metallb.yaml

# Add the rancher repo to helm
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest

# Apply CRDs for cert-manager
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.1/cert-manager.crds.yaml

# Add jetstack repos to helm
helm repo add jetstack https://charts.jetstack.io

# Update helm repos
helm repo update

# Install cert-manager onto the cluster
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.5.1

# WAIT for cert-manager to be fully installed, verify the installtion

# Create test
cat <<EOF > test-resources.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: cert-manager-test
---
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: test-selfsigned
  namespace: cert-manager-test
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: selfsigned-cert
  namespace: cert-manager-test
spec:
  dnsNames:
    - example.com
  secretName: selfsigned-cert-tls
  issuerRef:
    name: test-selfsigned
EOF

# Apply test
kubectl apply -f test-resources.yaml

# Verify test, it is succesful if it ends with "Certificate issued successfully"
kubectl describe certificate -n cert-manager-test

# Remove test
kubectl delete -f test-resources.yaml

# Create the namespace for rancher
kubectl create namespace cattle-system

# Install rancher
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.example.com

# Wait for rancher to install, check the deployment
kubectl -n cattle-system rollout status deploy/rancher

# Give rancher an external IP through metallb
kubectl expose deployment rancher -n cattle-system --type=LoadBalancer --name=rancher-lb --port=443

# Check what the IP is by viewing services of cattle-system
kubectl get services rancher-lb -n cattle-system


```




# Uninstalling k3s

```bash
# Master uninstall
/usr/local/bin/k3s-uninstall.sh

# Agent uninstall
/usr/local/bin/k3s-agent-uninstall.sh

# Unmount mounts
for mount in $(mount | grep tmpfs | grep '/var/lib/kubelet' | awk '{ print $3 }') /var/lib/kubelet /var/lib/rancher; do umount $mount; done

# Remove leftover directories
sudo rm -rf /etc/ceph \
       /etc/cni \
       /etc/kubernetes \
       /opt/cni \
       /opt/rke \
       /run/secrets/kubernetes.io \
       /run/calico \
       /run/flannel \
       /var/lib/calico \
       /var/lib/etcd \
       /var/lib/cni \
       /var/lib/kubelet \
       /var/lib/rancher/rke/log \
       /var/log/containers \
       /var/log/kube-audit \
       /var/log/pods \
       /var/run/calico

# Reboot
```
