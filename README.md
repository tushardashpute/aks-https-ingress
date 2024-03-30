# aks-https-ingress

Prerequisites:
--------------
You must have your domain register with you (This is required to create a custom SSL certificates)

Steps:
-----
1. Create AKS cluster
2. Create nginx ingress controller
3. Install cert-manager for SSL certificates in public-ingress namespace using Helm.
4. Create a CA cluster issuer for issuing certificates.
5. Create sample application and service.
6. Setup A Record of domain
7. Create an ingress route to configure the rules that route traffic to one of the two applications.
8. Verify the automatic created certificate.
9. Test the applications using Custom Domain.


1. Create AKS cluster

        # Create a resource group
        az group create --name myResourceGroup --location eastus
        
        # Create an AKS cluster
        az aks create --resource-group myResourceGroup --name myAKSCluster  --node-count 1 --generate-ssh-keys
        
        # Connect to the cluster
        az aks get-credentials --resource-group myResourceGroup --name myAKSCluster

2. Create nginx ingress controller

3. Install cert-manager for SSL certificates in public-ingress namespace using Helm.

4. Create a CA cluster issuer for issuing certificates.

5. Create sample application and service.

6. Setup A Record of domain

7. Create an ingress route to configure the rules that route traffic to one of the two applications.

8. Verify the automatic created certificate.

9. Test the applications using Custom Domain.
