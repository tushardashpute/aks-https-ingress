# aks-https-ingress

This guide demonstrates the process of setting up the NGINX ingress controller within an Azure Kubernetes Service (AKS) cluster. The ingress controller is specifically configured to utilize a fixed public IP address via an Azure Standard Load Balancer. To handle certificate management, the cert-manager project is deployed, automating the generation and setup of Let’s Encrypt certificates. Furthermore, it outlines the steps to integrate a custom domain with a certificate, allowing the application to run publicly.

Prerequisites:
--------------
You must have your domain registered with you (This is required to create a custom SSL certificates)

Steps:
-----
1. Create an AKS cluster
2. Create nginx ingress controller
3. Install cert-manager for SSL certificates in public-ingress namespace using Helm.
4. Create a CA cluster issuer for issuing certificates.
5. Create a sample application and service.
6. Setup A Record of the domain
7. Create an ingress route to configure the rules that route traffic to one of the two applications.
8. Verify the automatically created certificate.
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

  ![image](https://github.com/tushardashpute/aks-https-ingress/assets/74225291/5b4dc9ea-d8f1-4735-8849-f416bd5e0ecb)

3. Install cert-manager for SSL certificates in public-ingress namespace using Helm.

        # Label the cert-manager namespace to disable resource validation
        kubectl label namespace public-ingress cert-manager.io/disable-validation=true
        # Add the Jetstack Helm repository
        helm repo add jetstack https://charts.jetstack.io
        # Update your local Helm chart repository cache
        helm repo update
        # Install CRDs with kubectl
        kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.crds.yaml
        # Install the cert-manager Helm chart
        helm install cert-manager jetstack/cert-manager \
          --namespace public-ingress \
          --version v1.11.0

![image](https://github.com/tushardashpute/aks-https-ingress/assets/74225291/96bffdae-2ca7-4b35-b23e-da907b11477e)

4. Create a CA cluster issuer for issuing certificates.

        # Cluster Issuer
        apiVersion: cert-manager.io/v1
        kind: ClusterIssuer
        metadata:
          name: letsencrypt-production
        spec:
          acme:
            server: https://acme-v02.api.letsencrypt.org/directory
            email: #Use your mail id
            privateKeySecretRef:
              name: letsencrypt-production
            solvers:
              - http01:
                  ingress:
                    class: public

To create the issuer, use the kubectl command.

![image](https://github.com/tushardashpute/aks-https-ingress/assets/74225291/f4d2c46d-0b8f-4fe7-93fa-03f787fa04e9)

5. Create sample application and service.

        kubectl apply -f spring_deploy.yaml -n public-ingress
        kubectl apply -f spring_svc.yaml -n public-ingress

![image](https://github.com/tushardashpute/aks-https-ingress/assets/74225291/40154819-253b-40cc-b2d1-5dba400ef57e)

7. Setup A Record of domain

A record of custom domain will be the Public IP of Ingress Controller.

        kubectl get svc -n public-ingress

![image](https://github.com/tushardashpute/aks-https-ingress/assets/74225291/792596a1-914e-4a4c-aa0d-a226570fa7b2)

Copy this external ip of nginx controller and create A-Record for you domain.

![image](https://github.com/tushardashpute/aks-https-ingress/assets/74225291/9ac31865-82f6-4ae0-9633-eac5122681d2)

8. Create an ingress route to configure the rules that route traffic to one of the two applications.

        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
          name: ingress
          annotations:
            cert-manager.io/cluster-issuer: letsencrypt-production
            nginx.ingress.kubernetes.io/rewrite-target: /$1
            nginx.ingress.kubernetes.io/use-regex: "true"
        spec:
          ingressClassName: public
          tls:
          - hosts:
            - spring.tdashpute.online #Use your domain
            secretName: tls-secret
          rules:
          - host: spring.tdashpute.online #Use your domain
            http:
              paths:
              - path: /(.*)
                pathType: Prefix
                backend:
                  service:
                    name: springboot
                    port:
                      number: 80


        kubectl apply -f spring_ingress.yaml -n public-ingress

![image](https://github.com/tushardashpute/aks-https-ingress/assets/74225291/fd03affd-f69b-4fb1-8a88-070ebe00215f)

If you get this acem-solver ingresss the edit this ingress and add ingressClassName in spec section.

![image](https://github.com/tushardashpute/aks-https-ingress/assets/74225291/328d2a81-698d-4a18-b789-4d9bfecee435)

![image](https://github.com/tushardashpute/aks-https-ingress/assets/74225291/32011967-3e9d-4a87-969c-623d1c98a010)

![image](https://github.com/tushardashpute/aks-https-ingress/assets/74225291/57e3f39a-9542-4692-8415-5d09d09d1345)

10. Verify the automatically created certificate.

To verify that the certificate was created successfully, use the below command

![image](https://github.com/tushardashpute/aks-https-ingress/assets/74225291/9a3fb19e-4ace-4f53-a9c6-0846accb47f7)

11. Test the applications using Custom Domain.

Open a web browser to the FQDN of your Kubernetes ingress controller, such as https://spring.tdashpute.online/listallcustomers

![image](https://github.com/tushardashpute/aks-https-ingress/assets/74225291/5130df29-e40c-4139-a2f8-e1127a96c004)

Now the applications are secured using TLS certificate and are reachable using the custom domain.

![image](https://github.com/tushardashpute/aks-https-ingress/assets/74225291/2b83dd44-c4d6-4411-b221-68036328468d)
