---
name: study-on-canvas
description: HTML 학습 노트를 옆에 띄워 놓고 LLM과 대화하며 알게 된 개념을 실시간으로 기록한다. 개념을 설명할 때마다 후속 개념을 1~3개 사이드바에 쌓아, 아직 다루지 않은 형제 개념(다른 대화로 빠지며 잊힌 개념)을 놓치지 않도록 한다. 설명문은 세션 눈높이에 맞춰 쉽게·기본·깊게 세 레이어로 쌓고, 이해가 막히면 하위 개념으로 내려가는 대신 같은 개념을 더 쉬운 레이어로 다시 말해 준다. 설명 후 다음 주제를 먼저 묻거나 추천하지 않는다 — 무엇을 다룰지는 사용자가 정한다. 발동 조건 — 사용자가 `/study-on-canvas` 슬래시 명령을 직접 입력했을 때만 발동한다. 발동하지 않는 경우 — "캔버스로 대화하자", "학습 노트 켜줘" 같은 자연어 언급, "X 설명해줘", "쉽게 설명해줘", "정리해줘" 같은 일반 질문, 그리고 이미 캔버스 세션이 열려 있는 상태(이때는 재발동하지 않고 진행 중인 세션의 설명 사이클을 이어간다).
user-invocable: true
allowed-tools: Read, Edit, Write, Bash, AskUserQuestion
---

# 학습 노트 루틴 (`/study-on-canvas`)

LLM과 대화하며 새 개념을 이해할 때, 대화 내용을 실시간 HTML 노트에 기록하면서 아직 다루지 않은 후속 개념을 사이드바 `추천 하위 개념`에 남겨 놓치지 않도록 돕는 스킬.

사이드바에는 등장한 개념이 순서대로 계속 쌓인다. 다룬 개념도 지우지 않고, 노란색으로 표시만 바꿔 자리를 지킨다.

개념을 어느 깊이로 설명할지는 세션 눈높이와 설명 레이어가 정한다. 기준은 `references/audience-calibration.md` 에 있다. 설명문을 처음 쓰기 전에 그 문서를 읽는다.

## 발동 조건

`/study-on-canvas` 슬래시 명령으로만 발동한다. 자연어 호출은 지원하지 않는다.

설명·요약 계열 스킬(eli5 등)은 트리거 범위가 넓다. 자연어로도 발동하게 두면 "쉽게 설명해줘" 한마디에 다른 스킬이 끼어들어, 터미널에만 답하고 HTML 을 갱신하지 않는 사이클이 생긴다. 노트가 대화에 뒤처지는 순간 이 스킬은 쓸모를 잃으므로 진입점을 슬래시 명령 하나로 좁혀 둔다.

세션이 열려 있는 동안에는 사용자가 어떤 표현을 쓰든 아래 개념 설명 사이클을 유지한다. 다른 설명 스킬이 함께 발동하더라도 HTML 기록 단계는 건너뛰지 않는다.

## 핵심 원칙 (반드시 지킬 것)

1. 후속 개념은 매번 1~3개. 4개 이상 절대 금지. 사용자가 한 번에 압도되지 않도록.
2. 매 응답 사이클은 개념 설명 → 즉시 HTML 업데이트에서 끝난다. 여기서 멈춘다.
3. 설명을 마친 뒤 다음 행동을 묻거나 추천하지 않는다. "다음은 무엇을 볼까요", "A를 이어서 보면 좋습니다" 같은 유도 문장, 후속 개념을 고르게 하는 `AskUserQuestion` 모두 금지. 사이드바와 개념 지도가 선택지를 보여주므로 다음 주제는 사용자가 알아서 지목한다.
4. 파일 편집은 마커 기반 Edit만 사용. 파일 전체 재작성 금지. 개별 항목은 개별 마커로 감싸 정확히 타겟팅.
5. 노트의 언어는 세션 언어를 따른다 — 아래 "노트 언어" 절 참조. 기본값은 한국어.
6. 후속 개념의 정의: (a) 현재 개념 이해에 기여하는 하위 개념, 또는 (b) 사용자가 스스로 사고할 질문거리. 둘 중 무엇을 고를지는 세션 눈높이를 따른다.
7. 이해가 막혔다는 신호("잘 모르겠어", "더 쉽게")에는 하위 개념으로 내려가지 않는다. 같은 개념의 `쉽게` 레이어를 채우는 것으로 답한다. 아래로 파는 것과 옆에서 다시 말하는 것은 다른 동작이다.

