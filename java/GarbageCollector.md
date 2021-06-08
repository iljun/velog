# Garbage Collector
Java에서 JVM에 남아있는 더 이상 필요하지 않는 객체들을 삭제하는 작업이다.
줄여서 GC라고 부르며, GC가 일어나는 동안은 GC에 관여하지 않는 thread는 작업을 멈추게 된다.

### stop-the-world
GC 실행을 위해서 JVM이 application의 동작을 멈추는 과정을 `stop-the-world`라고 표현한다.
어느 GC이던 stop-the-world의 과정은 존재하고, GC 튜닝은 바로 stop-the-world의 시간을 줄이는 방법이다.

Java는 명시적으로 객체에 할당 된 memory를 할당하지 않는다. 객체를 null로 선언해도 객체에 할당된 memory는 남아있게 되고, 이건 추후 JVM이 GC를 통해 memory를 해제한다.

JVM에서 중요한 역할을 하는 공간이 2가지가 존재한다.
바로 `young 영역`과 `old 영역`이다.
- young 영역
  - 새롭게 생성한 객체들이 대부분 이 영역에 위치하며, young 영역에서 이루어는지 GC를 Minor GC라고 부른다.
- old 영역
  - young 영역에서 Minor GC가 발생했지만, 아직 참조가 남아있어 memory를 해제하지 못한 객체들은 이 old 영역으로 복사된다.
  - 물론 이 old 영역도 GC가 일어나며, old 영역에서 발생하는 GC는 Major GC라고 부르며 이때 stop-the-world가 발생하게 된다.
  
### young 영역의 구조 
young 영역은 `eden 영역`과 2개로 구성된 `Survivor 영역`으로 구성된다.
이 eden 영역과 survivor 영역의 역할은
- 객체가 처음 생성될때 eden 영역에 생성
- eden 영역에서 GC가 일어날때 삭제되지 않는 객체들은 survivor 영역으로 이동
- suvivor 영역이 가득차면, GC가 발생하며 이때 살아남은 객체들이 다른 survivor 영역으로 이동
- 이 과정을 반복하다 계속 살아남은 객체가 존재하면 이 객체는 old 영역으로 넘어가게 된다.(이때의 기존은 JVM 설정에서 횟수를 지정할 수 있는것으로 기억한다.)


### old 영역의 GC
young 영역에서 발생한 Minor GC가 계속 발생하고 old 영역으로 객체가 복사되면, 결국 old 영역에도 GC가 필요하게 된다.

Major GC의 알고리즘은 여러가지가 존재한다.
- Serial GC
- Parallel GC
- Parallel Old GC
- Concurrent Mark & Sweep GC
- G1 GC

#### Serial GC
이 알고리즘은 코어가 1개일 경우에만 사용하기 위해 만든 알고리즘이므로 일반적인 상황이 아니면 사용하면 안되는 알고리즘이다.

이 알고리즘은 old영역의 살아있는 객체를 `Mark`하는 것 부터 시작한다.
이후 heap의 앞 부분부터 살아있는 객체만을 남기고`Sweep`, 객체가 순차적으로 쌓이도록 heap의 가장 앞 부분부터 살아있는 객체들을 채우게 된다`Compaction`

#### Parallel GC
Serial GC와 동일한 알고리즘을 사용하지만 차이점은 Parallel GC는 말 그대로 multi Thread로 GC를 처리한다.

#### Parallel Old GC
Parallel GC와는 old 영역에 대한 알고리즘만 다르며, Mark-Summary-Compaction 과정을 통해 객체를 삭제한다.
Summary와 Sweep의 차이점은 앞에서 실행한 GC 영역에 대해서 별도로 살아있는 객체를 식별한다.
Old 영역에서 일어나는 GC의 처리량을 늘리기 위한 기법이다.

#### CMS GC
`Initial Mark - Concurrent Mark - Remark - Concurrent Sweep 단계로 진행되는 GC이다.`
`Initial Mark`
- class Loader에서 가장 가깝게 위치한 Tree를 1차적으로 탐색해 GC대상인 객체를 탐색한다.
- 이 과정해서 stop-the-world가 발생하지만 Tree를 순회하는건 매우 짧은 시간에 끝난다.
`Concurrent Mark`
- Initial Mark에서 탐색된 GC 대상인 객체들을 참조하는 객체들을 찾아 GC 대상인지 판단한다.
- 이때 stop-the-world는 발생하지 않는다.
`Remark`
- Concurrent Mark에서 찾은 GC 대상의 객체들을 검증하는 단계이다.
- 참조가 제거되었는지, GC 대상으로 추가된 객체들이 있는지 검증한다.
- 이 과정은 stop-the-world를 유발하기 때문에 multi thread로 빠르게 진행된다.
`Concurrent Sweep`
- stop-the-world의 과정없이 remark 단계에서 검증된 객체들을 모두 삭제한다.

여러 단계로 걸쳐 GC가 발생하기 때문에 응답속도가 매우 중요한 application에서 사용하기 적절하다.

이 GC의 단점은 2가지 정도가 있다.
- 다른 GC 방식보다 메모리와 CPU를 더 많이 사용한다.
- Compaction 단계가 기본적으로 제공되지 않는다.
  - Compaction가 기본적으로 제공되지 않기 때문에 Compaction단계가 실행되면 stop-the-world의 시간이 길어진다.
 
#### G1 GC
G1 알고리즘은 JVM 영역을 여러개의 Region으로 분리해 각각의 region을 현 상태에 맞게 각각의 역할(eden, survivor, old)에 맞게 동작한다. 큰 heap의 크기를 가진 환경에서 짧은 GC를 보장하기 위한 기법이다.

eden, survivor, old 영역과는 다른 영역인 Humongous, Available/Unused 영역도 존재한다.
- Humongous
  - Region의 50%가 넘는 큰 객체를 저장하기 위한 공간으로, 이 region에는 GC가 최적으로 동작하지 않는다.
- Available/Unused
  - 사용중이 아닌 Region을 의미한다.
  
G1 GC에서 full GC가 발생할때는 `Initial Mark -> Root Region Scan -> Concurrent Mark -> Remark -> Cleanup -> Copy` 과정을 거친다.

`Initial Mark`
- old 역할을 하는 Region 에 존재하는 객체들이 참조하는 Survivor Region 을 찾는다.
- 이때 stop-the-world가 발생한다.

`Root Region Scan`
- Initial Mark에서 찾은 Survivor Region에 대해서 GC 대상을 탐색한다.

`Concurrent Mark`
- 전체 heap에서 GC 대상을 찾게되며, 이때 GC 대상이 없는 region은 다음 단계에서 제외하게 된다.

`Remark`
- stop-the-world가 발생하며, 최종적으로 GC에서 제외할 객체를 탐색한다.

`Cleanup`
- 살아있는 객체가 가장 적은 region에 대해서 객체 삭제 작업을 진행한다.
- 그리고 비워진 region을 재사용 가능한 상태로 변경한다.

`Copy`
- Cleanup 과정에서 완전히 비워지지 않은 Region의 살아남은 객체들을 Available/Unused Region 에 복사하여 Compaction 작업을 수행한다.