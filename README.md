# Red Hat OpenShift AI Workshop Installation Guide

This repository contains the installation scripts and configurations for setting up a Red Hat OpenShift AI (RHOAI) workshop environment. The setup includes GPU support, service mesh, serverless computing, and ML/AI tooling.

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Installation Steps](#installation-steps)
- [GPU Configuration (Optional)](#gpu-configuration-optional)
- [Validation](#validation)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## 🔧 Prerequisites

Before beginning the installation, ensure you have the following operators installed through the OpenShift Operator Hub:

- **OpenShift Pipelines** - Tekton-based CI/CD pipeline solution
- **OpenShift Service Mesh 2** - Istio-based service mesh for microservices
- **OpenShift Serverless** - Knative-based serverless platform

## 🚀 Getting Started

### 1. Connect to Bastion Host

Connect to your bastion host using SSH:

```bash
ssh lab-user@bastion.xxxxx.sandboxXXXX.opentlc.com
```

### 2. Clone Repository

Clone this repository and navigate to the project directory:

```bash
git clone https://github.com/pkstaz/install-rhoai-workshop.git
cd install-rhoai-workshop
```

### 3. Install Helm CLI

Download and install the Helm package manager:

```bash
wget -O get-helm-3 https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 &> /dev/null
chmod 700 get-helm-3
./get-helm-3
rm -f get-helm-3
```

## 📦 Installation Steps

### GPU Configuration (Optional)

If you need GPU support for machine learning workloads, follow these steps:

#### Step 1: Set Infrastructure Name

```bash
INFRA_NAME=$(oc get infrastructure cluster -o json | jq -r '.status.infrastructureName')
```

#### Step 2: Configure Cloud Provider Variables

**For AWS Deployments:**

```bash
SUBNET=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.subnet.filters[0].values[0]}')
AVAILABILITYZONE=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.placement.availabilityZone}')
REGION=$(oc get machineset.machine.openshift.io -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.placement.region}')
```

**For Azure Red Hat OpenShift (ARO) Deployments:**

```bash
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

#### Step 3: Validate Environment Variables

Verify that all required environment variables are set:

```bash
echo "Infrastructure Name: $INFRA_NAME"
echo "Subnet: $SUBNET"
echo "Availability Zone: $AVAILABILITYZONE"
echo "Region: $REGION"

# For Azure deployments, also check:
echo "Image SKU: $IMAGESKU"
echo "Image Version: $IMAGEVERSION"
echo "Resource Group: $RESOURCEGROUP"
echo "Network Resource Group: $NETWORKRESOURCEGROUP"
echo "Virtual Network: $VNET"
echo "Load Balancer: $LOADBALANCER"
```

#### Step 4: Deploy GPU Machine Set

**For AWS (OpenShift Container Platform):**

```bash
helm install gpu-machine-set gpu-machine-set \
  --set infraName=$INFRA_NAME \
  --set subnet=$SUBNET \
  --set availabilityZone=$AVAILABILITYZONE \
  --set region=$REGION
```

**For Azure (Azure Red Hat OpenShift):**

```bash
helm install gpu-machine-set gpu-machine-set \
  --set infraName=$INFRA_NAME \
  --set subnet=$SUBNET \
  --set availabilityZone=$AVAILABILITYZONE \
  --set region=$REGION \
  --set azure.imageSku=$IMAGESKU \
  --set azure.imageVersion=$IMAGEVERSION \
  --set azure.resourceGroup=$RESOURCEGROUP \
  --set azure.networkResourceGroup=$NETWORKRESOURCEGROUP \
  --set azure.vNet=$VNET \
  --set azure.loadBalancer=$LOADBALANCER
```

#### Step 5: Install GPU Support Operators

Install the following operators through the OpenShift Operator Hub:

1. **Node Feature Discovery by Red Hat**
   - After installation, create a default `NodeFeatureDiscovery` object

2. **NVIDIA GPU Operator**
   - After installation, create a default `ClusterPolicy` object

### OpenShift AI Installation

Complete the setup by installing the OpenShift AI operator:

1. **OpenShift AI Operator**
   - Install through the Operator Hub
   - Create a default Data Science Cluster (DSC) object

## ✅ Validation

After completing the installation, verify that all components are running correctly:

```bash
# Check operator status
oc get csv -A | grep -E "(pipelines|mesh|serverless|ai)"

# Verify GPU nodes (if GPU was configured)
oc get nodes -l feature.node.kubernetes.io/pci-10de.present=true

# Check Data Science Cluster status
oc get dsc -A
```

## 🔍 Troubleshooting

### Common Issues

- **Environment variables not set**: Ensure you're in the correct OpenShift context and have proper permissions
- **Helm installation fails**: Verify network connectivity and operator prerequisites
- **GPU nodes not appearing**: Check machine set status and cloud provider quotas

### Getting Help

If you encounter issues:

1. Check the OpenShift console for operator status
2. Review pod logs for specific error messages
3. Consult the [Red Hat OpenShift AI documentation](https://docs.redhat.com/en/documentation/red_hat_openshift_ai/)

## 🤝 Contributing

Contributions are welcome! Please feel free to submit issues, feature requests, or pull requests to improve this installation guide.

## 📄 License

This project is part of Red Hat OpenShift AI workshop materials. Please refer to the appropriate Red Hat documentation for licensing information.

---

**Note**: This installation guide is designed for workshop and demonstration purposes. For production deployments, please consult the official Red Hat OpenShift AI documentation and follow enterprise best practices.