# Part 10: Deploying microservice applications in AKS (AzCLI + Docker + kustomize + kubectl + Azure Pipeline)

    Part1:   Manual Deployment (AzCLI + Docker Desktop + kubectl)  
    GitHub:  https://github.com/santosh-gh/k8s-01
    YouTube: https://youtu.be/zoJ7MMPVqFY

    Part2:   Automated Deployment (AzCLI + Docker + kubect + Azure Pipeline)
    GitHub:  https://github.com/santosh-gh/k8s-02
    YouTube: https://youtu.be/nnomaZVHg9I

    Part3:   Automated Infra Deployment (Bicep + Azure Pipeline)
    GitHub:  https://github.com/santosh-gh/k8s-03
    YouTube: https://www.youtube.com/watch?v=5PAdDPHn8F8

    Part4:   Manual Deployment (AzCLI + Docker Desktop + Helm charts + kubectl) 
    GitHub:  https://github.com/santosh-gh/k8s-04
    YouTube: https://www.youtube.com/watch?v=VAiR3sNavh0

    Part5:   Automated Deployment (AzCLI + Docker + Helm charts + kubectl + Azure Pipeline) 
    GitHub:  https://github.com/santosh-gh/k8s-04
    YouTube: https://www.youtube.com/watch?v=MnWe2KGRrxg&t=883s

    Part6:   Automated Deployment (AzCLI + Docker + Helm charts + kubectl + Azure Pipeline) 
             Dynamically update the image tag in values.yaml
    GitHub:  https://github.com/santosh-gh/k8s-06
    YouTube: https://www.youtube.com/watch?v=Nx0defm8T6g&t=11s

    Part7:   Automated Deployment (AzCLI + Docker + Helm charts + kubectl + Azure Pipeline)
             Store the helm chart in ACR
             Dynamically update the image tag in values.yaml
             Dynamically update the Chart version in Chart.yaml

    GitHub:  https://github.com/santosh-gh/k8s-07
    YouTube: https://www.youtube.com/watch?v=VAiR3sNavh0

    Part8:   Automated Deployment (AzCLI + Docker + Helm charts + kubectl + Azure Pipeline)
             Store the helm chart in ACR
             Dynamically update the image tag in values.yaml
             Dynamically update the Chart version in Chart.yaml
             Deploy into multiple environments (dev, test, prod) with approval gates

    GitHub:  https://github.com/santosh-gh/k8s-08
    YouTube: https://www.youtube.com/watch?v=VAiR3sNavh0

    Part9:   Manual Deployment (AzCLI + Docker + kustomize + kubectl)          
             Deploy into multiple environments (dev, test, prod) command line

    GitHub:  https://github.com/santosh-gh/k8s-09
    YouTube: https://www.youtube.com/watch?v=VAiR3sNavh0

    Part10:  Automated Deployment (AzCLI + Docker + kustomize + kubectl + Azure Pipeline)          
             Deploy into multiple environments (dev, test, prod) command line

    GitHub:  https://github.com/santosh-gh/k8s-10
    YouTube: https://www.youtube.com/watch?v=VAiR3sNavh0

# Architesture

![Store Architesture](aks-store-architecture.png)

    # Store front:      Web application for customers to view products and place orders.
    # Product service:  Shows product information.
    # Order service:    Places orders.
    # RabbitMQ Server:  Message queue for an order queue.


# Directory Structure

![Directory Structure](image.png)

# Tetechnology Stack

    Azure Pipelines
    Infra (AzCLI/Bicep)
    AKS
    ACR
    docker
    AzCLI
    kustomize
    kubectl    

# Steps

    1. Infra deployment:
    
       # Manual (Run from comand line)
         AzCLI: ./infra/azcli/script.sh
         Bicep: az deployment sub create --location uksouth --template-file ./infra/bicep/main.bicep --parameters ./infra/bicep/main.bicepparam 

       # Pipeline (Configure and run in Azure DevOps)
         AzCLI: azcli-infra-pipeline.yml
         Bicep: bicep-infra-pipeline.yml


       # Login to Azure

         az login
         az account set --subscription=<subscriptionId>
         az account show

       # Show existing resources

         az resource list

       # Create RG, ACR and AKS

       # AzCLI
         ./infra/azcli/script.sh

         OR

       # Bicep
         az deployment sub create --location uksouth --template-file ./infra/bicep/main.bicep --parameters ./infra/bicep/main.bicepparam

       # Connect to cluster

         RESOURCE_GROUP="rg-onlinestore-dev-uksouth-001"
         AKS_NAME="aks-onlinestore-dev-uksouth-001"
         az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_NAME --overwrite-existing

       # Short name for kubectl
         alias k=kubectl

       # Create name spaces
         k create ns dev
         k create ns test
         k create ns prod    

       # Show all existing objects
         k get all   

    2. Build and push images to ACR: 
       # CI Pipelines
         order-pipeline.yml
         product-pipeline.yml
         store-front-pipeline.yml

    3. Kustomize
       # Review
         kubectl kustomize ./manifests/config/base/
         kubectl kustomize ./manifests/rabbitmq/overlays/dev
         kubectl kustomize ./manifests/order/overlays/dev
         kubectl kustomize ./manifests/product/overlays/dev
         kubectl kustomize ./manifests/store-front/overlays/dev

         kubectl kustomize ./manifests/config/base/
         kubectl kustomize ./manifests/rabbitmq/overlays/test
         kubectl kustomize ./manifests/order/overlays/test
         kubectl kustomize ./manifests/product/overlays/test
         kubectl kustomize ./manifests/store-front/overlays/test

         kubectl kustomize ./manifests/config/base/
         kubectl kustomize ./manifests/rabbitmq/overlays/prod
         kubectl kustomize ./manifests/order/overlays/prod
         kubectl kustomize ./manifests/product/overlays/prod
         kubectl kustomize ./manifests/store-front/overlays/prod

       # Apply
         kubectl apply -k ./manifests/config/base/ -n dev
         kubectl apply -k ./manifests/rabbitmq/overlays/dev -n dev
         kubectl apply -k ./manifests/order/overlays/dev -n dev
         kubectl apply -k ./manifests/product/overlays/dev -n dev
         kubectl apply -k ./manifests/store-front/overlays/dev -n dev

         kubectl apply -k ./manifests/config/base/ -n test
         kubectl apply -k ./manifests/rabbitmq/overlays/test -n test
         kubectl apply -k ./manifests/order/overlays/test -n test
         kubectl apply -k ./manifests/product/overlays/test -n test
         kubectl apply -k ./manifests/store-front/overlays/test -n test

         kubectl apply -k ./manifests/config/base/ -n prod
         kubectl apply -k ./manifests/rabbitmq/overlays/prod -n prod
         kubectl apply -k ./manifests/order/overlays/prod -n prod
         kubectl apply -k ./manifests/product/overlays/prod -n prod
         kubectl apply -k ./manifests/store-front/overlays/prod -n prod    

    4. App deployment
       # CD Pipelines
         order-pipeline.yml
         product-pipeline.yml
         store-front-pipeline.yml

    5. Validate and Access the application

    6. Clean the Azure resources
    
# Verify the Deployment

    k get pods
    k get services
    curl <LoadBalancer public IP>:80
    Browse the app using http://<LoadBalancer public IP>:80

# Clean the k8s namespace

    k delete all --all -n default

# Clean the Azure resources

    az group delete --name rg-onlinestore-dev-uksouth-001 --yes --no-wait
