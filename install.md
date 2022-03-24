https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/21.0.3?topic=automation-installing

# Preparing for a starter deployment 
- https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/21.0.3?topic=deployments-preparing-starter-deployment

```
oc login --token=<token> --server=https://<cluster-ip>:<port>
export NAMESPACE=cp4ba-dv1
oc create namespace ${NAMESPACE} 
```  

```
vi service-account-for-starter.yaml
```  
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ibm-cp4ba-anyuid
imagePullSecrets:
- name: "admin.registrykey"
```
```  
oc adm policy add-scc-to-user anyuid -z ibm-cp4ba-anyuid -n ${NAMESPACE}
```  

# aws volumn storageclass pv pvc
## aws

operator-shared-pvc 1G
cp4a-shared-log-pvc 100G

https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/21.0.x?topic=operator-preparing-log-file-storage
make it pointing to aws efs
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pvc-operator-shared
  labels:
    topology.kubernetes.io/region: ap-southeast-2
    topology.kubernetes.io/zone: ap-southeast-2a
  annotations:
    kubernetes.io/createdby: aws-ebs-dynamic-provisioner
    pv.kubernetes.io/bound-by-controller: 'yes'
    pv.kubernetes.io/provisioned-by: kubernetes.io/aws-ebs 
  finalizers:
    - kubernetes.io/pv-protection   
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeID: 'aws://ap-southeast-2a/vol-095d9ec092c1ce5f4'
    fsType: ext4
  persistentVolumeReclaimPolicy: Retain
  storageClassName: gp2
  volumeMode: Filesystem
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: topology.kubernetes.io/region
              operator: In
              values:
                - ap-southeast-2
            - key: topology.kubernetes.io/zone
              operator: In
              values:
                - ap-southeast-2a
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pvc-cp4a-shared-log
  labels:
    topology.kubernetes.io/region: ap-southeast-2
    topology.kubernetes.io/zone: ap-southeast-2a
  annotations:
    kubernetes.io/createdby: aws-ebs-dynamic-provisioner
    pv.kubernetes.io/bound-by-controller: 'yes'
    pv.kubernetes.io/provisioned-by: kubernetes.io/aws-ebs 
  finalizers:
    - kubernetes.io/pv-protection   
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 100Gi
  awsElasticBlockStore:
    volumeID: 'aws://ap-southeast-2a/vol-095d9ec092c1ce5f4'
    fsType: ext4
  persistentVolumeReclaimPolicy: Retain
  storageClassName: gp2
  volumeMode: Filesystem
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: topology.kubernetes.io/region
              operator: In
              values:
                - ap-southeast-2
            - key: topology.kubernetes.io/zone
              operator: In
              values:
                - ap-southeast-2a
```

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: operator-shared-pvc
  namespace: cp4ba-dv1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  volumeName: pvc-operator-shared
 ---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cp4a-shared-log-pvc
  namespace: cp4ba-dv1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  volumeName: pvc-cp4a-shared-log
 ```
    
 
  
  

# Setting up the cluster in silent mode
```  
export CP4BA_AUTO_PLATFORM="OCP"
export CP4BA_AUTO_DEPLOYMENT_TYPE="starter"
export CP4BA_AUTO_ALL_NAMESPACES="No"
export CP4BA_AUTO_NAMESPACE="cp4ba-dv1"
export CP4BA_AUTO_CLUSTER_USER="cluster-admin"
export CP4BA_AUTO_STORAGE_CLASS_FAST_ROKS="gp2"
export CP4BA_AUTO_ENTITLEMENT_KEY="XXXXXXXXXXXXX"
```  
```
cd ibm-cp-automation/inventory/cp4aOperatorSdk/files/deploy/crs 
cd cert-kubernetes/scripts
./cp4a-clusteradmin-setup.sh
```

```
oc get pods
podname=$(oc get pod | grep ibm-cp4a-operator | awk '{print $1}')
oc logs $podname -c operator -n NAMESPACE
```

  
# Installing the capabilities by running the deployment script
```
oc get route console -n openshift-console -o yaml|grep routerCanonicalHostname
```
  
  
  
