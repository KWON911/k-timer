# 수업용 집중 타이머 — 사용성·접근성 개선 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Bring `index.html`(수업용 집중 타이머)의 접근성 공백을 메우고, 두 개의 편의 기능(시간 추가, 카운트다운 효과음, 글자색 커스터마이즈)을 추가하고, 컨트롤·설정 패널의 시각/정보 구조를 단순화한다.

**Architecture:** 대상은 단일 파일 `index.html`(HTML+CSS+JS 인라인, 외부 의존성 0)뿐이다. 새 파일을 만들지 않고, 기존 함수·CSS 변수·데이터 모델에 맞춰 점진적으로 수정한다. 설계 근거는 `docs/superpowers/specs/2026-08-06-usability-accessibility-design.md`.

**Tech Stack:** Vanilla JS, WebAudio(합성음), CSS(Grid/Flexbox/커스텀 프로퍼티), `localStorage`. 빌드 도구·프레임워크·패키지 매니저 없음.

## Global Constraints

- **의존성 0.** `npm`/번들러/테스트 프레임워크를 새로 들이지 않는다 (`PLAN.md`의 핵심 설계 원칙). 모든 검증은 **브라우저에서 `index.html`을 직접 열어 수동으로 확인**한다. 각 태스크의 "검증" 스텝은 자동화 테스트가 아니라 구체적인 조작-결과 체크리스트다.
- **파일은 `index.html` 하나(+ 문서 동기화용 `사용설명서.md`)만 수정한다.** 새 파일을 만들지 않는다.
- 마감시각(deadline) 기반 엔진 불변조건을 지킨다 — `setInterval`로 초를 깎는 방식을 도입하지 않는다. 시간 계산은 항상 `endsAt`/`remainingMs`/`totalMs` 조합을 그대로 따른다.
- `localStorage` 키 접두사 `focusTimer.`와 내보내기 JSON 스키마(`{app:"focus-timer", version:1, templates:[...]}`)를 깨지 않는다. 새 설정 키를 추가할 때는 `load()`의 sanitize 로직에 손상값 방어 코드를 반드시 같이 추가한다.
- 새 애니메이션은 `prefers-reduced-motion: reduce`를 존중한다(기존 `@media (prefers-reduced-motion:reduce)` 블록, `index.html:374` 부근에 추가).
- 새 인터랙티브 요소는 기존 `:focus-visible` 아웃라인 규칙을 상속받게 하고(커스텀 `outline:none`을 넣지 않는다), 아이콘만 남는 요소는 `aria-label`을 명시한다.
- 커밋은 논리적 단위(태스크)마다 하나씩, 한국어 커밋 메시지로 남긴다(기존 로그 톤 참고: `fix:`, `feat:`, `refactor:` 접두사).

---

### Task 1: D1 — 섹션 제목 축약 (`활동 단계` → `활동`)

**Files:**
- Modify: `index.html:519` (`<summary>활동 단계</summary>`)
- Modify: `사용설명서.md` — `활동 단계`를 언급하는 모든 곳(2, 3, 4, 5, 6장 등)

**Interfaces:**
- Consumes: 없음
- Produces: 화면에 보이는 섹션 이름이 `활동`으로 바뀜. 이후 태스크(D2/D3/D4)는 이 새 이름을 전제로 부제·캡션을 붙인다.

- [ ] **Step 1: `index.html`에서 표시 텍스트 변경**

`index.html:519`:
```html
<!-- 변경 전 -->
<summary>활동 단계</summary>
<!-- 변경 후 -->
<summary>활동</summary>
```
`id="secSteps"` 등 내부 식별자는 그대로 둔다 — 저장된 `openSecs` 값과 어긋나면 안 된다.

- [ ] **Step 2: `사용설명서.md`에서 `활동 단계`를 `활동`으로 맞춘다**

Grep으로 위치를 확인한다:
```bash
grep -n "활동 단계" 사용설명서.md
```
각 위치를 문맥에 맞게 고친다. 예시(2장 화면 구성 다이어그램, 5장 제목, 6장 안내):
```markdown
<!-- 변경 전 -->
설정 패널 → **활동 단계**
<!-- 변경 후 -->
설정 패널 → **활동**
```
"활동 단계로 수업 흐름 만들기" 같은 장 제목은 "활동으로 수업 흐름 만들기"처럼 자연스럽게 조사를 맞춰 고친다. 단, **본문 중 "활동"이 일반 명사로(예: "짧은 활동에 쓰면 좋습니다")로 쓰인 곳은 건드리지 않는다** — 섹션 이름을 가리키는 경우만 고친다.

- [ ] **Step 3: 수동 검증**

브라우저에서 `index.html`을 열고:
1. 설정(⚙) 패널을 연다.
2. 두 번째 아코디언 제목이 `활동`으로 보이는지 확인한다.
3. 그 안의 단계 목록·템플릿 기능이 기존과 동일하게 동작하는지 확인한다(펼침 상태 기억 포함 — 새로고침 후에도 열려 있던 구획이 유지되는지).
4. `사용설명서.md`를 열어 `활동 단계`가 남아있지 않은지 훑어본다(단, "짧은 활동" 같은 일반 명사 사용은 정상).

- [ ] **Step 4: 커밋**

```bash
git add index.html 사용설명서.md
git commit -m "refactor: 설정 패널의 '활동 단계' 섹션명을 '활동'으로 줄인다"
```

---

### Task 2: D2 + D4 + D5 — 부제·캡션·체크박스 문구 추가

**Files:**
- Modify: `index.html:501-586` (설정 패널 3개 섹션 summary, `untilBtn`/`fitBtn` 앞, 체크박스 라벨)
- Modify: CSS 블록 (`index.html` `<style>` 안, `.sec>summary` 근처, `index.html:289-299` 부근)

**Interfaces:**
- Consumes: Task 1의 `활동` 표기
- Produces: `.sec-sub`, `.caption` CSS 클래스 — Task 3(D3)이 같은 구획을 감쌀 때 이 마크업을 건드리지 않고 바깥에 wrapper만 추가한다.

- [ ] **Step 1: 부제용 CSS 추가**

`.sec>summary` 규칙 뒤(`index.html:299` 부근, `.sec>summary:hover{...}` 다음 줄)에 추가:
```css
/* D2 — 세 시작 방식 섹션의 한 줄 부제. summary 는 이미 flex 로 제목과 ▾ 를 양끝에 두므로,
   제목 텍스트를 span 으로 감싸 그 안에서만 세로로 쌓는다 */
.sec-title{display:flex;flex-direction:column;gap:1px;min-width:0}
.sec-sub{font-size:.76rem;font-weight:400;color:var(--muted)}
/* D4 — 헷갈리던 두 시각 버튼(이 시각까지 / 종료 시각에 맞춰 압축) 위에 붙는 한 줄 안내.
   기존 .total 과 같은 톤이지만 여백만 다르다(버튼 바로 위 캡션이라 아래 여백을 줄인다) */
.caption{font-size:.82rem;color:var(--muted);margin:10px 0 4px}
```

- [ ] **Step 2: 세 섹션의 `<summary>`에 부제 추가**

`index.html:501-502` (`secQuick`):
```html
<!-- 변경 전 -->
<details class="sec" id="secQuick" open>
    <summary>빠른 시작</summary>
<!-- 변경 후 -->
<details class="sec" id="secQuick" open>
    <summary><span class="sec-title"><span>빠른 시작</span><span class="sec-sub">숫자만 넣으면 바로 시작</span></span></summary>
```
`index.html:519` (`secSteps`, Task 1에서 이미 `활동`으로 바뀐 상태):
```html
<!-- 변경 전 -->
<summary>활동</summary>
<!-- 변경 후 -->
<summary><span class="sec-title"><span>활동</span><span class="sec-sub">도입 → 활동 → 정리, 순서대로 이어서</span></span></summary>
```
`index.html:552` (`secPomo`):
```html
<!-- 변경 전 -->
<summary>뽀모도로</summary>
<!-- 변경 후 -->
<summary><span class="sec-title"><span>뽀모도로</span><span class="sec-sub">집중 25분·휴식 5분처럼 자동 반복</span></span></summary>
```
`표시 & 소리`, `단축키`는 부제를 붙이지 않는다(설계 D2 범위 밖).

