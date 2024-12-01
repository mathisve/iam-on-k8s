# iam-on-k8s

## Create EKS cluster
```
eksctl create cluster -f eksctl.yaml
```

## Using secret
In this example, we go over how to use AWS IAM using user credentials stored in a Kubernetes secret.

## Using IRSA
In this example, we go over how to configure a service account to access an IAM Role.