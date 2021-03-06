# Kubernetes API Server Operator


The Kubernetes API Server operator manages and updates the [Kubernetes API server](https://github.com/kubernetes/kubernetes) deployed on top of
[OpenShift](https://openshift.io). The operator is based on OpenShift [library-go](https://github.com/openshift/library-go) framework and it
 is installed via [Cluster Version Operator](https://github.com/openshift/cluster-version-operator) (CVO).
 
It contains the following components:

* Operator
* Bootstrap manifest renderer
* Installer based on static pods
* Configuration observer

By default, the operator exposes [Prometheus](https://prometheus.io) metrics via `metrics` service.
The metrics are collected from following components:

* Kubernetes API Server Operator

## Configuration

The configuration observer component is responsible for reacting on external configuration changes.
For example, this allows external components ([registry](https://github.com/openshift/cluster-image-registry-operator), etcd, etc..)
to interact with the Kubernetes API server configuration ([KubeAPIServerConfig](https://github.com/openshift/api/blob/master/kubecontrolplane/v1/types.go#L14) custom resource).

Currently changes in following external components are being observed:

* `host-etcd` *endpoints* in *kube-system* namespace
  - The observed endpoint addresses are used to configure the `storageConfig.urls` in Kubernetes API server configuration.
* `cluster` *image.config.openshift.io* custom resource 
  - The observed CR resource is used to configure the `imagePolicyConfig.internalRegistryHostname` in Kubernetes API server configuration
* `cluster-config-v1` *configmap* in *kube-system* namespace
  - The observed configmap `install-config` is decoded and the `networking.podCIDR` and `networking.serviceCIDR` is extracted and used as input for `admissionPluginConfig.openshift.io/RestrictedEndpointsAdmission
.configuration.restrictedCIDRs` and `servicesSubnet`


The configuration for the Kubernetes API server is the result of merging:

* a [default config](https://github.com/openshift/cluster-kube-apiserver-operator/blob/master/bindata/v3.11.0/kube-apiserver/defaultconfig.yaml)
* observed config (compare observed values above) `spec.spec.unsupportedConfigOverrides` from the `kubeapiserveroperatorconfig`.

All of these are sparse configurations, i.e. unvalidated json snippets which are merged in order to form a valid configuration at the end. 

## Debugging

Operator also expose events that can help debugging issues. To get operator events, run following command:

```
$ oc get events -n  openshift-cluster-kube-apiserver-operator
```

This operator is configured via [`KubeAPIServerOperatorConfig`](https://github.com/openshift/cluster-kube-apiserver-operator/blob/master/pkg/apis/kubeapiserver/v1alpha1/types.go#L12) custom resource:

```
$ oc describe kubeapiserveroperatorconfig
```
```yaml
apiVersion: kubeapiserver.operator.openshift.io/v1alpha1
kind: KubeAPIServerOperatorConfig
metadata:
  name: instance
spec:
  managementState: Managed
```

The current operator status is reported using the `ClusterOperator` resource. To get the current status you can run follow command:

```
$ oc get clusteroperator openshift-cluster-kube-apiserver-operator
```
