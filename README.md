# 강의
- Inflearn > eks를 활용한 spring 운영서버 배포 (feat. devops의 모든것)

## 개념정리  

### 도커  
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
  
## 프로젝트
### 제목
- 내용