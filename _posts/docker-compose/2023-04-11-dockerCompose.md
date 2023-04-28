---
title: "docker compose 실습 정리"
excerpt: "도커 컴포즈"

categories:
  - Data Enginnering
tags:
  - ["docker", "docker compose"]

date_created: 2023-04-11T18:27:35+09:00
last_modified_at: 2023-04-11T18:27:59+09:00
toc: true
toc_sticky: true
---

원티드 백엔드 챌린지를 통해 도커에 대해 복습하는 시간을 가지고 있다. 사실 도커 잘 알고 있다고 생각했는데, 막상 업무 할 때 잘 쓰지 않아서 까먹은게 많은 것 같다.
그래서 도커 컴포즈를 이용해 ELK를 띄우는 실습을 진행해봤다. 진작에 했으면 선물도 받았을텐데 괜히 늑장 피우다가 늦게나마 제출했다.

# ELK Dockerfile
먼저 세 개의 컨테이너를 실행 시킬 건데, 싱글 노드 es, kibana, logstash를 컨테이너로 띄울거다. 그래서 각각의 Dockerfile을 만들어야 한다.
도커파일은 아주 간단하게 어디서 이미지를 끌어올 것인지만 적었다.

```
# elasticsearch.Dockerfile
FROM docker.elastic.co/elasticsearch/elasticsearch:7.17.9
```
kibana, logstash는 각각 해당 부분만 수정해줬다.

# 각 컨테이너의 설정파일
오랜만에 ELK 설정을 보면서 가물가물한 기억을 떠올렸다. 설정파일에는 각 컨테이너가 연결될 수 있도록 클러스터 이름이나, elasitcsearch의 호스트 이름 및 포트 등을 적어주었다.

```
# elasticseach.yml
cluster.name: "es-docker-cluster"
network.host: 0.0.0.0

# logstash.yml
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: [ "http://elasticsearch:9200" ]

# kibana.yml
server.name: kibana
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://elasticsearch:9200"]
```

그리고 로그스태시의 파이프라인 정의를 위한 conf파일을 간단하게 추가해줬다.
```
input {
	tcp {
		port => 5001
	}
}


output {
	elasticsearch {
		hosts => "elasticsearch:9200"
		user => "elastic"
		password => "changeme"
		index => "elk-logger"
	}
}
```

# docker-compose.yml
마지막으로 도커 컴포즈 파일을 작성해야 하는데, 지금까지는 누군가 작성한 컴포즈 파일만 가져다가 사용했던 것 같다. 직접 작성하려니 생각보다 버벅였다. 전체적인 개념 자체는 간단하지만 ELK에 대한 지식이 부족해서 그랬나..

전체적인 컴포즈 파일의 구성은 각 컨테이너 생성을 위한 도커파일 위치, 각 소프트웨어의 설정파일을 복사하는 대신 볼륨을 지정하여 사용할 수 있도록 해주기, 포트포워딩, 환경변수 설정, 네티워크 연결. 이 정도로 요약할 수 있을 것 같다.

```
version: '3.2'

services:
  elasticsearch:
    build:
      context: "${PWD}/ELK/elasticsearch/"
      dockerfile: elasticsearch.Dockerfile

    volumes:  # Long Syntax 
      - type: bind
        source: "${PWD}/ELK/elasticsearch/config/elasticsearch.yml"
        target: /usr/share/elasticsearch/config/elasticsearch.yml
        read_only: true
      - "${PWD}/ELK/elasticsearch/data:/usr/share/elasticsearch/data"  # data 저장 공간
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      ES_JAVA_OPTS: "-Xmx256m -Xms256m"
      ELASTIC_PASSWORD: password
      discovery.type: single-node  # 데모를 위한 싱글 노드 클러스터
    networks:
      - elk

  logstash:
    build:
      context: "${PWD}/ELK/logstash/"
      dockerfile: logstash.Dockerfile

    volumes:
      - type: bind
        source: "${PWD}/ELK/logstash/config/logstash.yml"
        target: /usr/share/logstash/config/logstash.yml
        read_only: true
      - type: bind
        source: "${PWD}/ELK/logstash/pipeline"
        target: /usr/share/logstash/pipeline
        read_only: true
    ports:
      - "5001:5001/tcp"
      - "5001:5001/udp"
      - "9600:9600"
    environment:
      LS_JAVA_OPTS: "-Xmx256m -Xms256m"
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    build:
      context: ELK/kibana/
      dockerfile: kibana.Dockerfile

    volumes:
      - type: bind
        source: "${PWD}/ELK/kibana/config/kibana.yml"
        target: /usr/share/kibana/config/kibana.yml
        read_only: true
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      - elasticsearch

networks:
  elk:
    driver: bridge
```

작성하면서 CLI에서는 볼륨 바인딩 시 콜론을 기준으로 앞뒤로 경로를 넣었는데, 컴포즈 파일에서도 short syntax로 그렇게 쓰면 된다. 하지만 경로가 좀 길어지니 보기 힘들어서 long syntax로 사용하는 것이 더 깔끔해 보였다. 그리고 또 하나 아직 의문인 점인데, 볼륨 바인딩에서 target path에 따옴표를 씌우면 경로를 찾을 수 없다는 에러가 나왔다. 이 부분은 아직 왜 그랬는지 이유를 모르겠다.

그리고 환경 변수에 es와 logstash의 실행을 위한 힙메모리 설정을 추가해놨다. 


# 결과 확인
```
docker compose up -d --build
```
![](/assets/img/2023-04-11-dockerCompose/elasticsearch.png)
![](/assets/img/2023-04-11-dockerCompose/logstash.png)
![](/assets/img/2023-04-11-dockerCompose/kibana.png)

위에서부터 순서대로 elasticsearch - logstash - kibana가 정상적으로 실행된 것을 볼 수 있다.


---

### Tip! docker compose down vs stop
> stop : 말그대로 container를 정지  
> down : container와 network를 정지 후 삭제

클린하게 down으로 모두 없애고 다시 compose up을 하는게 (이 데모에서는) 좋아보인다.
각자 상황에 맞게 stop만 할지 삭제까지 할지 선택해서 사용하면 될 것 같다.