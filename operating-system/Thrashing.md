# 스레싱 Thrashing

* 프로세스가 작업 중일 때 그 프로세스의 전체 데이터가 모두 물리 메모리에 할당되는 것은 비효율적 
* 작업에 필요한 일부분만 물리 메모리에 적재하고, 현재 필요하지 않은 부분은 스왑영역에 올려 메모리를 효율적으로 관리한다. 
* 만약 프로세스가 필요로 하는 데이터가 물리 메모리에 없다면 스와핑 과정을 거친다.
* `swap in` 필요한 데이터를 스왑 영역에서 물리 데이터로 가져오기
* `swap out` 물리 메모리에 있던 데이터를 스왑 영역으로 보내기
* 스와핑 작업은 디스크 I/O 작업으로 CPU를 사용하지 않는 작업이다. 
* 때문에 이 작업이 많아질수록 CPU가 일을 하는 시간은 감소한다. 


이때 CPU를 점유하고 있는 프로세스가 필요로 하는 데이터가 물리 메모리에 없는 경우를 **페이지 부재**(Page Fault)라고 한다.

메모리에 동시에 올라가 있는 프로세스의 수를 다중 프로그램 정도(**MPD**: Multi-Programming Degree)라고 한다.  운영체제는
CPU 이용률이 낮으면 MPD를 높인다. 하지만 과도하게 MPD가 높이지면 프로세스에게 할당되는 메모리 양이 감소하게 되고
결국 페이지 부재가 발생하여 프로세스 수행시간 보다 페이지 교체 시간이 많은 상태가 된다.

즉 스레싱이란 페이지 부재율이 증가하여 CPU 이용율이 급격히 떨어지는 현상이다. 

<br>

## 프레임 할당

스레싱은 각 프로세스에 프레임을 할당하는 정도와 연관되어 있다. 감당할 수 있을 정도의 페이지 부재를
일으키면서 전체 프로세스가 원할하게 돌아가게 하기 위해서 두가지의 프레임 할당 방법이 존재한다.  

### Local Replacement

* 프로세스 별로 할당된 프레임의 크기가 고정되어 있어, 페이지 부재가 발생하면 프로세스 안에서 페이지 교체 정책을 통해
자체적으로 메모리를 확보
* 메모리 상의 자기 자신의 프로세스 페이지에 대해서만 교체 작업을 수행

### Global Replacement

* 프로세스의 우선 순위를 따져 각 프로세스에게 동적으로 우선 순위 할당을 진행한다.
* 페이지 부재가 발생하면, 다른 프로세스가 가지고 있던 프레임을 가져와 사용할 수 있다.
* 메모리 상의 모든 프로세스 페이지에 대한 교체 작업을 수행

<br>

## 스레싱 해결 방안


### 워킹셋 알고리즘(Working Set)

* [지역성](https://github.com/jmxx219/CS-Study/blob/main/OperatingSystem/%EC%BA%90%EC%8B%9C%20%EB%A9%94%EB%AA%A8%EB%A6%AC.md#%EC%BA%90%EC%8B%9C-%EC%A7%80%EC%97%AD%EC%84%B1)을 활용하여 
프로세스가 많이 참조하는 페이지 집합을 메모리 공간에 계속 상주시켜 빈번한 페이지 교체현상을 줄이는 방법
* 즉 워킹셋 알고리즘은 지역성 집합이 메모리에 동시에 올라갈 수 있도록 보장하는 메모리 관리 알고리즘
* 워킹셋 윈도우란 페이지를 참조하는 횟수를 의미한다
  * ex) 10,000번의 명령어 수행을 워킹셋 윈도우로 정한다는 말은 현재 시점 T로부터 이전 10,000개의 명령어에 속한 녀석들이 워킹셋이 된다는 의미이다. 
* 시간이 지남에 따라 자주 참조하는 페이지들의 집합은 변할 수 있기 때문에 워킹셋도 시간에 따라 변한다.

<br>

### 페이지 부재 빈도 알고리즘(Page Fault Frequency)

* 프로세스의 페이지 부재율을 주기적으로 조사하고 이 값에 근거하여 각 프로세스에 할당할 프레임 양을 동적으로 조절하는 알고리즘
* 시스템이 미리 정해 놓은 상한값(Upper bound)을 넘어가거나 하한값(Lower bound)이하로 떨어지게 되면 운영체제가 메모리에 올라가있는 
프로세스의 수를 조절한다.
  * 페이지 부재 비율이 상한선을 초과하면 할당한 프레임이 적다는 것이므로 프레임 할당량을 늘려준다
  * 반대로 페이지 부재 비율이 하한선보다 낮다면 할당한 프레임을 어느정도 회수한다. 

<br>

<img alt="PFF" src="https://github.com/reddevilmidzy/CS-Study/assets/78539407/92c5b6c5-7619-4f00-abf5-86d2d56e8b23" height="350"/>


<br>
<br>

## 페이지 크기

현재 페이지의 일반적인 크기는 4KB ~ 4MB이다. 크기는 점점 커왔고, 현재도 계속 증가하고 있다.  

페이지 크기를 설정하는데에는 내부 단편화와 페이지 부재에 대한 `trade-off`가 있다.

> **단편화(Fragmentation)란**  
> 메모리 공간이 부분으로 나뉘어서 충분히 사용 가능한 메모리가 남아있지만, 프로세스 할당이 불가능한 상태
> 
> **내부 단편화(Internal Fragmentation)란**  
> 메모리를 할당할 때 프로세스가 필요한 양보다 더 큰 메모리가 할당되어서 메모리 공간이 낭비되는 상황
> 
> **외부 단편화(External Fragmentation)란**  
> 메모리가 할당되고 해제되는 작업이 반복될 때 작은 메모리 중간중간에 사용하지 않는 메모리가 많이 존재하여 총 
> 메모리 공간은 충분하지만 실제로 할당 할 수 없는 상황

<br>

### 페이지 크기를 작게 설정시

* 장점
  * 내부 단편화(Internal Fragmentation)가 줄어든다.
  * 메모리 사용의 효율성을 높일 수 있다.
* 단점
  * 개별 페이지에 대한 페이지 테이블을 들고 다니는 데 많은 메모리가 필요할 수 있다.
  * 잦은 페이지 부재로 인한 오버헤드가 커질 수 있다.

### 페이지 크기를 크게 설정시

* 장점
  * 필요한 페이지 테이블의 수는 작다
  * 페이지 부재를 핸들링하는데 드는 오버헤드가 적다.
* 단점
  * 내부 단편화가 크다.

<br>

#### ref

[운영체제 KOCW 양희재 교수님](http://www.kocw.net/home/search/kemView.do?kemId=978503)  
[[운영체제/OS] 프레임 할당(Frame allocation)과 Page fault 관련 기타 이슈들](https://studyandwrite.tistory.com/23)  
[[시나공 정보처리기사] 2415506 스래싱](https://youtu.be/6AfFVJraKZA?si=jZVo5SuRozpuH1bO)  
[스레싱 (Thrashing)](https://blog.skby.net/%EC%8A%A4%EB%A0%88%EC%8B%B1-thrashing/)  
[스레싱(thrashing)이란 무엇인가](https://straw961030.tistory.com/155)