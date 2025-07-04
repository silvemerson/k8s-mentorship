# Provisionar Ambiente


## Instale e configure o containerd

```bash
sudo apt-get install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo sed -i 's#sandbox_image = "registry.k8s.io/pause:3.8"#sandbox_image = "registry.k8s.io/pause:3.9"#' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

sudo tee /etc/crictl.yaml > /dev/null <<EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF

sudo chmod 644 /etc/crictl.yaml

```

## Instale Kubernetes

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```


### kubeadm - somente master

```bash

nano /var/lib/kubelet/kubeadm-flags.env
KUBELET_KUBEADM_ARGS=--container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock
kubeadm config images pull
sudo kubeadm init --apiserver-advertise-address=192.168.56.10 --pod-network-cidr=192.168.0.0/16
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