- [ ] **Step 3: 헷갈리는 두 시각 버튼에 캡션 추가**

`index.html:511-513` (`untilBtn` 앞, `빠른 시작` 안):
```html
<!-- 변경 전 -->
    <div class="row mt">
      <input type="time" id="untilTime" aria-label="종료 시각" style="width:110px">
      <button id="untilBtn" style="flex:1" title="지정한 시각까지 (벽시계 기준)">이 시각까지</button>
    </div>
<!-- 변경 후 -->
    <div class="caption">이 시간 하나만 잴 때</div>
    <div class="row mt">
      <input type="time" id="untilTime" aria-label="종료 시각" style="width:110px">
      <button id="untilBtn" style="flex:1" title="지정한 시각까지 (벽시계 기준)">이 시각까지</button>
    </div>
```
`index.html:544-546` (`fitBtn` 앞, `활동` 안):
```html
<!-- 변경 전 -->
    <div class="row mt">
      <input type="time" id="fitTime" aria-label="수업 종료 시각" style="width:110px">
      <button id="fitBtn" style="flex:1" title="남은 시간에 맞춰 각 단계를 비율대로 줄여 시작">종료 시각에 맞춰 압축 시작</button>
    </div>
<!-- 변경 후 -->
    <div class="caption">짜둔 단계들을 이 시각에 맞춰 줄여서 시작</div>
    <div class="row mt">
      <input type="time" id="fitTime" aria-label="수업 종료 시각" style="width:110px">
      <button id="fitBtn" style="flex:1" title="남은 시간에 맞춰 각 단계를 비율대로 줄여 시작">종료 시각에 맞춰 압축 시작</button>
    </div>
```

- [ ] **Step 4: 체크박스 문구 정리 (D5)**

`index.html:576-577`:
```html
<!-- 변경 전 -->
    <div class="toggle"><label for="soundChk">종료 알림음</label><input type="checkbox" id="soundChk"></div>
    <div class="toggle"><label for="preWarnChk">종료 30초 전 예고음</label><input type="checkbox" id="preWarnChk"></div>
<!-- 변경 후 -->
    <div class="toggle"><label for="soundChk">종료 알림음 (단계 끝날 때 소리)</label><input type="checkbox" id="soundChk"></div>
    <div class="toggle"><label for="preWarnChk">종료 30초 전 예고음 (낮은 소리 두 번)</label><input type="checkbox" id="preWarnChk"></div>
```

- [ ] **Step 5: 수동 검증**

1. 설정 패널을 열고 `빠른 시작`/`활동`/`뽀모도로` 제목 아래 회색 부제가 한 줄로 보이는지 확인한다(제목과 겹치거나 잘리지 않는지, 패널 폭이 가장 좁은 680px 미만 화면에서도 확인).
2. `이 시각까지` 버튼과 `종료 시각에 맞춰 압축 시작` 버튼 위에 캡션이 붙어 두 개를 구분하기 쉬워졌는지 확인한다.
3. `표시 & 소리`에서 두 체크박스 라벨이 길어져도 줄바꿈이 깨지지 않는지 확인한다.

- [ ] **Step 6: 커밋**

```bash
git add index.html
git commit -m "feat: 설정 패널에 부제·캡션을 붙여 각 항목이 무엇인지 바로 알 수 있게 한다"
```

---

### Task 3: D3 — 시각적 그룹화 (타이머 시작 3종 묶기, 단축키 톤 낮추기)

**Files:**
- Modify: `index.html:499-596` (`<aside class="panel">` 내부 구조)
- Modify: CSS 블록 (`.panel` 근처)

**Interfaces:**
- Consumes: Task 2의 `.sec-title`/`.sec-sub` 마크업(내용은 그대로, 바깥에 wrapper만 씌운다)
- Produces: `.sec-group`, `.sec-group-label`, `.sec-quiet` CSS 클래스

- [ ] **Step 1: 그룹 CSS 추가**

`.panel` 규칙 뒤(`index.html:281` 부근, `html[data-panel="closed"] .panel{...}` 다음)에 추가:
```css
/* D3 — 빠른 시작·활동·뽀모도로는 "타이머를 어떻게 시작할지"를 고르는 대체 가능한 방법들이라
   시각적으로 한 묶음임을 보여준다. 아코디언 상호작용(개별 펼침 상태 기억)은 그대로 둔다 —
   순전히 테두리·소제목만 두르는 시각적 구분이다 */
.sec-group{border:1px solid var(--line);border-radius:10px;padding:0 6px;margin-bottom:14px}
.sec-group>.sec:last-child{border-bottom:none}
.sec-group-label{
  font-size:.72rem;font-weight:700;letter-spacing:.04em;color:var(--muted);
  text-transform:uppercase;padding:10px 6px 2px;
}
/* 단축키는 참고 자료라는 인상을 주려고 톤을 낮춘다 */
.sec-quiet{opacity:.82;margin-top:10px}
.sec-quiet>summary{font-size:.94rem}
```

- [ ] **Step 2: HTML에 그룹 wrapper 추가**

`index.html:499-559` 범위(`secQuick`부터 `secPomo` 끝까지)를 감싼다:
```html
<!-- 변경 전 -->
<aside class="panel" id="panel">

  <details class="sec" id="secQuick" open>
    ...
  </details>

  <details class="sec" id="secSteps">
    ...
  </details>

  <details class="sec" id="secPomo">
    ...
  </details>

  <details class="sec" id="secDisp">
<!-- 변경 후 -->
<aside class="panel" id="panel">

  <div class="sec-group">
    <div class="sec-group-label">타이머 시작</div>

    <details class="sec" id="secQuick" open>
      ...
    </details>

    <details class="sec" id="secSteps">
      ...
    </details>

    <details class="sec" id="secPomo">
      ...
    </details>
  </div>

  <details class="sec" id="secDisp">
```
(세 `<details>`의 내용은 건드리지 않는다 — 들여쓰기만 한 단계 깊어진다.)

`index.html:588` (`secKeys`)에 `sec-quiet` 클래스 추가:
```html
<!-- 변경 전 -->
  <details class="sec" id="secKeys">
<!-- 변경 후 -->
  <details class="sec sec-quiet" id="secKeys">
```

- [ ] **Step 3: 수동 검증**

1. 설정 패널을 열고 `빠른 시작`/`활동`/`뽀모도로`가 하나의 테두리 안에 "타이머 시작" 소제목과 함께 묶여 보이는지 확인한다.
2. `표시 & 소리`가 그 아래 별도 블록으로 분리돼 보이는지 확인한다.
3. `단축키`가 톤이 낮아져(약간 흐리고 위 여백이 있어) 참고 자료처럼 보이는지 확인한다.
4. 세 시작 방식 아코디언을 동시에 여러 개 펼쳐두고 새로고침해도 그 상태가 유지되는지 확인한다(기존 `openSecs` 동작 보존 확인).
5. 라이트/다크 테마 모두에서 그룹 테두리 색이 잘 보이는지 확인한다.

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "feat: 설정 패널에서 3가지 타이머 시작 방식을 시각적으로 묶는다"
```

---

### Task 4: C2 — 설정 패널 트리거를 가장자리 탭으로 (`⚙ 설정` 버튼 제거)

**Files:**
- Modify: `index.html:456` (`<button id="panelBtn">` 마크업)
- Modify: `index.html:332` (`#panelBtn` CSS)
- Modify: `index.html:276` (`.panel` 기본 패딩)

