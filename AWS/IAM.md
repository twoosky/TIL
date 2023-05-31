# IAM : Identity Access Management

# IAM 이란?
IAM은 AWS 리소스에 대한 사용자의 접근 권한을 관리하는 서비스입니다.  
* `Root User` : AWS의 모든 서비스에 대해 접근 가능합니다. AWS 계정을 생성하면 해당 계정이 Root User가 됩니다. 
* `IAM User` : 특정 서비스에 대한 특정 권한만을 부여받아 리소스에 제한적으로 접근 가능합니다.

# IAM 구성
IAM에서 관리하는 리소스를 크게 보면 다음과 같습니다.
* IAM User
* IAM Group
* IAM Role
* IAM Policy

## IAM User
* AWS의 기능과 자원을 이용하는 사용자 또는 애플리케이션을 의미합니다.
* IAM User 생성시 부여받는 Access Key와 Secret Key를 통해 AWS API를 호출할 수 있습니다.
* 아래와 같이 각 IAM User 마다, 특정 권한을 부여해 리소스 접근을 제한합니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/7185e566-d9d8-4b12-9836-f2731429cadb" width="530" height="280">

## IAM Group
* IAM 사용자들의 집합입니다. 그룹에 속한 사용자에게 공통된 권한을 부여할 수 있습니다.
* 예를 들면, 특정 IAM Group에 Cloudwatch ReadOnly 권한을 부여한 경우, 해당 그룹에 속한 IAM User들에게도 자동으로 권한이 적용됩니다.
* 사용자는 여러 그룹에 속할 수 있습니다.

## IAM Role
* AWS 리소스에 대한 엑세스 권한이 없는 사용자나 특정 서비스에게 임시 엑세스 권한을 부여하는 것 입니다.
* IAM Role은 IAM User 또는 AWS 서비스에 역할 부임 시 임시 보안 자격 증명을 제공합니다.
* 따라서, 해당 증명 세션이 유효한 시간 동안에만 해당 역할을 사용할 수 있습니다.

**역할 사용 주체**
* 동일한 AWS 계정 내에 있는 IAM User
* 다른 AWS 계정의 IAM User
* AWS가 제공하는 서비스 (EC2 ..)

## IAM Policy
* IAM 자격 증명(User, Group, Role)에게 줄 수 있는 권한을 정의한 문서입니다.
* IAM 정책은 하나 이상의 AWS 리소스에 대한 권한을 JSON 형식으로 정의합니다.
* ex) AmazonEC2FullAccess, AmazonS3ReadOnlyAccess ..


## IAM 권한 검증 절차 예시
실제로 사용자/애플리케이션이 AWS 서비스에 접근할 때 어떠한 과정으로 권한 엑세스가 되는지 살펴보겠습니다.

**사용자(User)가 S3를 사용하고 싶은 경우**
1. 사용자(user)에게 S3 정책이 설정되어 있는가?
2. 사용자가 속한 그룹(group)에게 S3 정책이 설정되어 있는가?
3. 사용자의 역할(role)에 S3 정책이 설정되어 있는가?
<img src="https://github.com/twoosky/TIL/assets/50009240/b983a55d-2d35-4ca6-a313-188d3134c12c" width="700" height="380">

# IAM Hands-on
## IAM 사용자 생성 및 정책 부여하기
1. 사용자 메뉴에 들어가서 사용자 추가 버튼을 클릭합니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/d66ef1d9-0fa2-423a-ab9c-93942c57b80f" width="650" height="300">

2. 추가할 사용자 정보를 설정합니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/af297625-5f6c-48c9-a11e-203eab12daaf" width="650" height="600">

3. 사용자에게 AmazonS3FullAccess 정책을 부여한 뒤, 다음을 클릭합니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/314447be-3e77-402b-be0c-db80193a43a6" width="650" height="450">

4. 검토 및 생성은 건너뛰고, 사용자 생성을 클릭해 IAM User를 생성합니다.

5. 콘솔 url 내 12자리 숫자는 IAM User의 계정 ID입니다. 아래 정보로 IAM User에 로그인할 수 있습니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/4eed00f0-210c-418a-8524-3be126b40e5e" width="650" height="350">

6. accessKey, secretKey는 [해당 사용자 상세정보 -> 보안 자격 증명 - > 엑세스 키 만들기] 로 생성할 수 있습니다.

## IAM Group 생성하기
1. 사용자 그룹 메뉴에 들어가서 그룹 생성 버튼을 클릭합니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/5b331c22-f2fe-4d02-90ee-2ed69a8565bd" width="650" height="200">

2. 해당 그룹에 위에서 생성한 IAM User를 추가합니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/6638ef91-cc81-44cb-9b7e-66f25d751f74" width="650" height="400">

3. 해당 그룹에 AmazonEC2FullAccess 정책을 부여한 뒤 그룹을 생성합니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/16a31d35-6e1d-49b0-a3b7-d1c19d6504d2" width="650" height="300">

4. 아래와 같이 그룹에 속한 사용자에게 해당 그룹의 정책이 적용된 것을 볼 수 있습니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/8592cbc3-28e3-46df-9dd4-5d709427a3cb" width="800" height="430">

## IAM User 계정 접속하기
1. IAM User 계정 ID는 해당 사용자의 ARN의 숫자 12개
<img src="https://github.com/twoosky/TIL/assets/50009240/030df5ee-065e-4a90-9420-d321e38dd75c" width="650" height="300">

2. IAM User 계정 ID, 사용자 이름, 사용자 생성 시 설정했던 비밀번호로 IAM 계정 로그인
<img src="https://github.com/twoosky/TIL/assets/50009240/c7645a77-08e4-4cc9-adcb-6c889bf0d3c8" width="300" height="450">

<img src="https://github.com/twoosky/TIL/assets/50009240/56a81dff-1366-4030-9993-58ce57a1f5d5" width="300" height="400">


## IAM Role 생성하기
### 사용자에게 역할 부여하기
1. 역할 메뉴에 들어가서 역할 만들기 버튼을 클릭합니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/af66e6d1-40cf-4e40-a423-6523d88310a8" width="650" height="300">

2. 역할을 부여할 IAM User를 선택합니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/b08da8ef-b040-4a6e-8cc1-828e6beded03" width="650" height="450">

3. 역할에 부여할 원하는 정책을 선택합니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/a84b7aae-0d59-41e6-ac17-fd2bfe7229b9" width="650" height="550">

4. 역할 이름과 설명을 설정하고, 역할을 생성합니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/5fac288d-55e2-4e5c-a9a8-4c5d4ebe0e8d" width="650" height="600">

5. IAM 계정에 방금 생성한 역할이 부여되었음을 확인할 수 있습니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/9009001b-6881-4851-b95e-fc4d462fd4a8" width="650" height="200">

### 리소스에 역할 부여하기
1. 역할을 부여할 AWS 서비스를 선택합니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/f3c7f28f-9bb8-4576-9313-88a282f19846" width="600" height="500">

2. 역할에 부여할 원하는 정책을 선택합니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/c803bde5-5161-4966-b0c7-164e4c49a40a" width="600" height="600">

3. 역할 이름과 설명을 설정하고, 역할을 생성합니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/502e7a43-2340-4004-8102-ad03f54f4de4" width="600" height="550">

4. 위에서 생성한 EC2 역할은 EC2 인스턴스를 생성할 때 부여할 수 있습니다. 











          
          
          
          
          
          
          
          
          
          
          
          
