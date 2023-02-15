# Garbage Collector 종류
### 1. Serial GC
* GC를 처리하는 스레드가 1개이다.
* CPU 코어가 1개만 있을 때 사용하는 방식
* Mark-Compact Collection 알고리즘 사용
* Compact: Sweep 과정에서 발생하는 메모리 단편화를 방지하기 위해 흩어져있는 메모리를 왼쪽으로 몰아넣는 작업

### 2. Parallel GC
* GC를 처리하는 스레드가 여러개 있다.
* Serial GC보다 빠르게 객체를 처리할 수 있다.

### 3. Concurrent Mark Sweep GC (CMS GC)
* StopTheWorld에 의해 JavaApplication이 멈추는 현상을 줄이고자 만든 GC
* stop-the-world가 발생하면 GC를 실행하는 스레드를 제외한 나머지 스레드는 모두 작업을 멈춘다.
* Compaction 단계가 제공되지 않는다.
* 객체 마킹 작업을 한번에 하는 것이 아닌, 4번의 과정으로 나눠 마킹한다.
<img src="https://user-images.githubusercontent.com/50009240/219018153-4f88fc27-d3e7-41b2-ad8a-f97913e8abfa.png" width="500" height="380">

1. InitialMark : GCRoot가 참조하는 객체만 Mark
2. ConcurrentMark: 참조하는 객체를 따라가면서 지속적인 Mark
3. Remark: ConcurrentMark과정에서 변경된점이 없는지 다시 체크
4. ConcurrentSweep: StopTheWorld 없이 접근 할수 없는 객체를 제거 StopTheWorld를 최대한 줄이고자 함

### 4. G1 GC
* Heap을 동일한 크기의 Region으로 나눠 메모리 관리
* Region 단위로 탐색하고, 각각의 Region에서 GC가 발생한다.
* 빠른 처리 속도를 지원하면서 stop-the-world를 최소화한다. 
* CMS GC보다 효율적으로 동시에 Application과 GC를 진행할 수 있다.
* 메모리 Compaction 과정을 지원한다.
* Java9+의 Default GC
<br></br>

**Reference**  
* https://steady-coding.tistory.com/590  
* https://itkjspo56.tistory.com/285
