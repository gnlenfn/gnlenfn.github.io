---
title: "Map Reduce 프로그래밍"
excerpt: "파이썬으로 해보는 Spark 프로그래밍"

categories:
  - Data Enginnering
tags:
  - ["Spark", "Python", "Big Data", "Map Reduce", "Hadoop"]

date_created: 2023-04-05T22:27:35+09:00
last_modified_at: 2023-04-05T22:27:59+09:00

toc: true
toc_sticky: true
---

## Hadoop 1.0 & Hadoop 2.0
> An open source software platform for __distributed storage__ and __distributed processing__ of very large data sets on computer clusters built from commodity hardware

하둡은 평범한 스펙의 컴퓨터들의 클러스터를 가지고 분산 처리를 통해 대용량 데이터 처리를 지원하는 플랫폼이다. 마치 하나의 컴퓨터처럼 클러스터를 사용하는 것이다.
그리고 하둡 1.0은 분산 스토리지 시스템인 HDFS위에서 Map Reduce가 동작하면서 대용량 데이터를 처리하는 방식을 가지고 있었다. 하지만 맵리듀스 프로그래밍의 생산성이 딱히 높지 않았고 확장성에 제한이 있었기 때문에 하둡 2.0에서는 이 부분을 개선하는 변화를 겪는다.

하둡 2.0은 yarn이라는 리소스 매니저 레이어가 추가되어 범용 프레임워크로 발전하였다. 전체 적인 구성 자체는 1.0과 비슷하지만 각 노드에 컨테이너가 JVM이 되고 리소스 매니저는 각 노드의 노드 매니저를 통해 노드의 컨테이너를 받아 태스크를 실행하도록 하는 형태를 띄게 된다. 
![](/assets/img/spark%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/2023-04-05/hadoop2.png)
![](/assets/img/spark%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/2023-04-05/yarn.png)

Resource Manager(RM)은 전체 노드를 관리하는 역할, 각 Node Manager 중 한 개는 실행하는 어플리케이션을 관리하는 Application Manager(AM), 그리고 각 노드의 컨테이너를 담당하는 Node Manager(NM)으로 구성되어있다. 
1. RM이 클라이언트로부터 코드 받아옴
1. RM이 AM 선정
1. AM이 필요한 리소스 만큼 NM으로부터 컨테이너 받아옴
1. 각 태스크는 받아온 컨테이너에서 실행됨
1. 각 컨테이너가 잘 동작하고 있는지 AM에게 주기적으로 보고(heartbeat)

이때 하둡 1.0과 다르게 2.0에서 생긴 클러스터의 리소스 관리자를 yarn이라고 부른다.

## MapReduce
맵리듀스 프로그래밍은 맵과 리듀스로 이루어진 연산을 가리킨다. 맵리듀스가 다룰 수 있는 데이터 셋은 오로지 key-value 형식이며 immutable하다. 
![](/assets/img/spark%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/2023-04-05/MR.png)
 
### Map
HDFS 파일로 입력을 받으며 맵 연산의 내용을 개발자가 만들어주어야 한다. 입력받은 데이터 내용을 보고 (key, value) 쌍으로 이루어진 리스트를 만들어 반환한다.
입력과 동일한 페어를 그대로 출력해도 되고, 출력이 없어도 된다.
맵의 입력은 HDFS의 블록 한 개로 제한된다 (128MB)

### Shuffle
Mapper의 출력을 Reducer로 보내주는 프로세스. 이때 어느 Reducer로 보내줄 지 선택해야 하는데, 모듈러 계산 등을 통해 각 reducer로 결과를 보내준다. 이 과정에서 전송하는 데이터의 크기가 크거나 매우 많으면 네트워크 상의 과부하가 생기거나 병목이 생길 수 있다. 또한 일부 reducer에 많은 계산을 필요로하는 태스크가 몰릴 수 있는데 이런 경우를 skewed 됐다고 말한다.

### Sorting / Reduce
Reducer의 입력을 수신 받아 merge하는 과정. 같은 key를 가지는 것들 끼리 merge하는 것이다. 단어 세기 예시에서, {1: [1, 2, 1, 4], 2: [1, 1, 1]} 이런 식의 입력을 받으면 {1: 8, 2, 3} 이렇게 만들어 주는 것이다. 이렇게 만들어진 결과를 reducer의 입력으로 넣어준다. 그리고 그 결과를 최종적으로 다시 HDFS에 파일로 저장한다. 
Reduce의 입력은 reduce 태스크의 수나 어떤 값을 기준으로 맵이 key-value 페어를 만드냐에 따라 크기가 달라질 수 있다. 따라서 각 reducer들의 입력은 상이하게 다를 수 있고 이 경우 data skew가 발생했다고 한다.

위 과정을 여러번 반복하여 원하는 결과를 얻을 수 있도록 하는 것이 맵리듀스 프로그래밍. 따라서 한번 skew가 생기면 map의 입력에도 문제가 생길 수 있음. 

## MapReduce의 문제점
- 낮은 생산성
프로그래밍을 통해 원하는 결과를 얻기 위해 다양한 연산들이 존재하는데, 맵리듀스는 2가지 오퍼레이션만 존재하기 때문에 유연하지 못함. 또한 데이터 분포 등의 영향을 받아 튜닝이나 최적화가 쉽지 않음
- 배치 작업만 가능
하둡의 목적 자체가 스트리밍 데이터 처리 같이 데이터를 빠르게 처리하는 것 보다는 대용량의 데이터를 처리하는데 초첨이 맞춰져 있기 때문에 Latency보다는 Throughput을 중점적으로 처리하는 방식이다. 또한 모든 입출력을 디스크로 작업하기 때문에 큰 데이터를 다루기에 적합하고 작은 데이터를 빠르게 처리하는 데에는 부적합한 것
- Shuffling 후 Data Skew가 발생하기 쉬움
이 부분은 Spark 역시 가지고 있는 문제이며 항상 고려해야 할 문제이다. 그리고 개발자가 reduce의 수를 직접 결정해야 한다는 점 역시 큰 문제가 된다. 시스템이 알아서 적절하게 reduce 수를 정해준다면 skew 발생 상황이 더 줄어들 텐데..

## 대안의 등장
이미 앞서 언급했듯이 yarn이 MR의 한계를 극복하기 위해 나타났고 Spark역시 그 중 하나라고 한다. 그리고 구조화된 데이터의 경우 전통의 강자 SQL이 결국 좋은 결과를 보이기 때문에 Hive나 Presto같은 프레임워크가 등장하게 된다. (사실 둘다 안써봐서 잘 모름)

- Hive
     - MapReduce 위에서 구현. 대용량 ETL에 적합
- Prestro
    - Low Latency에 집중. 
    - Adhoc 쿼리에 적합
