---
layout: post
title:  "Java 11 JVM Log 정리"
subtitle: "Java8에서 달라지는 Java11 JVM Log"
date:   2020-12-01 10:47:00
categories: [study]
---

OpenJDK11은 OpenJDK8 이후의 첫번째 LTS(긴 기간 지원해주는)버전이다.
이 버전은 JVM 로그 설정이 많이 바뀌었다. 표준화(standardization) 및 단일화(unification)되는 방향으로 바뀌었다.

# 1. JVM 로그 읽는 법

보통 자바 코드로 작성된 코드는 아래와 같다.

```java
Logger logger = LogFactory.getLooger("core-logger");
logger.info("this is core logger log");
```

```java
2020-02-05 10:50:52.670  INFO [core-logger] [22] [pool-13-thread-1]: this is core logger log
```

실제 결과도 위와 같이 나온다.

time stamp, log level, core logger, log content 등을 포함하고 있다.

<br>
<br>

```java
[0.182s][debug][jit,compilation]    1       3       java.lang.StringLatin1::hashCode (42 bytes)
[0.183s][debug][jit,compilation]    2       3       java.lang.Object::<init> (1 bytes)
[0.183s][debug][jit,compilation]    3       3       java.lang.String::hashCode (49 bytes)
```

JVM로그도 비슷하다.

기본 JVM 로그 포맷은 아래와 같다.

```java
[start time] [log level] [log label, may contain multiple] log contents
```

log label은 여러개가 한번에 나올 수 있다.

설정에 따라서 달라지게 되고, 대부분의 태그는 JVM 개발자를 위한 것이고, 일부 태그는 JVM 사용하는 사용자들이 파라미터 튜닝이나 코드 튜닝을 위해 사용한다.

<br>
<br>

## A. GC 관련

gc log와 관련된 JVM 태그는 많이 있다.

#### Tag GC

GC의 발생 시간, 소요 시간, 메모리 크기를 나타낸다. (info level)

```
pause young (normal) (G1 evaluation pause) 3480m - > 1565m (5120m) 15.968ms
```

gc type, gc reason, collected memory size, duration and other information

위 정보를 담고 있다.

#### gc,age

gc의 age관련 정보가 나타난다. 
- trace 레벨이라면, 해당 age 이하의 모든 age 값의 크기와 각각의 age에서의 모든 객체의 전체 크기를 나타낸다. 
- debug 레벨이라면, 가장 높은 age와 기대되는 크기(현재 크기는 아니다)가 나온다.

```java
[2020-02-26T08:34:12.823+0000][debug][gc,age         ] gc(1661) Desired survivor size 167772160 bytes, new threshold 6 (max threshold 6)
[2020-02-26T08:34:12.823+0000][trace][gc,age         ] age   1:   16125960 bytes,   16125960 total
[2020-02-26T08:34:12.823+0000][trace][gc,age         ] age   2:   16259512 bytes,   32385472 total
[2020-02-26T08:34:12.823+0000][trace][gc,age         ] age   3:    2435240 bytes,   34820712 total
[2020-02-26T08:34:12.823+0000][trace][gc,age         ] age   4:   17179320 bytes,   52000032 total
[2020-02-26T08:34:12.823+0000][trace][gc,age         ] age   5:   43986952 bytes,   95986984 total
[2020-02-26T08:34:12.823+0000][trace][gc,age         ] age   6:   20858328 bytes,  116845312 total
```

#### gc,alloc / gc,alloc,region

위 두개의 파라미터는 g1gc의 gc가 끝난뒤 gc.alloc에만 유효하다.
- trace log에 어떤 쓰레드가 gc를 trigger했는지, gc 결과의 address를 리턴한다.
    - 이 로그는 gc를 디버깅할 때만 필요하다.
- debug level에서는 gc,alloc,region인 경우 각각의 gc 이후에 리전 갯수를 나타낸다.

```java
[2020-02-28T02:14:02.694+0000][trace][gc,alloc                    ] sdk-27692-2-amqp-t-4: Successfully scheduled collection returning 0x00000007ffc00000
[2020-02-28T02:16:00.372+0000][debug][gc,alloc,region             ] gc(7848) Mutator Allocation stats, regions: 677, wasted size: 63832B ( 0.0%)
```

#### gc,cpu

gc 문제 파악을 위해서 주로 먼저 파악되는 곳이다.
- info 레벨 로그로 나타난다.

```java
[2020-02-28T01:59:46.406+0000][info ][gc,cpu                      ] gc(7841) User=0.10s Sys=0.00s Real=0.04s
[2020-02-28T02:01:20.148+0000][info ][gc,cpu                      ] gc(7842) User=0.04s Sys=0.06s Real=0.04s
```

