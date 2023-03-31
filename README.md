# kubernetes-microservices-demo (draft)
This page gives a guide for deploying a microservices demo project (<https://github.com/GoogleCloudPlatform/microservices-demo>) to AWS EKS. The original deployment target is Google Kubernetes Engine (GKE). This guide consists two parts,

1. Deploy the [project](https://github.com/GoogleCloudPlatform/microservices-demo) to AWS EKS
2. Use [Locust](https://locust.io/) tool to perform load test

# 1. Deploy the project to AWS EKS
## 1.1 Prerequisite
1. Install the AWS CLI: [Getting Started](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)
2. Install kubernetes command line tool, kubectl: [Installation Guide](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
3. Install AWS EKS CLI, eksctl: [Installation Guide](https://eksctl.io/introduction/#installation)
4. Clone the project from [GitHub](https://github.com/GoogleCloudPlatform/microservices-demo)

## 1.2 Create EKS cluster
Once all tools are properly installed and the respository is cloned, you can create the EKS cluster with the following command.

`eksctl create cluster --name <cluster name> --region <aws region> --version <kubenertes version> --nodes <number of nodes> --node-type <node machine type>`

Example

`eksctl create cluster --name boutique-online --region ap-southeast-1 --version 1.25 --nodes 3 --node-type t2.small`

Remarks of parameters
- `name`: cluster name
- `version`: support 1.22, 1.23, 1.24, 1.25
- `nodes`: default 2 nodes
- `node-type`: EC2 machine type, default is m5.large

It will take 10 - 15 minutes to complete the process. Last, you will see

`EKS cluster "<cluster name>" in "<aws region>" region is ready`

![Create cluster](images/create-cluster.png)

## 1.3 Verify VPC, Nodes
You can then verfiy the status of the cluster just created. "ACTIVE" should be responsed after running the following command.

`aws eks describe-cluster --name <cluster name> --query cluster.status`

Example

`aws eks describe-cluster --name boutique-online --query cluster.status`

![Cluster Status CLI](images/cluster-status-cli.png)

Then review the status of the kubernetes system pods

`kubectl get pods -n kube-system`

[Todo: insert image of dashboard and command screen when it is ready] CLI

You should notice aws-node, coredns and kube-proxy pods are in `RUNNING` state

## 1.4 Create Namespace
After verification, you can create a namespace for the deployment.

`kubectl create namespace <namespace>`

Example

`kubectl create namespace boutique-online`

## 1.5 Deploy to EKS
Ensure the repository is cloned, change to the `microservices-demo` directory, run the following command for deployment.

`kubectl apply -f release/kubernetes-manifests.yaml -n <namespace>`

Example

`kubectl apply -f release/kubernetes-manifests.yaml -n boutique-online`

It will take 5 - 10 minutes to complete the deployment.

## 1.6 Review the deployment status and frontend external URL
When deployment is completed, you can review the status of pods and services using the following commands.

**Check the status of Pods**

`kubectl get pods -n <namespace>`

Example

`kubectl get pods -n boutique-online`

[Todo: insert sample images]

**Check the status of Services**

`kubectl get services -n <namespace>`

Example

`kubectl get services -n boutique-online`

[Todo: insert sample images]

**Retrieve the frontend external URL**

`kubectl get svc frontend-external -n <namespace>`

Example

`kubectl get svc frontend-external -n boutique-online`

[Todo: insert sample images]
## 1.7 Clean up the deployment and related resources
Lastly, when all deployment and tests are finished, you can clean up the resources accordingly.

First delete the deployment, then delete the cluster. All related resources will be deleted.

`kubectl delete -f kubernetes-manifests.yaml -n <namespace>`

`eksctl delete cluster --name <namespace> --region <aws region>`

Example

`kubectl delete -f kubernetes-manifests.yaml -n boutique-online`

`eksctl delete cluster --name boutique-online --region ap-southeast-1`

# 2. Use Locust tool to perform load test