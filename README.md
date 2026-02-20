# 강의
- Inflearn > eks를 활용한 spring 운영서버 배포 (feat. devops의 모든것)

## 정리
### 2. docker 개요
- 도커란?  
  - 애플리케이션을 개발, 배포, 실행을 용이하게 하는 오픈 소스 플랫폼
  - 소프트웨어를 컨테이너라는 표준화된 단위로 패키징하여 어디서든 동일하게 실행 가능
- 도커 이미지(Image)
  - 애플리케이션과 모든 필요한 설정을 포함하는 불변(Immutable)의 템플릿
  - 컨테이너를 실행하기 위한 압축파일과 같은 개념
  - 이미지를 기반으로 실제 실행 단위인 도커 컨테이너가 생성됨
  - 이미지 다운로드(pull)
    - docker pull nginx
  - 이미지 빌드(build)
    - docker build -t 이미지명:태그 .
    - docker build -t 이미지명:태그 -f DockerfileDev .
    - 빌드 컨텍스트란 Docker가 참조할 수 있는 파일과 디렉터리의 위치
    - 현재 위치라면 "." 으로 대체 가능
- 도커 허브(Docker Hub)
  - 다양한 도커 이미지를 찾고 공유할 수 있는 공개 저장소
  - 공식 이미지 및 사용자 제작 이미지를 다운로드 가능
  - 이미지 태그 및 빌드
    - docker build -t username/imagename:tag .
    - 기존 이미지 태그 변경: docker tag 이미지명 username/imagename:tag
  - 이미지 업로드(push)
    - docker push username/imagename:tag
    - 사전에 해당 repository 생성 필요
  - 이미지 다운로드(pull)
    - docker pull username/imagename:tag
- 컨테이너(Container)
  - 이미지로부터 생성되는 실제 실행 인스턴스
  - 라이브러리, 시스템 도구, 코드 등 실행에 필요한 모든 요소 포함
  - 컨테이너 실행(run)
    - docker run -d -p 8080:80 nginx

### 3. spring 빌드환경 이해
- MySQL 설정
  - 컨테이너 실행
    - docker run --name mysql-container -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=1234 mysql:8.0 --default-authentication-plugin=mysql_native_password
  - 스키마 생성
    - create database ordersystem;
- Redis 설정
  - 컨테이너 실행
    - docker run --name redis-container -d -p 6379:6379 redis
- Spring 설정
  - 소스코드 추가
    - 2.ordersystem 폴더 및 소스코드 추가

### 4. docker 이미지 빌드
- 이미지 빌드
  - Dockerfile을 통해 이미지를 생성한다.
  - 명령어
    - docker build -t 이미지명:[태그명] -f <도커 스크립트 파일경로> <빌드 컨텍스트 위치>
      - <도커 스크립트 파일경로>
        - 현재 위치가 Dockerfile이 있는 경로이고, 파일명이 기본값인 Dockerfile이라면 생략 가능하다.
        - DockerfileDev 등 파일명이 다를 경우에는 -f 옵션으로 파일명을 명시해야 한다.
      - <빌드 컨텍스트 위치>
        - Docker 이미지를 빌드할 때 참조할 수 있는 모든 파일과 디렉터리의 위치를 의미한다.
        - 현재 위치에 빌드 대상 파일이 있다면 "." 으로 대체 가능하다.
    - 예시
      - docker build -t demospring:v1.0 .

### 5. docker와 docker-compose 실행
- spring 컨테이너 실행 시 환경설정 주의 사항
  - Spring과 DB를 각각 Docker 컨테이너로 실행할 경우, application-local.yml의 DB 접속 정보를 localhost로 설정하면 정상적으로 연결되지 않는다.
  - 그 이유는 컨테이너 내부에서의 localhost는 해당 컨테이너 자기 자신을 의미하기 때문이다.
  - 즉, Spring 컨테이너에서의 localhost는 Spring 컨테이너 자신을 가리키므로, 별도로 실행된 DB 컨테이너에 접근할 수 없다.
  - 따라서 다음과 같이 설정해야 한다.
    - docker-compose 사용 시 → DB 컨테이너 이름(mysql-container 등)을 host로 지정
    - 로컬 DB에 접근할 경우 → host.docker.internal 사용
- 참고
  - DB는 Docker로 실행하고, Spring은 IntelliJ(로컬)에서 실행하는 경우에는 localhost로 접근 가능하다.
  - 이는 Docker의 포트 매핑(-p 3306:3306)을 통해 호스트와 컨테이너가 연결되기 때문이다.
- docker compose 활용
  - docker-compose란
    - 일반적으로 로컬 환경에서 복수의 컨테이너(Spring, MySQL, Redis 등)를 일괄 기동할 때 사용하는 도구이다.
    - docker 설치 시 함께 설치되며, docker-compose.yml 파일을 기반으로 동작한다.
    - 여러 컨테이너를 하나의 네트워크로 묶어 docker 간 통신이 가능하도록 구성할 수 있다.
    - 동일 네트워크에 속한 컨테이너는 컨테이너 이름을 host처럼 사용할 수 있다.
      - 예) jdbc:mysql://mysql-container:3306/ordersystem
  - docker-compose의 장점
    - 위에서 설명한 컨테이너 간 localhost 통신 문제를 해결할 수 있다.
      - docker-compose를 사용하지 않으면, Spring을 Docker로 실행하는지 IntelliJ(로컬)에서 실행하는지에 따라 application-local.yml의 host 정보를 매번 변경해야 한다.
      - docker-compose를 사용하면 컨테이너 이름 기반 통신이 가능하여 이러한 설정 변경이 필요 없다.
    - 컨테이너 이름을 기반으로 서비스 간 통신이 가능하여 별도의 IP 설정이 필요 없다.
    - 여러 컨테이너를 하나의 설정 파일로 관리할 수 있어 환경 구성이 단순해진다.
    - 개발 환경을 코드로 관리할 수 있어, 동일한 실행 환경을 쉽게 재현할 수 있다.
    - Spring, MySQL, Redis 등 여러 서비스를 하나의 명령어로 일괄 실행 및 종료할 수 있다.
  - 실행
    - docker-compose up -d
      - 정의된 모든 컨테이너를 백그라운드에서 일괄 실행한다.
    - docker-compose up -d --build
      - 이미지 변경 사항이 있을 경우 새로 빌드 후 실행한다.
  - 중지 및 삭제
    - docker-compose down
      - 실행 중인 컨테이너를 일괄 중지하고 네트워크를 함께 정리한다.

