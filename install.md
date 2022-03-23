https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/21.0.3?topic=automation-installing


oc login --token=<token> --server=https://<cluster-ip>:<port>
export NAMESPACE=cp4ba
oc create namespace ${NAMESPACE} 
  
vi service-account-for-starter.yaml
  
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ibm-cp4ba-anyuid
imagePullSecrets:
- name: "admin.registrykey"
  ```
  
oc adm policy add-scc-to-user anyuid -z ibm-cp4ba-anyuid -n ${NAMESPACE}
  
  
  curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl" 
  
  kubectl version --client
  
  wget https://github.com/IBM/cloud-pak/raw/master/repo/case/ibm-cp-automation-3.2.0.tgz
  tar -xvzf ibm-cp-automation-3.2.0.tgz
cd ibm-cp-automation/inventory/cp4aOperatorSdk/files/deploy/crs
tar -xvzf cert-k8s-21.0.3.tar
 
 kubectl create secret docker-registry admin.registrykey -n cp4ba \
   --docker-username=cp \
   --docker-password=<key> \
   --docker-server=cp.icr.io \
  
 pwd 
 /home/cloudshell-user/ibm-cp-automation/inventory/cp4aOperatorSdk/files/deploy/crs
 cd cert-kubernetes/scripts
  
  
  
  
export CP4BA_AUTO_PLATFORM="ROKS"
export CP4BA_AUTO_DEPLOYMENT_TYPE="starter"
export CP4BA_AUTO_ALL_NAMESPACES="No"
export CP4BA_AUTO_NAMESPACE="cp4ba"
export CP4BA_AUTO_CLUSTER_USER="cluster-admin"
export CP4BA_AUTO_STORAGE_CLASS_FAST_ROKS="cp4a-file-retain-gold-gid"
export CP4BA_AUTO_ENTITLEMENT_KEY="XXXXXXXXXXXXX"
  