## 노트 언어

사용자가 스킬을 부른 문장의 언어로 노트 UI 문구를 정한다. 한국어면 `ko`, 영어면 `en`. 판단이 애매하면 `ko`를 쓴다. 세션 도중에는 바꾸지 않는다.

템플릿 치환에 쓸 사전:

| 플레이스홀더 | ko | en |
| --- | --- | --- |
| `{{LANG}}` | `ko` | `en` |
| `{{L_TOPIC}}` | 주제 | Topic |
| `{{L_STARTED}}` | 시작일 | Started |
| `{{L_SUGGESTED}}` | 추천 하위 개념 | Suggested subtopics |
| `{{L_EMPTY}}` | 추천할 하위 개념이 없습니다. | No suggested subtopics yet. |
| `{{L_MAP}}` | 개념 지도 | Concept map |
| `{{L_MAP_OPEN}}` | 개념 지도 열기 | Open concept map |
| `{{L_MAP_EMPTY}}` | 아직 지도에 그릴 개념이 없습니다. | Nothing to map yet. |
| `{{L_CLOSE}}` | 닫기 | Close |
| `{{L_AUDIENCE}}` | 눈높이 | Level |
| `{{L_LV_EASY}}` | 쉽게 | Simple |
| `{{L_LV_MAIN}}` | 기본 | Standard |
| `{{L_LV_DEEP}}` | 깊게 | Deeper |

`{{L_LV_*}}` 는 템플릿 파일에 미리 박혀 있지 않다. 패널을 만들 때 레이어 전환 버튼 마크업을 넣으면서 해당 언어의 라벨을 직접 적는다.

`{{AUDIENCE}}` 는 언어 사전이 아니라 Step 1 에서 사용자가 고른 눈높이 라벨로 채운다 — ko 는 `입문` · `인접` · `실무`, en 은 `Newcomer` · `Adjacent` · `Practitioner`.

패널 breadcrumb 문구도 같은 언어를 따른다.

- ko: `상위 개념: <이름>` / 루트는 `상위 개념: (루트)`
- en: `Parent: <이름>` / 루트는 `Parent: (root)`

개념 설명 본문과 사용자에게 보내는 터미널 메시지도 같은 언어로 쓴다.

## 실행 플로우

### Step 1: 세션 시작

사용자가 `/study-on-canvas <주제>`를 실행하면:

0. 주제 확정 — 슬래시 커맨드 인자에 주제가 들어 있으면 그걸 루트 주제로 쓴다. 인자 없이 `/study-on-canvas` 만 실행했으면 `AskUserQuestion`으로 무엇을 공부할지 한 번 묻는다. 직전 대화에서 다루던 주제가 있으면 그것을 첫 옵션으로 제시한다.

1. 기준 경로와 날짜 확보 — Bash 한 번으로 둘 다 잡아 둔다. 이후 모든 파일 조작은 여기서 나온 절대 경로만 쓴다.

   ```bash
   echo "BASE=$(pwd)"; echo "TODAY=$(date +%Y-%m-%d)"
   ```

   `$(pwd)`는 세션이 시작된 작업 폴더다. 사용자가 대화 도중 다른 폴더를 언급해도 기준은 바뀌지 않는다.

