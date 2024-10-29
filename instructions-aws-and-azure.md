# Kubernetes on AWS and Azure

Highlights
- This is a bonus section added in 6 months after the course release
- RECOMMENDED APPROACH - First Learn with GCP and then come here!
- 99% of things with Kubernetes remain the same in AWS, GCP and Azure.
- ISTIO and HELM would be the same in all clouds. So, we will not be worry about them here!
- Download Github repo again!

## AWS

Prerequisites
- Install and Configure AWS CLI `aws configure`
- Install EKSCTL - https://eksctl.io/introduction/installation/ `eksctl version`
- EKS is not free. t2.medium is not in the free tier!


### Create EKS Cluster

Help for eksctl https://eksctl.io/

```
eksctl create cluster --name in28minutes-cluster --nodegroup-name in28minutes-cluster-node-group  --node-type t2.medium --nodes 3 --nodes-min 3 --nodes-max 7 --managed --asg-access

OR 

RECOMENDED : 
cd kubernetes-learning folder and run :
eksctl create cluster -f cluster-aws.yaml
aws eks update-kubeconfig --region us-east-1 --name robin-cluster
```

If you get this error
```
AWS::EKS::Cluster/ControlPlane: CREATE_FAILED – "Cannot create cluster 
'in28minutes-dev-cluster' because us-east-1e, the targeted availability zone, does not 
currently have sufficient capacity to support the cluster. Retry and choose from these 
availability zones: us-east-1a, us-east-1b, us-east-1c, us-east-1d, us-east-1f (Service: 
	AmazonEKS; Status Code: 400; Error Code: UnsupportedAvailabilityZoneException; Request 
	ID: a5580928-689d-4558-b3bd-2573131ec69e)
```

Add Availability Zones

```
--zones=us-east-1a,us-east-1b
```

Things to Note
- VPCs and SubNets are Created

### Deploying Basic Applications

```
kubectl create deployment hello-world-rest-api --image=in28min/hello-world-rest-api:0.0.1.RELEASE
kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080

kubectl create deployment todowebapp-h2 --image=in28min/todo-web-application-h2:0.0.1-SNAPSHOT
kubectl expose deployment todowebapp-h2 --type=LoadBalancer --port=8080

cd /Ranga/git/01.udemy-course-repos/kubernetes-crash-course/03-todo-web-application-mysql/backup/02-final-backup-at-end-of-course 

kubectl apply -f mysql-database-data-volume-persistentvolumeclaim-aws.yaml,mysql-deployment.yaml,mysql-service.yaml

kubectl apply -f config-map.yaml,secret.yaml,todo-web-application-deployment.yaml,todo-web-application-service.yaml

echo -n dummytodos | base64
```

### Delete Basic Applications

```
kubectl delete all -l app=hello-world-rest-api
kubectl delete all -l app=todowebapp-h2
kubectl delete all -l io.kompose.service=todo-web-application
kubectl delete all -l io.kompose.service=mysql
```

### Deploy Basic Microservices

```
kubectl apply -f 04-currency-exchange-microservice-basic/deployment.yaml 
kubectl apply -f 05-currency-conversion-microservice-basic/deployment.yaml
```

### Adding Ingress

- https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html
- Check out sample application - https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-ingress.yaml

```
Follow the Blog : https://awswithatiq.com/how-to-create-an-nginx-ingress-controller-with-aws-classic-load-balancer/

if you created new cluster then run bellow CMD :
kubectl apply -f 04-currency-exchange-microservice-basic/deployment.yaml 
kubectl apply -f 05-currency-conversion-microservice-basic/deployment.yaml

kubectl apply -f 05-currency-conversion-microservice-basic/ingress_aws.yaml
```

### Adding Application Insights

- https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-EKS-quickstart.html
- Attach CloudWatchAgentServerPolicy to the nodes

### Exploring Cluster Auto Scaler

- https://katharharshal1.medium.com/kubernetes-cluster-autoscaling-ca-using-aws-eks-4aab8c89f9a1
Testing auto scaler
```
kubectl create deployment autoscaler-demo --image=nginx
kubectl scale deployment autoscaler-demo --replicas=50
```

### Clean up the Cluster

https://docs.aws.amazon.com/eks/latest/userguide/delete-cluster.html

Delete all LoadBalancer services
```
kubectl get svc --all-namespaces
kubectl delete svc service-name
```

Delete the cluster
```
eksctl delete cluster --name in28minutes-cluster
```

## Azure

### Create AKS Cluster

Let's use Azure Cloud Shell

```
# az login # az cli

az group create --name kubernetes-resource-group --location westeurope

az ad sp create-for-rbac --skip-assignment --name kubernetes-cluster-service-principal

az aks create --name robin-cluster --node-count 4 --enable-addons monitoring --resource-group kubernetes-resource-group --vm-set-type VirtualMachineScaleSets --load-balancer-sku standard --enable-cluster-autoscaler  --min-count 1 --max-count 7  --generate-ssh-keys --service-principal <<appId>> --client-secret  <<password>>

if you see error saying : The subscription is not registered to use namespace 'Microsoft.ContainerService'
run :
* az provider register --namespace Microsoft.ContainerService
* az provider show -n Microsoft.ContainerService
* az provider register --namespace Microsoft.OperationsManagement
* https://learn.microsoft.com/en-us/answers/questions/2088453/(errcode-insufficientvcpuquota)-insufficient-regio
* Go to Subscription -> Free Tier -> Resource providers -> register Microsoft.Compute
* Go to Subscription -> Free Tier -> Resource providers -> Usage + quotas : you can see how many vcpu you have for the region

if You have less VCPU:
*go to Resource groups and create resource group with Australia Central and use that resource group in cluster creation
* if this error : Insufficient regional vcpu quota left for location australiacentral. left regional vcpu quota 4, requested quota 8
* modify the node to 1
ex: 
az aks create --name robin-cluster --node-count 4 --enable-addons monitoring --resource-group australial-resource-group --vm-set-type VirtualMachineScaleSets --load-balancer-sku standard --enable-cluster-autoscaler  --min-count 1 --max-count 7  --generate-ssh-keys --service-principal 1f9b64ae-****-****-****-************ --client-secret  _N78Q******************_****


az aks get-credentials --resource-group australial-resource-group --name robin-cluster
```

