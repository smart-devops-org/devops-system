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
  - vpc를 적용하지 않는 구조일 경우 ec2가 많아질 수록 복잡도가 급격히 증가하는 형태가 됨
  - 보안 그룹 설정을 통해 데이터 및 자원을 보호
  - 각 자원을 격리해 외부 간섭 없이 운영 가능

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

- TCP와 UDP를 활용하는 OSI 4계층의 L4 로드밸런서

  - ec2 -> 로드밸런서 -> 리스너 - 리스너 편집
    ![image](https://github.com/smart-devops-org/devops-system/assets/46982752/5995d40c-5656-4745-9c4b-e105770ff5e9)

- NLB 는 ALB와 달리 TCP와 UDP 만 이해하면 되기 때문에 ALB 보다 시스템 리소스 소모가 적음
- 신경 쓸 부분이 적기 때문에 NLB는 ALB 보다 더 많은 트래픽 처리가 가능하고 뛰어난 성능을 보장함
- 로드밸런싱 방식은 프로토콜, 원본 IP 주소, 원본 포트, 대상 IP 주소, 대상 포트, TCP 시퀀스 번호에 따라 흐름 해시 알고리즘을 사용하여 대상을 선택 => 위 5-tuple 중 하나라도 다르면 새로운 요청으로 간주되어 새로운 커넥션을 생성
- 고정 아이피 사용 private ip, public ip 까지도 고정으로 제공
  - IP를 통한 접근 제어를 수행하고자 할 경우, 변하지 않는 IP는 매우 중요한 요소
  - 반면에 ALB의 Public IP를 목적지 삼아 접근 제어를 실시하는 네트워크 장비(방화벽..) 사용시 불편함
  - ALB의 이러한 점을 보완하기 위해 NLB를 ALB 앞단에 두고 고정된 IP를 제공하는 응용 방법이 있음
  - NLB가 고정된 IP를 제공하니 NLB public IP를 목적지 삼아 접근 제어 정책을 만들기 편리함

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/a9dcf08a-4495-45df-ba3d-fb0c2cf1689a)

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/dd6a04c4-01ba-4a1b-98c2-c5839c515051)

### 참고 링크

- [NLB 쉽게 이해하기](https://aws-hyoh.tistory.com/135)
- [AWS-ALB와-NLB-차이점](https://no-easy-dev.tistory.com/entry/AWS-ALB%EC%99%80-NLB-%EC%B0%A8%EC%9D%B4%EC%A0%90)

---

## 1.8 Route 53

- AWS 에서 제공하는 DNS 웹 서비스
- 도메인과 관련된 다양한 서비스를 제공

### NS 타입

- 네임서버를 등록 해당 네임서버를 통해 사용자가 요청했을 때 ip 정보를 반환

### SOA 타입

- 필수 레코드
- 도메인 영역을 표시하는 역할을 하며 네임서버에게 어떤 기준에 의해 도메인을 관리하여야 하는지 알려주는 역할

### A 타입

- 하나의 도메인에 해당하는 IP 주소의 값을 가지고 있음

### CNAME

- 하나의 도메인에 다른 이름을 부여하는 방식

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/7280075e-c24b-4344-bf1a-c45ff9ebe97b)

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/4d9ce236-746a-4477-aa86-0cdc6eafddf8)

### 참고 링크

- [Amazon Route 53은 무엇인가요?](https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/Welcome.html)

---

### 1.9 Auto-scale

- 자동 크기 조정은 변화하는 요구 사항에 맞게 리소스 규모를 자동으로 조정하는 기능
- Cluster Autoscaler
  Kubernetes Cluster Autoscaler는 포드가 실패하거나 다른 노드로 다시 예약될 때(pending 상태의 파드가 존재할 경우) 클러스터의 노드 수를 자동으로 조정

- karpanter를 활용한 부분은 추후 테스트 예정

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/1f24b22c-2375-40c6-b917-b4fc8ad349bc)

### 참고 링크

