## 7장 캐시

웹 캐시는 자주 쓰이는 문서의 사본을 자동으로 보관하는 HTTP 장치다. 웹 요청이 캐시에 도착했을 때, 캐시된 로컬 사본이 존재한다면, 그 문서는 원 서버가 아니라 그 캐시로부터 제공된다.

- 불필요한 데이터 전송을 줄여, 네트워크 비용 감소.
- 네트워크 병목 감소, 대역폭을 늘리지 않고 빠른 페이지를 응답.
- 원 서버 요청을 줄여 서버 부하 감소, 더 빠른 응답.
- 페이지를 먼 곳에서 불러 올 때 생기는 거리로 인한 지연 감소.

### 7.1 불필요한 데이터 전송

복수의 클라이언트가 자주 쓰이는 원 서버의 페이지에 접근시, 각 요청마다 같은 문서를 반복 전송하게 된다. 이런 불필요한 전송은 네트워크 대역폭을 잡아먹고, 전송을 느리게하며, 웹 서버에 부하를 준다. 캐시를 이용해 첫번째 응답을 캐시에 보관하고 뒤이은 요청에서는 캐시된 사본을 응답으로 사용하면 중복으로 트래픽을 주고 받는 낭비를 줄일 수 있다.

### 7.2 대역폭 병목

특히 큰 문서를 응답받아야 하는 경우, 보통 네트워크는 (WAN보다) 로컬 클라이언트(LAN)에 더 넓은 대역폭을 제공하기 때문에 빠른 LAN에 있는 캐시로부터 cached사본을 가져온다면 성능을 대폭 개선할 수 있다.
<br /><img width="366" alt="Screen Shot 2022-11-13 at 7 39 39 PM" src="https://user-images.githubusercontent.com/54028005/201517458-6e2a47dc-99b1-468c-83a9-0b50c26a563b.png">

### 7.3 갑작스런 요청 쇄도(Flash Crowds)

트래픽 급증은 네트워크와 웹 서버에 심각한 장애 및 과부하를 야기시키며, 이때 캐싱이 특히 중요하다.

### 7.4 거리로 인한 지연

대역폭이 문제가 되지 않더라도 거리가 문제가 될 수 있다. 모든 네트워크 라우터는 저마다 인터넷 트래픽을 지연시키며, 병렬(parallel)이면서 keep-alive 커넥션(persistent-connection)이라해도 광속 그 자체는 상당한 지연을 유발한다.

### 7.5 적중과 부적중 (Hits and Misses)

- 캐시에 요청이 도착했을 때 그에 대응하는 사본이 있다면 그를 이용해 요청이 처리될 수 있으며 이를 cache hit라고 한다.
- 반면, 사본이 없다면 원 서버로 요청이 전달될 뿐인데, 이를 cache miss라고 부른다.
- 원 서버 콘텐츠는 변경될 수 있기 때문에, 캐시는 반드시 사본이 최신상태로 유지되는지 서버를 통해 주기적으로 점검해야한다. 이런 최신상태 체크를 재검사(Revalidation)이라고 한다.
- 캐시는 스스로 원한다면 언제든 사본을 재검사 할 수 있다. 그러나 캐싱된 문서가 수백만 개되는데에 비해 네트워크 대역폭은 부족하기 때문에, 클라이언트가 사본을 요청했으며 그 사본이 검사할 필요가 있을 정도로 충분히 오래된 경우에만 재검사를 한다.
  - 재검사 적중 / 느린 적중 : 캐시는 사본의 재검사가 필요할 때, 원 서버에 재검사 요청을 보낸다. -> 콘텐츠가 변경되지 않았다면 304 not modified 응답을 보낸다. -> 그 사본이 여전히 유효함을 알게 된 캐시는 즉각 임시로 사본이 최신이라고 표시한뒤 클라이언트에 제공한다.
- 속도 비교 : 순수 캐시 적중(캐시사본을 바로 클라이언트에 전송) < 재검사 적중 / 느린 적중 (원서버와 검사를 해야함) < 캐시 부적중 (원서버로부터 문서객체 전체를 받아와야함. 200 ok, 삭제됐다면 404 not found)
- If-Modified-Since 헤더 : 서버에 보내는 GET 요청 헤더에 추가해 보내면 캐시된 이후에 변경된 경우에만 사본을 보내달라는 의미.
- 캐시 (문서)적중률 : 캐시가 얼마나 큰지, 캐시 사용자들의 관심사가 얼마나 비슷한지, 캐시된 데이터가 얼마나 얼마나 자주 변경되거나 개인화되는지, 캐시가 어떻게 설정되어있는지에 따라 달라진다. 적중률을 예측하기 어렵기로 악명높지만, 캐시는 유용한 콘텐츠가 캐시에 머무르도록 보장하기 위해 노력하며 성능 개선에 상당한 도움이 된다.
- 바이트 적중률 : 문서들은 다 다른 크기이며 큰 객체를 덜 접근되지만 전체 트래픽에는 더 크게 기여하기 때문에, 바이트 단위 적중률 측정값을 더 유의미하게 보기도 한다.
  - 문서 적중률은 얼마나 많은 웹 트랜잭션을 외부로 보내지 않았는지에 대한 지표로 이를 개선하면 전체 지연 대기 시간을 줄여주며, - 바이트 적중률은 얼마나 많은 바이트가 인터넷으로 나가지 않았는지에 대한 지표로서 대역폭 절약을 최적화해준다.