JFR(Java Flight Recorder) statics와는 다르다. JFR의 경우는 gc 시작지점부터 gc 시간이고, 위 태그의 경우에는 start 태그가 시작한 지점부터라서 약간 다르다는 점은 조심해야한다.


#### gc,ergo / gc,ergo,cset / gc,ergo,ihop / gc,ergo,refine

위 태그는 Adaptive Size Policy를 위한 것이다.

- Adaptive Size Policy
    - G1GC는 자바9부터 IHOP 사이즈를 자동으로 조절한다.
        - IHOP: Initiating Heap Occupancy Percent, Java8기준 40%이상 old gen 사용량이 늘어나면 mixed gc가 발생하게 된다.(minor gc보다 한단계 높은 레벨의)

```java
[2020-02-28T01:59:46.367+0000][trace][gc,ergo,cset                ] gc(7841) Start choosing CSet. pending cards: 26996 predicted base time: 13.34ms remaining time: 186.66ms target pause time: 200.00ms
[2020-02-28T01:59:46.367+0000][trace][gc,ergo,cset                ] gc(7841) Add young regions to CSet. eden: 676 regions, survivors: 6 regions, predicted young region time: 19.02ms, target pause time: 200.00ms
[2020-02-28T01:59:46.367+0000][debug][gc,ergo,cset                ] gc(7841) Finish choosing CSet. old: 0 regions, predicted old region time: 0.00ms, time remaining: 167.64
[2020-02-28T01:59:46.389+0000][debug][gc,ergo                     ] gc(7841) Running g1 Clear Card Table Task using 4 workers for 7 units of work for 895 regions.
[2020-02-28T01:59:46.391+0000][debug][gc,ergo                     ] gc(7841) Running g1 Free Collection Set using 4 workers for collection set length 682
[2020-02-28T01:59:46.391+0000][trace][gc,ergo,refine              ] gc(7841) Updating Refinement Zones: update_rs time: 6.800ms, update_rs buffers: 397, update_rs goal time: 19.998ms
[2020-02-28T01:59:46.391+0000][debug][gc,ergo,refine              ] gc(7841) Updated Refinement Zones: green: 572, yellow: 1716, red: 2860
[2020-02-28T02:01:20.108+0000][trace][gc,ergo,cset                ] gc(7842) Start choosing CSet. pending cards: 25786 predicted base time: 12.87ms remaining time: 187.13ms target pause time: 200.00ms
[2020-02-28T02:01:20.108+0000][trace][gc,ergo,cset                ] gc(7842) Add young regions to CSet. eden: 677 regions, survivors: 5 regions, predicted young region time: 14.43ms, target pause time: 200.00ms
[2020-02-28T02:01:20.108+0000][debug][gc,ergo,cset                ] gc(7842) Finish choosing CSet. old: 0 regions, predicted old region time: 0.00ms, time remaining: 172.70
[2020-02-28T02:01:20.132+0000][debug][gc,ergo                     ] gc(7842) Running g1 Clear Card Table Task using 4 workers for 8 units of work for 903 regions.
[2020-02-28T02:01:20.133+0000][debug][gc,ergo                     ] gc(7842) Running g1 Free Collection Set using 4 workers for collection set length 682
[2020-02-28T02:01:20.133+0000][trace][gc,ergo,refine              ] gc(7842) Updating Refinement Zones: update_rs time: 6.303ms, update_rs buffers: 305, update_rs goal time: 19.997ms
[2020-02-28T02:01:20.133+0000][debug][gc,ergo,refine              ] gc(7842) Updated Refinement Zones: green: 572, yellow: 1716, red: 2860
[2020-02-28T02:04:36.095+0000][trace][gc,ergo,cset                ] gc(7843) Start choosing CSet. pending cards: 26115 predicted base time: 12.85ms remaining time: 187.15ms target pause time: 200.00ms
[2020-02-28T02:04:36.095+0000][trace][gc,ergo,cset                ] gc(7843) Add young regions to CSet. eden: 676 regions, survivors: 6 regions, predicted young region time: 69.11ms, target pause time: 200.00ms
[2020-02-28T02:04:36.095+0000][debug][gc,ergo,cset                ] gc(7843) Finish choosing CSet. old: 0 regions, predicted old region time: 0.00ms, time remaining: 118.04
[2020-02-28T02:04:36.118+0000][debug][gc,ergo                     ] gc(7843) Running g1 Clear Card Table Task using 4 workers for 7 units of work for 894 regions.
[2020-02-28T02:04:36.120+0000][debug][gc,ergo                     ] gc(7843) Running g1 Free Collection Set using 4 workers for collection set length 682
[2020-02-28T02:04:36.121+0000][trace][gc,ergo,refine              ] gc(7843) Updating Refinement Zones: update_rs time: 6.929ms, update_rs buffers: 364, update_rs goal time: 19.997ms
[2020-02-28T02:04:36.121+0000][debug][gc,ergo,refine              ] gc(7843) Updated Refinement Zones: green: 572, yellow: 1716, red: 2860
```