**Interfaces:**
- Consumes: 없음
- Produces: `#panelBtn`은 그대로 유지되는 id/aria 속성(`aria-expanded`, `aria-controls`) — Task 5(A3)가 이 요소를 포커스 트리거로 그대로 재사용한다.

- [ ] **Step 1: 버튼 라벨을 아이콘 전용으로, `aria-label` 추가**

`index.html:456`:
```html
<!-- 변경 전 -->
<button id="panelBtn" aria-expanded="false" aria-controls="panel" title="설정 (S)">⚙ 설정</button>
<!-- 변경 후 -->
<button id="panelBtn" aria-expanded="false" aria-controls="panel" aria-label="설정 패널 열기" title="설정 (S)">⚙</button>
```

- [ ] **Step 2: CSS로 "떠 있는 버튼"에서 "가장자리 탭"으로 재구성**

`index.html:332` (`#panelBtn{...}`) 교체:
```css
/* 변경 전 */
#panelBtn{position:fixed;top:12px;right:24px;z-index:7;box-shadow:var(--shadow)}

/* 변경 후 */
/* 화면 오른쪽 가장자리에 얇게 붙는 탭. 닫혀 있을 때는 화면 끝에, 패널이 열리면
   패널의 왼쪽 경계까지 따라가 "이게 곧 닫는 손잡이"가 된다(패널의 margin-right
   트랜지션과 같은 easing/시간을 써서 같이 움직이는 것처럼 보이게 한다). */
#panelBtn{
  position:fixed;top:50%;right:0;transform:translateY(-50%);
  z-index:8;width:26px;height:88px;padding:0;
  border-radius:10px 0 0 10px;border-right:none;
  display:flex;align-items:center;justify-content:center;
  font-size:1.15rem;box-shadow:var(--shadow);
  transition:right .28s ease;
}
html[data-panel="open"] #panelBtn{right:var(--panelW)}
/* 680px 미만은 패널이 전체 화면을 덮으므로(--panelW:100vw), 위 규칙을 그대로 두면
   탭이 화면 왼쪽 밖으로 완전히 사라져 태블릿 등 터치 기기에서 닫을 방법이 없어진다.
   그래서 이 폭에서는 열려 있어도 탭을 화면 모서리에 작은 정사각형으로 고정해 둔다. */
@media (max-width:680px){
  html[data-panel="open"] #panelBtn{
    right:14px;top:14px;transform:none;width:34px;height:34px;border-radius:8px;
  }
}
```

- [ ] **Step 3: 패널 상단 여백 정리**

`index.html:276` (`.panel{...}` 안의 `padding:54px 18px 18px;`)을 바꾼다. 기존 54px 상단 여백은 "떠 있는 ⚙설정 버튼이 첫 제목을 가리지 않도록" 비워둔 자리였는데, 이제 탭이 옆(넓은 화면) 또는 680px 미만 전용 오버라이드(`index.html:409`, 이미 `padding:54px 14px 18px`로 별도 지정돼 있어 안 건드려도 됨)로 옮겨가서 넓은 화면 기본값에는 더 이상 필요 없다:
```css
/* 변경 전 */
.panel{
  width:var(--panelW);background:var(--surface);border-left:1px solid var(--line);
  /* 위쪽 여백은 고정된 ⚙설정 버튼이 첫 제목을 가리지 않도록 비워 둔 자리다 */
  overflow-y:auto;padding:54px 18px 18px;display:flex;flex-direction:column;gap:4px;
  ...

/* 변경 후 */
.panel{
  width:var(--panelW);background:var(--surface);border-left:1px solid var(--line);
  overflow-y:auto;padding:18px;display:flex;flex-direction:column;gap:4px;
  ...
```
(`index.html:409`의 680px 미만 전용 `.panel{... padding:54px 14px 18px}` 규칙은 그대로 둔다 — 그 폭에서는 탭이 다시 모서리 버튼 모양으로 겹치기 때문에 여백이 여전히 필요하다.)

- [ ] **Step 4: 수동 검증**

1. 넓은 화면(1200px 이상)에서 탭이 화면 오른쪽 가장자리에 세로로 얇게 붙어 있는지 확인한다. 클릭하면 패널이 열리고 탭이 패널 왼쪽 경계까지 슬라이드해 붙는지 확인한다.
2. 탭을 다시 클릭하면 패널이 닫히고 탭이 오른쪽 끝으로 돌아오는지 확인한다.
3. `S` 키, `Esc` 키로도 여닫히는지 확인한다(기존 `togglePanel`/`applySettings` 로직은 안 건드렸으므로 동작해야 한다).
4. 창 폭을 680px 미만으로 줄이고, 패널을 열었을 때 탭이 화면 밖으로 사라지지 않고 오른쪽 위 모서리의 작은 버튼으로 보이는지, 눌러서 닫히는지 확인한다.
5. 패널이 닫혀 있을 때 첫 아코디언 제목(`빠른 시작`, Task 3 이후엔 그룹 소제목 포함)이 잘려 보이지 않는지 확인한다(상단 여백 축소로 인한 겹침이 없어야 한다).
6. 화면에 더 이상 "설정"이라는 텍스트 라벨이 보이지 않는지 확인한다(아이콘만).

- [ ] **Step 5: 커밋**

```bash
git add index.html
git commit -m "feat: 뜬 설정 버튼을 오른쪽 가장자리 탭으로 바꾼다"
```

---

### Task 5: A3 — 설정 패널 포커스 관리 (+ 좁은 화면 포커스 트랩)

**Files:**
- Modify: `index.html:1791-1794` (`function togglePanel`)
- Modify: JS — 새 keydown 리스너 추가 (`index.html:1804` 근처, 기존 단축키 리스너 바깥)

**Interfaces:**
- Consumes: Task 4의 `#panelBtn` (여전히 트리거 요소), `#panel` 안의 `summary`/`input`/`button`/`select` 마크업
- Produces: 없음(다른 태스크가 의존하는 새 함수 없음)

- [ ] **Step 1: `togglePanel`에 포커스 이동 추가**

`index.html:1791-1794`:
```javascript
// 변경 전
function togglePanel(force){
  state.settings.panelOpen = typeof force === "boolean" ? force : !state.settings.panelOpen;
  applySettings(); save();
}

// 변경 후
function togglePanel(force){
  const opening = typeof force === "boolean" ? force : !state.settings.panelOpen;
  state.settings.panelOpen = opening;
  applySettings(); save();
  // 여는 쪽: 패널 안 첫 포커스 가능 요소(대개 첫 구획의 제목)로 보낸다.
  // 닫는 쪽: 항상 트리거(탭)로 되돌려 포커스가 사라진 콘텐츠 안에 남지 않게 한다.
  if(opening){
    const first = document.querySelector("#panel summary, #panel input, #panel button, #panel select");
    if(first) first.focus();
  }else{
    $("panelBtn").focus();
  }
}
```

- [ ] **Step 2: 좁은 화면(680px 미만)에서 Tab 트랩 추가**

기존 단축키 `keydown` 리스너(`index.html:1805`) 바로 뒤, `document.addEventListener("visibilitychange", ...)`(`index.html:1838`) 앞에 새 리스너를 추가한다:
```javascript
/* 패널이 화면 전체를 덮는 폭(680px 미만)에서는 사실상 모달이다.
   Tab 이 패널 밖(가려진 스테이지)으로 새어나가지 않게 가둔다.
   680px 이상(패널·스테이지가 나란히 보이는 레이아웃)에서는 트랩을 걸지 않는다 —
   둘 다 보이는 상태에서 Tab 이동을 막을 이유가 없다. */
const isNarrowPanel = () => window.matchMedia("(max-width:680px)").matches;
document.addEventListener("keydown", e => {
  if(e.key !== "Tab" || !state.settings.panelOpen || !isNarrowPanel()) return;
  const focusables = $("panel").querySelectorAll(
    'summary, input:not([disabled]), select, button:not([disabled])'
  );
  if(!focusables.length) return;
  const first = focusables[0], last = focusables[focusables.length - 1];
  if(e.shiftKey && document.activeElement === first){ e.preventDefault(); last.focus(); }
  else if(!e.shiftKey && document.activeElement === last){ e.preventDefault(); first.focus(); }
});
```

