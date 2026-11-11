# ㈜벨콤 견적 시스템

내부 직원용 견적서 작성/관리 웹 애플리케이션입니다. 단일 HTML 파일로 동작하며, Firebase Firestore를 통해 실시간으로 단가표와 견적 이력이 동기화됩니다.

**접속 주소**: https://belcom-co.github.io/quote/

---

## 직원 사용 가이드

### 접속 방법
1. 위의 접속 주소를 브라우저로 엽니다.
2. 본인에게 발급된 **이메일**과 **비밀번호**를 입력 후 **로그인** 버튼을 누릅니다.
3. (브라우저 창을 닫으면 다음 접속 시 다시 로그인이 필요합니다.)

### 주요 기능
- **견적 작성**: 좌측 카테고리에서 항목을 선택하고 수량을 입력해 견적서 작성
- **실비 항목 직접 입력**: 좌측 하단의 `＋ 실비 항목 직접 입력` 버튼으로 단가표에 없는 항목 추가
- **엑셀 다운로드 / 인쇄·PDF**: 회사 로고가 포함된 견적서를 출력
- **견적 저장**: Firestore에 자동 저장되어 다른 직원도 즉시 확인 가능
- **견적 이력**: 저장된 견적을 클릭하여 다시 불러오기/수정 가능
- **단가표 관리**: 카테고리/항목 추가·수정·삭제, 엑셀 일괄 내보내기/가져오기

### 실시간 동기화
- 우측 상단의 `● 동기화됨` 표시는 Firestore와 연결되어 있음을 의미합니다.
- A 직원이 단가를 수정하면 B 직원 화면도 자동으로 갱신됩니다.

---

## 관리자 가이드

### 직원 계정 추가/삭제 (Firebase Authentication)
1. Firebase 콘솔 접속: https://console.firebase.google.com/project/quote-f7608
2. 좌측 메뉴 **Authentication** → **Users** 탭
3. **사용자 추가** 버튼 → 이메일·비밀번호 입력
4. 퇴사·삭제: 해당 사용자 우측 메뉴 → **사용자 삭제**
5. 비밀번호 재설정: 사용자 우측 메뉴 → **비밀번호 재설정** (해당 이메일로 재설정 링크 발송)

> 비밀번호는 **코드에 적혀 있지 않습니다**. Firebase가 안전하게 보관하므로 여기 콘솔에서만 관리하세요.

### Firebase 콘솔 접속
- 프로젝트 콘솔: https://console.firebase.google.com/project/quote-f7608
- Firestore 데이터 확인: 콘솔 > **Firestore Database** > **데이터** 탭
- 컬렉션 구조:
  - `pricelist/{카테고리키}` — 단가표 (description, items 배열, _order)
  - `quotes/{견적ID}` — 견적 이력 (client, project, date, items, supply, tax, total, savedAt, createdBy, updatedBy, note)

### 단가표 백업
- 단가표 관리 탭 > **엑셀 내보내기** 버튼으로 언제든 백업 파일 다운로드 가능

### 업데이트 (코드 수정 후 배포)
`git add .` 로 뭉뚱그리지 말고 **바뀐 파일만 명시**해서 추가하세요. (다른 워크스페이스/임시 파일 실수 커밋 방지)

```bash
git status                              # 어떤 파일이 바뀌었는지 먼저 확인
git add index.html README.md            # 예: 바뀐 파일만 골라서 추가
git commit -m "수정 내용 요약"
git push
```
GitHub Pages가 자동으로 1~2분 내에 배포 완료합니다.

### 보안 규칙 변경
`firestore.rules` / `storage.rules` 파일을 수정한 뒤에는 **Firebase 콘솔에 붙여넣고 "게시"** 해야 반영됩니다. (저장소에 파일만 바꿔서는 적용되지 않음.)
자세한 순서는 `DEPLOY.md` 참고.

---

## 기술 스택
- **Frontend**: 순수 HTML/CSS/JavaScript (단일 파일, 빌드 도구 불필요)
- **데이터베이스**: Firebase Firestore (실시간 동기화)
- **엑셀 라이브러리**: SheetJS (단가표), ExcelJS (견적서 + 로고)
- **호스팅**: GitHub Pages
- **인증**: Firebase Authentication (이메일/비밀번호, 세션 지속성)

## 보안 안내
- 로그인은 **Firebase Authentication**이 처리하며, 비밀번호는 코드에 저장되지 않습니다.
- Firestore/Storage 데이터는 보안 규칙(`request.auth != null`)으로 보호되므로, 로그인 없이 접근할 수 없습니다.
- 검색엔진 색인 차단(`noindex, nofollow`) 메타 태그가 적용되어 있습니다.
- `firebaseConfig` 의 `apiKey` 는 공개돼도 되는 값입니다(규칙으로 보호).
