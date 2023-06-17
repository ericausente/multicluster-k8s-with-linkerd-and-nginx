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


Do a linkerd check. 
After installation (whether CLI or Helm) you can validate that Linkerd is in a good state running:

```
% linkerd check
kubernetes-api
--------------
√ can initialize the client
√ can query the Kubernetes API

kubernetes-version
------------------
√ is running the minimum Kubernetes API version

linkerd-existence
-----------------
√ 'linkerd-config' config map exists
√ heartbeat ServiceAccount exist
√ control plane replica sets are ready
√ no unschedulable pods
√ control plane pods are ready
√ cluster networks contains all pods
√ cluster networks contains all services

linkerd-config
--------------
√ control plane Namespace exists
√ control plane ClusterRoles exist
√ control plane ClusterRoleBindings exist
√ control plane ServiceAccounts exist
√ control plane CustomResourceDefinitions exist
√ control plane MutatingWebhookConfigurations exist
√ control plane ValidatingWebhookConfigurations exist
√ proxy-init container runs as root user if docker container runtime is used

linkerd-identity
----------------
√ certificate config is valid
√ trust anchors are using supported crypto algorithm
√ trust anchors are within their validity period
√ trust anchors are valid for at least 60 days
√ issuer cert is using supported crypto algorithm
√ issuer cert is within its validity period
√ issuer cert is valid for at least 60 days
√ issuer cert is issued by the trust anchor

linkerd-webhooks-and-apisvc-tls
-------------------------------
√ proxy-injector webhook has valid cert
√ proxy-injector cert is valid for at least 60 days
√ sp-validator webhook has valid cert
√ sp-validator cert is valid for at least 60 days
√ policy-validator webhook has valid cert
√ policy-validator cert is valid for at least 60 days

linkerd-version
---------------
√ can determine the latest version
√ cli is up-to-date

control-plane-version
---------------------
√ can retrieve the control plane version
√ control plane is up-to-date
√ control plane and cli versions match

linkerd-control-plane-proxy
---------------------------
√ control plane proxies are healthy
√ control plane proxies are up-to-date
√ control plane proxies and cli versions match

Status check results are √
```

Let's install the dashboard. 


To install the viz extension, run:
```
linkerd viz install | kubectl apply -f - 
# install the on-cluster metrics stack
```
Once you’ve installed the extension, let’s validate everything one last time:
```
linkerd check

...
linkerd-viz
-----------
√ linkerd-viz Namespace exists
√ can initialize the client
√ linkerd-viz ClusterRoles exist
√ linkerd-viz ClusterRoleBindings exist
√ tap API server has valid cert
√ tap API server cert is valid for at least 60 days
√ tap API service is running
√ linkerd-viz pods are injected
√ viz extension pods are running
√ viz extension proxies are healthy
√ viz extension proxies are up-to-date
√ viz extension proxies and cli versions match
√ prometheus is installed and configured correctly
√ viz extension self-check
```

With the control plane and extensions installed and running, we’re now ready to explore Linkerd! Access the dashboard with:
```
linkerd viz dashboard &

 % linkerd viz dashboard &
Linkerd dashboard available at:
http://localhost:50750
Grafana dashboard available at:
http://localhost:50750/grafana
Opening Linkerd dashboard in the default browser

```

From the dashbaord: 
Mesh of proxies - dataplane
We need an interface point of those proxies - this is the control plane

Certificate Management, Dashboard and so on... 



Now let's install the demo app

Congratulations, Linkerd is installed! However, it’s not doing anything just yet. To see Linkerd in action, we’re going to need an application.

Let’s install a demo application called Emojivoto. Emojivoto is a simple standalone Kubernetes application that uses a mix of gRPC and HTTP calls to allow the user to vote on their favorite emojis.

Install Emojivoto into the emojivoto namespace by running:
```
curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/emojivoto.yml \
  | kubectl apply -f -
```  

To change from the default namespace to one named ‘emojivoto’ you would enter:
```
% kubectl config set-context --current --namespace="emojivoto"

% kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
emoji-68cdd48fc7-jdjd7      1/1     Running   0          3m12s
vote-bot-85c88b944d-49ghb   1/1     Running   0          3m12s
voting-5b7f854444-bkt4z     1/1     Running   0          3m12s
web-679ccff67b-9wrqk        1/1     Running   0          3m11s
```


Before we mesh it, let’s take a look at Emojivoto in its natural state. We’ll do this by forwarding traffic to its web-svc service so that we can point our browser to it. Forward web-svc locally to port 8080 by running:
```
kubectl -n emojivoto port-forward svc/web-svc 8080:80
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





