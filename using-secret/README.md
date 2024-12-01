# Using Secret

## Create Policy
```
policy_name=example-s3-policy
aws iam create-policy --policy-name ${policy_name} --policy-document file://iam-policy.json
```

## Create User
```
user_name=s3-k8s-user
aws iam create-user --user-name ${user_name}
account_id=$(aws sts get-caller-identity --query "Account" --output text)
aws iam attach-user-policy --user-name ${user_name} --policy-arn arn:aws:iam::${account_id}:policy/${policy_name}
```

## Create credentials
```
aws iam create-access-key --user-name ${user_name} | jq -r
```

## Create secret
```
access_key_id=AKIAQUBKYT6WCD2OO6MH
secret_access_key=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

kubectl create secret generic aws-credentials --dry-run=client -o yaml \
  --from-literal=AWS_ACCESS_KEY_ID=${access_key_id} \
  --from-literal=AWS_SECRET_ACCESS_KEY=${secret_access_key} > secret.yaml 

kubectl apply -f secret.yaml
```

## Create deployment
```
kubectl apply -f deploy.yaml
```