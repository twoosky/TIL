# Synchronized
* synchronized는 lock을 사용해 동기화를 시킨다.
* synchronized는 4가지의 사용법이 있다.
* sychronized method, sychronized block, static sychronized method, static synchonized block.

## 1. synchronized method
* synchronized method는 클래스의 인스턴스에 대하여 lock을 건다.
```java
public class Main {
  public static void main(String[] args) {
    A a = new A();
    Thread thread1 = new Thread(() -> {
      a.run("thread1");
    };
    Thread thread2 = new Thread(() -> {
      a.run("thread2");
    };

    thread1.start();
    thread2.start();
  }
}
```