#### gc,heap / gc,heap,region

- debug level인 경우 일반적인 heap 정보를 알려준다.
- trace인 경우, gc,heap,region 로그에 각각의 리전에 대한 상세 정보가 나타난다. gc 디버깅을 위해 보통 필요하다.

보통은 gc와 heap log만 신경쓰면 된다.

```java
[2020-02-28T06:01:20.787+0000][debug][gc,heap                     ] gc(7922) Heap before gc invocations=7922 (full 0): garbage-first heap   total 8388608K, used 4076387K [0x0000000600000000, 0x0000000800000000)
[2020-02-28T06:01:20.787+0000][debug][gc,heap                     ] gc(7922)   region size 4096K, 682 young (2793472K), 5 survivors (20480K)
[2020-02-28T06:01:20.787+0000][debug][gc,heap                     ] gc(7922)  Metaspace       used 163068K, capacity 166731K, committed 169728K, reserved 1198080K[2020-02-28T06:01:20.787+0000][debug][gc,heap                     ] gc(7922)   class space    used 18180K, capacity 19580K, committed 20480K, reserved 1048576K
[2020-02-28T06:01:20.787+0000][trace][gc,heap,region              ] gc(7922) Heap Regions: E=young(eden), S=young(survivor), O=old, HS=humongous(starts), HC=humongous(continues), CS=collection set, F=free, A=archive, TAMS=top-at-mark-start (previous, next)[2020-02-28T06:01:20.787+0000][trace][gc,heap,region              ] gc(7922) |   0|0x0000000600000000, 0x0000000600400000, 0x0000000600400000|100%| O|  |TAMS 0x0000000600400000, 0x0000000600000000| Untracked
[2020-02-28T06:01:20.787+0000][trace][gc,heap,region              ] gc(7922) |   1|0x0000000600400000, 0x0000000600800000, 0x0000000600800000|100%| O|  |TAMS 0x0000000600800000, 0x0000000600400000| Untracked
[2020-02-28T06:01:20.787+0000][trace][gc,heap,region              ] gc(7922) |   2|0x0000000600800000, 0x0000000600c00000, 0x0000000600c00000|100%| O|  |TAMS 0x0000000600c00000, 0x0000000600800000| Untracked
[2020-02-28T06:01:20.787+0000][trace][gc,heap,region              ] gc(7922) |   3|0x0000000600c00000, 0x0000000601000000, 0x0000000601000000|100%| O|  |TAMS 0x0000000601000000, 0x0000000600c00000| Untracked
[2020-02-28T06:01:20.787+0000][trace][gc,heap,region              ] gc(7922) |   4|0x0000000601000000, 0x0000000601400000, 0x0000000601400000|100%| O|  |TAMS 0x0000000601400000, 0x0000000601000000| Untracked
[2020-02-28T06:01:20.787+0000][trace][gc,heap,region              ] gc(7922) |   5|0x0000000601400000, 0x0000000601800000, 0x0000000601800000|100%| O|  |TAMS 0x0000000601800000, 0x0000000601400000| Untracked
[2020-02-28T06:01:20.787+0000][trace][gc,heap,region              ] gc(7922) |   6|0x0000000601800000, 0x0000000601c00000, 0x0000000601c00000|100%| O|  |TAMS 0x0000000601c00000, 0x0000000601800000| Untracked
```

#### gc,humongous

g1gc 사용하게 되면, evalutation failure이나 humongous allocation을 종종 만나게 된다. 그 이유를 알 필요가 있다면, 이 태그를 살펴보면 된다.

- Humongous Allocation : 리전 절반 이상의 크기의 객체를 old 영역에 저장할 경우 발생

