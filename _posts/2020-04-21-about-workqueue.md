---
title: About  Workqueue
layout: post
image: "/assets/img/"
description: Description Repository 'workqueue'
main-class: wq
color: "#B31917"
introduction: Description Repository 'workqueue'
---

## #1 Thread
Process를 설계하다보면, 실시간처리, Blocking I/O처리,  단일Context등의 관리를 위해 
Thread를 사용한 병렬처리를 요구하는 경우가 발생한다. 

>  Thread가 요구되는 간단한 예

![audiofile_speaker](../images/audiofile_speaker.png)

프로그램이 구동되면 하나의 오디오 파일에서 2개의 출력 장치로 
Sound를 출력하는 아래의 예시 코드를 보자
  
{% highlight c++ %}
    void main()
    {
     open_file();      
     while(!end_file())
     {
      unsigned char *data = read_file();
      write_audio1(data);
      write_audio2(data);
     }
    }
{% endhighlight %}

진입점에서 오디오파일을 열고 파일의 데이터를 전부 읽을때까지 while루프 안에서 
1번, 2번 audio 출력장치로 data를 내보내는 코드이다.

### 여기서 발생할수 있는 문제점은 무엇일까?

문제 발생지점은 write_audio1(data);, write_audio2(data); 부분이다. 
항상 1번 출력을 완료한후 2번을 출력하고, 다시 2번 출력을 완료한후 1번출력을 수행하기 때문에
개별 출력 관점에서 볼때 음단절이 발생할 확률이 크다.
예시에서는 단2개의 출력이지만 n개의, 즉 출력 장치가 많아 질수록 위 문제점은 더 부각될것이다.

### 해결 방법은?

write_audio 수행을 병렬로 처리하는것이다. 
각자의 출력장치가 다른 출력장치의 결과가 반환되기를 기다린후 동작하는 것이 아닌 
각자의 Thread에서 개별적으로 출력하는것이다.

> *(Thead로  분리한다해도 단일 코어등 Spec이 낮은 하드웨어에선 동일한 결과가 발생할것이라 생각될수 있지만,
> Cpu는 데이터를 Bus를 통해 장치로 내보내는 I/O작업 과정에서 또는 기타I/O등에서 Context Switching을 발생시키기 때문에 
> 단일 Thread에서 작업되는 내용보다 더욱 효율적이다.)*

 '각자의 Thread에서 개별적으로 출력하는것'은 여러 구조설계방안과 단순 Thread처리가 가능하지만 
여기서는 #2의 Workqueue를 기반한 구조설계방안으로 접근할것이다. 
먼저 #2 정의를 살펴보고 #3에서 실제 적용예시를 살펴보자






## #2 Workqueue

Workqueue는 Event 송/수신을 통해 기존의 Function Call에 기반한 프로그래밍이 아닌
Event 기반의 Thread프로그래밍을 가능하게한다. 
생성한 Thread를 Event를 요청하는 '요청자'와 수신된 Event를 처리하는 '수신자'로 분리하며,
![wq_1](../images/wq_1.png)  
<center>(요청자임과 동시에 수신자가 될수있으며, 반대의 경우도 동일하다)</center>
  <br>
	<br>
Naming에서도 알 수 있듯이 Queue에 기반하여 Event를 송/수신하고,  때문에 별도의 CriticalSection구역을 
지정 하지 않아도 원자적인 Data 처리가 가능하다.
![wq_5](../images/wq_5.png)                          

                          
요청자는 Blocking Mode 또는 None Blocking Mode로 수신자에게 Event 전달 할 수 있기 때문에
단순 Event전달 요청으로 수행을 완료할수도 필요에 따라 수신 Thread의 Event수행결과를 반환받고 
수행을 완료할 수도있다. 
![wq_2](../images/wq_2.png) 
<center>(None Block Mode)</center>
![wq_3](../images/wq_3.png) 
<center>(Block Mode)</center>
  <br>
	<br>
수신자는 자신의 EventQueue를 감시하며 외부 요청에의한 Event Trigger시 해당 요청을 수행한다. 
EventQueue를 감시함에 있어서 Cpu자원을 소비하지 않기 때문에 다른 Thread의 동작에 영향을 주지 않고
Sleep하는것이 가능하고 Timeout을 설정할수 있기때문에 자신만의 Scheduling을 가져갈수있다. 
![wq_4](../images/wq_4.png) 
<br>
<br>
> 간단히 요약하면 Thread간의 Communication 도구 이다.




