# EKS 공부 + kubernetes object 정리 + devops tool + CI/CD 사용 프로젝트

## 개요

### 목적

1. vpc, subnet ,보안 그룹, eks, node-group, 로드밸런서, route53, auto-scale ,EBS,ECR ,IAM 등 eks로 운영환경 구축시 필요간 기본적인 세팅을 혼자 할 수 있도록 한다(AWS 리소스 파악 정리)
2. kubernetes 사용 필요한 object에 대한 기초를 확실히 한다
3. devops 운영시 필요한 tools에 대한 이해 및 정리 한다
4. github Action or jenkins , ArgoCD를 활용한 CI/CD 환경을 구축한다

### 의의

| 해당 프로젝트를 통해 기초적인 부분으로 어떻게 devops 기본 능력치 확인,정리 및 이직 준비할 때의 자료로 사용한다

# 1. AWS 리소스 파악 정리

## 1.1 VPC 란

- AWS (Amazon Web Service) 에서 제공하는 네트워크 서비스
- 네트워크 관련 여러가지 서비스들을 조합하여 네트워크 환경을 직접 설계 및 구축할 수 있으며 언제든지 유연하게 구성을 변경 가능
- 각 VPC는 완전히 독립적이기 때문에 VPC간 통신을 원한다면 VPC 피어링 서비스를 고려해야 함
  ![image](https://github.com/smart-devops-org/devops-system/assets/46982752/a4c3eeba-ea43-4f89-8a38-a3ff39cbc0f9)
- vpc 를 적용하는 이유
  ![image](https://github.com/smart-devops-org/devops-system/assets/46982752/87e8df3e-3e00-495a-a4fa-284952729b1f)

## 1.2 VPC 기본 구성

### 서브넷

- VPC의 IP 주소 범위
- 서브넷은 단일 가용 영역에 상주해야함
- 서브넷을 추가한 후에 VPC에 AWS 리소스를 배포 가능
  - 퍼블릭 서브넷 : 공개된 네트워크으로 인터넷과 직접적으로 통신가능
  - 프라이빗 서브넷 : 비공개된 네트워크로 내부적으로 통신하는 용도로 사용

### IP 대역

- 퍼블릭 IPv4 주소 및 IPv6 GUA 주소를 AWS로 가져오고 VPC의 리소스(예: EC2 인스턴스, NAT 게이트웨이, Network Load Balancer)에 할당 가능
- 한번 설정된 아이피 대역은 수정할 수 없으며 각 VPC는 하나의 리전에 종속됨
- VPC에서 사용하는 사설 아이피 대역

```
- 10.0.0.0 ~ 10.255.255.255(10/8 prefix)

- 172.16.0.0 ~ 172.31.255.255(182.16/12 prefix)

- 192.168.0.0 ~ 192.168.255.255(192.168/16 prefix)
```

### 라우터

- 네트워크 통신을 수행할 때 거쳐 가능 경로를 잡아 주는 역할을 라우팅이라고 함
- 라우팅을 수행하는 장비
- VPC 생성시 자동으로 가상 라우터도 생성

### 라우팅 테이블

- 라우터는 라우팅 테이블을 통해 경로를 파악 원하는 목적지 대상으로 데이터를 전달
- VPC 생성시 자동으로 가상 라우팅 테이블도 생성

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/692066f2-500c-41bd-9646-224ae7c3e0ab)

### 게이트웨이 및 엔드 포인트

- VPC와 인터넷을 연결해주는 역할
- 인터넷과 연결되어있는 서브넷을 퍼블릭서브넷, 인터넷과연결되어있지않는 서브넷을 프라이빗서브넷
  - 인터넷 게이트웨이 : 인터넷 구간으로 나가는 관문, VPC 당 1개만 연결가능
  - NAT 게이트웨이 : NAT 게이트웨이는 프라이빗서브넷이 인터넷과 통신하기위한 아웃바운드 인스턴스

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/96c0242f-36df-4460-91b9-25c11771b711)
![image](https://github.com/smart-devops-org/devops-system/assets/46982752/92180344-c07a-4d2c-bf3c-e4c32ed1a1ad)

### 보안 그룹

- 통신 규칙을 정하여 지정한 통신 대상을 걸러주는 필터링 역할
- 인스턴스 레벨 곧 컴퓨팅 자원에 대한 필터링
- Stateful한 방식으로 동작하는 보안그룹은 모든 허용을 차단하도록 기본설정 되어있으며 필요한 설정은 허용
- 각각의 보안그룹별로도 별도의 트래픽을 설정할 수 있으며 서브넷에도 설정할 수 있지만 각각의 EC2인스턴스에도 적용 가능

### NACL

- 통신 규칙을 정하여 지정한 통신 대상을 걸러주는 필터링 역할
- 서브넷 레벨 에서의 필터링

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/543f26d7-3749-45ee-b316-7f927d021a77)

### 참고 링크

