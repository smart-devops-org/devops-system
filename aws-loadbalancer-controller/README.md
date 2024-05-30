## AWS load Balancer controller

https://catalog.us-east-1.prod.workshops.aws/v2/workshops/9c0aa9ab-90a9-44a6-abe1-8dff360ae428/ko-KR/60-ingress-controller/100-launch-alb

[참고](https://docs.aws.amazon.com/ko_kr/prescriptive-guidance/latest/patterns/set-up-end-to-end-encryption-for-applications-on-amazon-eks-using-cert-manager-and-let-s-encrypt.html)

1. AWS load Balancer controller 배포

- https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/lbc-helm.html

- iam 정책 생성

```
curl -O  https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.7.1/docs/install/iam_policy.json

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

```
{
    "Policy": {
        "PolicyName": "AWSLoadBalancerControllerIAMPolicy",
        "PolicyId": "ANPATHCVWPHSRG3QQQRL5",
        "Arn": "arn:aws:iam::221370546661:policy/AWSLoadBalancerControllerIAMPolicy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2024-04-22T06:31:39+00:00",
        "UpdateDate": "2024-04-22T06:31:39+00:00"
    }
}
```

- iam 역할 생성

```
eksctl create iamserviceaccount \
  --cluster=smart-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::221370546661:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

```
# IRSA 정보 확인
eksctl get iamserviceaccount --cluster smart-cluster

# Kubernetes 서비스 어카운트 확인
kubectl get serviceaccounts -n kube-system aws-load-balancer-controller -o yaml

```

```
2024-04-22 15:33:48 [ℹ]  1 existing iamserviceaccount(s) (external-dns/external-dns) will be excluded
2024-04-22 15:33:48 [ℹ]  1 iamserviceaccount (kube-system/aws-load-balancer-controller) was included (based on the include/exclude rules)
2024-04-22 15:33:48 [!]  serviceaccounts that exist in Kubernetes will be excluded, use --override-existing-serviceaccounts to override
2024-04-22 15:33:48 [ℹ]  1 task: {
    2 sequential sub-tasks: {
        create IAM role for serviceaccount "kube-system/aws-load-balancer-controller",
        create serviceaccount "kube-system/aws-load-balancer-controller",
    } }2024-04-22 15:33:48 [ℹ]  building iamserviceaccount stack "eksctl-smart-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
2024-04-22 15:33:49 [ℹ]  deploying stack "eksctl-smart-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
2024-04-22 15:33:49 [ℹ]  waiting for CloudFormation stack "eksctl-smart-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"

2024-04-22 15:34:19 [ℹ]  waiting for CloudFormation stack "eksctl-smart-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
2024-04-22 15:34:19 [ℹ]  created serviceaccount "kube-system/aws-load-balancer-controller"
```

- aws load balaoncer controller 설치

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=smart-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

- 확인

```
kubectl get po -n kube-system | egrep -o "aws-load-balancer[a-zA-Z0-9-]+"
```

```
# Kubernetes CRD 확인
kubectl get crd

# ingressclassparams.elbv2.k8s.aws
# targetgroupbindings.elbv2.k8s.aws

# AWS Load Balancer Controller Role 확인
kubectl describe clusterroles.rbac.authorization.k8s.io aws-load-balancer-controller-role

# AWS Load Balancer Controller 확인
kubectl get deployment -n kube-system aws-load-balancer-controller

kubectl describe deploy -n kube-system aws-load-balancer-controller
```

https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/network-load-balancing.html