2. 저장 경로와 눈높이 확인 — `AskUserQuestion` 한 번에 두 질문을 담아 묻는다. 아래 "AskUserQuestion 사용 패턴" 참조.

   저장 경로:
   - 기본 옵션 라벨: `<BASE>/learning-notes/<TODAY>-<슬러그>.html`
   - 대안: `다른 경로 지정` (Other로 자유 입력)
   - 슬러그 규칙: 한글·비ASCII 문자는 그대로 유지, 공백은 하이픈으로 치환, 파일시스템 예약 문자(`/ \ : * ? " < > |`)만 제거.
   - 사용자가 상대 경로를 입력하면 `<BASE>` 기준으로 절대 경로화한다.

   눈높이: `입문` · `인접` · `실무` 중 하나. 세션 내내 고정한다. 판단 기준과 각 값이 설명문에 미치는 영향은 `references/audience-calibration.md` 에 있다. 이 값을 정한 직후 그 문서를 읽고, 이후 모든 설명문을 그 기준으로 쓴다.

3. 디렉토리 생성 & 템플릿 복사 — Bash로:

   ```bash
   TPL="${CLAUDE_PLUGIN_ROOT}/skills/study-on-canvas/template/study-note.html"
   [ -f "$TPL" ] || TPL="$(ls -1 \
     "$HOME/.claude/skills/study-on-canvas/template/study-note.html" \
     ".claude/skills/study-on-canvas/template/study-note.html" 2>/dev/null | head -1)"
   mkdir -p "$(dirname "$NOTE")"
   cp "$TPL" "$NOTE"
   ```

   `CLAUDE_PLUGIN_ROOT`은 플러그인으로 설치했을 때 자동으로 채워진다. 수동 복사로 설치한 경우를 위해 폴백 탐색을 남겨 둔다. 둘 다 실패하면 템플릿을 찾지 못했다고 알리고 멈춘다.

4. 템플릿 치환 — Edit 도구로 치환한다.
   - `{{TITLE}}` → 루트 주제 이름
   - `{{ROOT_TOPIC}}` → 루트 주제 이름
   - `{{DATE}}` → Step 1-1에서 얻은 `TODAY`
   - `{{LANG}}`과 `{{L_*}}` → "노트 언어" 표의 해당 언어 값
   - `{{AUDIENCE}}` → Step 1-2에서 고른 눈높이 라벨

   치환 후 파일에 `{{`가 남아 있지 않은지 `grep -n '{{' "$NOTE"` 로 확인한다.

5. 루트 주제를 첫 탭으로 등록 — 아래 "HTML 조작 사양"의 "새 개념 추가" 절차를 그대로 따르되, 상위 개념 없음(루트).

6. 브라우저로 열기 — 기본 브라우저로 연다. OS별로 명령이 다르다.

   ```bash
   case "$(uname -s)" in
     Darwin) open "$NOTE" ;;
     Linux) xdg-open "$NOTE" >/dev/null 2>&1 & ;;
     MINGW*|MSYS*|CYGWIN*) start "" "$NOTE" ;;
     *) false ;;
   esac || echo "자동 열기 실패 — 브라우저에서 직접 열어주세요"
   ```

   그 뒤 사용자에게 안내한다.

   ```
   학습 노트 생성 & 브라우저에서 열었습니다: <경로>
   파일 갱신 시 새로고침(F5 / ⌘R) 하시면 됩니다.
   ```

   자동 열기가 실패하면 경로만 알려주고 직접 열도록 안내한다. 스킬 동작에는 영향이 없다.

### Step 2: 개념 설명 사이클

새 개념 A를 다룰 때마다:

1. 새 탭·패널 추가 — "HTML 조작 사양 › 새 개념 추가" 참조. 사이드바에 이미 카드가 있는 개념이면 카드는 손대지 않는다. 패널이 생기는 순간 그 카드는 자동으로 노란색 처리된다.
2. 터미널에 요약 설명 — 세션 눈높이에 맞춘 한 화면 분량의 설명. 필요한 경우 예시·비유 포함.
3. 같은 내용을 패널의 `기본` 레이어에 HTML로 삽입 — "HTML 조작 사양 › 탭 내용 채우기" 참조. 이 시점에 `쉽게`·`깊게` 레이어는 만들지 않는다.
4. 후속 개념 1~3개 결정 — 하위 개념 또는 사고 유도 질문. 무엇을 고를지는 세션 눈높이를 따른다 (`references/audience-calibration.md` 5절).
5. 후속 개념을 사이드바에 카드로 추가 — "HTML 조작 사양 › 후속 개념 등록" 참조.

여기서 사이클이 끝난다. 사용자가 다음 개념을 지목하면 그때 이 사이클을 다시 돈다.

`쉽게`·`깊게` 레이어는 선제적으로 채우지 않는다. 사용자가 요청할 때만 붙인다 — 아래 Step 3 참조.

### Step 3: 특수 케이스

- 사이드바 밖 주제를 사용자가 물으면: 신규 개념으로 취급해 탭·패널을 만들고, 사이드바에도 카드를 추가한다. 상위 개념이 분명하면 그 슬러그를 `data-parent`에 적고, 없으면 비워 둔다(개념 지도에서 별도 뿌리로 선다).
- "C로 돌아가고 싶어" 같은 요청: 사용자가 브라우저에서 탭이나 카드를 클릭하면 된다. 원한다면 터미널에서도 해당 개념부터 다시 설명하며 그 탭에 내용을 추가해도 됨.

#### 더 쉽게 — 이해가 막혔을 때

"잘 모르겠어", "더 쉽게", "무슨 말인지 모르겠다" 같은 신호가 오면 새 개념 사이클을 돌지 않는다. 방금 다룬 개념의 패널에 `쉽게` 레이어를 추가하는 것으로 답한다.

지켜야 할 것:

- 탭을 새로 만들지 않는다. 기존 패널 안에 레이어만 하나 늘어난다.
- 후속 개념 카드를 추가하지 않는다. 이해를 돕는 동작이지 탐색을 넓히는 동작이 아니다. 사이드바는 그대로 둔다.
- `기본` 레이어의 기존 설명은 지우지 않는다. 쉬운 설명과 원래 설명이 나란히 남아야 어디를 단순화했는지 보인다.
- 비유 하나로 3문장 이내. 정확도를 80%까지 깎아도 된다.

#### 더 깊게 — 한 걸음 더 들어갈 때

"왜 그렇게 되는지", "예외는 없는지", "다른 방법과 비교하면" 같은 질문에는 같은 방식으로 `깊게` 레이어를 추가한다. 이것도 탭·카드를 만들지 않는다.

다만 답이 독립된 개념 하나로 설 만큼 크면 그때는 새 개념 사이클로 넘긴다. 판단 기준은 분량이다 — 스크롤 한 화면을 넘기면 개념으로 분리한다.

#### Teach-back 요약 — 배운 것을 남에게 설명해 보기

사용자가 "정리해서 매니저한테 설명하고 싶어", "요약해줘" 처럼 특정 대상에게 전달할 요약을 요청하면 노트 맨 끝에 요약 탭을 하나 만든다. 배운 내용을 남의 눈높이로 옮겨 보는 일 자체가 이해 점검이다.

- 탭 라벨: ko 는 `요약: <대상>에게`, en 은 `Teach-back: <audience>`. 탭 버튼에 `teachback` 클래스를 붙인다.
- 슬러그는 `teachback-<대상슬러그>` (예: `teachback-manager`). 대상이 여럿이면 각각 탭을 만든다.
- 사이드바 카드를 만들지 않는다. `data-parent` 도 쓰지 않는다. 개념이 아니라 산출물이라 개념 지도에 들어가면 안 된다.
- 내용은 노트에 이미 있는 개념만으로 구성한다. 요약하면서 새 개념을 끌어오지 않는다.
- 대상별 프레이밍은 `references/audience-calibration.md` 3절 표를 따른다. 매니저면 결정과 리스크, 디자이너면 사용자 경험 쪽으로 기운다.
- 레이어는 `기본` 하나만 쓴다.