- [Amazon VPC란 무엇인가?](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/what-is-amazon-vpc.html)
- [가장 쉽게 VPC 개념 잡기](https://medium.com/harrythegreat/aws-%EA%B0%80%EC%9E%A5%EC%89%BD%EA%B2%8C-vpc-%EA%B0%9C%EB%85%90%EC%9E%A1%EA%B8%B0-71eef95a7098)

---

## 1.3 EKS

- Amazon EKS 관리형 서비스는 컨트롤 플레인을 직접 구성하지 않고서 k8s를 손쉽게 사용할 수 있도록 편리함을 제공
- AWS에서 제공하는 VPC, ELB, IAM 등 특정 기능들을 같이 활용하고자 할 때 유용
  ![image](https://github.com/smart-devops-org/devops-system/assets/46982752/d6c0f121-45dc-4e7b-a9b7-61ebe2744543)

## 1.4 EKS 특징

### VPC(Virtual Private Cloud) 와 통합

- 쿠버네티스 클러스터에서는 파드 네트워크로 워커 노드의 네트워크와는 다른 자체 네트워크 체계를 배치.
- 클러스터 자체 네트워크를 사용하기 때문에 명시적으로 엔드 포인트를 설정하지 않으면 클러스터 외부에서 파드에 통신이 불가능 =>([서비스명].[네임스페이스명].cluster.local)
- Amazon EKS에서는 Amazon VPC 통합 네트워킹을 지원하고 있어 파드에서 VPC 내부 주소 대역을 사용할 수 있고 클러스터 외부와의 통신이 가능하고 심리스(Seamless)하게 구현할 수 있다.

cf) 심리스(Seamless): 서비스 접근을 단순하게 하는 것 혹은 복잡한 기술이나 기능을 설명하지 않아도 서비스 기능을 직관적으로 구현하는 뜻

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/cdffe545-9f2b-471e-b90f-3a9a732f3d49)

### ELB와의 연계

- k8s 클러스터 외부에서 접속할 때는 서비스를 사용해 엔드포인트를 생성할 필요가 있음
- 가장 전형적인 엔드포인드가 로드밸런서
- EKS 에서는 k8s의 서비스 타입 중 하나인 LoadBalancer를 설정하면 자동적으로 AWS의 로드밸런서 서비스인 ELB가 생성
- HTTPS나 경로 기반 라우팅 등 L7 로드밸런서 기능을 AWS 서비스로 구현 가능
  ![image](https://github.com/smart-devops-org/devops-system/assets/46982752/a192c378-fa2a-4ac3-8cc3-0910fe90bbe7)

### IAM 을 통한 인증과 인가

- 쿠버네티스 클러스터는 kubectl 이라는 명령줄 도구를 사용하여 조작.
- 이때 해당 조작이 허가된 사용자에 의한 것음을 올바르게 인증해야 함.
- 또 인증된 사용자에게 어떤 조작을 허가할지에 대한 인가 구조도 필요
  ![image](https://github.com/smart-devops-org/devops-system/assets/46982752/271c7ab8-3cc0-4597-b19d-4393852e14dd)

### 워커 노드

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/0ea30909-6156-490c-be9a-6f98997835c1)

- EKS 클러스터의 유지 관리나 버전 업그레이드할 때 필요한 가상 머신 설정을 쉽게 해주는 관리형 노드 그룹 구조와 가상 머신을 의식하지 않고 파드를 배포할 수 있는 파게이트(Fargate)
- 하지만, 파게이트는 워커 노드 관리가 필요 없는 만큼 파드가 배포되는 호스트에 사용자 접근도 제한되며 거기에 따른 제약도 있다.

|           | EC2                                               | Fargate                                                             |
| --------- | ------------------------------------------------- | ------------------------------------------------------------------- |
| 관리 부하 | 높음                                              | 낮음                                                                |
| 제약      | 적음                                              | 많음                                                                |
| 특징      | 관리형 노드 그룹으로 EC2에 통합된 워커노드로 동작 | 워커 노드도 완전히 AWS가 관리하며 사용자는 EC2를 관리하지 않아도 됨 |

### AWS side workflow

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/233de1fe-7b8b-47ac-90d0-a1ac0a6880e2)

1. 쿠버네티스의 api 서버를 각 가용영역에 배포
2. api 데이터, 쿠버네티스의 상태 데이터를 확인하기 위해 etcd를 같이 배포
3. 쿠버네티스에서 오는 call 에 대한 IAM 구성
4. 쿠버네티스 마스터 노드의 오토스케일링 설정
5. 클러스터가 안정적으로 구현하도록 여기에 연결할 수 있는 로드밸런서를 구성

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/2d13f3c0-4bab-460b-8ec6-00ce9d4ad3a4)

### 참고 링크

