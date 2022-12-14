## 17 내용 협상과 트랜스코딩

- 내용협상이란 무엇인가
- 웹 애플리케이션이 어떻게 내용 협상을 수행하는가

### 17.1 내용 협상 기법
- 서버에 있는 페이지들 중 어떤 것이 클라이언트에게 맞는지 판단하는 세 가지 기준이 있다.

|기법|어떻게 동작하는가|장점|단점|
|:-:|:-:|:-:|:-:|
|클라이언트 주도|클라이언트가 요청을 보내면 서버는 클라이언트에게 선택지를 보내주고 클라이언트가 선택|서버 입장에서 가장 구현하기 쉽다. 클라이언트는 최선의 선택을 할 수 있다.|대기시간이 증가함. 최소 두 번의 요청이 필요|
|서버 주도|서버가 클라이언트의 요청 헤더를 검증해서 어떤 버전을 제공할 지 결정|클라이언트 주도 협상보다 빠름. HTTP는 서버가 가장 적절한 것을 선택할 수 있도록 q값 메커니즘을 제공, 서버가 다운스트림 장치에게 요청이 어떻게 평가되는지 말해줄 수 있도록 하기 위해 Vary 헤더를 제공.|만약 결정이 뻔하지 않으면(헤더에 맞는 것이 없으면) 서버는 추측해야 한다.|
|투명|투명한 중간장치(주로 프락시 캐시)가 서버를 대신해 협상|웹 서버가 협상을 할 필요가 없다. 클라이언트 주도 협상보다 빠름|투명 협상을 어떻게 하는지에 대한 정형화된 명세가 없다.|

### 17.2 클라이언트 주도 협상
- 클라이언트의 요청을 받았을 때 가능한 페이지의 목록을 응답으로 돌려줘 클라이언트가 보고 싶은 것을 선택하는 방법
- 서버 입장에서 가장 구현하기 쉽고 최선의 사본이 선택될 것이다. 
- 각 페이지에 두 번의 요청이 필요하다 (1. 목록 2. 선택한 사본)
- 여러개의 URL(주 페이지 하나와 각 특정 조건별 페이지들)을 요구한다. 

### 17.3 서버 주도 협상
- 클라이언트 주도 협상의 주요 단점인 대기시간 증가를 줄이기 위한 방법은 서버가 어떤 페이지를 돌려줄 것인지 결정하게 하는 것이다. 
- 클라이언트는 반드시 자신이 무엇을 선호하는지에 대한 정보를 내용 협상 헤더에 담아 서버에 전달해야 한다.

#### 17.3.1 내용 협상 헤더
|헤더|설명|
|:--|:-:|
|Accept|선호하는 미디어 타입|
|Accept-Language|선호하는 언어|
|Accept-Charset|선호하는 차셋|
|Accept-Encoding|선호하는 인코딩|

- 15장 엔터티 헤더들과 비슷하지만 분명한 차이가 있다.
- 엔터티 헤더는 메시지를 서버에서 클라이언트를 전송할 때 필요한 메시지 본문의 속성을 가리킨다.
- 내용 협상 헤더는 클라이언트와 서버가 선호 정보를 서로 교환하고 문서들의 여러 버전 중 하나를 선택하는 것을 도와 클라이언트의 선호에 가장 잘 맞는 문서를 제공해 주기 위한 목적으로 사용된다.

#### 17.3.2 내용 협상 헤더의 품질값
- q값을 사용해 선호의 카테고리마다 여러 선택 가능한 항목을 선호도와 함께 나열할 수 있다.
```http
Accept-Language: en;q=0.5, fr;q=0.0, ko;q=1.0, tr;q=0.0
```
- 때때로 서버는 클라의 선호에 대응하는 문서를 하나도 갖고있지 않을 수 도 있다.  
- 이 경우, 서버는 클라의 선호에 맞추기 위해 **문서를 고치거나 트랜스코딩** 할 수 있다.

#### 17.3.3 그 외의 헤더들에 의해 결정
- 서버는 User-Agent와 같은 클라이언트의 다른 요청 헤더들을 이용해 알맞은 요청을 만들어내려고 시도할 수 있다.
- 이 사례에서 '최선'에 가장 가까운 대응을 찾아낼 수 있는 q값 매커니즘은 없다.
- 서버는 정확한 대응을 찾아내거나 아니면 그냥 갖고 있는 것을 제공해야 한다.
- 캐시는 반드시 캐시된 문서의 올바른 ‘최선의’ 버전을 제공해주려 해야 하기 때문에, HTTP 프로토콜은 서버가 응답에 넣어 보낼 수 있는 `Vary` 헤더를 정의한다.
	-   `Vary` 헤더는 캐시에게 서버가 내줄 응답의 최선의 버전을 결정하기 위해 어떤 요청 헤더를 참고하고 있는지 말해준다.


