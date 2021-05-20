# Gremlin-Operator

## Steps to run gremlin-operator:

In this document are the steps to run the Gremlin operator as a deployment inside a OpenShift cluster.

## Prerequistes:

Since the gremlin operator is designed to run inside an OpenShift cluster, hence set it up first. For local tests we recommend to use one of the following solutions:
* [crc](https://github.com/code-ready/crc), which creates a single-node openshift cluster on your laptop.

## Deployment:

The Gremlin operator can be installed simply by running it as a deployment inside the cluster:

First, clone the repository and change to the directory:

```sh
git clone https://github.com/yashoza19/gremlin-operator.git
cd gremlin-operator
```

By default, a new namespace will be created with the name `gremlin-system` and will be used for the deployment.

Make sure you can currently access an OpenShift cluster:

```sh
oc get projects
```

Run the following to deploy the operator. This will also install the RBAC manifests from `config/rbac`:

```sh
make deploy
```

The make deploy command will create the resources required:

```sh
$ make deploy
cd config/manager && /usr/local/bin/kustomize edit set image controller=quay.io/yoza/gremlin-controller:2.16.2
/usr/local/bin/kustomize build config/default | kubectl apply -f -
namespace/gremlin-system created
customresourcedefinition.apiextensions.k8s.io/gremlins.gremlin.com created
serviceaccount/gremlin-controller-manager created
role.rbac.authorization.k8s.io/gremlin-leader-election-role created
clusterrole.rbac.authorization.k8s.io/gremlin-manager-role created
clusterrole.rbac.authorization.k8s.io/gremlin-metrics-reader created
clusterrole.rbac.authorization.k8s.io/gremlin-proxy-role created
rolebinding.rbac.authorization.k8s.io/gremlin-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/gremlin-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/gremlin-proxy-rolebinding created
configmap/gremlin-manager-config created
service/gremlin-controller-manager-metrics-service created
deployment.apps/gremlin-controller-manager created
```
Switch to the `gremlin-system` namespace:

```sh
oc project gremlin-system
```

Verify the controller/operator is up and running:

```sh
oc get deployment
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
gremlin-controller-manager   1/1     1            1           30s
```

```sh
oc get pods
NAME                                          READY   STATUS    RESTARTS   AGE
gremlin-controller-manager-5df4d895d6-ths5x   2/2     Running   0          45s
```

Once the controller/operator is running, we want to create our custom CR in the `config/sample` directory. To create the CR, we first need to add the `teamID` and `teamSecret` to the sample CR.

The gremlin CR should look like this:

```
apiVersion: gremlin.com/v1alpha1
kind: Gremlin
metadata:
  name: gremlin-sample
  namespace: gremlin-system
spec:
  gremlin:
    container:
      driver: crio-runc
    hostPID: true
    podSecurity:
      seccomp:
        enabled: true
      securityContextConstraints:
        create: true
    secret:
      clusterID: my-cluster
      managed: true
      teamID: **********
      teamSecret: **********
      type: secret
```

Create the gremlin CR that was modified:

```sh
oc apply -f config/samples/gremlin_v1alpha1_gremlin.yaml
```

To verify the creation of the gremlin CR:

```sh
oc get gremlin gremlin-sample
NAME             AGE
gremlin-sample   1m
```
Verify the creation of the Operands (Gremlin DaemonSet and Chao Deployment):

```sh
oc get pods
NAME                                          READY   STATUS    RESTARTS   AGE
chao-5c87fdf855-hflnm                         1/1     Running   0          63s
gremlin-controller-manager-74b65979d6-nmgvr   1/2     Running   2          13m
gremlin-sample-5b2t5                          1/1     Running   0          63s
gremlin-sample-dzf9k                          1/1     Running   0          63s
gremlin-sample-kdjmw
```


## Cleanup:

Clean up the gremlin CR first:
```
oc delete -f config/samples/gremlin_v1alpha1_gremlin.yaml
```

**Note:** Make sure the above custom resource has been deleted before proceeding to run make undeploy, as helm-operator’s controller adds finalizers to the custom resources. Otherwise your cluster may have dangling custom resource objects that cannot be deleted.

```
make undeploy
```