- [Amazon EKS란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/what-is-eks.html)
- [Amazon EKS 란 ?](https://nearhome.tistory.com/128)

## 1.5 nodegroup (관리형 노드 그룹)

- 관리형 노드 그룹은 Amazon EKS Kubernetes 클러스터의 노드(Amazon EC2 인스턴스) 프로비저닝 및 수명 주기 관리를 자동화
- Amazon EKS 관리형 노드 그룹은 사용자를 위해 Amazon EC2 인스턴스를 생성하고 관리
- Amazon EKS에서 관리하는 Amazon EC2 Auto Scaling 그룹의 일부로 프로비저닝됩니다. 게다가 Amazon EC2 인스턴스 및 Auto Scaling 그룹을 포함한 모든 리소스는 AWS 계정 내에서 실행
- 관리형 노드 그룹의 Auto Scaling 그룹은 그룹을 생성할 때 지정하는 모든 서브넷에 걸쳐 있습니다.
- Amazon EKS는 관리형 노드 그룹 리소스를 태깅하여 Kubernetes Cluster Autoscaler를 사용하도록 구성

### 참고 링크

- [관리형 노드 그룹](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/managed-node-groups.html)
- [Amazon EKS 노드 그룹에 대한 역할 사용](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/using-service-linked-roles-eks-nodegroups.html)
- [관리형 노드 그룹 생성](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/create-managed-node-group.html)
- [AWS EKS란? 특징과 노드 구성 방식 + 배포방식](https://velog.io/@rockwellvinca/EKS)
- [EKS NodeGroup생성 전략 3가지](https://kim-dragon.tistory.com/54)

## 1.6 로드 밸런서

- Elastic Load Balancing은 둘 이상의 가용 영역에서 EC2 인스턴스, 컨테이너, IP 주소 등 여러 대상에 걸쳐 수신되는 트래픽을 자동으로 분산
  ![image](https://github.com/smart-devops-org/devops-system/assets/46982752/31c03c75-9944-45cd-a711-2684300cc111)

### ELB 의 기본 구성

- VPC 에 탑재 , 사용자의 요청을 받고 이를 VPC 내의 리소스에 적절히 부하 분산
- 외부의 요청을 받아들이는 리스너와 요청을 분산/전달할 리소스의 집합인 대상 그룹으로 구성
- ELB는 다수의 리스너와 대상 그룹을 거느릴 수 있음
- 대상 그룹 내의 리소스들은 헬스 체크를 활용해 끊임없이 상태를 확인 받음

> 리스너는 외부의 요쳥을 받아들이기 때문에 서비스 포트를 보유하고 있으며 외부의 요청은 서비스 포트로 접속하는 요청만을 처리

> 대상 그룹 또한 서비스 포트를 보유하고 있으며 부하분산 대상인 EC2는 해당 포트가 열려 있어야 함(NodePort) 사용 L4 스위치와 비교해보면 리스너는 Virtual Server에 , 대상 그룹은 Pool에 해당함

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/bb4e3567-f47b-4504-8bb6-ca729c1b6fa2)

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/680be043-94ef-40e7-a84e-3a608a4941ef)

### 참고 링크

- [AWS 로드 밸런싱](https://aws.amazon.com/ko/elasticloadbalancing/)
- [Application Load Balancer란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/elasticloadbalancing/latest/application/introduction.html)
- [ELB 쉽게 이해하기](https://aws-hyoh.tistory.com/128)
- [Amazon Web Service Network 쉽게 이해하기](https://aws-hyoh.tistory.com/category/Amazon%20Web%20Service%20Network%20%EC%89%BD%EA%B2%8C%20%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)

## 1.7 ELB의 종류

- ALB, NLB, CLB 3가지가 존재

### Application Load Balancer(ALB)

```
- OSI 7계층의 해당하는 로드밸런서
- HTTP, HTTPS의 특성을 주로 다루는 로드밸런서
- 단순 부하분산 뿐만 아니라 http의 헤더 정보를 이용해 부하분산을 가능
  - ec2 -> 로드밸런서 -> 리스너 -> 규칙 -> ALB 규칙 보기/편집
- HTTP의 헤더의 값을 보고 이 요청은 어느 대상 그룹으로 보낼지 판단 가능
  - ec2 -> 로드밸런서 -> 리스너 -> 규칙 -> ALB 규칙 보기/편집 -> 조건 추가
- 부하분산 방식 설정 가능
  - ec2 -> 로드밸런서 -> 대상 그룹 - 그룹 상세 - 속성 - 로드 밸런싱 알고리즘
- 인증서를 탑재할 수 있어 대상 그룹의 EC2를 대신하여 SSL 암호와/ 복호화를 대신 진행 가능(ACM 사용)
- 로드밸런서는 요청을 받으면 우선순위에 따라 리스너 규칙을 평가하여 적용할 규칙을 결정한 다음, 규칙 작업의 대상 그룹에서대상을 선택
- 프록시 서버로서의 역할
- AWS Web Aplication Firewal 과 연동(방화벽 서비스와 연동 가능)
```

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/c2cdddaa-fe4c-4d15-b61b-5d6802d0278e)

- http의 헤더 정보 상세 규칙 설정 가능

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/b7d9e607-6808-4beb-9a96-0c056aef8993)

- 부하분산

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/33dc295e-5610-42b8-b341-102775b00190)

- 인증서 탑재 가능

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/bcd871f2-6e68-4076-bf36-cbbfb8631656)

- 프록시 서버로서의 역할

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/3201bd22-c64b-4a0a-8a32-fcb05a24f2eb)

- 방화벽과 연동

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/97126e01-f2f5-4a4e-ad3d-9c63c5b8d385)

### Network Load Balancer(NLB)

-