## 화면 구성 (템플릿이 알아서 처리하는 것)

- 다크 테마 고정. 색은 `:root` CSS 변수로만 정의돼 있으니 색을 바꿀 일이 있으면 변수만 고친다.
- 데스크톱: 좌측 본문 + 우측 320px 사이드바. 화면 높이를 꽉 채우고 본문·사이드바가 각자 안에서 스크롤한다.
- 900px 이하 모바일: 사이드바가 하단 시트로 바뀌고, 우하단 FAB을 눌러야 열린다. 스크림 탭·핸들 탭·Esc로 닫힌다.
- 사이드바 카드는 `data-parent`(상위 개념 슬러그)로 묶여, 같은 상위 개념끼리 그룹 제목 아래 모인다. 제목 줄(`.group-title`)은 로드 시 JS가 만들어 붙이므로 HTML에 직접 쓰지 않는다. 그룹 순서와 그룹 안 순서는 카드가 등장한 순서를 그대로 따른다.
- 설명문(패널)이 있는 개념의 카드는 노란색으로 자동 전환된다. 판정 기준은 같은 `data-id`를 가진 `.panel`의 존재 여부 하나뿐이라, 탭·패널을 만들면 그걸로 끝이다. 카드에 클래스를 손으로 붙이지 않는다.
- 노란 카드는 클릭하면 그 설명문으로 이동한다. 중립색 카드(아직 설명문 없음)는 클릭해도 아무 일도 일어나지 않는다.
- 헤더 우상단 지도 버튼을 누르면 탭·본문·사이드바 자리에 개념 지도가 뜬다. 카드의 상·하위 관계를 트리로 그린 화면이고, 색 규칙은 사이드바와 같다. 닫기 버튼이나 Esc로 빠져나온다. 지도는 열 때마다 현재 카드 목록으로 다시 그려지므로 별도 관리가 필요 없다.
- FAB의 개수 배지와 빈 상태 문구는 로드 시 카드 개수를 세어 자동 갱신된다. 카드가 0개면 FAB 자체가 숨는다.
- 헤더의 눈높이 배지는 `{{AUDIENCE}}` 치환으로 한 번 박히고 그 뒤로 바뀌지 않는다.
- 설명 레이어 전환 버튼은 로드 시 JS가 패널마다 다시 계산한다. 실제로 존재하는 레이어의 버튼만 남기고, 레이어가 하나뿐이면 버튼 묶음 전체를 숨긴다. 즉 `쉽게`·`깊게` 를 만들지 않은 패널에서는 전환 UI가 아예 보이지 않는다.
- 기본으로 펼쳐지는 레이어는 `기본`이다. `.explanation` 에 `data-level` 을 다르게 적으면 그 레이어가 먼저 뜬다.

## HTML 조작 사양

템플릿에는 다음 마커가 배치되어 있다. 모든 삽입·삭제는 이 마커 사이에서만 수행한다.

```
<!-- TABS_INSERT -->              ← 탭 바 삽입 지점
<!-- PANELS_INSERT -->            ← 탭 콘텐츠 삽입 지점
<!-- SIDEBAR_SUGGESTED_INSERT --> ← 추천 하위 개념 리스트 삽입 지점
```

각 항목은 개별 마커로 감싸 개별 Edit이 가능하다. id는 kebab-case 슬러그다 (예: `transformer`, `self-attention`, `positional-encoding`).

### 새 개념 추가 (탭 + 패널)

Edit 도구로 탭 바의 맨 뒤에 삽입한다. 기존 탭이 있으면 마지막 `<!-- /ITEM:<id>:TAB -->` 바로 다음 줄에, 하나도 없으면 `<!-- TABS_INSERT -->` 바로 다음 줄에 넣는다. 탭은 다룬 순서대로 쌓여야 하므로 앞에 끼워 넣지 않는다.

