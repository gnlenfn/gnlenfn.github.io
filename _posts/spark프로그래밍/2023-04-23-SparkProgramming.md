---
title: "스파크 프로그래밍과 SQL"
excerpt: "파이썬으로 해보는 Spark 프로그래밍"

categories:
  - Data Enginnering
tags:
  - ["Spark", "Python", "Big Data", "SQL"]

date_created: 2023-04-23T18:27:35+09:00
last_modified_at: 2023-04-23T18:27:59+09:00
---

스파크 데이토 구조 중 pyspark는 DataFrame을 주로 사용한다고 했었다. 그리고 DataFrame을 사용해 데이터를 다루는 실습도 했는데, 일부 경우는 오히려 그냥 SQL을 사용하는 것이 더 간단하고 직관적이었다.
빅데이터가 대두된 이후 처음에는 이제 SQL의 시대는 끝이다 라는 말도 있었다지만 결국 구조화된 데이터를 다루는 데에는 SQL만한 것이 없다는 결론이 났다는게 한기용 님의 말씀이었다. Hive, Presto, BigQuery 등 모두 SQL을 기반으로 데이터를 다룰 수 있게 해주기 때문에 SQL 공부도 필수적이라는 이야기였다. 

그래서 오늘은 자주 사용할 SQL 함수들에 대해 정리해두려고 한다.

## JOIN
조인은 SQL을 해봤다면 당연히 가장 유용하게 써야하는 기능인 것 같다. 특정 키를 기준으로 테이블들의 어떤 컬럼을 표현해줄 지 정할 수 있다.
그래서 뭐 LEFT JOIN, INNER JOIN 이런 것들에 대해 정리할 것은 아니고 스파크에서 JOIN을 쓸 때 생각할 점 한 가지만 정리한다.
### Shuffle JOIN, Broadcast JOIN
- Shuffle JOIN
    - 일반적인 조인 방식이다.
    - 조인을 수행하면 같은 조인키를 가진 레코드들이 같은 파티션 내에 있어야한다.
    - 따라서 모든 데이터를 각 파티션들로 복사해야하기 때문에 대규모 shuffle로 인해 비용이 많이 드는 조인이 될 수 있다

- BroadcastJOIN
    - 큰 데이터와 작은 데이터 간의 조인이다
    - 작은 데이터프레임을 나머지 데이터프레임이 있는 파티션으로 뿌려주는 방식

- 두 방식 중 어떤 방식이 더 좋을지는 테스트해봐야 알 수 있다.
- Spark SQL을 쓰면 최적화를 내부에서 대신 해준다고 하는데... (이부분은 더 공부해야 할 부분이다!)

## Window, Rank 함수 등 Aggregation

```SQL
-- 총 매출이 가장 많은 사용자 10명 찾기
    SELECT
        userid,
        SUM(amount) total_amout,
        RANK() OVER (ORDER BY SUM(amount) DESC) rank
    FROM session_transaction st
    JOIN user_session_channel usc ON st.sessionid = usc.sessionid
    GROUP BY userid
    ORDER BY rank
    LIMIT 10;
```

```SQL
-- 사용자별 처음 및 마지막 채널 찾기
SELECT DISTINCT A.userid,
    FIRST_VALUE(A.channel) OVER(partition by A.userid order by B.ts
rows between unbounded preceding and unbounded following) AS First_Channel,
    LAST_VALUE(A.channel) OVER(partition by A.userid order by B.ts 
rows between unbounded preceding and unbounded following) AS Last_Channel
FROM user_session_channel A
LEFT JOIN session_timestamp B
    ON A.sessionid = B.sessionid

-- ROW_NUMER 사용
WITH RECORD AS (
    SELECT
        userid,
        channel,
        ROW_NUMBER() OVER (PARTITION BY userid ORDER BY ts ASC) AS seq_first,
        ROW_NUMBER() OVER (PARTITION BY userid ORDER BY ts DESC) AS seq_last
    FROM user_session_channel u
    LEFT JOIN session_timestamp t
        ON u.sessionid = t.sessionid
)
SELECT 
    f.userid,
    f.channel first_channel,
    l.channel last_channel
FROM RECORD f
JOIN RECORD l ON f.userid = l.userid
WHERE f.seq_first = 1 and l.seq_last = 1
ORDER BY userid;
```

```SQL
-- 월별 채널별 총 방문자, 구매 방문자, 구매 전환율 계산
SELECT LEFT(ts, 7) month,
    usc.channel,
    COUNT(DISTINCT userid) uniqueUsers,
    COUNT(DISTINCT (CASE WHEN amount >= 0 THEN userid END)) paidUsers,
    SUM(amount) grossRevenue,
    SUM(CASE WHEN refunded is not True THEN amount END) netRevenue,
    ROUND(COUNT(DISTINCT CASE WHEN amount >= 0 THEN userid END) * 100
         / COUNT(DISTINCT userid), 2) conversionRate
    FROM user_session_channel usc
    LEFT JOIN session_timestamp t ON t.sessionid = usc.sessionid
    LEFT JOIN session_transaction st ON st.sessionid = usc.sessionid
    GROUP BY 1, 2
    ORDER BY 1, 2;
```