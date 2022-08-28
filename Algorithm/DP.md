# Dynamic Programming
* 다이나믹 프로그래밍은 `동적 계획법`이라고도 부른다.
* 큰 문제를 작은 문제로 나누어 푸는 방식의 알고리즘이다. 즉, 한 번 계산한 문제는 다시 계산하지 않도록 하는 알고리즘  


다이나믹 프로그래밍은 다음의 조건을 만족할 때 사용할 수 있다.    
1. 최적 부분 구조 (Optimal Substructure)
    * 큰 문제를 작은 문제로 나눌 수 있으며, 작은 문제의 답을 모아서 큰 문제를 해결할 수 있다.
2. 중복되는 부분 문제 (Overlapping Subproblem)
    * 동일한 작은 문제를 반복적으로 해결해야 한다.

## 피보나치 수열
* 피보나치 수열은 DP로 해결할 수 있는 문제들 중 하나이다.
* 피보나치 수열을 점화식으로 표현하면 다음과 같다.
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbOtyDC%2FbtqJ0ocZjjW%2Fyl4WyWeHzTzXKFdfGRu9jk%2Fimg.png" width="400" height="50">

* 수학적 점화식을 프로그래밍으로 표현하려면 단순 재귀함수를 사용하면 간단하다.
```java
public class Main {
  public static void main(String[] args) {
    System.out.println(fibo(4));
  }
  
  public static int fibo(int x) {
    if(x == 1 || x ==2){
      return 1;
    }
    return fibo(x-1) + fibo(x-2);
  }
}
```
실행 결과
```
3
```
### 피보나치  수열을 단순 재귀함수로 구현했을 때의 문제점
* 위와 같이 코드를 작성하게 되면 심각한 문제가 생길 수 있다.    
* `f(n)` 함수에서 `n`이 커지면 커질수록 수행시간이 기하급수적으로 증가하기 때문이다.  
<img src="https://velog.velcdn.com/images%2Fkimdukbae%2Fpost%2Fd90e2e4d-1a1f-43f2-82e2-edd9f629aeab%2Fimage.png" width="600" height="370">

* `f(6)`을 계산할 때 그림과 같이 `f(2)`가 여러 번 호출되는 것을 확인할 수 있다.
* 즉, 같은 연산을 여러 번 수행하므로 시간 복잡도가 엄청 커지게 된다. 
* 피보나치 수열의 시간 복잡도는 `O(2ᴺ)`이다. 예를 들어 `f(30)`을 계산하기 위해선 약 10억번의 연산을 수행해야 한다.
* 따라서 DP를 사용하면 이러한 문제를 효율적으로 해결할 수 있다.

## DP로 피보나치 수열 구현하기
### 메모이제이션(Memoization)
* DP를 구현하는 방법 중 하나이다.
* 한 번 계산한 결과를 메모리 공간에 메모하는 기법이다.
  * 같은 문제를 다시 호출하면 메모했던 결과를 그대로 가져온다.
  * 값을 기록해 놓는다는 점에서 `캐싱(Caching)` 이라고도 한다.

### 탑다운(Top-Down) vs 바텀업(Bottom-Up)
* `탑다운(Top-Down)`
  * 큰 문제를 해결하기 위해 작은 문제를 호출하는 방식
  * 탑다운(메모이제이션) 방식은 `하향식`이라고도 한다.
  * 점화식을 재귀호출을 통해 구현하므로 코드 가독성이 좋다.
* `바텀업(Botton-Up)`
  * 가장 작은 문제들부터 답을 구해가며 전체 문제의 답을 찾는 방식
  * 바텀업 방식은 `상향식`이라고도 한다.
  * 재귀호출을 하지 않기 때문에 시간과 메모리 사용량을 줄일 수 있다.  

다이나믹 프로그래밍의 전형적인 형태는 바텀업(Bottom-Up) 방식이다.  

### 피보나치 수열 탑다운 방식 구현
* 메모이제이션 기법 사용
```java
public class Main {
  // 한 번 계산된 결과를 메모이제이션(Memoization)하기 위한 배열 초기화
  public static long[] d = new long[100];
  
  public static void main(String[] args) {
    System.out.println(fibo(6));
  }
  
  // 피보나치 함수를 재귀함수로 구현(탑다운 다이나믹 프로그래밍 방식)
  public static long fibo(int x) {
    // 종료 조건: 1 혹은 2일 때 1 반환
    if(x == 1 || x == 2){
      return 1;
    }
    
    // 이미 계산한 적 있는 문제라면 배열에 저장된 값 반환
    if(d[x] != 0){
      return d[x];
    }
    
    // 아직 계산하지 않은 문제라면 점화식에 따라서 피보나치 결과 반환
    d[x] = fibo(x-1) + fibo(x-2);
    return d[x];
  }
}
```
실행 결과
```
8
```
메모이제이션을 사용하여 `f(6)`을 구하는 과정은 아래와 같다.
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbBIN6j%2FbtqJ8fTmL4d%2FSXQvqQkt9XDSH6bRgvZExK%2Fimg.png" width="600" height="370">

이미 계산된 결과를 메모리에 저장하면 다음과 같이 색칠된 노드만 처리할 것을 기대할 수 있다.  
실제로 호출되는 함수에 대해서만 확인해 보면 다음과 같이 방문한다.  
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FM6UZ6%2FbtqJ4HCSbh5%2FQ6aQKN6Yks3dShewkJ1TPK%2Fimg.png" widht="600" height="370">

> f(6) -> f(5) -> f(4) -> f(3) -> f(2) -> f(1) -> f(2) -> f(3) -> f(4)

메모이제이션을 이용하는 경우 피보나치 수열 함수의 시간 복잡도는 `O(N)` 이다.  

### 피보나치 수열 바텀업 방식 구현
```java
public class Main {
  public static void main(String[] args) {
    d[1] = 1;
    d[2] = 1;
    int n = 50;
    
    for(int i = 3; i <= n; i++) {
      d[i] = d[i-1] + d[i-2];
    }
    System.out.println(d[n]);
  }
}
```
재귀호출을 사용하지 않고 단순히 반복문을 이용해 문제를 해결하는 방식이다.  