```html
<!-- ITEM:transformer:TAB -->
      <button class="tab" data-id="transformer">Transformer</button>
<!-- /ITEM:transformer:TAB -->
```

패널도 같은 규칙으로 맨 뒤에 삽입한다 (마지막 `<!-- /ITEM:<id>:PANEL -->` 다음, 없으면 `<!-- PANELS_INSERT -->` 다음):

```html
<!-- ITEM:transformer:PANEL -->
      <article class="panel" data-id="transformer">
        <h2>Transformer</h2>
        <div class="breadcrumb">상위 개념: <a class="parent-link" data-id="attention">Attention</a></div>
        <div class="explanation" data-level="main">
          <!-- 전환 버튼 + 레이어 — 아래 "탭 내용 채우기" 절의 골격을 그대로 넣는다 -->
        </div>
      </article>
<!-- /ITEM:transformer:PANEL -->
```

breadcrumb 라벨은 "노트 언어" 절의 문구를 쓴다. 루트 주제의 경우 breadcrumb 줄은 생략하거나 `상위 개념: (루트)`로 표기. 화살표(`↳`) 기호는 쓰지 않는다.

### 탭 내용 채우기 (설명 레이어)

`.explanation` 안은 전환 버튼 묶음 하나와 레이어들로 이루어진다. 새 개념을 만들 때는 항상 이 골격을 통째로 넣는다. 버튼 세 개는 다 적어 두되 레이어는 `기본` 하나만 채운다 — 없는 레이어의 버튼은 JS가 알아서 숨긴다.

```html
        <div class="explanation" data-level="main">
          <div class="layer-switch" role="group">
            <button class="layer-btn" type="button" data-level="eli5">쉽게</button>
            <button class="layer-btn" type="button" data-level="main">기본</button>
            <button class="layer-btn" type="button" data-level="deep">깊게</button>
          </div>
<!-- LAYER:transformer:main -->
          <div class="layer" data-level="main">
            <!-- 여기에 개념 설명 HTML 삽입 -->
          </div>
<!-- /LAYER:transformer:main -->
        </div>
```

버튼 라벨은 "노트 언어" 표의 `{{L_LV_*}}` 값을 쓴다.

레이어를 나중에 덧붙일 때도 마커로 감싼다. `쉽게`는 `<!-- LAYER:<id>:main -->` 바로 앞에, `깊게`는 `<!-- /LAYER:<id>:main -->` 바로 뒤에 넣어 쉽게 → 기본 → 깊게 순서를 지킨다.

```html
<!-- LAYER:transformer:eli5 -->
          <div class="layer" data-level="eli5">
            <p>비유 한 단락.</p>
          </div>
<!-- /LAYER:transformer:eli5 -->
```

각 레이어 안에서 쓸 수 있는 마크업:

- `<p>`, `<ul>`, `<ol>`, `<strong>`, `<em>`
- `<code>`, `<pre><code>`
- `<blockquote>`

분량 기준:

| 레이어 | 분량 |
| --- | --- |
| `쉽게` | 3문장 이내. 비유 하나 |
| `기본` | 스크롤 1~2 화면. 넘치면 후속 개념으로 분리 |
| `깊게` | 필요한 만큼. 한 화면을 넘기면 별도 개념으로 분리 |

### teach-back 요약 탭 추가

개념 탭과 같은 위치에 같은 방식으로 넣되, 탭 버튼에 `teachback` 클래스를 붙이고 사이드바 카드는 만들지 않는다.

```html
<!-- ITEM:teachback-manager:TAB -->
      <button class="tab teachback" data-id="teachback-manager">요약: 매니저에게</button>
<!-- /ITEM:teachback-manager:TAB -->
```

패널의 breadcrumb 자리에는 상위 개념 대신 대상을 적는다 (`대상: 매니저`). 레이어는 `기본` 하나만 쓴다.