- 적중과 부적중의 구별 : HTTP는 클라이언트에게 캐시 적중 여부를 알려주지 않는다. 두 경우 모두 200OK. 클라이언트가 알아낼 수 있느 방법으로
  - via헤더에서 캐시에 무슨일이 일어났는지에 대한 부가 정보,
  - Date헤더의 응답 생성일 정보,
  - Age헤더의 응답이 얼마나 오래되었는지에 대한 정보를 이용할 수 있다.

### 7.6 캐시 토폴로지

- 캐시는 한 명의 사용자에게만 할당될 수도 있고(개인 전용 캐시) 반대로 수천 명의 사용자들 간에 공유될 수도 있다.(공용캐시)
- 개인 전용 캐시는 한 명의 사용자가 자주 찾는 페이지를 담고, 공공용 캐시는 사용자 집단에게 자주 쓰이는 페이지를 담는다.

#### 7.6.1 개인 전용 캐시

- 대부분의 브라우저는 개인 전용 캐시를 내장하고 있고, 자주 쓰이는 문서를 로컬의 디스크와 메모리에 캐시해두고 사용자가 캐시 사이즈와 설정을 수정할 수 있도록 허용
- 캐시에 담긴 데이터를 확인하는 것도 가능

#### 7.6.2 공용 프락시 캐시

- 공용 캐시는 캐시 프락시 서버 혹은 프락시 캐시라고 불리는 특별한 종류의 공유된 프락시 서버다.
- 프락시 캐시는 로컬 캐시에서 문서를 제공하거나 사용자 입장에서 서버에 접근
- 여러 사용자가 접근하기 때문에 불필요한 트래픽을 줄일 수 있는 더 많은 기회가 있다.

- 프락시 캐시는 프락시를 위한 규칙에 따른다.
- 수동 프락시를 지정하거나 프락시 자동설정 파일을 설정함으로써 브라우저가 프락시 캐시를 사용하도록 설정할 수 있다.
- 인터셉트 프락시를 사용함으로써 브라우저의 설정 없이 HTTP 요청이 캐시를 통하도록 강제할 수 있다.

#### 7.6.3 프락시 캐시 계층들

- 작은 캐시에서 캐시 부적중이 발생했을 때 부모 캐시가 걸러 남겨진 트래픽을 처리하도록 하는 계층구조를 말한다.
- 클라이언트 주위에는 작고 저렴한 캐시를, 계층 상단에는 많은 사용자들에 의해 공유되는 문서를 유지하는 크고 강력한 캐시를 사용하자는 것이다.
- 계층이 깊으면 요청은 캐시의 긴 연쇄를 따라가게 될 것이고, 이런 프락시 연쇄가 길어질 수록 현저한 성능 저하가 발생한다.

#### 7.6.4 캐시망, 콘텐츠라우팅, 피어링

- 어떤 네트워크 아키텍처는 복잡한 캐시망을 만든다.
- 캐시망 안에서 콘텐츠 라우팅을 위해 설계된 캐시들은 다음과 같은 일들을 할 수 있다.

> - URL에 근거해, 부모 캐시와 원 서버 중 하나를 동적으로 선택
> - URL에 근거해 특정 부모 캐시를 동적으로 선택
> - 부모 캐시에게 가기 전, 캐시된 사본을 로컬에서 검색
> - 다른 캐시들이 그들의 캐시된 콘텐츠에 부분적으로 접근할 수 있도록 허용하되, 그들의 캐시를 통한 인터넷 트랜짓은 허용하지 않는다.

- 이런 복잡한 캐시 사이의 관계는 서로 다른 조직들이 상호 이득을 위해 그들의 캐시를 연결해 서로를 찾아볼 수 있도록 한다.
- HTTP는 형제 캐시를 지원하지 않기 때문에, 인터넷 캐시 프로토콜(ICP)이나 하이퍼텍스트 캐시 프로토콜(HTCP) 같은 프로토콜을 이용해 HTTP를 확장했다.

### 7.7 캐시 처리 단계

> 1. 요청 받기
> 2. 파싱
> 3. 검색
> 4. 신선도 검사
> 5. 응답 생성
> 6. 발송
> 7. 로깅

1. 요청받기

- 캐시는 네트워크 커넥션에서의 활동을 감지, 들어오는 데이터를 읽어온다.
- 고성능 캐시는 여러개의 커넥션으로부터 동시에 데이터를 읽고 메시지 전체가 도착 전 트랜잭션 처리를 시작한다.

2. 파싱

- 캐시는 요청 메시지를 여러 부분으로 파싱해 헤더 부분을 조작하기 쉬운 자료구조에 담는다.

3. 검색

- 캐시는 URL을 알아내고 그에 해당하는 로컬 사본이 있느지 검사.
- 로컬 복사본은 메모리, 디스크 혹은 다른 컴퓨터에 있을 수도 있다.
- 전문적인 수준의 캐시는 객체를 로컬 캐시에서 가져올 수 있는지 판단하기 위해 빠른 알고리즘을 사용.
- 만약 문서를 로컬에서 가져올 수 없으면 상황이나 설정에 따라 원서버나 부모 프락시에서 가져오거나 혹은 실패를 반환.