### 6. 쿠버네티스 개요
- 쿠버네티스란?
  - 컨테이너화된 애플리케이션을 대규모로 운영하기 위한 컨테이너 오케스트레이션 도구이다.
  - 주로 Docker와 같은 컨테이너 기술과 함께 사용되며, 여러 컨테이너를 자동으로 배포·확장·관리하는 플랫폼이다.
  - 쿠버네티스는 클러스터 기반으로 구성되며, 클러스터는 컨테이너화된 애플리케이션을 실행하기 위한 여러 컴퓨터(노드)의 집합이다.
- 주요 구성 요소
  - 주요 구성 요소
    - ![kubernetes_architecture.png](image/kubernetes_architecture.png)
  - 클러스터
    - 쿠버네티스의 전체 실행 환경을 의미한다.
    - 모든 노드와 쿠버네티스 API 서버 및 관련 구성 요소를 포함한다.
    - 하나의 전체 환경 단위를 하나의 클러스터라고 한다.
    - 클러스터는 크게 마스터 노드(Control Plane)와 워커 노드로 구성된다.
  - 마스터 노드(Control Plane)
    - 클러스터의 관리와 제어를 담당한다.
    - 주요 구성 요소
      - API 서버 (kube-apiserver)
        - 클러스터의 중앙 집중식 관리 시스템이다.
        - 사용자, 내부 구성 요소, 외부 도구가 쿠버네티스 API를 통해 통신하는 진입점이다.
        - 개발자의 명령어(kubectl 등)는 API 서버로 전달된다.
      - 스케줄러 (kube-scheduler)
        - 새로 생성된 파드를 감지하고, 실행할 최적의 워커 노드를 선택한다.
      - etcd
        - 클러스터의 상태 저장소이다.
        - ConfigMap, Secret 등 모든 클러스터 상태 값을 key-value 형태로 저장하는 고가용성 분산 데이터베이스이다.
      - kube-controller-manager
        - 클러스터의 상태 유지를 담당한다.
        - Deployment, ReplicaSet 등을 감시하며 원하는 상태(desired state)와 실제 상태를 일치시키도록 조정한다.
    - 마스터 노드는 안정성과 고가용성이 매우 중요하다.
    - AWS EKS와 같은 관리형 서비스는 마스터 노드의 생성 및 관리를 자동으로 지원한다.
  - 워커 노드
    - 실제 애플리케이션(파드)이 실행되는 노드이다.
    - 각 워커 노드에는 Kubelet이라는 에이전트가 실행되어 마스터 노드와 통신한다.
    - AWS 환경에서는 EC2 인스턴스가 워커 노드 역할을 수행하며, 해당 노드에서 서비스 파드가 실행된다.
    - 주요 구성 요소
      - kubelet
        - API 서버와 통신하여 파드를 시작·중지·관리한다.
        - 파드 상태를 주기적으로 API 서버에 보고한다.
      - kube-proxy
        - 네트워크 프록시 및 로드 밸런서 역할을 수행한다.
        - 네트워크 규칙을 설정하여 파드 간 통신 및 외부에서 파드로의 접근을 가능하게 한다.

### 7. 쿠버네티스 주요 구성 요소
- 구성 요소
  - ![쿠버네티스_핵심요소.png](image/%E1%84%8F%E1%85%AE%E1%84%87%E1%85%A5%E1%84%82%E1%85%A6%E1%84%90%E1%85%B5%E1%84%89%E1%85%B3_%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%AD%E1%84%89%E1%85%A9.png)
- 쿠버네티스 핵심 요소
  - 네임스페이스 (Namespace)
    - 클러스터 내 리소스를 논리적으로 분리하는 단위
    - 하나의 클러스터 안에서 개발/운영/테스트 환경을 구분할 때 사용
  - Pod
    - 쿠버네티스에서 배포할 수 있는 가장 작은 단위
    - 1개 이상의 컨테이너로 구성된 배포 단위
    - 일반적으로는 1개의 컨테이너로 구성
  - Service
    - 서비스 디스커버리를 위한 리소스(Resource)
    - 클러스터 내에서 실행 중인 Pod들에 대한 라우팅 및 로드밸런싱 제공
    - 기본 타입은 ClusterIP로, 내부 통신 전용
    - LoadBalancer 등 외부에서 직접 접근 가능한 타입도 존재
  - Ingress
    - 클러스터 외부에서 내부 Service로 트래픽을 라우팅하는 리소스
    - 외부 요청을 내부 Service에 연결해주는 규칙 정의
    - 도메인(Host) 기반 라우팅 지원
  - ReplicaSet
    - 지정된 수의 Pod 복제본이 항상 실행되도록 보장하는 선언적 리소스
    - 장애 발생 시 자동으로 새로운 Pod 생성
  - Deployment
    - ReplicaSet을 관리하는 상위 리소스
    - 단순 개수 유지뿐 아니라 롤링 업데이트, 롤백 등 배포 전략 지원
    - 무중단 배포 및 버전 관리에 활용
  - ConfigMap / Secret
    - ConfigMap
      - 설정 파일과 같은 일반 데이터를 Key-Value 형태로 관리
      - 민감하지 않은 데이터 저장
    - Secret
      - 비밀번호, 인증서 등 민감 정보를 저장
      - Base64 인코딩 형태로 관리