- [Amazon EKS 클러스터를 비용 효율적으로 오토스케일링하기](https://aws.amazon.com/ko/blogs/tech/amazon-eks-cluster-auto-scaling-karpenter-bp/)
- [EKS Autoscaling](https://velog.io/@xgro/aews-EKS-Study-5%EC%A3%BC%EC%B0%A8)

---

## 1.10 IAM

- 사용자의 AWS 리소스에 대한 액세스를 안전하게 제어하는데 도움이 되는 서비스
- AWS 사용자 및 그룹을 생성 및 관리하고 권한을 사용하여 AWS 리소스에 대한 액세스 권한을 부여하거나 거부 가능

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/dd583080-4dc0-475b-8059-4d770830301c)

### IAM 작동 방식

> 보안주체 -> 요청 -> 인증 -> 인가 -> 작업 또는 연산 -> 리소스

| 종류      | 설명                                                                                                                                                                                                                                 |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 보안 주체 | AWS 리소스에 대한 조치 또는 작업을 수행하도록 요청하는 AWS의 엔티티(사용자,서비스 또는 역할) 주체는 보안 자격 증명을 사용하거나 임시 보안 자격 증명으로 IAM 역할을 가정하여 인증                                                     |
| 요청      | 사용자, 서비스 또는 애플리케이션이 AWS 리소스에 대한 조치 또는 작업을 수행하도록 요청, 요청자는 보안 자격증명을 사용하거나 임스 보안 자격 증명으로 IAM 역할을 가정하여 인증된다                                                      |
| 인증      | IAM은 보안 주체와 연결된 권한을 확인 이러한 권한은 사용자, 그룹 또는 역할에 연결된 IAM 정책에 의해 정의 정첵은 어떤 조건에서 어떤 리소스에 대해 어떤 작업이 허용되거나 거부되는지 지정하는 JSON 문서                                 |
| 인가      | 보안 주체와 연결된 권한을 정의하는 정책을 기반으로 함, 정책은 사용자, 그룹 또는 역할에 연결할 수 있음, 보안 주체가 AWS 리소스에 대한 작업을 수행하도록 요청하면 IAM은 연결된 정책을 평가하여 요청이 허용되는지 또는 거부 되는지 결정 |
| 작업 연산 | 요청 엔티티가 평가된 정책을 기반으로 요청된 작업 또는 연산을 수행할 권한이 있는 경우 IAM은 요청을 진행하도록 허용, 그렇지 않으면 IAM이 요청을 거부하고 작업이 수행되지 않음                                                          |
| 리소스    | 요청이 IAM에 의해 승인된 경우 지정된 AWS 리소스에서 작업이 수행된다                                                                                                                                                                  |

### IAM 구성

1. 사용자

- 실제 AWS를 사용하는 사람 혹은 어플리케이션

2. 그룹

- 사용자의 집합으로, 그룹에 속한 사용자는 그룹에 부여된 권한을 행사, 동일한 의미에서 같은 권한을 가진 사용자들을 하나의 그룹으로 묶어 관리 > 각 devops, developer, sqa 등으로 권한을 나누어 관리

3. 정책

- 사용자와 그룹이 무엇을 할 수 있는가? 를 정의하는 문서, 정책을 JSON 형식으로 정의

4. 역할

- AWS 리소스에 부여하여 AWS의 리소스가 무엇을 할 수 있는가에 대해 정의

> IAM 자격 증명 공급자 유형 OIDC()

- OAuth 2.0 프로토콜 위에 있는 간단한 ID 계층으로 클라이언트가 권한 부여 서버에서 수행한 인증을 기반으로 최종 사용자의 ID를 확인 가능

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/a792e49e-e846-48ed-ba89-1b7f9ad6608f)

### 참고 링크

- [IAM이란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/introduction.html)
- [IAM 동작 방식](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/intro-structure.html)

---

## 1.11 EBS 란?

- mazon EC2 인스턴스에 연결하는 스토리지 볼륨입니다. 볼륨을 인스턴스에 연결하면 해당 볼륨을 컴퓨터에 연결된 로컬 하드 드라이브처럼 사용 가능

### EBS 기능

- 여러 가지 볼륨 유형
- 확장성
- 백업 및 복구
- 데이터 보호
- 데이터 가용성 및 내구성
- 데이터 보관

### 참고 링크

