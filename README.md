# nvidia-gpu-k8s-pinning-tests
testing pinning of nvidia gpus to specific workloads in k8s

## Server Setup

Tested on Ubuntu 24.04 LTS with 2 x NVIDIA A100 GPUs and nvidia drivers already installed.

Assumes server is available via `ssh gputests`

Install RKE2

```bash
curl -sfL https://get.rke2.io | sudo sh -
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service
echo 'PATH=$PATH:/var/lib/rancher/rke2/bin/' >> ~/.bashrc
echo 'export KUBECONFIG=/etc/rancher/rke2/rke2.yaml' >> ~/.bashrc
sudo chown $USER /etc/rancher/rke2/rke2.yaml
. ~/.bashrc
```

Copy Manifests

```bash
ssh gputests mkdir -p manifests
scp ./manifests/*.yaml gputests:manifests/
```

Deploy GPU Operator

```bash
kubectl apply -f manifests/gpu-operator-helm-chart.yaml
```

Get the 2nd GPU UUID

```bash
nvidia-smi -L
```

Deploy test workload pinned to the 2nd GPU

```bash
sed -i 's/__DEVICE_UUID__/2ND_GPU_UUID/' manifests/nvidia-smi-deployment.yaml
kubectl apply -f manifests/nvidia-smi-deployment.yaml
kubectl exec deploy/nvidia-smi -- nvidia-smi -L
```

It should return the 2nd device UUID