- 주요 특징 및 장점
  - 자동 복구(Self-Healing)
    - 컨테이너 또는 Pod 장애 발생 시 자동 재시작
    - 원하는 상태(Desired State)를 유지하도록 지속적으로 조정
  - 서비스 발견 및 로드밸런싱
    - Service를 통해 내부 DNS 기반 서비스 디스커버리 제공
    - 트래픽을 여러 Pod로 자동 분산
  - 네임스페이스 기반 멀티 환경 지원
    - 개발/운영/테스트 환경을 논리적으로 분리 가능
  - 오토 스케일(Auto Scaling)
    - HPA(Horizontal Pod Autoscaler)
      - CPU/메모리 등 자원 사용률 기준으로 Pod 자동 증설/감소
    - Cluster Autoscaler
      - 노드 자원이 부족할 경우 인스턴스 자동 증설/감소
  - 기타 기능
    - 자동화된 롤아웃 배포(무중단 배포)
    - Pod별 CPU, 메모리 자원 요청/제한 설정 지원

### 8. AWS 핵심요소 - EC2
- Amazon EC2 (Elastic Compute Cloud)
  - AWS에서 제공하는 가상 서버(컴퓨팅) 서비스
  - 다양한 운영 체제(Linux, Windows 등)와 애플리케이션을 실행할 수 있는 확장 가능한 인프라
  - 사용자는 필요한 사양(CPU, 메모리, 스토리지 등)에 맞는 인스턴스 타입을 선택하여 생성 가능
  - 온디맨드, 예약 인스턴스, 스팟 인스턴스 등 다양한 과금 방식 지원
- EC2 주요 개념
  - 인스턴스(Instance)
    - EC2에서 생성한 가상 서버 1대를 의미
    - 선택한 인스턴스 타입에 따라 성능과 비용이 달라짐
  - AMI (Amazon Machine Image)
    - 인스턴스를 생성하기 위한 템플릿 이미지
    - OS, 애플리케이션, 설정 정보 등을 포함
  - 키 페어(Key Pair)
    - EC2 인스턴스에 SSH로 접속하기 위한 인증 수단
    - 공개키/개인키 방식 사용
  - EBS (Elastic Block Store)
    - EC2에 연결하여 사용하는 블록 스토리지
    - 인스턴스 종료 후에도 데이터 유지 가능(옵션 설정에 따라 다름)
- 네트워크 및 보안 관련 핵심 요소
  - 보안 그룹(Security Group)
    - 인스턴스 단위의 가상 방화벽
    - 인바운드(들어오는 트래픽) 및 아웃바운드(나가는 트래픽) 규칙 정의
    - 예: 22번 포트(SSH), 80번 포트(HTTP), 443번 포트(HTTPS) 허용 설정
    - 기본적으로 허용된 트래픽만 통과시키는 화이트리스트 방식
  - 탄력적 IP(Elastic IP)
    - 고정 공인 IP 주소
    - 인스턴스가 재시작되더라도 동일한 공인 IP 유지 가능
- 로드 밸런서(Elastic Load Balancer, ELB)
  - 여러 EC2 인스턴스에 트래픽을 분산시키는 서비스
  - 고가용성 및 확장성 확보에 핵심 역할
  - 로드 밸런서 유형
    - ALB (Application Load Balancer) – HTTP/HTTPS 기반
    - NLB (Network Load Balancer) – TCP/UDP 기반 고성능 처리
  - 타겟 그룹(Target Group)
    - 로드 밸런서가 트래픽을 전달할 백엔드 리소스(EC2 등)의 그룹
    - 헬스 체크를 통해 정상 인스턴스에만 트래픽 전달
- 오토 스케일링(Auto Scaling)
  - 트래픽 증가/감소에 따라 EC2 인스턴스를 자동으로 증설/감소
  - CloudWatch 지표(CPU 사용률 등)를 기준으로 동작
  - 고가용성과 비용 효율성을 동시에 확보 가능
- EC2 활용 관점 정리
  - 애플리케이션 서버, API 서버, 배치 서버 등 다양한 용도로 사용
  - 쿠버네티스(EKS) 환경에서는 워커 노드로 활용
  - 보안 그룹, 로드 밸런서, 오토 스케일링과 함께 구성하여 안정적인 아키텍처 설계 가능

### 9. AWS 핵심요소 - VPC
- AWS VPC (Virtual Private Cloud)
  - AWS 클라우드 내에서 격리된 가상 네트워크를 생성(provisioning)하는 서비스
  - 사용자는 자신의 네트워크 환경처럼 IP 주소 범위, 서브넷, 라우팅 테이블, 게이트웨이 등을 직접 설계 및 제어 가능
  - 즉, AWS 상에서 인프라를 구성하기 위한 네트워크의 기본 단위
- VPC 주요 개념
  - CIDR 블록 (IP 주소의 범위(대역) 를 표기하는 방식. 형식 : IP주소/마스크비트)
    - VPC의 전체 IP 주소 범위를 정의
    - 예: 10.0.0.0/16 → 10.0.0.0 ~ 10.0.255.255 범위 사용 가능
    - 사설 IP 대역을 사용하여 내부 네트워크 구성
  - 서브넷(Subnet)
    - VPC 내 IP 범위를 세분화한 네트워크 단위
    - 예: 10.0.1.0/24 → 10.0.1.0 ~ 10.0.1.255
    - 일반적으로 가용 영역(AZ) 단위로 생성
    - EC2, RDS 등의 리소스는 특정 서브넷에 생성되며 해당 범위 내 IP를 할당받음
    - 퍼블릭 서브넷 / 프라이빗 서브넷으로 구분 가능
      - 퍼블릭 서브넷: 인터넷 게이트웨이(IGW)를 통해 외부 통신 가능
      - 프라이빗 서브넷: 외부에서 직접 접근 불가, 내부 서비스용
  - 인터넷 게이트웨이(IGW)
    - VPC와 인터넷을 연결하는 관문 역할
    - 퍼블릭 서브넷과 연결되어 외부 통신 가능하게 함
  - NAT 게이트웨이
    - 프라이빗 서브넷의 인스턴스가 외부 인터넷으로 나갈 수 있도록 지원
    - 외부에서 직접 접근은 불가능하도록 유지
  - 라우팅 테이블(Route Table)
    - 네트워크 트래픽의 이동 경로를 정의
    - 각 서브넷은 하나의 라우팅 테이블과 연결

