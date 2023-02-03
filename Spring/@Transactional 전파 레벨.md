# @Transactional 전파 레벨
* `@Transactional`: 해당 메서드를 하나의 트랜잭션 안에서 진행할 수 있도록 만들어주는 역할
*  `트랜잭션 전파 레벨`: 진행되고 있는 트랜잭션에서 다른 트랜잭션이 호출될 때 합류 조건
*  @Transactional의 Propagation 옵션을 통해 전파 레벨 설정 가능
* DB에서 자체적으로 제공해주는 트랜잭션 격리레벨과 다르게 전파레벨은 스프링에서 개발자들의 편의를 위해 제공해주는 기능이다.

### 1. REQUIRED (기본값)
* 부모 트랜잭션이 존재한다면 부모 트랜잭션으로 합류하고, 부모 트랜잭션이 없다면 새로운 트랜잭션 생성
* 중간에 롤백이 발생한다면 모두 하나의 트랜잭션이므로 부모, 자식 모두 롤백
<img src="https://user-images.githubusercontent.com/50009240/216627596-4a865108-e1d9-4b59-8493-db500e77f771.png" width="520" height="470">

### 2. REQUIRES_NEW
* 무조건 새로운 트랜잭션 생성
* 각각의 트랜잭션이 롤백되더라도 서로 영향을 주지 않는다.
<img src="https://user-images.githubusercontent.com/50009240/216628909-c0303497-eac0-46d1-ad8a-e30f6bb6dddc.png" width="550" height="500">

### 3. MANDATORY
* 부모 트랜잭션에 합류한다. 만약 부모 트랜잭션이 없다면 예외를 발생시킨다.
<img src="https://user-images.githubusercontent.com/50009240/216629692-4ffbbfee-b69c-45d9-b0a3-21de15cd4a9a.png" width="550" height="450">                            

### 4. NESTED
* 부모 트랜잭션이 존재한다면, 중첩 트랜잭션을 생성하고, 존재하지 않는다면 새로운 트랜잭션을 생성한다.
* 중첩된 트랜잭션 내부에서 롤백 발생 시 해당 중첩 트랜잭션의 시작 지점까지만 롤백된다.
* 중첩 트랜잭션은 부모 트랜잭션이 커밋될 때 같이 커밋된다.
* REQUIRES_NEW와 차이점은 중첩 트랜잭션 커밋 시점이다.
<img src="https://user-images.githubusercontent.com/50009240/216630649-c4002046-2a8f-46b0-abcf-91acfb4e6dda.png" width="550" height="500">

### 5. NEVER
* 트랜잭션을 생성하지 않는다.
* 부모 트랜잭션이 존재한다면 예외를 발생시키고, 존재하지 않는다면 트랜잭션을 생성하지 않고 기능을 진행한다.
<img src="https://user-images.githubusercontent.com/50009240/216631085-48e9bfa2-690f-4eab-b0c0-d4eda7318de5.png" width="520" height="430">

### 6. SUPPORTS
* 부모 트랜잭션이 존재한다면, 합류한다.
* 부모 트랜잭션이 없다면 트랜잭션을 생성하지 않고 기능을 진행한다.

### 7. NOT_SUPPORTS
* 부모 트랜잭션이 있다면 보류시킨다.
* 진행중인 부모 트랜잭션이 없다면, 트랜잭션을 생성하지 않고 기능을 진행한다.
<br></br>
> Reference: https://deveric.tistory.com/86
