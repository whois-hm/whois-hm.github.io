---
title: Heap Manage
layout: post
image: "/assets/img"
description: Managing to Heap Memory
main-class: Heap
color: "#B31917"
introduction: Managing to Heap Memory
---

## #1 Intro
우리가 만든 프로그램은 구동시 또는 RunTime에  Os에 의한 자원할당,
필요한 자원을 요청하면서 작업을 수행한다. 

이번 Post에선 이러한  Os에서 제공되는 Memory영역의 OS / Stack / Data / Heap중
"Heap 영역"에 관한 내용을 정리하면서,  사용에 있어 발생할 수 있는 Exception 과 
해결하는 방안도 함께 알아보고자 한다.

<table width = "300"  height = "100">
	<tr align = "center" >
		<td>Os</td>
	</tr>
<tr align = "center" >
		<td>Data</td>
	</tr>
	<tr align = "center" >
		<td>Stack</td>
	</tr>
	<tr align = "center" bgcolor="#41AF39" >
		<td>Heap</td>
	</tr>
</table>

## #2 Heap


Heap Memory는 필요에 의해 "동적할당"이 이뤄지는 영역이다. 

다시말해 Runtime시 특정자료를 보관하기 위한 "사용자 Memory"이며,
Os는 각 Process마다 특정 크기의 Heap Memory를 Virtual Memory형태로 제공하기 때문에
우리는 필요에 따라 원하는 크기만큼의 Memory Block을 Heap에서 가져와 사용하는것이다.

``` c++
  /*Memory Block을 요청하기*/
  void *memory_block = malloc(size);
```


> Virtual Memory<br>
> Memory라함은 무한한 크기를 갖는 공간이 아닌,  물리적 System에 종속되는 한정된 크기를 갖는 공간이다. 
> 그럼 어떻게 우리는 저 한정된 크기의 Memory를 여러 Process에서 보다 더 많은 자원으로 사용이 가능해질까, 
> Os에서는 이에 대한 운영방법으로 물리Memory와 가상Memory를 분리하고, 이 가상Memory를 Process에게 할당해주며
> 여기서 사용되는 가상Memory를 물리Memory와 적절히 Mapping한다

> Stack? Heap?<br>
>위에서 말한대로 Process는 Os에게서 할당받은 영역이 Heap도 존재하지만 다른성격의 Stack영역도 함께 제공받는다.
>이 둘의 가장 큰 차이점은 Stack은 Os가 관리하는 영역이고 Heap은 사용자가 관리하는 영역이라는 점이다. 
>우리가 생성해내는 local변수, Function Return Address, Function인자등은 사용자가 직접 할당과 해지를 관리하지 않는 
	>Last In First Out구조의 영역이고 Heap은 우리가 필요한경우 할당과 이에 따른 해지를 직접 관리해야 하는 영역이다.
	
	
## #3Exception 

#### 사용과 반환


Heap Memory는 "사용자 Memory"라고 했다. 
이 의미는 Os가 관리 권한을 사용자에게 제공하는것으로도 볼 수 있기 때문에, 사용하는 대에는 일정 규칙이 존재하며
이 규칙을 지켜야 안정적인 프로그램을 운영할 수 있다. 

여기서 규칙은 복잡하고 어려운 것이 아니라, 사용과 반환을 명확히 하라는 것이다. 
Heap에서 Block을 요구만 한다면 아무리 Virtual Memory라 해도 이 크기는 제한적일 수 밖에 없기 때문에 
사용을 완료한 Block은 반환하여, 다음번 재활용이 가능하게 만들어 줘야 한다. 

``` c++
  /*Memory Block을 요청하기*/
  void *memory_block = malloc(size);
  /*생성된 Block사용*/
  block_use(memory_block);
  /*사용이 완료된 Block 반환*/
  free(memory_block);
  memory_block = NULL;
```