### 9. AWS 핵심요소 - RDS
- Amazon RDS (Relational Database Service)
  - 관계형 데이터베이스를 쉽게 설정·운영·확장할 수 있는 AWS 관리형 서비스
  - 인프라 관리(패치, 백업, 장애 복구 등)를 AWS가 대신 수행
  - 지원 엔진: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server 등
- RDS 주요 특징
  - 자동 백업 및 스냅샷
    - 지정된 보존 기간 동안 자동 백업 수행
    - 수동 스냅샷 생성 가능
  - Multi-AZ 배포
    - 기본 DB 인스턴스와 대기(Standby) 인스턴스를 다른 AZ에 구성
    - 장애 발생 시 자동 Failover 지원
    - 고가용성(HA) 보장
  - 읽기 전용 복제본(Read Replica)
    - 읽기 트래픽 분산을 위한 복제 인스턴스
    - 대규모 조회 성능 향상에 활용
  - 자동 패치 및 유지 관리
    - OS 및 DB 엔진 패치를 자동으로 적용
    - 유지 관리 윈도우 설정 가능
- 리전(Region)과 가용 영역(AZ)
  - 리전(Region)
    - 물리적으로 분리된 AWS 인프라의 지리적 집합
    - 전 세계 여러 지역에 분산
  - 가용 영역(Availability Zone, AZ)
    - 하나의 리전 내에 존재하는 독립적인 데이터 센터 그룹
    - 고가용성을 위해 서로 물리적으로 분리되어 있음

### 10. AWS 핵심요소 - IAM 등
- IAM (Identity and Access Management)
  - AWS 리소스에 대한 인증(Authentication)과 권한 부여(Authorization)를 관리하는 서비스
  - "누가(Who) 무엇을(What) 어디까지(Which Resource) 할 수 있는지"를 정의하는 보안의 핵심 요소
- IAM 주요 구성 요소
  - 사용자(User)
    - AWS 서비스에 접근하는 실체(사람 또는 애플리케이션)
    - 루트 사용자(Root User)
      - AWS 계정 생성 시 자동 생성
      - 이메일/비밀번호로 로그인
      - 모든 권한을 가지므로 일상적인 사용은 지양
    - IAM 사용자(IAM User)
      - 개별 계정 ID(12자리), 사용자 이름, 비밀번호로 로그인
      - 필요한 권한만 부여하여 최소 권한 원칙(Least Privilege) 적용
  - 액세스 키(Access Key & Secret Access Key)
    - AWS CLI, SDK, API 호출 등 프로그래매틱 접근을 위한 인증 정보
    - 애플리케이션 또는 스크립트에서 AWS 서비스 호출 시 사용
    - 보안상 루트 사용자의 액세스 키 발급은 권장하지 않음
  - 정책(Policy)
    - JSON 형식의 권한 정의 문서
    - 특정 사용자/그룹/역할이 수행할 수 있는 작업(Action)과 대상 리소스(Resource)를 명시
    - 예: S3 버킷 읽기 허용, EC2 생성 권한 부여 등
    - AWS 관리형 정책 / 사용자 정의 정책으로 구분
  - 역할(Role)
    - 특정 서비스 또는 사용자에게 임시 권한을 부여하는 개념
    - 장기 자격 증명을 저장하지 않고도 안전하게 권한 위임 가능
    - 예: EC2 인스턴스가 S3에 접근하도록 IAM Role 부여
    - EKS, Lambda 등 서비스 연동 시 필수적으로 사용
- IAM 보안 관점 정리
  - 최소 권한 원칙 적용
  - 루트 계정은 MFA(다중 인증) 설정 필수
  - 액세스 키는 주기적 교체 및 노출 방지
  - 역할(Role) 기반 접근 제어 권장

### 10. AWS 핵심요소 - IAM 등 (기타 주요 서비스)
- EKS (Elastic Kubernetes Service)
  - Kubernetes를 AWS에서 관리형으로 제공하는 서비스
  - Control Plane을 AWS가 관리하여 운영 부담 감소
  - IAM과 연동하여 클러스터 접근 제어 가능
- ElastiCache
  - Redis/Memcached를 관리형으로 제공하는 인메모리 캐시 서비스
  - 다중 AZ 복제 및 자동 장애 조치(Failover) 지원
  - 세션 저장, 캐시 처리, 대기 시간 단축에 활용
- MSK (Managed Streaming for Kafka)
  - Apache Kafka를 AWS에서 관리형으로 제공
  - 다중 AZ 배포를 통한 고가용성 보장
  - 대규모 이벤트 스트리밍 및 실시간 데이터 파이프라인 구축에 활용
- S3 (Simple Storage Service)
  - 객체 스토리지 서비스
  - 정적 파일 호스팅, 백업, 로그 저장, 아카이빙 등에 사용
  - 높은 내구성(99.999999999%) 제공
  - 버전 관리 및 수명 주기 정책 설정 가능
- Amazon MQ
  - RabbitMQ, ActiveMQ를 관리형으로 제공하는 메시지 브로커 서비스
  - 레거시 시스템과의 메시징 통합에 활용
- Amazon OpenSearch Service
  - OpenSearch 기반의 관리형 검색/로그 분석 서비스
  - 대규모 데이터 인덱싱 및 검색 처리
  - 클러스터 자동 확장 및 고가용성 지원
