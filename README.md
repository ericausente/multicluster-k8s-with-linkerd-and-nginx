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



Step 1: Install the CLI

If this is your first time running Linkerd, you will need to download the linkerd CLI onto your local machine. The CLI will allow you to interact with your Linkerd deployment.

To install the CLI manually, run:
```
curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install | sh
```

Add the linkerd CLI to your path with:
```
  export PATH=$PATH:/Users/e.ausente/.linkerd2/bin
```

validate that Linkerd can be installed
```
e.ausente@C02DR4L1MD6M hydra-issue-404-502 % linkerd check --pre
zsh: command not found: linkerd
e.ausente@C02DR4L1MD6M hydra-issue-404-502 % export PATH=$PATH:/Users/e.ausente/.linkerd2/bin
e.ausente@C02DR4L1MD6M hydra-issue-404-502 % linkerd check --pre
kubernetes-api
--------------
√ can initialize the client
√ can query the Kubernetes API

kubernetes-version
------------------
√ is running the minimum Kubernetes API version

pre-kubernetes-setup
--------------------
√ control plane namespace does not already exist
√ can create non-namespaced resources
√ can create ServiceAccounts
√ can create Services
√ can create Deployments
√ can create CronJobs
√ can create ConfigMaps
√ can create Secrets
√ can read Secrets
√ can read extension-apiserver-authentication configmap
√ no clock skew detected

linkerd-version
---------------
√ can determine the latest version
√ cli is up-to-date

Status check results are √
```

Once installed, verify the CLI is running correctly with:
```
linkerd version
```

Installing with the CLI

Once you have a cluster ready, installing Linkerd is as easy as running linkerd install --crds, which installs the Linkerd CRDs, followed by linkerd install, which installs the Linkerd control plane. Both of these commands generate Kubernetes manifests, which can be applied to your cluster to install Linkerd.

For example:
```
# install the CRDs first
linkerd install --crds | kubectl apply -f -

# install the Linkerd control plane once the CRDs have been installed
linkerd install | kubectl apply -f -
```
This basic installation should work for most cases. However, there are some configuration options are provided as flags for install. See the CLI reference documentation for a complete list of options. You can also use tools like Kustomize to programmatically alter this manifest.


Output: 
```
e.ausente@C02DR4L1MD6M hydra-issue-404-502 % linkerd install --crds | kubectl apply -f -

Rendering Linkerd CRDs...
Next, run `linkerd install | kubectl apply -f -` to install the control plane.

customresourcedefinition.apiextensions.k8s.io/authorizationpolicies.policy.linkerd.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.policy.linkerd.io created
customresourcedefinition.apiextensions.k8s.io/meshtlsauthentications.policy.linkerd.io created
customresourcedefinition.apiextensions.k8s.io/networkauthentications.policy.linkerd.io created
customresourcedefinition.apiextensions.k8s.io/serverauthorizations.policy.linkerd.io created
customresourcedefinition.apiextensions.k8s.io/servers.policy.linkerd.io created
customresourcedefinition.apiextensions.k8s.io/serviceprofiles.linkerd.io created
e.ausente@C02DR4L1MD6M hydra-issue-404-502 % linkerd install | kubectl apply -f -

namespace/linkerd created
clusterrole.rbac.authorization.k8s.io/linkerd-linkerd-identity created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-linkerd-identity created
serviceaccount/linkerd-identity created
clusterrole.rbac.authorization.k8s.io/linkerd-linkerd-destination created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-linkerd-destination created
serviceaccount/linkerd-destination created
secret/linkerd-sp-validator-k8s-tls created
validatingwebhookconfiguration.admissionregistration.k8s.io/linkerd-sp-validator-webhook-config created
secret/linkerd-policy-validator-k8s-tls created
validatingwebhookconfiguration.admissionregistration.k8s.io/linkerd-policy-validator-webhook-config created
clusterrole.rbac.authorization.k8s.io/linkerd-policy created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-destination-policy created
role.rbac.authorization.k8s.io/linkerd-heartbeat created
rolebinding.rbac.authorization.k8s.io/linkerd-heartbeat created
clusterrole.rbac.authorization.k8s.io/linkerd-heartbeat created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-heartbeat created
serviceaccount/linkerd-heartbeat created
clusterrole.rbac.authorization.k8s.io/linkerd-linkerd-proxy-injector created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-linkerd-proxy-injector created
serviceaccount/linkerd-proxy-injector created
secret/linkerd-proxy-injector-k8s-tls created
mutatingwebhookconfiguration.admissionregistration.k8s.io/linkerd-proxy-injector-webhook-config created
configmap/linkerd-config created
role.rbac.authorization.k8s.io/ext-namespace-metadata-linkerd-config created
secret/linkerd-identity-issuer created
configmap/linkerd-identity-trust-roots created
service/linkerd-identity created
service/linkerd-identity-headless created
deployment.apps/linkerd-identity created
service/linkerd-dst created
service/linkerd-dst-headless created
service/linkerd-sp-validator created
service/linkerd-policy created
service/linkerd-policy-validator created
deployment.apps/linkerd-destination created
cronjob.batch/linkerd-heartbeat created
deployment.apps/linkerd-proxy-injector created
service/linkerd-proxy-injector created
secret/linkerd-config-overrides created
e.ausente@C02DR4L1MD6M hydra-issue-404-502 % 
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





