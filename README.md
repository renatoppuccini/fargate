# Goal
The goal of this document is to describe how Red Hat Advanced Cluster Management for Kubernetes can manage an EKS cluster with Fargate deployment.
It walks through the EKS + Fargate clsuter creation, app deployment, FargateProfile for the app deployment, Options for ACM deployment and finally ACM management/monitoring.

# Deploy EKS Fargate Cluster

```
% eksctl create cluster --name my-cluster --region us-east-2 --fargate

2023-09-26 11:11:18 [ℹ]  eksctl version 0.146.0
2023-09-26 11:11:18 [ℹ]  using region us-east-2
2023-09-26 11:11:18 [ℹ]  setting availability zones to [us-east-2a us-east-2b us-east-2c]
...
2023-09-26 11:25:05 [✔]  EKS cluster "my-cluster" in "us-east-2" region is ready
```

# After cluster is installed, check the nodes using the `kubectl` tool.

```
% kubectl get nodes
NAME                                                    STATUS   ROLES    AGE     VERSION
fargate-ip-192-168-145-122.us-east-2.compute.internal   Ready    <none>   7m17s   v1.25.8-eks-f4dc2c0
fargate-ip-192-168-181-231.us-east-2.compute.internal   Ready    <none>   7m16s   v1.25.8-eks-f4dc2c0
```
# Create the sample app namespace
```
% kubectl create namespace eks-sample-app
namespace/eks-sample-app created
```

# Create Fargate profile with the app namespace in it.
```
% eksctl create fargateprofile --cluster my-cluster --name test-fargate --namespace eks-sample-app         
2023-09-26 11:39:43 [ℹ]  creating Fargate profile "test-fargate" on EKS cluster "my-cluster"
2023-09-26 11:40:16 [ℹ]  created Fargate profile "test-fargate" on EKS cluster "my-cluster"
```

# Deploy the app.
```
% kubectl apply -f eks-sample-app.yaml 
deployment.apps/eks-sample-linux-deployment created
```

# Deploy the service (LB).
```
% kubectl apply -f eks-sample-service.yaml
service/eks-sample-linux-service created
```
# Deploy ACM
In order to deploy ACM pods to manage/monitor the EKS cluster with Fargate, it is needed to create nodePool or a fargateProfile. 
One or the other works, it does need to be both.

## AWS documentation on how to create NodePools:
https://docs.aws.amazon.com/eks/latest/userguide/create-managed-node-group.html
```
eksctl create nodegroup \
  --cluster my-cluster-name \
  --region region-code \
  --name my-mng \
  --node-type m5.large \
  --nodes 3
```
```
% eksctl create nodegroup --cluster my-cluster --region us-east-2 --name acm-group --node-type m5.xlarge --nodes 2
2023-09-26 13:06:51 [ℹ]  will use version 1.25 for new nodegroup(s) based on control plane version
2023-09-26 13:06:52 [ℹ]  nodegroup "acm-group" will use "" [AmazonLinux2/1.25]
2023-09-26 13:06:52 [ℹ]  1 nodegroup (acm-group) was included (based on the include/exclude rules)
2023-09-26 13:06:52 [ℹ]  will create a CloudFormation stack for each of 1 managed nodegroups in cluster "my-cluster"
...
...
2023-09-26 13:10:33 [✔]  created 0 nodegroup(s) in cluster "my-cluster"
...
...
2023-09-26 13:10:33 [✔]  created 1 managed nodegroup(s) in cluster "my-cluster"
2023-09-26 13:10:33 [ℹ]  checking security group configuration for all nodegroups
2023-09-26 13:10:33 [ℹ]  all nodegroups have up-to-date cloudformation templates
```

## Creating Fargate Profile with 2 namespaces via the AWS Web Console:
https://docs.aws.amazon.com/eks/latest/userguide/fargate-profile.html#create-fargate-profile

### the following namespaces should be added to the fargate profile if used:
```
open-cluster-management-agent
open-cluster-management-agent-addon
```
![Screen Shot 2023-09-26 at 4 25 23 PM](https://github.com/renatoppuccini/fargate/assets/1215178/8df350c2-7792-4d71-8240-45ef3637a07c)
![Screen Shot 2023-09-26 at 4 25 36 PM](https://github.com/renatoppuccini/fargate/assets/1215178/dae98d61-92bb-4471-a582-032ba5ea10b6)


# RESULTS
After the ACM is deployed, it should look like this:
## ACM Detects Fargate nodes:
![image (3)](https://github.com/renatoppuccini/fargate/assets/1215178/43b6e51c-c7b8-4bea-9112-dba3b97826c9)
![image (4)](https://github.com/renatoppuccini/fargate/assets/1215178/dcdaa06b-6b81-40d9-b90b-c06cd3aff998)

## ACM detects Fargate Workload:
![image (5)](https://github.com/renatoppuccini/fargate/assets/1215178/22b6be12-eae7-4130-9699-57fdc47d4962)

