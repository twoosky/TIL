# IAM : Identity Access Management

## IAM 이란?
IAM은 AWS 리소스에 대한 사용자의 접근 권한을 관리하는 서비스입니다.  
* `Root User` : AWS의 모든 서비스에 대해 접근 가능합니다.
* `IAM User` : 특정 서비스에 대한 특정 권한만을 부여받아 리소스에 제한적으로 접근 가능합니다.

## IAM User
* AWS의 기능과 자원을 이용하는 사용자 또는 애플리케이션을 의미합니다.
* IAM User 생성시 부여받는 Access Key와 Secret Key를 통해 AWS API를 호출할 수 있습니다.
* 아래와 같이 각 IAM User 마다, 특정 권한을 부여해 리소스 접근을 제한합니다.
<img src="https://github.com/twoosky/TIL/assets/50009240/7185e566-d9d8-4b12-9836-f2731429cadb" width="520" height="300">

## IAM Group
* IAM 사용자들의 집합입니다. 그룹에 속한 사용자에게 공통된 권한을 부여할 수 있습니다.
* 예를 들면, 특정 IAM Group에 Cloudwatch ReadOnly 권한을 부여한 경우, 해당 그룹에 속한 IAM User들에게도 자동으로 권한이 적용됩니다.
* 사용자는 여러 그룹에 속할 수 있습니다.

## IAM Role
* 리소스에 대한 엑세스 권한이 없는 사용자나 특정 서비스에게 일시적으로 권한을 위임하는 것 입니다.
* 역할에는 IA
