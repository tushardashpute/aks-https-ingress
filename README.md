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


**1. Create AKS cluster**

        # Create a resource group
        az group create --name myResourceGroup --location eastus
        
        # Create an AKS cluster
        az aks create --resource-group myResourceGroup --name myAKSCluster  --node-count 1 --generate-ssh-keys
        
        # Connect to the cluster
        az aks get-credentials --resource-group myResourceGroup --name myAKSCluster

**2. Create nginx ingress controller**

        # Create a namespace for ingress resources
        kubectl create namespace public-ingress
        
        # Add the Helm repository
        helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
        helm repo update
        
        # Use Helm to deploy an NGINX ingress controller
        helm upgrade --install ingress-nginx ingress-nginx \
          --repo https://kubernetes.github.io/ingress-nginx \
          --namespace public-ingress \
          --set controller.config.http2=true \
          --set controller.config.http2-push="on" \
          --set controller.config.http2-push-preload="on" \
          --set controller.ingressClassByName=true \
          --set controller.ingressClassResource.controllerValue=k8s.io/ingress-nginx \
          --set controller.ingressClassResource.enabled=true \
          --set controller.ingressClassResource.name=public \
          --set controller.service.externalTrafficPolicy=Local \
          --set controller.setAsDefaultIngress=true
  
3. Install cert-manager for SSL certificates in public-ingress namespace using Helm.

4. Create a CA cluster issuer for issuing certificates.

5. Create sample application and service.

6. Setup A Record of domain

7. Create an ingress route to configure the rules that route traffic to one of the two applications.

8. Verify the automatic created certificate.

9. Test the applications using Custom Domain.