- [ ] **Step 3: 수동 검증**

1. 마우스를 쓰지 않고 `Tab`으로 페이지를 돌다 `S`를 눌러 패널을 연다 — 포커스가 패널 안 첫 항목(첫 아코디언 제목)으로 이동하는지 확인한다.
2. `Esc`를 눌러 닫는다 — 포커스가 오른쪽 가장자리 탭으로 돌아오는지, 계속 `Tab`을 누르면 다음 포커스가 자연스럽게 이어지는지 확인한다.
3. 창 폭을 680px 미만으로 줄이고 패널을 연다. `Tab`을 반복해서 마지막 항목에서 한 번 더 누르면 첫 항목으로 돌아오는지(포커스가 뒤에 숨겨진 스테이지 버튼으로 새지 않는지) 확인한다. `Shift+Tab`으로 역방향도 확인한다.
4. 창 폭을 1200px 이상으로 넓히고 패널을 연 채 `Tab`을 눌러 포커스가 스테이지 쪽 버튼(예: 전체화면)으로도 자유롭게 넘어가는지 확인한다(트랩이 걸리지 않아야 한다).

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "fix: 설정 패널 여닫을 때 포커스를 관리하고 좁은 화면에서 포커스 트랩을 건다"
```

---

### Task 6: A1 — 상태 전환 음성 안내 강화 (`announce` 헬�퍼 도입)

**Files:**
- Modify: `index.html:1263-1268` (`render()`의 분당 안내 블록)
- Modify: `index.html:954-1073` (`start`, `pause`, `onStepEnd`, `finish`)

**Interfaces:**
- Consumes: 없음
- Produces: `function announce(text)` — Task 10(B1)이 "N분/초 추가함" 안내에 재사용한다.

- [ ] **Step 1: `announce` 헬퍼 추가**

`render()` 함수 바로 앞(`index.html:1203` 위, `function fmt` 다음)에 추가:
```javascript
/* aria-live 영역에 문구를 채운다. render() 의 분당 안내와 같은 요소를 쓰므로,
   상태 전환 안내는 항상 render() 호출 "다음"에 불러야 분당 안내에 덮이지 않는다. */
function announce(text){
  $("live").textContent = text;
}
```

- [ ] **Step 2: 기존 분당 안내를 헬퍼로 교체**

`index.html:1263-1268`:
```javascript
// 변경 전
  const min = Math.ceil(st.remainingMs / 60000);
  if(st.status === "running" && min !== lastMinute){
    lastMinute = min;
    $("live").textContent = `남은 시간 약 ${min}분`;
  }

// 변경 후
  const min = Math.ceil(st.remainingMs / 60000);
  if(st.status === "running" && min !== lastMinute){
    lastMinute = min;
    announce(`남은 시간 약 ${min}분`);
  }
```

- [ ] **Step 3: `start()`에 안내 추가**

`index.html:954-972`, 함수 끝(`startTick(); acquireWake(); render();` 다음 줄)에 추가:
```javascript
function start(){
  state.overrunFrom = null;
  if(state.status === "finished") armStep(0);
  if(state.absoluteEnd){
    state.remainingMs = Math.max(0, state.absoluteEnd - Date.now());
    if(state.remainingMs <= 0){
      noteOverride = "지정한 종료 시각이 지났습니다 — 새 시각을 입력하세요";
      render(); return;
    }
  }
  if(!state.queue.length || state.remainingMs <= 0) return;
  noteOverride = "";
  state.endsAt = state.absoluteEnd || (Date.now() + state.remainingMs);
  state.status = "running";
  startTick(); acquireWake(); render();
  announce("시작함");   // render() 의 분당 안내를 덮어써야 하므로 반드시 render() 다음
}
```

- [ ] **Step 4: `pause()`에 안내 추가**

`index.html:993-998`:
```javascript
// 변경 전
function pause(){
  if(state.status !== "running") return;
  state.remainingMs = Math.max(0, state.endsAt - Date.now());
  state.endsAt = null; state.status = "paused";
  stopTick(); releaseWake(); render();
}

// 변경 후
function pause(){
  if(state.status !== "running") return;
  state.remainingMs = Math.max(0, state.endsAt - Date.now());
  state.endsAt = null; state.status = "paused";
  stopTick(); releaseWake(); render();
  announce("일시정지함");
}
```

- [ ] **Step 5: `onStepEnd()`의 두 분기에 안내 추가**

`index.html:1049-1073`:
```javascript
// 변경 전
  if(state.settings.autoAdvance){
    armStep(state.index + 1);
    state.endsAt = plannedEnd + state.remainingMs;
    state.status = "running";
    startTick(); render();
  }else{
    releaseWake();
    armStep(state.index + 1);
    state.status = "paused";
    noteOverride = "다음 단계 준비됨 — 시작을 누르세요";
    render();
    beginOverrun(plannedEnd);
  }

// 변경 후
  if(state.settings.autoAdvance){
    armStep(state.index + 1);
    state.endsAt = plannedEnd + state.remainingMs;
    state.status = "running";
    startTick(); render();
    announce(`${state.queue[state.index].name} 시작, ${durationLabel(state.totalMs)}`);
  }else{
    releaseWake();
    armStep(state.index + 1);
    state.status = "paused";
    noteOverride = "다음 단계 준비됨 — 시작을 누르세요";
    render();
    announce("다음 단계 준비됨, 시작을 누르세요");
    beginOverrun(plannedEnd);
  }
```

- [ ] **Step 6: `finish()`에 안내 추가**

`index.html:1020-1026`:
```javascript
// 변경 전
function finish(){
  stopTick(); releaseWake();
  state.status = "finished";
  state.remainingMs = 0; state.endsAt = null;
  noteOverride = "모든 단계가 끝났습니다";
  render();
}

// 변경 후
function finish(){
  stopTick(); releaseWake();
  state.status = "finished";
  state.remainingMs = 0; state.endsAt = null;
  noteOverride = "모든 단계가 끝났습니다";
  render();
  announce("모든 단계가 끝났습니다");
}
```

- [ ] **Step 7: 수동 검증**

Windows 내레이터(`Ctrl+Win+Enter`) 또는 NVDA를 켜고:
1. 프리셋으로 타이머를 시작한다 — "시작함"이 읽히는지 확인한다(분당 "남은 시간 약 N분"에 곧바로 덮이지 않아야 한다).
2. `Space`로 일시정지한다 — "일시정지함"이 읽히는지 확인한다.
3. 활동 단계 2개 이상으로 큐를 만들어 자동 진행 켠 채로 한 단계를 끝낸다 — 다음 단계 이름과 길이가 읽히는지 확인한다.
4. 자동 진행을 끄고 같은 걸 반복한다 — "다음 단계 준비됨, 시작을 누르세요"가 읽히는지 확인한다.
5. 마지막 단계까지 진행해 전체가 끝난다 — "모든 단계가 끝났습니다"가 읽히는지 확인한다.
6. 콘솔에 오류가 없는지 확인한다.

- [ ] **Step 8: 커밋**

```bash
git add index.html
git commit -m "feat: 시작·일시정지·단계전환·종료 시점에 스크린리더 안내를 추가한다"
```

---

### Task 7: A2 — 제목 편집 어포던스 (`#stepName` `aria-label`)

**Files:**
- Modify: `index.html:1327-1344` (`function renderStepName`)

**Interfaces:**
- Consumes: 없음
- Produces: 없음

- [ ] **Step 1: `canEdit` 분기에 `aria-label` 토글 추가**

