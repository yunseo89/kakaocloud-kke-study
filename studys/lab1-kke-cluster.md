## 쿠버네티스 클러스터 구축 및 웹 서비스 생성 실습
참고자료: https://docs.kakaocloud.com/tutorial/container/k8s-engine-k8s-cluster

### 1. 시나리오 소개 
- 쿠버네티스 엔진을 사용해 카카오클라우드에서 쿠버네티스 클러스터를 생성하고, 클러스터에 웹 서비스를 배포한다.
- 예제 프로젝트의 컨테이너 이미지를 컨테이너 레지스트리에서 pull 하고, 클러스터 내에 구성된 서비스로 배포한 뒤 외부 접속을 위한 로드밸런서를 설정한다.

### 2. 시작하기 전에..
- 네트워크 환경 구축
- 예제 프로젝트 컨테이너 이미지 배포 가 필요하다.
--- 
## 0. 네트워크 환경 구축
참고자료: https://docs.kakaocloud.com/tutorial/networking-content-delivery/private-subnet

### 1. 시나리오 소개
- 카카오클라우드 VPC 환경에서는 각 가용영역에 NAT 인스턴스를 별도로 배치할 수 있다.