이를 지키지 않거나 혹은 실수로 반환하는 과정을 누락했다면, Process에 Memory Leak이 발생하고 Os는 
Process에 더이상 사용가능한 Memory가 없음을 "Out of Memory"로 출력하고 다운시킨다. 

#### 잘못된 사용
(이 내용은 Heap과 마찬가지로 Stack에서도 발생할수 있는 Exception 이다.)
c/c++은주소 접근에 자유롭기에 다룰수 있는 기능들이 많지만 이에 따른 위험요소 또한 많아지고,  
신경써야 할 부분이 많은것은 사실이다. 

Heap이라는 Memory또한 Pointer라는 접근자로 Address를 참조하게 되는데,
이런 참조과정에서 무효한 Address를 참조하거나 범위를 초과하는 Address를 참조하는등
Exception을 발생시키는 요인들이 많이 존재한다.
<br>
<center>첫번째무효한 Address 접근의 예</center>
``` c++
  /*Memory Block을 요청을위한 변수 생성*/
  void *memory_block = NULL;
  /*NULL Pointer Exception*/
  block_use(memory_block);
```

<center>두번째 무효한 Address 접근의 예</center>
``` c++
  /*Memory Block을 요청하기*/
  void *memory_block = malloc(size);
  /*생성된 Block사용*/
  block_use(memory_block);
  /*사용이 완료된 Block 반환*/
  free(memory_block);
  /*반환한 변수를 초기화 하지 않음*/
  //memory_block = NULL;
  /*변수 재활용*/
  /*invalid Address Exception*/
  block_use(memory_block);
```

<center>범위를 초과하는 Address 접근의 예</center>
``` c++
  /*Memory Block을 요청을위한 변수 생성*/
  char *memory_block = malloc(sizeof(char) * 5);
  /*생성 사이즈는 5이지만 크기를 넘어선 Loop*/
  for(int i = 0; i < 10; i++)
  {
    /*i >= 5 Invalid Address Exception*/
    printf("%c\n", memory_block[i]);
  }  
```
이런 Exception들은 참조되는 순간 바로 Process를 다운시킴으로, 
항상 참조 접근에 대한 유효성 검사를 올바르게 진행해야한다.

### #3 Solutions
#2에 제시된 Exception들은 단순하기 때문에 문제 발생지점을 찾는데 많은 시간이 들지 않는다. 
하지만 규모가 커지고 참조하는 Point가 많아질수록 이를 해결하는데에 굉장히 많은 
시간을 소요하게 할 수 있고,  System에 따라 Process가 바로 다운되지 않는 경우도 
존재하기 때문에, 잠재적인 문제점을 가지고 있는 Process가 될 수 도있다.

#### 그럼 어떻게 해야 할까?<br>

보통 다운되는 흐름이 눈에 보이는경우는, 특별한 도구 없이 수정하기도하지만,
GDB, OpenSource등을 활용해 해결하는 경우도 다반사이다. 하지만 이는  Complier가 지원되지 않는경우가 있을수 있고, OpenSource를 사용하는것도 때론 
Poring하는데에 더많은 시간을 소비할수도있다. 

때문에 여기서 알아보는 해결방안은 "코드내에 Exception발생을 추적할 수 있는 기능"을 추가하는 것이다. 

"기능을 추가한다"라는 것이 조금 부담스러울 수 있다. <br>
하지만 Hooking기법 또는 동일한 Interface에 서로 다른 Plug_in을 적용해 이를 외부에서 명시적으로 사용한다면,<br>
"코드내에 Exception발생을 추적할 수 있는 기능"Mode, <br>
"일반"Mode<br>
로의 Switching이 원활해지기 때문에,  개발 당시에는 "코드내에 Exception발생을 추적할 수 있는 기능"Mode로개발하고 
프로그램을 Release하는 시점에는  "일반"Mode로 배포함으로써
"완성도 높은 프로그램"을 이뤄낼 수 있다.

