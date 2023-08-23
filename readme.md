# :cloud: This repository features sample files to create a shared services cluster in EKS Anywhere
### :heavy_exclamation_mark: Apply the changes before applying the files via Kustomize or GitOps
The set of templates will deploy:
* External snapshotter
* PowerStore CSI drivers
* PowerStore Storage class and volumesnaphotclass
* MetalLB and related configuration
* NGINX Ingress controller
* PowerProtect Data Manager Service Account and RBAC
* Cluster Autoscaler
```
cd ssc
grep -rl EKSA_CLUSTER_NAME . | xargs sed -i "s/EKSA_CLUSTER_NAME/$CLUSTER_NAME/g"
kubectl kustomize | kubectl apply -f -
```
* An example of cluster template used
* Note that we have used the disableCSI field to prevent the creation of default storage class
```
apiVersion: anywhere.eks.amazonaws.com/v1alpha1
kind: Cluster
metadata:
 name: workload-cluster-name
spec:
  registryMirrorConfiguration:
    endpoint: "harbor-proxycache.oidc.thecloudgarage.com"
    insecureSkipVerify: true
    port: 443
    ociNamespaces:
      - registry: docker.io
        namespace: proxy.docker.io
      - registry: gcr.io
        namespace: proxy.gcr.io
      - registry: k8s.gcr.io
        namespace: proxy.k8s.gcr.io
      - registry: quay.io
        namespace: proxy.quay.io
      - registry: registry.k8s.io
        namespace: proxy.registry.k8s.io
  clusterNetwork:
    cni: cilium
    pods:
      cidrBlocks:
      - 192.168.0.0/16
    services:
      cidrBlocks:
      - 10.96.0.0/12
  controlPlaneConfiguration:
    count: 2
    endpoint:
      host: "api-server-ip"
    machineGroupRef:
      kind: VSphereMachineConfig
      name: workload-cluster-name-cp
  datacenterRef:
    kind: VSphereDatacenterConfig
    name: workload-cluster-name-dcconfig
  externalEtcdConfiguration:
    count: 3
    machineGroupRef:
      kind: VSphereMachineConfig
      name: workload-cluster-name-etcd
  kubernetesVersion: "1.21"
  managementCluster:
    name: management-cluster-name
  workerNodeGroupConfigurations:
  - count: 2
    machineGroupRef:
      kind: VSphereMachineConfig
      name: workload-cluster-name-wk
    name: md-0
    labels:
      group: md-0
---
apiVersion: anywhere.eks.amazonaws.com/v1alpha1
kind: VSphereDatacenterConfig
metadata:
  name: workload-cluster-name-dcconfig
spec:
  datacenter: "IAC-SSC"
  insecure: true
  network: "iac_pg"
  server: "vc.iac.ssc"
  thumbprint: "F7:A3:92:55:2D:73:B1:BA:1C:77:A8:AC:A3:AD:F3:62:8A:E0:53:CE"
  disableCSI: true
---
apiVersion: anywhere.eks.amazonaws.com/v1alpha1
kind: VSphereMachineConfig
metadata:
  name: workload-cluster-name-cp
spec:
  datastore: "temp_CommonDS"
  diskGiB: 25
  folder: "test-eks-anywhere"
  memoryMiB: 16384
  numCPUs: 8
  osFamily: ubuntu
  template: "ubuntu-2004-kube-v1.21"
  resourcePool: "Test"
  users:
  - name: capv
    sshAuthorizedKeys:
    - ""

---
apiVersion: anywhere.eks.amazonaws.com/v1alpha1
kind: VSphereMachineConfig
metadata:
  name: workload-cluster-name-wk
spec:
  datastore: "temp_CommonDS"
  diskGiB: 25
  folder: "test-eks-anywhere"
  memoryMiB: 16384
  numCPUs: 8
  osFamily: ubuntu
  template: "ubuntu-2004-kube-v1.21"
  resourcePool: "Test"
  users:
  - name: capv
    sshAuthorizedKeys:
    - ""

---
apiVersion: anywhere.eks.amazonaws.com/v1alpha1
kind: VSphereMachineConfig
metadata:
  name: workload-cluster-name-etcd
spec:
  datastore: "temp_CommonDS"
  diskGiB: 25
  folder: "test-eks-anywhere"
  memoryMiB: 8192
  numCPUs: 4
  osFamily: ubuntu
  template: "ubuntu-2004-kube-v1.21"
  resourcePool: "Test"
  users:
  - name: capv
    sshAuthorizedKeys:
    - ""

---
```
