# React 프론트엔드 규칙 (Cursor)

> **용도:** Cursor 에이전트가 프론트엔드(`frontend/`) 작업 시 매번 같은 프롬프트를 반복하지 않도록, 이 파일을 `@docs/DevOps/Frontend/REACT_RULES.md` 로 참조한다.

---

## 핵심 원칙: `useState` 남용 금지

React에서 **`useState`를 필드·플래그마다 여러 개 두지 않는다.**

- 관련 상태는 **하나의 객체**로 묶는다.
- 폼 입력값은 가능하면 **비제어(uncontrolled) + `FormData`** 로 제출 시 읽는다.
- `errors`, `submitting`, `message` 같은 UI 상태도 **폼(또는 화면) 단위 객체**로 묶는다.

---

## Cursor 에이전트 지시문 (복사·참조용)

아래를 사용자가 매번 입력하지 않아도, 이 MD를 참조한 작업에서는 **기본으로 적용**한다.

```
React에서 useState는 많이 사용하면 안 됩니다.

폼 제출은 다음 패턴을 사용하세요:
- handleSubmit 시 FormData + Object.fromEntries
- e: React.FormEvent<HTMLFormElement>
- name 속성이 있는 input (가능하면 value/onChange 제거)

여러 개의 useState는 관련 항목끼리 객체로 압축하세요:
- UI: { tab, showPassword, ... }
- 폼 상태: { errors, submitting, message }
- patch 함수로 부분 업데이트: setX(prev => ({ ...prev, ...patch }))

검증은 submit 핸들러에서 formProps 기준으로 수행하세요.

입력값·민감 데이터는 alert/confirm/prompt, toast, 인라인 message에 절대 넣지 마세요.
에러는 `lib/user-facing-error.ts`의 safeApiErrorMessage로만 표시하세요.
```

---

## 1. 폼 제출: `FormData` 패턴

```tsx
const handleSignup = async (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault()
  const formData = new FormData(e.currentTarget)
  const formProps = Object.fromEntries(formData.entries()) as SignupFormProps

  // formProps.username, formProps.password 등으로 검증·API 호출
}
```

**JSX 요구사항**

- 각 `<Input>` / `<input>`에 `name` 속성 필수 (`username`, `password`, …)
- `type="submit"` 버튼으로 제출
- 비밀번호 표시 토글 등 UI만 `useState` 객체(`ui`)로 관리

---

## 2. 상태 압축: 객체 `useState`

### ❌ 지양

```tsx
const [username, setUsername] = useState("")
const [password, setPassword] = useState("")
const [loginErrors, setLoginErrors] = useState({})
const [loginSubmitting, setLoginSubmitting] = useState(false)
const [loginMessage, setLoginMessage] = useState<string | null>(null)
const [tab, setTab] = useState("login")
const [showPassword, setShowPassword] = useState(false)
```

### ✅ 권장

```tsx
type FormStatus = {
  errors: Record<string, string>
  submitting: boolean
  message: string | null
}

type UiState = {
  tab: "login" | "signup"
  showPassword: boolean
}

const [ui, setUi] = useState<UiState>({ tab: "login", showPassword: false })
const [login, setLogin] = useState<FormStatus>({
  errors: {},
  submitting: false,
  message: null,
})

const patchLogin = (patch: Partial<FormStatus>) =>
  setLogin((prev) => ({ ...prev, ...patch }))
```

**부분 업데이트 예**

```tsx
patchLogin({ message: null })
patchLogin({ errors: {}, submitting: true })
patchLogin({ message: "로그인에 성공했습니다.", submitting: false })
```

---

## 3. 검증 분리

submit 핸들러 안에서 `formProps`를 받는 순수 함수로 검증한다.

```tsx
function validateSignup(formProps: SignupFormProps) {
  const errors: Record<string, string> = {}
  if (!formProps.username.trim()) errors.username = "아이디를 입력해주세요"
  // ...
  return errors
}

const errors = validateSignup(formProps)
if (Object.keys(errors).length) {
  patchSignup({ errors })
  return
}
```

