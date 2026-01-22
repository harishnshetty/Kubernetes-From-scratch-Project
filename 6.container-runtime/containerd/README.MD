# Container Runtime

ctr image pull docker.io/library/alpine:latest

sudo apt install -y containerd.io

sudo mkdir -p /etc/containerd

sudo containerd config default | sudo tee /etc/containerd/config.toml

sudo systemctl restart containerd

sudo systemctl enable containerd

nerdctl run -it --rm docker.io/library/alpine:latest

crictl