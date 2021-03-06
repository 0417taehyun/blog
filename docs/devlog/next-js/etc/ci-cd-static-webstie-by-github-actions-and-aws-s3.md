---
title: "[ Next.js ] AWS, GitHub Actions를 활용한 정적 웹사이트 CI/CD"
description: "[ Next.js ] AWS, GitHub Actions를 활용한 정적 웹사이트 CI/CD"
tags: 
    - "2021"
    - NextJS
---

# [ Next.js ] AWS, GitHub Actions를 활용한 정적 웹사이트 CI/CD

## 도입

이 글을 통해 **AWS IAM**, **S3**, **CloudFront**, **Route53**, **Certificate Manager**을 활용해 Next.js 프로젝트를 정적 웹 사이트로 배포하고 **GitHub Actions**를 활용하여 해당 배포를 자동화하는 방법에 대해 배웁니다. 그리고 이 과정에서 **GitHub Actions** 및 배포 방식 중 하나인 **CI/CD**에 관한 기초적인 개념을 익힐 수 있습니다.  

!!! info "참고"

    이 글에 사용된 모든 소스 코드는 맨 아래 [참고](#참고) 부분에 있는 [GitHub 레포지토리](https://github.com/week-with-me/next-js-deployment-practice)에서 확인할 수 있습니다.


**배울 수 있는 것**과 **배울 수 없는 것**, 그리고 **유의사항**을 각각 정리하면 아래와 같습니다.  

해당 글을 통해 **배울 수 있는 것**은 아래와 같습니다.  

* **AWS IAM**을 통한 사용자 계정 관리
* **AWS S3**를 통한 정적 웹 사이트 배포
* **AWS CloudFront**, **Route53**, **Certificatie Manager**을 통한 구매한 도메인 및 HTTPS 리다이렉션 적용
* **GitHub Actions**를 활용한 배포 자동화(**CI/CD**)


해당 글을 통해 **배울 수 없는 것**은 아래와 같습니다.  

* **Git** 및 **GitHub** 기초
* **Next.js**를 포함한 **프론트엔드** 기초
* **AWS** 핵심 개념
* **WEB** 및 **HTTP** 핵심 개념

유의사항은 아래와 같습니다.  

* 도메인 구매 비용 및 AWS **과금이 발생**할 수 있습니다.
* 쉽게 따라할 수 있는 만큼 이미지가 많아 **글이 깁니다**.**
* 글이 작성된 이후 시간이 지나 AWS 또는 GitHub 서비스 UI 등의 **변화가 있을 수 있습니다**.


## 정적 웹 사이트 배포

### 프로젝트 생성

`npx create-next-app next-js-deployment-practice --typescript` 라는 명령어를 통해 타입스크립트(TypeScript)를 사용하는 새로운 Next.js 프로젝트를 생성합니다. 명령어를 통해 알 수 있듯 생성된 프로젝트의 이름은 `next-js-deployment-practice` 입니다.  

!!! info "참고"

    굳이 타입스크립트를 사용하는 프로젝트를 생성하지 않아도 괜찮고 심지어는 Next.js가 아닌 다른 프론트엔드 라이브러리 또는 프레임워크를 사용해도 좋습니다. 다만 이 과정에서 배포하게 되는 디렉토리의 이름이 달라질 수 있습니다.


### AWS IAM

#### 개념

서비스를 만들 때 보통 다수의 인원이 함께 개발에 착수합니다. 이후 배포를 할 때 다양한 클라우드 호스팅 서비스를 사용할 수 있는데, 이때 사용자를 구분하는 것이 보안적인 측면에서 좋습니다.  

쉽게 생각하면 프론트엔드 개발만 하는 사람에게 백엔드 관련 서버를 다운시킨다거나 혹은 다른 사람들의 계정을 지울 수 없게 권한을 제약하는 것입니다.  

바로 이러한 개별 또는 그룹의 권한 설정을 가능하게 해주는 AWS의 서비스가 바로 **IAM(Identity and Access Management)**입니다.  

#### IAM 계정 생성
AWS를 루트 계정으로 로그인 한 이후 아래 이미지와 같이 **IAM**을 검색하여 **사용자** 탭에 들어갑니다. 이후 **사용자 추가** 버튼을 누릅니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/1.png">

아래 이미지와 같이 본인이 원하는 이름으로 **사용자 이름**을 설정하고 **AWS 자격 증명 유형**의 경우 **액세스 키**와 **암호**를 모두 체크합니다.  

**액세스 키**의 경우 **GitHub Actions** 환경 또는 설명에 적혀있는 것처럼 터미널과 같은 CLI(Command Line Interface) 환경에서 AWS 작업을 하기 위해 필요합니다.  

**암호**의 경우 해당 사용자 (이 예시에서는 `weekwith.me`) 가 AWS 웹 페이지에 로그인할 때 암호를 사용할 수 있게 해줍니다. **콘솔 비밀번호**의 경우 무엇을 선택해도 괜찮지만 보안을 위해 **자동 생성된 비밀 번호**로 설정하였습니다. 처음 로그인할 때는 이를 통해 자동으로 생성된 비밀번호를 입력해서 접속해야 합니다. 이후 **비밀번호 재설정 필요 옵션**을 체크하였기 때문에 사용자 (이 예시에서는 `weekwith.me`) 가  첫 로그인 이후 직접 본인이 원하는 비밀번호로 수정할 수 있습니다. 이에 관해서는 아래에서 더 자세히 알 수 있습니다.

<img src="https://weekwith.me/images/next-js/ci-cd/2.png">

이제 해당 사용자가 어떤 AWS 서비스를 이용할 수 있는지 권한을 부여해줍니다. 우리의 경우 앞서 이야기했던 것처럼 **S3**, **Route 53**, **CloudFront**, **Certificate Manager**를 이용하기 때문에 아래 이미지와 같이 각각의 정책을 정책 필터에서 검색하여 체크해줍니다. 이때 각각의 서비스에 관해 FullAccess 권한이 아닌 더 제한적인 권한을 부여할 수 있지만 원활한 진행을 위해 `FullAccess`로 권한을 부여합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/3.png">

**태그**의 경우 선택 사항입니다. 추후 해당 사용자를 빠르게 검색하거나 인지할 수 있게 아래와 같이 `study`라는 키에 `next-js-deployment-practice`라는 값을 입력했습니다. 굳이 따라하지 않으셔도 좋습니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/4.png">

태그까지 마무리를 지었다면 아래 이미지와 같이 사용자가 생성된 것을 확인할 수 있습니다. 이때 권한 요약 부분에 존재해야 할 권한은 아래와 같습니다.  

* AmazonS3FullAccess
* AmazonRoute53FullAccess
* AmazonCloudFrontFullAccess
* IAMUserChangePassword
* AWSCertificateManagerFullAccess

!!! warning "주의"

    해당 이미지에는 ACM 관련 권한이 빠져있습니다.

<img src="https://weekwith.me/images/next-js/ci-cd/5.png">

그 뒤에 사용자 만들기 버튼을 누르고나면 아래 이미지와 같이 액세스 키와 관련된 CSV 파일을 다운로드 받을 수 있는 페이지가 나타납니다.  

!!! danger "위험"

    이때 꼭 해당 CSV 파일을 다운로드 받고 외부에 유출하거나 삭제해서는 안 됩니다.

    만약 해당 파일을 잃어버리거나, 파일이 유출되었을 경우 액세스 키를 삭제하고 재생성해야 합니다.

<img src="https://weekwith.me/images/next-js/ci-cd/6.png">

다운로드 받은 CSV 파일을 열어보면 아래 이미지와 같습니다. 해당 파일에서 기억해야 할 부분은 아래와 같습니다.  

* **Password**: IAM 계정으로 로그인할 때 필요한, 자동으로 생성된 비밀번호입니다.
* **Access key ID**: 추후 GitHub Actions를 활용하여 AWS 서비스에 접근 및 해당 서비스를 사용할 때 필요한 액세스 키입니다.
* **Secret access key**: 추후 GitHub Actions를 활용하여 AWS 서비스에 접근 및 해당 서비스를 사용할 때 필요한 시크릿 키입니다.
* **Console login link**: 해당 IAM 계정으로 로그인할 때 사용되는 URL입니다.

<img src="https://weekwith.me/images/next-js/ci-cd/7.png">

#### 액세스 키 재발급 또는 자동 생성된 비밀번호 재생성

만약 액세스 키를 잃어버리거나 해당 키가 외부에 유출되었을 때는 키를 비활성화하고 재생성해야 합니다. 또는 자동으로 생성했던 IAM 계정의 비밀번호를 잃어버려 다시 생성해야 하는 경우도 재생성해야 합니다. 이 과정이 필요 없다면 건너 뛰셔도 좋습니다.  

액세스 키 또는 **IAM** 계정 비밀번호를 재생성하기 위해서는 루트 사용자로 로그인한 뒤 아래 이미지와 같이 IAM으로 가서 생성했던 사용자를 클릭합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/8.png">

이제 아래 이미지와 같이 **보안 자격 증명** 탭으로 이동합니다. 만약 비밀번호를 재생성해야 하는 경우 **콘솔 비밀번호**를 활용하고, 액세스 키를 재생성해야 하는 경우 기존에 존재하던 액세스 키를 **비활성화**한 뒤 **액세스 키 만들기**를 통해 새로 생성합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/9.png">

기존 액세스 키를 성공적으로 비활성화하고 새로 생성할 경우 아래 이미지와 같습니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/10.png">

재생성된 CSV 파일을 확인하면 아래 이미지와 같습니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/11.png">

#### IAM 계정 로그인

IAM 계정으로 로그인하기 위해서는 아래 이미지와 같이 **계정 ID**, **계정 별칭** 또는 **해당 계정으로 로그인할 수 있는 URL**을 사용해야 합니다. 이번에는 계정 별칭을 한 번 사용해보겠습니다. 간단하게 본인이 원하는 별칭으로 생성하면 됩니다. 

<img src="https://weekwith.me/images/next-js/ci-cd/12.png">

이제 루트 계정에서 로그아웃한 뒤 아래 이미지와 같이 기존 AWS 로그인 화면으로 이동합니다. **IAM 사용자**를 선택하고 본인이 만든 **계정 별칭**을 입력합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/13.png">


아래 이미지와 같이 **계정 별칭**, **사용자 이름**, 자동으로 생성된 비밀번호를 **암호**에 입력하고 로그인합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/14.png">


IAM 계정을 생성할 때 **비밀번호 재설정 필요 옵션**을 체크했기 때문에 아래 이미지와 같이 비밀번호를 재생성하는 화면이 나타납니다. 원하는 비밀번호로 재생성하면 됩니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/15.png">


#### 기타

IAM과 관련해서는 그룹 및 MFA와 같은 개념이 추가적으로 존재합니다. 관련하여 궁금한 분들은 직접 검색하길 추천드립니다. 개별 사용자로 IAM 계정을 만드는 경우는 그룹으로 묶이는 경우보다 덜 자주 발생합니다.



### AWS Route53

#### 개념

Route53은 도메인 구매 및 관리를 위한 서비스입니다. 이 글에서는 도메인 구매를 [호스팅케이알(HK)](https://www.hosting.kr/)에서 진행하였으며 Route53에서는 다른 AWS 서비스와의 연동을 위한 도메인 관리 기능만 사용합니다.  

#### 호스팅 생성

아래 이미지와 같이 Route53을 검색하여 호스팅 영역으로 이동해 아래 이미지와 같이 새로운 호스팅 영역을 생성합니다. 이때 도메인 이름에는 본인이 구매한 도메인을 입력하고 유형은 퍼블릭 호스팅 영역을 선택하여 생성합니다. 설명 및 태그의 경우 선택 사항입니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/17.png">

#### 네임서버 등록

생성을 마치면 아래 이미지와 같이 **레코드**를 확인할 수 있습니다. 여기서 중요한 건 네임서버(Name Server)를 의미하는 **NS**입니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/18.png">

이제 해당 **NS** 값 네 개를 복사하여 아래 이미지와 같이 본인이 구매한 도메인 업체에 접속하여 네임서버를 입력합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/19.png">


### ACM

#### 개념

HTTP보다 보안이 높은 HTTPS를 사용하기 위해서는 SSL/TLS 인증서를 발급 받아야합니다.  

ACM(AWs Cerficate Manager)은 이런 인증서를 발급해주는 서비스입니다.  


#### 인증서 생성

아래 이미지와 같이 Certificate Manager를 검색하여 이동한 뒤 **인증서 프로비저닝**을 선택합니다.  

!!! "warning" 주의

    CloudFront에서 적용 가능한 인증서의 지역은 `us-east-1`이어야하기 때문에 먼저 우측 상단에서 해당 지역(`us-east-1`)이 맞는지 확인한 뒤 아닐 경우 선택하고나서 인증서를 생성합니다.

<img src="https://weekwith.me/images/next-js/ci-cd/20.png">


다음으로 아래 이미지와 같이 **공인 인증서 요청**을 선택합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/21.png">

해당 인증서를 적용할 도메인을 입력합니다. 아래 이미지와 같이 앞서 Route53 서비스에 입력했던 도메인(`artfrom.life`)을 입력합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/22.png">

다음으로 아래 이미지와 같이 검증 방법을 **DNS 검증**을 선택합니다. 이를 통해 앞서 Route53에 등록한 도메인을 통해 검증 받을 수 있습니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/23.png">

**검증** 전 마지막 **검토 및 요청**에서 아래 이미지와 같이 본인이 알맞게 입력 및 선택했는지 한 번 더 확인합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/24.png">

다음으로 **검증**에서 아래 이미지와 같이 **CNAME**을 기록할 수 있게 **이름**과 **값**을 알려줍니다. Route 53에서 레코드 생성을 눌러 직접 입력하는 대신 AWS에서 대신 간편하게 해당 도메인에 **CNAME**을 입력할 수 있게 합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/25.png">

이제 기다리면 아래 이미지와 같이 **상태**가 **발급 완료**가 된 것을 확인할 수 있습니다. 이제부터 정삭적으로 해당 도메인에서 HTTPS를 사용할 수 있게 되었습니다.  


### AWS S3

#### 개념

AWS S3는 파일을 저장할 수 있는 파일 스토리지(File Storage) 중 하나입니다. S3의 가장 큰 특징 중 하나는 다른 AWS 서비스에서도 S3에 업로드 된 파일에 바로 접근할 수 있다는 것입니다. 파일 스토리지라는 이름을 통해 알 수 있듯 S3에는 보통 웹 애플리케이션에서 사용되는 이미지 등이 업로드 됩니다. 이번에는 S3를 통해 정적 웹 사이트를 호스팅해보겠습니다.  

#### 버킷 생성

**S3**를 검색하여 아래 이미지와 같이 **버킷**으로 이동한 다음 **버킷 만들기** 버튼을 클릭합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/26.png">

이제 아래 이미지와 같이 생성할 **버킷 이름**을 입력하고 맨 아래 **버킷 만들기** 버튼을 클릭하여 버킷을 생성합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/27.png">

이제 아래 이미지와 같이 만든 **버킷** (이 예시에서는 `next-js-deployment-pratice`) 으로 이동하여 **객체** 속성에서 **업로드** 버튼을 클릭하여 배포할 파일 및 폴더를 업로드합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/28.png">

#### Next.js 배포 관련 주의 사항

Next.js의 빌드는 React.js와는 조금 다릅니다. `next export`에 관해 이미 알고 있다면 해당 부분을 넘어가도 좋습니다.  

Next.js를 빌드하기 위해서는 `package.json` 파일 속 `"scripts"` 객체의 `"build"` 값에 `next export`를 추가해야 합니다.  

정리하면 `"build": "next build && next export"` 와 같습니다.  

next build 의 경우 `.next`라는 폴더에 애플리케이션을 빌드합니다. `next export`의 경우 `.out` 폴더에 모든 페이지를 정적 HTML 파일로 내보냅니다. 따라서 `next export`를 통해 생성된 `.out` 폴더에는 빌드된 `.next` 폴더가 포함되어 있습니다.  

관련된 더 자세한 설명은 이 글에서는 생략하겠습니다. 혹시 궁금한 분들은 Next.js 공식 도큐먼트에 나와있는 [정적 HTML 내보내기](https://nextjs.org/docs/advanced-features/static-html-export)를 확인하기 바랍니다.  


#### 빌드 파일 및 폴더 업로드

이제 아래 이미지와 같이 **파일 추가** 및 **폴더 추가** 버튼을 눌러 `.out` 폴더 내에 있는 프로젝트의 빌드 파일을 업로드합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/32.png">


#### 정적 웹 사이트 배포

모든 빌드 파일 및 폴더를 업로드 완료했으면 이제 아래 이미지와 같이 **속성** 탭으로 넘어옵니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/33.png">

그리고 아래 이미지와 같이 해당 탭 맨 아래 존재하는 **정적 웹 사이트 호스팅** 부분에서 **편집** 버튼을 클릭합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/34.png">

아래 이미지와 같이 **정적 웹 사이트 호스팅**을 **활성화**하고 **호스팅 유형**을 **정적 웹 사이트 호스팅**으로 설정합니다.  

**인덱스 문서**의 경우 루트 경로(`/`)로 접근했을 때 보여질 문서를 의미합니다. **오류 문서**의 경우 오류가 발생했을 때 보여질 문서로 이 예시에서는 루트 경로로 리다이렉션 되게 하기 위해 **인덱스 문서** 및 **오류 문서** 모두 `index.html`로 설정합니다.

!!! warning "주의"

    `.out` 폴더 내에 있는 파일 및 폴더를 버킷에 정상적으로 업로드 했으면 `index.html`이 해당 버킷의 루트 디렉토리에 존재합니다.

    만약 `index.html` 파일이 루트 디렉토리에 존재하지 않을 경우 S3는 해당 파일을 읽을 수 없습니다.

<img src="https://weekwith.me/images/next-js/ci-cd/35.png">

정적 웹 사이트 호스팅을 완료하면 아래 이미지와 같이 **버킷 웹 사이트 엔드포인트**가 생성된 것을 확인할 수 있습니다. 아래 URL로 한 번 접속해봅시다.  

<img src="https://weekwith.me/images/next-js/ci-cd/36.png">

그러면 이제 아래 이미지와 같이 **403 Forbidden** 오류를 반환한 것을 확인할 수 있습니다. 반환된 오류의 메세지를 확인해보면 **AccessDenied**라고 되어 있습니다. 이는 퍼블릭 액세스 권한을 막아놨기 때문입니다. 따라서 이제 버킷의 권한을 수정해줘야 합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/37.png">

이제 해당 버킷의 **권한** 탭으로 넘어와 **퍼블릭 액세스 차단(버킷 설정)** 부분을 **편집** 버튼을 눌러 수정합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/38.png">

아래 이미지와 같이 외부에서 조회할 수 있게, 다시 말해 어느 곳에서나 정적 웹 사이트를 열람할 수 있게 **모든 퍼블릭 액세스 차단**을 **비활성**합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/39.png">

이제 아래 이미지와 같이 조금 내려와 **버킷 정책** 부분에서 **편집**을 눌러 정책을 설정해줍니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/40.png">

본인이 직접 정책을 입력할 수 있지만 조금 더 간단하게 설정하기 위해 아래 이미지와 같이 **정책 생성기** 버튼을 눌러 도움을 받습니다. 이때 **버킷 ARN** (이 예시에서는 `arn:aws:s3:::next-js-deployment-practice`) 을 복사합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/41.png">

이제 아래 이미지와 같이 정책을 선택합니다. 하나씩 정리하면 아래와 같습니다.  

* **Select Type of Policy**: 정책 유형을 의미합니다. S3 Bucket Policy를 선택합니다.
* **Effect**: 정책의 유형을 허용(Allow) 또는 거부(Deny)로 선택할 수 있습니다. 허용(Allow)을 선택합니다.
* **Principal**: 특정 부분에만 권한을 부여할 수 있습니다. * 를 입력하여 모든 부분에 권한을 허용합니다.
* **Actions**: 어떤 권한을 부여할 것인지 선택합니다. 정적 웹 사이트 열람, 다시 말해 조회( GET )만 허용하는 것이기 때문에 GetObject 만을 선택합니다.

<img src="https://weekwith.me/images/next-js/ci-cd/42.png">

이제 아래 이미지와 같이 **Amazon Resource Name (ARN)**에 아까 복사한 **버킷 ARN** (이 예시에서는 `arn:aws:s3:::next-js-deployment-practice`) 을 입력하고 **Add Statement** 버튼을 클릭합니다.

<img src="https://weekwith.me/images/next-js/ci-cd/43.png">

이제 아래 이미지와 같이 정리된 정책을 확인하고 **Generate Policy** 버튼을 눌러 정책을 생성하고 이를 복사합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/44.png">

다시 **버킷 정책 편집**으로 돌아와 아래 이미지와 같이 앞서 복사한 정책을 붙여넣습니다.  

이때 유의할 점은 `Resource` 부분의 끝에 `/*` 를 추가해야 합니다. 이는 해당 버킷 (이 예시에서는 `arn:aws:s3:::next-js-deployment-practice`) 에 업로드 된 모든 객체에 대해 해당 정책을 적용하겠다는 의미입니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/45.png">

버킷 정책을 성공적으로 마무리하면 아래 이미지와 같이 생성되어 있습니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/46.png">

이제 다시 **버킷 웹 사이트 엔드포인트**에 접근하면 아래 이미지와 같이 정상적으로 정적 웹 사이트가 배포된 것을 확인할 수 있습니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/47.png">


#### 기타

S3 서비스 및 특히 정책에 관해서는 심화된 내용이 많습니다. 관련해서는 다른 책, 강의 또는 AWS 공식 웹 사이트에 제공되어 있는 [자습서](https://aws.amazon.com/ko/getting-started/hands-on/?awsf.getting-started-category=*all&awsf.getting-started-level=*all&awsf.getting-started-content-type=*all)를 통해 찾아보기 바랍니다.


### AWS CloudFront

#### 개념

CloudFront는 AWS에서 제공하는 CDN(Contents Delivery Network) 서비스입니다.  

CDN과 관련된 설명은 [[ React의 모든 것 ] 01. DOM과 Virtual DOM, 그리고 CDN]()에서 간단하게 확인해보기 바랍니다.  


#### 배포 생성

이제 CloudFront 서비스를 통해 해당 정적 웹 사이트에 구매한 도메인을 연결하고 HTTP로 접근하는 연결을 전부 HTTPS로 리다이렉션 해주겠습니다.  

우선 아래 이미지와 같이 CloudFront를 검색한 뒤 **배포**로 이동하여 **배포 생성** 버튼을 누릅니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/48.png">

아래 이미지와 같이 **원본 도메인**에 앞서 만들었던 S3 버킷의 도메인 (이 예시에서는 `next-js-deployment-practice.s3.us-east-1.amazonaws.com`) 을 입력 또는 선택합니다. 아래 **이름**의 경우 해당 **원본 도메인**을 입력 또는 선택할 경우 자동으로 입력됩니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/49.png">

HTTP로 접근하는 모든 연결을 HTTPS로 리다이렉션 해주기 위해 아래 이미지와 같이 **뷰어 프로토콜 정책**을 **Redirect HTTP to HTTPS**로 선택합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/50.png">

이제 아래 이미지와 같이 **대체 도메인 이름(CNAME)**, **사용자 정의 SSL 인증서**, **기본값 루트 객체**를 입력하여 S3를 통해 배포되었던 정적 웹 사이트에 구매한 도메인 (이 예시에서는 `artfrom.life`) 및 HTTPS를 적용합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/51.png">

아래 이미지와 같이 **대체 도메인 이름(CNAME)**에 본인이 구매하여 Route5에 등록한 도메인 (이 예시에서는 `artfrom.life`) 을 입력하고 관련해서 ACM을 통해 만들었던 인증서를 **사용자 정의 SSL 인증서**에서 선택합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/52.png">


**기본값 루트 객체**의 경우 아래 이미지와 같이 `index.html`을 입력한 뒤 배포 생성 버튼을 클릭하여 CloudFront을 배포합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/53.png">


#### Route53 서비스 연결

이제 배포된 CloudFront를 Route53 서비스에 연결하여 구매한 도메인으로 접근했을 때 S3에 배포된 정적 웹 사이트가 연결되게 해야합니다.  

이를 위해 아래 이미지와 같이 Route53 서비스로 이동하여 **호스팅 영역**에서 이전에 생성했던 호스팅 (이 예시에서는 `artfrom.life`) 을 선택하고 **레코드 생성** 버튼을 눌러 레코드를 생성합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/54.png">

아래 이미지와 같이 CloudFront 서비스를 연결하기 위해 **별칭** 토글을 클릭한 뒤 해당 **트래픽 라우팅 대상**에 **CloudFront 배포에 대한 별칭**을 선택하고 그 아래 생성한 CloudFront 배포를 선택합니다. 그리고 **레코드 생성** 버튼을 눌러 해당 레코드를 생성합니다.  

이때 유의할 점은 다른 AWS 서비스 중 하나인 CloudFront를 Route53과 연동하는 것이기 때문에 **레코드 유형**이 **A - IPv4 주소 및 일부 AWS 리소스로 트래픽 라우팅**으로 되어 있어야 한다는 것입니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/55.png">

이제 브라우저에서 본인이 구매한 도메인 (이 예시에서는 `artfrom.life`) 으로 접속해보면 아래 이미지와 같이 **버킷 웹 사이트 엔드포인트**로 접속했던 것과 동일한 페이지가 나타나는 것을 알 수 있습니다. 또한 주소를 자세히 살펴보시면 HTTPS 적용이 된 것을 알 수 있습니다.

<img src="https://weekwith.me/images/next-js/ci-cd/56.png">


### GitHub Actions를 통한 CI/CD

#### CI/CD

CI(Continuous Intergration)와 CD(Continuous Deployment 또는 Delivery)는 단어 그대로 지속적인 통합과 배포(또는 제공)를 의미합니다.  

지속적인 통합을 의미하는 CI(Continuous Intergration)의 경우 애플리케이션에 대한 새로운 변경 사항이 발생했을 때 해당 변경 사항을 자동으로 빌드 및 테스트하여 통합하는 걸 의미합니다. CI를 사용하지 않았을 경우 새로운 변경 사항이 발생했을 때 충돌(Conflict) 등의 이슈가 발생하여 서비스에 장애가 발생할 수 있으며 이를 해결 및 예방하기 위해 수동(Manually)으로 빌드 및 테스트를 진행해야 합니다. CI를 사용할 경우 이러한 과정을 자동으로 작업해주고 결과를 개발자에게 알려줍니다.  

지속적인 배포를 의미하는 CD(Continuous Deployment)의 경우 쉽게 애플리케이션 배포를 자동화하는 것을 의미합니다. 예를 들어 깃헙(Github)과 같은 클라우드 저장소를 사용하여 애플리케이션에 대한 소스 코드를 관리할 때 이를 AWS 같은 클라우드 호스팅 서비스에 배포할 경우 CD를 사용하지 않으면 위 예시와 같이 직접(Manually) 배포를 진행해야 합니다. CD를 사용할 경우 이러한 과정을 자동으로 작업해주고 결과를 개발자에게 알려줍니다.  


CI/CD 파이프라인을 이미지로 표현하면 아래와 같습니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/65.png">

[CircleCI](https://circleci.com/), [Travis CI](https://travis-ci.org/), [Jenkins](https://www.jenkins.io/) 등 다양한 CI/CD 툴(Tool)이 존재하지만 이번에는 GitHub Actions를 활용하여 간단하게 적용해보겠습니다.  


#### Workflow 파일 설정

프로젝트의 루트 폴더에 `.github` 폴더를 만든 뒤 해당 폴더 내에 `worflows` 폴더를 만들고 아래 이미지와 같이 `yml` 파일 (이 예시에서는 `deploy.yml`) 을 만듭니다.

<img src="https://weekwith.me/images/next-js/ci-cd/62.png">


GitHub Action의 경우 기본적으로 **Worflow**, **Event**, **Job**, **Step**, **Action** 등의 개념으로 구성되어 있습니다.  


* **Workflow**: GitHub Action이 실행하게 될 프로세스입니다. `.github` 폴더 속 `workflows` 폴더 내에 있는 `yml` 파일 (이 예시에서는 `deploy.yml`) 을 실행합니다.

* **Event**: **Workflow** 실행을 유발(Trigger)하는 작업입니다. 이 예시에서는 `on` **Event**를 통해 `main` 브랜치에 `push` 되었을 때 **Workflow**를 실행합니다.

* **Job**: 여러 **Step**으로 구성되어 GitHub 내의 가상 인스턴스에서 실행됩니다.

* **Step**: 실행해야 할 태스크들의 집합입니다. **Action** 등을 실행할 수 있습니다. 

* **Action**: [GitHub Marketplace](https://github.com/marketplace?type=actions) 등에 작성된 **Action**을 사용하거나 또는 본인이 직접 `run`을 통해 명령어를 실행하게 입력할 수 있습니다. 다음은 이 예시에서 사용된 **Action**입니다. `name`을 기준으로 작성되어 있습니다.

    * **Git Checkout**: `checkout@v2` **Action**을 활용하여 깃헙(GitHub)에 업로드된 소스 코드를 가져옵니다.

    * **Use Node.js version 14.x**: `setup-node@v1` **Action**을 활용하여 `14.x` 버전의 노드를 가상 인스턴스에 설치합니다.

    * **Build**: `run` **Action**을 활용하여 가상 인스턴스에 패키지를 설치(이 예시에서는 `yarn install --frozen-lockfile`)하고 빌드 (이 예시에서는 `yarn build`) 합니다.
    
    * **Configure AWS credentials**: `configure-aws-credentials@v1` **Action**을 활용하여 가상 인스턴스에서 AWS CLI를 통해 필요한 서비스에 접근할 수 있게 `aws-access-key-id`, `aws-secret-access-key`, `aws-region`을 입력하여 사용자를 등록합니다.

    * **Deploy to S3**: `run` **Action**을 활용하여, AWS CLI를 통해 (이 예시에서는 `sync`를 사용하는데 이는 동기화를 의미합니다. `--acl public-read` 옵션의 경우 해당 업로드 파일 및 폴더를 읽을 수 있는 객체로 설정하고 `--delete`의 경우 기존의 객체에 동기화하는 객체 내에 존재하지 않는 파일 및 폴더가 있을 경우 자동으로 삭제합니다.) 빌드된 파일 및 폴더를 S3 서비스에 업로드합니다.

    * **CloudFront Invalidate Cache**: `run` **Action**을 활용하여, AWS CLI를 통해(이 예시에서는 `cloudfront create-invalidation`를 사용합니다. 이때 `--distruibution-id` 옵션은 CloudFront에 기존에 배포된 아이디를 의미하며 `--paths` 의 경우 삭제할 캐시의 경로를 의미합니다. 이 예시에서는 전부 삭제하여 새로 업로드된 S3를 연결하기 때문에 `'/*'` 로 설정합니다.) 기존 배포한 콘텐츠(캐시)를 삭제하고 S3에 새로 배포된 콘텐츠를 연결합니다.


이때 해당 `yml` 파일 (이 예시에서는 `deploy.yml`) 을 잘 보면 마치 변수를 사용하듯 `secrets.SOMETHING` 과 같은 형태로 작성되어 있는 걸 확인할 수 있습니다. 이는 AWS 액세스 키와 같이 보안에 취약한 값들을 GitHub 레포지토리 내에 **Secrets**라는 곳에서 관리하여 **Action** 단계에서 사용할 때 불러오게 (이 예시에서는 `SOMETHING`) 합니다.  

#### GitHub Secret

이제 레포지토리에 비밀 값을 등록하기 위해 아래 이미지와 같이 배포하고자 하는 애플리케이션이 업로드되어 있는 레포지토리로 이동하여 **Settings**로 넘어갑니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/58.png">

이제 아래 이미지와 같이 왼쪽 메뉴들 중에서 **Secrets**를 클릭합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/59.png">

이제 아래 이미지와 같이 **New repository secret** 버튼을 클릭하여 비밀 값을 하나씩 등록합니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/60.png">

아래 이미지와 같이 **Name**의 경우 `yml` 파일 (이 예시에서는 `deploy.yml`) 내에 작성된 `secrets.SOMETHING` 부분에서 `SOMETHING`에 해당합니다. 따라서 해당 이름과 동일하게 입력한 다음 **Value** 또한 알맞게 입력합니다. 이 예시에서는 `AWS_ACCESS_KEY_ID`를 **Name**에 입력하고 이에 알맞는 액세스 키를 이전에 IAM 계정을 생성하며 다운로드 받았던 CSV 파일에서 복사하여 붙여 넣었습니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/61.png">

이 예시에서 입력해야 할 **Secrets**은 아래와 같습니다.  

* `AWS_ACCESS_KEY_ID` : IAM 계정을 생성할 때 발급 받은 액세스 키입니다.
* `AWS_SECRET_ACCESS_KEY` : IAM 계정을 생성할 때 발급 받은 시크릿 키입니다.
* `AWS_REGION` : AWS 지역을 의미하며 이 예시에서는 `us-east-1`로 설정하였습니다.
* `BUILD_DIRECTORY` : S3에 업로드할 폴더를 의미하며 이 예시에서는 `next export`를 통해 생성되는 `out`입니다.
* `AWS_S3_BUCKET_NAME` : 빌드된 파일 및 폴더가 업로드될 S3 버킷의 이름을 의미하며 `s3://next-js-deployment-practice`와 같습니다.
* `AWS_CLOUDFRONT_DISTRIBUTION_ID` : S3와 연결된 CloudFront의 배포 ID입니다.


해당 **Secrets**를 모두 입력했으면 이제 푸쉬(`git push`)하여 변경 사항을 메인(`main`) 브랜치에 반영합니다. 그러면 아래 이미지와 같이 **Actions**에서 성공적으로 AWS 서비스에 배포가 된 것을 확인할 수 있습니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/63.png">

다시 본인의 도메인 (이 예시에서는 `artfrom.life`) 에 접근하면 GitHub 레포지토리에 변경 사항만 적용시켰을 뿐인데 AWS에도 자동으로 반영되어 아래 이미지와 같이 성공적으로 배포 및 적용된 것을 확인할 수 있습니다.  

<img src="https://weekwith.me/images/next-js/ci-cd/64.png">


## 결론

**AWS IAM**, **S3**, **CloudFront**, **Route53**, **Certificatoin Manager**를 사용하여 구매한 도메인을 통해 정적 웹 사이트를 배포해봤습니다.  

그리고 **GitHub Actions**를 활용해 **CI/CD** 파이프라인을 구축하여 애플리케이션에 변동 사항이 생겼을 때 이를 자동으로 빌드하고 배포되게 하였습니다.  

--- 

## 참고

[GitHub 레포지토리](https://github.com/week-with-me/next-js-deployment-practice)