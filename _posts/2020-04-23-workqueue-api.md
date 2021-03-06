---
title: Workqueue API
layout: post
image: "/assets/img"
description: Workqueue API Document
main-class: wq_api
color: "#B31917"
introduction: How to use Workqueue
---

## #1 Intro
"About Workqueue" Post에 적용되는  api와,  기타 Utility api를 제공하는 Library 소개.
<br>

## #2 First thing To Know
Library는 Os 종속 없이 Windows / Linux 모두  Porting이 가능하도록 개발될 예정이며
현재는 Linux 기반에서만 Porting이 되어있다. 

#### Repository               [https://github.com/whois-hm/workqueue](https://github.com/whois-hm/workqueue)
#### Workqueue란 ?     [about workqueue](https://whois-hm.github.io/about-workqueue)
### Heap관리               [heap manage](https://whois-hm.github.io//heap-manage)
## #3Description
``` c++
-------------------------------------------------------------------------------------------
/*
Des           : Workqueue를 생성한다.
Param size    : 사용할 Event의 최대 크기(Queue Element Size)
Param length  : Queue의 갯수
Return        : 생성 성공시 HANDLE 실패시 NULL
*/
-------------------------------------------------------------------------------------------
WQ_API struct WQ *WQ_open(_dword size, _dword length);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : 생성한 Workqueue를 종료한다.
Param pwq     : "WQ_open"에서 반환된 HANDLE Address
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void WQ_close(struct WQ **pwq);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : Event를 요청한다.
Param wq      : 요청하고자 하는 "WQ_open"에서 반환된 HANDLE
Param par     : 보내고자 하는 Event Address
Param npar    : 보내고자 하는 Event의 크기 (par Size)
Param time    : Event를 Queue에 넣기 까지 대기하는 시간
Param flags   : WQ_SEND_FLAGS_BLOCK 지정시 Blocking Mode 지정하지 않을시 Non Block Mode
Return        : WQ_SIGNALED Event가 Queue에 전달되었음
                WQEINVAL 잘못된 Parameter
                WQ_TIMED_OUT 지정된 시간동안 Event를 Queue에 넣지 못하였음
                WQ_FAIL 예상치 못한 Error가발생했음
*/
-------------------------------------------------------------------------------------------
WQ_API _dword WQ_send(struct WQ *wq, void *par, _dword npar, _dword time, int flags);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : Event발생 여부를 확인한다.
Param wq      : 수신하고자 하는 "WQ_open"에서 반환된 HANDLE
Param ppar    : "WQ_send"를 통해 보내진 Event를 넘겨받는 Address
Param pnpar   : "WQ_send"를 통해 보내진 Event의 크기 (ppar Size)
Param time    : Event발생 여부를 확인하는 Timeout (INFINITE지정시 무한대기)
Return        : WQ_SIGNALED Event가 발생됨
                WQEINVAL 잘못된 Parameter
                WQ_TIMED_OUT 지정된 Timeout시간동안 Event가 발생하지 않음
                WQ_FAIL 예상치 못한 Error가발생했음
*/
-------------------------------------------------------------------------------------------
WQ_API _dword WQ_recv(struct WQ *wq, void **ppar, _dword *pnpar, _dword time);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : Thread를 생성한다.
Param fn      : 생성된 Thread에서 수행될 Function
Param p       : Parameter fn의 인자로 전달될 Address
Param pnpar   : "WQ_send"를 통해 보내진 Event의 크기 (ppar Size)
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API struct BackGround *BackGround_turn(BackGround_function fn, void *p);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : 생성된 Thread의 Routine이 완료될때 까지 대기한후 생성된 Thread를 종료한다.
Param pb      : "BackGround_turn"에서 반환된 HANDLE Address
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void BackGround_turnoff(struct BackGround **pb);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : 호출지점 Thread의 ID를 반환한다.
Return        : 호출한 Thread의 ID
*/
-------------------------------------------------------------------------------------------
WQ_API threadid BackGround_id();
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : Memory를 Hex Edit형태로 Console에 출력한다.
Param mem     : 출력하고자하는 Memory의 Start Address
Param size    : 출력하고자하는 Memory의 Size (mem Size)
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void Printf_memory(void *mem, _dword size);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : System Boot이후의 경과 시간을 얻어온다.
Return        : System Boot이후의 경과 시간
*/
-------------------------------------------------------------------------------------------
WQ_API _dword Time_tickcount();
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : 호출자 Thread를 Sleep상태로 병경한다.
Param time    : Sleep Milliseconds
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void Time_sleep(_dword time);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : String 또는 Integer값을 연속해서 저장할수 있는 Dictionary를 생성한다.
Return        : Dictionary생성시 HANDLE 실패시 NULL 
*/
-------------------------------------------------------------------------------------------
WQ_API struct Dictionary *Dictionary_create();
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : "Dictionary_create"에서 반환받은 HANDLE을 삭제한다.
Param pd      : "Dictionary_create"에서 반환받은 HANDLE Address
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void Dictionary_delete(struct Dictionary **pd);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : String에 대한 Memory를 새로 생성하여 저장한다.
Param d       : "Dictionary_create"에서 반환받은 HANDLE
Param k       : 저장한 String을 찾을 Key값
Param v       : 저장할 String값
Return        : WQENOMEM Memory생성에 실패하였음
                WQEINVAL 잘못된 Parameter
                WQEALREADY 동일한 Key값이 이미 저장되어있음
                WQ_OK 저장에 성공하였음
              
*/
-------------------------------------------------------------------------------------------
WQ_API _dword Dictionary_add_char(struct Dictionary *d, char *k, char *v);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : Integer에 대한 값을 저장한다.
Param d       : "Dictionary_create"에서 반환받은 HANDLE
Param k       : 저장한 String을 찾을 Key값
Param v       : 저장할 Integer값
Return        : WQENOMEM Memory생성에 실패하였음
                WQEINVAL 잘못된 Parameter
                WQEALREADY 동일한 Key값이 이미 저장되어있음
                WQ_OK 저장에 성공하였음
              
*/
-------------------------------------------------------------------------------------------
WQ_API _dword Dictionary_add_int(struct Dictionary *d, char *k, int v);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : 저장된 Key를 삭제한다.
Param d       : "Dictionary_create"에서 반환받은 HANDLE
Param k       : 삭제할 Key값
Return        : None              
*/
-------------------------------------------------------------------------------------------
WQ_API void Dictionary_remove(struct Dictionary *d, char *k);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : 저장된 String을 참조하여 얻어온다.
Param d       : "Dictionary_create"에서 반환받은 HANDLE
Param k       : 찾을 Key값
Return        : Key에 대한 String을 찾으면 해당 String의 참조 Address 없으면 NULL              
*/
-------------------------------------------------------------------------------------------
WQ_API char *Dictionary_refchar(struct Dictionary *d, char *k);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : 저장된 String을 새로운 Memory를 생성하여 복사해온다.
Param d       : "Dictionary_create"에서 반환받은 HANDLE
Param k       : 찾을 Key값
Return        : Key에 대한 String을 찾으면 새로 생성해 복사된 Address 없으면 NULL              
*/
-------------------------------------------------------------------------------------------
WQ_API char *Dictionary_allocchar(struct Dictionary *d, char *k);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : "Dictionary_allocchar"에서 반환받은 Address를 삭제한다.
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void Dictionary_freechar(char **allocchar);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : 저장된 Integer값을 얻어온다.
Param d       : "Dictionary_create"에서 반환받은 HANDLE
Param k       : 찾을 Key값
Return        : Key에 대한 Interger를 찾으면 해당 값 없으면 0
*/
-------------------------------------------------------------------------------------------
WQ_API int Dictionary_copyint(struct Dictionary *d, char *k);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : Dictionary에 해당 Key의 존재 유무를 검사한다.
Param d       : "Dictionary_create"에서 반환받은 HANDLE
Param k       : 찾을 Key값
Return        : Key가 존재하면 1 없으면 -1
*/
-------------------------------------------------------------------------------------------
WQ_API int Dictionary_haskey(struct Dictionary *d, char *k);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : CriticalSection 객체를 생성한다.
Param cs      : 객체를 얻어올 Address
Return        : 객체를 얻어오면 0 / 잘못된 Parameter이면 WQEINVAL / Error가 발생 하면 0보다 작은 음수
*/
-------------------------------------------------------------------------------------------
WQ_API int CS_open(criticalsection *cs);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : CriticalSection 구역에 Lock을 건다
Param cs      : "CS_open"에서 얻어온 객체 Address
Param time    : Lock을 얻어올때 까지 대기하는 Milliseconds 시간 (INFINITE지정시 무한 대기)
Return        : WQ_SIGNALED CriticalSection구역에 Lock을 걸고 진입하는데 성공함
                WQEINVAL 잘못된 Parameter
                WQ_TIMED_OUT 지정된 Timeout시간동안 Lock을 얻는데 실패하였음
                WQ_FAIL 예상치 못한 Error가발생했음
*/
-------------------------------------------------------------------------------------------
WQ_API _dword CS_lock(criticalsection *cs, _dword time);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : "CS_lock"이 걸어둔 Lock을 해지한다.
Param cs      : "CS_open"에서 얻어온 객체 Address
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void CS_unlock(criticalsection *cs);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : "CS_open"에서 얻어온 객체를 삭제한다.
Param cs      : "CS_open"에서 얻어온 객체 Address
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void CS_close(criticalsection *cs);
 -------------------------------------------------------------------------------------------
 
 -------------------------------------------------------------------------------------------
/*
Des           : Semaphore 객체를 생성한다.
Param sem     : 객체를 얻어올 Address
Param i       : 생성초기 Semaphore값
Param m       : 최대 Semaphore값
Return        : 객체를 얻어오면 0 / Error가 발생 하면 0보다 작은 음수
*/
-------------------------------------------------------------------------------------------
WQ_API int SEMA_open(semaphore *sem, _dword i, _dword m);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : Semaphore 하나를 가져온다.
Param sem     : "SEMA_open"에서 가져온 객체 Address
Param time    : Semaphore를 얻어올때 까지 대기하는 Milliseconds 시간 (INFINITE지정시 무한 대기)
Return        : WQ_SIGNALED Semaphore 하나를 가져오는데 성공함
                WQEINVAL 잘못된 Parameter
                WQ_TIMED_OUT 지정된 Timeout시간동안 Lock을 얻는데 실패하였음
                WQ_FAIL 예상치 못한 Error가발생했음
*/
-------------------------------------------------------------------------------------------
WQ_API _dword SEMA_lock(semaphore *sem, _dword time);
-------------------------------------------------------------------------------------------

 -------------------------------------------------------------------------------------------
/*
Des           : "SEMA_lock"에서 가져온 Semaphore를 돌려준다.
Param sem     : "SEMA_open"에서 가져온 객체 Address
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void SEMA_unlock(semaphore *sem);
-------------------------------------------------------------------------------------------

 -------------------------------------------------------------------------------------------
/*
Des           : "SEMA_open"에서 가져온 객체를 삭제한다.
Param sem     : "SEMA_open"에서 가져온 객체 Address
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void SEMA_close(semaphore *sem);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : "libwq_malloc"사용을 위해 내부 값을 생성해낸다.
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void libwq_heap_testinit();
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : "libwq_heap_testinit"에서 생성된 값을 삭제한다.
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void libwq_heap_testdeinit();
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : "malloc"을 대체 하여 Memory protect기능과 Meta data를 갖는 Heap Memory를 얻어온다.
Param Size    : 요청하는 Heap Memory의 크기
Return        : 성공시 Heap Memory의 Start Address 실패시 NULL
*/
-------------------------------------------------------------------------------------------
WQ_API void *libwq_malloc(size_t size);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : "Free"를 대체하여 "libwq_malloc"에서 얻어온 Heap Memory를 반환한다. 
Param mem     : "libwq_malloc"에서 반환받은 Memory
Return        : None
*/
-------------------------------------------------------------------------------------------
void libwq_free(void *mem);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : "libwq_malloc"에서 반환받은 Memory들의 Meta data를 Console로 출력한다.
Return        : None
*/
-------------------------------------------------------------------------------------------
void libwq_print_heap();
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : SmartPtr사용을 위한 Golbal값을 초기화한다.
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void SP_using();
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : SmartPointer Memory Block을 생성한다.
Param size    : 사용할 Block Size
Param refer   : 참조될 카운트 초기값
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void *SP_malloc(size_t size, int refer);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : SmartPointer Memory Block의 참조 카운트값을 증가시킨다.
Param spmem   : "SP_malloc" 통해 반환 받은 Memory Block
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void *SP_ref(void *spmem);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : SmartPointer Memory Block의 참조 카운트값을 감소키며,
                                              참조 카운트가 0이 되면 Memory Block을 해지한다.
Param spmem   : "SP_malloc" 통해 반환 받은 Memory Block
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void SP_unref(void *spmem);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : SmartPointer Memory Block의 참조 카운트값을 감소키며,
                                              참조 카운트가 0이 되면 Memory Block을 해지한다.
Param spmem   : "SP_malloc" 통해 반환 받은 Memory Block
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void SP_free(void *spmem);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : SmartPtr 사용을 종료하며 "SP_using"에서 생성된 Global값을 해지한다.
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void SP_unused();
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des                        : NetWork Group을 생성한다.
Param max_routeindex       : 생성될 Route Table의 마지막 Index
Param max_routeparsize     : Route Parameter의 최대 사이즈
Param phosts               : Router아래 생성될 Host들의 정보를 담는 배열
Param nhosts               : "phosts"의 갯수
Param message_decoder      : 타 Group의 Message 수신 Decoder
Param message_port         : 외부 Group의 Message를 수신할 Port
Param message_buffer_size  : Message수신 Buffer 사이즈
Return                     : 성공시 생성된 Router, 실패시 NULL
*/
-------------------------------------------------------------------------------------------
WQ_API struct net_router *NetGroup_router_create(unsigned max_routeindex,
		unsigned max_routeparsize,
		struct net_host_register_data *phosts,
		unsigned nhosts,
		_router_message_decoder message_decoder,
		int message_port,
		unsigned message_buffer_size);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des           : 생성된 NetWork Group을 삭제한다.
Param router  : "NetGroup_router_create"에서 반환받은 값
Return        : None
*/
-------------------------------------------------------------------------------------------
WQ_API void NetGroup_router_delete(struct net_router ** router);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des            : Local Group unicast를 위한 연결을 시도한다.
Param router   : "NetGroup_router_create"에서 반환받은 값
Param hostname :  "net_host_register_data"에 등록된 연결할 Host의 Index
Return         : 성공시 연결된 Host 실패시 NULL
*/
-------------------------------------------------------------------------------------------
WQ_API struct net_host *NetGroup_host_connect(struct net_router * router,
		unsigned hostname);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des                : "NetGroup_host_connect"를 통해 Host 연결을 끊는다.
Param targethost   : "NetGroup_host_connect"에서 반환받은 값
Return             :  None
*/
-------------------------------------------------------------------------------------------
WQ_API void NetGroup_host_disconnect(struct net_host **targethost);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des                : Local Group간 Unicast를 전송한다.
Param targethost   : "NetGroup_host_connect"에서 반환받은 값
Param host         : 자신의 Host 
Param par          : 전달할 Parameter
Param npar         : 전달할 Parameter Size
Param flag         : Nonblocking or Blocking 모드선택
Return             :  None
*/
-------------------------------------------------------------------------------------------
WQ_API void NetGroup_host_unicast(struct net_host *target_host,
		struct net_host *host,
		void *par,
		unsigned npar,
		unsigned flag);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des                : Local Group간 Broadcast를 전송한다.
Param host         : 자신의 Host 
Param par          : 전달할 Parameter
Param npar         : 전달할 Parameter Size
Return             :  None
*/
-------------------------------------------------------------------------------------------
WQ_API void NetGroup_host_broadcast(struct net_host *host,
		void *par,
		unsigned npar);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des                 : Local Group간 Multicast를 전송한다.
Param host          : 자신의 Host 
Param routeindex    : 전달할 Routing Index
Param par           : 전달할 Parameter
Param npar          : 전달할 Parameter Size
Return              :  None
*/
-------------------------------------------------------------------------------------------
WQ_API void NetGroup_host_multicast(struct net_host *host,
		unsigned routeindex,
		void *par,
		unsigned npar);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des                 : External Group에 Message를 전송한다.
Param router        : "NetGroup_router_create"에서 반환받은 값
Param ip            : External Group IP주소
Param port          : External Group Port번호
Param par           : 전달할 Parameter
Param npar          : 전달할 Parameter Size
Param encoder       : "par"에 전달된 값을 Protocol형태로 변환하는 Callback Encoder
Return              :  -1 잘못된 Parameter 0 전송실패 1 전송성공
*/
-------------------------------------------------------------------------------------------
WQ_API int NetGroup_router_send_message(struct net_router *router,
		char const *ip,
		int port,
		void *par,
		unsigned npar,
		_router_message_encoder encoder);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des                : "NetGroup_router_send_message"에서 사용할 Protocol Message Buffer
                                                              를 생성한다.
Param want         : 생성할 Buffer의 크기
Param clear        : 버퍼 Clear여부
Return             :  None
*/
-------------------------------------------------------------------------------------------
WQ_API void NetGroup_router_send_message_buffer_realloc(struct _router_message_buffer *b,
		unsigned want,
		int clear);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des                 : Local Group간의 Multicast Route Table에 수신을 등록한다.
Param routeindex    : 등록할 Routing Index
Param router        : "NetGroup_router_create"에서 반환받은 값
Param host          : 자신 Host
Return              : None
*/
-------------------------------------------------------------------------------------------
WQ_API void NetGroup_membership_join(unsigned routeindex,
		struct net_router *router,
		struct net_host *host);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des                 : Local Group간의 Multicast Route Table에 등록을 해지한다.
Param routeindex    : 해지할 Routing Index
Param router        : "NetGroup_router_create"에서 반환받은 값
Param host          : 자신 Host
Return              : None
*/
-------------------------------------------------------------------------------------------
WQ_API void NetGroup_membership_drop(unsigned routeindex,
		struct net_router *router,
		struct net_host *host);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des                 : 자신의 Scheduler에 Timer를 생성한다.
Param host          : 자신 Host
Param fn_alarm      : Timer 만료후 호출될 Callback
Param mstime        : Timer 생성이후 만료될때 까지의 Milliseconds
Param repeat        : Timer 반복호출 여부
Return              : 성공시 생성된 Timer 실패시 NULL
*/
-------------------------------------------------------------------------------------------
WQ_API struct net_host_alarm *NetGroup_host_alarm_start(struct net_host *host,
		void (*fn_alarm)(struct net_router *router, struct net_host *host, void *meta),
		unsigned mstime,
		int repeat);
-------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------
/*
Des                 : 생성된 Timer를 제거한다.
Param host          : 자신 Host
Param alarm         : "NetGroup_host_alarm_start"를 통해 반환받은 Timer
Return               : None
*/
-------------------------------------------------------------------------------------------
WQ_API void NetGroup_host_alarm_end(struct net_host *host,
		struct net_host_alarm **alarm);
-------------------------------------------------------------------------------------------



```


## #4 As I Finished...
내가 이 Library를 별도로 운영하는건 OpenSource처럼 무언가 큰 의미를 갖는것은 아니다. 

개발함에 있어서 자주 사용되는 기능들을 모아 두어 매번 재활용 가능하게 만들어 두기 위함이며

동일한 기능을 갖되  Os 마다 제공되는 API들을 하나의 Interface로 활용하기 위함이며

디버깅에 있어 더 빠른, 더 확실함을 증명하기 위함이며

개인적으로 Os Platform을 학습하기 위함이며

Library를 작성해보면서 '사용자'와 '제공자'측면에서의 Interface 를 학습하기 위함이다.

때문에 필요하다면 조금씩 기능을 확장하고 코드 커스터마이징을 해나갈것이다.
