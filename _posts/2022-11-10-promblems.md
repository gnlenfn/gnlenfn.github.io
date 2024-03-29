---
title: "장애대응 간접체험하기"
excerpt: ""

categories:
  - review
tags:
  - ["후기", "장애대응", "실무"]

date_created: 2022-11-10T00:21:22+09:00
last_modified_at: 2022-11-10T00:18:22+09:00
toc: true
toc_sticky: true
---

# 사내 개발 환경 장애 대응 간접 경험기
오늘은 서비스 운영 중에 일어날지도 모르는 일을 간접적으로 경험한 것 같다. 우선, 오늘 있었던 일은 개발 환경의 어떤 VM이 리소스를 과하게 잡아먹고 있었고 (특히 디스크 사용량) 이로 인해 아마도 엘라스틱 서치가 뭔가 있다 라는 추측을 했고 그 원인을 찾아야 했다. 하지만 전체 리소스 사용량의 95%가 넘어가면 언제 다운될지 모르는 상태가 되기 때문에 좀 다급한 상태였다. 

### 개발 환경 구성과 나의 역할?
나는 고작 신입이지만 이 일과 무슨 관련이 있었나? 바로 엘라스틱 서치를 담당했던 팀에 몸담았었기 때문이다. (과거형인 이유는 그 팀은 모두 퇴사하고 나도 다른 팀으로 옮겨 다른 업무를 하게됐기 때문.) 하지만 내가 알기로 사내 개발환경에서 사용하는 엘라스틱 서치는 2개의 클러스터가 있고 하나는 가상환경에 있지만 shutdown 된 상태였고 나머지 하나는 물리 서버에 직접 설치했기 때문에 가상환경 구성에 영향을 끼치지 않았다. 그런데 조사 결과를 보니 shutdown 된 3개의 VM이 전체 사용량 중 20%를 차지하고 있었다. 그래서 지금 당장 급한 리소스 사용량 감소를 위해서는 해당 VM을 삭제하기로 잠정 결정은 됐다. 해당 VM은 10월 중순부터 shutdown 됐고 사용중인 팀은 거의 없었기 때문에 데이터 삭제가 아닌 VM 자체를 완전히 삭제하기로 했다.

### 그럼 해결 된건가?
일단 지금 당장 급한 불은 껐다 라고는하는데 사실 나는 잘 모르겠다. 우선 해당 엘라스틱 서치의 디스크 사용량이 그렇게 많아지 이유는 Index Lifecyle Management(ILM)이 제대로 작동하지 않았기 떄문이다. 이것은 여유가 있는 물리 서버에서도 마찬가지였다. 원래 회사에서는 데이터를 장기간 보관할 계획이 없었다고 한다. 나도 데이터 엔지니어로 입사 했지만 솔루션 개발로 자리를 옮긴 이유도 현재 회사에서는 딱히 데이터를 활용해 뭔가 더 나은 가치를 창출할 것 같지는 않았기 때문이다. 그런데 AI 연구 인력들이 생기고 AI 조직에서는 항상 많은 데이터를 필요로하고 요구하기 때문에 이전 팀에서도 데이터 저장기간을 무제한으로 늘려놓았던 것이다. 그 후로는 당연히 정리 되지 않는 데이터가 무자비하게 늘어나고 말았다.  
또 하나의 문제점은 수집되는 데이터가 내가 알고있던 것에 비해 더 많고 다양했다. 원래 엘라스틱 서치에 데이터를 저장하고 (현재 회사는 엘라스틱 서치를 Timeseres DB처럼 사용중이다) 사용하는 곳은 대시보드에 들어갈 데이터였다. 그래서 정해진 메트릭들과 일부 로그만 저장하고 있었다. 하지만 AI조직의 필요로 인해 훨씬 많은 데이터가 수집되고 있었고 새로 추가된 데이터들은 ILM 설정은 커녕 index alias도 설정이 없어서 어떻게 조회해서 사용중인 건지도 파악하기 힘들었다. (AI 조직과의 소통이 중요한데 우리 팀이 와해됐기 때문에...)

우선은 회사에서 데이터 조직을 새로 구성하고 있는 것으로 보인다. 그리고 거기엔 AI 조직도 참여하고 있고. 나는 그 TF에는 포함 안되어 있지만 인수인계 과정에 대한 질문 정도는 받게 되지 않을까 싶다. 

### 추후 대책은?
일단 VM 삭제로 리소스가 얼마나 여유가 생겼는지는 아직 모르겠다. 디스크 사용량 20%가 고작 VM 3개가 차지하고 있던게 물론 크긴 한데 전체 사용량이 어느정도가 유지되어야하는지 잘 몰라서 20% 확보가 큰 건지 감이 안온다. (개인적인 PC 사용에서는 웬만하면 50%도 잘 안넘기는지라..) 내일 회사가면 여유가 있는건지 물어봐야할 것 같다.  
VM 삭제는 임시 방편이었고 결국은 데이터 수집과 저장에 대한 전략이 필요할 것이다. 내가 생각나는 것은 당연히 ILM을 통해 일정 기간 저장 후 데이터 삭제하는 방법일 것이다. 그런데 AI조직은 당연히 그것에 반대를 할테고 아마 적절한 타협이 필요할 것이다. 그런데 저장 기간 보다도 수집 대상에 대한 논의가 더 필요해 보였다. 자세한 부분은 회사 내부 이야기 일테니 이 부분은 넘어가야겠다.  

---

장애 대응에 간접 경험이라 한 이유는 사실 장애라기엔 서비스에 문제가 크지는 않은 사내 개발 환경이었고 (물론 다운되면 큰일이겠지만..) 내 업무였지만 현재는 아닌 파트에서 벌어졌기 때문에 직접 적극적으로 대응하지는 않았기 때문이다. 특히 이후 대책에 대해 논의하는 부분이 이번 일의 가장 핵심이 될 것 같은데 아마 나는 그 TF에는 참여하지 않게될 것 같다. 이건 좀 아쉬운 부분. 이런거 하나하나가 면접때 풀 썰이 아니었나 싶은데 아쉽다!!!