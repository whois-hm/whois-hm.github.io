---
title: Smart Pointer
layout: post
image: "/assets/img"
description: Using Smart Pointer
main-class: pointer
color: "#B31917"
introduction: Using Smart Pointer
---

## #1 Intro

우리는 [여기서](https://github.com/whois-hm/heap-manage)  Virtual Memory를 요청하고 반환받아 User 영역에서 사용되는 Heap Memory라는 개념을 알아봄과 동시에 
항상 요청/해지, 즉 사용을 완료한 Block은 다시 반환하여 다시 가용가능한 Block으로 만들어줘함을 확인하였다.
보통 Block을 생성한 주체가 이에 대한 해지의 책임을 가져가는것이 일반적이지만,  n개의 Thread를 사용하게 되고, 서로 관계를 갖기 시작하면서 Free Point가 애매모호 해지거나, 
경우에 따라 거대한 Block을 운영하는 경우 운영방법 따라 메모리 효율성이 떨어질 가능성이 존재한다.
이번 Post 에선 위에서 말한 

#### 1.Ambiguous Free Point

#### 2.메모리 효율성

에 대한 예를 돌어보면서 이를 해결해 나아가는 방법을 알아가보고자 한다. 

## #2  Occurrence Case


한가지 프로세스를 만든다 가정해보자. 
이 프로세스는 한개의 Pixel Frame입력장치에서 입력을 받아 2개의 출력장치로 PixelFrame을 내보내 동영상을 재생하는 프로그램이다. 
자. 위 모든 과정을 하나의 Thread에서 처리하면 안된다는것을 우리는 [여기](https://github.com/whois-hm/about-workqueue) #1 Thread의 병렬처리 필요성에서 확인하였다. 
그럼 생각해보자 우리는 위의 링그 #1 그림처럼 입력장치에서 Data를 가져오는 Thread하나, 출력장치로 Data를 전달하는 Thread 두개 크게 이렇게 프로세스를 구성하게 될 것이다. 
위 구성은 입력장치 Thread에서 Data Buffer를 생성하여 2개의 Thread로 전달하게 될 것인데, 

바로 여기서 <U><I>Ambiguous Free Point</I></U>  문제가 발생한다. 

Memory DataBlock은 입력Thread에서 생성하고 이 생성된 한개의 Block은 2개의 Thread에서 참조해야하는 문제가 발생한것이다. 
출력을 담당하는 Thread에서 사용이 완료되면 여기서 해지 하면 되지 않을까? 하는 물음이 생기지만 이는 Thread의 Scheduling을 이해하지 못함으로써 발생하는 물음이다. 
Thread는 Cpu의 Scheduling하에 스위칭이 발생하므로 각 출력Thread는 누가먼저 DataBuffer사용을 완료할지 전혀 알수없으며, 만약 어느 한쪽에서 해지를 하게 된다면, 
이를 아직 사용중이던 다른 출력 Thread는 엉뚱한 Memory를 참조하여 Exception을 발생시킬것이다.

그럼 위 문제를 해결하기 위해 다른 방향으로 생각을 해보자.
 1. Ambiguous Free Point 문제는 동일한 주소의 Block을 n개의 Thread에서 참조하여 발생하는 문제이니 각 Thread마다 개별적으로 DataBlock을 생성해서 전달해주면 되겠다!

 2.먼저 커다란 Buffer를 생성해두고 이를 Process가 종료될때까지 재사용하면 되겠다!
 라고 생각될 수 있다. 
 
 물론 위 2가지 방법이 틀린 방법은 아니다. 
 
 하지만 여기서  <U><I>메모리 효율성 문제</I></U> 가 발생한다. 
 
 1번의 경우 잦은 할당과해지로 운영에 비효율성이 커지고 메모리 단편화가 발생하게 되며,  2번의 경우 입력 Data가 가변인경우 처리가 힘들어지거나, 원자적 처리를 위한 
 추가 비용이 발생하게된다.



## #3 Resolution

#2에서 발생한 문제의 원인은 2가지이다. 

동일한 Address  Memory에 n개의 Thread에서 <U><I>안정적인 참조</I></U>,  그리고 참조가 모두 완료되는 순간 할당된 <U><I>Memory Block 반환</I></U> 이 두가지이다. 

전체 구조에서 이를 해결하고자 하면 #2에서 말한것과 같이 추가적인 비용이 발생하거나, 복잡한 구조 설계 방향을 가져갈 수 있다. 

하지만 이를 Memory Block의 입장에서 해결하고자하면 간단하게 처리 될 수 있다. 

Memory Block은 자신이 사용될 주체들의 참조 목록만 알 수 있다면, 먼저 사용을 완료한 주체가 반환요청을 하더라도 다른 모든 주체들이 반환을 요청한 상태가 아니라면, 

참조 목록에서 제외만하는 것이다. 이후 이렇게 제외해 나가다 모든 주체가 반환을 하여 목록에 아무도 남아있지않다면 그때 실제 Memory Block을 반환하는것이다. 

위에서 말한 참조 목록이란 복잡한 네이밍이 아닌 단순 Reference Count값만 으로도 충분하다, MemoryBlock은 어떤일을하는 주체가 자신을 참조할 것인지 알 필요가 없기때문이다. 


>그럼 이 방법을 가지고  Ambiguous Free Point 를 적용해보자. 
>Memory Block은 Reference Count값을 가지고 있기 때문에, 어느 누가 먼저 완료했다고 반환을 요청하더라도 문제되지 않는다. Thread Scheduling과는 상관없이 결국 마지막에 반환을 요청하는 
>주체에 맞춰 실제 Block을 반환할 것이기 때문에...


>다음 메모리 효율성에 적용해보자. 
>- 우리는 개별Thread에 각자 Block을 생성하여 전달하지 않고 다만 Reference Count만 증가 시킨다.
>- 커다란 Buffer를 미리생성해두는 것이 아닌 필요한 순간에 할당한다. 
>물론 이 해결방법도 원자적 처리를 위한 추가 비용이 발생하는것은 피할 수 없다. 
>다만 프로세스의 Core에서 이를 처리하지 않고 외부에서 이를 붙여나가기 시작한다면, 경우에 따라 서로 종속성을 갖는 얽히고 얽힌 코드가 될 가능성이 크다.

## #4 How to use
``` c++
/*Smart Pointer 사용을 위한 Global 초기화*/
SP_using()
	
/*block 요청 */
void *sp_block = SP_malloc(10, //생성 Block크기 
1 //보통 1값으로 최초 생성자의 참조값만 생성하지만, 필요에 의해 참조값을 미리 설정할수 있다.
);

/*위에서 생성한 'sp_block'에 참조 카운트를 증가시킨다. (현재 Count = 2)*/
void *sp_ref_block = SP_ref(sp_block);

/*참조 카운트 감소 (현재 Count = 1) 생성된 Memory는 해제되지 않는다.*/
SP_unref(sp_blcok);

/*참조 카운트 감소 (현재 Count = 0) 생성된 Memory 해제.*/
SP_unref(sp_ref_block);

/*Smart Pointer 사용 종료*/
SP_unused();
```


## #5As I Finished…
이 Smart Pointer는 c++ stl에 적용된 shared_ptr기술이다. 
이 기술을 따로 개발하여 보고 Posting하는 이유는 c++ 컴파일러 버전에서 지원하지 않는경우를 대비하기 위해서이며, 
직접 개발해봄으로써 원리를 파악하고 Pointer라는 개념을 더 친숙하게 다뤄보기 위함이며, 
문제 해결에 대한 처리 능력을 향상시키기 위함이다. 
이 기능을 적용함으로써 생성한 Memory Block에 대해 더 유연해지고, 효율성이 높아져 활용 분야가 많을것으로 생각된다.
