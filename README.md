
# Kubernetes Clusters

## Clusters on Windows - WSL

### Install Docker

[Docker Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

Uninstall the old versions.

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

Setup the repository

```bash
sudo apt-get update
```

```bash
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

Add the Docker registry

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```  

Install the docker engine

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Now Docker is installed. To be able to run without `root`provilege,

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker 
```

To configure to start on boot

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

If this doesn't work (seems flaky in WSL), add the following snippet to your `.bashrc` file,

```bash
# Start Docker daemon automatically when logging in if not running.
RUNNING=`ps aux | grep dockerd | grep -v grep`
if [ -z "$RUNNING" ]; then
    sudo dockerd > /dev/null 2>&1 &
    disown
fi
```

### Install Kind

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64
chmod +x ./kind
mv ./kind /some-dir-in-your-PATH/kind
```

### Create a simple cluster

```bash
kind create cluster --image kindest/node:v1.20.7 --name kind-local-luc
```

### Create a cluster with Ingress

```bash
kind create cluster --image kindest/node:v1.20.7 --name kind-local-luc --config cluster.yaml
```

Install the ingress controller (NGINX)

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

wait for it to be available.

```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

#### Test the ingress

```bash
kubectl apply -f http-echo/example.yaml
```

### Install a proper tool to manage your local cluster

[Lens](https://k8slens.dev/)

You can create an account/space.

Sync Lens with the `~/.kube/config` file in WSL.

- Select the  `Clusters` tag.
- Press the `+` sign

Enter `\\wsl$` in the explorer window.

Select your distro and chose the `/home/<user>/.kube/` folder.

You should now see your local cluster from Lens.

### Install Prometheus

Add the repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

Install with default configuration

```bash
helm install prometheus-local prometheus-community/prometheus
```
