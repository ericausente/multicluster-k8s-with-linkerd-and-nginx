# NGINX Ingress controller with Linkerd

This document will walk you through how to install NGINX Ingress controller with Linkerd. Linkerd works with both NGINX Ingress controller open source and NGINX Ingress controller plus version.

We are going to install the Linkderd sidecar proxy with NGINX Ingress controller.

We need to install the `Linkerd` control plane utility.
Linkerd has a great document on how to install the linkerd control plane utility on their website.
You can find the install steps on `Linkerd` website: [Linkerd Service mesh](https://linkerd.io/2.13/getting-started/)

Once you have the `Linkerd` control plane utility installed, you will need to inject the NGINX Ingress controller deployment and your application with the `Linkerd` sidecar.

We are going to use the NGINX Ingress controller custom resource definitions (CRDs) and take advantage of the advanced capabilites they provide.

We can install NGINX Ingress controller two ways: manfiets and with helm.

Using manifests:
You can install NGINX Ingress controller using our manifests from our documentation website.
[NGINX Ingress controller manifest installation](https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/)


Once you have installed NGINX Ingress controller, you will need to inject the deployment with the `Linkerd` sidecar.
This is accomplished by using the `linkerd` control plane utility.

Here is an example:

```bash
kubectl get deployment -n nginx-ingress nginx-ingress -oyaml | linkerd inject - | kubectl apply -f
```

In the above example, my `NGINX Ingress controller` deployment is in the `nginx-ingress` namespace. Adjust accordingly to your environment.


If you are using `Helm`, you can inject the sidecar in two ways.

```
 helm install my-release oci://ghcr.io/nginxinc/charts/nginx-ingress --version 0.17.1
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/ericausente/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/ericausente/.kube/config
Pulled: ghcr.io/nginxinc/charts/nginx-ingress:0.17.1
Digest: sha256:876eb9dc0022d4517f36909822454b2422c49c32ca72d615ba0b2ac6947e7977
NAME: my-release
LAST DEPLOYED: Tue Jun 20 01:00:28 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The NGINX Ingress Controller has been installed.
```

1. You can annotation your `Helm` depoloyment with the following annotation:

```yaml
controller:
  pod:
    ## The annotations of the Ingress Controller pod.
    annotations: { linkerd.io/inject: enabled }
```

This annotation will tell `Linkerd` to automatically inject the sidecar during the install of NGINX Ingress controller using `helm`.



2. Inject the linkerd sidecar to an exisiting `helm` install

If you have an exisiting `Helm` install and want to inject the exisiting install, you can run the following:

```bash
kubectl get deployment -n nginx-ingress <name_of_helm_release> -oyaml | linkerd inject - | kubectl apply -f
```

For example, in my lab I ran the following to inject my `helm` release named `kic01-nginx-ingress-controller` in the `nginx-ingress` namespace:

```bash
$ kubectl get deployment my-release-nginx-ingress-controller -oyaml | linkerd inject - | kubectl apply -f -

deployment "my-release-nginx-ingress-controller" injected

Warning: resource deployments/my-release-nginx-ingress-controller is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/my-release-nginx-ingress-controller configured ```

If we do a `kubectl get pods -n nginx-ingress` on the NGINX Ingress controller, we can see we now have `2/2` pods ready. That confirms that the `Linkerd` sidecar has been successfully injected into NGINX Ingress controller.

```bash
$ kubectl get pods
NAME                                                   READY   STATUS    RESTARTS   AGE
my-release-nginx-ingress-controller-698b6794f8-qtzf5   2/2     Running   0          45s
```

With NGINX Ingress controller successfully deployed with the `Linkerd` sidecar proxy, we can now install our test application.
For this example, we are going to use the `httpbin` image.

```
curl -OsL https://raw.githubusercontent.com/openservicemesh/osm-docs/release-v1.2/manifests/samples/httpbin/httpbin.yaml
```

Apply the test application:

```bash
kubectl apply -f httpbin.yaml
```

To inject an exisiting deployment:
```
$ kubectl get deployment httpbin -oyaml | linkerd inject - | kubectl apply -f -

deployment "httpbin" injected

deployment.apps/httpbin configured
```

You can confirm that the application has been successfully injected with the `linkerd` sidecar:

```bash
$ kubectl get pods
NAME                                                   READY   STATUS    RESTARTS   AGE
httpbin-595f48449f-9f9n9                               2/2     Running   0          23s
my-release-nginx-ingress-controller-698b6794f8-qtzf5   2/2     Running   0          3m43s
```


We can now start sending traffic to NGINX Ingress controller, to verify that `Linkerd` is doing the sidecar traffic connections.

```bash
curl -k https://httpbin.example.com -I
HTTP/1.1 200 OK
Server: nginx/1.23.4
Date: Sat, 20 May 2023 00:08:31 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 9593
Connection: keep-alive
access-control-allow-credentials: true
access-control-allow-origin: *
```

You can further view the status of NGINX Ingress and Linkerd by using the Viz dashboard available in Linkerd:

```bash
linkerd viz dashboard
```