- CloudWatch
  - AWS 리소스 및 애플리케이션 모니터링 서비스
  - 로그 수집, 지표(Metric) 분석, 알람 설정 가능
  - 오토 스케일링, 장애 감지, 운영 모니터링에 필수적인 서비스

### 12. 도메인 구매 및 route53설정
- 가비아 도메인
  - 도메인 검색
  - 네임서버 설정
    - 가비아 네임서버 사용
  - 결제
  - 서비스 관리 > 도메인이 조회되면 등록이 완료된 것
- Route53
  - 생성
    - Route 53
    - 시작하기
    - 호스팅 영역 생성
    - 도메인 이름
      - 가비아에서 구매한 도메인 주소
    - 유형
      - 퍼블릭 호스팅 영역
    - 호스팅 영역 생성
  - 가비아 <> Route53 연결
    - 가비아
      - 서비스 관리 > 도메인 > 관리 (도메인 상세 페이지로 이동됨)
      - 도메인 상세 페이지 > 네임서버 > 설정
      - 1차 ~ 4차 호스트명에 route53의 레코드 > 값/트래픽 라우팅 대상 등록 (호스트 마지막에 .로 끝나는데 .은 제외하고 등록)
        - ex.
          - ns-1681.awsdns-18.co.uk
          - ns-723.awsdns-26.net
          - ns-456.awsdns-57.com
          - ns-1057.awsdns-04.org
- 설정확인
  - https://www.whatsmydns.net/
  - 구매한 도메인으로 검색했을 때, O 표시로 나온다면 정상적으로 등록완료 (시간이 다소 걸림)

### 13. 클러스터 생성 및 aws cli, kubectl 세팅
- iam 설정 (10. AWS핵심요소-iam등 깅의 참고)
  - 액세스 관리 > 사용자 > 사용자생성
  - 'AWS Management Console에 대한 사용자 액세스 권한 제공 – 선택 사항' 체크
  - 콘솔암호 > 자동 생성된 암호
  - '사용자는 다음 로그인 시 새 암호를 생성해야 합니다 - 권장' 체크 해제
  - 권한 설정
    - 권한옵션 > 직접 정책 연결 > AdministratorAccess 체크
  - 사용자 생성 (생성된 정보는 따로 기입해둘 것 - 로그인 시 사용됨)
  - 이후 로그인 시에는
    - 생성 후 만들어진 url로 접근 후, user id / password 로 로그인
  - 위 정보로 로그인 후 > IAM > 사용자 > 사용자 선택 > 액세스 키 만들기
  - Command Line Interface(CLI) 선택
  - 태그 생략
  - 액세스 키도 따로 기입해둘 것
- eks 생성 (과금을 막기 위해서 최소한의 사양으로 설정)
  - Amazon Elastic Kubernetes Service > 클러스터 생성 (리전이 서울인지 확인)
  - 사용자 지정 구성 
  - 'EKS 자율 모드 - 신규' 해제
  - 클러스터 IAM 역할 > 권한 역할 생성 > 추천해준 역할로 생성
  - 나머지 옵션은 default로 설정하고 생성완료
- eks 설정
  - 노드 생성
    - Amazon Elastic Kubernetes Service > 클러스터 > 클러스터명 클릭 > 컴퓨팅 탭 > 노드 그룹 추가
    - 노드 IAM 역할 > 권장 역할 생성 > (아래의 절차를 통해 역할 생성 후) 만들어진 역할 선택
      - 신뢰할 수 있는 엔터티 유형 > 'AWS 서비스' 선택
      - 사용 사례 > 'EC2' 콤보박스 선택 및 라디오박스 선택
      - 자동으로 선택되어있는 역할 선택 > 역할 이름 입력 후 생성
    - 노드 그룹 컴퓨팅 구성
      - 인스턴스 유형만 변경
    - 나머지 옵션은 default로 설정하고 생성완료
- aws cli 설치
  - 설치
    - https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/getting-started-install.html
    - 설치 후 > 터미널에서 명령어로 설치여부 확인 가능
      - aws --version
  - 설정
    - aws configure
      - iam 설정에서 만든 access key 와 password 입력 
      - 리전 : ap-northeast-2
      - output format : json
- kubectl 설치
  - 설치
    - brew install kubectl
  - 설정
    - aws eks update-kubeconfig --region ap-northeast-2 --name <eks 클러스터명>
      - ex. aws eks update-kubeconfig --region ap-northeast-2 --name my-cluster
  - 노드검색 (설치 및 설정 확인용)
    - kubectl get nodes
      - 앞서 설치한 노드 2개가 조회되면 설치 및 설정완료

### 14. namespace
- namespace
  - 네임스페이스란 클러스터 내의 리소스를 분리된 그룹으로 나누는 단위
- 구조
  - ![namespace.png](image/namespace.png)
- 실습
  - 네임스페이스 생성
    - kubectl create namespace <namespace 이름>
    - ex. kubectl create namespace study
  - 전체 네임스페이스 목록 조회
      - kubectl get namespaces
  - 네임스페이스 내의 pod, service 등 자원 조회
      - kubectl get pods -n <namespace 이름>
        - kubectl get pods -n study
      - kubectl get services -n <namespace 이름>
        - kubectl get services -n study

### 15. Pod
- Pod
  - Pod는 쿠버네티스에서 배포할 수 있는 가장 작은 단위이며, 1개 이상의 컨테이너로 구성된 배포 단위
- 구조
  - ![pod.png](image/pod.png)
