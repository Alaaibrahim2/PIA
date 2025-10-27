# 🚀 Access Amazon S3 from EKS Pod using Pod Identity

This repo demonstrates how to use **Amazon EKS Pod Identity Agent** to allow pods to access AWS resources (like S3) securely — **without using IAM keys** or `aws configure`.

---

## 🧠 Concept Overview

1. **IAM Role for Pod Identity**
   - Create an IAM Role trusted by `pods.eks.amazonaws.com`.
   - Attach permissions (e.g., `AmazonS3ReadOnlyAccess`).

2. **Pod Identity Association**
   - Link the IAM Role to a Kubernetes ServiceAccount (and Namespace).

3. **Webhook Mutation**
   - When a Pod uses that ServiceAccount, EKS automatically injects:
     - Environment vars like `AWS_CONTAINER_CREDENTIALS_FULL_URI`
     - A projected token for authentication.

4. **Pod Requests Credentials**
   - Inside the Pod, AWS SDK or CLI detects these vars and requests credentials from the EKS Pod Identity Agent.

5. **Pod Identity Agent**
   - Exchanges the token with the EKS Auth API → receives temporary credentials → allows secure access to AWS.

---

## 🧩 Step-by-Step Demo

### 1️⃣ Install Pod Identity Agent
In EKS Console:
Search for **EKS Pod Identity Agent** → Click **Next** → **Create**

Verify installation:
```bash
kubectl get daemonset -n kube-system | grep pod-identity
kubectl get pods -n kube-system | grep pod-identity
kubectl exec -it aws-cli -- aws s3 ls
3️⃣ Create IAM Role for Pod Identity
Go to IAM → Roles → Create Role

Choose Trusted entity: Custom trust policy

Paste this JSON:
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": { "Service": "pods.eks.amazonaws.com" },
            "Action": ["sts:AssumeRole", "sts:TagSession"]
        }
    ]
}
4️⃣ Create Pod Identity Association
From EKS Console → Cluster → Access → Pod Identity Associations:

Namespace: default

Service Account: aws-cli-sa

IAM Role: EKS-PodIdentity-S3-ReadOnly-Role

Click Create
kubectl exec -it aws-cli -- aws s3 ls