`index.html:1336-1343`:
```javascript
// 변경 전
  const canEdit = !!(cur && cur.editable);
  if(canEdit !== lastEditable){
    lastEditable = canEdit;
    el.setAttribute("contenteditable", canEdit ? "plaintext-only" : "false");
    el.dataset.edit = canEdit ? "on" : "off";
    if(canEdit) el.setAttribute("title", "눌러서 이름을 고칠 수 있습니다 (비우면 원래대로)");
    else el.removeAttribute("title");
  }

// 변경 후
  const canEdit = !!(cur && cur.editable);
  if(canEdit !== lastEditable){
    lastEditable = canEdit;
    el.setAttribute("contenteditable", canEdit ? "plaintext-only" : "false");
    el.dataset.edit = canEdit ? "on" : "off";
    if(canEdit) el.setAttribute("title", "눌러서 이름을 고칠 수 있습니다 (비우면 원래대로)");
    else el.removeAttribute("title");
  }
  // aria-label 은 편집 가능 여부가 바뀔 때뿐 아니라 이름 자체가 바뀔 때도 최신 상태를 반영해야 한다
  el.setAttribute("aria-label", canEdit ? `제목: ${want}. 눌러서 고쳐 쓸 수 있습니다` : "");
  if(!canEdit) el.removeAttribute("aria-label");
```
(`want`는 이 함수 위쪽에서 이미 계산된 변수 — `index.html:1330` `const want = cur ? cur.name : "타이머";`를 그대로 참조한다.)

- [ ] **Step 2: 수동 검증**

1. 브라우저 개발자 도구에서 `#stepName` 엘리먼트를 열고, 프리셋(`5분 집중` 등, 편집 가능)을 실행 중일 때 `aria-label`이 `제목: 5분 집중. 눌러서 고쳐 쓸 수 있습니다`처럼 보이는지 확인한다.
2. 활동 단계나 뽀모도로(편집 불가)로 전환하면 `aria-label` 속성이 사라지는지(또는 비어있는지) 확인한다.
3. 스크린리더로 `Tab`을 눌러 제목 영역에 포커스가 갈 때 편집 가능 여부와 현재 이름이 함께 읽히는지 확인한다.
4. 제목을 실제로 고쳐 써서 기존 동작(Enter 확정, Esc 취소, 빈 값 입력 시 원래 이름 복귀)이 그대로인지 확인한다.

- [ ] **Step 3: 커밋**

```bash
git add index.html
git commit -m "fix: 제목 편집 가능 여부를 스크린리더에도 aria-label 로 알린다"
```

---

### Task 8: A4 + A5 — 1분 전 비색상 경고 신호 + 미니 버튼 터치 타겟 확대

**Files:**
- Modify: CSS 블록 (`index.html:372-376` 부근 — `danger` 펄스 규칙 옆에 `warn` 규칙 추가)
- Modify: `index.html:312` (`.step .mini button`)
- Modify: `index.html:413` (680px 미만 중복 규칙 제거)

**Interfaces:**
- Consumes: 없음
- Produces: 없음

- [ ] **Step 1: 1분 전 경고에 정적 인디케이터 추가**

`index.html:372` 부근(`body[data-warn="danger"] #time{animation:pulse 1s ease-in-out infinite}` 다음)에 추가:
```css
/* A4 — 10초 전(danger)은 펄스 애니메이션으로 색맹이어도 알아채지만, 1분 전(warn)은
   색상 변화뿐이었다. 카운터 오른쪽 위에 정적인 삼각형을 그려 애니메이션 없이도
   구분되게 한다(prefers-reduced-motion 과 무관하게 항상 보인다) */
body[data-warn="warn"] .clock-wrap::after{
  content:"";
  position:absolute; top:8%; right:8%;
  width:9%; aspect-ratio:1; max-width:20px; max-height:20px; min-width:10px; min-height:10px;
  background:var(--warn);
  clip-path:polygon(50% 6%, 6% 94%, 94% 94%);
}
```

- [ ] **Step 2: 미니 버튼 터치 타겟 확대**

`index.html:312`:
```css
/* 변경 전 */
.step .mini button{padding:6px 7px;font-size:.8rem;line-height:1;border-radius:6px}
/* 변경 후 */
.step .mini button{padding:8px 9px;font-size:.8rem;line-height:1;border-radius:6px}
```
`index.html:413`(680px 미만 미디어쿼리 안)의 중복 규칙은 이제 같은 값이므로 삭제한다:
```css
/* 삭제 */
  .step .mini button{padding:8px 9px}     /* 손가락으로 누를 수 있는 크기 */
```

- [ ] **Step 3: 수동 검증**

1. 프리셋을 1분보다 살짝 긴 값(예: `1:10`)으로 직접 입력해 시작하고, 남은 시간이 1분 이하로 내려갈 때 카운터 오른쪽 위에 작은 삼각형이 나타나는지 확인한다. 10초 이하로 내려가면 기존 펄스+빨간색으로 바뀌면서 삼각형은 사라져도 무방한지(경고색이 이미 최우선이므로) 확인한다.
2. 링/원판/숫자만 세 표시 방식 모두에서 삼각형이 카운터를 가리지 않고 잘 보이는지 확인한다.
3. 설정 패널의 활동 단계 목록에서 ↑/↓/✕ 버튼이 이전보다 살짝 커졌는지, 태블릿 폭(680~900px)에서도 오조작 없이 눌리는지 확인한다.
4. `prefers-reduced-motion: reduce`를 켠 상태(OS 설정 또는 브라우저 에뮬레이션)에서도 삼각형이 그대로 보이는지 확인한다(애니메이션이 아니므로 영향받지 않아야 한다).

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "fix: 1분 전 경고에 비색상 신호를 추가하고 미니 버튼 터치 타겟을 키운다"
```

---

### Task 9: B2 — 10초 카운트다운 효과음

**Files:**
- Modify: `index.html:686-690` (`state.settings` 기본값)
- Modify: `index.html:741-781` (`load()` — sanitize)
- Modify: `index.html:903-913` (`function chime`)
- Modify: `index.html:933, 942` (`preWarned` 근처 — 새 트래커 변수)
- Modify: `index.html:1037-1048` (`function tick`)
- Modify: `index.html:576-579` (표시 & 소리 체크박스 HTML)
- Modify: `index.html:1486-1506` (`applySettings`)
- Modify: `index.html:1739-1776` 부근 (이벤트 리스너 등록)

**Interfaces:**
- Consumes: 기존 `chime(kind)`, `state.settings`, `danger` 임계값(`remainingMs <= 10000`)
- Produces: 모듈 스코프 변수 `lastBeepSecond` — Task 10(B1)이 시간 추가 시 리셋한다.

- [ ] **Step 1: 설정 기본값 추가**

`index.html:686-690`:
```javascript
// 변경 전
  settings: { theme:"light", display:"ring", sound:true, volume:0.6, autoAdvance:true,
              panelOpen:false, preWarn:true, overrun:true, showClock:true,
              baseColor:"",
              openSecs:["secQuick"],
              quickName:"" },

// 변경 후
  settings: { theme:"light", display:"ring", sound:true, volume:0.6, autoAdvance:true,
              panelOpen:false, preWarn:true, overrun:true, showClock:true,
              baseColor:"",
              countdownBeep:true,   // 종료 10초 전부터 매초 효과음, 기본 켬
              openSecs:["secQuick"],
              quickName:"" },
```

- [ ] **Step 2: `load()`에 손상값 방어 추가**

`index.html:763` (`if(!isHex(state.settings.baseColor)) state.settings.baseColor = "";` 다음 줄)에 추가:
```javascript
state.settings.countdownBeep = typeof state.settings.countdownBeep === "boolean" ? state.settings.countdownBeep : true;
```

- [ ] **Step 3: `chime()`에 `tick` 종류 추가**

`index.html:903-913`:
```javascript
// 변경 전
function chime(kind){
  if(!state.settings.sound) return;
  const v = Math.max(0.001, state.settings.volume * 0.32);
  if(kind === "pre"){
    [0, 0.16].forEach(t => beep(587.33, t, 0.14, v * 0.55));
    return;
  }
  const notes = kind === "final" ? [523.25, 659.25, 783.99, 1046.50] : [880, 1108.73, 1318.51];
  notes.forEach((f, i) => beep(f, i * 0.17, kind === "final" ? 0.55 : 0.36, v));
}

