# Using IAM Role for Service Account (IRSA)

## Create OIDC provider
https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html
```
cluster_name=small-cluster
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```

## Create IAM Policy
https://docs.aws.amazon.com/eks/latest/userguide/associate-service-account-role.html#irsa-associate-role-procedure
```
aws iam create-policy --policy-name s3-list-object --policy-document file://iam-policy.json
```

## Create Kubernetes Service Account
https://docs.aws.amazon.com/eks/latest/userguide/associate-service-account-role.html#_step_2_create_and_associate_iam_role

```
kubectl apply -f service-account.yaml
```

## Create Trust Relationship & Role
```
cluster_region=us-east-1
cluster_name=small-cluster

account_id=$(aws sts get-caller-identity --query "Account" --output text)
oidc_provider=$(aws eks describe-cluster --name $cluster_name --region $cluster_region --query "cluster.identity.oidc.issuer" --output text | sed -e "s/^https:\/\///")

namespace=default
service_account=my-service-account

role_name=s3-list-objects

cat >> trust-relationship.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::$account_id:oidc-provider/${oidc_provider}"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "${oidc_provider}:aud": "sts.amazonaws.com",
          "${oidc_provider}:sub": "system:serviceaccount:$namespace:$service_account"
        }
      }
    }
  ]
}
EOF

aws iam create-role --role-name $role_name --assume-role-policy-document file://trust-relationship.json

aws iam attach-role-policy --role-name $role_name --policy-arn=arn:aws:iam::$account_id:policy/s3-list-object
```

## Annotate Service Account
```
kubectl annotate serviceaccount -n $namespace $service_account eks.amazonaws.com/role-arn=arn:aws:iam::${account_id}:role/$role_name
```

## Deploy pod with Service Account
```
kubectl apply -f deploy.yaml
```