- 실습
  - 생성
    - 생성은 명령어를 통해 생성하거나 스크립트를 통해 생성할 수 있다. (여기서는 스크립트를 이용)
    - 1.k8s_basic > 1.pod_basic 로 이동
    - kubectl apply -f nginx_pod.yml
  - 조회
    - kubectl get pods -n <namespace 이름>
      - kubectl get pods -n study
  - 삭제
    - kubectl delete pod <pod명>
      - kubectl delete pod my-nginx
    - 또는 kubectl delete -f <파일명>
      - kubectl delete -f nginx_pod.yml
  - 생성 확인
    - pod 접속 (pod는 외부와 단절된 상태으므로 외부에서 nginx 호출이 불가하여 직접 접속하여 확인이 필요함)
      - kubectl exec -it <pod 이름> -n <namespace 이름> -- /bin/sh
        - kubectl exec -it my-nginx -n study -- /bin/sh
    - curl을 통한 nginx 응답확인 (pod 접속 후 실행)
      - curl http://localhost

### 16. Service
- Service
  - Service는 클러스터 내에서 실행 중인 Pod들에 대한 네트워크 접근 방법을 제공
  - pod는 동적으로 생성되고 주소 변경 가능성이 높고, 라우팅의 어려움이 있어서 Service를 이용하여 이러한 단점을 보완
- 구조
  - ![service.png](image/service.png)
- 스크립트
  - ```
    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
      namespace: my-namespace
    spec:
      ports:
        - port: 80
          targetPort: 80
      selector:
        app: nginx
    ```
  - selector
    - `selector.app: nginx` 는 `app=nginx` 라는 label을 가진 Pod들을 대상으로 지정한다.
    - 해당 label을 가진 Pod들에게 Service 네트워크가 연결된다.
    - 여러 개의 Pod가 selector와 매칭될 경우, Service가 자동으로 로드밸런싱을 수행한다.
  - port
    - Service가 클러스터 내부에서 리슨(listen)하는 포트이다.
    - 다른 Pod 또는 클러스터 내부 컴포넌트가 Service에 접근할 때 사용하는 포트이다.
  - targetPort
    - Service가 실제로 트래픽을 전달하는 대상 Pod의 포트이다.
    - Service로 들어온 요청을 Pod 내부 컨테이너의 해당 포트로 포워딩한다.
- 실습
  - 생성
    - 1.k8s_basic > 1.pod_basic 로 이동
    - kubectl apply -f nginx_service.yml
  - 생성 확인
    - service를 통해 pod로 요청이 들어오는지 확인을 위하여 (pod 접속 > 서비스를 통한 pod 호출이 어색한 흐름이지만, 현재 pod 말고는 접근할 방법이 없기 때문에 이렇게 간접 확인)  
      - pod 접속
        - kubectl exec -it my-nginx -n study -- /bin/sh
      - service 호출
        - curl http://<service-name>:<port>
        - ex) curl http://my-service:80

### 17. Replicaset
- ReplicaSet
  - ReplicaSet은 지정된 수의 Pod 복제본이 항상 실행되도록 보장하는 쿠버네티스 리소스이다.
  - 사용자가 정의한 `replicas` 개수만큼 Pod를 유지하며, 장애로 인해 Pod가 종료되면 자동으로 새로운 Pod를 생성한다.
  - `selector`를 기준으로 자신이 관리할 Pod를 식별한다.
  - 주로 단독으로 사용하기보다는 Deployment에 의해 관리되는 하위 리소스로 사용된다.
  - 즉, Pod의 개수 유지(가용성 확보)에 초점을 둔 리소스이다.
- 스크립트
  - ```
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: nginx-replicaset
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
            - name: nginx
              image: nginx
              ports:
                - containerPort: 80
    ```
  - replicas
    - `replicas: 2` 는 동일한 Pod를 2개 항상 유지하겠다는 의미이다.
    - Pod 중 하나에 장애가 발생하면 ReplicaSet이 자동으로 새로운 Pod를 생성한다.
  - selector.matchLabels
    - ReplicaSet이 관리해야 할 Pod를 식별하는 기준이다.
    - `app: nginx` 라벨을 가진 Pod만 관리 대상이 된다.
    - 이미 존재하는 Pod라도 해당 라벨이 일치하면 관리 대상에 포함된다.
  - template
    - 새로운 Pod를 생성할 때 사용하는 템플릿이다.
    - ReplicaSet은 이 템플릿을 기반으로 Pod를 생성한다.
  - template.metadata.labels
    - 새로 생성되는 Pod에 부여할 라벨을 정의한다.
    - 위 예제에서는 모든 생성 Pod에 `app: nginx` 라벨이 자동으로 추가된다.
    - 이 라벨은 반드시 `selector.matchLabels`와 일치해야 한다.
      - 일치하지 않으면 ReplicaSet이 해당 Pod를 관리하지 못한다.
    - 또한 Service에서 사용하는 `selector`와도 동일해야 네트워크 연결 및 로드밸런싱이 정상 동작한다.
- 실습
  - 생성
    - 1.k8s_basic > 2.multi_pod 로 이동
    - kubectl apply -f nginx_replicaset.yml
  - 확인
    - 2개의 pod 중 1개를 삭제하면, 1개가 자동으로 설정되는지 확인
    - 현재 pod 확인
      - kubectl get pods -n study
    - pod 1개 삭제
      - kubectl delete pod nginx-replicaset-x994g -n study
    - 다시 pod를 조회(kubectl get pods -n study) 하면, 1개의 pod가 이름이 바뀐 것을 확인할 수 있음

### 18. Deployment
- Deployment
  - Deployment는 ReplicaSet을 기반으로 동작하는 상위 리소스이다.
  - Deployment를 생성하면 내부적으로 ReplicaSet을 생성하고, 해당 ReplicaSet이 실제 Pod를 관리한다.
  - 즉, Pod 직접 관리가 아닌 ReplicaSet을 관리하는 리소스이다.
  - 특징
    - 버전 관리(Revision)
      - spec.template 값이 변경되면 새로운 ReplicaSet이 생성된다.
      - 각 변경 이력은 Revision으로 관리된다.
    - 롤링 업데이트(Rolling Update)
      - 새로운 Pod를 먼저 생성한 뒤, 기존 Pod를 순차적으로 종료한다.
      - 서비스 중단 없이 무중단 배포가 가능하다.
      - 기본 배포 전략은 Rolling Update이다.
    - 롤백(Rollback)
      - 문제가 발생하면 이전 Revision으로 손쉽게 되돌릴 수 있다.
    - 확장성(Scaling)
      - replicas 값을 변경하여 Pod 개수를 손쉽게 확장/축소할 수 있다.
  - 정리
    - ReplicaSet이 “Pod 개수 유지”에 초점을 둔다면,
    - Deployment는 “배포 전략 + 버전 관리 + 운영 자동화”까지 포함하는 상위 개념이다.
