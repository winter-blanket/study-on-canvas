# study-on-canvas

LLM과 대화하며 새 개념을 배울 때, 대화 내용을 실시간 HTML 학습 노트로 남기는 Claude Code 플러그인.

개념을 하나 설명할 때마다 후속 개념을 1~3개씩 사이드바에 쌓아 둔다. 하나를 파고들다 옆길로 새더라도 아직 다루지 않은 형제 개념이 화면에 그대로 남아 있어, "그때 뭘 더 보려고 했더라"가 사라진다.

## 무엇을 하는가

- `/study-on-canvas <주제>`로 세션을 시작하면 다크 테마 HTML 노트 한 장이 생기고 브라우저에서 열린다.
- 개념을 설명할 때마다 탭 하나와 설명 패널 하나가 노트에 추가된다.
- 동시에 후속 개념 1~3개가 사이드바 카드로 쌓인다. 4개 이상은 만들지 않는다 — 한 번에 압도되지 않게.
- 이미 다룬 개념의 카드는 지워지지 않고 노란색 + 취소선으로만 바뀐다. 자리가 그대로 남아 지도가 된다.
- 헤더의 지도 버튼을 누르면 카드들의 상·하위 관계가 트리(개념 지도)로 그려진다.
- 설명이 끝나면 거기서 멈춘다. "다음은 무엇을 볼까요"라고 묻지 않는다. 무엇을 다룰지는 사용자가 사이드바를 보고 정한다.

노트는 의존성 없는 단일 HTML 파일이라 그대로 열고, 옮기고, 공유할 수 있다. 모바일 폭에서는 사이드바가 하단 시트로 바뀐다.

## 설치

Claude Code에서:

```
/plugin marketplace add winter-blanket/study-on-canvas
/plugin install study-on-canvas@study-on-canvas
```

## 사용법

```
/study-on-canvas:study-on-canvas 트랜스포머
```

저장 위치를 한 번 물어본다. 기본값은 세션을 시작한 폴더 아래 `learning-notes/YYYY-MM-DD-<주제>.html`이다. 이후에는 그냥 대화하면 된다.

```
> 트랜스포머 알려줘        → 탭 생성 + 카드 3개 (Self-Attention / Positional Encoding / Multi-Head Attention)
> self-attention 알려줘    → 탭 추가, 첫 카드는 노란색 + 취소선으로 전환, 새 카드 2개
```

노트가 갱신되면 브라우저에서 새로고침하면 된다.

노트를 git 저장소 안에 만들면 커밋에 딸려 들어갈 수 있다. 원치 않으면 `learning-notes/`를 `.gitignore`에 넣으면 된다.

## 언어

노트 UI는 한국어가 기본이고, 영어도 지원한다. 스킬을 부른 문장의 언어를 보고 자동으로 정한다 — 영어로 물으면 영어 노트가 나온다. 세션 도중에는 바뀌지 않는다.

## 요구사항

- Claude Code. `disable-model-invocation`·`user-invocable` 같은 스킬 프론트매터와 `AskUserQuestion` 도구를 쓰기 때문에 다른 하네스에서는 동작하지 않는다.
- macOS·Linux·Windows(Git Bash/MSYS) 모두에서 브라우저 자동 열기를 시도한다. 실패하면 파일 경로만 안내하고, 스킬 동작 자체에는 영향이 없다.

## 구조

```
.claude-plugin/
  plugin.json          플러그인 매니페스트
  marketplace.json     단일 플러그인 마켓플레이스 정의
skills/study-on-canvas/
  SKILL.md             스킬 본문 — 실행 플로우와 HTML 조작 사양
  template/
    study-note.html    노트 템플릿 (단일 파일, 외부 의존성 없음)
```

템플릿은 `{{TITLE}}`·`{{DATE}}` 같은 플레이스홀더와 `<!-- TABS_INSERT -->` 류의 마커만 갖고 있다. 스킬은 파일 전체를 다시 쓰지 않고 이 마커 사이에만 삽입한다.

## 커스터마이즈

- 색: `study-note.html`의 `:root` CSS 변수만 고치면 된다. 다크 테마 고정이다.
- UI 문구: SKILL.md의 "노트 언어" 표에 언어별 사전이 있다. 언어를 추가하려면 이 표에 열을 하나 더 만들면 된다.
- 후속 개념 개수: SKILL.md 핵심 원칙 1번.

## 라이선스

MIT