## #3 Apply to
이제 #1의 개선방안을 #2를 기반으로 구조설계를 변경해 보자.
먼저의생각해볼점은
<br>
1.  Thread 구성?
2.  Thread Stack 생성된 출력장치의 Meta data관리?
3.  파일Instance를 2개의 Thread에서 접근한다?
4.  Thread BusyLock?
5. 출력장치와의 Interface?


### Workqueue를 기반하면 위의 생각해볼 사항들을 다음과 같이 정리할 수 있다.

> Thread 구성?

#2에서 Workqueue는 생성된 Thread를 Event 수신자로서 동작하는것이 가능하다고 하였다. 
이는 자신이 파일에서 직접 Data를 읽지 않고 외부로 부터 Event로 Data를 수신할 수 있다는 말이된다. 
그렇기에 각 출력장치에 대응하는 1:1의 Thread를 생성한다.
![match](../images/match.png) 

>  Thread Stack 생성된 출력장치의 Meta data관리?

우리가 프로그래밍을 하다보면 이론적으로 '캡슐화'라는 용어를 접하게 된다. 
"'캡슐화'는 필요한 속성 또는 행위를 하나로 묶고 외부에서 사용하지 못하도록 은닉한다. "
로 의미를 해석할 수 있다. 
캡슐화를 하지 않고 전역적으로 사용되는 속성들이 늘어나면 Module의 개념을 잃어버리게 되고,
 필요한 동작과 전혀 관계없는 코드에서도 모두 참조가 가능하므로, 뒤죽박죽 섞인, 나아가선 전혀 관리 할 수 없는 코드가 될것이다.
 또한 System의 Static한 Resource들은 자신만의 속성( Meta data)을 가지고 있기 때문에 여러 Thread또는 전역적으로 참조가 가능해지면,
 System에 예기치 못한 상황을 발생 시킬수 있다. 
 
 그럼 이제 위의 캡슐화 내용을 충족시키기 위한 방법으로 Thread의 Stack에 Meta Data를 생성하자. 
 Thread Stack에 생성된 Meta Data는 강제로  Pointer로 엮지 않는이상 외부로 노출될 수 없고, 해당 Thread 내에서 모든 처리가 가능해지기 때문에 
 Module의 의미를 가질 수 있다. 
 
 "그럼 다른 곳에서 Meta Data가 필요해 지면 어떻게 할 것인가?" 이와 같은 질문이 생길수 있다. 
 Meta Data는 그 Resource에 대한 동작을 수행하기위한 내용들이기 때문에 수행해줄 어떠한 Module만 있다면 
 내가 그 Meta Data를 직접 참조하지 않아도 되며, 단지 요청만 하면되는 것이다.
 ![request](../images/request.png)
 
 
> 파일Instance를 2개의 Thread에서 접근한다?

file 또한 System에 존재하는 유일한 Resource로 볼 수 있다. 
위에서 이미 유일한 Resource에 대한 접근 방법을 설명하였다. 
그렇다면 파일에 대한 Module(Thread) 를 생성해야 하는 것인가?
물론 별도로 만들어도 큰 문제는 발생하지 않는다. 
다만 우리에겐 이미 main thread가 생성되어 있으므로 재생 요청자를 main thread로 설정하자
![main_request](../images/main_request.png)


> Thread BusyLock?

#2에서 "수신자는 EventQueue를 감시함에 있어서 Cpu자원을 소비하지 않는다."
처럼 Workqueue는 다른 Thread의 동작에 영향을 주지 않는다.


> 출력장치와의 Interface?

지금까진 MainThread에서 출력 Thread로 재생을요청하는 Interface만 존재한다. 
출력장치라는건 Output에 대한 Spec이 변경될 수도 있고 때론 System의 문제로 출력을 할 수 없는 상황 이 발생할 수 있다. 
한 예로  더이상 출력하지 못한 상황이 발생한다면, 더이상 MainThread로 부터 재생을 요청하는 Event를 수신 하지 않아도 될것이다. 
그렇다면 어떻게 Event를 수신 하지 않도록 변경할 수 있을까?

#2에서 Workqueue는 요청자임과 동시에 수신자가 될 수 있음을 설명하였다. 
지금까지 수신자 역할만 해오던 출력장치Thread에 요청자 기능을 추가하고 요청기능만 수행하던 MainThread에 수신기능을 추가하면
위 내용을 모두 만족한다.
![com](../images/com.png)