// 변경 후
function chime(kind){
  if(!state.settings.sound) return;
  const v = Math.max(0.001, state.settings.volume * 0.32);
  if(kind === "pre"){
    [0, 0.16].forEach(t => beep(587.33, t, 0.14, v * 0.55));
    return;
  }
  // 10초 카운트다운 틱 — 예고음(낮은 음 두 번)·종료음(상승 화음)과 겹치지 않는
  // 짧고 높은 단발음. 매초 울리므로 아주 짧게(0.07s) 끊는다.
  if(kind === "tick"){
    beep(1046.5, 0, 0.07, v * 0.5);
    return;
  }
  const notes = kind === "final" ? [523.25, 659.25, 783.99, 1046.50] : [880, 1108.73, 1318.51];
  notes.forEach((f, i) => beep(f, i * 0.17, kind === "final" ? 0.55 : 0.36, v));
}
```

- [ ] **Step 4: 초 경계 추적 변수 추가**

`index.html:933` 부근:
```javascript
// 변경 전
let preWarned = false;             // 이번 단계에서 30초 예고음을 이미 울렸는가

// 변경 후
let preWarned = false;             // 이번 단계에서 30초 예고음을 이미 울렸는가
let lastBeepSecond = 0;            // 10초 카운트다운에서 마지막으로 튼 초(0 = 아직 없음)
```
`index.html:942` (`armStep` 안, `state.overrunFrom = null; preWarned = false;`) 다음에 추가:
```javascript
// 변경 전
  state.overrunFrom = null; preWarned = false;

// 변경 후
  state.overrunFrom = null; preWarned = false; lastBeepSecond = 0;
```

- [ ] **Step 5: `tick()`에 카운트다운 재생 로직 추가**

`index.html:1037-1048`:
```javascript
// 변경 전
function tick(){
  if(state.overrunFrom){ render(); return; }
  if(state.status !== "running") return;
  state.remainingMs = Math.max(0, state.endsAt - Date.now());
  if(state.remainingMs <= 0){ onStepEnd(); return; }
  if(!preWarned && state.settings.preWarn && state.totalMs > 45000 && state.remainingMs <= 30000){
    preWarned = true;
    chime("pre");
  }
  render();
}

// 변경 후
function tick(){
  if(state.overrunFrom){ render(); return; }
  if(state.status !== "running") return;
  state.remainingMs = Math.max(0, state.endsAt - Date.now());
  if(state.remainingMs <= 0){ onStepEnd(); return; }
  if(!preWarned && state.settings.preWarn && state.totalMs > 45000 && state.remainingMs <= 30000){
    preWarned = true;
    chime("pre");
  }
  // 10초 이하 구간(기존 danger 임계값 재사용)에서 초 경계를 지날 때마다 한 번씩
  if(state.settings.countdownBeep && state.remainingMs <= 10000){
    const secLeft = Math.ceil(state.remainingMs / 1000);
    if(secLeft !== lastBeepSecond && secLeft > 0){
      lastBeepSecond = secLeft;
      chime("tick");
    }
  }
  render();
}
```

- [ ] **Step 6: 체크박스 HTML 추가**

`index.html:577` (`preWarnChk` 토글) 다음에 추가:
```html
<div class="toggle"><label for="countdownChk">종료 10초 전부터 매초 효과음</label><input type="checkbox" id="countdownChk"></div>
```

- [ ] **Step 7: `applySettings()`와 이벤트 리스너에 연결**

`index.html:1495` (`$("preWarnChk").checked = s.preWarn;`) 다음에 추가:
```javascript
$("countdownChk").checked = s.countdownBeep;
```
`index.html:1753` 근처(`$("preWarnChk").onchange = ...`가 있는 줄) 다음에 추가:
```javascript
$("countdownChk").onchange = e => { state.settings.countdownBeep = e.target.checked; save(); };
```

- [ ] **Step 8: 수동 검증**

1. 표시 & 소리에서 "종료 10초 전부터 매초 효과음"이 기본으로 켜져 있는지 확인한다.
2. `10` 프리셋 대신 짧은 시간(직접 입력 `0:12` 등)으로 시작해 10초 이하로 내려갈 때 매초 짧고 높은 틱음이 나는지, 기존 예고음·종료음과 소리가 겹치거나 헷갈리지 않는지 귀로 확인한다.
3. 체크박스를 끄고 다시 시작하면 틱음이 나지 않는지 확인한다.
4. 소리 전체 끄기(`종료 알림음` 체크 해제)를 하면 틱음도 함께 꺼지는지 확인한다(`chime`이 `state.settings.sound`를 먼저 확인하므로 자동으로 보장돼야 한다).
5. 새로고침 후 체크박스 상태가 유지되는지 확인한다.

- [ ] **Step 9: 커밋**

```bash
git add index.html
git commit -m "feat: 종료 10초 전부터 매초 카운트다운 효과음을 추가한다"
```

---

### Task 10: B1 — 시간 추가 버튼 (+10초 / +1분)

**Files:**
- Modify: `index.html:490-495` (`.controls` 버튼 마크업)
- Modify: `index.html:1256-1261` (`render()`의 버튼 비활성화 로직)
- Modify: JS — 새 `addTime(ms)` 함수, 이벤트 리스너 등록

**Interfaces:**
- Consumes: Task 6의 `announce(text)`, Task 9의 `lastBeepSecond`/`preWarned`
- Produces: `function addTime(ms)` — 다른 태스크가 의존하지 않음(버튼 클릭 핸들러에서만 호출)

- [ ] **Step 1: 버튼 마크업 추가**

`index.html:490-495`:
```html
<!-- 변경 전 -->
  <div class="controls">
    <button id="startBtn" class="primary">시작</button>
    <button id="resetBtn" title="초기화 (R)">초기화</button>
    <button id="nextBtn" title="다음 단계 (N)">다음 ▶</button>
    <button id="fsBtn" title="전체화면 (F)">⛶ 전체화면</button>
  </div>

<!-- 변경 후 -->
  <div class="controls">
    <button id="startBtn" class="primary">시작</button>
    <button id="resetBtn" title="초기화 (R)">초기화</button>
    <button id="addTenSecBtn" title="10초 추가">+10초</button>
    <button id="addOneMinBtn" title="1분 추가">+1분</button>
    <button id="nextBtn" title="다음 단계 (N)">다음 ▶</button>
    <button id="fsBtn" title="전체화면 (F)">⛶ 전체화면</button>
  </div>
```

- [ ] **Step 2: `addTime(ms)` 함수 작성**

`function next()`(`index.html:1013-1019`) 다음에 추가:
```javascript
/* 실행 중이거나 일시정지 상태일 때 남은 시간을 늘린다.
   totalMs 도 같이 늘려야 게이지 비율(remainingMs/totalMs)이 100%를 넘지 않는다.
   absoluteEnd(이 시각까지 모드)가 있으면 목표 시각 자체를 밀어 "이 시각까지"라는
   의미를 유지한다. running 중이면 endsAt 도 같이 밀어 마감시각 기준 엔진과 어긋나지 않게 한다. */
