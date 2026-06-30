# 제안 심사 보조 (Proposal Review Assistant)

TOP 제안활동 · 부적합 심사를 돕는 단일 페이지 웹 앱입니다. 브라우저에서 바로 열어 사용하거나, GitHub Pages로 배포된 주소로 접속해 사용할 수 있습니다.

## 주요 기능

- **새로작성** — 제안 정보·내용 입력 (안전 여부 체크 포함)
- **자동 입력** — 캡처 이미지·메시지(붙여넣기)·`.txt`/`.md`/`.msg`를 **Gemini**가 읽어 제안 정보·본문을 자동 추출 (`.json`은 그대로 반영)
- **AI 심사** — Gemini가 제안 내용을 분석해 판정·심사평 3안을 제시 → 선택·복사
- **중간 저장(임시저장)** — 작성 중인 내용을 Firestore에 보관 후 이어서 작성
- **확정 저장** — 확정된 제안을 Firestore에 기록, 목록에서 삭제, CSV 내보내기
- **제안현황** — 팀 선택 → **안전제안 KPI**(인원의 50% 이상 안전제안 제출) 달성 현황, 팀별·인원별 상세

## 사용 전 설정

### 1) Gemini API 키
AI 기능은 **Google Gemini 무료 모델**(`gemini-2.0-flash`)을 사용하며, 앱에 기본 키가 내장되어 있어 바로 동작합니다.

> ⚠️ **보안:** 공개 사이트에 내장된 키는 누구나 추출·사용할 수 있습니다. 운영 시에는 [Google AI Studio](https://aistudio.google.com/apikey)에서 본인 키를 발급해 **⚙ Gemini 키** 버튼에 입력(브라우저에만 저장)하고, 키에 **API 제한**(Generative Language API)·사용량 한도를 거는 것을 권장합니다. 내장 키는 주기적으로 재발급하세요.

### 2) Firestore 보안 규칙
확정 목록·임시저장은 Firebase Firestore에 저장됩니다. Firebase 콘솔에서 아래 두 컬렉션의 규칙을 설정해야 합니다.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /confirmed/{doc} { allow read, write: if true; }
    match /drafts/{doc}    { allow read, write: if true; }
  }
}
```

> ⚠️ 위 규칙은 **누구나 읽기/쓰기 가능**한 개방형입니다. 공용 도구로 쓸 경우 모든 사용자가 같은 데이터베이스를 공유하게 됩니다. 접근을 제한하려면 Firebase Authentication을 붙이고 `if request.auth != null` 등의 조건으로 바꾸세요.

## 기술 메모

- 단일 `index.html` (의존성 빌드 불필요). Firebase v12 SDK는 CDN에서 동적 로드.
- AI: Google Generative Language API(`gemini-2.0-flash`), 브라우저에서 직접 호출.
- Firebase 웹 설정값은 클라이언트에 공개되는 식별자이며, 보안은 위의 Firestore 규칙으로 합니다.
- Gemini 키는 클라이언트에 노출되므로 키 제한·프록시 등 별도 보호가 필요합니다(위 보안 안내 참고).