4. 신선도 검사

- HTTP는 캐시가 일정 기간동안 서버 문서의 사본을 보유할 수 있도록 한다.
- 이 기간동안 문서는 '신선'한 것으로 간주, 캐시는 서버와 접촉없이 이 문서를 제공한다.
- 신선도 한계를 넘어설 정도로 오래 갖고 있으면 그 객체는 '신선하지 않은' 것으로 간주, 캐시는 문서 제공 전 문서에 어떤 변경이 있었는지 검사하기 위해 서버와 재검사.
- 신선도 검사 규칙 이 장 나머지의 대부분에 거쳐 설명.

5. 응답 생성

- 캐시된 응답을 원 서버에서 온 것 처럼 보이게 하기 위해 캐시는 캐시된 서버 응답 헤더를 토대로 응답 헤더를 생성.
- 캐시는 클라이언트에 맞게 이 헤더를 조정해야 하는 책임이 있따.
- 예를 들어, 클라이언트가 HTTP/1.1 응답을 기대하는 상황에서 서버가 HTTP/1.0 응답을 반환했다면 캐시는 반드시 헤더를 적절하게 번역해야 한다.
- 캐시 신선도 정보(Cache-Control, Age, Expires 헤더)와 요청이 프락시 캐시를 거쳐갔음을 알려주기 위해 Via헤더를 포함시킨다.
- 캐시가 Date 헤더를 조정해서는 안된다. Date 헤더는 그 객체가 원 서버에서 최초로 생겨난 일시이다.

6. 전송

- 응답 헤더가 준비되면 캐시는 응답을 클라이언트에게 돌려준다.
- 모든 프락시 서버들과 마찬가지로, 프락시 캐시는 클라이언트와의 커넥션을 유지할 필요가 있음.
- 고성능 캐시는 종종 로컬 저장장치와 네트워크 I/O 버퍼 사이에서 문서의 콘텐츠 복사를 피함으로써 데이터를 효과적으로 전송하기 위해 노력한다.

7. 로깅

- 대부분의 캐시는 로그 파일과 캐시 사용에 대한 통계를 유지한다.
- 각 캐시 트랜잭션이 완료된 후, 캐시는 통계 캐시 적중과 부적중 횟수(그외 다른 관련 지표들)에 대한 통계를 갱신, 로그 파일에 요청 종류와 URL, 그리고 무엇이 일어났는지 알려주는 항목을 추가.
- 가장 많이 쓰이는 캐시 로그 포맷은 스쿼드 로그 포맷과 넷스케이프 확장 공용 로그 포맷. 많은 캐시 제품이 커스텀 로그 파일을 허용한다.

8. 캐시 처리 플로우 차트

  <img width="758" alt="캐시 GET 요청 플로우 차트" src="https://user-images.githubusercontent.com/87509645/201481003-8d373e68-e6a5-4948-8b9b-2eab3e6dcf6a.png">
  
### 8. 사본을 신선하게 유지하기

- 오래된 데이터를 제공하는 캐시는 불필요하며, 캐시된 데이터는 서버의 데이터와 일치하도록 관리되어야 함
- 문서 만료와 서버 재검사: HTTP는 서버에서 캐시가 어떻게 관리되고 있는지 모르더라도<br>사본이 서버와 충분히 일치하도록 유지해주는 단순한 메커니즘을 갖고 있음

#### 8.1 문서 만료

- HTTP는 **Expires** 와 **Cache-Control** 헤더들을 이용해서 원 서버가 문서의 유효기간을 설정할 수 있게 함

