# IBM operator downlaod
check the latest one from https://github.com/IBM/cloud-pak/tree/master/repo/case/ibm-cp-automation

```
wget https://github.com/IBM/cloud-pak/raw/master/repo/case/ibm-cp-automation-3.2.6.tgz
tar -xvzf ibm-cp-automation-3.2.6.tgz
cd ibm-cp-automation/inventory/cp4aOperatorSdk/files/deploy/crs
tar -xvzf cert-k8s-21.0.3.tar
```


https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/21.0.3?topic=automation-installing


# Preparing for a starter deployment 
- https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/21.0.3?topic=deployments-preparing-starter-deployment

```
oc login --token=<token> --server=https://<cluster-ip>:<port>
```
```
export NAMESPACE=cp4ba-dv1
oc create namespace ${NAMESPACE} 
oc project ${NAMESPACE}
```  
```
kubectl create secret docker-registry admin.registrykey -n ${NAMESPACE} \
   --docker-server=cp.icr.io \
   --docker-username=cp \
   --docker-password="<user_password>" \
   --docker-email=<user_email>
```
```
kubectl create secret docker-registry ibm-entitlement-key -n ${NAMESPACE} \
   --docker-username=cp \
   --docker-password="<user_password>" \
   --docker-server=cp.icr.io
```

# service account

```
vi ervice-account-for-privileged.yaml
```  
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ibm-cp4ba-privileged
imagePullSecrets:
- name: "admin.registrykey"
```
```
oc apply -f service-account-for-privileged.yaml -n ${NAMESPACE}
oc adm policy add-scc-to-user privileged -z ibm-cp4ba-privileged -n ${NAMESPACE}
```

```
vi service-account-for-anyuid.yaml
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
oc apply -f service-account-for-anyuid.yaml -n ${NAMESPACE}
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
  storageClassName: gp2-csi
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
  storageClassName: gp2-csi
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
    
 NOTE : KMS encrypted ??
  
  

# Setting up the cluster in silent mode
```  
export CP4BA_AUTO_PLATFORM="OCP"
export CP4BA_AUTO_DEPLOYMENT_TYPE="starter"
export CP4BA_AUTO_ALL_NAMESPACES="No"
export CP4BA_AUTO_NAMESPACE="cp4ba-dv1"
export CP4BA_AUTO_CLUSTER_USER="cluster-admin"
export CP4BA_AUTO_STORAGE_CLASS_FAST_ROKS="gp2-csi"
export CP4BA_AUTO_ENTITLEMENT_KEY="XXXXXXXXXXXXX"
```  
```
cd ibm-cp-automation/inventory/cp4aOperatorSdk/files/deploy/crs 
cd cert-kubernetes/scripts
./cp4a-clusteradmin-setup.sh
```
```
Click Workloads > Secrets, check below 2 exists

admin.registrykey
ibm-entitlement-key
```

# 



### process check
```
Please check the status of Pod by issue cmd:
oc describe pod ibm-cp4a-operator-767f585b78-bqqmf -n cp4ba-dv1

  Warning  Failed                  11m (x4 over 12m)     kubelet                  Failed to pull image "cp.icr.io/cp/cp4a/icp4a-operator@sha256:blabla": rpc error: code = Unknown desc = reading manifest sha256:blabla in cp.icr.io/cp/cp4a/icp4a-operator: denied: insufficient scope
  Warning  Failed                  11m (x4 over 12m)     kubelet                  Error: ErrImagePull
  Warning  Failed                  10m (x6 over 12m)     kubelet                  Error: ImagePullBackOff
  Normal   BackOff                 2m33s (x41 over 12m)  kubelet                  Back-off pulling image 


Please check the status of ReplicaSet by issue cmd:  NOTE : OKAY
oc describe rs ibm-cp4a-operator-767f585b78 -n cp4ba-dv1

Please check the status of PVC by issue cmd: NOTE : OKAY
oc describe pvc operator-shared-pvc -n cp4ba-dv1
oc describe pvc cp4a-shared-log-pvc -n cp4ba-dv1

```
### result check
```

oc get pods
podname=$(oc get pod | grep ibm-cp4a-operator | awk '{print $1}')
oc logs $podname -c operator -n NAMESPACE
```

  
# Installing the capabilities by running the deployment script
https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/21.0.3?topic=scripts-installing-capabilities-by-running-deployment-script

```
oc get route console -n openshift-console -o yaml|grep routerCanonicalHostname
```
```
oc login -u cp4a-user -p cp4a-pwd

  
  
