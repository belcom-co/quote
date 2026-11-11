# 견적 시스템 배포 안내 (Firebase Auth + 보안 규칙 적용)

> 2026-11-11 작성. 그로우 통합 전 임시 조치.
> 백업 완료: `quote-backup\2026-11-11\` (quotes 60건, pricelist 9개, 이미지 314개).

이 순서를 **반드시 지켜야** 견적 시스템이 중간에 멈추지 않습니다.
(코드 먼저 배포하지 않고 규칙부터 올리면 전 직원 접근이 끊기고, 반대로 규칙 없이 배포해도 로그인 전까지 데이터가 노출된 상태가 그대로 유지됩니다.)

보안은 **이중**으로 걸립니다:
1. Firebase 콘솔에서 **외부 가입을 아예 차단** (①-③ 참고)
2. Firestore/Storage 규칙에서 **화이트리스트 이메일만 허용** (④ 참고)

---

## ① 콘솔: Authentication 활성화 + 외부 가입 차단 + 직원 계정 생성 (규리가 직접)

### 1-1. 공급업체 사용 설정
1. https://console.firebase.google.com/project/quote-f7608 접속
2. 좌측 메뉴 → **Authentication** → **시작하기**
3. **Sign-in method** 탭 → **이메일/비밀번호** 선택 → **사용 설정** 켜고 저장
   - "이메일 링크(비밀번호 없는 로그인)"은 꺼둠

### 1-2. 외부 가입 차단 (⚠️ 꼭 켜둘 것)
4. **Authentication** → **설정** 탭 → **사용자 작업**
5. **"만들기(가입) 사용 설정"** 체크 해제 → 저장
   - 이렇게 하면 공개된 `firebaseConfig`로 외부인이 임의 가입할 수 없음
   - 규리만 콘솔 Users 탭에서 수동으로 계정을 만들 수 있음

### 1-3. 직원 계정 생성
6. **Users** 탭 → **사용자 추가**
   - 직원마다 계정 1개씩 (이메일 + 임시 비밀번호)
   - 임시 비밀번호는 직원에게 전달하고, 로그인 후 "비밀번호 재설정 메일 받기"로 직접 바꾸도록 안내

> 현재 `firestore.rules` / `storage.rules` 의 허용 이메일 목록: **`kgr.belcom@gmail.com` 1개**.
> 직원을 추가하려면 콘솔에만 추가하지 말고 반드시 **아래 "직원 추가/퇴사 관리" 섹션**대로 규칙 파일도 수정해야 합니다.

---

## ② 코드 push → GitHub Pages 배포

로컬에서:

```powershell
git add .gitignore README.md DEPLOY.md firestore.rules storage.rules index.html
git commit -m "feat: Firebase Auth 로그인 교체 + 보안 규칙 추가"
git push
```

GitHub Pages가 자동으로 1~2분 내 반영. https://belcom-co.github.io/quote/ 새로고침해 확인.

---

## ③ 로그인 동작 확인 (규칙 반영 전)

이 시점에는 아직 규칙이 "모두 열림" 상태라 **비로그인으로도 데이터가 보임**. 다만 UI는 로그인부터 요구함.

- [ ] `https://belcom-co.github.io/quote/` 접속 → **로그인 화면**이 뜸
- [ ] 방금 만든 직원 계정으로 로그인 → 앱 진입
- [ ] **단가표 관리** 탭 → 카테고리/항목이 그대로 보임
- [ ] **견적 이력** 탭 → 기존 60건이 그대로 보임
- [ ] 아무 견적 열기 → 이미지 시트가 깨지지 않고 보임
- [ ] 테스트 견적 **저장** → 콘솔 Firestore에서 해당 문서의 `updatedBy`에 로그인 이메일이 기록됐는지 확인
- [ ] 로그인 화면의 **"비밀번호 재설정 메일 받기"** 클릭 → 이메일 입력 → 메일 수신 확인
- [ ] **로그아웃** 버튼 클릭 → 로그인 화면으로 돌아감
- [ ] 브라우저 창 닫았다 다시 열기 → 다시 로그인 요구됨 (세션 지속성)

여기까지 OK여야 ④로 넘어감. 하나라도 어긋나면 ⑥ **롤백**.

---

## ④ 콘솔: Firestore + Storage 규칙 게시 (규리가 직접)

### Firestore 규칙

1. Firebase 콘솔 → **Firestore Database** 좌측 메뉴
2. 상단 데이터베이스 선택 드롭다운에서 **belcom** 선택 (⚠️ `(default)` 아님)
3. **규칙** 탭 → 기존 내용 전체 선택해서 지우고, 저장소의 `firestore.rules` 파일 내용을 그대로 복붙
4. **게시** 버튼

### Storage 규칙

1. Firebase 콘솔 → **Storage** 좌측 메뉴
2. **규칙** 탭 → 기존 내용 전체 선택해서 지우고, 저장소의 `storage.rules` 파일 내용을 그대로 복붙
3. **게시** 버튼

