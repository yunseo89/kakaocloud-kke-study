## 쿠버네티스 클러스터 구축 및 웹 서비스 생성 실습
참고자료: https://docs.kakaocloud.com/tutorial/container/k8s-engine-k8s-cluster

### 1. 시나리오 소개 
- 쿠버네티스 엔진을 사용해 카카오클라우드에서 쿠버네티스 클러스터를 생성하고, 클러스터에 웹 서비스를 배포한다.
- 예제 프로젝트의 컨테이너 이미지를 컨테이너 레지스트리에서 pull 하고, 클러스터 내에 구성된 서비스로 배포한 뒤 외부 접속을 위한 로드밸런서를 설정한다.

### 2. 시작하기 전에..
- 네트워크 환경 구축
- 예제 프로젝트 컨테이너 이미지 배포 가 필요하다.
--- 
## 네트워크 환경 구축
참고자료: https://docs.kakaocloud.com/tutorial/networking-content-delivery/private-subnet

### 1. 시나리오 소개
- 카카오클라우드 VPC 환경에서는 각 가용영역에 NAT 인스턴스를 별도로 배치할 수 있다. 특정 가용영역에 장애가 발생해도 다른 가용영역의 워크로드가 해당 영역의 NAT 인스턴스를 통해 안정적으로 외부와 통신할 수 있도록 보장한다.(프라이빗 서브넷 기준) 이를 통해 서비스 가용성을 높이고, 단일 지점 장애(SPOF)를 방지할 수 있다.
- 프라이빗 서브넷에 위치한 인스턴스가 인터넷과 안전하게 통신할 수 있도록 NAT 인스턴스를 구성해보자.

<img width="900" height="800" alt="image" src="https://github.com/user-attachments/assets/eefca99d-6578-4d0e-a364-db9d0a84d0cc" />

### step 1. VPC, 서브넷 생성
### step 2. NAT 인스턴스를 위한 보안그룹 생성
### step 3. NAT 인스턴스 생성 & step 4. NAT 인스턴스 설정
- 프라이빗 서브넷의 인스턴스가 인터넷을 트래픽을 보내는 것을 허용하지만, 인터넷부터 오는 직접적인 접근은 차단한다.
  인스턴스가 필요한 업데이트와 패치를 받을 수 있도록 한다.(보안그룹의 'stateful'한 특성 때문에 아웃바운드에 대한 응답인 인바운드가 허용)<br>
  -> 즉, 인스턴스 쪽에서 먼저 접근을 시도한 외부에 대해서만 아웃바운드+인바운드 허용 됨.
- 두개 인스턴스에 원격으로 접속해서 설정을 수행함
  
<img width="800" height="300" alt="image" src="https://github.com/user-attachments/assets/9c55d674-ec23-47fa-ae45-a1d76a790d05" />

-> 사용 가능한 네트워크 인터페이스를 자동으로 식별해서 선택하고, ip 포워딩을 활성화하고(ip_forward = 1), 선택된 인터페이스에 대한 네트워크 트래픽
마스러레이딩을 자동 구성한다 = 출발지가 인스턴스의 사설ip가 아닌, NAT의 퍼블릭 ip로 보임

### step 5. 패킷 송신 허용 ip 수정

- 인스턴스는 기본적으로 본인을 목적지로 하는 트래픽만을 수신한다. 그러나 NAT 인스턴스는 트래픽의 목적지가 자신이 아니어도 트래픽을 보내고 받을 수 있어야 한다. 따라서 0.0.0.0/0 을 목적지로 하는 패킷을 송신하도록 허용해준다.

### step 6. 라우팅 테이블 생성 및 설정

- 라우팅 테이블을 통해 서브넷 내부에서 특정 목적지를 향하는 요청에 대한 규칙을 설정할 수 있다. 프라이빗 서브넷에서 외부로 나가는 요청은 해당 서브넷이 속한 가용영역의 NAT로 라우팅 되게 한다.(nat -> 외부)

- vpc 토폴로지 탭에서 구성을 확인한다.
- 
<img width="700" height="750" alt="image" src="https://github.com/user-attachments/assets/d774ba58-4fea-4b83-a2d3-78521d9a1880" />

----
## Container Registry를 이용한 이미지 저장 및 사용
참고문서: https://docs.kakaocloud.com/tutorial/container/cr-basic

### 1. 시나리오 소개

- 카카오클라우드의 CR을 사용하여 컨테이너 이미지를 저장하고, 이를 활용한다.
- 로컬 환경에서 예제 프로젝트를 도커 이미지로 빌드하고, CR 업로드 후 이미지 등록 및 다운로드까지의 전 과정을 실습한다.

### 시작하기 전에...
- Homebrew, Git, Docker 설치가 필요

### step 1. container registry 생성
### step 2. 예제 프로젝트 도커 이미지 빌드
- 서버 프로젝트 설치
- 서버 프로젝트 빌드

#### 서버 프로젝트란?
> 백엔드

docker build -t kakaocloud-library-server:latest --platform linux/amd64 -f ./server/deploy/Dockerfile ./server 
명령어를 통해 도커파일을 이미지로 만듦. 도커가 리눅스 OS도 깔고, 필요한 프로그램을 설치함...

+ -t = Name tag의 약자, kakaocloud-library-server 라는 이미지명에 버전은 latest로 적는다
+ --platform linux/amd64 = Mac은 ARM cpu를 사용하고, 카카오클라우드 서버는 AMD64를 사용한다. 따라서 클라우드 서버에 맞춰서 포장시킴
+ -f = file의 약자, 도커파일의 위치를 지정함 // ./server 에 context(재료)가 있어 지정해줌.

docker tag kakaocloud-library-server:latest ${PROJECT_NAME}.kr-central-2.kcr.dev/${REPOSITORY_NAME}/kakaocloud-library-server:latest


= docker tag [기존이름] [새로운이름]의 구조, 이름은 그대로 주소만 추가!

+ kr-central-2.kcr.dev = kakao container registry의 약자, 도커 이미지 전용 창고 주소이다.
+ kakaocloud-library-server:latest = 목적지에 도착해서 보관될 최종 이미지의 이름과 버전(latest)

- 클라이언트 프로젝트 빌드
- 클라이언트 프로젝트 태그 설정
  
#### 클라이언트 프로젝트란?
> 프론트엔드
> -> 서버용, 클라이언트용 도커 이미지를 따로 만들어서 클라우드에 올리는게 정석이다(=MSA)

### step 3. 예제 프로젝트 이미지 업로드
- CR 로그인
- 이미지 업로드

### step 4. 결과 확인

#### 카클 콘솔 상에서 확인

<img width="3790" height="1346" alt="image" src="https://github.com/user-attachments/assets/53ef1ca6-bfd2-4b81-8f23-9f7b12933e12" />


#### 터미널에서 확인(docker images)

<img width="1708" height="284" alt="image" src="https://github.com/user-attachments/assets/9b2549dc-d61a-4869-b319-056bfb2bd5fc" />

