- 실습
  - 생성
    - 1.k8s_basic > 2.multi_pod 로 이동
    - kubectl apply -f nginx_deployment.yml
  - 롤백
    - kubectl rollout undo deployment <deployment명>
  - 새로운 ReplicaSet 및 Revision 생성 방법
    - 방법 1) image 변경 후 apply
      - `nginx_deployment.yml` 파일에서 `spec.template.image` 값을 변경한 뒤 다시 apply
      - 예:
        - `nginx:1.21.6`
        - `nginx:1.22.1`
        - `nginx:1.24.0`
      - `spec.template` 값이 변경되면 Kubernetes는 이를 새로운 버전으로 인식
      - 새로운 ReplicaSet과 새로운 Revision이 생성된다.
      - 이후 롤링 업데이트 방식으로 Pod가 교체된다.
    - 방법 2) rollout restart 사용 (실무에서 자주 사용)
      - 명령어
        - `kubectl rollout restart deployment <deployment명>`
      - 동작 원리
        - Deployment의 `spec.template`에 내부적으로 변경을 발생시켜 Kubernetes가 "새로운 버전"으로 인식하게 만든다.
        - 그 결과 새로운 ReplicaSet이 생성되고, 기존 Pod를 순차적으로 교체한다.
      - 왜 필요한가?
        - 실무에서는 보통 `latest` 태그를 사용하는 경우가 많다.
        - 예:
          - `image: myapp:latest`
        - 새로운 이미지를 빌드해도 image 이름은 동일하다.
        - 즉, YAML 상의 변경사항이 없기 때문에 Kubernetes는 "변경 없음"으로 판단하여 새로운 ReplicaSet을 생성하지 않는다.
      - 문제점
        - 실제로는 이미지 내용이 바뀌었지만, Kubernetes는 이를 감지하지 못해 기존 Pod가 계속 실행된다.
      - 해결 방법
        - `rollout restart`를 사용하면 강제로 새로운 ReplicaSet을 생성하고 Pod를 재생성하여 최신 이미지를 반영할 수 있다.

### 19. Ingress - 이론 ~ 20. Ingress - 실습
- Ingress
  - 클러스터 외부의 HTTP/HTTPS 요청을 내부 Service로 라우팅하는 리소스이다.
  - 도메인(Host) 또는 경로(Path) 기반으로 트래픽 전달 규칙을 정의한다.
  - 단독으로는 동작하지 않으며 Ingress Controller가 필요하다.
- Ingress Controller
  - Ingress에 정의된 규칙을 실제로 처리하는 구성 요소이다.
  - 클러스터 내부에서 동작하며 외부 트래픽을 받아 Service로 전달한다.
  - 예: NGINX Ingress Controller, AWS Load Balancer Controller
- 구조
  - ![ingress.png](image/ingress.png)
- 실습
  - Ingress Controller 설치
    - kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/aws/deploy.yaml
  - 로드 밸런서 및 route 53 설정
    - EC2 > 로드 밸런서 > 활성 상태인지 확인
    - route 53 > 호스팅 영역 > 호스팅 선택 > 레코드 생성
      - 레코드 이름 입력 (ex. server)
      - 레코드 유형 > 'A – IPv4 주소 및 일부 AWS 리소스로 트래픽 라우팅' 선택
      - 별칭 활성화
        - 트래픽 라우팅 대상 : 'Network Load Balancer에 대한 별칭'
        - 리전 : 서울
        - 위 2개를 선택했다면, 콤보박스에 선택할 수 있는 로드밸런서 확인가능
          - 이때 선택 가능한 로드밸런서는 EC2에서 활성 상태인 로드밸런서의 ID와 일치하는지 확인필요
  - ingress 적용
    - 1.k8s_basic > 2.multi_pod 로 이동
    - kubectl apply -f ingress_nginx.yml
  - 브라우저에 'https://server.choi1992.shop/study/' 접속
    - nginx의 welcome 페이지가 나온다면 정상

### 21. 아키텍처 개요
- 구조
  - ![spring_architecture.png](image/spring_architecture.png)

### 22. 인프라 자원 생성
- RDS 생성
  - Aurora and RDS > 데이터베이스 > 데이터베이스 생성
    - 손쉬운 생성
    - MySQL
    - 샌드박스 (= 프리티어)
    - 이름 : admin
    - 자격 증명 관리 : 자체관리
    - 암호 자동 생성 : 체크
    - 생성 중 화면에서 상단 모달에 '연결 세부 정보 보기' 클릭하여 비밀번호 따로 기입
  - 추가설정
    - 퍼블릭 액세스
      - 생성된 데이터베이스명 클릭
      - 추가구성 : '퍼블릭 액세스 가능' 선택 후 수정 > 즉시적용
    - 인바운드 규칙 (실습할 때는 기본적으로 설정되었기에 진행하지 않음)
      - 생성된 데이터베이스명 클릭
      - 보안그룹
      - TCP / 3306 포트 추가
  - 스키마 생성
    - create database ordersystem;
- ECR 생성
  - Amazon ECR > 프라이빗 레지스트리 > 리포지토리 > 프라이빗 리포지토리 생성
    - 리포지토리 이름 : (강의에서는) order-backend
    - 나머지 설정은 default로 생성
- Redis 설정
  - 2.ordersystem > k8s > k8s-ordersystem 로 이동
  - kubectl apply -f redis.yml