---

## 4. 참고 구현 (본 저장소)

로그인·회원가입 폼 적용 예:

- `frontend/lib/form-status.ts` — `FormStatus`, `patchState`
- `frontend/lib/user-facing-error.ts` — API 에러를 안전한 문장으로 (`safeApiErrorMessage`)
- `frontend/app/login/auth-forms.tsx` — `ui`, `login`, `signup` + `FormData`
- `frontend/components/gemini-chat-panel.tsx` — `ui`, `chat` + form submit
- `frontend/components/mova/mova-ai-chat-bar.tsx` — `chat` + form submit
- `frontend/components/mova/mova-search-bar.tsx` — `search` 객체 (디바운스 검색)
- `frontend/components/header-weather.tsx` — `current`, `forecast` 객체
- `frontend/components/mova/title/mova-title-view.tsx` — `ui` + 리뷰 `FormData`

새 폼·설정 화면·모달 폼을 추가·리팩터할 때 위 파일과 **동일한 패턴**을 따른다.

---

## 5. 예외 (여러 `useState`가 허용되는 경우)

- 서로 무관한 **전역 UI** (예: 단일 모달 open 플래그 한 개)는 분리 가능
- **서버/URL에서 오는 데이터**와 **폼 draft**가 완전히 분리된 경우
- 그래도 **3개 이상**의 연관 `useState`가 생기면 객체 병합을 먼저 검토

---

## 6. 입력값 노출 금지 (`alert` · 에러 메시지)

### 금지

- `window.alert()` / `confirm()` / `prompt()` — **사용하지 않는다** (디버그·데모 포함).
- alert·toast·`FormStatus.message`·채팅 `error`에 **`formProps` 전체**, **비밀번호**, **이메일**, **API 요청 body**를 넣지 않는다.
- 에러 처리에서 `JSON.stringify(body)`, `JSON.stringify(formProps)`, `String(unknownDetail)` 로 객체 전체를 UI에 뿌리지 않는다.

### ✅ 권장

- 성공/실패는 폼 아래 **고정 문구** 또는 `errors` 필드별 메시지 (`auth-forms.tsx` 패턴).
- API 실패 시 `safeApiErrorMessage(detail, fallback, status)` 사용 (`frontend/lib/user-facing-error.ts`).
- 서버 `detail`이 문자열일 때만 그대로 표시하고, 그 외에는 **짧은 fallback** (예: `"로그인에 실패했습니다."`).

```tsx
import { safeApiErrorMessage } from "@/lib/user-facing-error"

patchLogin({
  message: safeApiErrorMessage(body.detail, "로그인에 실패했습니다.", res.status),
})
```

### ❌ 지양

```tsx
alert(JSON.stringify(formProps))
alert(`입력: ${formProps.password}`)
patchSignup({ message: String(body.detail) }) // detail이 객체일 때 [object Object] 또는 payload 노출
```

`role="alert"` 가 붙은 `<p>` 는 **접근성용 라벨**이며, 브라우저 `alert()` 와 무관하다.

---

## 7. 체크리스트 (PR·리뷰 전)

- [ ] 폼 필드마다 `useState`가 있지 않은가?
- [ ] submit에서 `FormData` + `Object.fromEntries`를 쓰는가?
- [ ] `errors` / `submitting` / `message`가 한 객체로 묶였는가?
- [ ] `patchX(partial)` 형태로 상태 업데이트가 일관적인가?
- [ ] input에 `name`이 있고, 불필요한 `value`/`onChange`가 제거되었는가?
- [ ] `alert` / `confirm` / `prompt` 가 없고, 에러에 입력값·JSON payload가 없는가?
- [ ] API 에러는 `safeApiErrorMessage` 또는 필드별 `errors`만 쓰는가?
