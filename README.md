# IAM Policy 생성

- AWS Load Balancer Controller를 위한 IAM Policy를 다운로드

```
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.3.1/docs/install/iam_policy.json

```

# IAM policy를 생성

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

# IAM Role 및 Kubernetes의 Service Account 생성

```
eksctl create iamserviceaccount \
  --cluster=smart-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::931639357206:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --approve
```

# helm reopsitory 추가 및 설치

```
helm repo add eks https://aws.github.io/eks-charts
helm repo update

 helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=smart-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set image.repository=602401143452.dkr.ecr.ap-northeast-2.amazonaws.com/amazon/aws-load-balancer-controller
```

# 확인

```
kubectl get deployment -n kube-system aws-load-balancer-controller

```
