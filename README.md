# Deploying a 2048 Game Application on Amazon EKS with Fargate
This guide provides step-by-step instructions to deploy a 2048 game sample application on Amazon EKS using Kubernetes manifests. We will leverage AWS Fargate for serverless compute, and use AWS Load Balancer Controller for ingress management. This deployment also integrates a Kubernetes service account with an IAM role for enhanced security and control.

## Prerequisites
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
