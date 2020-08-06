## HostConfig Operator


## Overview
An ansible based operator for performing host configuration LCM operations on Kubernetes Nodes. It is built to perform the configuration on kubernetes nodes after the intial kubernetes setup is done on the nodes. It is managed by the cluster itself.

Current implementation have been tested with running one replica of the hostconfig operator deployment on one of the master node in the kubernetes setup.

Once the hostconfig operator is deployed and the corresponding CRD is created on the kubernetes cluster, we can then create the HostConfig CR objects to perform the required configuration on the nodes.  

The host configuration on the kubernetes nodes is done by executing the appropriate ansible playbook on that Kubernetes node by the hostconfig operator pod.


## Scope and Features
* Perform host configuration LCM operations on Kubernetes hosts
* LCM operations managed using HostConfig CR objects
* Inventory built dynamically, at the time of playbook execution
* Connects to hosts using the secrets associated with the nodes, which have the ssh keys associated in them.
* Supports execution based on host-groups, which are built based out of labels associated with kubernetes nodes
* Supports serial/parallel execution of configuration on hosts
* Supports host selection with AND and OR operations of the labels mentioned in the host-groups of the CR object
* Reconcile on failed nodes, based on reconcile period - feature available from ansible-operator
* Current support is available to perform `sysctl` and `ulimit` operations on the kubernetes nodes
* WIP: Display the status of each Hostconfig CR object as part of the `kubectl describe hostconfig <name>`

## Architecture

![](Deployment_Architecture.png)


Hostconfig operator will be running as a kubernetes deployment on the target kubernetes cluster.

**Hostconfig Operator Code**

The code base for the ansible operator is available at: https://github.com/SirishaGopigiri/airship-host-config/tree/integration

The repository also have vagrants scripts to build kubernetes cluster on the Vagrant VMs and then test the ansible-operator pod.

## Deployment and Host Configuration Flow

The hostconfig operator deployment sequence

![](deployment_flow.png)

Using operator pod to perform host configuration on kubernetes nodes

![](CR_creation_flow.png)


## How to Deploy(On existing kubernetes cluster)

**Pre-requisite:**

1. The Kubernetes nodes should be labelled with any one of the below label to execute based on host-groups, if not labelled by default executes on all the nodes as no selection happens.
    Valid labels:
    * [`topology.kubernetes.io/region`](https://kubernetes.io/docs/reference/kubernetes-api/labels-annotations-taints/#topologykubernetesiozone)
    * [`topology.kubernetes.io/zone`](https://kubernetes.io/docs/reference/kubernetes-api/labels-annotations-taints/#topologykubernetesioregion)
    * `kubernetes.io/role`
    * [`kubernetes.io/hostname`](https://kubernetes.io/docs/reference/kubernetes-api/labels-annotations-taints/#kubernetes-io-hostname)
    * [`kubernetes.io/arch`](https://kubernetes.io/docs/reference/kubernetes-api/labels-annotations-taints/#kubernetes-io-arch)
    * [`kubernetes.io/os`](https://kubernetes.io/docs/reference/kubernetes-api/labels-annotations-taints/#kubernetes-io-os)

2. **Operator pod connecting to Kubernetes Nodes:**

    The kubernetes nodes should be annotated with secret name having the username and private key as part of the contents.
2. **Operator pod connecting to Kubernetes Nodes:**

    The kubernetes nodes should be annotated with secret name having the username and private key as part of the contents.

git clone the hostconfig repository

`git clone -b integration https://github.com/SirishaGopigiri/airship-host-config.git`

Move to airship-host-config directory

`cd airship-host-config/airship-host-config`

Create a HostConfig CRD

`kubectl create -f deploy/crds/hostconfig.airshipit.org_hostconfigs_crd.yaml`

Create hostconfig role, service account, role-binding and cluster-role-binding which is used to deploy and manage the operations done using the hostconfig operator pod

`kubectl create -f deploy/role.yaml`
`kubectl create -f deploy/service_account.yaml`
`kubectl create -f deploy/role_binding.yaml`
`kubectl create -f deploy/cluster_role_binding.yaml`

Now deploy the hostconfig operator pod

`kubectl create -f deploy/operator.yaml`

Once the hostconfig operator pod is deployed, we can create the desired HostConfig CR with the required configuration. And this CR can be passed to the operator pod which performs the required operation.

Some example CRs are available in the demo_examples directory.

## Airshipctl integration

The hostconfig operator can be integrated with airshipctl code by changing the manifests folder, which are used to build the target workload cluster.

For the proposed changes, please refer to below Patch set:

https://review.opendev.org/#/c/744098

To deploy the operator on the cluster using `airshipctl phase apply`, the function needs to be invoked from the corresponding kuztomization.yaml file. This is WIP.

## References

1. https://docs.openshift.com/container-platform/4.1/applications/operator_sdk/osdk-ansible.html
2. https://github.com/operator-framework/operator-sdk
3. https://docs.ansible.com/ansible/latest/modules/sysctl_module.html
4. https://docs.ansible.com/ansible/latest/modules/pam_limits_module.html