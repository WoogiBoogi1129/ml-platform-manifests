# Kubeadm Configuration

This directory contains kubeadm configuration files for setting up a Kubernetes cluster with Dynamic Resource Allocation (DRA) support.

## Features

- **Kubernetes Version**: 1.34.2
- **DRA Support**: `resource.k8s.io/v1beta1` API enabled
- **Feature Gates**: `DynamicResourceAllocation=true` on all components
- **Container Runtime**: containerd with systemd cgroup driver
- **Network**: Flannel CNI (configurable)

## Files

| File | Description |
|------|-------------|
| `kubeadm-config.yaml` | Control plane initialization configuration |
| `kubeadm-join-config.yaml` | Worker node join configuration template |
| `install-prerequisites.sh` | Script to install all prerequisites |
| `init-cluster.sh` | Script to initialize the control plane |

## Quick Start

### 1. Install Prerequisites (on all nodes)

```bash
sudo ./install-prerequisites.sh
```

### 2. Initialize Control Plane (on master node)

```bash
sudo ./init-cluster.sh
```

### 3. Join Worker Nodes

After initialization, use the join command printed by the init script:

```bash
sudo kubeadm join <control-plane>:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>
```

Or use the join configuration file (update placeholders first):

```bash
# Edit kubeadm-join-config.yaml with your values
sudo kubeadm join --config kubeadm-join-config.yaml
```

## DRA Configuration Details

### API Server

```yaml
apiServer:
  extraArgs:
    - name: runtime-config
      value: "resource.k8s.io/v1beta1=true"
    - name: feature-gates
      value: "DynamicResourceAllocation=true"
```

### Controller Manager

```yaml
controllerManager:
  extraArgs:
    - name: feature-gates
      value: "DynamicResourceAllocation=true"
```

### Scheduler

```yaml
scheduler:
  extraArgs:
    - name: feature-gates
      value: "DynamicResourceAllocation=true"
```

### Kubelet

```yaml
featureGates:
  DynamicResourceAllocation: true
```

## Verification

After cluster initialization, verify DRA is enabled:

```bash
# Check API resources
kubectl api-resources | grep resourceslice

# Check ResourceSlices (after installing DRA driver)
kubectl get resourceslices

# Check ResourceClaims
kubectl get resourceclaims -A
```

## GPU Node Setup

For GPU nodes, additional setup is required:

1. **Install NVIDIA Drivers**
2. **Install NVIDIA Container Toolkit**
3. **Install k8s-dra-driver-gpu**

See `applications/k8s-dra-driver-gpu/README.md` for details.

## Customization

### Change Pod Network CIDR

Edit `kubeadm-config.yaml`:

```yaml
networking:
  podSubnet: "10.244.0.0/16"  # Change this
```

### Add API Server SANs

For multi-master or load balancer setups:

```yaml
apiServer:
  certSANs:
    - "k8s.example.com"
    - "192.168.1.100"
```

### Set Control Plane Endpoint

For HA clusters:

```yaml
controlPlaneEndpoint: "k8s.example.com:6443"
```

