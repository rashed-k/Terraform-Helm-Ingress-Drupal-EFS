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








