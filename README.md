# EKS-Repo

Managed Control Plane: EKS takes care of managing the Kubernetes control plane components, such as the API server, controller manager, and etcd. AWS handles upgrades, patches, and ensures high availability of the control plane.

Automated Updates: EKS automatically updates the Kubernetes version, eliminating the need for manual intervention and ensuring that the cluster stays up-to-date with the latest features and security patches.

Scalability: EKS can automatically scale the Kubernetes control plane based on demand, ensuring the cluster remains responsive as the workload increases.

AWS Integration: EKS seamlessly integrates with various AWS services, such as AWS IAM for authentication and authorization, Amazon VPC for networking, and AWS Load Balancers for service exposure.

Security and Compliance: EKS is designed to meet various security standards and compliance requirements, providing a secure and compliant environment for running containerized workloads.

Monitoring and Logging: EKS integrates with AWS CloudWatch for monitoring cluster health and performance metrics, making it easier to track and troubleshoot issues.

Ecosystem and Community: Being a managed service, EKS benefits from continuous improvement, support, and contributions from the broader Kubernetes community.

**Prerequisites**

- Installed kubect,eksctl and aws cli
- Then is to crate EKS cluster, many ways like terraform script, through UI and eksctl command
-  here we are using eksctl command

-  command to create eks cluster
eksctl create cluster --name demo-cluster --region us-east-1 --fargate

by fargate it would be easy to mange worker nodes because it is aws manged 

Once it cretaed walk through the console we will see lot of features there, fargate will create some node already
and fargate by default attached to 2 NS (default and kube-system) for new ns we should create new fargate group

to generate kubeconfig which helps to interact with cluster via kubectl or cli instead Ui dashboard

aws eks update-kubeconfig --name <cluster>  --regoin <region name>

like mentioned creating new fargate profile for new ns

eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \     just name of app
    --namespace game-2048

kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
all files needed to run resources

Next is ing controller without controller ing resource is waste

iam authentication needed to communicate with alb controller pods
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve

add iam policy

curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

create iam policy

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json

create iam role

eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

create alb controller with helm command

