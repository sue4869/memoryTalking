# 04.13

### 1. Hashtable vs ConcurrentHashMap

- Hashtable
    - 모든 메소드를 Single Threaded Execution으로 한다.
    - 동기화를 위해 synchronized 키워드를 이용해서 메소드 전체에 락을 검
- ConcurrnetHashMap
    - 내부의 데이터 구조를 분할함으로써 구조적으로 간섭하지 않는 쓰레드 사이에 배타제어가 일어나지 않게 한다.
        
        각각의 bucket별로 동기화를 진행하기에 다른 bucket에 속해 있을 경우, 별도의 lock 없이 운용한다. 
        
        각 Table bucket을 독립적으로 잠그는 방식
        
        ex) put 메소드 
        
         - 빈 Hash Bucket에 노드를 사용할 경우 : lock 사용하지 않고, compare and swap을 사용
            - 💡 여러 쓰레드가 접근하면? : CAS 알고리즘으로 다시 한번 확인하여 원자성 보장을 한다.
            
                즉, bucket 값이 비어있을 경우 volatile 변수에 접근(가시성 보장)하여 기대값(null)과 일치하는지 한 번 더 확인하여 일치할 경우에만 노드를 생성한다(원자성 보장). 이와 같이 동시성을 처리함으로 Thread-safety를 보장한다
            
                CAS 알고리즘 : CAS 알고리즘은 현재 스레드가 가지고 있는 기존값과 메모리가 가지고 있는 값을 비교해 같은 경우 변경할 값을 메모리에 반영하고 true를 반환한다. 
        - 이미 Bucket에 노드가 존재할 경우 : `synchronized` 을 이용해 하나의 thread만 접근하도록 제어한다. 서로 다른 thread가 같은 hash bucket에 접근할 때만 해당 block이 lock된다.
            
            ![Untitled](https://user-images.githubusercontent.com/68679529/231639019-212f8e3a-3bb5-4070-ac04-7071c33e44b2.png)
            
        
    - Hashtable 과는 다르게, 주요 method 에 `synchronized`  키워드가 선언되어 있진 않다.
        
        ex) get메소드
        
        ConcurrentHashMap에서의 get을 살펴보면, `synchroized` keyword를 발견할 수 없다
        

### 2. sort - compareTo

```java
Collections.sort(parking);

static class Parking implements Comparable<Parking> {
        String carNumber;
        String getInTime;
        boolean getOut;
        int totalTime = 0;

        @Override
        public int compareTo(Parking other) {
            int thisCarName = Integer.parseInt(this.carNumber);
            int otherCarName = Integer.parseInt(other.carNumber);

            if(thisCarName - otherCarName < 0) {
                return -1;
            }
            return 1;
        }
    }
```

int값에 의해 **정렬이 진행될 때 자리바꿈(=정렬) 여부를 결정한다.**

만약 **return값이 0이나 음수이면 자리바꿈을 하지 않고**, **양수이면 자리바꿈을 수행**한다.