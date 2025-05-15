# install-rhoai-workshop


### Prerequisites

Install operators through Operator Hub
- OpenShift Pipeline
- OpenShift Service Mesh 2
- OpenShift Serverless
- Node Feature Discovery by Red Hat
    - Create default NodeFeatureDiscovery object
- NVIDIA GPU Operator
    - Create a default Cluster Policy
- Helm 3

### Install Helm cli
```
wget -O get-helm-3 https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 &> /dev/null
chmod 700 get-helm-3
./get-helm-3
rm -f get-helm-3
```

### Install OpenShift AI
- OpenShift AI operator 
    - Create default DSC (Data Science Cluster)

### Enable GPU
You must connect to bastion:
```
ssh lab-user@bastion.xxxxx.sandboxXXXX.opentlc.com
```
clone this repository
```
git clone https://github.com/pkstaz/install-rhoai-workshop.git
cd install-rhoai-workshop
```
Create env
```
OCP_DOMAIN=$(oc get IngressController default -n openshift-ingress-operator -o json | jq -r '.status.domain')
```
```
INFRA_NAME=$(oc get infrastructure cluster -o json | jq -r '.status.infrastructureName')
```
```
SUBNET=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.subnet.filters[0].values[0]}')
```
```
AVAILABILITYZONE=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.placement.availabilityZone}')
```
```
REGION=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.placement.region}')
```

create machine-set using helm
```
helm install gpu-machine-set demo-cluster-config --set infraName=$INFRA_NAME --set subnet=$SUBNET --set availabilityZone=$AVAILABILITYZONE --set region=$REGION
```
