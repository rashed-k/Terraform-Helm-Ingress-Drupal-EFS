# Creating EKS Cluster from Terraform Code - 

Use the Terraform Folder from the repository which includes all AWS EKS code required for terraform to provision an EKS cluster. (The Files have been taken from https://github.com/hashicorp/terraform-provider-aws, you can also use other examples which suits best for your use case.)
</br > 

Note - The instance will require AWS Cli as it requires to give permission to provision AWS services or else the terraform will provide an output with an error. 

```
terraform init

terraform fmt

terraform validate

terraform plan

terraform apply 

```

# Helm - Nginx Ingress Controller 

You can use the following command once helm is installed to deploy nginx ingress controller workloads (source -https://kubernetes.github.io/ingress-nginx/deploy/ ) -

```
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace

```
# Helm - Drupal   (bitnami/drupal)

Use the following commands to pull the latest helm repository of Drupal application by Bitnami - 

```
helm pull bitnami/drupal

tar -zxvf drupal-12.2.11.tgz
``` 
</b > 

Once the file is pulled and unzipped, add ingress configuartion into values.yaml file inside drupal folder. 

```
ingress:
  ## @param ingress.enabled Enable ingress controller resource
  ##
  enabled: true 
  ## DEPRECATED: Use ingress.annotations instead of ingress.certManager
  ## certManager: false
  ##

  ## @param ingress.pathType Ingress Path type
  ##
  pathType: ImplementationSpecific
  ## @param ingress.apiVersion Override API Version (automatically detected if not set)
  ##
  apiVersion: ""
  ## @param ingress.ingressClassName IngressClass that will be be used to implement the Ingress (Kubernetes 1.18+)
  ## This is supported in Kubernetes 1.18+ and required if you have more than one IngressClass marked as the default for your cluster .
  ## ref: https://kubernetes.io/blog/2020/04/02/improvements-to-the-ingress-api-in-kubernetes-1.18/
  ##
  ingressClassName: "nginx"
  ## @param ingress.hostname Default host for the ingress resource
  ##
  hostname: www.funapp.click
  ## @param ingress.path The Path to Drupal. You may need to set this to '/*' in order to use this
  ## with ALB ingress controllers.
  ##
  path: /
  ## @param ingress.annotations Additional annotations for the Ingress resource. To enable certificate autogeneration, place here your cert-manager annotations.
  ## For a full list of possible ingress annotations, please see
  ## ref: https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/annotations.md
  ## Use this parameter to set the required annotations for cert-manager, see
  ## ref: https://cert-manager.io/docs/usage/ingress/#supported-annotations

```

As the helm chart provided by bitnami has Type Loadbalancer for Drupal Service, If your use case needs ingress controller to control the traffic then based on this particular use case change the type = Loadbalancer - type = ClusterIP in the values.yaml file - 

```

## Kubernetes configuration. For minikube, set this to NodePort, elsewhere use LoadBalancer
##
service:
  ## @param service.type Kubernetes Service type
  ##
  type: ClusterIP
  ## @param service.ports.http Service HTTP port
  ## @param service.ports.https Service HTTPS port
  ##
  ports:
    http: 80
    https: 443
  ## @param service.loadBalancerSourceRanges Restricts access for LoadBalancer (only with `service.type: LoadBalancer`)
  ## e.g:
  ## loadBalancerSourceRanges:
  ##   - 0.0.0.0/0
  ##
  loadBalancerSourceRanges: []
  ## @param service.loadBalancerIP loadBalancerIP for the Drupal Service (optional, cloud specific)
  ## ref: https://kubernetes.io/docs/user-guide/services/#type-loadbalancer
  loadBalancerIP: ""
  ## @param service.nodePorts [object] Kubernetes node port
  ## nodePorts:
  ##   http: <to set explicitly, choose port between 30000-32767>
  ##   https: <to set explicitly, choose port between 30000-32767>
  ##

```