<center>Plug_In ?</center>
Plug_In이란 것은 단지 하나의 Interface에서 여러 도구를 필요에 맞게 끼워 맞추는것이라 보면된다. 
C++ 로 예를 들면 부모 객체의 Virtual Function을 자식 객체에서 재정의 하는 것과 같은데,
C를 기반한다면 객체가 없기때문에 Function Pointer로 Plug_In을 유도하거나 
아래와 같이 define문으로 Plug_In을 유도할 수 있다.


``` c
/*정의부*/
#if defined Exception Trace
  #define our_malloc(size)  Trace_malloc(size)
  #define our_free(p)       Trace_free(p)
#else 
  #define our_malloc(size) malloc(size)
  #define our_free(p)      free(p)
#endif
	
  /*사용부*/
  int main()
  {
     void *mem = our_malloc(1);
     our_free(mem);
  }
```
<center>Hooking ?</center>
이 기법의 정의는 함수호출, 메시지, 이벤트등을 중간에서 바꾸거나 가로채는 방법을 의미한다. 
 <center> 요청 -> Os</center>
<center> 요청 -> 중간자 -> Os</center>
<br>
C++은 new와 delete연산자 대한 오버로딩이 가능하기 때문에 직접적인 Hooking을 사용할 수 있다.
``` c
inline void *operator new(std::size_t size)
{
  void *block = Trace_malloc(size);
  return block;
}
inline void operator delete(void *mem)
{
  Trace_free(mem);
}
```


자, 그럼 지금까지 "코드내에 Exception발생을 추적할 수 있는 기능"을 추가하는 것에서 
외부에서 사용할 Interface부분을 보았다. 

#### <center>이제 실제 "추적 기능"을 살펴보자.</center>
추적기능은 말 그대로 추적할수 있는 정보, Meta data를 보관하고 있으면 되고, 이 정보들은 적절한 시점 "외부에 제공된 Interface"가 호출된 지점
에서 오류가 발생하면 Console출력 또는 Console출력이후 Process 강제 다운을 통해 제공하면된다. 

Meta data를 보관하는 방법은 사용자가 Memory Block을 요청할때 요청한 크기보다 Meta data를 담을 수 있도록 좀 더 큰  사이즈로 변경해서
Block을 할당하고 할당된 Memory내에  Meta data를 삽입한후 사용자에겐 Metadata 다음의 주소를 Start Address로 Return 해준다. 

여기서 이 추적기능을 통해 무엇을 알고싶을까
1. 현재 사용률을 알고싶다.
2. 잘못된 참조위치를 확인하고 싶다.
3. Memory Block이 어디서 생성된것인지 알고싶다. 

위 3가지 정보들을 알기위해선 다음과 같은 Meta정보가 필요함을 알수있다.
<center>현재 사용률을 알고싶다.</center>
현재 사용률을 표시하려면 지금까지 요구된 Block이 몇개인지 Block의 크기는 얼마인지알아야 하며,
Block들의 위치를 직접적으로 참조할 수 있어야하기 때문에 List라는 자료구조가 필요하다. 

<center>잘못된 참조위치를 확인하고 싶다.</center>
NULL Pointer Exception 또는 유효하지 않은 Pointer, Double Free등의 잘못된 참조를 
알기위해선 현재까지 사용된 Block을 순회하며, 일치하는 Address를 찾아야하기 때문에 
List 자료구조가 필요하며, 요청한 사이즈만큼만을 사용하였는지 확인하기위해 Checksum정보가 필요하다. 

<center>Memory Block이 어디서 생성된것인지 알고싶다. </center>
Memory Block을 요청할 당시의 호출 Stack이 필요하므로   backtrace정보를 담아야한다.

정리하면 List Link ,  Block사이즈, Checksum, Bactrace를 Meta정보로 사용해야한다. 
<br>
그럼 이제 Memory Block을 디자인 해보자.

