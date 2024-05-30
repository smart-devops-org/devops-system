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
