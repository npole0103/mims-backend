# mims-backend
Medical Information Monitoring Service - Backend

## 세나클 소프트 인턴 프로젝트

### 프로젝트 소개

#### 서비스명 : MIMS(Medical Information Monitoring Service)
![MIMS_logo](https://user-images.githubusercontent.com/37138188/187113565-99506f26-9a52-421b-b65a-644c0e5f5085.png)

#### 기간 : 22.07.25. ~ 22.08.31.

#### 인원 : 2명(Full-Stack)

#### 역할
- 백엔드 개발

#### 느낀점
- 테스트 코드 중요성 -> 데이터 PoC
- 의사소통의 중요성 -> 이해하는 부분이 달라서 구현에 문제가 생기는 경우
- 코드 모듈화의 중요성 -> 테스트 코드 짤 때, 모듈화를 신경쓰지 않고 짰더니 결합도가 높아져서, 분리가 힘든 상황을 맞딱드림. 테스트 코드를 짜더라도, 모듈화를 잘하자.
- DB 엔티티 설계 -> 인덱싱을 안 걸꺼면 여러 칼럼들 Json으로 관리할 것인지? 등등
- 테스크 스케줄링 하기 -> 주기적으로 실행하고 하는 process로 테스크를 만들고 스케줄링으로 관리
rout
### 프로젝트 개요

EMR 서비스
- 가이드 지침 + 공단 지침 -> 데이터
- 청구나 심사가 오픈되어 있고 추가되거나 수정되는 부분을 계속 반영해야 함
- EX) 코로나 검사에 대한 공제가 새로 생김. 이것이 고지 형태로 나옴.
- 이런 변화된 부분들을 빠르게 서비스에 반영하는 것이 중요.
- 이런 고지들은 여러 기관 사이트에 분포되어 있음.

**각 사이트별 추가/변경되는 고지들을 모니터링 하는 서비스의 필요성**

### 기술
Backend
- Spring Boot(v.2.7.2)
- MySQL
- Spring JPA
- MapStruct
- QueryDSL
- Selenium

Frontend
- React.js
- Recoil
- Scss
- Axios
- React-icons
- Http-proxy-middleware

### 이슈 및 트러블 슈팅

1. URL 리다이렉션이 안되는 웹페이지
    1. HttpClient로 GET, POST 요청 직접 하는 방법
    2. 셀레니움에서 XPath 경로를 입력받아 순차 접속 하는 방법

2. 스케줄링, 모니터링 아이템에 따른 일정 주기로 크롤링 후 알림 기능
- Springframework ThreadPoolTaskScheduler 사용

3. Service 처리 로직
- boilerplate 참고 + 비지니스 로직은 서비스 단에서 처리

4. 파일 및 폴더구조
```
- business
    - service
- config
    - querydsl
    - scheduler
- controller
- domain
    - converter
    - dto
    - enumtype
    - mapper
        - decorator
    - model
- repository
    - custom
    - implementation
- utils
```

5. N:M 관계의 DB 모델 구조에서 Entity 내에 상속까지 쓰는 경우. 조회할 테이블을 알 수 없는 상황 MonitoringReceiver와 MonitoringItem이 ManyToOne / OneToMany로 연결되어 있는 상황에서
상속 받은 각 3개의 테이블을 N:M으로 매핑 불가. 모니터링 아이템 아이디를 3개를 만들어주던지, 모니터링 리시버를 3개를 만들어주던지 해야함.
- Spring 상속 전략으로 Single Table 전략 사용. 각 아이템에 대한 3개의 테이블을 싱글 테이블로 관리.

6. 추후 다른 사이트를 추가하더라도 무리 없게 공통적인 데이터에 한에 추출
    - 크게 3가지로 분류
    1. HTML Tag 분석
    2. Json Api 분석
    3. Content 분석

7. 모니터링 아이템 비교 알고리즘에 List 형태의 Post 데이터를 Map으로 치환하여 key 검색
- Map으로 치환하여 find(containsKey) 할 때 O(1)의 시간복잡도

8. deleteAll 메소드 쿼리 30번 vs deleteAllInBatch 메소드 쿼리문 1개 성능 최적화
- deleteAll 쿼리문 대신 deleteAllInBatch 사용하여 쿼리 최적화

9. 메소드 분리 전략. 비지니스 로직 공통 부분 리펙토링 진행
- 테스트 케이스 작성 후 공통으로 사용되는 로직을 메소드화 하여 재사용

10. check 로직 내부의 sendMessage()를 MessageDto를 생성해 메소드 분리
- check 메소드와 send 메소드 2개로 분리
- check 메소드는 순수하게 새로운/수정된 글만 체킹
- send 메소드는 check에서 도출된 MessageDto를 기반으로 슬랙 메시지 전송

11. 테스트 코드 작성 시 Transactional + RollBack DB 저장 이슈
- 테스트 코드는 Transactional이 붙어있어도 테스트 완료 직후 RollBack 되어 실제로는 저장되지 않음.
- `@Rollback(false)` 어노테이션을 사용하여 이를 해결

