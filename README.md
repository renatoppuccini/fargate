# Deploy EKS Fargate Cluster

```
% eksctl create cluster --name my-cluster --region us-east-2 --fargate

2023-09-26 11:11:18 [ℹ]  eksctl version 0.146.0
2023-09-26 11:11:18 [ℹ]  using region us-east-2
2023-09-26 11:11:18 [ℹ]  setting availability zones to [us-east-2a us-east-2b us-east-2c]
...
2023-09-26 11:25:05 [✔]  EKS cluster "my-cluster" in "us-east-2" region is ready
```

After cluster is installed, check the nodes using the `kubectl` tool.

```
% kubectl get nodes
NAME                                                    STATUS   ROLES    AGE     VERSION
fargate-ip-192-168-145-122.us-east-2.compute.internal   Ready    <none>   7m17s   v1.25.8-eks-f4dc2c0
fargate-ip-192-168-181-231.us-east-2.compute.internal   Ready    <none>   7m16s   v1.25.8-eks-f4dc2c0
```
Create the sample app namespace
```
% kubectl create namespace eks-sample-app
namespace/eks-sample-app created
```

Create profile with the namespace in it.
```
% eksctl create fargateprofile --cluster my-cluster --name test-fargate --namespace eks-sample-app         
2023-09-26 11:39:43 [ℹ]  creating Fargate profile "test-fargate" on EKS cluster "my-cluster"
2023-09-26 11:40:16 [ℹ]  created Fargate profile "test-fargate" on EKS cluster "my-cluster"
```

Deploy the app.
```
% kubectl apply -f eks-sample-app.yaml 
deployment.apps/eks-sample-linux-deployment created
```

Deploy the service (LB).
```
% kubectl apply -f eks-sample-service.yaml
service/eks-sample-linux-service created
```
In order to deploy ACM pods to manage/monitor the EKS cluster, it is needed to create nodePool or a fargateProfile for ACM. one or the other works, it does need to be both.

Creating NodePools:
https://docs.aws.amazon.com/eks/latest/userguide/create-managed-node-group.html

Creating Fargate Profile with 2 namespaces via the AWS Web Console:
https://docs.aws.amazon.com/eks/latest/userguide/fargate-profile.html#create-fargate-profile

the following namespaces should be added to the fargate profile if used:
```
open-cluster-management-agent
open-cluster-management-agent-addon
```

