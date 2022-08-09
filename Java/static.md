# static
**static**은 필드와 메서드에 붙이는 제한자이다.  
* `static 특징`
  * 프로그램 시작 전에 Method 영역 메모리에 로딩된다.
  * 객체를 생성하지 않고도 static 필드, 메서드 사용이 가능하다.
* `static 장점`
  * static은 메모리를 한번만 할당하므로 효율적인 메모리 사용이 가능하다.
  * 객체를 생성하지 않고 사용하기 때문에 속도가 빠르다. 
* `static 단점`
  * 무분별한 static의 사용은 메모리 유수의 원인이 된다.
    * static은 프로그램 종료 시점에 메모리를 반환하므로 GC의 대상이 아니다.

### static Method
* static 메서드는 오버라이딩 할 수 없다.
* static 메서드 안에서는 static 변수만 접근 가능하다.  
> 아래와 같이 static 메서드에서 인스턴스 변수(count)에 접근이 불가능하다.
> ```java
> public class LikeCount {
>  int count;
>  
>  public static int getCount() {
>    return count;   // 접근 불가능
>  }
>}
>```
> 아래와 같이 static 메소드 안에선 static 변수만 접근 가능하다.
>```java
>public class LikeCount {
>  static int count;
>  
>  public static int getCount() {
>    return count;  // count가 static 변수이므로 접근 가능
>  }
>}
>```

### static 변수
* 같은 클래스에서 생성된 객체들은 static을 붙인 필드의 값을 공유한다.
> 두 객체는 static 변수의 메모리를 공유한다.
> ```java
> public class Likecount {
>   static int count;
>
> public LikeCount() {
>   this.count++;
>   System.out.println("좋아요 개수 : " + count);
> }
> 
> public static void main(String[] args) {         
>   LikeCount lc1 = new LikeCount();        
>   LikeCount lc2 = new LikeCount();     
> }
>
> [Output]
> 좋아요 개수 : 1
> 좋아요 개수 : 2

## 결론
* 인스턴스들이 공통적으로 사용해야하는 것에 static을 붙인다.
* static이 붙은 멤버변수는 인스턴스를 생성하지 않아도 사용할 수 있다. (클래스명.static 변수)
* static이 붙은 메소드에서는 static 변수만 접근 가능하다.
  * static 메소드는 인스턴스 생성 없이 호출 가능한 반면, 인스턴스 변수는 인스턴스를 생성해야만 존재하기 때문에 
    staic이 붙은 메소드에서 인스턴스 변수를 사용할 수 없다.
* 메소드 내에서 인스턴스 변수를 사용하지 않는다면, static을 붙이는 것을 고려한다.
  * 메소드 호출시간이 짧아지기 때문에 효율이 좋다.
