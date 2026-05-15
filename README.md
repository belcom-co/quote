# ㈜벨콤 견적 시스템

내부 직원용 견적서 작성/관리 웹 애플리케이션입니다. 단일 HTML 파일로 동작하며, Firebase Firestore를 통해 실시간으로 단가표와 견적 이력이 동기화됩니다.

**접속 주소**: https://belcom-co.github.io/quote/

---

## 직원 사용 가이드

### 접속 방법
1. 위의 접속 주소를 브라우저로 엽니다.
2. 비밀번호 입력창이 나타나면 비밀번호를 입력 후 **접속** 버튼을 누릅니다.
3. (브라우저 창을 닫으면 다음 접속 시 다시 비밀번호 입력이 필요합니다.)

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

### 비밀번호 변경
1. `index.html` 파일을 열고 `ACCESS_PASSWORD` 검색
2. 다음 줄의 값을 새 비밀번호로 변경:
   ```js
   // 비밀번호 변경 시 이 줄을 수정하세요
   const ACCESS_PASSWORD = "새비밀번호";
   ```
3. 저장 후 `git add . && git commit -m "비밀번호 변경" && git push`

### Firebase 콘솔 접속
- 프로젝트 콘솔: https://console.firebase.google.com/project/quote-f7608
- Firestore 데이터 확인: 콘솔 > **Firestore Database** > **데이터** 탭
- 컬렉션 구조:
  - `pricelist/{카테고리키}` — 단가표 (description, items 배열, _order)
  - `quotes/{견적ID}` — 견적 이력 (client, project, date, items, supply, tax, total, savedAt, note)

### 단가표 백업
- 단가표 관리 탭 > **엑셀 내보내기** 버튼으로 언제든 백업 파일 다운로드 가능

### 업데이트 (코드 수정 후 배포)
```bash
git add .
git commit -m "수정 내용 요약"
git push
```
GitHub Pages가 자동으로 1~2분 내에 배포 완료합니다.

---

## 기술 스택
- **Frontend**: 순수 HTML/CSS/JavaScript (단일 파일, 빌드 도구 불필요)
- **데이터베이스**: Firebase Firestore (실시간 동기화)
- **엑셀 라이브러리**: SheetJS (단가표), ExcelJS (견적서 + 로고)
- **호스팅**: GitHub Pages
- **인증**: 클라이언트 측 비밀번호 (sessionStorage)

## 보안 안내
- 비밀번호는 클라이언트 코드에 평문으로 저장되어 있으므로 **외부에 URL을 공개하지 마세요**.
- 검색엔진 색인 차단(`noindex, nofollow`) 메타 태그가 적용되어 있습니다.
- 민감 데이터(고객사 정보 등)는 Firestore 보안 규칙으로 추가 보호됩니다.