- [Amazon Elastic Block Store란 무엇인가요?](https://docs.aws.amazon.com/ko_kr/ebs/latest/userguide/what-is-ebs.html)
- [AWS-📚-EBS-개념-사용법-💯-정리-EBS-Volume-추가하기](https://inpa.tistory.com/entry/AWS-%F0%9F%93%9A-EBS-%EA%B0%9C%EB%85%90-%EC%82%AC%EC%9A%A9%EB%B2%95-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC-EBS-Volume-%EC%B6%94%EA%B0%80%ED%95%98%EA%B8%B0)

## 1. 12 ECR 이란?

- Docker 컨테이너 이미지를 손쉽게 저장, 관리, 및 배포할 수 있게 해주는 완전 관리형 Docker 컨테이너 레지스트리

### ECR 기능

- 수명 주기 정책을 통해 이미지의 수명 주기를 관리한다.
- 사용되지 않는 이미지를 정리하는 규칙을 정의하여 리포지토리에 적용할 수 있다.
- 각 리포지토리는 푸시 시 스캔하도록 구성할 수 있으며, 이 과정을 통해 컨테이너 이미지의 소프트웨어 취약성을 식별하는 데 도움을 줄 수 있다.
- 교차 리전 및 교차 계정 복제를 통해 이미지를 필요한 곳에 쉽게 배치할 수 있다.

### 참고 링크

- [Amazon Elastic Container Registry란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/AmazonECR/latest/userguide/what-is-ecr.html)
- [How-to-Set-up-Build-Process-Teamcity-And-ECR](https://oliveyoung.tech/blog/2022-06-15/How-to-Set-up-Build-Process-Teamcity-And-ECR/)

---

# 2 kubernetes 기본 Object 정리

## 2.1 Workload

- 쿠버네티스에서 구동되는 어플리케이션
- 워크로드가 단일 구성 요소든 함께 작동하는 여러 구성 요소이든 상관 없이 쿠버네티스에서는 파드 세트 네에서 실행함
- Pod의 set을 관리하는 Workload Resources를 사용할 수 있다

### Cluster

- 컨테이너 형태의 어플리케이션을 호스팅하는 물리/가상 환경의 노드들로 이루어진 집합

| 마스터 노드(컨트롤 플레인)                                                                                                                                                                | 워커 노드                                                                                                              |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| 컨테이너를 통제하는 역할, 대규모의 컨테이너를 운영하려면 각 워커 노드들의 가용 리소스 현황을 고려하여 최적의 컨테이너 배치와 모니터링 그리고 각 컨테이너에 대한 효율적인 추적 관리를 수행 | 다른 컨테이너들이 선적된 컨테이너 선의 역할, 각기 다른 목적과 기능으로 세분화된 컨테이너들이 실제 배치되는 노드를 의미 |

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/2279767b-0b1d-4803-9a0e-f2294412abad)

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/3cf6a8fc-ebec-45d7-a0fa-323a78d1b1e1)

- 마스터 노드
  | 구성요소 | 설명 |
  | ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
  | etcd | 클러스터 안의 각 구성요소들에 대한 정보가 키-값 형태로 저장된 데이터베이스 |
  | 스케줄러(kube-scheduler) | 애플리케이션 구동에 필요한 각 컨테이너에 대해 클러스터 내 최적의 배포를 수행하는 스케줄러 |
  | 컨트롤러 매니저(kube-controller-manager) | 노드(Node), 디플로이먼트(Deployment), 서비스 어카운트(Service Account) 등 클러스터에서 구동되는 리소스들을 유지 관리하는 프로세스들의 집합 |
  | DNS 서버 | 클러스터 안에서 특정 도메인을 찾을 때 사용되는 네임 서버 (구성도에는 kube-dns라 되어 있으나, 쿠버네티스 1.12 버전부터는 CoreDNS로 대체됨) |
  | API 서버(kube-apiserver) | 클러스터 구성 요소들의 상호 통신에 필요한 쿠버네티스 API를 관리하는 컴포넌트 |

- 워커 노드
  | 구성요소 | 설명 |
  | ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
  | kubelet | 클러스터의 각 노드에서 API 서버를 통해 들어오는 신호를 모니터링하고 파드에서 컨테이너가 제 기능대로 정상 동작하도록 관리하는 에이전트 |
  | kube-proxy | 클러스터의 각 노드에서 실행되는 네트워크 프록시 서비스 |
  | 컨테이너 런타임 엔진 | 노드에 배포된 파드(Pod) 내 컨테이너들을 구동시키는 엔진 |

### node

- 컨트롤 플레인에서 할당된 요청 태스크를 수행하는 머신
- 클러스터 = 마스터 노드 + 워커 노드

### taints && toleration

- 특정 노드에 테인트를 설정 가능, 노드가 파드셋을 제외하도록 하는 노드에 설정하는 속성
- 기본적으로 테인트를 설정한 노드는 파드들을 스케줄링 하지 않음
- 스케줄링하려면 파드에 톨러레이션을 설정해야함
- 테인트와 톨러레이션은 주로 노드를 특정 역할만 하도록 만들 때 사용
- . GPU가 있는 노드에는 실제로 GPU 자원을 사용하는 파드들만 실행되도록 설정

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/78776a71-0267-416b-9cd7-9d1409a02357)