```java
[2020-02-28T06:01:20.831+0000][debug][gc,humongous                ] gc(7922) Live humongous region 219 object size 2160888 start 0x0000000636c00000  with remset 1 code roots 0 is marked 0 reclaim candidate 0 type array 0
[2020-02-28T06:01:20.831+0000][debug][gc,humongous                ] gc(7922) Live humongous region 412 object size 2160888 start 0x0000000667000000  with remset 1 code roots 0 is marked 0 reclaim candidate 0 type array 0
[2020-02-28T06:01:20.831+0000][debug][gc,humongous                ] gc(7922) Live humongous region 443 object size 3241320 start 0x000000066ec00000  with remset 1 code roots 0 is marked 0 reclaim candidate 0 type array 0
[2020-02-28T06:01:20.831+0000][debug][gc,humongous                ] gc(7922) Live humongous region 489 object size 2160888 start 0x000000067a400000  with remset 2 code roots 0 is marked 0 reclaim candidate 0 type array 0
[2020-02-28T06:01:20.831+0000][debug][gc,humongous                ] gc(7922) Live humongous region 490 object size 2160888 start 0x000000067a800000  with remset 1 code roots 0 is marked 0 reclaim candidate 0 type array 0
[2020-02-28T06:01:20.831+0000][debug][gc,humongous                ] gc(7922) Live humongous region 499 object size 7292936 start 0x000000067cc00000  with remset 2 code roots 0 is marked 0 reclaim candidate 0 type array 0
[2020-02-28T06:01:20.831+0000][debug][gc,humongous                ] gc(7922) Live humongous region 536 object size 2160888 start 0x0000000686000000  with remset 2 code roots 0 is marked 0 reclaim candidate 0 type array 0
[2020-02-28T06:01:20.831+0000][debug][gc,humongous                ] gc(7922) Live humongous region 656 object size 2160888 start 0x00000006a4000000  with remset 1 code roots 0 is marked 0 reclaim candidate 0 type array 0
[2020-02-28T06:01:20.831+0000][debug][gc,humongous                ] gc(7922) Live humongous region 768 object size 2160888 start 0x00000006c0000000  with remset 1 code roots 0 is marked 0 reclaim candidate 0 type array 0
[2020-02-28T06:01:20.831+0000][debug][gc,humongous                ] gc(7922) Live humongous region 786 object size 2160888 start 0x00000006c4800000  with remset 1 code roots 0 is marked 0 reclaim candidate 0 type array 0
```

#### GC,metaspace / GC,metaspace,freelist / GC,metaspace,freelist,blocks

metaspace와 관련된 gc log를 확인해야 하는 경우, gc와 관련된 메타스페이스의 변경을 알 수 있다.

```java
[2020-02-28T04:32:13.123+0000][info ][gc,metaspace                ] gc(7896) Metaspace: 163062K->163062K(1198080K)
[2020-02-28T04:35:44.956+0000][trace][gc,metaspace,freelist       ] SpaceManager::grow_and_allocate for 49 words 109 words used 19 words left
[2020-02-28T04:35:44.956+0000][trace][gc,metaspace,freelist       ] ChunkManager::free_chunks_get: free_list: 0x00007fddccb89770 chunks left: 433.
[2020-02-28T04:35:44.956+0000][trace][gc,metaspace,freelist       ] ChunkManager::chunk_freelist_allocate: 0x00007fddccb89770 chunk 0x00007fdc74221000  size 128 count 433 Free chunk total 255616  count 824
[2020-02-28T04:35:44.956+0000][trace][gc,metaspace,freelist,blocks] returning block at 0x00007fdd95575b68 size = 19
[2020-02-28T04:35:44.956+0000][trace][gc,metaspace,freelist       ] SpaceManager::added chunk: 
[2020-02-28T04:35:44.956+0000][trace][gc,metaspace,freelist       ] Metachunk: bottom 0x00007fdc74221000 top 0x00007fdc74221040 end 0x00007fdc74221400 size 128 (specialized)
[2020-02-28T04:35:44.956+0000][trace][gc,metaspace,freelist       ] Free chunk total 255616  count 824
[2020-02-28T04:36:35.367+0000][info ][gc,metaspace                ] gc(7897) Metaspace: 163065K->163065K(1198080K)
```

#### gc,phases / gc,phases,ref / gc,phases,task / gc,ref / gc,start / gc,ref,start

위 태그들은 gc step과 관련이 있다. gc 알고리즘에 대해 알고 싶다면, 확인해보면 좋다.

