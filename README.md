# Deploying a 2048 Game Application on Amazon EKS with Fargate
This guide provides step-by-step instructions to deploy a 2048 game sample application on Amazon EKS using Kubernetes manifests. We will leverage AWS Fargate for serverless compute, and use AWS Load Balancer Controller for ingress management. This deployment also integrates a Kubernetes service account with an IAM role for enhanced security and control.

## Prerequisites
Refer to the documents for each of these and set them up:
    - AWS CLI installed and configured
    - eksctl installed
    - kubectl installed
    - Helm installed

## Step-by-Step Instructions
### 1. Create an EKS Cluster
Create an EKS cluster named test-cluster in the us-east-2 region using Fargate:
~~~
eksctl create cluster --name test-cluster --region us-east-2 --fargate
~~~

This command initializes an EKS cluster with Fargate profiles, allowing you to run serverless pods.

### 2. Update kubeconfig
Update your kubeconfig file to interact with the newly created EKS cluster:
~~~
aws eks update-kubeconfig --name test-cluster --region us-east-2
~~~
This command updates the local Kubernetes configuration to include the new EKS cluster, enabling kubectl commands to interact with the cluster.

### 3. Create a Fargate Profile
Create a Fargate profile for the namespace game-2048:
~~~
eksctl create fargateprofile \
    --cluster test-cluster \
    --region us-east-2 \
    --name alb-sample-app \
    --namespace game-2048
~~~
This command creates a Fargate profile which specifies the namespaces (in this case, game-2048) that will use Fargate for compute resources.

### 4. Deploy the 2048 Game Application
Deploy the 2048 game application using Kubernetes manifests:
~~~
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
~~~
This command deploys the 2048 game application into the game-2048 namespace.

### 5. Verify Deployment
Check the status of the pods, services, and ingress in the game-2048 namespace:
~~~
kubectl get pods -n game-2048
kubectl get svc -n game-2048
kubectl get ingress -n game-2048
~~~
These commands help you verify that the application pods, services, and ingress resources have been created and are running as expected.

### 6. Associate IAM OIDC Provider
Associate an IAM OIDC provider with the EKS cluster:
~~~
eksctl utils associate-iam-oidc-provider --cluster test-cluster --approve
~~~
This command associates an OIDC provider with your EKS cluster, enabling IAM roles to be assumed by Kubernetes service accounts.

### 7. Create IAM Policy for AWS Load Balancer Controller
Download the IAM policy for the AWS Load Balancer Controller:
~~~
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
~~~
Create the IAM policy:
~~~
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
~~~
This command creates an IAM policy necessary for the AWS Load Balancer Controller to function.

### 8. Create IAM Service Account
Create a Kubernetes service account with the IAM role:
~~~
eksctl create iamserviceaccount \
    --cluster=test-cluster \
    --namespace=kube-system \
    --name=aws-load-balancer-controller \
    --role-name AmazonEKSLoadBalancerControllerRole \
    --attach-policy-arn=arn:aws:iam::637423455653:policy/AWSLoadBalancerControllerIAMPolicy \
    --approve
~~~
This command creates an IAM role and associates it with a Kubernetes service account, granting the necessary permissions to the AWS Load Balancer Controller.

### 9. Install AWS Load Balancer Controller using Helm
Add the EKS Helm repository:
~~~
helm repo add eks https://aws.github.io/eks-charts
~~~
Update the Helm repository:
~~~
helm repo update eks
~~~
Install the AWS Load Balancer Controller:
~~~
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
    --set clusterName=test-cluster \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set region=us-east-2 \
    --set vpcId=vpc-078d9e3f0ba0740f1
~~~
This command installs the AWS Load Balancer Controller using Helm, configuring it to manage ingress resources in the EKS cluster.

### 10. Verify AWS Load Balancer Controller Deployment
Check the status of the AWS Load Balancer Controller deployment:
~~~
kubectl get deployment -n kube-system aws-load-balancer-controller
~~~

## Conclusion
By following these steps, you have successfully deployed a 2048 game application on Amazon EKS with Fargate, integrating AWS Load Balancer Controller for ingress management and securing the setup with a Kubernetes service account associated with an IAM role. This demonstrates the powerful capabilities of orchestrating containerized applications using Kubernetes in the AWS ecosystem. Continue exploring AWS EKS to unlock further potential and possibilities.









































