### 17.4 투명 협상
- 투명 협상은 클라이언트 이방에서 협상하는 중개자 프락시를 둠으로써 클라이언트와의 메시지 교환을 최소화하는 동시에 서버 주도 협상으로 인한 부하를 서버에서 제거한다.
- 투명한 내용 협상을 지원하기 위해, HTTP에서 정의한 `Vary` 헤더를 사용한다.
	- 서버는 어떤 요청 헤더를 검사해야 하는지 프락시에게 `Vary` 헤더를 통해 말해준다.

#### 17.4.1 캐시와 얼터네이트(alternate)
<img width="400" alt="" src="https://user-images.githubusercontent.com/87509645/211205056-3213b951-d291-484b-bd80-40aaf70a4018.png">

- 두 사용자의 같은 URL에 대해 두 개의 다른 문서를 각각 캐시에 저장하게 된다. 이를 배리언트나 얼터네이트라고 부른다.
- 내용 협상은 배리언트 중 클라이언트의 요청에 가장 잘 맞는 것을 선택하는 과정으로 볼 수 있다.

#### 17.4.2 Vary 헤더
- 만약 서버가 어떤 페이지를 반환할 것인지 판단하기 위해 다른 헤더들을 사용하고 있따면, 캐시는 반드시 그 헤더들이 무엇인지 알아야 하고 캐시된 페이지 중 어떤 것을 반환할 지 선택할 때 서버가 했던 것과 같은 논리를 적용해야 한다.
- Vary 응답 헤더는 서버가 문서를 선택하거나 커스텀 콘텐츠를 생성할 때 고려한 클라이언트 요청 헤더 모두를 나열한다.
<img width="400" alt="" src="https://user-images.githubusercontent.com/87509645/211206405-7343df25-d11d-4ee9-bf23-f7d86ea94aeb.png">

### 17.5 트랜스코딩
- 내용 협상을 통해 클라이언트 요구에 맞는 서버의 응답을 처리하는 매커니즘에 대해 알아보았다.
- 그러나 서버가 클라이언트 요구에 맞는 문서를 갖고 있지 않으면?
- 이론적으로 서버는 기존의 문서를 클라이언트가 사용할 수 있는 무언가로 변환할 수 있다. 이를 **트랜스코딩**이라고 한다.
|전|후|
|:-:|:-:|
|HTML 문서|WML 문서|
|고해상도 이미지|저해상도 이미지|
|64K색 이미지|흑백 이미지|
|프레임을 포함한 복잡한 페이지|프레임이나 이미지가 없는 단순한 텍스트 페이지|
|자바 애플릿이 있는 HTML 페이지|자바 애플릿이 없는 페이지|
|광고가 있는 페이지|광고가 없는 페이지|

- 트랜스코딩에는 포맷변환, 정보 합성, 내용 주입의 세 종류가 있다.

#### 17.5.1 포맷 변환
- 포맷 변환은 데이터를 클라이언트가 볼 수 있도록 한 포맷에서 다른 포맷으로 변환하는 것이다.

#### 17.5.2 정보 합성
- 문서에서 정보의 요점을 추출하는 것
- 간단한 예로 각 절의 제목에 기반한 문서의 개요 생성이나 페이지에서 광고 및 로고 제거

#### 17.5.3 내용 주입
- 앞의 두 종류의 트랜스코딩은 일반적으로 웹 문서의 양을 줄이지만 오히려 양을 늘리는 내용 주입 트랜스코딩도 있다.
- 자동 광고 생성이나 사용자 추적 시스템 등의 예가 있다.

#### 17.5.4 트랜스코딩 vs 정적으로 미리 생성해놓기
- 트랜스코딩의 대안으로 웹 서버에서 웹페이지의 여러 사본을 만드는 것이 있다. 그러나 현실적인 기법이 되지 못한다.
- 정적으로 미리 생성해 둔다면 작은 변화로 인한 수정과 저장하기 위한 많은 공간, 페이지 관리 측면이 어려워진다.
- 루트 페이지를 그때그때 필요할 때마다 변환하는 것은 정적으로 미리 생성해 놓는 것보다 더 쉬운 해결책이다.

### 17.6 다음 단계
- 내용 협상에 대한 이야기는 다음 두 가지 이유로 Accept나 Content 관련 헤더들에서 끝나지 않는다.

> 1. HTTP의 내용 협상은 성능 제약을 초래한다.
> 	: 적절한 콘텐츠를 위해 여러 배리언트를 탐생하는 것이나, 혹은 가장 잘 맞는 것을 '추측'하려 하는 것은 비용이 크다. 
> 2. HTTP는 내용 협상이 필요한 유일한 프로토콜이 아니다.
> 	: 미디어 스트리밍과 팩스는 클라이언트와 서버가 클라이언트의 요청에 대한 최적의 답을 하기 위해 논의해야 할 필요가 있는 두 가지 다른 예다. 