# EKS 처음 사용하기

- aws cli 설치 확인

```
aws --version

```

- aws 계정 로그인 - iam 사용자 계정 생성(권한 그룹 생성 후 연결) - 액세스 키 발급

```
aws configure
```

- 로그인된 aws 계정 사용자 id 확인

```
aws sts get-caller-identity
```

## 1. eks 클러스터용 vpc 및 서브넷 생성

```
 aws cloudformation create-stack \
  --region ap-northeast-2 \
  --stack-name smart-cluster-vpc \
  --template-url https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml

```

- 결과

```
{
    "StackId": "arn:aws:cloudformation:ap-northeast-2:221370546661:stack/smart-cluster-vpc/fd90c030-1525-11ef-8dfe-025254541243"
}
```

- 서브넷 아이디/ 보안 그룹 아이디

```
aws ec2 describe-subnets --region ap-northeast-2
aws ec2 describe-security-groups --region ap-northeast-2
```

## 2. eks 클러스터 iam 역할 생성

```
aws iam create-role --role-name smart-ClusterRole --assume-role-policy-document file://aws/iam/TrustPolicyForEKS.json

```

- 결과

```

{
    "Role": {
        "Path": "/",
        "RoleName": "smart-ClusterRole",
        "RoleId": "AROATHCVWPHS5UJQ2GJKK",
        "Arn": "arn:aws:iam::221370546661:role/smart-ClusterRole",
        "CreateDate": "2024-04-20T20:21:25+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "eks.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        }
    }
}
```

## 3. 필요한 eks 관리형 iam 정책을 역할에 연결

```
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy \
  --role-name smart-ClusterRole

aws iam attach-role-policy \
  --role-name smart-ClusterRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy

aws iam attach-role-policy \
  --role-name smart-ClusterRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy

aws iam attach-role-policy \
  --role-name smart-ClusterRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly

aws iam attach-role-policy \
  --role-name smart-ClusterRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEBSCSIDriverPolicy

```

## 4. eks 클러스터 생성

```
aws eks create-cluster \
  --region ap-northeast-2 \
  --name smart-cluster \
  --role-arn arn:aws:iam::221370546661:role/smart-ClusterRole \
  --resources-vpc-config subnetIds=subnet-04c1fa565da32dbaf,subnet-054ad0183286e717b,subnet-0b3861508471e6c1e,subnet-094b2fc905022e004,securityGroupIds=sg-032a2e66db2413eb0
```

- 결과

```
{
    "cluster": {
        "name": "smart-cluster",
        "arn": "arn:aws:eks:ap-northeast-2:221370546661:cluster/smart-cluster",
        "createdAt": "2024-04-21T05:41:48.958000+09:00",
        "version": "1.29",
        "roleArn": "arn:aws:iam::221370546661:role/smart-ClusterRole",
        "resourcesVpcConfig": {
            "subnetIds": [
                "subnet-0019ca7f778a66290",
                "subnet-0f7b5e4951552c963",
                "subnet-0fa59931c4338ec23",
                "subnet-094625a83c3c53f88"
            ],
            "securityGroupIds": [
                "sg-0545bdb2852f2d23e"
            ],
            "vpcId": "vpc-063287596d983a8ba",
            "endpointPublicAccess": true,
            "endpointPrivateAccess": false,
            "publicAccessCidrs": [
                "0.0.0.0/0"
            ]
        },
        "kubernetesNetworkConfig": {
            "serviceIpv4Cidr": "10.100.0.0/16",
            "ipFamily": "ipv4"
        },
        "logging": {
            "clusterLogging": [
                {
                    "types": [
                        "api",
                        "audit",
                        "authenticator",
                        "controllerManager",
                        "scheduler"
                    ],
                    "enabled": false
                }
            ]
        },
        "status": "CREATING",
        "certificateAuthority": {},
        "platformVersion": "eks.6",
        "tags": {},
        "accessConfig": {
            "bootstrapClusterCreatorAdminPermissions": true,
            "authenticationMode": "CONFIG_MAP"
        }
    }
}
```

## 5. kubeconfig 파일 업데이트

```
aws eks --region ap-northeast-2 update-kubeconfig --name smart-cluster
```

## 6. 노드 그룹 생성

```
aws eks create-nodegroup --cluster-name smart-cluster --nodegroup-name smart-cluster-nodegroup --node-role arn:aws:iam::221370546661:role/smart-ClusterRole --subnets subnet-04c1fa565da32dbaf subnet-054ad0183286e717b subnet-0b3861508471e6c1e subnet-094b2fc905022e004 --region ap-northeast-2
```

- 결과

```
{
    "nodegroup": {
        "nodegroupName": "smart-cluster-nodegroup",
        "nodegroupArn": "arn:aws:eks:ap-northeast-2:221370546661:nodegroup/smart-cluster/smart-cluster-nodegroup/c8c7c620-77d6-1
6e7-1d7e-6cd6f613bd34",
        "clusterName": "smart-cluster",
        "version": "1.29",
        "releaseVersion": "1.29.3-20240514",
        "createdAt": "2024-05-19T00:10:19.970000+09:00",
        "modifiedAt": "2024-05-19T00:10:19.970000+09:00",
        "status": "CREATING",
        "capacityType": "ON_DEMAND",
        "scalingConfig": {
            "minSize": 1,
            "maxSize": 2,
            "desiredSize": 2
        },
        "instanceTypes": [
            "t3.medium"
        ],
        "subnets": [
            "subnet-04c1fa565da32dbaf",
            "subnet-054ad0183286e717b",
            "subnet-0b3861508471e6c1e",
            "subnet-094b2fc905022e004"
        ],
        "amiType": "AL2_x86_64",
        "nodeRole": "arn:aws:iam::221370546661:role/smart-ClusterRole",
        "diskSize": 20,
        "health": {
            "issues": []
        },
        "updateConfig": {
            "maxUnavailable": 1
        },
        "tags": {}
    }
}
```

