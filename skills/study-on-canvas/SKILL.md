---
name: study-on-canvas
description: 사용자가 `/study-on-canvas <주제>`로 학습 세션을 시작하면 HTML 학습 노트를 생성하고, LLM과 대화하며 알게 된 개념을 실시간으로 기록한다. 개념을 설명할 때마다 후속 개념을 1~3개 사이드바에 쌓아, 아직 다루지 않은 형제 개념(다른 대화로 빠지며 잊힌 개념)을 놓치지 않도록 한다. 설명 후 다음 주제를 먼저 묻거나 추천하지 않는다 — 무엇을 다룰지는 사용자가 정한다.
disable-model-invocation: true
user-invocable: true
allowed-tools: Read, Edit, Write, Bash, AskUserQuestion
---

# 학습 노트 루틴 (`/study-on-canvas`)

LLM과 대화하며 새 개념을 이해할 때, 대화 내용을 실시간 HTML 노트에 기록하면서 아직 다루지 않은 후속 개념을 사이드바 `추천 하위 개념`에 남겨 놓치지 않도록 돕는 스킬.

사이드바에는 등장한 개념이 순서대로 계속 쌓인다. 다룬 개념도 지우지 않고, 취소선과 노란색으로 표시만 바꿔 자리를 지킨다.

## 핵심 원칙 (반드시 지킬 것)

1. 후속 개념은 매번 1~3개. 4개 이상 절대 금지. 사용자가 한 번에 압도되지 않도록.
2. 매 응답 사이클은 개념 설명 → 즉시 HTML 업데이트에서 끝난다. 여기서 멈춘다.
3. 설명을 마친 뒤 다음 행동을 묻거나 추천하지 않는다. "다음은 무엇을 볼까요", "A를 이어서 보면 좋습니다" 같은 유도 문장, 후속 개념을 고르게 하는 `AskUserQuestion` 모두 금지. 사이드바와 개념 지도가 선택지를 보여주므로 다음 주제는 사용자가 알아서 지목한다.
4. 파일 편집은 마커 기반 Edit만 사용. 파일 전체 재작성 금지. 개별 항목은 개별 마커로 감싸 정확히 타겟팅.
5. 노트의 언어는 세션 언어를 따른다 — 아래 "노트 언어" 절 참조. 기본값은 한국어.
6. 후속 개념의 정의: (a) 현재 개념 이해에 기여하는 하위 개념, 또는 (b) 사용자가 스스로 사고할 질문거리.

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

패널 breadcrumb 문구도 같은 언어를 따른다.

- ko: `상위 개념: <이름>` / 루트는 `상위 개념: (루트)`
- en: `Parent: <이름>` / 루트는 `Parent: (root)`

개념 설명 본문과 사용자에게 보내는 터미널 메시지도 같은 언어로 쓴다.

## 실행 플로우

### Step 1: 세션 시작

사용자가 `/study-on-canvas <주제>`를 실행하면:

1. 기준 경로와 날짜 확보 — Bash 한 번으로 둘 다 잡아 둔다. 이후 모든 파일 조작은 여기서 나온 절대 경로만 쓴다.

   ```bash
   echo "BASE=$(pwd)"; echo "TODAY=$(date +%Y-%m-%d)"
   ```

   `$(pwd)`는 세션이 시작된 작업 폴더다. 사용자가 대화 도중 다른 폴더를 언급해도 기준은 바뀌지 않는다.

2. 저장 경로 확인 — `AskUserQuestion`으로 물어본다.
   - 기본 옵션 라벨: `<BASE>/learning-notes/<TODAY>-<슬러그>.html`
   - 대안: `다른 경로 지정` (Other로 자유 입력)
   - 슬러그 규칙: 한글·비ASCII 문자는 그대로 유지, 공백은 하이픈으로 치환, 파일시스템 예약 문자(`/ \ : * ? " < > |`)만 제거.
   - 사용자가 상대 경로를 입력하면 `<BASE>` 기준으로 절대 경로화한다.

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

1. 새 탭·패널 추가 — "HTML 조작 사양 › 새 개념 추가" 참조. 사이드바에 이미 카드가 있는 개념이면 카드는 손대지 않는다. 패널이 생기는 순간 그 카드는 자동으로 취소선 처리된다.
2. 터미널에 요약 설명 — 한 화면 분량의 이해 가능한 설명. 필요한 경우 예시·비유 포함.
3. 같은 내용을 패널에 HTML로 삽입 — "HTML 조작 사양 › 탭 내용 채우기" 참조.
4. 후속 개념 1~3개 결정 — 하위 개념 또는 사고 유도 질문.
5. 후속 개념을 사이드바에 카드로 추가 — "HTML 조작 사양 › 후속 개념 등록" 참조.

여기서 사이클이 끝난다. 사용자가 다음 개념을 지목하면 그때 이 사이클을 다시 돈다.

### Step 3: 특수 케이스

- 사이드바 밖 주제를 사용자가 물으면: 신규 개념으로 취급해 탭·패널을 만들고, 사이드바에도 카드를 추가한다. 상위 개념이 분명하면 그 슬러그를 `data-parent`에 적고, 없으면 비워 둔다(개념 지도에서 별도 뿌리로 선다).
- "C로 돌아가고 싶어" 같은 요청: 사용자가 브라우저에서 탭이나 카드를 클릭하면 된다. 원한다면 터미널에서도 해당 개념부터 다시 설명하며 그 탭에 내용을 추가해도 됨.

## 화면 구성 (템플릿이 알아서 처리하는 것)

