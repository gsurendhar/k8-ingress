# AWS Load Balancer Controller Setup on EKS

This guide sets up the **AWS Load Balancer Controller** for your EKS cluster.  
It configures IAM OIDC, creates the necessary IAM policy and service account, and installs the controller using Helm.

---

## 📋 Prerequisites

Before running the setup, ensure that you have:

- An **EKS cluster** up and running.
- **AWS CLI**, **eksctl**, **kubectl**, and **Helm** installed and configured.
- Sufficient IAM permissions to create policies and roles.

---

## 🧩 Environment Variables

Update and export your environment variables:

```bash
REGION_CODE=us-east-1
CLUSTER_NAME=roboshop-dev
ACC_ID=29328
```

🔗 Step 1: Associate IAM OIDC Provider
Associate your cluster with an IAM OIDC provider.
This allows IAM roles to be used by Kubernetes service accounts.


```
eksctl utils associate-iam-oidc-provider \
  --region $REGION_CODE \
  --cluster $CLUSTER_NAME \
  --approve
```

🛡️ Step 2: Create IAM Policy for Load Balancer Controller
Download the AWS IAM policy JSON for the Load Balancer Controller:


```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.2.1/docs/install/iam_policy.json
```


Create the IAM policy:

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

👤 Step 3: Create IAM Service Account
Create the service account in the kube-system namespace and attach the IAM policy to it:

```
eksctl create iamserviceaccount \
--cluster=$CLUSTER_NAME \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::$ACC_ID:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--approve
```

📦 Step 4: Install Helm Chart Repository
Add the EKS charts Helm repository:

```
helm repo add eks https://aws.github.io/eks-charts
```


⚙️ Step 5: Apply Custom Resource Definitions (CRDs)
Apply the necessary CRDs for the AWS Load Balancer Controller:

```
kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master"
```

🚀 Step 6: Install AWS Load Balancer Controller
Install the controller using Helm:

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$CLUSTER_NAME
```

✅ Step 7: Verify Installation
Check that the controller pods are running:

```
kubectl get pods -n kube-system
```
You should see something like:

`aws-load-balancer-controller-xxxxxx   Running   1/1   ...`

🧾 Notes
If you are using EKS clusters created via Terraform or other IaC tools, make sure OIDC is enabled.

You can view the policy details in the IAM console under Policies → `AWSLoadBalancerControllerIAMPolicy`.













```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.2.1/docs/install/iam_policy.json
```




