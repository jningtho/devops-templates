# Installing K8s on AWS with kops

Installing K8s cluster on AWS uses `kops`.

Features:
- Fully automated installation
- Uses DNS to identify clusters
- Self-healing: runs through Auto-Scaling Groups
- Limited OS support (Debian preferred, Ubuntu 16.04 supported, early support for CentOS & RHEL)
- High-Availability support
- Can directly provision, or generate Terraform manifests

# Creating Cluster on AWS

1. Install kops

MacOS:

```bash
# Install via Homebrew
brew update && brew install kops
```

Linux:

```bash
wget https://github.com/kubernetes/kops/releases/download/1.8.0/kops-linux-amd64
chmod +x kops-linux-amd64
mv kops-linux-amd64 /usr/local/bin/kops
```

2.