### 참고 자료

- [kubernetes-cluster-components](https://seongjin.me/kubernetes-cluster-components/)

### Nodes

> 워크로드 resource를 사용하여 파드의 라이프사이클을 관리

### Deployments && ReplicaSet

- pod, replicaSet이 어떻게 구성되어야 하는지 정의
- replicaset: 지정된 수의 파드가 항상 실행되도록 보장
- 버전 업데이트 등으로 인해 원하는 정의가 변경되었을 때는 현재 상태에서 원하는 상태로 바뀌도록 변경
- 변경 사항을 저장하는 revision을 남겨서 문제 발생 시에 이전 버전으로 롤백도 가능

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/44e3e892-e38d-4829-b7ba-8ed84cc6d881)

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: test-replicaset
spec:
  template:
    metadata:
      name: test-replicaset
      labels:
        app: test-replicaset
    spec:
      containers:
      - name: test-replicaset
        image: nginx
        ports:
        - containerPort: 80
  replicas: 3
  selector:
    matchLabels:
      app: test-replicaset
```

```
- apiVersion apps/v1 → 쿠버네티스의 apps/v1 API를 사용
- kind: ReplicaSet → ReplicaSet의 작업으로 명시
- metadata.name → Replicaset 이름을 설정
- spec.template.metadata → 어떤 파드를 실행할지에 대한 정보를 하위에 설정 
- spec.template.metadata.name → 생성될 파드의 이름을 지정
- spec.template.metadata.labels.app:test-replicaset → 식별하는 레이블이 앱 컨테이너이며 test-replicaset 으로 식별
- spec.spec → 이 하위의 옵션들은 컨테이너에 대한 설정을 합니다. 위 코드에선 컨테이너 명, 이미지, 포트를 지정 했다.
- replicas → 파드의 개수를 몇개 유지할 것 인지 설정, 기본값은 1 
- selector → 어떤 레이블의 파드를 선택하여 관리할지에 대한 설정. 앱 컨테이너의 test-replicaset 레이블을 식별하여 해당되는 파드들을 관리하며, 이 필드가 없을 경우 spec.template.metadata.labels.app 에 적은 내용들을 기본값으로 사용
```

### StatefulSets

- Deployment와 달리 Stateful한 pod를 관리하기 위한 controller
- pod들의 고유성과 순서를 보장
- Database는 마스터노드가 기동된 후에 워커노드가 순차적으로 기동되어야하는 경우가 많은데, 이런 경우에 사용한다.
- 개별 포드가 Persistent Volumn(PV)을 생성하여 연결하도록 실행한다. Pod가 비정상 종료된 경우에 새 Pod가 기존 Pod에 연결된 PV를 담당하게

### DaemonSet

- 모든 노드에 동일한 파드를 실행시키고 싶은 경우 활용
- 리소스 모니터링, 로그 수집기 등에 유용. 클러스터에 노드가 추가/ 삭제 되면 자동으로 파드도 생성/ 삭제됨
- 모든(또는 일부) 노드가 파드의 사본을 실행하도록 한다. 노드가 클러스터에 추가되면 파드도 함께 추가된다. 노드가 클러스터에서 제거되면 해당 파드는 가비지(garbage)로 수집된다. 데몬셋을 삭제하면 데몬셋이 생성한 파드들이 함께 정리

- 용도
  ```
  모든 노드에서 클러스터 스토리지 데몬 실행
  모든 노드에서 로그 수집 데몬 실행
  모든 노드에서 노드 모니터링 데몬 실행
  ```

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/9981cb38-3a76-43b0-872b-1299002befb4)

### Jobs/CronJob

- Job은 하나 이상의 Pod를 지정하고 지정된 수의 파드를 성공적으로 실행하고 종료될 때까지 포드의 실행을 재시도 한다.
- 노드의 H/W 장애나 재부팅 등으로 인해 파드가 정상 실행이 되지 않았을 경우 Job은 새로운 파드를 시작하도록 할 수 있다.
- 즉, 백업이나 특정 배치 파일들처럼 한번 실행하고 종료되는 성격의 작업에 사용될 수 있다.
- 일반적으로 Pod가 시작되면 항상 실행하는 것을 기대하는 것과 달리 Job은 실행되면 몇분 또는 몇 시간, 몇일 뒤에 종료되는 것 (또는 주기적으로 실행되는 작업)을 실행한다. 그리고 실행이 잘 되었다는 결과를 사용자가 확인 가능(Automatic Clean-up for Finished Jobs)

- CronJob은 지정한 일정에 따라 반복적으로 Job을 생성

## 2.2 서비스

- 파드를 통해 실행되고 있는 애플리케이션을 네트워크에 노출시키는 가상의 컴포넌트
- 쿠버네티스의 파드는 무언가 구동 중인 상태를 유지하기 위해 동원되는 일회성 자원으로 언제든 다른 노드로 옮겨지거나 삭제될 수 있음
- 파드는 생성될 때마다 새로운 내부 IP를 받게 되므로, 이것만으로 클러스터 내/외부와 통신을 계속 유지하기 어려움
- 파드가 외부와 통신할 수 있도록 클러스터 내부에서 고정적인 IP를 갖는 서비스를 이용하여 여러 파드들에게 단일한 네트워크 진입점을 부여하는 역할
  > targetPort: 파드의 애플리케이션쪽에서 열려있는 포트를 의미 서비스로 들어온 트래픽은 해당 파드의 <클러스터 내부 IP>:<targetPort>로 넘어가게 됨

> port : 서비스 쪽에서 해당 파드를 향해 열려있는 포트를 의미한다.

- 아래와 같은 서비스 타입으로 구분됨

### ClusterIP

- ClusterIP는 파드들이 클러스터 내부의 다른 리소스들과 통신할 수 있도록 해주는 가상의 클러스터 전용 IP다. 이 유형의 서비스는 <ClusterIP>로 들어온 클러스터 내부 트래픽을 해당 파드의 <파드IP>:<targetPort>로 넘겨주도록 동작하므로, 오직 클러스터 내부에서만 접근 가능하게 된다. 쿠버네티스가 지원하는 기본적인 형태의 서비스

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: ClusterIP
  ports:
  - name: http
    protocol: TCP
    targetPort: 9376
    port: 80
  - name: https
    protocol: TCP
    targetPort: 9377
    port: 443
  selector:
    app: myapp
    type: frontend
```

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/25fd1205-b454-48ff-8504-e87b83a783ea)

