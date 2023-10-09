# 프로젝트명

**프로그래밍 과제 제출 환경 통합 및 검사 자동화 플랫폼: MoaRoom(모아룸)**

**링크**: http://moaroom-front.duckdns.org:3000

# 팀원
### [금나연](https://github.com/NayeonKeum): Infra & Frontend
### [김민지](https://github.com/mjms0214): Backend & Frontend

# 로고
<img src="https://github.com/MoaRoom/.github/assets/68985625/283e32c3-c089-4a15-9d3a-39c7133f99f8" width=30%/>&nbsp;&nbsp;<img src="https://github.com/MoaRoom/.github/assets/68985625/1ddba96d-a9d2-49b6-b066-79ff941855e3" width=30%/>

# 협력기관(국가)

㈜그렙([programmers.co.kr](http://programmers.co.kr/))(대한민국)

# 가. 목표 및 기대효과

## **목표**

- Kubernetes를 이용하여 제출자 별로 독립적인 컨테이너(환경)를 할당하고, 채점자는 이러한 리소스들을 손쉽게 관리하고 제출자의 구현물을 열람할 수 있는 플랫폼을 개발하고자 함
- 채점자로 하여금 제출자의 수에 구애받지 않고, 시간과 노력을 절감할 수 있는 자동 채점 기능을 구현하고자 함

## 기대효과

- 가상 머신보다 가볍지만 마찬가지로 독립성을 보장하는 컨테이너를 쉽고 빠르게 배포/확장하고 관리를 자동화해주는 오픈소스 Kubernetes를 이용하여 제출자 별로 상호 독립적인 환경을 조성할 수 있음.
- 제출자로 하여금 별다른 가상 환경이나 통합 개발 환경 없이 웹 접속 만으로 과제를 수행할 수 있음.
- 프로그래밍에 대한 관심과 더불어 증가한 채점자 수 대 제출자 수의 비율에 자동 채점 기능을 이용하여 관리의 용이성을 향상시킬 수 있음.

# 나. 과제 내용

## 과제내용

### **주요 기능**

- 채점자(교수/조교)
    - 대시보드를 통해 제출 인원에 맞는 가상 환경 관리
    - 제출된 과제를 로컬 PC에 다운로드, 실행하지 않고도 열람 가능
    - 자동화된 채점 기능
- 제출자(학생)
    - 채점자의 기준에 맞는 개발 환경에서 과제 수행, 제출 가능

## 추진내용

### 서비스 개발

- [[MoaRoom-Front](https://github.com/MoaRoom/MoaRoom-Front)] 프론트엔드
    - **사용 기술:** React.ts 18.2.0, TypeScript 4.0, Ubuntu 20.04(AWS EC2)
    - **개발 환경:** Visual Studio Code 1.68
    - **구현**
        - 교수: 로그인 → 강의 리스트 확인 → 채점 대시보드, 컨테이너 할당 대시보드
        - 학생: 로그인 → 강의 리스트 확인 → 과제 리스트, 컨테이너 할당 요청 리스트
        - 공통: 마이페이지(수강 중/개설한 강의 목록 확인 가능, 사용자 정보 확인 및 수정)
- [[MoaRoom-Back](https://github.com/MoaRoom/MoaRoom-Back)] 백엔드
    - **사용 기술:** Spring Boot 2.7.10, JAVA 11.0.18, PostgreSQL 15.2(AWS RDS), Ubuntu 20.04(AWS EC2)
    - **개발 환경:** IntelliJ 2022.3
    - **구현:**
        - Create: 회원 가입, 강의 생성, 과제 생성
        - Read: 사용자 정보 불러오기, 강의/과제 리스트 불러오기
        - Update: 사용자 정보 수정
        - Delete: 강의 삭제, 과제 삭제

### 인프라 개발

![졸프_SW아키텍처 drawio](https://github.com/MoaRoom/Moaroom-Provisioning-Infra/assets/68985625/69a5fac7-455a-4934-b219-127eaab97e88)

- [[Moaroom-Provisioning-Infra](https://github.com/MoaRoom/Moaroom-Provisioning-Infra)]Infrastructure provisioning pipeline
    
    - **사용 기술:** Kubernetes 1.25.8, Kubespray 2.21.0, Ansible 2.12, Docker 20.10.17, containerd 1.6.6, Python 3.8
    - **개발 환경:** MacOS Ventura 13.2.1
    - **구현:**
        - 개발자 로컬 환경에 있는 kubespray, Ansible configuration 파일 빌드
        - AWS EC2에 Kubernetes 환경 생성

- [[MoaRoom-Infra](https://github.com/MoaRoom/MoaRoom-Infra)] CI/CD pipeline
    
    - **사용 기술:** ArgoCD to v2.5.5, helm 3.10.3, Jenkins 2.388, Dockerhub, Github
    - **개발 환경:** MacOS Ventura 13.2.1
    - **구현:**
        - 개발자의 원격 브랜치에서 Docker 빌드에 필요한 파일 또는 Kubernetes configuration 관련 파일을 Github에 푸시
        - Docker 빌드 관련 파일 수정은 Webhook을 통해 Jenkins 서버로 전달, Kubernetes 관련 파일에 적용 및 Dockerhub에 image 생성 후 푸시
        - Kubernetes 관련된 파일에서 변경된 내용은 Argo를 이용하여 AWS EC2의 K8s cluster에 전달 후 변경 사항 적용

- Cloud architecture
    - **사용 기술:** EBS, EC2, EKS, ELB,  RDS, Route53, S3
    - **개발 환경:** Amazon Web Services
    - **구현**
        - Kubernetes cluster
            - **Service:** Public ELB를 통해 들어온 외부 트래픽을 Load Balancer 타입의 Router Service 및 Pod를 통해 Master(교수), Playground(학생, slave) Service에 전달
            - **Management:** CI/CD pipeline에 필요한 Jenkins 및 ArgoCD의 Service 및 Pod 등을 통해 필요 과정 수행
            - **Logging:** ELK Stack으로 구축한 Elasticsearch Cluster에서 Service, Management 관련 Pod들을 모니터링, 필요시 Kibana의 Service를 ELB에 연결하여 관리자에게 대시보드 제공
        - Service
            - **WEB Server(EC2, EBS):** 프론트엔드 단에서 Public ELB를 통해 들어온 외부 트래픽을 처리, 재사용 가능한 EBS 볼륨 사용
            - **WAS Server(EC2, EBS):** Private Subnet에서 WEB server에서 필요한 API 제공, 재사용 가능한 EBS 볼륨 사용
            - **DB Server(RDS-PostgreSQL):** Private Subnet에서 서비스 관련 정보 저장
            - **Storage Server(S3):** Private Subnet에서 서비스에 필요한 파일, 영상 등 미디어 자료 저장



### 🚨 MoaRoom는 숙명여자대학교 IT공학전공 23-1 졸업프로젝트 출품작입니다. 수업 기간(2023.2 ~ 2023.8) 동안 무단 도용, 배포 및 상업적 사용을 금지합니다. 🚨
