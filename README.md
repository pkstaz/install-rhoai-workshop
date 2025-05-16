# install-rhoai-workshop


### Prerequisites
Install operators through Operator Hub
- OpenShift Pipeline
- OpenShift Service Mesh 2
- OpenShift Serverless

### Connect to Bastion
You must connect to bastion:
```
ssh lab-user@bastion.xxxxx.sandboxXXXX.opentlc.com
```
clone this repository
```
git clone https://github.com/pkstaz/install-rhoai-workshop.git
cd install-rhoai-workshop
```

### Install Helm cli
```
wget -O get-helm-3 https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 &> /dev/null
chmod 700 get-helm-3
./get-helm-3
rm -f get-helm-3
```


### Enable GPU (optional)

Create environment variables:

INFRA_NAME=$(oc get infrastructure cluster -o json | jq -r '.status.infrastructureName')

For AWS:
```
SUBNET=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.subnet.filters[0].values[0]}')

AVAILABILITYZONE=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.placement.availabilityZone}')

REGION=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.placement.region}')
```

For Azure:
```
IMAGESKU=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.image.sku}')

IMAGEVERSION=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.image.version}')

RESOURCEGROUP=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.resourceGroup}')

NETWORKRESOURCEGROUP=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.networkResourceGroup}')

VNET=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.vnet}')

SUBNET=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.subnet}')

AVAILABILITYZONE=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.zone}')

REGION=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.location}')

LOADBALANCER=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.publicLoadBalancer}')
```

Validate env

```
echo $INFRA_NAME
echo $SUBNET
echo $AVAILABILITYZONE
echo $REGION

echo $IMAGESKU
echo $IMAGEVERSION
echo $RESOURCEGROUP
echo $NETWORKRESOURCEGROUP
echo $VNET
echo $LOADBALANCER
```

create machine-set (OCP/AWS) using helm
```
helm install gpu-machine-set gpu-machine-set --set infraName=$INFRA_NAME --set subnet=$SUBNET --set availabilityZone=$AVAILABILITYZONE --set region=$REGION
```

create machine-set (ARO) using helm
```
helm install gpu-machine-set gpu-machine-set --set infraName=$INFRA_NAME --set subnet=$SUBNET --set availabilityZone=$AVAILABILITYZONE --set region=$REGION --set azure.imageSku=$IMAGESKU --set azure.imageVersion=$IMAGEVERSION --set azure.resourceGroup=$RESOURCEGROUP --set networkResourceGroup=$NETWORKRESOURCEGROUP --set vNet=$VNET --set loadBalancer=$LOADBALANCER
```

Install operators through Operator Hub
- Node Feature Discovery by Red Hat
    - Create default NodeFeatureDiscovery object
- NVIDIA GPU Operator
    - Create a default Cluster Policy

### Install OpenShift AI
- OpenShift AI operator 
    - Create default DSC (Data Science Cluster)
