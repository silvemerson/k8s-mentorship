# Provisionar Ambiente

## Acessar o k8s-loadbalancer e instalar o HAProxy

```bash
sudo apt install -y haproxy

edite: /etc/haproxy/haproxy.cfg
```
```bash
global
  log /dev/log local0
  log /dev/log local1 notice
  daemon
  maxconn 2048
  user haproxy
  group haproxy

defaults
  log     global
  mode    tcp
  option  tcplog
  timeout connect 10s
  timeout client  1m
  timeout server  1m

frontend k8s-api
  bind *:6443
  default_backend k8s-masters

backend k8s-masters
  balance roundrobin
  option tcp-check
  default-server inter 10s fall 3 rise 2
  server k8s-master-1 192.168.56.11:6443 check
  server k8s-master-2 192.168.56.12:6443 check
  server k8s-master-3 192.168.56.13:6443 check
```

```bash
sudo systemctl restart haproxy
```


### kubeadm - somente master

```bash
nano /var/lib/kubelet/kubeadm-flags.env
KUBELET_KUBEADM_ARGS=--container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock

```

```bash


kubeadm config images pull

sudo kubeadm init \
  --control-plane-endpoint "192.168.56.10:6443" \
  --upload-certs \
  --pod-network-cidr=192.168.0.0/16

```
###  Executar join nos outros masters

```bash
sudo kubeadm join 192.168.56.10:6443 --control-plane --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH> --certificate-key <KEY>

```
###  Executar join nos workers

```bash
sudo kubeadm join 192.168.56.10:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>

```
### em Master01
```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Validando que o containerd está em uso

```bash
crictl info | grep -i runtime
```

## configurar rede

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
```

## Instale o Docker (opcional, mas útil para builds)

```bash
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker

```