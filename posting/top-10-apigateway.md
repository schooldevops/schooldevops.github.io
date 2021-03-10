top 10 api-gateway

1. Kong Gateway (OSS)
- 인기있는 오픈소스 api gateway
- 다양한 곳에 디플로이 하도록 향상된 클라우드 네이티브 API Gateway 이다. 
- Lua 프로그래밍 언어로 개발
- 하이브리드, 멀티 클라우드 인프라스트럭쳐 지원
- msa, 분산 아키텍처에 특화됨
- 고성능, 확장성, 포터블 지원
- Kong 은 가볍고 빠르고, 확장성이 있다. 
- 데이터베이스 설정이 없고, 인메모리 스토리지, 네이티브 Kubernetes CRD 이용
- 기능
	- 로드밸런싱 (다른 알고리즘 지원)
	- 로깅
	- 인증 (OAuth2)
	- rate-limiting
	- transformation
	- live monitoring
	- service discovery
	- caching
	- failure detection, recovery
	- clustering
	- 노드, 서버리스와 통합 가능 
- 웹소켓 지원, 프록시 지원
- 관리를 위한 커맨드라인 인터페이스 제공
- 높은 확장성 플러그인을 통한 통합
- Restful 관리가능

2. Tyk
- Go 로 개발됨
- 오픈소스, 강력하고, 라이트하며 완젼한 기능을 가짐
- 클라우드 네이티브, 고성능, 플러그가능한 아키텍처 기반의 오픈 스탠다드
- 돌립적으로 수행하며, Redis 필요 (데이터 스토어)
- 안전하게 배포, 다양한 서비스, REST, GraphQL 등을 지원
- 다양한 인증메소드, 쿼타, 레이팅 리밋, 버젼 컨트롤, 알림, 이벤트, 모니터링, 분석 등 지원
- 서비스 디스커버리, 온더플라이 전송, 가상 엔드포인트, 릴리즈 전 목 제공
- API 문서 제공, API 개발자 포탈 지원, CSM(컨텐츠 관리) 지원, 관리된 API 배포 가능, 서드파티 개발자 사인업, API 수신, 소유한 키 관리

3. KrakenD
- Go 로 개발
- 고성능 오픈소스, 단순, 플러그인 가능한 API 개이트웨이 (stateless architecture) 
- 어디서든지 수행가능
- 데이터베이스가 없음
- 단순설정, 제한이 없는 엔드포인트와 백엔드
- 모니터링, 캐싱, 쿼타, 레이트 리밋, QOS (현재 Call, circuit breaker, grained timeout) 전송, aggregation, 소스 머지등
- 필터링(화이트리스트, 블랙 리스트), 디코딩
- 프록시 기능 제공 (로드밸런서), 프로토몰 변환, Oauth
- 보안 기능으로 SSL 과 보안 정책 지원
- KrakenDesigner을 이용하여 API Gateway 의 행위를 설정 가능
- 소스코드 변경없이 플러그인, 임베디드 스크립트, 미들웨어 등 지원 

4. Gravitee.io API Platform
- 오픈소스, 자바기반, 사용하기 쉬운 API 관리 플랫폼
- 보안, 퍼블리싱, 분석, 문서화등 지원
- 3개의 주요 모듈
	- API Management
		- 오픈소스, 단순, 강력, 유연, 가벼움, 매우 빠른 API 
	- Access Management
		- 유연, 가벼움, 다재다능한, 쉽게사용 가능
		- OAuth2/OpenID 연결 프로토콜
		- 프로바이더 브로커를 식별
		- 중앙화된 인증, 권한 서비스 제공 
	- Alert Engine:
		- 사용자 설정 얼릿을 제공
		- 노티 수신
		- API 의 효과적인 모니터링
		- 복수 채널 노티, 이상행동 검출
- Cockpit 에 설치 가능
- API 디자인 툴, 완젼 관리형, 멀티 태넌시 지원