function addTime(ms){
  if(state.status !== "running" && state.status !== "paused") return;
  const wasWarn = state.remainingMs <= 60000, wasDanger = state.remainingMs <= 10000;
  state.remainingMs += ms;
  state.totalMs += ms;
  if(state.absoluteEnd) state.absoluteEnd += ms;
  if(state.status === "running") state.endsAt += ms;
  // 경고 구간을 벗어났으면 예고음/카운트다운 플래그를 리셋해, 나중에 다시 그 구간에
  // 들어왔을 때 정상적으로 다시 울리게 한다
  if(wasWarn && state.remainingMs > 60000) preWarned = false;
  if(wasDanger && state.remainingMs > 10000) lastBeepSecond = 0;
  render();
  announce(`${ms >= 60000 ? Math.round(ms / 60000) + "분" : Math.round(ms / 1000) + "초"} 추가함, 남은 시간 ${fmt(state.remainingMs)}`);
}
```

- [ ] **Step 3: 버튼 활성화/비활성화를 `render()`에 연결**

`index.html:1259` (`$("nextBtn").disabled = ...` 다음 줄)에 추가:
```javascript
const canAddTime = st.status === "running" || st.status === "paused";
$("addTenSecBtn").disabled = $("addOneMinBtn").disabled = !canAddTime;
```

- [ ] **Step 4: 이벤트 리스너 등록**

`index.html:1574` (`$("resetBtn").onclick = guardedReset;`) 다음에 추가:
```javascript
$("addTenSecBtn").onclick = () => addTime(10000);
$("addOneMinBtn").onclick = () => addTime(60000);
```

- [ ] **Step 5: 수동 검증**

1. 타이머를 시작하지 않은 상태(유휴)에서 `+10초`/`+1분` 버튼이 비활성(회색)인지 확인한다.
2. 프리셋으로 시작한 뒤 `+1분`을 누르면 남은 시간이 정확히 1분 늘고, 게이지가 100%를 넘거나 이상하게 점프하지 않는지 확인한다(링/원판 모드 둘 다).
3. `이 시각까지` 모드로 시작한 뒤 `+10초`를 누르면 왼쪽 위 "종료 예정" 표시도 10초 밀리는지 확인한다.
4. 활동 단계로 실행 중에 `+1분`을 눌러도 같은 큐 항목에 정상 적용되는지 확인한다(모드 구분 없이 동작).
5. 일시정지 상태에서 `+10초`를 눌러도 동작하는지, 남은 시간 표시가 즉시 갱신되는지 확인한다.
6. 남은 시간이 1분 10초일 때 `+10초`를 여러 번 눌러 10초 이하로 내려갔다가 다시 올라가게 만든 뒤, 나중에 자연스럽게 10초 이하로 다시 내려갈 때 Task 9의 카운트다운 효과음이 다시 정상적으로 울리는지 확인한다(리셋 로직 확인).
7. 스크린리더로 버튼을 누르면 "1분 추가함, 남은 시간 …"이 읽히는지 확인한다.
8. 전체가 종료된(`finished`) 상태에서 버튼이 다시 비활성화되는지 확인한다.

- [ ] **Step 6: 커밋**

```bash
git add index.html
git commit -m "feat: 실행 중인 타이머에 +10초/+1분 시간 추가 버튼을 넣는다"
```

---

### Task 11: B3 — 글자색 사용자 지정 (라이트/다크 개별 저장)

**Files:**
- Modify: `index.html:623-626` (`THEME_COLORS`)
- Modify: `index.html:686-690` (`state.settings` 기본값 — `textLight`, `textDark`)
- Modify: `index.html:741-781` (`load()` — sanitize)
- Modify: `index.html:570-575` (표시 & 소리 HTML — 글자색 picker 추가)
- Modify: `index.html:1486-1527` (`applySettings`/`applyBaseColor` 옆에 `applyTextColor` 추가)
- Modify: `index.html:1756-1758` 부근 (이벤트 리스너)

**Interfaces:**
- Consumes: 기존 `isHex`, `contrastRatio`, `THEME_COLORS` 패턴(`applyBaseColor`를 그대로 본뜬다)
- Produces: 없음

- [ ] **Step 1: `THEME_COLORS`에 텍스트 기본값 추가**

`index.html:623-626`:
```javascript
// 변경 전
const THEME_COLORS = {
  light: { accent: "#2563eb", bg: "#f3f5f8" },
  dark:  { accent: "#5b93ff", bg: "#0d1016" }
};

// 변경 후
const THEME_COLORS = {
  light: { accent: "#2563eb", bg: "#f3f5f8", text: "#0E1420" },
  dark:  { accent: "#5b93ff", bg: "#0d1016", text: "#EDF1F8" }
};
```
(`text` 값은 CSS `:root`/`html[data-theme="dark"]`의 `--text`와 동일해야 한다 — `index.html:21`, `:36`.)

- [ ] **Step 2: 설정 기본값 추가**

`index.html:686-690`(Task 9에서 `countdownBeep`을 이미 추가한 상태) 다음에 이어서:
```javascript
// 변경 전
              baseColor:"",
              countdownBeep:true,
              openSecs:["secQuick"],
              quickName:"" },

// 변경 후
              baseColor:"",
              countdownBeep:true,
              textLight:"", textDark:"",   // "" = 테마 기본 글자색을 그대로 쓴다. 테마별로 따로 저장한다
              openSecs:["secQuick"],
              quickName:"" },
```

- [ ] **Step 3: `load()`에 손상값 방어 추가**

Task 9에서 추가한 `countdownBeep` 줄 다음에 추가:
```javascript
if(!isHex(state.settings.textLight)) state.settings.textLight = "";
if(!isHex(state.settings.textDark))  state.settings.textDark  = "";
```

- [ ] **Step 4: HTML에 글자색 picker 추가**

`index.html:570-575`(`baseMsg` div 다음)에 추가:
```html
<!-- 기존 baseMsg div 다음 -->
    <label for="textColor">글자색</label>
    <div class="row" style="margin:6px 0 2px">
      <input type="color" id="textColor" aria-label="글자색 고르기">
      <button id="textReset" style="flex:1">테마 기본값으로</button>
    </div>
    <div class="total" id="textMsg"></div>
```

- [ ] **Step 5: `applyTextColor()` 작성 및 연결**

`function applyBaseColor()`(`index.html:1509-1527`) 다음에 추가:
```javascript
/* 글자색 적용 + 명암비 안내. 기본 타이머 색상과 달리 라이트/다크를 따로 저장한다 —
   글자색은 배경과 짝을 이뤄야 읽히므로, 값 하나를 공유하면 테마 전환 시 글자가
   안 보이는 사고로 바로 이어진다. 본문 글자 기준(4.5:1)은 큰 숫자 기준(3:1)보다 엄격하다. */
