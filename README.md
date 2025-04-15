# k3d-gpu-ubuntu24_04

This repo contains required files/scripts written article at 

https://www.linkedin.com/pulse/creating-gpu-accelerated-kubernetes-cluster-k3d-ubuntu-2404-xauhe


## TLDR

1. Install GPU based Ubuntu 24.04 server

2. On Server

```bash
./install_gpu_drivers_on_ubuntu2404.sh
```
3. Build Docker image and push to Container Registry

Edit IMAGE_REGISTRY at build.sh

```bash
./build.sh
```

4. Create Cluster

Change image and find out eth0 IP address (ie first eth card)

```bash
k3d cluster create cluster --image=crtechmakers.azurecr.io/k3s:v1.31.7-k3s1-cuda-12.8.1-base-ubuntu24.04 --gpus 1 -p 192.168.50.230:8081:80@loadbalancer
```

5. Deploy test container and check logs

```bash
kubectl apply -f  tests/test-cuda-vector-add.yml 
```
and

```
$ kubectl logs -n default cuda-vector-add

[Vector addition of 50000 elements]
Copy input data from the host memory to the CUDA device
CUDA kernel launch with 196 blocks of 256 threads
Copy output data from the CUDA device to the host memory
Test PASSED
Done        
```

# Creating a GPU-Accelerated Kubernetes Cluster with K3D on Ubuntu 24.04

As the demand for machine learning, scientific computing, and GPU-accelerated workloads continues to rise, deploying lightweight yet GPU-capable Kubernetes environments is increasingly valuable. In this post, we demonstrate how to set up a GPU-enabled Kubernetes cluster using [K3D](https://k3d.io) on Ubuntu 24.04, targeting the latest stable versions of Kubernetes, CUDA, and the NVIDIA driver stack.

This article draws inspiration from the official K3D guide on [building a customized K3s image with CUDA support](https://k3d.io/v5.8.3/usage/advanced/cuda/#building-a-customized-k3s-image). However, the official guide references older software versions. Our goal was to modernize that approach and use the most up-to-date releases — including Ubuntu 24.04, CUDA 12.8, and Kubernetes v1.31 — while maintaining compatibility with the NVIDIA container runtime.

---

## Software Versions Used

| Component                  | Version                        |
|---------------------------|--------------------------------|
| Host Operating System     | Ubuntu 24.04                   |
| CUDA Toolkit              | 12.8.1                         |
| NVIDIA Driver             | 570                            |
| K3s (Kubernetes)          | v1.31.7+k3s1                   |
| K3D CLI                   | v5.8.3                         |
| NVIDIA Container Toolkit  | Latest (Ubuntu 22.04 repo)     |
| NVIDIA Device Plugin      | v0.17.1                        |
| Docker                    | Latest stable (April 2025)     |
| Azure CLI                 | Latest                         |