5. Gloo Edge
- 오픈소스, Go rlqks
- Kubernetes-native 인그레스 컨트롤러 (Envoy proxy) 상에서 수행
- 차새대 클라우드 네이티브 API Gateway
- Legacy app wldnjs
- 마이크로 서비스 지원
- 강력한 함수 레벨의 라우팅 (레가시 앱과 통합, 마이크로 서비스, 서버레스)
- 하이브리드 어플리케이션 지원, 다른 형태의 기술 통합, 다른 클라우드에서 수행중인 프로토콜 통합
- API Gateway 기능을 지원하여, 레이팅 리밋, 서킷 브레이킹, 리트라이, 캐싱, 외부 인증, 권한 제공
- 변환, 서비스메시, 완젼 자동화 디스커비리, 시큐리티 제공
- GrapQL, gRPC, OepnTracing, NATS 등을 지원

6. Goku API Gateway
- 오픈소스 마이크로 서비스 게이트웨이
- 클라우드 네이티브 아키텍처
- Go 로 개발
- 유니파이된 인증, 플로우 컨트롤, security protection
- 내부 오픈 API 개발 플랫폼
- 고성능 HTTP 포워딩
- 동적 라우팅
- 서비스 오케스트레이션
- 멀티 태넌시 관리
- ACL 제공
- 클러스터 디플로이, 동적 서비스 등록, 백엔드 로드 밸런싱
- API 헬스 체크, API 연결, 재연결 기능
- 내장 대시보드, 설정을 쉽게, 강력한 플러그인 시스템, 

7. WSO2 API Microgateway
- 오픈소스 클라우드 네이티브, 개발 중심, API gateway 의 탈 중앙화
- Java 로 개발, 생성 프로세스 단순화, 디플로이, 보안 API, 분산 마이크로 서비스 오케스트레이션
- 가벼움, stateless container, 낮은 메모리 풋프린트, 복수개의 마이크로 서비스 취합하여 단일 API fh wprhd
- 런타임 디스커버리 
- 레거시 API 포맷 변환 (요청/응답) 모던화 수행
- OAS 를 이용하여 API 개발에 콜라보, 독립적인 테스트 가능
- 확장가능, 다른 컴포넌트와 의존성이 없음
- 레이팅 리밋, 서비스 디스커버리, 요청, 응답 변환, 로드밸런싱, 페일오버, 서킷 브레이킹, 유연한 도커와 쿠버네티스 통합
- 인증, 권한 기반의 OAuth2.0 API 키 제공
- Basic Auth, mutual TLS 제공

8. Fusio
- 오픈소스
- PHP 기반의 api 관리 솔루션, REST API 관리
- API 관리 플랫폼을 이용한 API 엔드포인트 허용, 요청, 변환 수행
- 모든 필요한 툴 제공 (다른 데이터 소스로 부터 데이터 취득) --> 완젼한 응답 커스터마이즈화
- 비즈니스 기능 제공, 마이크로서비스, 자바스크립트 어플리케이션, 모바일앱등을 노출
- OpenAPI 제터레이션 지원, SDK 생성 지원, 수신 레이어로 pub/sub 지원, 단순한 페이먼트 시스템
- 커맨드라인 클라이언트 지원, API 와 YAML 로 배포 지원

9. Apiman
- 오픈소스, 자바기반
- 풍부한 API 디자인, 설정레이어
- 매우 빠른 수행
- 스탠드 얼론 시스템, 존재하는 프레임워크에 내장되어 수행
- 유연성, 폴리시 기반의 거버넌스, 풍부한 관리 레이어, 완젼한 비동기 처리
- 스로틀링, 쿼타, 보안 중앙화, 빌링, 메트릭, 많은 다른 기능들

10. API Umbrella
- Ruby 를 이용하여 작성
- API 키, 레이트 리밋, 분석, 캐싱
- 멀티 태넌시, Admin 을 지원하여 API 엄브렐러의 모든 애스펙트 관리
- API 라우팅 설정, 사용자 관리, 뷰잉 분석
- 모든 관리기능이 REST로 이용가능

from: https://www.tecmint.com/open-source-api-gateways-and-management-tools/
