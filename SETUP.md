# 설정 가이드 — GitHub Pages + SharePoint 실시간 연동

## 이미 확보된 정보
- Client ID: `0b2bac3c-b1fb-4b99-bd0e-4dbb18875505`
- Tenant ID: `00efd395-d024-40e7-be0f-842f884fd0dd`
- GitHub 저장소: `https://github.com/niceseed0/doc-repository`
- GitHub Pages 주소(예정): `https://niceseed0.github.io/doc-repository/`
- SharePoint 사이트: `https://elitegslph.sharepoint.com/sites/ELITEGLOBALMANCOMM`
- 리스트 이름: `Document Repository System`
- 권한: Files.ReadWrite.All, Sites.ReadWrite.All, User.Read (Delegated, 관리자 동의 완료)

---

## 1단계 — Azure AD 앱에 Redirect URI 등록

1. [Azure Portal](https://portal.azure.com) → **Entra ID** → **App registrations** → 해당 앱(Client ID: `0b2bac3c...`) 선택
2. 왼쪽 메뉴 **Authentication** 클릭
3. **Add a platform** → **Single-page application (SPA)** 선택 (Web이 아니라 반드시 SPA로! 그래야 클라이언트 시크릿 없이 브라우저에서 안전하게 로그인됩니다)
4. Redirect URI에 다음을 추가:
   ```
   https://niceseed0.github.io/doc-repository/
   ```
5. **Save**

> ⚠️ 이미 "Web" 플랫폼으로 등록되어 있다면, 그 밑에는 SPA용 Redirect URI를 추가할 수 없습니다. 반드시 "Single-page application" 플랫폼 섹션에 추가해주세요.

---

## 2단계 — GitHub Pages 배포

로컬 또는 GitHub 웹에서 저장소(`niceseed0/doc-repository`)에 이 폴더의 `index.html` 파일을 커밋합니다.

```bash
git clone https://github.com/niceseed0/doc-repository.git
cd doc-repository
# index.html 파일을 복사해 넣기
git add index.html
git commit -m "SharePoint 실시간 연동 문서 저장소"
git push
```

그다음:
1. GitHub 저장소 → **Settings** → **Pages**
2. Source: **Deploy from a branch**, Branch: `main` (또는 `master`), 폴더: `/ (root)`
3. 저장 후 몇 분 뒤 `https://niceseed0.github.io/doc-repository/` 에서 접속 가능

---

## 3단계 — SharePoint 리스트 컬럼 확인 (중요)

앱은 리스트 컬럼을 **자동으로 읽어서** 화면을 구성합니다. 다만 몇 가지 주의할 점이 있습니다:

### 부서(탭) 구분 컬럼
- 코드에서 `DEPARTMENT`, `Department`, `TEAM`, `Team`, `부서` 라는 이름의 컬럼을 자동으로 찾아서 탭으로 사용합니다.
- 이 중 어느 것도 없으면 탭 없이 "전체 문서" 하나로만 보여집니다.
- 리스트에 다른 이름의 컬럼을 쓰고 계시다면 `index.html` 안의 이 부분을 수정해주세요:
  ```js
  groupFieldCandidates: ["DEPARTMENT", "Department", "TEAM", "Team", "부서"],
  ```

### 사람(Person) / 조회(Lookup) 타입 컬럼 — 주의
- `OWNER` 같은 컬럼이 **Person 또는 Lookup 타입**으로 만들어져 있으면, 쓰기(수정/추가)가 제대로 반영되지 않을 수 있습니다.
- **가장 안정적인 방법**: SharePoint 리스트에서 `OWNER` 컬럼을 **한 줄 텍스트(Single line of text)** 타입으로 만들어서 이름을 직접 입력하는 방식을 권장합니다.
- 이미 Person 타입으로 되어 있다면 일단 실행은 해보되, 저장이 안 되면 이 부분이 원인입니다.

### 링크(LINK) 컬럼
- **Hyperlink 타입**으로 되어 있으면 자동으로 인식해서 "🔗 Open" 버튼으로 보여줍니다.
- 단순 텍스트 컬럼이어도 문제 없이 동작합니다 (URL 형식이면 링크로 표시).

---

## 4단계 — 접속 및 테스트

1. `https://niceseed0.github.io/doc-repository/` 접속
2. "Microsoft 계정으로 로그인" 클릭 → 회사 계정으로 로그인
3. 로그인 후 자동으로 SharePoint 리스트를 읽어와 표로 보여줍니다
4. 행 수정(✏️) → 저장(✓) 하면 **바로 SharePoint 리스트에 반영**됩니다
5. 다른 사람이 접속해서 새로고침(⟳)하면 방금 바뀐 내용이 그대로 보입니다

---

## 자주 발생하는 오류

| 오류 메시지 | 원인 | 해결 |
|---|---|---|
| `AADSTS50011: redirect URI mismatch` | Azure AD에 Redirect URI 미등록/오타 | 1단계 다시 확인, `https://` 및 마지막 `/` 까지 정확히 일치해야 함 |
| `"Document Repository System" 리스트를 찾을 수 없습니다` | 리스트명 오타 또는 사이트 경로 다름 | 에러 메시지에 실제 리스트 목록이 함께 표시되니 정확한 이름 확인 |
| Graph API 403 Forbidden | 권한 동의가 반영 안 됨(캐시) | 로그아웃 후 재로그인, 또는 몇 분 후 재시도 |
| 저장은 되는데 값이 이상하게 들어감 | Person/Lookup 타입 컬럼 | 위 "Person/Lookup 타입 컬럼" 항목 참고 |

---

## 참고 — 지금 구조의 한계
- 완전한 "같은 셀 동시 편집" 충돌 방지는 지원하지 않습니다 (저장 시점 기준으로 덮어씁니다). 보통 팀 규모에서는 문제 없는 수준입니다.
- 오프라인에서는 동작하지 않습니다(SharePoint 접속 필요).
- 로그인한 사용자 본인이 SharePoint 리스트에 대한 편집 권한이 있어야 실제로 저장됩니다(권한 없는 사용자는 읽기만 가능하거나 오류가 표시됩니다).
