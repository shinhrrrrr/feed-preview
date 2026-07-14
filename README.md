# 피드 무드 미리보기 (Firebase Firestore + Vercel 버전)

인스타그램 피드 미리보기 툴을 팀 전체가 하나의 URL로 접속해서, 같은 데이터(계정별 최신 6컷, 프로필 사진, 다크모드 설정)를 보고 편집할 수 있도록 만든 버전입니다.

저장은 **Firestore만** 씁니다(Firebase Storage는 사용하지 않음). 사진은 브라우저에서 자동으로 리사이즈·압축한 뒤 Firestore 문서에 직접 저장돼요. 그래서 **카드 등록 없이 완전 무료 요금제(Spark)로 충분**합니다.

> 2024년 11월부터 Firebase Storage는 무료 요금제에서 빠지고 Blaze(카드 등록 필요) 요금제에서만 쓸 수 있게 바뀌었어요. 이 버전은 그 문제를 피하려고 처음부터 Storage 없이 설계했습니다.

---

## 1. Firebase 프로젝트 만들기

1. https://console.firebase.google.com 접속 → **프로젝트 추가**
2. 프로젝트 이름 입력 (예: `tripicka-feed`) → Google Analytics는 꺼도 무방
3. 프로젝트 안으로 들어가면 왼쪽에 메뉴가 나타납니다 (안 보이면 왼쪽 위 ☰ 아이콘으로 펼치기)
4. **빌드 > Firestore Database** → **데이터베이스 만들기** 클릭
   - 위치(리전): **`asia-northeast3 (서울)`** 선택 (나중에 변경 불가하니 처음에 잘 고르기)
   - 보안 규칙: **테스트 모드로 시작** 선택 → 만들기
5. 왼쪽 메뉴 **프로젝트 설정(톱니바퀴) > 일반** → 아래로 스크롤 → **내 앱 > 웹 앱 추가**(`</>` 아이콘) → 앱 닉네임 아무거나 입력 → 등록
6. 화면에 나오는 `firebaseConfig` 객체를 통째로 복사 (Storage는 안 쓰므로 `storageBucket` 값은 무시해도 됩니다)

## 2. 코드에 설정 값 채우기

`index.html` 상단에서 아래 부분을 찾아서, 방금 복사한 값으로 바꿔주세요.

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  appId: "YOUR_APP_ID"
};
```

> 이 값들은 "비밀키"가 아니라 클라이언트에 그대로 노출되는 공개 설정값이라, 코드에 그대로 넣어도 괜찮습니다. 실제 데이터 보호는 아래 3번의 보안 규칙이 담당합니다.

## 3. 보안 규칙 적용 (중요)

Firebase 콘솔 → **Firestore Database > 규칙** 탭 → 이 저장소의 `firestore.rules` 내용을 그대로 붙여넣고 **게시**.

⚠️ 지금은 `allow read, write: if true` (누구나 읽고 쓸 수 있음)로 열려 있어요. URL과 firebaseConfig를 아는 사람만 접근 가능하다는 전제의 "내부 팀용 최소 보안"입니다. 외부 노출이 걱정되면 나중에 Firebase Authentication을 붙여서 로그인한 사람만 쓰게 제한할 수 있어요 (필요하면 다시 요청해주세요).

## 4. GitHub에 올리기

터미널에서 이 폴더(`tripicka-feed-site`) 기준으로:

```bash
git init
git add .
git commit -m "feed preview tool"
git branch -M main
git remote add origin https://github.com/{본인계정}/tripicka-feed-site.git
git push -u origin main
```

(GitHub에서 새 저장소를 먼저 만들어두세요: https://github.com/new)

## 5. Vercel로 배포

1. https://vercel.com 접속 → GitHub 계정으로 로그인
2. **Add New > Project** → 방금 만든 GitHub 저장소 선택 → **Import**
3. Framework Preset은 **Other**로 두고 그대로 **Deploy** (빌드 설정 필요 없음, `index.html` 하나짜리 정적 사이트라 그대로 서빙됩니다)
4. 배포 완료되면 `https://your-project.vercel.app` 같은 주소가 발급돼요 — 이 URL을 팀원에게 공유하면 끝

## 6. 확인

배포된 URL로 접속해서 사진을 올려보고, 다른 브라우저(또는 팀원 기기)에서 같은 URL로 들어가 같은 데이터가 보이는지 확인하면 끝입니다.

---

### 무료 사용량 안내

Firestore Spark(무료) 요금제 기준 하루 읽기 5만 회, 쓰기 2만 회, 저장용량 1GiB까지 무료예요. 계정 10개, 계정당 사진 6장 규모로는 전혀 걱정할 필요 없는 수준입니다.

### 화질 관련

사진은 업로드 시 자동으로 **최대 1000px, JPEG 75% 품질**로 압축돼서 저장돼요 (프로필 사진은 320px, 80%). Firestore 문서 하나가 1MiB를 넘을 수 없다는 제한 때문인데, 인스타그램 피드 무드 확인용으로는 충분한 화질이지만 원본 그대로는 아니라는 점 참고해주세요. 더 높은 화질이 필요하면 다시 말씀해주세요 (그럴 땐 Storage + Blaze 요금제 방식으로 다시 전환하면 됩니다).

### 파일 구성

- `index.html` — 툴 전체 (UI + Firebase Firestore 연동 코드가 한 파일에 들어있음, 빌드 과정 불필요)
- `firestore.rules` — Firestore 보안 규칙

### 데이터 구조

- `feedAccounts/{accountId}` 문서: `{ name, avatarDataUrl, dark, order: [itemId, ...] }`
- `feedAccounts/{accountId}/items/{itemId}` 문서: `{ dataUrl }` (압축된 base64 사진 1장)
- 계정당 최신 6컷만 유지하도록 클라이언트에서 자동으로 초과분을 정리합니다.