### NodePort

- NodePort는 외부에서 노드 IP의 특정 포트(<NodeIP>:<NodePort>)로 들어오는 요청을 감지하여, 해당 포트와 연결된 파드로 트래픽을 전달하는 유형의 서비스다. 이때 클러스터 내부로 들어온 트래픽을 특정 파드로 연결하기 위한 ClusterIP 역시 자동으로 생성
- 할당 가능한 포트 번호의 범위는 30000에서 32767 사이

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/cd6c5da7-c292-4a67-9330-606e00afdaed)

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
  - targetPort: 80		# 애플리케이션(파드)을 노출하는 포트
    port: 80			# 서비스를 노출하는 포트
    nodePort: 30008		# 외부 사용자가 애플리케이션에 접근하기 위한 포트번호(선택)
  selector:				# 이 서비스가 적용될 파드 정보를 지정
    app: myapp
    type: frontend
```

### LoadBalancer

- 별도의 외부 로드 밸런서를 제공하는 클라우드(AWS, Azure, GCP 등) 환경을 고려하여, 해당 로드 밸런서를 클러스터의 서비스로 프로비저닝할 수 있는 LoadBalancer 유형도 제공

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/747d52c9-06c4-4b21-aac8-c3cff647d94f)

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 80				# 서비스를 노출하는 포트
      targetPort: 80		# 애플리케이션(파드)를 노출하는 포트
  clusterIP: 10.0.171.239	# 클러스터 IP
  selector:
    app: myapp
    type: frontend
status:
  loadBalancer:				# 프로비저닝된 로드 밸런서 정보
    ingress:
    - ip: 192.0.2.127
```