> 두 규칙 모두 `isAllowedUser()` 로 바뀜 → 로그인 + 화이트리스트 통과한 사용자만 접근 가능.
> 게시 즉시 **화이트리스트에 없는 로그인**조차 차단됨 (⑤에서 확인).

---

## ⑤ 비로그인 + 비허가 계정 차단 확인

- [ ] **시크릿 창**으로 `https://belcom-co.github.io/quote/` 접속 → 로그인 화면
- [ ] 로그인하지 않고 개발자 도구(F12) → **네트워크** 탭 열어 두고 로그인 화면 로드
  - firestore.googleapis.com 호출이 **없거나 401/403**이어야 정상
- [ ] 허용 목록(`kgr.belcom@gmail.com` 등)에 있는 계정으로 로그인 → 단가표/이력 정상
- [ ] (선택) 임의로 규칙 밖 이메일로 콘솔에서 테스트 계정을 만들어 로그인 → 데이터가 아예 안 보이거나 permission-denied 에러 뜨면 OK (테스트 후 즉시 삭제)

---

## ⑥ 문제가 생겼을 때 되돌리기 (롤백)

### 규칙만 되돌리기 (가장 흔한 케이스)
Firebase 콘솔에서 각 규칙 탭의 아래 내용을 붙여넣고 **게시**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} { allow read, write: if true; }
  }
}
```

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} { allow read, write: if true; }
  }
}
```

이러면 즉시 **이전 상태**로 돌아감(로그인 없어도 읽고 쓸 수 있음).

### 코드도 되돌리기

```powershell
git revert HEAD
git push
```

GitHub Pages가 1~2분 내 이전 버전으로 복원.

---

## 🔧 직원 추가 / 퇴사 관리 (운영 중 반복 작업)

> 콘솔 계정 ≠ 접근 권한. **반드시 두 곳 다** 반영해야 로그인 + 데이터 접근이 모두 됨.

### 직원 추가 시
1. Firebase 콘솔 → **Authentication** → **Users** → **사용자 추가**
   - 이메일 + 임시 비밀번호 입력 → 추가
2. 로컬에서 `firestore.rules`, `storage.rules` **두 파일**을 엽니다.
3. 두 파일 안의 `isAllowedUser()` 함수의 **이메일 목록에 새 이메일 추가**
   - 두 파일 목록이 **동일해야** 합니다.
   - 마지막 항목 뒤에는 콤마(,)가 없어야 합니다. 그 외에는 모두 콤마로 구분.
4. Firebase 콘솔 → **Firestore Database** → (belcom 선택) → **규칙** 탭에 `firestore.rules` 내용 전체 복붙 → **게시**
5. Firebase 콘솔 → **Storage** → **규칙** 탭에 `storage.rules` 내용 전체 복붙 → **게시**
6. 로컬에서 git 커밋 & push (규칙 파일 변경 기록 유지)
   ```powershell
   git add firestore.rules storage.rules
   git commit -m "chore: 허용 이메일 추가 (<직원 이름/이메일>)"
   git push
   ```
7. 직원에게 임시 비밀번호 전달 + **"로그인 후 '비밀번호 재설정 메일 받기'로 본인 비밀번호 변경"** 안내

### 직원 퇴사 시
1. Firebase 콘솔 → **Authentication** → **Users** → 해당 계정 우측 메뉴 → **사용자 삭제**
2. 로컬에서 `firestore.rules`, `storage.rules` 두 파일의 이메일 목록에서 **해당 이메일 제거**
3. Firestore 규칙 탭에 `firestore.rules` 복붙 → **게시**
4. Storage 규칙 탭에 `storage.rules` 복붙 → **게시**
5. 로컬에서 git 커밋 & push
   ```powershell
   git add firestore.rules storage.rules
   git commit -m "chore: 허용 이메일 제거 (<직원 이름/이메일>)"
   git push
   ```

### 비밀번호 재설정
- **직원 스스로**: 로그인 화면의 **"비밀번호 재설정 메일 받기"** 클릭 → 본인 이메일 입력 → 수신 메일의 링크로 변경
- **관리자가 보낼 때**: Firebase 콘솔 → **Authentication** → **Users** → 해당 사용자 우측 메뉴 → **비밀번호 재설정**

---

## 참고

- `firebaseConfig`의 apiKey는 공개돼도 되는 값입니다(규칙으로 보호).
- 세션 지속성: 창/탭을 닫으면 로그아웃됩니다. 같은 창 내 새로고침은 로그인 유지.
- 기존 60건 견적에는 `createdBy`/`updatedBy` 필드가 없습니다. 이후 수정 시 `updatedBy`만 채워집니다(의도된 동작).
- 이 설정은 **그로우(Grow) 통합 전 임시 조치**입니다. 통합 시점에 Firebase Auth/규칙을 걷어내고 그로우 로그인 세션 기반으로 재구성 예정.
