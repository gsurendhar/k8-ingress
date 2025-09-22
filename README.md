# AWS ALB Ingress Controller Setup for Kubernetes (EKS)

This guide explains how to install and configure the **AWS Load Balancer Controller** to manage Kubernetes `Ingress` resources using an **Application Load Balancer (ALB)** on Amazon EKS.

---

## ✅ Prerequisites

- An EKS Cluster already created
- `eksctl`, `kubectl`, `aws-cli`, and `helm` installed and configured
- IAM permissions to create roles and policies
- VPC and Subnets tagged for ALB (see [Tagging Requirements](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html#subnet-tagging))

---

## ⚙️ Step-by-Step Setup

### 1. Enable IAM OIDC Provider

Enable the OIDC identity provider for your cluster (only needed once per cluster):

```bash
REGION_CODE=us-east-1
CLUSTER_NAME=roboshop-dev
ACC_ID=<your-aws-account-id>

eksctl utils associate-iam-oidc-provider \
  --region $REGION_CODE \
  --cluster $CLUSTER_NAME \
  --approve
```

---

### 2. Download IAM Policy for Load Balancer Controller

Download the latest IAM policy JSON:

```bash
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.13.4/docs/install/iam_policy.json
```

---

### 3. Create IAM Policy

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam-policy.json
```

---

### 4. Create IAM Role and Kubernetes ServiceAccount

This step maps an AWS IAM role to a Kubernetes ServiceAccount:

```bash
eksctl create iamserviceaccount \
  --cluster=$CLUSTER_NAME \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::$ACC_ID:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --region=$REGION_CODE \
  --approve
```

---

### 5. Add Helm Repo and Install the Controller

First, add the Helm repo:

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

Install the controller:

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=$CLUSTER_NAME \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

> **Note:** Double-check your VPC ID and region if you encounter errors related to networking or permissions.

---

## 🌐 Example Ingress Resource for ALB

Here is a basic `Ingress` YAML that works with the AWS ALB Ingress Controller:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
  namespace: default
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
spec:
  ingressClassName: alb
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: demo-service
                port:
                  number: 80
```

> Replace `demo-service` with the name of your Kubernetes service.

---

## 📌 Common Ingress Annotations for ALB

| Annotation                               | Description                                   |
| ---------------------------------------- | --------------------------------------------- |
| `alb.ingress.kubernetes.io/scheme`       | `internet-facing` or `internal`               |
| `alb.ingress.kubernetes.io/target-type`  | `ip` (for pods) or `instance` (for node port) |
| `alb.ingress.kubernetes.io/listen-ports` | List of ALB listener ports                    |
| `alb.ingress.kubernetes.io/group.name`   | Group multiple ingresses into the same ALB    |

See full list here:
🔗 [AWS ALB Controller Annotations](https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/guide/ingress/annotations/)

---

## 📈 Verify Installation

Check if the controller is running:

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

To see the ALB created:

```bash
kubectl describe ingress demo-ingress
```

---

## 📚 Official Documentation

* [AWS Load Balancer Controller GitHub](https://github.com/kubernetes-sigs/aws-load-balancer-controller)
* [Install Guide on AWS Docs](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)

```

---