### 후속 개념 등록 (추천 하위 개념)

Edit 도구로 카드 목록의 맨 뒤에 삽입한다. 기존 카드가 있으면 마지막 `<!-- /SB:<id>:SUGGESTED -->` 바로 다음 줄에, 하나도 없으면 `<!-- SIDEBAR_SUGGESTED_INSERT -->` 바로 다음 줄에 넣는다. 카드는 등장 순서대로 쌓여야 하므로 앞에 끼워 넣지 않는다.

```html
<!-- SB:self-attention:SUGGESTED -->
        <li class="sidebar-item suggested" data-id="self-attention" data-parent="transformer">
          <span class="label">Self-Attention</span>
        </li>
<!-- /SB:self-attention:SUGGESTED -->
```

`data-parent`에는 상위 개념의 슬러그(그 개념의 `data-id`)를 적는다. 표시 이름이 아니라 슬러그다 — 그룹 제목과 개념 지도의 부모-자식 연결이 이 값으로 이뤄진다. 상위 개념이 없으면 비우거나 속성을 생략한다.

### 다룬 개념 표시 (편집 없음)

개념을 실제로 다뤄 탭·패널이 생기면 그 카드는 노란색으로 바뀐다. 카드는 지우지도, 옮기지도, 고치지도 않는다. 순서도 그대로다.

이 전환은 로드 시 JS가 `data-id`와 같은 `.panel`이 있는지 보고 판정한다. 즉 편집할 것이 없다. 카드에 `covered` 클래스를 손으로 붙이지 말 것 — 붙여도 다시 계산되므로 의미가 없고, 패널 없이 붙이면 지워진다.

## AskUserQuestion 사용 패턴

`AskUserQuestion`은 세션 시작 시 저장 경로와 눈높이를 물을 때만 쓴다(주제가 주어지지 않았으면 주제도 함께). 설명을 마친 뒤 다음 개념을 고르게 하는 용도로는 쓰지 않는다.

### 세션 시작 시 저장 경로 + 눈높이

한 번의 호출에 두 질문을 담는다.

```json
[
  {
    "question": "학습 노트를 어디에 저장할까요?",
    "header": "저장 경로",
    "multiSelect": false,
    "options": [
      {
        "label": "./learning-notes/2026-07-21-트랜스포머.html",
        "description": "현재 작업 폴더의 learning-notes/ 아래에 저장 (기본)"
      },
      {
        "label": "다른 경로 지정",
        "description": "Other 선택 후 원하는 절대 경로 또는 상대 경로 입력"
      }
    ]
  },
  {
    "question": "이 주제를 어느 눈높이로 설명할까요?",
    "header": "눈높이",
    "multiSelect": false,
    "options": [
      {
        "label": "입문",
        "description": "이 분야를 처음 본다. 용어가 나오면 그 자리에서 정의하고 비유를 많이 쓴다"
      },
      {
        "label": "인접",
        "description": "주변 분야 경험이 있다. 아는 개념과의 대조로 설명한다 (기본값)"
      },
      {
        "label": "실무",
        "description": "그 분야에서 일한다. 기초는 건너뛰고 trade-off와 엣지 케이스에 분량을 쓴다"
      }
    ]
  }
]
```

경로 라벨에는 Step 1-1에서 얻은 실제 `BASE`·`TODAY` 값을 채워 넣는다. 라벨이 길어지면 `~` 축약형으로 보여줘도 되지만, 실제 파일 조작은 절대 경로로 한다.

## 예시 흐름

