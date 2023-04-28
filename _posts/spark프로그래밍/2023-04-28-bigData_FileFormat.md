---
title: "빅데이터를 위한 파일 포맷"
excerpt: "parquet? avro?"

categories:
  - Data Enginnering
tags:
  - ["Spark", "Python", "Big Data"]

date_created: 2023-04-28T18:27:35+09:00
last_modified_at: 2023-04-28T18:27:59+09:00
---

## 데이터 저장 포맷
![](/assets/img/spark%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/2023-04-28/file_format.png)
일반 text파일은 데이터 분석시 완전히 비구조화된 자료이므로 거의 사용하지 않을 것이다. (raw data가 아니라면) 지금까지 본 파일 형식중 가장 많은 것은 Semi-structured가 아닐까 생각한다. 웹개발이나 작은 데이터분석을 하면 항상 볼 수있는 json, csv 등이 그렇다.
하지만 빅데이터를 다루면서, 특히 하둡에서는 기본 파일 포맷으로 parquet를 사용하며 parquet 외에도 orc, avro 등의 형식을 사용할 수 있다.

Semi-structured와 Structured의 가장 큰 차이는 사람이 읽을 수 있느냐? 일 것이다. parquet와 같은 구조화된 파일은 바이너리 형식이기 때문에 사람이 직접 읽을 수 없는 형태를 띄고 있다. 그리고 구조화된 데이터는 스키마를 포함하며 빅데이터를 다루기 위한 분산 환경에 최적화 되었기 때문에 여러개로 나뉘어질 수 있다. 따라서 확장성이 높고 분산처리도 가능한 것이다.

## Column 기반 저장 vs Row 기반 저장
![](/assets/img/spark%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/2023-04-28/%08row_column.png)
하나의 데이터프레임을 파일로 저장할 때 어떤 방향을 기준으로 정할 것인가에 따라 장점이 생기게 된다. row 기준으로 저장하는 경우 쓰기에 최적화 되어 있으며 모든 필드에 접근해야할 때 유리하다. 반면 column을 기준으로 저장하면 특정 필드를 찾아 접근하는 경우 유리하며 이러한 경우 읽기에 더 최적화되어 있다고 할 수 있다. 
row기반은 큰 분량을 쓸 때 유리하며 column 기반은 큰 분량을 읽어 분석할 때 유리하다고 할 수 있다.
따라서 상황에 따라 어떤 방식이 유리한지 판단해서 사용해야 할 것이다.
> Columnar : 읽기에 최적화, 압축률 good 하지만 전체 데이터 재구성 느림  
Row-oriented : 쓰기에 최적화, 계속해서 쓰는경우 (streaming)에 적절

### 각 파일 형식의 비교
![](/assets/img/spark%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/2023-04-28/comparison.png)
1. Arvo
    - row-based format
    - 스키마를 헤더 부분에 json으로 저장하여 human readable
    - 강력한 schema evolution 지원

2. ORC
    - columnar format
    - Hive에 최적화되어있음

3. Parquet
    - columnar format
    - Spark의 기본 파일 포맷
    - row의 그룹을 컬럼기준으로 모아서 저장
    - 일종의 하이브리드?

> Schema Evolution  
데이터의 컬럼이 자주 추가되거나 삭제되는 경우 json이나 csv형태에서는 스키마가 변한 파일들을 동시에 불러올 수는 없다. 하지만 Schema Evolution을 통해 이것을 가능하게 해준다. 위의 세 형식 모두 지원하지만 Avro가 가장 그쪽으로는 뛰어나다고 한다.

> Nested Columns  
데이터프레임 구성이 복잡하여 컬럼아래 또 컬럼이 있는 경우라면 nested column을 지원해야 한다. csv는 불가능하지만 json은 가능하다. Parquet와 Avro는 역시 지원한다. 