![image](https://docs.oracle.com/en/java/javase/11/gctuning/img/jsgct_dt_001_grbgcltncyl.png)

G1GC의 GC Phase


<br>
<br>

## B. 클래스 로딩은 런타임 컴파일과 관련이 있다.

#### class,preorder / class,init / class,load / class,unload

이름과 같이, 클래스 초기화, 클래스 로딩, 클래스 언로딩과 관련된 로그이다.
- 보통 info 레벨 로그로도 충분하다.
- trace 레벨를 확인하면 클래스 로딩 프로세스에 대해서 알 수 있다.

```java
[8.931s][debug][class,preorder  ] com.fasterxml.jackson.core.PrettyPrinter source: file:/D:/Repositories/maven/com/fasterxml/jackson/core/jackson-core/2.10.0/jackson-core-2.10.0.jar
[8.931s][info][class,init              ] 2740 Initializing 'com/fasterxml/jackson/core/PrettyPrinter' (0x0000000801399220)
[8.934s][info][class,load              ] com.fasterxml.jackson.core.PrettyPrinter source: file:/D:/Repositories/maven/com/fasterxml/jackson/core/jackson-core/2.10.0/jackson-core-2.10.0.jar
```

#### jit,compilation

보통 컴파일 최적화를 위해서 필요하다.

```java
[2020-02-28T03:01:51.619+0000][debug][jit,compilation] 153756   !   4       jdk.internal.reflect.GeneratedConstructorAccessor161::newInstance (49 bytes)   made zombie
[2020-02-28T03:01:51.620+0000][debug][jit,compilation] 153219       4       io.lettuce.core.protocol.CommandArgs$IntegerArgument::encode (12 bytes)   made zombie
[2020-02-28T03:01:51.623+0000][debug][jit,compilation] 153192       4       io.lettuce.core.protocol.CommandArgs$StringArgument::writeString (60 bytes)   made zombie
[2020-02-28T03:01:54.911+0000][debug][jit,compilation] 157252   !   4       jdk.internal.reflect.GeneratedConstructorAccessor161::newInstance (49 bytes)
```

## C. 다른 런타임 관련

#### monitorinformation

동기화 lock과 관련된 로그, 데드락을 확인하기 위해 사용된다.

```java
[5.033s][debug][monitorinflation] Deflating object 0x0000000708310378 , mark 0x0000021cef446002 , type java.lang.ref.ReferenceQueue$Lock
[5.033s][debug][monitorinflation] Inflating object 0x0000000708310378 , mark 0x0000021cf085c002 , type java.lang.ref.ReferenceQueue$Lock
[5.035s][debug][monitorinflation] Deflating object 0x0000000708310378 , mark 0x0000021cf085c002 , type java.lang.ref.ReferenceQueue$Lock
[5.035s][debug][monitorinflation] Inflating object 0x0000000708310378 , mark 0x0000021cef445e02 , type java.lang.ref.ReferenceQueue$Lock
```

#### biasedlocking

- Intrinsic Lock(Built-in Lock)의 동작 방식
    - Intrinsic Lock은 메모리에 저장되는 오브젝트 헤더의 데이터와 모니터 객체를 기반으로 동작한다.
    - ![lock](assets/img/object.png)
    - 오브젝트 헤더의 가장 첫 데이터인 Mark Word는 Intrinsic Lock과 Garbage Collection을 위한 정보를 저장하기 위해 사용된다.
        - Mark Word는 Biased와 Tag 필드의 데이터에 의해 5 가지 상태 중 하나를 표현하는데 Unlock, Biased, Light-Wight Locked, Heavy-Weight Locked, Marked For GC 상태이다.
        - 이 중에서 Biased(Biasable), Light-Weight Locked, Heavy-Weight Locked 상태가 Lock을 획득하였음을 표현하고 있다.
        - Biased, Light-Weight Locked, Heavy-Weight Locked 순서로 Lock 획득과 반환의 성능이 우수하며, 우리가 일반적으로 알고 있는 Monitor에 의한 전통적인 방식은 Heavy-Weight Lock을 의미한다.
        - 최대한 Biased Lock을 획득하려고 하지만, 그렇지 못할 경우(Lock Contention의 정도에 따라서) Light-Weight Lock, Heavy-Weight Lock 순서로 Lock을 Upgrade 하여 획득을 시도한다.
- info 레벨에서 baised lock에 대해 볼 수 있고, trace에서는 lock contention의 구체적인 내용까지 볼 수 있다.

```java
[7.273s][info ][biasedlocking] Revoking bias by walking my own stack:
[7.273s][info ][biasedlocking] Revoking bias of object 0x0000000711b1ca40, mark 0x000001c6d0acc905, type sun.net.www.protocol.jar.URLJarFile, prototype header 0x0000000000000105, allow rebias 0, requesting thread 0x000001c6d0acc800
[7.273s][info ][biasedlocking]   Revoked bias of object biased toward live thread (0x000001c6d0acc800)
[7.273s][trace][biasedlocking]    mon_info->owner (0x00000007022634d8) != obj (0x0000000711b1ca40)
[7.273s][trace][biasedlocking]    mon_info->owner (0x0000000711b200d8) != obj (0x0000000711b1ca40)
[7.273s][trace][biasedlocking]    mon_info->owner (0x0000000711b200d8) != obj (0x0000000711b1ca40)
[7.273s][trace][biasedlocking]    mon_info->owner (0x0000000702970260) != obj (0x0000000711b1ca40)
[7.273s][info ][biasedlocking]   Revoked bias of currently-unlocked object
```

<br>
<br>
<br>
<br>

# 2. JVM 로그 설정

Configuration Format:

```java
-Xlog[:[what][:[output][:[decorators][:output-options[,...]]]]]
```

아무것도 설정되어있지 않으면 기본은 아래와 같다.

```java
-Xlog:all=warning:stdout:uptime,level,tags
```

":"콜론으로 구분되는 설정이다.
- 내용:로그 출력:decorator:output-option 와 같이 구성되어 있다.

설정되지 않은 값들은 기본값으로 사용되기 때문에, 아래 설정은 서로 같은 설정이다.
- -Xlog:all=warning and - Xlog::stdout and - xlog::: uptime, level, tags and - Xlog:all=warning:stdout and - Xlog::stdout:uptime,level,tags and - Xlog:all=warning:stdout:uptime,level,tags
- -Xlog:gc*=info and - Xlog:gc*=info:stdout:uptime,level,tags
- -Xlog::file=/project/log/app.log::filecount=50,filesize=100M and - Xlog:all=warning:file=/project/log/app.log:uptime,level,tags:filecount=50,filesize=100M

<br>
<br>

## A. 무엇을

tag, log level은 아래와 같이 설정 가능하다.

### 실제 적용시 해설, In practice

- -Xlog:gc=info, info level 이상의 gc 태그의 로그만 로그에 포함하겠다는 의미.
- -Xlog:gc*=info, info level 이상의 모든 gc와 관련된 태그를 포함하겠다는 의미
- -Xlog:gc+age=debug, debug level 이상의 gc와 age 태그 모두를 포함하는 로그를 포함하겠다는 의미
- -Xlog:gc*=info,gc+heap=debug,gc+heap+region=debug, info level 이상의 모든 gc 태그 로그는 포함하지만, gc,heap과 gc,heap,region인 label은 debug를 포함하겠다는 의미
- -Xlog:gc*=info,age*=debug / -Xlog:gc*=info,gc+age=debug 앞의 2개의 설정은 동일하다. 이런 설정은 JVM이 로그 설정을 합쳐서 하나로 만든다.

### 로그 레벨

- off: shut down
- Trace: including trace, debug, info, warning and error logs
- Debug: including debug, info, warning, error
- Info: including info, warning, error
- Warning: including warning, error
- Error: error only

아무 레벨도 없으면 기본값은 info이다. - Xlog:gc* / -Xlog:gc*=info 앞의 두 설정은 같다.

### 태그가 틀린 경우

태그가 잘못(모두 있는 태그인 경우) 설정된 경우, 이런 경우에는 에러가 나고 종료된다.

-Xlog:gc+phase=debug 두 태그 모두 존재하지만, gc하위에 phase는 없다.

```java
[0.005s][error][logging] Invalid tag 'phase' in log selection. Did you mean 'phases'?
Invalid -Xlog option '-Xlog:gc+phase=debug', see error log for details.
```

전혀 없는 태그가 선택된 경우에는 에러 로그만 나오고, 계속 진행된다.

add태그는 상위 태그에도 없고, phase는 하위 태그에만 있다.

```java
[0.006s][warning][logging] No tag set matches selection: gc+add. Did you mean any of the following? gc* gc+metaspace* gc+ref* gc+stringtable gc+compaction
[0.007s][warning][logging] No tag set matches selection: phases. Did you mean any of the following? phases* gc+phases* gc+phases+start* gc+phases+task gc+phases+ref
```

## B. 로그 출력

3가지 종류가 있다.
- stdout: standard output
- stderr: standard error output
- file=filename output to file
    - file로 저장하는 경우에는 filecount=50,filesize=100M 50개 파일까지 각각의 파일의 크기는 100메가로 제한할 수 있다.


## C. Decorators

|       sign         |                             의미                             |
|:------------------:|:---------------------------------------------------------------:|
| time or t          | 현재 시간, ISO-8601 format                                   |
| utc me or utc      | UTC time                                                        |
| uptime or u        | Startup부터의 시간, milliseconds 정도로 정확하다.             |
| timemillis or tm   | Millisecond timestamp, System.currentTimeMillis()와 같다 |
| uptimemillis or um | Startup부터의 miliseconds로 표시된 시간                            |
| timenanos or tn    | Nanosecond timestamp, System.nanoTime()과 같다           |
| uptimenanos or un  | Startup부터의 Nanosecond로 표시된 시간                           |
| hostname or hn     | Host name                                                       |
| pid or p           | Process number                                                  |
| tid or ti          | Thread number                                                   |
| level or l         | log level                                                       |
| tags or tg         | Log label, 앞에서 다뤘던 log label하고 같은 것           |

uptime, level, tags가 설정되면 아래와 같게 나온다.

```java
[2020-02-26T08:34:12.823+0000][debug][gc,age         ] gc(1661) Desired survivor size 167772160 bytes, new threshold 6 (max threshold 6)
```

## D. 이전 버전의 로그 설정을 새 버전의 로그 설정으로 변환하기

GC 관련: 설명은 앞서 내용을 참조하면 됨

|                         이전 버전 parameter                         |                                             새 버전 parameter                                            |
|:------------------------------------------------------------------:|:--------------------------------------------------------------------------------------------------------------------------------:|
| g1PrintHeapRegions                                                 | -Xlog:gc+region=trace                                                                                                            |
| gcLogFileSize and numberofglogfiles and UsegcLogFileRotation       | 위에 나오는 outfile 설정을 하면 됨                                                                             |
| PrintTenuringDistribution                                          | -Xlog:gc+age*=level                                  |
| PrintAdaptiveSizePolicy                                            | -Xlog:gc+ergo*=level                             |
| Printgc                                                            | -Xlog:gc=info or -Xlog:gc, gc 태그만 있는 로그 출력                                                               |
| PrintgcDetails                                                     | -Xlog:gc*=info or -Xlog:gc*                                                                                                     |
| PrintgcApplicationConcurrentTime and PrintgcApplicationStoppedTime | -Xlog:safepoint=log or - Xlog:safepoint. 자바의 gc로 인한 stop-the-world에 관한 로그를 출력  |
| PrintgcTaskTimeStamps                                              | -Xlog:gc+task*=debug                                                                                                             |
| PrintHeapAtgc                                                      | -Xlog:gc+heap=trace                                                                                                              |
| PrintReferencegc                                                   | -Xlog:gc+ref*=debug                                                                                                              |
| PrintStringDeduplicationStatistics                                 | -Xlog:gc+stringdedup*=debug                                                                                                      |
| PrintgcDateStamps                                                  | 데코레이터를 이용해서 설정가능, time or t를 사용하면 된다.                             |
| PrintgcCause and PrintgcID                                         | 기본적으로 프린트되기 때문에 따로 설정이 필요하지 않다.                            |

<br>
<br>

다른 파라미터들:

|     이전 버전 parameter    |                                               새 버전 parameter                                              |
|:-------------------------:|:------------------------------------------------------------------------------------------------------------------------------------:|
| TraceExceptions           | -Xlog:exceptions=info, info 레벨 이상의 JVM에 의한 error exception 로그를 나타낸다. 기본 값은 error 레벨만 나타난다.                           |
| TraceClassLoadingPreorder | -Xlog:class+preorder=debug                                                                                                           |
| TraceClassLoading         | -Xlog:class+load=info, 클래스 로딩 로그를 나타낸다. info 로그면 충분하다.                                                                     |
| TraceClassUnloading       | -Xlog:class+unload=info, 클래스 언로딩 로그를 나타낸다.                                           |
| TraceClassLoadingPreorder | -Xlog:class+preorder=debug                                                                                                           |
| TraceClassInitialization  | -Xlog:class+init=info                                                                                                                |
| TraceClassResolution      | -Xlog:class+resolve=debug                                                                                                            |
| TraceClassPaths           | -Xlog:class+path=info                                                                                                                |
| TraceLoaderConstraints    | -Xlog:class+loader+constraints=info                                                                                                  |
| VerboseVerification       | -Xlog:verification=info                                                                                                              |
| TraceSafepoint            | -Xlog:safepoint=debug                                                                                                                |
| TraceSafepointCleanupTime | -Xlog:safepoint+cleanup=info                                                                                                         |
| TraceMonitorInflation     | -Xlog:monitorinflation=debug                                                                                                         |
| TraceBiasedLocking        | -Xlog:biasedlocking=level, 위에 나오는 biasedlocking 을 참고하면 된다. |
| TraceRedefineClasses      | -Xlog:redefine+class*=level                                                                                                          |


<br>
<br>
<br>
<br>

# 3. Dynamic하게 JVM Log Level을 조정하기

jcmd 커맨드를 이용하면 런타임 상황에서도 JVM log 설정을 조정할 수 있다.

main command는 VM.log이다.

만약 JVM 프로세스가 22번이라면, 로그 포맷을 아래 명령어로 볼 수 있다.

```java
jcmd 22 VM.log
``` 

결과:
```java
22
Syntax : VM.log [options]

Options: (options must be specified using the <key> or <key>=<value> syntax)
        output : [optional] The name or index (#<index>) of output to configure. (STRING, no default value)
        output_options : [optional] Options for the output. (STRING, no default value)
        what : [optional] Configures what tags to log. (STRING, no default value)
        decorators : [optional] Configures which decorators to use. Use 'none' or an empty value to remove all. (STRING, no default value)
        disable : [optional] Turns off all logging and clears the log configuration. (BOOLEAN, no default value)
        list : [optional] Lists current log configuration. (BOOLEAN, no default value)
        rotate : [optional] Rotates all logs. (BOOLEAN, no default value)
```

### 현재 설정된 내용 보기

만약에 22번 프로세스를 아래의 내용의 옵션으로 실행했다면:

```java
-Xlog:gc*=debug:file=/project/log/gc.log:utctime,level,tags:filecount=50,filesize=100M 
-Xlog:jit+compilation=debug:file=/project/log/jit_compile.log:utctime,level,tags:filecount=10,filesize=100M
```

```java
jcmd 22 VM.log list
```

결과:

```java
22:

Log output configuration:
 #0: stdout all=warning uptime,level,tags
 #1: stderr all=off uptime,level,tags
 #2: file=/project/log/gc.log all=off,gc*=debug utctime,level,tags filecount=50,filesize=100M
 #3: file=/project/log/jit_compile.log all=off,jit+compilation=debug utctime,level,tags filecount=10,filesize=100M
```

아래의 2개 라인이 JVM에서 기본값으로 입력한 값이다.
```java
#0: stdout all=warning uptime,level,tags / / represents the warn level log of all tags in standard output, with the format of [uptime][level][tags] log content
#1: stderr all=off uptime,level,tags / / indicates that no log is output in standard error output
```

`#2`, `#3`은 앞서 입력한 파라미터와 같다.

### 로그 rotate도 가능

```java
jcmd 22 VM.log rotate
```

결과:

```java
22:
Command executed successfully
```

### 모든 로그 사용하지 않고, 로그 관련 파라미터 없애기

```java
jcmd 22 VM.log disable
```

결과:

```java
22:
Command executed successfully
```

```java
jcmd 22 VM.log list
```

```java
22:

Log output configuration:
 #0: stdout all=off uptime,level,tags
 #1: stderr all=off uptime,level,tags
```

### 새 로그 output 설정

```java
jcmd 22 VM.log output=/project/core/log/gc.log output_options="filecount=50,filesize=100M" decorators="utctime,level,tags" what="gc*=debug"
```

```java
22:
Command executed successfully

jcmd 22 VM.log list

22:

Log output configuration:
 #0: stdout all=off uptime,level,tags
 #1: stderr all=off uptime,level,tags
 #2: file=/project/core/log/gc.log all=off,gc*=debug utctime,level,tags filecount=50,filesize=100M (reconfigured)
```

`(reconfigured)`로 나온다.

### 로그 output 설정 수정

수정할 경우에는 output으로만 구분가능하다. output은 수정이 불가능하다. output 수정하고 싶다면 disable하고 다시 설정하는 방법밖에 없다.

앞서 설정된 내용을 info level로 수정하는 커맨드

```java
jcmd 22 VM.log output=/project/core/log/gc.log what="gc*=info"
```

결과:

```java
22:
Command executed successfully

jcmd 22 VM.log list
22:

Log output configuration:
 #0: stdout all=off uptime,level,tags
 #1: stderr all=off uptime,level,tags
 #2: file=/project/core/log/gc.log all=off,gc*=info uptime,level,tags filecount=50,filesize=100M (reconfigured)
```

label의 머지는 자동으로 된다.

만약에 더 적은 범위의 설정을 하려고 한다면,

```java
jcmd 22 VM.log output=/project/core/log/gc.log what="gc+age=info"
```

결과는 변동없이 머지된 상태이다.

```java
22:
Command executed successfully

jcmd 22 VM.log list

22:

Log output configuration:
 #0: stdout all=off uptime,level,tags
 #1: stderr all=off uptime,level,tags
 #2: file=/project/core/log/gc.log all=off,gc*=info uptime,level,tags filecount=50,filesize=100M (reconfigured)
```

만약 레벨이 다른 설정이 아래와 같이 들어간다면 정상적으로 입력된다.

```java
jcmd 22 VM.log output=/project/core/log/gc.log what="gc+age=debug"
```

```java
22:
Command executed successfully

jcmd 22 VM.log list

22:

Log output configuration:
 #0: stdout all=off uptime,level,tags
 #1: stderr all=off uptime,level,tags
 #2: file=/project/core/log/gc.log all=off,gc*=info,age*=debug uptime,level,tags filecount=50,filesize=100M (reconfigured)
```

```java
jcmd 22 VM.log output=/project/core/log/gc.log what="gc+alloc+region=debug"

22:
Command executed successfully

jcmd 22 VM.log list

22:

Log output configuration:
 #0: stdout all=off uptime,level,tags
 #1: stderr all=off uptime,level,tags
 #2: file=/project/core/log/gc.log all=off,gc*=info,age*=debug,alloc+region*=debug uptime,level,tags filecount=50,filesize=100M (reconfigured)
```

```java
jcmd 22 VM.log output=/project/core/log/gc.log what="gc+heap=debug,gc+heap+region=debug"

22:

Command executed successfully

jcmd 22 VM.log list

22:

Log output configuration:
 #0: stdout all=off uptime,level,tags
 #1: stderr all=off uptime,level,tags
 #2: file=/project/core/log/gc.log all=off,gc*=info,age*=debug,region*=debug,gc+heap=debug,gc+region=info uptime,level,tags filecount=50,filesize=100M (reconfigured)
```

머지가 필요한 경우에는 머지하고 필요하지 않은 경우에는 해당 옵션을 잘 추가해준다.

<br>
<br>
<br>
<br>

출처
- https://programmer.group/analysis-and-use-of-the-log-related-parameters-of-openjdk-11-jvm.html
- https://bestugi.tistory.com/m/40