### 23. 이미지 빌드/push
- 스프링 설정
  - profile 변경
    - application.yml의 profiles를 local에서 prod로 변경
  - DB 등 접속정보
    - application.yml에 접속정보를 그대로 적으면 보안상 위험할 수 있기 때문에 k8s의 secret을 사용
- 로컬에서 ECR 로그인
  - aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 874777259027.dkr.ecr.ap-northeast-2.amazonaws.com/order-backend
- 로컬에서 도커 이미지 빌드 및 push
  - 빌드
    - 2.ordersystem의 dockerfile 위치로 이동
    - docker build -t 874777259027.dkr.ecr.ap-northeast-2.amazonaws.com/order-backend:latest .
  - push
    - docker push 874777259027.dkr.ecr.ap-northeast-2.amazonaws.com/order-backend:latest
  - 확인
    - aws > ECR > 이미지에서 확인 가능
  - 추가 수정사항
    - 위에 빌드 & push로 했을 때, '24. 쿠버네티스 자원 생성 > Deployment 적용' 단계에서 오류 발생.
    - 원인을 파악하다보니 아키텍처 mismatch 문제. 따라서 아래 스크립트로 실행해야함.
    - docker buildx build --platform linux/amd64 -t 874777259027.dkr.ecr.ap-northeast-2.amazonaws.com/order-backend:latest --push .

### 24. 쿠버네티스 자원 생성
- Secret
  - 생성
    - kubectl create secret generic my-app-secrets --from-literal=DB_HOST=<생성된 도메인 주소> --from-literal=DB_PW=<발급 받은 비밀번호> -n study
  - 조회
    - kubectl get secret my-app-secrets -o yaml -n study
      - 조회된 값은 Base64로 인코딩된 값 (디코딩하면 실제 값을 알 수 있다.) 
  - 삭제
    - kubectl delete secret my-app-secrets -n study
  - 수정
    - Secret은 수정하는 명령어를 제공하지 않으므로, 삭제 후 생성해야한다.
- Deployment 적용
  - 적용
    - 2.ordersystem > k8s > k8s-ordersystem 로 이동
    - kubectl apply -f depl_svc.yml
  - 확인
    - kubectl get pods -n study
      - 모두 running 상태인지 확인. 만약 그렇지 않다면,
        - kubectl logs -f ordersystem-backend-67d987d857-4m8v6 -n study
        - 또는 kubectl describe pod ordersystem-backend-67d987d857-4m8v6 -n study
    - 모두 Running 상태가 되었고, DB의 member 테이블에 admin 계정이 생성되었다면 정상
- ingress
  - ingress 컨트롤러 적용
    - kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/aws/deploy.yaml
  - ingress 적용
    - kubectl apply -f ingress.yml
  - 확인
    - 브라우저에서 https://server.choi1992.shop/health 접속

### 25. https통신을 위한 인증서 작업
- 정리가 꼭 필요한 항목이 아니라 제외

### 26. github actions와 CI/CD 자동화
- GitHub Actions 개념
  - GitHub Actions 이란
    - 저장소에서 직접 CI/CD 파이프라인을 구성하는 기능
    - 이벤트 기반으로 개발 워크플로우를 자동화
  - 기본 구성 요소
    - 워크플로우 (Workflows)
      - 하나 이상의 Job을 포함하는 자동화 프로세스
      - 특정 이벤트 발생 시 실행
      - .github/workflows 경로에 YAML 파일로 정의
    - 이벤트 (Events)
      - 워크플로우 실행을 트리거하는 활동
      - 예: push, pull_request, issue 생성 등
    - 작업 (Jobs)
      - 워크플로우 내 실행 단위
      - 여러 Step으로 구성
    - 러너 (Runners) - runs-on
      - 워크플로우가 실행되는 가상 서버
      - Linux, Windows, macOS 환경 제공
    - 액션 (Actions) - uses, run
      - 재사용 가능한 작업 단위
      - 공식 액션 및 커뮤니티 액션 사용 가능
- 실습
  - .github > workflows > yml 파일생성
  - Secret 생성
    - 깃허브 > Repo > Settings > Secrets and variables > Actions > Repository secrets
      - AWS_KEY 생성 (aws 생성할 때 만든 key)
      - AWS_SECRET 생성 (aws 생성할 때 만든 secret)
  - HealthController.java의 응답값 변경 후 commit / push
    - 브라우저에서 https://server.choi1992.shop/health 접속 (변경사항이 자동 반영되었다면, 응답값이 변경된 것을 확인할 수 있다.)

### 27. 오토스케일 - pod자동확장
- 개념
  - Kubernetes에서 Deployment의 자동 확장(autoscaling)을 설정하려면 Horizontal Pod Autoscaler (HPA)를 사용
  - HPA는 CPU 사용량이나 사용자 정의 파라미터에 기반하여 파드의 수를 자동으로 조정
- 실습
  - hpa
    - 적용
      - 2.ordersystem > k8s > k8s-ordersystem 로 이동
      - kubectl apply -f hpa.yml
    - 메트릭 서버와 HPA를 통한 pod 현황 조회
      - kubectl get hpa ordersystem-backend-hpa -n study -w
  - 부하
    - 특정 pod로 접속 (새로운 터미널에서 진행)
      - ex. kubectl exec -it ordersystem-backend-745b796bf7-b264k -n study -- /bin/sh
    - 부하 시작 (접속한 Pod에서 아래 명령어 입력)
      - while true; do curl -s http://ordersystem-backend-service/product/list; done
    - '메트릭 서버와 HPA를 통한 pod 현황 조회' 에서 pod가 늘고있는 것을 확인할 수 있음. 또한 pod를 조회해도 여러개의 pod가 생성된 것을 확인할 수 있음

### 28 ~ 30. 오토스케일 - 인스턴스(EC2)
- 정리가 꼭 필요한 항목이 아니라 제외