```
# max 개수 수정
eksctl scale nodegroup --cluster=smart-cluster --name=smart-cluster-nodegroup --nodes-max=4
# 노드 그룹 노드 추가
eksctl scale nodegroup --cluster=smart-cluster --name=smart-cluster-nodegroup --nodes=3

# 노드 그룹 노드 삭제
eksctl delete nodegroup --cluster=smart-cluster --name=smart-cluster-nodegroup --region=ap-northeast-2

# 클러스터 삭제
eksctl delete cluster smart-cluster
```

# AWS EKS에 동작중인 컨테이너에 도메인 연결하기

- route53에 도메인 등록하는 부분은 aws 웹으로 직접 진행
- 가비아에서 도메인을 산 뒤에 route53에 등록한 ns로 가비아쪽 교체

## external-dns란

- public dns 서버를 통해 kubernetes 리소스를 검색할 수 있음
- kube-dns 처럼, 원하는 dns 정보를 찾기 위해서는 kubernetes API 에서 리소스 목록을 검색함

## route 53 특징

- 사용자의 요청을 EC2 인스턴스, ELB, S3 등에 효과적으로 연결
- 동적으로 사용자에게 노출될 DNS 레코드 타입과 값 조절
- 로드 밸런싱 기능

## EKS 와 Router 53

[참고](https://velog.io/@ironkey/AWS-EKS%EC%97%90%EC%84%9C-%EB%8F%99%EC%9E%91%EC%A4%91%EC%9D%B8-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EC%97%90-External-DNS%EB%A1%9C-%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%97%B0%EA%B2%B0%ED%95%98%EA%B8%B0)

- https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/aws.md
  > Route 53 ↔ ALB ↔ Node ↔ Pod로 트래픽이 라우팅됨
- iam role [external DNS]
- eks에 [exteranl DNS ] role 연결 -> serviceaccount.yaml
- deployment.yaml 배포시 spec.template.spec 에 serviceAccountName
- service.yaml 생성
- ingress.yaml 배포시 annotation에 설정

| aws | external-dns |
| --- | ------------ |

|kubernetes.io/ingress.class:alb
alb.ingress.kubernetes.io/scheme: internet-facing
alb.ingress.kubernetes.io/certificate-arn: ACM ARN
alb.ingress.kubernetes.io/ssl-policy: ELBSecurityPolicy
alb.ingress.kubernetes.io/listen-ports: '["HTTP": 80, "HTTPS": 443]:'
alb.ingress.kubernetes.io/actions.ssl-redirect: {"Type": "redirect", "RedirectConfig": {"Protocol": "HTTPS", "Port": 443, "StatusCode": "HTTP_301"}}|external-dns.alpha.kubernetes.io/hostname: hollyhostname.nett
|

## External DNS 설치

- https://repost.aws/ko/knowledge-center/eks-set-up-externaldns
- 쿠버네티스 서비스 계정에 IAM 역할을 사용하려면 OIDC 공급자가 필요
- IAM OIDC 자격증명 공급자 생성

```
eksctl utils associate-iam-oidc-provider --cluster smart-cluster --approve
```

- external dns 용도의 namespace 생성

```
k create ns external-dns
```

- external-dns 가 route53을 제어할 수 있도록 정책 생성

```
aws iam create-policy --policy-name AllowExternalDNSUpdates --policy-document file://aws/iam/external-dns-route53-policy.json
```

```
{
    "Policy": {
        "PolicyName": "AllowExternalDNSUpdates",
        "PolicyId": "ANPATHCVWPHSQUQ6BPYHZ",
        "Arn": "arn:aws:iam::221370546661:policy/AllowExternalDNSUpdates",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2024-04-21T01:02:30+00:00",
        "UpdateDate": "2024-04-21T01:02:30+00:00"
    }
}
```

- iam service account 생성

```
eksctl create iamserviceaccount \
    --name <service_account_name> \ ### Exteranl DNS service account명 = external-dns
    --namespace <service_account_namespace> \ ### External DNS namespace명 = external-dns
    --cluster <cluster_name> \ ### AWS EKS 클러스터명
    --attach-policy-arn <IAM_policy_ARN> \ ### 앞서 생성한 Policy arn
    --approve \
    --override-existing-serviceaccounts
```

```
eksctl create iamserviceaccount \
    --name external-dns \
    --namespace external-dns \
    --cluster smart-cluster \
    --attach-policy-arn arn:aws:iam::221370546661:policy/AllowExternalDNSUpdates \
    --approve \
    --override-existing-serviceaccounts
```

- 확인

```
k get sa -n external-dns
```

## external-dns 배포

https://repost.aws/ko/knowledge-center/eks-set-up-externaldns

- rbac 가 켜져 있는지 확인

```
kubectl api-versions | grep rbac.authorization.k8s.io

```

- external-dns 매니페스트 배포

```
k apply -f external-dns/values.yaml
```

- 도메인 적용 테스트
  - 확인 사항

```
    annotations:
        external-dns.alpha.kubernetes.io/hostname: 실제 적용 도메인
spec:
  type: LoadBalancer
...
```

```
k apply -f external-dns/test.yaml
```

해당 작업까지 eks에서 서비스를 external-dns를 통해 외부에서 접속하는 방법까지 진행
아래 내용은 https로 접근 하기 위해 cert-manager를 적용하는 방법 진행

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