12. 유틸 클래스 vs 서비스 클래스
- https://hamryt.tistory.com/2
- https://mygumi.tistory.com/253

**JAVA Collection Time Complexity**
- https://www.grepiu.com/post/9

**Inheritance Single-Table 전략**
- https://browndwarf.tistory.com/54
- https://tech.junhabaek.net/hibernate-jpa-%EC%83%81%EC%86%8D-%EC%A0%84%EB%9E%B5-inheritance-strategy-%EC%9D%98-%EC%9B%90%EB%A6%AC%EC%99%80-%EC%98%88%EC%8B%9C-1132428cc905#bb57


**싱글테이블 전략 사용시 레포지토리 설정**
- https://stackoverflow.com/questions/63656497/jpa-repository-with-single-table-inheritance-hibernate
- https://stackoverflow.com/questions/43350397/spring-data-jpa-inheritance-in-repositories

**셀레니움 초기 세팅 for Java**
- https://velog.io/@joyoo1221/%EC%9E%90%EB%B0%94-%EC%85%80%EB%A0%88%EB%8B%88%EC%9B%80selenium%EC%9C%BC%EB%A1%9C-%ED%81%AC%EB%A1%A4%EB%A7%81%ED%95%98%EA%B8%B0-1

**headless 옵션 시에 maximize 작동 안함**
- headless는 어떤 창을 최대화 해야할지 모르기 떄문에 maximize 함수가 작동하지 않아, notInteractable 오류 발생
- https://tousu.in/qa/?qa=517883/selenium-not-able-to-maximize-chrome-window-in-headless-mode

**문자열 자르기(substring, split)**
- https://coding-factory.tistory.com/126

**다중 공백 정규표현식**
- https://paranwater.tistory.com/493

**HttpClient**
- https://digitalbourgeois.tistory.com/58
- https://www.techiedelight.com/ko/send-http-post-request-java/
- https://helloworld92.tistory.com/3 한글깨짐 현상 해결
- https://stackoverflow.com/questions/12059278/how-to-post-json-request-using-apache-httpclient JSON payload 전달법


**스프링 테스크 스케줄러 총정리**
- https://m.blog.naver.com/adamdoha/222056799681
- https://technicalsand.com/spring-task-scheduler-examples/#6-dynamically-schedule-spring-tasks-to-run-later

**테스트 코드에서 LazyInitializationException**
- https://velog.io/@ohjinseo/%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%BD%94%EB%93%9C%EC%97%90%EC%84%9C-getOne-%EB%98%90%EB%8A%94-getById%EC%93%B8-%EC%8B%9C-LazyInitializationException-could-not-initialize-proxy-no-Session

**테스트 코드에서 트랜젝션 사용시 save 안됨**
- 이유 : 테스트를 위해 열린 트랜잭션은 테스트 코드 끝나면 자동으로 롤백됨. 영속성 컨텍스트 세션(영속성 컨텍스트를 관리하는 엔티티 매니저)이 유지 안됨.
- https://stackoverflow.com/questions/70908330/spring-data-jparepository-saveandflush-not-working-in-transactional-test-method
- https://stackoverflow.com/questions/43658630/spring-boot-test-transactional-not-saving

**테스트 코드 AssertThat**
- https://hseungyeon.tistory.com/328

**서비스 vs 유틸리티 유틸화**
- https://stackoverflow.com/questions/7270681/utility-class-in-spring-application-should-i-use-static-methods-or-not
- https://soranta.tistory.com/3
- https://groups.google.com/g/ksug/c/K2BT414VKtU

**HTTP 상태 코드**
- https://sanghaklee.tistory.com/61

**@Transactional**
- 트랜잭션 : 더 이상 쪼갤 수 없는 최소 작업 단위
- 트랜잭션은 commit 되거나, rollback으로 실패 이후 취소되어야 함.
- https://mangkyu.tistory.com/170
- https://heekim0719.tistory.com/409

**AOP(Aspect Oriented Programming) -관점 지향 프로그래밍**
- https://velog.io/@gillog/AOP%EA%B4%80%EC%A0%90-%EC%A7%80%ED%96%A5-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D
- https://greendreamtrre.tistory.com/601

**JPA 양방향 순환 참조**
- https://dev-coco.tistory.com/133
- https://velog.io/@minchae75/Spring-Boot-JPA-%EC%88%9C%ED%99%98-%EC%B0%B8%EC%A1%B0-%ED%95%B4%EA%B2%B0

**Mapper + queryDSL -> 매핑 효율적으로, 대용량 데이터 서치**
- https://github.com/ces518/TIL/blob/master/conference/%EC%9A%B0%EC%95%84%EC%BD%982020/%EC%88%98%EC%8B%AD%EC%96%B5%EA%B1%B4%EC%97%90%EC%84%9C_QUERYDSL_%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0.md
- https://jessyt.tistory.com/72
- https://jaehoney.tistory.com/212
