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



 
 kubectl create secret docker-registry admin.registrykey -n cp4ba \
   --docker-username=cp \
   --docker-password=<key> \
   --docker-server=cp.icr.io \
  
 pwd 
 /home/cloudshell-user/ibm-cp-automation/inventory/cp4aOperatorSdk/files/deploy/crs
 cd cert-kubernetes/scripts
  
  

# Setting up the cluster in silent mode
```  
export CP4BA_AUTO_PLATFORM="ROKS"
export CP4BA_AUTO_DEPLOYMENT_TYPE="starter"
export CP4BA_AUTO_ALL_NAMESPACES="No"
export CP4BA_AUTO_NAMESPACE="cp4ba-dv1"
export CP4BA_AUTO_CLUSTER_USER="cluster-admin"
export CP4BA_AUTO_STORAGE_CLASS_FAST_ROKS="cp4a-file-retain-gold-gid"
export CP4BA_AUTO_STORAGE_CLASS_FAST_ROKS="gp2-csi"
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
  
  
  
