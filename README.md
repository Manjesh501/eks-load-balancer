# AWS Load Balancer Controller on EKS with Fargate

This repository provides a guide to setting up the AWS Load Balancer Controller on an Amazon EKS cluster using Fargate. Follow the steps below to deploy  2048 game application using an Application Load Balancer (ALB) on AWS.

## Prerequisites

Before you begin, ensure that you have the following tools installed:

- [AWS CLI](https://aws.amazon.com/cli/)
- [eksctl](https://eksctl.io/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Helm](https://helm.sh/)

## Steps

### 1. Create an EKS Cluster

Create an EKS cluster with Fargate as the compute option:

```bash
eksctl create cluster --name demo-cluster --region us-east-1 --fargate
```

### 2. Associate IAM OIDC Provider

Associate the IAM OIDC provider with your EKS cluster:

```bash
eksctl utils associate-iam-oidc-provider --cluster demo-cluster --approve
```

### 3. Create IAM Policy for AWS Load Balancer Controller

Download and create the IAM policy required for the AWS Load Balancer Controller:

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
```

### 4. Create IAM Service Account

Create the IAM service account for the AWS Load Balancer Controller:

```bash
eksctl create iamserviceaccount --cluster=demo-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy --approve
```

> **Note:** Replace `<your-aws-account-id>` with your actual AWS account ID.

### 5. Create a Fargate Profile

Create a Fargate profile for the `game-2048` namespace:

```bash
eksctl create fargateprofile --cluster demo-cluster --region us-east-1 --name alb-sample-app --namespace game-2048
```

### 6. Add the Helm Repository

Add the EKS Helm repository and update it:

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

### 7. Install AWS Load Balancer Controller

Install the AWS Load Balancer Controller using Helm:

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=demo-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-1 --set vpcId=vpc-0a9d6729f6def08e4
```

### 8. Deploy the Sample Application

Deploy the 2048 game application using the following command:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

## Conclusion

After following these steps, the 2048 game application will be deployed on your EKS cluster, accessible through an Application Load Balancer.