![Expires 헤더와 Cache-Control 헤더](https://user-images.githubusercontent.com/75058239/200664741-5c78cfe9-6439-4894-b135-2c73ea8c094e.png)

- 이 만료 기간 전에 캐시는 서버와의 접촉 없이 사본을 제공할 수 있음
- 캐시된 문서가 만료되었다면, 캐시는 반드시 서버와 문서에 변경된 것이 있는지 검사해야 하고,<br>변경사항이 있다면 신선한 사본을 새로운 유효기간과 함께 받아와야 함

#### 8.2 유효기간과 나이

- Expires 헤더와 Cache-Control: max-age 응답 헤더는 유효기간을 명시한다는 같은 일을 하지만,<br>절대 시간은 컴퓨터 시계가 올바르게 맞춰져 있을 것을 요구함

|          헤더          | 설명                                                                                                                                           |
| :--------------------: | :--------------------------------------------------------------------------------------------------------------------------------------------- |
| Cache-Control: max-age | 문서 최대 나이를 정의<br>최대 나이는 문서의 첫 생성 이후 신선하지 않다고 간주될 때까지 경과한 시간의 최댓값<br>`Cache-Control: max-age=484200` |
|        Expires         | 절대 유효기간 명시<br>`Expires: Fir, 05 Jul 2002, 05:00:00 GMT`                                                                                |

※ 모든 HTTP 날짜와 시간은 GMT로 표현된다는 점에 주의해야 함! (KST보다는 9시간 느림)

#### 8.3 서버 재검사

- 캐시된 문서의 만료는 해당 문서가 원 서버에 존재하는 것과 반드시 다르다는 의미는 아님
  - 다만 이제 검사할 시간이 되었다는 뜻
- 서버 재검사 과정
  - 재검사 결과 컨텐츠가 변경되었다면, 캐시는 새로운 사본을 가져와서 저장한 뒤에 클라이언트에게도 이를 보내줌
  - 재검사 결과 컨텐츠가 변경되지 않았다면, 캐시는 새 만료일을 포함한 새 헤더들만 가져와서 캐시 안의 헤더들을 갱신함
- 이런 괜찮은 시스템을 통해 캐시는 매 요청마다 신선도를 검증할 필요가 없고, 문서가 만료되었을 때만 서버와 재검사하면 됨
  - 신선하지 않은 컨텐츠를 제공하지 않으면서도, 서버 트래픽도 절약하고, 사용자 응답 시간도 개선함
- HTTP 프로토콜은 캐시가 다음 중 하나를 반환하는 행동을 적절히 취하도록 요구함
  - '충분히 신선한' 캐시된 사본
  - 재검사가 이루어졌기에 충분히 신선하다고 확신할 수 있는 캐시된 사본
  - 재검사해야 하는 원 서버가 다운된 경우를 위한 에러 메시지
  - 사본이 부정확한 경우 경고 메시지가 부착된 캐시된 사본

#### 8.4 조건부 메소드와의 재검사

- HTTP의 조건부 메소드는 재검사를 효율적으로 만들어줌
- HTTP는 캐시가 서버에게 '조건부 GET' 요청을 보낼 수 있도록 해줌
  - 서버가 갖고 있는 문서가 캐시의 것과 다를 때만 객체 본문을 보내달라고 하는 요청
- 신선도 검사와 객체를 받아오는 작업은 하나의 조건부 GET으로 결합해서 처리 가능!
- 조건부 GET은 GET 요청에 특별한 조건부 헤더를 추가함으로써 시작됨
- HTTP는 5가지 조건부 요청 헤더를 정의하며, 이 중에서 2가지 헤더가 가장 유용하게 쓰임

|            헤더            | 설명                                                                                                       |
| :------------------------: | :--------------------------------------------------------------------------------------------------------- |
| If-Modified-Since: \<date> | 문서가 주어진 날짜 이후로 수정되었다면 요청 메소드 처리<br>Last-Modified 응답 헤더와 함께 쓰임             |
|   If-None-Match: \<tags>   | 문서의 일련번호와 같은 특별한 태그를 활용하는 방법<br>캐시된 태그가 서버 문서의 태그와 다를 때만 요청 처리 |

※ 나머지 3개의 조건부 요청 헤더

- `If-Unmodified-Since` : 문서를 부분적으로 받았을 때, 전에 일부분 받았던 문서가 여전히 신선한지를 확인할 때 유용
- `If-Range` : 불완전한 문서의 캐싱 지원
- `If-Match` : 웹 서버에 대한 동시성 제어를 할 때 유용
  - 예를 들면 문서가 변경되지 않았을 때에만 PUT 요청을 보낼 수도 있음

#### 8.5 If-Modified-Since: 날짜 재검사

- 가장 흔하게 쓰이는 캐시 재검사 헤더
- 'IMS' 요청이라고도 불림
- 서버에게 리소스가 특정 날짜 이후로 변경된 경우에만 본문을 보내달라고 함

※ 처리 과정

- 문서가 주어진 날짜 이후에 변경되었다면 IMS 조건은 참이 되고, GET 요청은 평범하게 성공함
  - 새 문서가 새로운 만료 날짜와 그 외 다른 정보들이 담긴 헤더들과 함께 캐시에게 반환됨
- 문서가 주어진 날짜 이후에 변경되지 않았다면 IMS 조건은 거짓이 되고, 서버는 작은 304 Not Modified 응답을 돌려줌
  - 응답 헤더들은 갱신이 필요한 것만 갱신해서 보내줌

<br>

- If-Modified-Since 헤더는 서버 응답 헤더의 Last-Modified 헤더와 함께 동작함
  - 원 서버는 제공하는 문서에 최근 변경 일시를 붙여줌
- 몇몇 웹 서버는 IMS를 실제 날짜 비교 대신 변경일 간의 문자열 비교로 수행하는 경우도 있음
  - '이 날짜 이후 변경되었다면'이 아닌, '정확히 이 날짜가 마지막 변경일이 아니라면'
  - 예를 들면 일련번호 같은 것을 최근 변경 일시로 사용하는 경우
  - IMS 헤더를 시간에 대한 값으로 활용할 수는 없겠지만, 캐시 만료 관련 동작에는 문제가 없음

#### 8.6 If-None-Match: 엔티티 태그 재검사

- 최근 변경 일시 재검사가 어려운 상황들
  - 어떤 문서는 일정 시간 간격으로 다시 쓰여지는데, 실제로는 같은 데이터를 포함하는 경우
  - 철자나 주석의 변경 등 캐시가 데이터를 다시 읽어들이기에는 너무 사소한 변경일 경우
  - 어떤 서버들은 그들의 페이지에 대한 최근 변경 일시를 정확히 판별할 수 없음
  - 1초보다 적은 간격으로 갱신되는 문서들은, 1초의 정밀도가 충분하지 않을 수 있음

![if-none-match](https://user-images.githubusercontent.com/75058239/200953520-19074d75-2744-4af4-8bb2-f5b00242b307.jpeg)

- 퍼블리셔는 문서 변경 후, 문서의 엔티티 태그를 새로운 버전으로 표현할 수 있음
- If-None-Match 헤더에 일치 여부를 확인하고 싶은 엔티티 태그를 담아서 보내면, 서버는 ETag 헤더와 함께 응답을 보내줌
  - 일치한다면 304 Not Modified 응답을 같은 엔티티 태그를 가진 ETag 헤더와 함께 반환
  - 엔티티 태그가 변경되었다면 200 OK와 함께 변경된 엔티티 태그를 가진 ETag 헤더와 함께 반환
- 캐시가 객체에 대한 여러 사본을 갖고 있는 경우, If-None-Match 헤더에 여러 개의 엔티티 태그를 포함시킬 수 있음

```
If-None-Match: "v2.4","v2.5","v2.6"
If-None-Match: "foobar","A34FAC0095","Profiles in Courage"
```

#### 8.7 약한 검사기와 강한 검사기

- 최근 변경일시와 엔티티 태그는 둘 다 캐시 검사기
- 서버는 때때로 모든 캐시 사본을 무효화하지 않고 문서를 살짝 고치고 싶은 경우가 있음
- 이에 HTTP/1.1은 컨텐츠가 조금 변경되어도 "그 정도면 같은 것"이라고 서버가 주장할 수 있도록, **weak validator**를 지원함
  - **strong validator**는 컨텐츠가 바뀔 때마다 변경 여부를 갱신함
  - **weak validator**는 어느 정도 컨텐츠 변경을 허용하지만, 중요한 의미가 변경되면 변경 여부를 갱신함
- weak validator는 조건부 요청 헤더의 값에 `W/` 접두사를 붙여서 구분하도록 함

```
ETag: W/"v2.6"
If-None-Match: W/"v2.6"
```

#### 8.8 언제 엔티티 태그를 사용하고 언제 Last-Modified 일시를 사용하는가

- HTTP/1.1 클라이언트는 만약 서버가 엔티티 태그를 반환했다면 반드시 엔티티 태그 검사기를 사용해야 함
- 만약 서버가 Last-Modified 값만을 반환했다면 클라이언트는 IMS 검사를 사용할 수 있음
- 만약 IMS와 엔티티 태그 모두 사용 가능하다면,<br>HTTP/1.0과 HTTP/1.1 명세상 모두 적절히 응답하도록 두 가지 재검사 정책을 모두 사용해야 함
  - 강한 엔티티 태그 검사 대신 약한 엔티티 태그 검사를 활용할 순 있음
  - HTTP/1.1 캐시나 서버가 IMS와 엔티티 태그 조건부 헤더 둘 다 받았다면,<br>모든 조건부 헤더가 조건에 부합해야만 304 Not Modified 응답을 반환할 수 있음

<br>

### 9. 캐시 제어

- HTTP 명세는 서버가 캐시를 얼마 오래 지속할 것인지 설정할 수 있는 여러 가지 방법을 정의함

#### 9.1 no-cache와 no-store 응답 헤더

- no-store와 no-cache 헤더는 캐시가 검증되지 않은 캐시 객체로 응답하는 것을 막아줌

```
Cache-Control: no-store
Cache-Control: no-cache
Pragma: no-cache
```

- `no-store` : 캐시가 응답의 사본을 만드는 것을 금지
  - 캐시는 프록시 서버처럼 no-store 응답을 전달하고 나면 해당 객체를 삭제
- `no-cache` : 로컬 캐시 저장소에 저장할 순 있지만 서버와 재검사를 하지 않고서는 캐시에서 클라이언트로 제공할 수 없음
- `Pragma: no-cache` : HTTP/1.0+와의 호환성을 위한 것

#### 9.2 Max-Age 응답 헤더

- `Cache-Control: max-age` 헤더는 신선하다고 간주된 문서가 서버로부터 온 이후로 흐른 시간이며, 초로 표현됨
  - `s-maxage`는 공용 캐시에만 적용됨

```
Cache-Control: max-age=3600
Cache-Control: s-maxage=3600
```

- maximum aging 값을 0으로 설정함으로써, 캐시가 매 접근마다 캐시를 하지 않거나 혹은 refresh 하도록 요청할 수 있음

#### 9.3 Expires 응답 헤더

- deprecated 헤더
- 초 대신 만료 날짜를 명시함
- 신선도 수명의 근사값은 만료일과 생성일의 초 단위 시간차를 계산해서 얻을 수 있음

```
Expires: Fir, 05 Jul 2002, 05:00:00 GMT
```

#### 9.4 Must-Revalidate 응답 헤더

- 캐시는 성능 개선을 위해 신선하지 않은 객체를 제공하도록 설정될 수 있음
- 만약 캐시가 만료 정보를 엄격히 따르길 원한다면, 원 서버는 다음 헤더를 추가해줄 수 있음

```
Cache-Cotrol: must-revalidate
```

- 위 응답 헤더는 신선하지 않은 사본을 원 서버와의 최초 재검사 없이는 제공할 수 없음을 의미함
- 만약 캐시가 must-revalidate 신선도 검사를 시도했을 때 원 서버가 사용 불가라면<br>캐시는 반드시 504 Gateway Timeout error를 반환해야 함

#### 9.5 휴리스틱 만료

- 만약 응답이 Cache-Control: max-age 헤더나 Expires 헤더 중 어느 것도 포함하지 않는다면<br>캐시는 경험적인 방법으로(heuristic) 최대 나이를 계산할 것
- 어떤 알고리듬이든 사용 가능하지만 계산 결과 maximum aging이 24시간보다 크다면<br>Heuristic Expriation 경고(경고 13) 헤더가 응답 헤더에 추가되어야 함
  - 이 경고 정보를 사용자에게 출력해주는 브라우저는 거의 없음
- LM 인자 알고리듬
  - 유명한 휴리스틱 만료 알고리듬 중 하나
  - 문서가 최근 변경 일시를 포함하고 있다면 사용 가능
  - 최근 변경 일시를 문서가 얼마나 자주 바뀌는지에 대한 추정을 위해 사용
  - 만약 최근 변경 일시가 오래 전이라면 변경 가능성도 그리 크지 않으니 캐시에 더 오래 보관하도록 함
  - 반대로 최근 변경 일시가 오래되지 않았다면 변경 가능성도 크므로 짧은 기간 동안만 캐시해야 함

※ LM 알고리듬에 대한 펄 pseudocode

```perl
$마지막_수정_이후_경과시간 = max(0, $서버_Date - $서버_Last_Modified);
$서버_신선도_한계 = int($마지막_수정_이후_경과시간 * $lm_인자);
```

![LM 인자 알고리듬](https://user-images.githubusercontent.com/75058239/201102166-833537c6-8b11-4f7e-8fa8-ca5ecb3b0279.PNG)

- 일반적으로 사람들은 휴리스틱 신선도 유지기간에 상한을 설정함
  - 보통 1주일로 하지만, 보수적인 경우 하루로 설정하기도 함
- 만약 최근 변경일조차 없고, 캐시가 신선도에 대한 아무런 단서도 얻을 수 없을 때는 default 값을 설정함 (보통 1시간 or 하루)
  - 보수적인 경우 휴리스틱 문서들에 대해 0의 신선도 수명을 설정함
- 휴리스틱 신선도 계산은 생각보다 흔히 수행됨
  - 많은 원 서버들이 아직도 Expires와 max-age 헤더를 생성하지 못함
  - 그러므로 캐시 만료 기본값을 신중하게 선택해야 함

#### 9.6 클라이언트 신선도 제약

- 브라우저의 refresh나 reload 버튼은 Cache-Control 요청 헤더가 추가된 GET 요청을 발생시켜서 강제로 재검사하고 문서를 가져옴
- 클라이언트는 Cache-Control 요청 헤더를 사용해서 만료 제약의 강도를 조절할 수 있음

|                           지시어                            | 목적                                                                                                                           |
| :---------------------------------------------------------: | :----------------------------------------------------------------------------------------------------------------------------- |
| Cache-Control: max-stale<br>Cache-Control: max-stale = \<s> | 신선하지 않은 문서도 자유롭게 제공 가능<br>매개변수가 지정되면 매개변수 값만큼 지난 문서도 받아들임<br>캐싱 규칙을 느슨하게 함 |
|               Cache-Control: min-fresh = \<s>               | 클라이언트는 지금으로부터 s초 후까지 신선한 문서만을 받아들임<br>캐싱 규칙을 엄격하게 함                                       |
|                Cache-Control: max-age = \<s>                | 캐시는 s초보다 오래된 캐시 문서를 반환할 수 없음<br>max-stale과 함께 사용되지 않는 이상, 캐싱 규칙을 엄격하게 함               |
|         Cache-Control: no-cache<br>Pragma: no-cache         | 클라이언트는 캐시된 리소스를 재검사하기 전에는 받아들이지 않을 것                                                              |
|                   Cache-Control: no-store                   | 캐시는 저장소에서 문서의 흔적을 최대한 빨리 삭제해야 함<br>문서에 민감한 정보가 있을 경우 사용                                 |
|                Cache-Control: only-if-cached                | 클라이언트는 캐시에 들어있는 사본만을 원함                                                                                     |

#### 9.7 주의할 점

- 문서 만료는 완벽한 시스템이 아님
- 유효기간을 까마득한 미래로 설정해버리면 어떤 변경도 캐시에 반영되지 않을 것
  - DNS와 같은 인터넷 프로토콜에서도 사용되는 TTL 기법의 한 형식인데,<br>HTTP는 만료일을 덮어쓰고 강제로 reload하는 메커니즘을 제공하기는 함
- 유효기간을 길게 잡거나 아예 설정도 하지 않는 경우에 주의해야 함

## 10 캐시 제어 설정

- 웹 서버들은 캐시 제어와 만료 HTTP 헤더들을 설정하는 서로 다른 메커니즘을 제공한다.

### 10.1 아파치로 HTTP 헤더 제어하기

- 아파치 웹서버는 HTTP 캐시 제어 헤더를 설정할 수 있는 여러 메커니즘이 있고, 그 중 많은 것이 디폴트로 가능하지 않게 되어 있어서 사용하려면 활성화해야 한다.

**mod_headers**

- 개별 헤더들을 설정할 수 있는 지시어를 이용하여 아파치 설정 파일에 설정을 추가할 수 있다.
  - 어떤 디렉터리의 모든 HTML 파일을 캐시되지 않도록 설정하는 예
  ```
  <Files *.html>
  Header set Cache-control no-cache
  </Files>
  ```

**mod_expires**

- 적절한 만료 날짜가 담긴 Expires 헤더를 자동으로 생성하는 프로그램 로직을 제공한다.
- 문서에 마지막으로 접근한 날 혹은 수정한 날 이후의 일정 시한으로 유효기간을 설정할 수 있게 한다.
  - 예
  ```
  ExpiresDefault A3600
  ExpiresDefault M86400
  ExpiresDefault "access plus 1 week"
  ExpiresByType text/html "modification plus 2 days 6 hours 12 minutes"
  ```

**mod_cern_meta**

- HTTP 헤더들의 파일을 특정 객체와 연결시켜준다.
- 제어하고자 하는 파일에 각 대응되는 메타파일들을 생성하게 되므로, 각 메타파일에 원하는 헤더를 추가하면 된다.

### 10.2 HTTP-EQUIV를 통한 HTML 캐시 제어

- 웹 서버는 제공할 문서에 올바른 캐시 제어 헤더들을 부여하기 위해 설정 파일들과 상호작용한다.
- HTML 2.0은 <META HTTP-EQUIV> 태그를 정의하여, 웹 서버 설정 파일과 상호작용 없이도 쉽게 HTML문서에 HTTP 헤더 정보를 부여할 수 있다.

```
<HTML>
  <HEAD>
    <TITLE>My Document</TITLE>
    <META HTTP-EQUIV="Cache-control" CONTENT="no-cache">
  </HEAD>
```

- HTTP-EQUIV 태그는 웹 서버에서 사용되도록 의도되었으나 해당 기능을 지원하는 웹 서버나 프락시가 거의 없다. 서버의 부하를 가중시키고, 설정값이 정적이고, HTML을 제외한 다른 타입의 파일은 지원하지 않기 때문이다.
- 몇몇 브라우저는 HTTP-EQUIV 태그를 파싱하고 실제 HTTP 헤더처럼 다룬다. 태그를 지원하는 HTML 브라우저들은 중간의 프락시 캐시와는 다른 캐시 제어 규칙을 적용하기 때문에 캐시 만료에 대한 동작에 혼란을 초래한다.
- 문서의 캐시제어 요청과 커뮤니케이션 하는 확실한 방법은 올바르게 설정된 서버가 보내온 HTTP 헤더를 이용하는 것이다.

![HTTP-EQUIV](https://user-images.githubusercontent.com/74203440/201524706-77574bc2-67c0-4c7f-846b-c68d198b7f1b.jpeg)

## 11. 자세한 알고리즘

- HTTP 명세는 문서의 나이와 캐시 신선도를 계산하는 알고리즘을 제공한다.

### 11.1 나이와 신선도 수명

- 캐시된 문서가 제공하기에 충분히 신선한지 알려주려면 캐시된 사본의 나이와 신선도 수명을 계산하면 된다.
- 캐시된 사본의 나이가 신선도 수명보다 작으면 사본은 제공하기 신선한 것이다.

```
$충분히_신선한가 = ($나이 < $신선도_수명)
```

**문서의 나이**

- 문서의 나이는 서버가 문서를 보낸(혹은 서버가 마지막으로 재검사한) 후 그 문서가 '나이를 먹은' 시간의 총합이다.
- Age헤더를 통하거나 서버가 생성한 Date 헤더를 통해 문서의 나이를 판별한다.

**신선도 수명**

- 문서의 나이가 신선도 수명을 넘으면 클라이언트에게 제공하기에 충분히 신선하지 않은 것이다.
- 문서의 유효기간과 신선도에 영향을 주는 클라이언트의 모든 요청을 고려하여 신선도 수명을 계산한다.

- 클라이언트에 따라 조금 신선하지 않은 문서를 받을 수도 있고, 조금이라도 신선하지 않은 문서는 받지 않을 수도 있다.
  - Cache-Control: max-stale 헤더 / min-fresh 헤더
- 캐시는 서버 만료 정보와 클라이언트 신선도 요구사항을 조합해서 최대 신선도 수명을 판별한다.

### 11.2 나이 계산

- 응답의 나이는 응답이 서버에서 생성되었을(혹은 서버로부터 재검사되었을) 때부터 지금까지의 총 시간이다.
  - 응답이 인터넷 상의 라우터들과 게이트웨이들 사이를 떠돌아다닌 시간과 응답이 캐시에 머무른 시간을 포함한다.

```
$겉보기_나이 = max(0, $응답을_받은_시각 - $Date_헤더값)
$보정된_겉보기_나이 = max($겉보기_나이, $Age_헤더값)
$응답_지연_추정값 = ($응답을_받은_시각 - $요청을_보낸_시각)
$문서가_우리의_캐시에_도착했을_때의_나이 = $보정된_겉보기_나이 + $응답_지연_추정값
$사본이_우리의_캐시에_머무른_시간 = $현재_시각 - $응답을_받은_시각

$나이 = $문서가_우리의_캐시에_도착했을_때의_나이 + $사본이_우리의_캐시에_머무른_시간
```

- a. 캐시는 응답이 캐시에 도착했을 때 Date나 Age 헤더를 분석해서 얼마나 오래된 것인지 알 수 있다.
- b. 캐시는 그 문서가 로컬 캐시에 얼마나 오래 머물렀는지 알 수 있다.
- a+b = 응답의 전체 나이

- 캐시는 캐시된 사본이 로컬에서 얼마나 오래 캐시되었는지 쉽게 알아낼 수 있지만, 모든 서버가 동기화된 시계를 갖고 있지 않고 또 어디서 응답이 왔는지 모르기 때문에 캐시에서 온 응답의 나이를 알아내는 것이 어렵다.

**겉보기 나이는 Date 헤더에 기반한다.**

- 클라이언트와 서버의 시계 설정 차이로 인한 문제를 클록 스큐라고 하는데 이 때문에 겉보기 나이는 부정확하고 음수가 되기도 한다.
- 만약 나이가 음수인 경우, 0으로 만들어야 한다.

```
$겉보기_나이 = max(0, $응답을_받은_시각 - $Date_헤더값)
$문서가_우리의_캐시에_도착했을_때의_나이 = $겉보기_나이
```

**점층적 나이 계산**

- 문서가 프락시나 캐시를 통과할 때마다 그 장치들이 Age헤더에 상대적인 나이를 누적해서 더하도록 한다.
- HTTP/1.1을 이해하는 애플리케이션은 문서가 머무른 시간과 네트워크 이동 시간 만큼 Age 헤더의 값을 늘려야 한다.
- 비-HTTP/1.1 장치는 Age헤더를 인식하지 못하고 헤더를 수정하거나 삭제해버리기 때문에 Age헤더는 상대 나이에 대한 추정값인 상태로 남는다.

```
$겉보기_나이 = max(0, $응답을_받은_시각 - $Date_헤더값)
$보정된_겉보기_나이 = max($겉보기_나이, $Age_헤더값)
$문서가_우리의_캐시에_도착했을_때의_나이 = $보정된_겉보기_나이
```

**네트워크 지연에 대한 보상**

- Date 헤더는 언제 문서가 원 서버를 떠났는지 나타내지만 문서가 캐시로 옮겨가는 동안의 시간은 표현하지 않는다.
- HTTP/1.1은 캐시에서 서버로 갔다가 다시 캐시로 돌아오느라 발생한 지연에 대하여 전체 왕복 시간을 더해 네트워크 지연을 보수적으로 교정한다.

```
$겉보기_나이 = max(0, $응답을_받은_시각 - $Date_헤더값)
$보정된_겉보기_나이 = max($겉보기_나이, $Age_헤더값)
$응답_지연_추정값 = ($응답을_받은_시각 - $요청을_보낸_시각)
$문서가_우리의_캐시에_도착했을_때의_나이 = $보정된_겉보기_나이 + $응답_지연_추정값
```

### 11.3 완전한 나이 계산 알고리즘

```
$나이 = $문서가_우리의_캐시에_도착했을_때의_나이 + $사본이_우리의_캐시에_머무른_시간
```

![완벽한 나이 계산](https://user-images.githubusercontent.com/74203440/201665816-e4ce7dff-f16c-41a1-864e-4abcb04c5f72.jpeg)

### 11.4 신선도 수명 계산

- 신선도 수명은 문서가 특정 클라이언트에게 제공해주기에는 더 이상 신선하지 않게 될 때까지 얼마나 오랜 시간 동안 가져올 수 있도록 허용되는지 말해준다.
- 서버는 문서가 얼마나 자주 변경되어 발행되는지에 대한 정보를 가지고 있을 수 있고, 클라이언트마다 각기 다른 가이드라인으로 신선한 콘텐츠를 받아들인다.

## 12. 캐시와 광고

- 캐시는 사용자를 도와 더 좋은 경험을 제공하고, 네트워크 사업자들이 트래픽을 줄일 수 있도록 도와준다.

### 12.1 광고 회사의 딜레마

- 캐시를 통해 같은 데이터를 방문자에게 반복으로 보여주는 경우, 콘텐츠 제공자는 네트워크 서비스 요금을 줄이면서 다중노출이 가능하다.
- 그러나 캐시는 원 서버가 실제 접근 횟수를 알 수 없게 숨길 수 있기 때문에, 접근 횟수에 따라 돈을 버는 방식이라면 콘텐츠 제공자에게는 긍정적이지 않다.

### 12.2 퍼블리셔의 응답

- 캐시가 트래픽을 흡수하지 못하도록 무력화 시키기도 하는데 이는 해당 사이트에 대한 캐싱의 긍정적인 효과를 감소시킨다.
- 따라서, 모든 접근에 대해 원 서버와 재검사하도록 캐시를 설정하면 매 접근마다 캐시 적중이 있었음을 알리면서 본문 데이터는 전송하지 않는다. 단, 트랜잭션을 느리게 만든다.

### 12.3 로그 마이그레이션

- 캐시는 모든 적중의 로그를 유지할 수 있지만 크기 때문에 옮기기 어렵다.
- 캐시 로그는 개별 콘텐츠 제공자별로 분리될 수 있도록 표준화되어 있지 않고 인증과 프라이버시 이슈가 동반된다.

### 12.4 적중 측정과 사용량 제한

- 특정 URL에 대한 캐시 적중 횟수를 정기적으로 서버에게 돌려주는 Meter라는 헤더를 추가하는 방법이 있다.
- 서버는 캐시가 서버에게 보고하기 전까지 문서 제공 횟수나 소모 처리 시간을 제어할 수 있다. (사용량 제한)