### ExternalName

- 서비스에 selector 대신 DNS name을 직접 명시하고자 할 때 사용
- 필요한 DNS 주소를 기입하면, 클러스터의 DNS 서비스가 해당 주소에 대한 CNAME 레코드를 반환

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

### Ingress

- 클러스터 외부의 HTTP 및 HTTPS 경로를 클러스터 내의 서비스 에 노출합니다. 트래픽 라우팅은 Ingress 리소스에 정의된 규칙에 따라 제어

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/99a64b84-aaf7-469f-9967-c00c1c8fb04c)

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80
```

### 참고 링크

- [Service](https://kubernetes.io/docs/concepts/services-networking/service/?ref=seongjin.me)
- [Kubernetes NodePort vs LoadBalancer vs Ingress? When should I use what?](https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0)

## 2.3 Storage

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/9dde393c-dca6-436f-979d-382d15e829b8)

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/4bfdbcd0-1b18-4c1c-8483-ce6245d994e5)

| pv                 | pvc       | storageclass                                                       |
| ------------------ | --------- | ------------------------------------------------------------------ |
| 스토리지 볼륨 정의 | pv를 요청 | 디스크 생성시 디스크 타입을 정의하도록 동적 프로비저닝 방식을 이용 |

- 볼륨(pv)와 클레임(pvc) 에 대한 생명 주기

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/4143f0ad-3295-46ec-ae47-5ee88723bc68)

### 참고 링크

- [스토리지클래스(StorageClass)와 PV와 PVC의 생명주기](https://velog.io/@rockwellvinca/kubernetes-%EC%8A%A4%ED%86%A0%EB%A6%AC%EC%A7%80%ED%81%B4%EB%9E%98%EC%8A%A4StorageClass%EC%99%80-PV%EC%99%80-PVC%EC%9D%98-%EC%83%9D%EB%AA%85%EC%A3%BC%EA%B8%B0)

## 2.4 config

### ConfigMap

- 일반적으로 컨테이너를 사용할 때, 개발과 서비스 사이의 차이로 발생하는 문제점을 방지하기 위해 dev, production용 컨테이너는 같아야 함
- 하지만, 각각의 컨테이너가 서로 다른 설정이 필요한 경우가 있음

### Secret이란

- password, OAuth token, ssh key와 같은 민감한 정보를 저장하는 용도로 사용
- configMap을 포함한 일반적인 오브젝트 값을 쿠버네티스 DB에 저장되는 반면, Secret은 메모리에 저장되며 1Mbute의 제한을 가짐

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/34400a2a-a021-4722-aa7f-6845b71cffa5)

### ConfigMap, Secret의 3가지 사용방법

1. 상수를 환경변수로 사용
   ![image](https://github.com/smart-devops-org/devops-system/assets/46982752/5a1512d6-3ba3-4340-9d31-494f1047a60e)

2. 파일을 환경변수로 사용
   ![image](https://github.com/smart-devops-org/devops-system/assets/46982752/c9ef7910-47f2-4b63-8092-ec31e576ed0e)
3. Pod내부의 파일로 마운트해 사용
   ![image](https://github.com/smart-devops-org/devops-system/assets/46982752/701b76b1-a8ee-4b2f-b31b-8aa4d0a28b73)

## 2.5 네임스페이스

### namespace

- 리소스를 논리적으로 나누기 위한 방법을 제공
- Namespace는 이름의 범위를 제공 - 어떠한 Pod나 Deployment 등의 이름은 NameSpace내에서는 유일해야함
- 즉, 같은 클러스터 내에서 운용되는 애플리케이션이지만 독립된 클러스터에서 운용되는 것처럼 지원

### 클러스터 기본 namespace

| default                                                                    | kube-system                                                                     | kube-public                                                                                        | kube-node-lease                                                                                                                                                                       |
| -------------------------------------------------------------------------- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Namespace를 지정하지 않은 경우에 기본적으로 할당되는 Namespace이다         | 쿠버네티스 시스템에 의해 생성되는 API 오브젝트들을 관리하기 위한 Namespace이다. | 클러스터 내 모든 사용자로부터 접근 가능하고 읽을 수 있는 오브젝트들을 관리하기 위한 Namespace이다. | 쿠버네티스 클러스터 내 노드의 연결 정보를 관리하기 위한 Namespace이다.                                                                                                                |
| 지금까지 사용해왔던 방식이다. (지금까지 Namespace를 지정했던 적은 없었다.) | 클러스터 레벨의 관리자 영역의 Namespace라고 이해하면 좋다.                      | 사용자가 직접 다루지는 않지만 누구든 접근 가능하기 때문에 주의할 필요는 있다.                      | 역시 사용자가 직접 다루기보단, 쿠버네티스 자체적으로 컨트롤플레인과 노드간의 연결을 잘 하기 위해서 lease라는 오브젝트를 관리하게 되는데 => 즉 lease를 잘 관리하기 위한 Namespace이다. |

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/5e7b3342-643e-40a8-abed-b5abfbf82cd1)

### ResourceQuota

- 네임스페이스마다 오브젝트가 사용할 수 있는 자원의 제한을 걸어둘 수 있음
- Namepace 마다 총 리소스의 사용을 제한하는 제약조건의 역할을 함

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-1
  namespace: nm-3
spec:
  hard:
    requests.memory: 1Gi
    limits.memory: 1Gi
```

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/1ac4f4d9-8e09-4293-8d94-19196574ae61)

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/1aa641ab-e6f4-44bd-8833-62be5175a2e4)

