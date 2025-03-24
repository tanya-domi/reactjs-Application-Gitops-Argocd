# Three-tier application deployment with DevSecOps principles on AWS using Kubernetes, GitOps, ArgoCD, and GitHub Actions.

# Project Introduction:
DevSecOps Kubernetes Project Guide: Learn to set up a robust Three-Tier architecture on AWS using Kubernetes (EKS) with DevOps best practices and security measures. 
Gain hands-on experience in deploying, securing, and monitoring a scalable application.

![dude](https://github.com/user-attachments/assets/b7f28dfb-ad8a-4cac-a8dd-a3f768d02ae4)

Step 1: SSH Exchange between local computer and Github account.

Step 1: CREATE AWS Resources.

Step 2: Install Terraform & AWS CLI.

Step 3: Deploy the Jumphost Server(EC2) using Terraform on Github Actions.

Step 4: Configure the Jumphost.

Step 5: Setup Docker Repositories to allow image push for Frontend & Backend images.

Step 6: Configure Sonar Cloud for our app_code Pipeline.

Step 7: Setup Synk Token for the app code pipeline.

Step 8: Review and Deploy Application Code.

Step 9: Configure ArgoCD.

Step 10: Set up the Monitoring for our EKS Cluster using Prometheus and Grafana.

Step 11: Deploy Quiz Application using ArgoCD.

Step 12: Creating an A-Record in AWS Route 53 Using ALB DNS.

Step 13: Clean up.

Conclusion

# Project Overview:
In this project, we will cover the following key aspects:

1. IAM User Setup: Create an AWS IAM user with required permissions.
2. Infrastructure as Code (IaC): Use Terraform and AWS CLI to set up a Jumphost (EC2).
3. GitHub Actions: Configure workflows with tools like Snyk, Docker, SonarQube, Terraform, kubectl, AWS CLI, and Trivy.
4. EKS Deployment: Use eksctl to create an Amazon EKS cluster.
5. Load Balancer: Set up an AWS Application Load Balancer (ALB) for the cluster.
6. DockerHub Repositories: Automatically create frontend and backend image repositories.
7. ArgoCD: Install ArgoCD for continuous delivery and GitOps.
8. SonarQube: Integrate for code quality checks.
9. Snyk: Integrate for vulnerability and dependency scanning.
10. Trivy: Add Trivy for container image scanning.
11. GitHub Pipelines: Create workflows to deploy backend and frontend to EKS.
12. Monitoring: Use Helm, Prometheus, and Grafana for cluster monitoring.
13. ArgoCD Deployment: Deploy the app (database, backend, frontend) with ArgoCD.
14. DNS Setup: Configure custom subdomains for app access.
15. Data Persistence: Use Persistent Volumes (PVs) for database data storage.
16. Conclusion: Summarize achievements and monitor the EKS cluster with Grafana.

# Prerequisites:
An AWS account with the necessary permissions to create resources.
Terraform and AWS CLI installed on your local computer.
Familiarity with Kubernetes, Docker, CICD pipelines, Github Actions, Terraform, and DevOps principles.

Step 1: CREATE AWS Resources
Create an IAM user and generate the AWS Access key
Create a new IAM User on AWS and give it the AdministratorAccess for testing purposes (not recommended for your Organization's Projects) 

Step 2: Install Terraform & AWS CLI
Install & Configure Terraform and AWS CLI on your local machine

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install

Step 3: Deploy the Jumphost Server(EC2) using Terraform on Github Actions

Step 4: Configure the Jumphost
We have installed some services such as Docker, Terraform, Kubectl, eksctl, AWSCLI, Trivy
Validate whether all our tools are installed or not.
docker --version
docker ps
terraform --version
kubectl version
aws --version
trivy --version
eksctl version

# Create an Eks cluster using the below commands.
This might take 15-20 minutes. Also adjust the node count
eksctl create cluster --name quizapp-eks-cluster --region us-east-1 --node-type t2.large --nodes-min 2 --nodes-max 4

Run the command below to connect to the EKS cluster created allowing Kubernetes operations on that cluster.

aws eks update-kubeconfig --region us-east-1 --name quizapp-eks-cluster

Once the cluster is created, you can validate whether your nodes are ready or not by the below command

kubectl get nodes

# Configure Load Balancer on the EKS
Configure the Load Balancer on our EKS because our application will have an ingress controller. Download the policy for the LoadBalancer prerequisite.
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

# Create IAM policy
Create the IAM policy using the below command
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json

# Create OIDC Provider
To allows the cluster to integrate with AWS IAM for assigning IAM roles to Kubernetes service accounts, enhancing security and management.
eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=quizapp-eks-cluster --approve

# Create Service Account
eksctl create iamserviceaccount --cluster=quizapp-eks-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<ACCOUNT-ID>:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-east-1

# Run the below command to deploy the AWS Load Balancer Controller using Helm

sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=quizapp-eks-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller

Wait for 2 minutes and run the following command below to check whether aws-load-balancer-controller pods are running or not.

kubectl get deployment -n kube-system aws-load-balancer-controller


Step 5: Setup Docker Repositories to allow image push for Frontend & Backend images

# Create Docker Secret or generate a Access Token

Step 6: Configure Sonar Cloud for app_code Pipeline
Sonar cloud will be using for Code Quality Analysis of our application code.

Step 7: Setup Synk Token for the app code pipeline
Integrate for vulnerability and dependency scanning.

Step 8: Review and Deploy Application Code
git commit -am "updated manifest files"
git push

[![React.js CI](https://github.com/tanya-domi/reactjs-Application/actions/workflows/ci.yaml/badge.svg)](https://github.com/tanya-domi/reactjs-Application/actions/workflows/ci.yaml)


![gitaction 11](https://github.com/user-attachments/assets/9f632b2b-dc97-459d-a4ff-dbf34257b087)


Step 9: Configure ArgoCD
Create the namespace for the EKS Cluster. In your jumphost server terminal

kubectl create namespace quiz
kubectl get namespaces

# kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml

To confirm argoCD pods are running. All pods must be running, to validate run the below command

# kubectl get pods -n argocd

# expose the argoCD server as LoadBalancer using the below command
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# To access the argoCD, copy the LoadBalancer DNS and hit on your browser.

we need to get the password for our argoCD server to perform the deployment.
we need a pre-requisite which is jq. This has already been Installed or you can install it using the command below.

sudo apt install jq -y

export ARGOCD_SERVER=`kubectl get svc argocd-server -n argocd -o json | jq --raw-output '.status.loadBalancer.ingress[0].hostname'`
export ARGO_PWD=`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`
echo $ARGO_PWD

Step 10
Set up the Monitoring for our EKS Cluster using Prometheus and Grafana.
We can monitor the Cluster Specifications and other necessary things.
We installed monitoring using Helm Add all the helm repos, prometheus, grafana repo by using the below command

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Install the Prometheus
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace

# Install the Grafana
helm install grafana grafana/grafana -n monitoring --create-namespace

# Get Grafana admin user password using:
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

# confirm the services using the below command

kubectl get svc -n monitoring

# View your Kubernetes Cluster Data.

![nnn](https://github.com/user-attachments/assets/68941b16-8771-4c8f-a466-947150e6a670)

# For Nodes

![grafan 1](https://github.com/user-attachments/assets/9b413482-2d0f-469e-9b32-dbc5754531f4)

Step 11: Deploy Quiz Application using ArgoCD.


![ARgo 2](https://github.com/user-attachments/assets/0f864c91-9394-464a-8f65-ba2b54331cdf)

# Deployment is synced and healthy
![notee](https://github.com/user-attachments/assets/d90146b5-429a-4132-9e4f-eac4924fc092)

Step 12: Creating an A-Record in AWS Route 53 Using ALB DNS
Create A-records using DNS service in aws [Route53].Create an A-record in AWS Route 53 that points to your Application Load Balancer (ALB).

# Logged into the simple quiz application
![app!2](https://github.com/user-attachments/assets/edc07489-0022-4b43-8bf2-f8033e918784)


# Conclusion:
In this comprehensive DevSecOps Kubernetes project, we successfully:
- Set up IAM user and AWS infrastructure with Terraform.
- Deployed Infrastructure on AWS using Github Actions and Terraform and, configured tools.
- Configured EKS cluster and Load Balancer.
- Implemented monitoring with Helm, Prometheus, and Grafana.
- Installed and configured ArgoCD for GitOps practices.
- Created Github Action pipelines for CI/CD, deploying a three-tier architecture application.
- Ensured data persistence with persistent volumes and claims.


