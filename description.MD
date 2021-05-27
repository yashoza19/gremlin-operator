## Introduction

Gremlin provides you with the framework to safely, securely, and simply simulate real outages with an ever-growing library of attacks. Using Chaos Engineering to improve system resilience, Gremlin’s “Failure as a Service” makes it easy to find weaknesses in your system before they cause problems for your customers.

## Overview

Gremlin allows targeting objects within your Kubernetes clusters. After selecting a cluster, you can filter the visible set of objects by selecting a namespace. Select any of your Deployments, ReplicaSets, StatefulSets, DaemonSets, or Pods. When one object is selected, all child objects will also be targeted. For example, when selecting a DaemonSet, all of the pods within will be selected.

The Gremlin operator aims to provide easy configuration and installation of the Gremlin Client components with secret-based authentication(Team ID and Team Secret) to get up and running quickly.

## Usage

* Gremlin requires authentication during installation. Before deploying the Gremlin CR, make sure to change the Gremlin credentials(TeamID and TeamSecret)
* Deploy a basic Gremlin client on to the OpenShift cluster by creating a new Gremlin resource in the namespace where the operator is installed.

```
apiVersion: gremlin.com/v1alpha1
kind: Gremlin
metadata:
  name: gremlin-sample
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
      teamID: "*******-******-******"
      teamSecret: "*******-******-******"
      type: secret
```

## Note

Creating the Gremlin CR while result in creation of a custom Security Context Contraint(SCC) which is required by the Gremlin Operator

## Run Attacks

You can now run attacks on your cluster, including Kubernetes attacks[link](https://app.gremlin.com/attacks/new/kubernetes).

See the [documentation](https://argocd-operator.readthedocs.io) and examples on
[github](https://github.com/argoproj-labs/argocd-operator) for more information.