### LimitRange

- 기본적으로 컨테이너는 쿠버네티스 클러스터에서 무제한 컴퓨팅 리소스로 실
- 클러스터 관리자는 네임스페이스별로 리소스 사용과 생성을 제한
- 하나의 파드 또는 컨테이너가 사용 가능한 모든 리소스를 독점할 수 있다는 우려가 있습니다.
- LimitRange는 이를 해결할 수 있는 도구로, 네임스페이스에서 움직이는 포드 하나하나의 리소스량의 상한을 설정합니다.

```
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-1
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.4Gi
    maxLimitRequestRatio:
      memory: 3
    defaultRequest:
      memory: 0.1Gi
    default:
      memory: 0.2Gi
```

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/b33194bd-c8a5-4689-86ba-e2039ef038ce)

---

## 2.6 Access Control

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/24f898ec-3b95-40fa-99f6-f383eeae7f4e)

### 인증

- 사용자는 api server에 인증 파라미터를 같이 포함하여 요청을 보냄
  - 인증을 할 수 없으면 401 상태와 함께 거부됨

```
# 어떤 인증 방법을 사용하는지 확인 가능
k -v=7 get no
```

- serviceaccount 생성 후 토큰 확인

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: test-a
  namespace: default
secrets:
  - name: secret-sa-test-a
```

```
apiVersion: v1
kind: Secret
metadata:
  name: secret-sa-test-a
  namespace: default
  annotations:
    kubernetes.io/service-account.name: "test-a"
type: kubernetes.io/service-account-token

```

- 토큰을 포함하여 http 요청으로 확인
  - 해당 부분까지는 권한을 아직 안넣었으므로 403으로 반환된 role, rolebinding 등으로 필요한 권한을 넣어줘야함

```
curl -k -X GET -H "Authorization: Bearer 토큰" https://3929D75683562F7DFBE99C3BBA94C090.gr7.ap-northeast-2.eks.amazonaws.com/api/v1/namespace/default/pods
```

![image](https://github.com/smart-devops-org/devops-system/assets/46982752/cb90009a-2db9-4dea-bbab-cf4829506c34)

### 어플리케이션 운영의 인증 관리

- 운영자는 serviceAccount의 secret 토큰을 주입(마운트)
- 개발자는 토큰을 사용하여 인증을 한다는 것을 알고 있어야 함
- 관리자는 적절한 권한 관리 설정이 필요

> 위와 같은 부분으로 각 권한에 맞는 그룹 developer, devops, sqa 등으로 service account를 만들고 각 그룹에 맞는 권한을 세팅해 주는 식으로 사용 가능

### Role & RoleBinding을 이용한 권한 제어

- Role

  - 어떤 API를 이용할 수 있는지 정의
  - 쿠버네티스의 사용권한을 정의
  - 지정된 네임스페이스에서만 유효

- RoleBinding
  - 사용자/그룹 또는 Service Account와 역할을 연결