function applyTextColor(){
  const s = state.settings;
  const theme = THEME_COLORS[s.theme] || THEME_COLORS.light;
  const key = s.theme === "dark" ? "textDark" : "textLight";
  const custom = s[key];

  if(isHex(custom)) document.body.style.setProperty("--text", custom);
  else               document.body.style.removeProperty("--text");

  const effective = isHex(custom) ? custom : theme.text;
  $("textColor").value = effective;

  const ratio = contrastRatio(effective, theme.bg);
  const msg = $("textMsg");
  if(!isHex(custom)){
    msg.textContent = "테마 기본값 사용 중";
  }else if(ratio < 4.5){
    msg.textContent = `⚠ 배경과 명암비 ${ratio.toFixed(1)}:1 — 글자가 흐려 보일 수 있습니다 (4.5:1 이상 권장)`;
  }else{
    msg.textContent = `명암비 ${ratio.toFixed(1)}:1 — 잘 보입니다`;
  }
}
function setTextColor(v){
  const key = state.settings.theme === "dark" ? "textDark" : "textLight";
  state.settings[key] = isHex(v) ? v : "";
  applyTextColor(); save();
}
```
`function applySettings()`(`index.html:1493` `applyBaseColor();` 다음 줄)에 추가:
```javascript
applyTextColor();
```

- [ ] **Step 6: 이벤트 리스너 등록**

`index.html:1757-1758`(`$("baseColor").oninput`/`$("baseReset").onclick`) 다음에 추가:
```javascript
$("textColor").oninput = e => { setTextColor(e.target.value); };
$("textReset").onclick = () => { setTextColor(""); };
```

- [ ] **Step 7: 수동 검증**

1. 라이트 테마에서 글자색 picker로 눈에 띄게 다른 색(예: 진한 남색 대신 진한 갈색)을 고른다 — 라벨·제목 등 본문 글자색이 즉시 바뀌는지, 명암비 문구가 갱신되는지 확인한다.
2. `T` 키로 다크 테마로 전환한다 — 라이트에서 고른 색이 적용되지 않고, 다크 테마의 (아직 지정 안 한) 기본 글자색이 보이는지 확인한다.
3. 다크 테마에서 다른 색을 고르고 다시 `T`로 라이트로 돌아온다 — 라이트 테마에서 처음 고른 색이 그대로 남아 있는지 확인한다(테마별 독립 저장 확인).
4. 새로고침 후에도 두 테마 각각의 커스텀 글자색이 유지되는지 확인한다.
5. 아주 흐린 색(배경과 명암비 4.5 미만)을 골라 경고 문구(`⚠ ... 4.5:1 이상 권장`)가 뜨는지 확인한다.
6. `테마 기본값으로`를 누르면 해당 테마만 기본값으로 돌아가는지(다른 테마의 커스텀 값은 그대로인지) 확인한다.
7. 내보내기(↓ 내보내기는 템플릿용이라 영향 없음 — 대신) `localStorage`의 `focusTimer.settings` 값을 개발자 도구로 열어 `textLight`/`textDark` 키가 저장돼 있는지 확인한다.

- [ ] **Step 8: 커밋**

```bash
git add index.html
git commit -m "feat: 라이트/다크 테마별로 글자색을 따로 지정할 수 있게 한다"
```

---

### Task 12: C1 — 보조 컨트롤 버튼 아이콘화

**Files:**
- Modify: `index.html:490-497` (`.controls` 버튼 마크업 — `resetBtn`, `nextBtn`, `fsBtn`)
- Modify: CSS 블록 (`.controls button` 근처)

**Interfaces:**
- Consumes: Task 10에서 추가된 `#addTenSecBtn`/`#addOneMinBtn`(그대로 텍스트 유지, 이 태스크에서 건드리지 않음)
- Produces: 없음

- [ ] **Step 1: 아이콘 전용으로 바꿀 근거 정리(코드 변경 없음, 문서화)**

`+10초`/`+1분`은 보편적인 아이콘이 없어(숫자+단위 조합을 한 글자 아이콘으로 줄이면 오히려 교실 뒤에서 못 알아본다) 텍스트를 유지한다. `초기화`/`다음`/`전체화면`은 이미 널리 쓰이는 글리프(↺/▶/⛶)가 있으므로 아이콘 전용으로 바꾼다. 이 판단은 `PLAN.md`의 "교실 뒤에서 읽히는 것이 설계 목적" 원칙(Bahnschrift 폰트 채택 이유)과 같은 맥락이다.

- [ ] **Step 2: 마크업을 아이콘+`aria-label`로 교체**

`index.html:490-497`(Task 10 이후 상태):
```html
<!-- 변경 전 -->
  <div class="controls">
    <button id="startBtn" class="primary">시작</button>
    <button id="resetBtn" title="초기화 (R)">초기화</button>
    <button id="addTenSecBtn" title="10초 추가">+10초</button>
    <button id="addOneMinBtn" title="1분 추가">+1분</button>
    <button id="nextBtn" title="다음 단계 (N)">다음 ▶</button>
    <button id="fsBtn" title="전체화면 (F)">⛶ 전체화면</button>
  </div>

<!-- 변경 후 -->
  <div class="controls">
    <button id="startBtn" class="primary">시작</button>
    <button id="resetBtn" class="icon" title="초기화 (R)" aria-label="초기화">↺</button>
    <button id="addTenSecBtn" title="10초 추가">+10초</button>
    <button id="addOneMinBtn" title="1분 추가">+1분</button>
    <button id="nextBtn" class="icon" title="다음 단계 (N)" aria-label="다음 단계">▶</button>
    <button id="fsBtn" class="icon" title="전체화면 (F)" aria-label="전체화면">⛶</button>
  </div>
```

- [ ] **Step 3: 아이콘 버튼 CSS 추가**

`.controls button{font-size:clamp(.9rem,1.8vmin,1.1rem)}`(`index.html:256`) 다음에 추가:
```css
/* 아이콘만 남은 보조 버튼 — 글리프가 작아 보이지 않게 살짝 키우고, 좌우 패딩을
   좁혀 텍스트 버튼(시작/+10초/+1분)보다 시각적 무게를 줄인다 */
.controls button.icon{font-size:1.3em;line-height:1;padding:8px 14px}
```

- [ ] **Step 4: 수동 검증**

1. 컨트롤 바가 `[시작] [↺] [+10초] [+1분] [▶] [⛶]`처럼 보이는지, 6개가 늘어나도 줄바꿈 없이 한 줄에 들어가는지(가장 좁은 지원 폭 기준으로도) 확인한다.
2. 각 아이콘 버튼에 마우스를 올리면 `title` 툴팁(초기화 (R) 등)이 보이는지 확인한다.
3. 개발자 도구에서 `#resetBtn`/`#nextBtn`/`#fsBtn`의 접근성 이름(Accessible Name)이 `aria-label`로 정확히 잡히는지 확인한다(브라우저 접근성 검사기 또는 스크린리더로 Tab 이동하며 확인).
4. 실제 클릭 동작(초기화 확인창, 다음 단계 이동, 전체화면 토글)이 아이콘으로 바뀐 뒤에도 그대로인지 확인한다.
5. 라이트/다크 테마 모두에서 아이콘 글자가 잘 보이는지 확인한다.

- [ ] **Step 5: 커밋**

```bash
git add index.html
git commit -m "feat: 초기화·다음·전체화면 버튼을 아이콘 전용으로 단순화한다"
```

---

## Self-Review

**Spec coverage:**
- A1→Task 6, A2→Task 7, A3→Task 5, A4→Task 8, A5→Task 8, B1→Task 10, B2→Task 9, B3→Task 11, C1→Task 12, C2→Task 4, D1→Task 1, D2→Task 2, D3→Task 3, D4→Task 2, D5→Task 2. 15개 스펙 항목 모두 태스크로 매핑됨.
- 스펙의 "빠른 시작 → 바로" 개명 계획은 브레인스토밍 중 재검토 끝에 **기각**됐고(`사용설명서.md`의 기존 "바로 시작하려면" 문구와 충돌), 스펙·이 계획 모두 `활동 단계 → 활동`만 반영하도록 이미 수정됨(Task 1).

**Placeholder scan:** "TBD"/"나중에"/"적절히 처리" 같은 자리표시자 없음. 모든 스텝에 실제 코드/문구가 들어 있음.

**Type/name consistency 확인:**
- `announce()`(Task 6 도입) — Task 10에서 동일한 이름·시그니처(`announce(text)`)로 재사용됨.
- `lastBeepSecond`(Task 9 도입) — Task 10에서 동일 변수명으로 리셋됨.
- `preWarned`(기존) — Task 10에서 동일 변수명으로 리셋됨.
- `applyTextColor()`(Task 11) — `applyBaseColor()`와 동일한 패턴(`isHex`/`contrastRatio`/`$("...").value` 갱신)을 따름, `applySettings()`에서 같은 방식으로 호출됨.
- `#panelBtn`(Task 4에서 스타일만 변경) — Task 5(A3)에서 동일 id로 포커스 대상 삼음. id를 바꾸지 않았으므로 `aria-controls="panel"`, `applySettings()`의 `$("panelBtn").setAttribute("aria-expanded", ...)` 참조 모두 그대로 유효.
- `.sec-title`/`.sec-sub`(Task 2) — Task 3이 그 바깥에 `.sec-group` wrapper만 추가하고 내부 마크업은 그대로 둠, 충돌 없음.
