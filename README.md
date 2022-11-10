# internal-services-controller PoC

Proof of concept to see if a controller can watch 2 clusters for Custom Resources

## Description

This PoC demonstrates that a local cluster can connect to a remote cluster and watch/update custom resources.

## Building

```
make build
```

## Running

* We are going to assume we have 2 clusters: **remote** and **local**
* The controller will ultimately run on the **local** cluster, but it will also connect and watch the **remote** cluster.

### Cluster setup

Let's say we have 2 KUBECONFIG files:

- local.kubeconfig
- remote.kubeconfig

#### Create local cluster (if required)

```
kind create cluster -n local
kind get kubeconfig -n local > local.kubeconfig
```

### Apply CRDs to both clusters

We need to apply the CRDs to both clusters

```
KUBECONFIG=local.kubeconfig oc apply -f  config/crd/bases/appstudio.redhat.com_requests.yaml
KUBECONFIG=remote.kubeconfig oc apply -f  config/crd/bases/appstudio.redhat.com_requests.yaml
``` 

### Startup manager

```
export KUBECONFIG=local.kubeconfig
./bin/manager -remoteconfig remote.kubeconfig
```

### Create CR in remote cluster 

```
KUBECONFIG=remote.kubeconfig oc apply -f config/samples/_v1alpha1_request.yaml
```

### Examine CR

```
KUBECONFIG=remote.kubeconfig oc get request/request-sample -o yaml
```

You should see:

```
status:
  seen: "true"
```

---

## How this code was created

```
clone and build [KCP operator SDK|https://github.com/fgiloux/kcp-operator-sdk]
mkdir new-dir
cd new-dir
kcp-operator-sdk init --component-config --domain appstudio.redhat.com --repo github.com/hacbs-release/internal-services-controller
kcp-operator-sdk create api --version v1alpha1 --kind Request
make manifests apiresourceschemas
make build
```

