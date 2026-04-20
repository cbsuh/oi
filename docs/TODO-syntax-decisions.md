# 미결정 Syntax 사항 — TODO

> Phase 1 (Language Design) 단계에서 Phase 2 (Tree-Walking Interpreter) 구현 전에 결정해야 할 항목들.
> `/design-decision` 워크플로우로 하나씩 진행.
>
> 마지막 업데이트: 2026-04-20

## ✅ 완료

- [x] if/else 조건 표현식 → **ADR-0014**
- [x] 제어 흐름 괄호 규칙 (모두 괄호 없음) → **ADR-0014**
- [x] for/while 루프 → **ADR-0015**
- [x] break/continue 미도입 → **ADR-0015**
- [x] Range 문법 (`..`, `..=`) → **ADR-0015**
- [x] 숫자 타입 (`Int` 64-bit + 고정폭 옵트인) → **ADR-0016**
- [x] 리터럴 문법 (`0x`, `0b`, `0o`, `_`, 과학적 표기법) → **ADR-0016**
- [x] 문자 타입 (`Char` 미도입, `String`만) → **ADR-0016**
- [x] 컬렉션 리터럴 (`[]`, `()`, `Map.from()`, `Set.from()`) → **ADR-0016**
- [x] Unit 타입 (`Unit` 키워드) → **ADR-0016**

## 🔴 Tier 1 — Phase 2 파서 착수 전 반드시 결정

### 모듈 시스템 상세
- [ ] 모듈 ↔ 파일 관계: 1 파일 = 1 모듈? 중첩 모듈?
- [ ] `use` 세부 규칙: 와일드카드 (`use core.*`), aliasing (`use db.Connection as Conn`)
- [ ] 패키지/의존성: multi-file import 규칙

### Effect System 정밀 문법
- [ ] Effect 합성/전파 규칙: callee의 effect가 caller에 자동 전파되는가?
- [ ] Effect 제한: `without` 메커니즘 필요 여부
- [ ] Effect 계층: `io`가 `network`를 포함하는 등의 상하 관계
- [ ] Custom Effect 문법: `trait`으로 할지 `effect` 키워드를 쓸지

### `actor` / `concurrent` 구체 문법
- [ ] `actor` 선언 문법 확정 (최상위 키워드? `state` 필드?)
- [ ] `on` 메시지 핸들러: 인라인 타입 정의 vs 별도 `type`
- [ ] `spawn()` 반환 타입 (`ActorRef[T]`?)
- [ ] `send` vs `ask` 정확한 문법
- [ ] `concurrent.all` 클로저 문법 (`||` vs 일반 람다)
- [ ] 타임아웃/취소 시맨틱

## 🟡 Tier 2 — Phase 2 진행 중 결정 가능

### Contract 검증 시맨틱
- [ ] `requires` 실패 시 동작 (panic? 에러 타입?)
- [ ] `ensures` 검증 시점 (런타임 항상? debug만?)
- [ ] `result =>` 바인딩: 예약어 vs 패턴
- [ ] Contract에서 effectful 함수 호출 가능 여부

### 타입 시스템 세부사항
- [ ] 타입 별칭: `type UserId = Int` (alias? newtype?)
- [ ] Record 업데이트 문법: `{ user with name: "Bob" }`
- [ ] Tuple 타입: `(Int, String)` 지원 여부
- [ ] 제네릭 variance (covariant/contravariant)
- [ ] `Option[T]` 축약: `String?` 여부

### 어노테이션 시스템
- [ ] `@` 어노테이션과 `---` 메타데이터의 역할 구분 명확화
- [ ] 사용자 정의 어노테이션 지원 여부

## 🟢 Tier 3 — Phase 3+ 이후로 미뤄도 안전

- [ ] Supervision tree 문법
- [ ] Distributed actor 문법
- [ ] FFI/interop 문법
- [ ] Serialization 어노테이션 (`@field(n)`)
- [ ] `lazy` 스코프 문법
- [ ] Package manager / build system
