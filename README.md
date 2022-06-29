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
  enabled: true   <-------------------- TRUE
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
  ingressClassName: "nginx"    <-----------------NGINX
  ## @param ingress.hostname Default host for the ingress resource
  ##
  hostname: www.funapp.click    <--------------- Host
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

As the helm chart provided by bitnami has Type Loadbalancer for Drupal Service, If your use case needs ingress controller to control external traffic from external loadbalancer then based on this particular use case change the type = Loadbalancer - type = ClusterIP in the values.yaml file - 

```

## Kubernetes configuration. For minikube, set this to NodePort, elsewhere use LoadBalancer
##
service:
  ## @param service.type Kubernetes Service type
  ##
  type: ClusterIP   <-------------------- Change Type from Loadbalancer to ClusterIP
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

From the chart pulled from bitnami/drupal, the persistence volume claim is configured to storageclass = gp2 (EBS) and accessmodes = ReadWriteOnce. Due to this configuration the pods of drupal application will not be able to scale up, as the requirement for scaling will need the following configuration - 

```
persistence:
  ## @param persistence.enabled Enable persistence using PVC
  ##
  enabled: true   
  ## @param persistence.storageClass PVC Storage Class for Drupal volume
  ## If defined, storageClassName: <storageClass>
  ## If set to "-", storageClassName: "", which disables dynamic provisioning
  ## If undefined (the default) or set to null, no storageClassName spec is
  ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
  ##   GKE, AWS & OpenStack)
  ##
  storageClass: "efs"       <------------------------------- change to efs
  ## @param persistence.accessModes PVC Access Mode for Drupal volume
  ## Requires persistence.enabled: true
  ## If defined, PVC must be created manually before volume will be bound
  ##
  accessModes:
    - ReadWriteMany        <--------------------------------- change to ReadWriteMany
  ## @param persistence.size PVC Storage Request for Drupal volume
  ##
  size: 8Gi
  ## @param persistence.existingClaim A manually managed Persistent Volume Claim
  ## Requires persistence.enabled: true
  ## If defined, PVC must be created manually before volume will be bound
  ##
  existingClaim: ""
  ## @param persistence.hostPath If defined, the drupal-data volume will mount to the specified hostPath.
  ## Requires persistence.enabled: true
  ## Requires persistence.existingClaim: nil|false
  ## Default: nil.
```

# AWS EFS configuration

Create EFS

- Name - EFS-Drupal  (Can be any name)
- Virtual Private Cloud (VPC) - *EKS cluster VPC must be selected
- Availability and durability - Regional or One Zone (Select according to use case)

</br >

Once EFS is created, copy any one IP address of file system in network. (It's always better do select the one in same availability zone.) Add IP addess in deployment.yaml file inside efs-privisoner folder

From the repository, deploy workloads for efs-provisioner from efs-provisioner and 

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  namespace: storage
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
      - name: nfs-client-provisioner
        image: k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
        volumeMounts:
        - name: nfs-client-root
          mountPath: /persistentvolumes
        env:
        - name: PROVISIONER_NAME
          value: k8s-sigs.io/nfs-subdir-external-provisioner
        - name: NFS_SERVER
          value: 10.0.0.1                 <--------- Change the Value of IP address according to your EFS network
        - name: NFS_PATH
          value: /
      volumes:
      - name: nfs-client-root
        nfs:
          server: 10.0.0.1               <--------- Change the Value of IP address according to your EFS network
          path: /
```










