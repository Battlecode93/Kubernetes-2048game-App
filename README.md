# 2048 Gaming App on Kubernetes Cluster using Fargate

This project demonstrates the deployment of the popular "2048" game on an AWS EKS cluster using Fargate for serverless Kubernetes. The project includes setting up an EKS cluster, configuring Kubernetes access, creating a Fargate profile, deploying the application, and managing traffic using an Application Load Balancer (ALB) controller.


## Architecture
The application is deployed on an Amazon EKS cluster using Fargate, a serverless compute engine for Kubernetes. An ALB is used to manage inbound traffic, with Kubernetes Ingress resources to route traffic to the appropriate services.

## Setup and Deployment

### 1. Create EKS Cluster
Create the EKS cluster using `eksctl`:

```bash
eksctl create cluster --name project-cluster --region us-east-1 --fargate
```

### 2. Configure EKS Cluster Access
Configure kubectl access to the EKS cluster:

```bash
aws eks update-kubeconfig --name project-cluster --region us-east-1
```

### 3. Create Fargate Profile
Create a Fargate profile to run the 2048 gaming app:

```bash
eksctl create fargateprofile \
  --cluster project-cluster \
  --region us-east-1 \
  --name alb-sample-app \
  --namespace game-2048
```

### 4. Deploy the Application
Deploy the application, service, and ingress:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

### 5. Set Up OIDC Provider
Associate an OIDC provider with the EKS cluster to enable IAM roles for Kubernetes service accounts:

```bash
eksctl utils associate-iam-oidc-provider --cluster project-cluster --approve
```

### 6. Install ALB Controller
Install the AWS Load Balancer Controller:

1. Download the IAM policy:

    ```bash
    curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
    ```

2. Create the IAM policy:

    ```bash
    aws iam create-policy \
      --policy-name AWSLoadBalancerControllerIAMPolicy \
      --policy-document file://iam_policy.json
    ```

3. Create the IAM service account:

    ```bash
    eksctl create iamserviceaccount \
      --cluster=project-cluster \
      --namespace=kube-system \
      --name=aws-load-balancer-controller \
      --role-name AmazonEKSLoadBalancerControllerRole \
      --attach-policy-arn=arn:aws:iam::194474181188:policy/AWSLoadBalancerControllerIAMPolicy \
      --approve
    ```

4. Deploy the ALB controller:

    ```bash
    helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
      --set clusterName=project-cluster \
      --set serviceAccount.create=false \
      --set serviceAccount.name=aws-load-balancer-controller \
      --set region=us-east-1 \
      --set vpcId=vpc-02367930cafe5a44f
    ```

### 7. Verify Deployment
Check the status of the ALB controller deployment:

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

## Accessing the Application
Use the DNS name of the ALB to access the 2048 game application.

## Summary
This project demonstrates deploying a gaming app on a Kubernetes cluster using Fargate, integrating AWS services like IAM and ALB to provide secure, scalable, and efficient access to the application.
