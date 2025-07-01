# Passo a passo para configurar ambiente


1. Acesse a VM docker-master e docker-node:

```vagrant ssh <nome-da-maquina> ```

2. Instale o Docker e NFS-client em docker-master e docker-node:

```bash
sudo apt install nfs-common -y
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker vagrant

```

3. Ative e inicie o serviço[ docker-master e docker-node]:

```bash
sudo systemctl enable docker
sudo systemctl start docker

docker volume create \
  --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.56.10,nolock,soft,rw \
  --opt device=:/srv/nfs_share \
  webdata


```

4. [Apenas no docker-master] Inicialize o Swarm:

```bash
docker swarm init --advertise-addr 192.168.56.11
```
Copie o docker swarm join ... que será exibido.

5. [Apenas no docker-node1] Ingressar ao Swarm:

6. Configurar NFS Server

```bash
sudo apt update
sudo apt install -y nfs-kernel-server
sudo mkdir -p /srv/nfs_share
sudo chown nobody:nogroup /srv/nfs_share
echo "/srv/nfs_share 192.168.56.0/24(rw,sync,no_subtree_check)" | sudo tee -a /etc/exports
sudo exportfs -rav
sudo systemctl restart nfs-kernel-server

```

