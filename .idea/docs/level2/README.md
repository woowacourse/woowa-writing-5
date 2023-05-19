# GC란 무엇인가?

---
- C, C++에서는 코드 레벨에서 메모리를 할당 받고, 해제해야 했다
    - 자칫 실수하면 Memory Leak이 발생될 수 있다
    - 이렇게 수동으로 메모리를 관리하는 건 어려운 일이다

- 하지만 java에선 이를 GC가 해주면서 코드레벨에서 해방되었다
- GC라는 친구가 Heap 메모리에서, root set에서 unreachable한 객체를 삭제 시켜줌


# GC가 필요한 이유?

---

### 장점

- Memory Leak이 발생되지 않음
- 휴먼 에러 발생 가능성 낮춤
    - 대표적인 휴먼 에러로는
        - 해제된 memory에 접근 시도
        - memory 이중 해제가 있다

### 단점

- 성능 저하
    - 어떤 메모리를 해제해야할지 검사하고 삭제하는 이 과정 또한 결국 CPU자원과 메모리를 필요로 한다
    - 대규모 데이터가있을 수록 비용은 더 증가
- 개발자는 언제 메모리가 해제되는지 모른다
    - jvm은 GC를 실행시키기 위해 잠시 applicaiton 실행을 멈춘다
    - 실시간성이 매우 강조된다면, 이런 특징이 적합하지 않을 수 있다

## jvm의 GC algorithms: Mark and Sweep

---
- root set으로부터 Heap 영역의 모든 객체를 스캔 → Mark
- root set에서 unreachable한 객체를 Heap 영역에서 제거 → Sweep
- Heap이 바로 GC에 의해 관리되는 영역이기 때문에, Heap 영역에서 mark and sweep이 이루어진다

## Mark and Sweep의 두 가지 특징

---

- **의도적으로 GC를 실행시켜야한다**
    - jvm에겐 “이쯤 되면 GC 실행시켜야지”라는 나름의 기준이 존재
    - 이 기준을 알기 위해선 jvm의 Heap 영역을 들여다봐야한다
- **application 실행과 GC 실행이 병행된다**

## Mark and Sweep 특징1: 의도적으로 GC를 실행시켜야한다

### Heap

- young generation, old generation으로 나뉨
- 이 generation마다 GC도 나뉜다
- young generation: 새로운 객체들이 할당
- old generation: young generation에서 오랫동안 살아남은 객체들이 존재하는 영역

## Q. 왜 이렇게 둘로 나눠요?

- 할당된 객체는 오랫동안 참조되지 않는 게 대부분이다
    - 즉, 금방 garbage 상태가 된다
- 하지만, 오래된 객체에서 젊은 객체로의 참조는 거의 없다

### Heap이 하나라면 오래된 객체까지 스캔해서 비효율적. 차라리 나눠서, 오래된 객체는 따로 빼두고!! 할당되지 얼마 안 된 객체들만 주기적으로 스캔하는 게 훨씬 효율적이며 비용을 줄일 수 있다

### young generation

- 여기서 발생하는 GC는 minor GC
- 3영역으로 나뉨
    - Eden
    - survival 0
    - survival 1
- 여기서 eden에 객체가 꽉 차면 minor GC 실행됨
    - mark and sweep이 진행됨
    - root로 부터 reachable이라 판단된 객체는 survival 0영역으로 옮겨짐
        - 이 때, 옮겨지면서 age-bit가 1 올라간다
        - age-bit가 15가 되면, 오래 쓰이는 친구라고 인식하고 old generation으로 옮겨진다
- eden이 또 꽉 차면 minor Gc가 실행되겠지?
    - reachable이라고 판단된 객체들이 survival 1로 이동하며 age-bit 1증가!
    - 이 때, survival 0에 있던 친구들도 survival 1로 이동하며 age-bit 1 또 증가!
- 또 꽉 차면 여기 있는 친구들이 또 survival 1로 이동된다.
- 즉, minor GC가 실행될 때 마다 survival 0, 1을 왔다 갔다하며 age-bit가 증가되는 구조임

### old generation

- 여기서 발생하는 GC는 major GC
    - minor GC보다 더 오래 걸린다
- old generation이 꽉 차면 major GC 실행됨
- 언젠간 old generation도 꽉 차는 날이 오겠지? 그렇게 되면 major GC 발생
    - mark and sweep 방식을 통해 필요 없는 메모리를 비워준다


## mark and sweep 특징2: application 실행과 GC실행이 병행된다

### JVM은 어떻게 GC와 application을 같이 실행할까?

- 사실, 완전히 같이 실행되는 건 아니다
- JVM은 GC를 실행하기 위해 **Application 실행을 멈춘다  = stop the world**
- 이 stop the world시간이 짧을 수록 최적화 된 것
- 여러 방식이 있는데 자주 쓰이는 것만 소개를 해보겠다

### parallel GC

- java 8에서 기본적으로 쓰이는 GC 방식
- multi core 환경에서 사용
- 여러개의 thread로 GC를 실행하기 때문에 stop the world 시간이 짧다

### G1 GC

- java 9부터 기본적으로 쓰이는 GC방식
- heap을 일정 크기의 region으로 잘게 나눠서 young generation, old generation으로 나눠서 활용
- 이 때 region은 그 때 그 때 개수를 알아서 튜닝해준다
- 이에 따라 stop the world 최소화 가능

# Summary

### GC

- garbage찾아서 memory 해제시켜줌

### Garbage 구분법

- Root set과 unreachable한 객체라면 garbage로 보고 garbage 취급

### GC 동작 방법

- mark and sweep을 이용해 Unreachable 객체 삭제함

### GC 발생 과정

- young generation 가득 차면 minor GC 발생
- old generation 가득 차면 major GC 발생
