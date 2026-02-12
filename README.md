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