|16 Address + 가변|Tail Checksum|4Byte|Block사이즈 OverFlow Checksum |
|16 Address|User Block|가변Byte|Return StartAddress|
|12 Address|Head Checksum|4Byte|StartAddress UnderFlow Checksum|
|8 Address|User Block Size|4Byte|User Block의 사이즈|
|4 Address|BackTrace |4Byte|호출 스택|
|0 Address|List Link|4Byte|Block의 Link정보|

편의상 0번지 주소를 Block의 Start Address로하여 Metadata(20Byte)와 사용자가 요청한 크기의 만큼의 Block(가변Byte)을 생성했다.
우리는 위 Block을 내부의 List Link정보로 전체를 운영할것이고, 사용자에게는 항상 Block의 16번지Address를 반환해주면서
Metadata를 기반하여 Exception발생 지점에서 어떠한 이유로 발생할것인지, 현재 Memory의 사용률을 얼마나 되는지에 대한 정보를 Os가 Process를 다운시키기전에 
미리 확인할 수 있다.

위 내용을 Code로 표현하면 다음과 같다. 
``` c
/*사용자가 Block을 요청하는 경우*/
/*malloc Interface함수명이 'Trace_malloc'이라 가정*/
void *Trace_malloc(size_t size)
{
    unsigned MetaDataSize = ListLinkSize + BackTraceSize + UserBlockSize + HeadChksum + TailChksum;
    void *TraceBlock = malloc(size + MetaDataSize);
    *(TraceBlock + ListLinkStartAddress) = A;
    *(TraceBlock + BackTraceStartAddress) = B;
    *(TraceBlock + UserBlockStartAddress) = C;
    *(TraceBlock + HeadChksumStartAddress) = D;
    *(TraceBlock + TailchksumStartAddress) = E;
    LinkThisBlock(TraceBlock);
    return TraceBlock + UserBlockStartAddress;
}
```
``` c
/*사용자가 Block을 반환하는 경우*/
/*free Interface함수명이 'Trace_free'이라 가정*/
void Trace_free(void *mem)
{    
    void *TraceBlock = mem;
    if(!FindBlock(TraceBlock))
    {
       printf("등록되지 않은 Block을 반환하였습니다. \n");
       exit(0);
    }
    if(BrokenHeadChksum(TraceBlock))
    {
       printf("memory의 시작주소 이전 주소 을 참조하였습니다.\n");
       printf("이 memory는 %d에서 생성되었습니다. 여기를 확인하세요\n", GetBackTrace(TraceBlock));
       exit(0);
    }
    if(BrokenTailChksum)
    {
       printf("memory의 크기를 넘어선 주소 을 참조하였습니다.\n");
       printf("이 memory는 %d에서 생성되었습니다. 여기를 확인하세요\n", GetBackTrace(TraceBlock));
       exit(0);
    }
    Free(GetStartAddressfromTraceBlock(TraceBlock));
}
```
``` c
/*사용률을 확인하고자 하는경우*/
void Print_TraceBlock()
{
  unsigned totalsize = 0;
  unsigned totalnumberof_count = 0;
  for(Trace = TraceHead; Trace != NULL; Trace = GetNextListLink(Trace))
  {
    printf("이 memory는 %d에서 생성되었습니다\n", GetBackTrace(Trace));
    printf("사이즈는 %d입니다.\n", GetUserBlockSize(Trace));
		
    totalsize += GetUserBlockSize(Trace);
    totalnumberof_count++;
  }
  printf("전체 Block의 갯수는 %d개 입니다.\n", totalnumberof_count);
  printf("전체 할당사이즈는 %dByte입니다.\n", totalsize);
}
```


### #4 As I Finished...
음...  Memory사용이 중요한건 누구나가 할거다....!

그냥.. 이렇게 하니까 편하더라구...=_=

나는 #3의 내용에 좀 더 살 붙이기 해서 사용한다. 
궁금하다면 [요기로](https://github.com/whois-hm/workqueue/blob/master/Heap.c)