```
User: /study-on-canvas 트랜스포머

LLM: [Bash로 BASE·TODAY 확보]
     [AskUserQuestion 저장 경로 + 눈높이 → 기본 경로 · 인접]
     [파일 생성: <BASE>/learning-notes/2026-07-21-트랜스포머.html]
     [references/audience-calibration.md 읽기]
     [Edit로 트랜스포머 탭·패널 삽입 — 기본 레이어만 채움]

     트랜스포머는 2017년 "Attention Is All You Need"에서 제안된 신경망 구조로,
     기존 RNN 없이 self-attention만으로 시퀀스를 처리합니다. ...

     [Edit로 카드 3개 삽입: Self-Attention / Positional Encoding / Multi-Head Attention]
     (여기서 끝. 다음 주제를 묻지 않는다)

User: self-attention 알려줘

LLM: [Edit로 Self-Attention 탭·패널 추가 — 카드는 손대지 않음]

     Self-Attention은 시퀀스 안의 각 토큰이 다른 모든 토큰과 얼마나 관련 있는지
     계산하는 메커니즘입니다. Query · Key · Value 세 벡터를 만들어... ...

     [Edit로 카드 2개 삽입: Query·Key·Value 계산 / Scaled Dot-Product]
     (여기서 끝)

User: Query Key Value가 뭔 소린지 모르겠어

LLM: [Edit로 self-attention 패널에 쉽게 레이어만 추가 — 탭·카드 손대지 않음]

     도서관에서 책을 찾는다고 해봅시다. 내가 들고 간 검색어가 Query,
     책등에 붙은 제목표가 Key, 책 내용이 Value 입니다. ...
     (여기서 끝. 새 개념으로 취급하지 않는다)

User: 이거 팀에 공유해야 하는데 정리해줘

LLM: [Edit로 teachback-team 탭·패널 추가 — 사이드바 카드는 만들지 않음]
```

이 시점에서 화면은:

- 탭: 트랜스포머, Self-Attention, 요약: 팀에게(구분선 뒤 노란색)
- Self-Attention 패널: `쉽게`·`기본` 두 버튼이 뜨고 기본이 펼쳐져 있다. 트랜스포머 패널은 레이어가 하나뿐이라 버튼이 보이지 않는다
- 사이드바: `트랜스포머` 제목 아래 Self-Attention(노란색)·Positional Encoding·Multi-Head Attention, `Self-Attention` 제목 아래 Query·Key·Value 계산·Scaled Dot-Product
- 개념 지도: 트랜스포머 아래 세 갈래, 그중 Self-Attention 아래 다시 두 갈래. 요약 탭은 지도에 없다

Positional Encoding·Multi-Head Attention은 Self-Attention 대화로 빠지는 동안에도 사이드바에 남아있어, 사용자가 언제든 되돌아가 다룰 수 있다. "모르겠어"에 하위 개념을 더 얹지 않은 덕에 사이드바도 불어나지 않았다.

## 주의

- 사용자가 파일 위치를 수동으로 옮기거나 이름을 바꿔도 스킬 동작에는 영향 없음 (Edit는 현재 세션의 파일 경로만 쓴다). 세션 재시작 시에는 새 `/study-on-canvas`로 새 파일을 만들면 된다.
- 노트가 git 저장소 안에 생기면 커밋에 딸려 들어갈 수 있다. 원치 않으면 `learning-notes/`를 `.gitignore`에 넣도록 안내한다.
- HTML 편집 실패(마커 오탐, indentation 불일치 등)가 발생하면 그 편집만 되돌리고 다시 시도. 파일 전체 재작성은 마지막 수단.
- 같은 세션 안에서 동일한 슬러그(id)가 중복되지 않도록 주의. 중복 위험 시 뒤에 `-2`, `-3` 등 접미사 부여. teach-back 탭은 `teachback-` 접두사를 쓰므로 개념 슬러그와 부딪히지 않는다.
- 레이어를 추가할 때 같은 `data-level`이 한 패널에 두 번 들어가지 않도록 주의. 이미 있으면 새로 넣지 말고 그 레이어 안의 내용을 고친다.
- 눈높이는 세션 도중 바꾸지 않는다. 사용자가 수준 변경을 요청하면 노트 하나를 두 수준이 섞인 상태로 만들지 말고 새 세션을 권한다.