### Deploying Basic Applications

```
kubectl create deployment hello-world-rest-api --image=in28min/hello-world-rest-api:0.0.1.RELEASE

kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080

kubectl create deployment todowebapp-h2 --image=in28min/todo-web-application-h2:0.0.1-SNAPSHOT

kubectl expose deployment todowebapp-h2 --type=LoadBalancer --port=8080

cd kubernetes-crash-course/03-todo-web-application-mysql/backup/02-final-backup-at-end-of-course 
kubectl apply -f mysql-database-data-volume-persistentvolumeclaim.yaml,mysql-deployment.yaml,mysql-service.yaml

#WAIT FOR MYSQL TO BE UP AND RUNNING

kubectl apply -f config-map.yaml,secret.yaml,todo-web-application-deployment.yaml,todo-web-application-service.yaml
```

### Delete Basic Applications

```
kubectl delete all -l app=hello-world-rest-api
kubectl delete all -l app=todowebapp-h2
kubectl delete all -l io.kompose.service=todo-web-application
kubectl delete all -l io.kompose.service=mysql
```

### Deploy Microservices

```
kubectl apply -f 04-currency-exchange-microservice-basic/deployment.yaml 
kubectl apply -f 05-currency-conversion-microservice-basic/deployment.yaml
# watch -n 0.1 curl
# Explore Logs
```

### Create Ingress

Install Ingress Controller
```
kubectl create serviceaccount nginx-ingress-controller
kubectl create clusterrolebinding nginx-ingress-controller --clusterrole=cluster-admin --serviceaccount=default:nginx-ingress-controller
kubectl create namespace ingress-nginx

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx 
helm repo update

helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-nginx --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-resource-group"="<Your Resurce Group>" --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-internal"="true"

ex:
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-nginx --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-resource-group"="australial-resource-group" --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-internal"="true"

kubectl get pods -n ingress-nginx
NAME                                                      READY   STATUS    RESTARTS   AGE
nginx-ingress-ingress-nginx-controller-64c8ffc67f-2b6vl   1/1     Running   0          34s

kubectl get service  -n ingress-nginx
NAME                                               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
nginx-ingress-ingress-nginx-controller             LoadBalancer   10.0.40.176    10.224.0.5    80:31746/TCP,443:30443/TCP   44s
nginx-ingress-ingress-nginx-controller-admission   ClusterIP      10.0.187.221   <none>        443/TCP                      44s
----------------
Note :
sometime if your ingress is still showing PENDING on EXTERNAL IP,then you need grant permision to your regional group
az role assignment create --assignee <service-principal-id> --role "Network Contributor" --scope /subscriptions/<subscription-id>/resourceGroups/<resource-group-name>

ex: Error syncing load balancer: failed to ensure load balancer: Retriable: false, RetryAfter: 0s, HTTPStatusCode: 403, RawError: {"error":{"code":"AuthorizationFailed","message":"The client '4ed3***-****-****-****-********' with object id '4ed3***-****-****-****-********' does not have authorization to perform action 'Microsoft.Network/publicIPAddresses/read' over scope '/subscriptions/9e******-****-****-****-************/resourceGroups/australial-resource-group/providers/Microsoft.Network' or the scope is invalid. If access was recently granted, please refresh your credentials."}}

This above error was found in when i went to service and Ingress -> clicked on service : nginx-ingress-ingress-nginx-controller ->
inside click events, if you see the above error then grant the permision
-----------------
```

More details - https://docs.microsoft.com/en-us/azure/aks/ingress-tls

Setup Ingress
```
kubectl apply -f 05-currency-conversion-microservice-basic/ingress_azure.yaml

kubectl get ingress
NAME              CLASS    HOSTS   ADDRESS      PORTS   AGE
gateway-ingress   <none>   *       10.224.0.5   80      54s

If URL is Not Hitting then debug
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx nginx-ingress-ingress-nginx-controller-64c8ffc67f-59jn2

For EXTERNAL ADRESS not working Issue run: 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0-beta.0/deploy/static/provider/cloud/deploy.yaml
From this Doc
https://kubernetes.github.io/ingress-nginx/deploy/#azure
```
### Exploring Cluster Auto Scaler

Testing auto scaler
```
kubectl create deployment autoscaler-demo --image=nginx
kubectl scale deployment autoscaler-demo --replicas=50
```

### Clean up the cluster

Delete all LoadBalancer services
```
kubectl get svc --all-namespaces
kubectl delete svc service-name
```

Delete the cluster
```
az aks delete --name robin-cluster --resource-group kubernetes-resource-group
```