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
