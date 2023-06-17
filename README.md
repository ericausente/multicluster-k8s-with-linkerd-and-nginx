# multicluster-k8s-with-linkerd-and-nginx

Step-by-step instructions for setting up a multicluster Kubernetes environment using service mirroring. Service mirroring enables consistent observability, security, and routing for both in-cluster and cross-cluster calls without requiring changes to the application code. The following steps outline the process:

Prerequisites
Install EKSCTL: Follow the installation instructions provided here.
https://github.com/weaveworks/eksctl/blob/main/README.md#installation

Cluster Setup
    Create the West Cluster:

```
eksctl create cluster \
  --version 1.27 \
  --region ap-southeast-1 \
  --nodes 1 \
  --name k8s-cluster-west
```

Create the East Cluster:
```
eksctl create cluster \
  --version 1.27 \
  --region ap-southeast-1 \
  --nodes 1 \
  --name k8s-cluster-east
```


Configure kubeconfig

Update kubeconfig for the West Cluster:
```
aws eks update-kubeconfig --name k8s-cluster-west
```

Update kubeconfig for the East Cluster:
```
aws eks update-kubeconfig --name k8s-cluster-east
```

Goal / Pending to do: 

Configure the service mirror:
    Specify the remote cluster to mirror services from.
    Ensure proper service discovery and endpoint configuration.

Validate the service mirroring setup:
    Verify that services from the remote cluster are mirrored locally.
    Test cross-cluster communication between applications.

Customize service mirroring behavior (optional):
    Adjust settings for observability, security, and routing as per your requirements.

