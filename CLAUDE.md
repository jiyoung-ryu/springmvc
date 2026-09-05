# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 성격

Spring MVC 학습용 리포지토리다. 강의 진도에 맞춰 예제를 하나씩 추가하는 구조이며, 프로덕션 코드가 아니다. 다음을 전제로 작업한다.

- 컨트롤러는 같은 기능을 `v1`, `v2`, `v3`… 로 반복 구현한다. 중복이 아니라 저수준 서블릿 API에서 스프링 어노테이션으로 넘어가는 과정을 단계별로 남긴 것이다. **리팩터링으로 이전 버전을 지우거나 하나로 합치지 말 것.**
- 권장되지 않는 방식으로 소개된 코드(예: `ResponseViewController.responseViewV3`의 void 반환)도 비교 대상으로 남겨둔다.
- 주석 처리된 대안 코드(`//@Controller`, `//@ResponseBody`, `//private final Logger log = ...`)는 같은 일을 하는 다른 방법을 보여주기 위한 것이니 정리하지 말 것.

커밋은 진도 회차 단위로 하나씩 만든다. 예: `feat: 56. HTTP 응답 - HTTP API, 메시지 바디에 직접 입력`

## 명령어

```bash
./gradlew bootRun        # 앱 실행 (localhost:8080)
./gradlew compileJava    # 컴파일만 (커밋 전 확인용)
./gradlew build          # 빌드 + 전체 테스트
./gradlew test           # 전체 테스트
./gradlew test --tests 'hello.springmvc.SpringmvcApplicationTests'   # 단일 테스트 클래스
./gradlew test --tests '*.SpringmvcApplicationTests.contextLoads'    # 단일 테스트 메서드
```

## 버전 주의사항

이 프로젝트는 **Spring Boot 4.1.1 / Java 25**다. 대부분의 Spring MVC 예제 코드는 Boot 3.x / Java 17 기준이라 그대로 옮기면 컴파일되지 않는 지점이 있다.

- **Jackson 3** — `ObjectMapper`는 `com.fasterxml.jackson.databind`가 아니라 **`tools.jackson.databind.ObjectMapper`**.
- **스타터 이름** — `spring-boot-starter-web`이 아니라 **`spring-boot-starter-webmvc`**. 테스트도 `spring-boot-starter-webmvc-test` / `spring-boot-starter-thymeleaf-test`.
- 서블릿 API는 `javax.*`가 아니라 `jakarta.*`.

외부 예제를 옮겨올 때는 커밋 전 `./gradlew compileJava`로 확인한다.

## 코드 구조

`src/main/java/hello/springmvc/basic/` 아래를 주제별로 나눠 둔다.

- `requestmapping/` — `@RequestMapping` 매핑 규칙 예제 (`MappingController`), 클래스 레벨 매핑 + REST 스타일 (`MappingClassController`)
- `request/` — 요청 조회. 헤더(`RequestHeaderController`), 쿼리 파라미터·폼(`RequestParamController`), 메시지 바디 텍스트(`RequestBodyStringController`), JSON(`RequestBodyJsonController`)
- `response/` — 응답. 뷰 템플릿(`ResponseViewController`), 메시지 바디 직접 출력(`ResponseBodyController`)
- `HelloData` — `@ModelAttribute` / `@RequestBody` 바인딩 대상으로 여러 컨트롤러가 공유하는 롬복 `@Data` DTO

리소스:
- `src/main/resources/static/` — 정적 리소스. `basic/hello-form.html`은 `/request-param-v1`로 POST하는 테스트 폼.
- `src/main/resources/templates/response/hello.html` — 타임리프 뷰. `${data}` 하나만 출력한다.

로깅은 롬복 `@Slf4j` + SLF4J를 쓰고, 인자는 항상 `{}` 플레이스홀더로 넘긴다(`log.info("username={}", username)`). 문자열 `+` 연결은 로그 레벨이 꺼져 있어도 연산이 일어나므로 쓰지 않는다.
