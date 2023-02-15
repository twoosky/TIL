# Garbage Collector
* JVM의 Heap 영역에서 사용하지 않는 동적할당 메모리 영역을 주기적으로 삭제하는 프로세스  
* C/C++ 에서는 GC가 없어 수동으로 메모리 할당과 해제를 해줘야 했지만, Java는 JVM에 탑재되어 있는 GC가 메모리 관리를 해준다.

## 1. Garbage Collector 대상
* 객체에 레퍼런스가 있다면 Reachable로 구분되고, 객체에 유효한 레퍼런스가 없다면 Unreachable로 구분해 제거한다.
* Reachable: 객체가 참조되고 있는 상태
* Unreachable: 객체가 참조되고 있지 않은 상태 (GC의 대상이 됨)
<img src="https://user-images.githubusercontent.com/50009240/218995174-71cfd0bd-41d6-4076-95f5-dd688c92efb2.png" width="540" height="240">

* 객체들은 실질적으로 Heap 영역에서 생성되고, Method 영역, Stack 영역 등에서는 Heap 영역 객체의 주소만 참조한다.
* 메서드 종료와 같은 특정 이벤트로 인해 Heap 영역 객체의 메모리 주소를 갖는 참조 변수가 삭제되면 
* 위 그림의 빨간색 객체와 같이 어디서도 참조되고 있지 않은 객체들이 발생하게 된다. 
* 이러한 객체들을 Unreachable하다고 하며, Garbage Collector의 대상이된다.

## 2. Heap에 있는 객체에 대한 참조 종류
1. Heap 내의 다른 객체에 의한 참조
2. Stack 영역의 참조 변수에 의한 참조
3. Native Stack 영역, 즉 JNI(Java Native Interface)에 의해 생성된 객체에 대한 참조
4. Method 영역의 static 변수에 의한 참조

* 이들 중 Heap 내의 다른 객체에 의한 참조를 제외한 나머지 3개가 Root Set이다.
* Root Set을 통해 Reachable 객체를 판단한다.

## 3. GC 동작 원리: Mark And Sweep 
1. Mark 과정: Root Set부터 참조하는 객체를 스캔해 각각 Heap영역의 해당 객체(Reachable Object)를 마킹한다.
2. Mark 과정: Reachable Object가 참조하고 있는 객체도 찾아서 마킹한다.
3. Sweep 과정: 마킹되지 않은 객체 즉, Unreachable 객체들을 Heap에서 제거한다.

## 3. Garbage Collector 동작 과정
**1. 새로운 객체는 Eden 영역에 할당된다.**   
* age-bit는 0으로 할당된다. age-bit는 Minor GC에서 살아남을 때마다 1씩 증가한다.  
<img src="https://user-images.githubusercontent.com/50009240/218998345-3e4c1022-032c-4590-8e97-799f72f0e136.png" width="600" height="200">

**2. Eden 영역이 꽉차면 Minor GC가 발생하고, Survival 0 영역으로 옮겨진다.**  
* Eden 영역에 대해 Mark And Sweep 과정이 발생한다.  
* Eden 영역의 Reachable 객체는 Survival 0으로 옮겨지고, Unreachable 객체는 메모리에서 해제된다.  
* Survival 0으로 옮겨진 객체의 age-bit은 1 증가한다.  
<img src="https://user-images.githubusercontent.com/50009240/218998953-09972164-d956-4905-8a82-2a3f42519c81.png" width="600" height="200">  
<img src="https://user-images.githubusercontent.com/50009240/219007596-68d64a5f-2d7a-4fb8-857b-a65caca35a05.png" width="600" height="200">

**3. Survival 0 영역이 꽉차면 Minor GC가 발생하고, Survival 1 영역으로 옮겨진다.**   
* 1번, 2번 과정을 반복하다 Survival 0 영역이 꽉찬 경우, Survival 0 영역에 대해 Mar And Sweep 과정이 발생한다.  
* Survival 1으로 옮겨진 객체의 age-bit은 1 증가한다.
* 이후 Eden 영역의 Reachable 객체는 Survival 1로 옮겨진다.
<img src="https://user-images.githubusercontent.com/50009240/219008402-27c1601c-7783-418e-a0c8-7a3c5c79dd19.png" width="600" height="200">
<img src="https://user-images.githubusercontent.com/50009240/219008488-1eb2b211-5f0d-4258-ab13-1869d66aa7d1.png" width="600" height="200">

**4. Survival 1 영역이 꽉차면 Minor GC가 발생하고, Survival 0 영역으로 옮겨진다.**
* Survival 두 영역 중 한 곳은 항상 비워둔다. 
* 둘 중 한 곳이 꽉차면 다른 Survival 영역으로 옮기는 방식으로 진행된다.
* Survival 0으로 옮겨진 객체의 age-bit은 1 증가한다.
<img src="https://user-images.githubusercontent.com/50009240/219009250-af059e9c-8725-4bbf-a957-2da917571c5f.png" width="600" height="200">
<img src="https://user-images.githubusercontent.com/50009240/219009984-c3fe4e54-2c75-49a0-b7b0-0fff837dadd0.png" width="600" height="200">

**5. 위 과정을 반복하다, age bit가 특정 값 이상이 되면, Old Generation 영역으로 옮겨진다. (Promotion 과정)**    




**6. Old Generation 영역이 꽉차면 Major GC가 실행된다.**
* Mark And Sweep 방식을 통해 Old Generation 영역의 메모리를 회수하는 것을 Major GC라고 한다.
* Minor GC보다 Major GC가 stop-the-wrold 현상이 더 길다.
<img src="https://user-images.githubusercontent.com/50009240/219011554-d7e2b3fb-bde4-43a7-a627-e3f5059dd477.png" width="600" height="200">



## 4. Garbage Collector 한계
1. GC의 메모리 해제 타이밍을 개발자가 정확히 알기 어렵다.
2. GC가 동작하는 동안 다른 스레드는 작업을 멈추기 때문에 오버헤드가 발생한다. (stop-the-world)



즉, 어느 순간에는 실행 중인 애플리케이션이 GC에게 컴퓨터 리소스를 내어 주어야 한다
