# WPF 기반 클라이언트 애플리케이션 개요

## 1. 플랫폼 선택 이유 (WPF vs Web)

- **성능**: WPF는 네이티브 환경에서 실행되어 Web 애플리케이션보다 높은 퍼포먼스를 제공합니다.
- **메모리 사용의 유연성**: 웹 브라우저의 메모리 제약에서 벗어나, 시스템 H/W에 따라 더 자유로운 자원 활용 가능.
- **.NET API 활용**:
  - `Clipboard`, `File`, `Encryption`, `Compression` 등 다양한 기능을 손쉽게 구현.
  - UI 구성 요소가 기본적으로 풍부하게 제공되어 별도 외부 라이브러리 없이도 고급 UI 구현 가능.
- **배포 및 업데이트**: ClickOnce를 통해 자동 업데이트 가능.
  - 최초 1회 설치의 부담은 있으나, 지속적인 유지보수 편의성으로 충분히 상쇄 가능.

---

## 2. 기술 아키텍처

### 사용자 규모
- 약 **500명**의 사용자 대상.

### 네트워크 구조
- 모든 서버 통신은 **API Gateway**를 통해 이루어짐.
  - 인증, 라우팅, 로깅 등 중앙 집중화된 처리 가능.

### 보안 구조

- **로그인 절차**:
  - 클라이언트에서 RSA 공개키로 **ID/PW 암호화**.
  - 클라이언트에서 **AES 키 생성** 후 서버로 전송.
  - 서버에서 인증 성공 시 **SessionId 발급**.
- **세션 관리**:
  - `SessionId`, 사용자 정보, AES 키는 로컬 저장.
  - AES는 **DPAPI**로 암호화하여 안전하게 저장.
  - 세션은 **최대 24시간 유지**, 검증 시 자동 연장.
  - 로그아웃 시 서버 측 세션은 즉시 폐기.
- **동시 접속 제한**:
  - 설치형 앱 구조로 인해 **다중 로그인 불가**, 세션 충돌 없음.

---

## 3. 기능 및 사용자 경험

### Navigation
- WPF의 **Frame Navigation** 사용.
- Web과 유사한 UX 제공.
- **Custom URL Scheme**으로 특정 페이지 및 필터 공유 가능  
  예: `myapp://chart/overview?team=A&range=7d`

### 데이터 보안
- **전송 중**: RSA + AES 혼합 암호화.
- **저장 시**: AES는 DPAPI로 로컬에 안전하게 저장.

### 데이터 전송
- json이 아닌 MemoryPack을 사용해 payload의 용량 최소화 및 serialize, deserialize 성능 최대화.

### 자동 업데이트
- ClickOnce를 통한 무중단 업데이트 제공.

---

## 4. 요약

| 항목                 | 설명 |
|----------------------|------|
| **플랫폼**            | WPF (.NET) |
| **배포**              | ClickOnce 자동 업데이트 |
| **보안**              | RSA + AES, DPAPI, 단일 로그인 |
| **Navigation**        | Frame Navigation + Custom URL Scheme |
| **세션 관리**         | SessionId 24시간 유지, Validation 시 연장 |
| **사용자 수**         | 약 500명 |
| **설치 기반 장점**    | 브라우저 제약 없음, 풍부한 메모리/자원 활용, 고급 UI 구현 가능 |


결론: 언제 Registry를 고려할까?
Registry 사용이 적합한 경우

저장할 데이터가 크지 않고 (수백 바이트 이하)

해당 정보를 숨기고 싶고 (예: 사용자 정보 노출 방지)

삭제나 위조를 어렵게 하고 싶으며

플랫폼이 Windows 전용이고, 보안 요구가 있는 경우

예시:

로그인 세션 정보 (SessionId)

userId, 부서 등 민감하지만 자주 바뀌지 않는 데이터
