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

### 1. Install kops

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

### 2. Create a route53 domain for your cluster

kops uses DNS for discovery and easier to reach the K8s API server from Clients.
e.g: - DNS for domain - k8s.dev.mydomain.com
     - API Server endpoint - api.8s.dev.mydomain.com

Route53 hosted zone: - k8s.dev.mydomain.com
CLI:
```aws route53 create-hosted-zone --name dev.mydomain.com --caller-reference 1
```
Set up your NS records in the parent domain, so that records in the domain will resolve. Here, you would create NS records in mydomain.com for dev. If it is a root domain name you would configure the NS records at your domain registrar (e.g. example.com would need to be configured where you bought example.com).

Use the `dig` tool to confirm cluster is configured correctly. You should see the 4 AWS NS records that Route53 assigned your hosted zone.

### 3. Create an S3 bucket to store your clusters state

`kops` lets you manage your clusters even after installation. To do this, it must keep track of the clusters that you have created, along with their configuration, the keys they are using etc. This information is stored in an S3 bucket. S3 permissions are used to control access to the bucket.

Multiple clusters can use the same S3 bucket, and you can share an S3 bucket between your colleagues that administer the same clusters - this is much easier than passing around kubecfg files. But anyone with access to the S3 bucket will have administrative access to all your clusters, so you don’t want to share it beyond the operations team.

So typically you have one S3 bucket for each ops team (and often the name will correspond to the name of the hosted zone above!)

From our example, lets create `clusters.dev.mydomain.com` as the S3 bucket name.

`Export AWS_PROFILE (if you need to select a profile for the AWS CLI to work)
Create the S3 bucket using `aws s3 mb s3://clusters.dev.mydomain.com` from CLI
You can export KOPS_STATE_STORE=s3://clusters.dev.mydomain.com and then `kops` will use this location by default. Its best practice putting this in your bash profile or similar.`

### 4. Build your cluster configuration

Run “kops create cluster” to create your cluster configuration:
```bash
kops create cluster --zones=us-east-1a useast1.dev.mydomain.com
```
kops will create the configuration for your cluster. Note that it only creates the configuration, it does not actually create the cloud resources - do that in the next step with a
```
kops update cluster
```
This lets us review the configuration or change it.

- List Clusters: `kops get cluster`
- Edit Cluster: `kops edit cluster useast1.dev.mydomain.com`
- Edit Node instance group: `kops edit ig --name=useast1.dev.mydomain.com nodes`
- Edit master instance group: `kops edit ig --name=useast1.dev.mydomain.com master-us-east-1a`

Instance groups are registered as K8s nodes and is implemented via Auto-Scaling Groups.

### 5. Create the cluster in AWS

Run “kops update cluster” to create your cluster in AWS:

```
kops update cluster useast1.dev.mydomain.com --yes
```
`kops update cluster` will be used whenever a change is made in the configuration of the cluster, it applies the changes  made to the configuration to the cluster - reconfiguring AWS or `k8s` as needed.

For example, after you `kops edit ig nodes`, then `kops update cluster --yes` to apply your configuration, and sometimes you will also have to `kops rolling-update cluster` to roll out the configuration immediately.

Without ``--yes, kops update cluster` will show you a preview of what it is going to do.