- 다크 테마 고정. 색은 `:root` CSS 변수로만 정의돼 있으니 색을 바꿀 일이 있으면 변수만 고친다.
- 데스크톱: 좌측 본문 + 우측 320px 사이드바. 화면 높이를 꽉 채우고 본문·사이드바가 각자 안에서 스크롤한다.
- 900px 이하 모바일: 사이드바가 하단 시트로 바뀌고, 우하단 FAB을 눌러야 열린다. 스크림 탭·핸들 탭·Esc로 닫힌다.
- 사이드바 카드는 `data-parent`(상위 개념 슬러그)로 묶여, 같은 상위 개념끼리 그룹 제목 아래 모인다. 제목 줄(`.group-title`)은 로드 시 JS가 만들어 붙이므로 HTML에 직접 쓰지 않는다. 그룹 순서와 그룹 안 순서는 카드가 등장한 순서를 그대로 따른다.
- 설명문(패널)이 있는 개념의 카드는 노란색 + 취소선으로 자동 전환된다. 판정 기준은 같은 `data-id`를 가진 `.panel`의 존재 여부 하나뿐이라, 탭·패널을 만들면 그걸로 끝이다. 카드에 클래스를 손으로 붙이지 않는다.
- 노란 카드는 클릭하면 그 설명문으로 이동한다. 중립색 카드(아직 설명문 없음)는 클릭해도 아무 일도 일어나지 않는다.
- 헤더 우상단 지도 버튼을 누르면 탭·본문·사이드바 자리에 개념 지도가 뜬다. 카드의 상·하위 관계를 트리로 그린 화면이고, 색 규칙은 사이드바와 같다. 닫기 버튼이나 Esc로 빠져나온다. 지도는 열 때마다 현재 카드 목록으로 다시 그려지므로 별도 관리가 필요 없다.
- FAB의 개수 배지와 빈 상태 문구는 로드 시 카드 개수를 세어 자동 갱신된다. 카드가 0개면 FAB 자체가 숨는다.

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
        <div class="explanation">
          <!-- 여기에 개념 설명 HTML 삽입 -->
        </div>
      </article>
<!-- /ITEM:transformer:PANEL -->
```

breadcrumb 라벨은 "노트 언어" 절의 문구를 쓴다. 루트 주제의 경우 breadcrumb 줄은 생략하거나 `상위 개념: (루트)`로 표기. 화살표(`↳`) 기호는 쓰지 않는다.

### 탭 내용 채우기

`<div class="explanation">` 안에 개념 설명을 HTML로 삽입한다. 사용 가능한 마크업:

- `<p>`, `<ul>`, `<ol>`, `<strong>`, `<em>`
- `<code>`, `<pre><code>`
- `<blockquote>`

주의: 한 개념당 스크롤 1~2 화면 분량. 지나치게 길어지면 후속 개념으로 분리.

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

개념을 실제로 다뤄 탭·패널이 생기면 그 카드는 노란색 + 취소선으로 바뀐다. 카드는 지우지도, 옮기지도, 고치지도 않는다. 순서도 그대로다.

이 전환은 로드 시 JS가 `data-id`와 같은 `.panel`이 있는지 보고 판정한다. 즉 편집할 것이 없다. 카드에 `covered` 클래스를 손으로 붙이지 말 것 — 붙여도 다시 계산되므로 의미가 없고, 패널 없이 붙이면 지워진다.

## AskUserQuestion 사용 패턴

`AskUserQuestion`은 세션 시작 시 저장 경로를 물을 때만 쓴다. 설명을 마친 뒤 다음 개념을 고르게 하는 용도로는 쓰지 않는다.

### 세션 시작 시 저장 경로

```json
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
}
```

라벨에는 Step 1-1에서 얻은 실제 `BASE`·`TODAY` 값을 채워 넣는다. 라벨이 길어지면 `~` 축약형으로 보여줘도 되지만, 실제 파일 조작은 절대 경로로 한다.

## 예시 흐름

```
User: /study-on-canvas 트랜스포머

LLM: [Bash로 BASE·TODAY 확보]
     [AskUserQuestion 저장 경로 → 사용자 엔터]
     [파일 생성: <BASE>/learning-notes/2026-07-21-트랜스포머.html]
     [Edit로 트랜스포머 탭·패널 삽입]

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
```

이 시점에서 화면은:

- 탭: 트랜스포머, Self-Attention
- 사이드바: `트랜스포머` 제목 아래 Self-Attention(노란색·취소선)·Positional Encoding·Multi-Head Attention, `Self-Attention` 제목 아래 Query·Key·Value 계산·Scaled Dot-Product
- 개념 지도: 트랜스포머 아래 세 갈래, 그중 Self-Attention 아래 다시 두 갈래

Positional Encoding·Multi-Head Attention은 Self-Attention 대화로 빠지는 동안에도 사이드바에 남아있어, 사용자가 언제든 되돌아가 다룰 수 있다.

## 주의

- 사용자가 파일 위치를 수동으로 옮기거나 이름을 바꿔도 스킬 동작에는 영향 없음 (Edit는 현재 세션의 파일 경로만 쓴다). 세션 재시작 시에는 새 `/study-on-canvas`로 새 파일을 만들면 된다.
- 노트가 git 저장소 안에 생기면 커밋에 딸려 들어갈 수 있다. 원치 않으면 `learning-notes/`를 `.gitignore`에 넣도록 안내한다.
- HTML 편집 실패(마커 오탐, indentation 불일치 등)가 발생하면 그 편집만 되돌리고 다시 시도. 파일 전체 재작성은 마지막 수단.
- 같은 세션 안에서 동일한 슬러그(id)가 중복되지 않도록 주의. 중복 위험 시 뒤에 `-2`, `-3` 등 접미사 부여.
