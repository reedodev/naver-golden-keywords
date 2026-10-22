<p align="center"><img src="docs/banner.png" alt="네이버 황금키워드: 찾는 사람은 많고 글은 적은 키워드" width="100%"></p>

<p align="center">
<a href="https://github.com/reedodev/naver-golden-keywords/actions/workflows/test.yml"><img src="https://github.com/reedodev/naver-golden-keywords/actions/workflows/test.yml/badge.svg" alt="test"></a>
<img src="https://img.shields.io/badge/Claude_Code-skill-2547F4" alt="Claude Code skill">
<img src="https://img.shields.io/badge/license-MIT-black" alt="MIT">
</p>

# 네이버 황금키워드 (naver-golden-keywords)

네이버 검색광고 API의 월간 검색량과 검색 API의 블로그 문서수로 경쟁률을 계산해 S·A·B 등급 황금키워드를 고르고, 연관 키워드까지 함께 채점합니다. 키 값은 화면에 출력하지 않습니다.

## 처음 쓰시는 분을 위한 안내

### 이 스킬이 해결하는 문제

블로그 글감을 고를 때 검색하는 사람이 많은 키워드를 고르기 쉽지만, 이미 글이 수십만 개 쌓여 있으면 새 글이 눈에 띄기 어렵습니다. 이 스킬은 월간 검색량과 블로그 글 수를 비교해 "찾는 사람은 많은데 글은 적은" 키워드를 등급(S·A·B)으로 알려 줍니다.

- 상황 1: 오늘 쓸 글의 키워드 후보 5개 중 어떤 것이 경쟁이 덜한지 비교하고 싶을 때
- 상황 2: 큰 키워드 하나에서 연관 키워드를 찾아 더 좁은 주제를 발견하고 싶을 때

### 용어 풀이

- **터미널**: 명령을 글자로 입력하는 창입니다. Windows는 PowerShell 또는 Windows Terminal, Mac은 터미널 앱입니다.
- **Claude Code**: 터미널에서 쓰는 AI 코딩 에이전트(사용자 대신 파일을 읽고 명령을 실행해 주는 AI)입니다.
- **스킬**: 에이전트가 읽고 그대로 따르는 사용 설명서(SKILL.md)와 실행 파일을 묶은 폴더입니다.
- **API 키**: 네이버가 "이 사람이 맞다"고 확인해 주는 비밀 번호 같은 값입니다. 남에게 보이거나 공개 저장소에 올리면 안 됩니다.
- **경쟁률**: 블로그 문서수를 월간 검색량으로 나눈 값입니다. 작을수록 글이 적은 키워드입니다.

### 준비 확인

**1. Node.js 18 이상**을 확인합니다.

```bash
node -v
```

`v18.0.0` 이상의 숫자가 나오면 됩니다. 명령을 찾을 수 없다고 나오면 [nodejs.org](https://nodejs.org)에서 LTS 버전을 설치하거나, Windows는 `winget install OpenJS.NodeJS.LTS`(설치 후 터미널을 다시 엽니다), Mac은 `brew install node`를 실행합니다.

**2. 네이버 API 키 5개**가 필요합니다. 둘 다 무료로 발급받을 수 있고, 검색광고는 광고비를 쓰지 않아도 발급됩니다.

| 어디서 | 받는 값 | 환경 변수 이름 |
|---|---|---|
| [searchad.naver.com](https://searchad.naver.com) 로그인 → 도구 → API 사용 관리 | API 키, 비밀키, 고객 ID | `NAVER_AD_API_KEY`, `NAVER_AD_SECRET`, `NAVER_AD_CUSTOMER_ID` |
| [developers.naver.com](https://developers.naver.com) 애플리케이션 등록 → 검색 API 선택 | Client ID, Client Secret | `NAVER_CLIENT_ID`, `NAVER_CLIENT_SECRET` |

스크립트를 실행하는 폴더에 `.env` 파일을 만들고 아래처럼 `=` 뒤에 값을 붙여 넣습니다. 이 저장소의 `.gitignore`가 `.env`를 제외하므로 실수로 올라가지 않지만, 다른 곳에 복사하실 때는 주의하시기 바랍니다.

```
NAVER_AD_API_KEY=
NAVER_AD_SECRET=
NAVER_AD_CUSTOMER_ID=
NAVER_CLIENT_ID=
NAVER_CLIENT_SECRET=
```

예시 파일 `skills/naver-golden-keywords/.env.example`을 복사해 시작하셔도 됩니다. 스크립트는 키 값을 화면에 출력하지 않습니다.

### 따라 하기 3단계

**1단계. 설치합니다.** Claude Code 안에서 아래 두 줄을 입력합니다. Claude Code 없이 스크립트만 쓰려면 아래 "바로 해 보기"로 바로 가셔도 됩니다.

```
/plugin marketplace add reedodev/naver-golden-keywords
/plugin install naver-golden-keywords@naver-golden-keywords
```

**2단계. 에이전트에게 이렇게 말합니다.** 아래 문장을 복사해서 쓰시면 됩니다. `캠핑의자`는 조사하려는 키워드로 바꿉니다.

1. `naver-golden-keywords 스킬로 "캠핑의자"의 월간 검색량과 블로그 문서수를 조사하고, 연관 키워드 20개까지 같이 채점해 줘.`
2. `S, A, B 등급이 하나도 없으면 등급 기준을 낮춰서(--rules) 다시 보여 줘.`
3. `결과를 result.csv로 저장하고, 경쟁률이 낮은 순으로 상위 3개를 표로 정리해 줘.`

**3단계. 결과를 확인합니다.** 표에서 `등급` 칸에 S·A·B가 있는 키워드가 글감 후보입니다. `-`는 기준에 못 미친다는 뜻입니다.

### 실제 실행 결과 예시

2026년 10월 22일에 실제 API 키로 실행한 결과입니다. 숫자는 조회 시점마다 달라집니다.

```bash
node skills/naver-golden-keywords/scripts/golden-keywords.mjs 캠핑의자 --related 5 --csv result.csv
```

```
등급  월검색량   문서수      경쟁률   키워드
-        65700     263745     4.014   에어텐트 (연관)
-        83400    1481925    17.769   캠핑의자
-        48300    1688635    34.961   캠핑카 (연관)
-        52000    2069566    39.799   캠핑테이블 (연관)
-        79500   26211065   329.699   의자 (연관)
-        45480   15938109   350.442   캠핑 (연관)

0/6 개가 등급 안에 들었습니다(기준: S 2000+ / 0.1, A 1000+ / 0.5, B 500+ / 2). 검색량은 최근 30일 기준, 문서수는 조회 시점 기준입니다.
```

읽는 법은 다음과 같습니다.

- 위쪽에 있는 키워드일수록 경쟁률이 낮습니다. `(연관)`은 입력한 키워드가 아니라 네이버가 연관 키워드로 알려 준 것입니다.
- 이 예시는 검색량이 큰 대신 글도 매우 많아서(경쟁률이 4 이상) 6개 모두 등급이 없습니다. 오류가 아니라 "이 키워드들은 새 글이 어렵다"는 결과입니다. 더 구체적인 키워드(예: 계절, 용도를 붙인 말)로 다시 조사해 보시기 바랍니다.
- `result.csv`는 같은 내용을 엑셀에서 열 수 있게 저장한 파일입니다.

### 막힐 때

| 증상 | 원인 | 해결 |
|---|---|---|
| `missing credentials: NAVER_AD_API_KEY, ...` (종료 코드 2) | API 키가 설정되지 않았습니다 | 위 「준비 확인」의 `.env` 파일을 실행하는 폴더에 만들고 5개 값을 채웁니다 |
| `error: searchad HTTP 403 (check the Search Ad API key, ...)` (종료 코드 1) | 검색광고 API 키·비밀키·고객 ID 중 틀린 값이 있습니다 | 검색광고 화면의 값을 다시 복사합니다. 공백이나 줄바꿈이 섞이지 않았는지 봅니다 |
| `usage: node golden-keywords.mjs <keyword...> ...` (종료 코드 2) | 조사할 키워드를 적지 않았습니다 | 명령 뒤에 키워드를 적습니다. 띄어쓰기가 있으면 따옴표로 묶습니다 |
| 표의 등급이 전부 `-`이고 `0/6 개가 등급 안에 들었습니다` | 오류가 아니라 기준(S·A·B)에 맞는 키워드가 없다는 결과입니다 | 더 구체적인 키워드로 바꾸거나 `--rules`로 기준을 낮춥니다 |

<!-- prereq-adapt -->
## 무엇이 필요한가요?

꼭 필요한 것은 Node 18 이상, 네이버 검색광고 계정과 API 키 3개, 네이버 개발자센터 앱(검색 API) 키 2개입니다. 권장·선택 준비물까지 정리한 표는 [SKILL.md의 준비물](skills/naver-golden-keywords/SKILL.md#준비물-쓰기-전에-있어야-하는-것)에 있습니다.

## 바꿔 쓰기

에이전트에게 다음처럼 말하면 옵션을 바꿔서 실행합니다.

- "검색량이 적어도 경쟁 없는 키워드를 보여 줘" → `--rules S:300:0.2,A:200:1,B:100:3` 처럼 등급 기준을 낮춥니다
- "등급 이름을 골드·실버로 해 줘" → `--rules 골드:30000:3,실버:10000:10`
- "연관 키워드도 같이 30개 봐 줘" → `--related 30`

말과 옵션의 대응표는 [SKILL.md의 바꿔 쓰기](skills/naver-golden-keywords/SKILL.md#바꿔-쓰기-이렇게-말하세요)에 있습니다.

## 설치

**Claude Code 플러그인으로 설치**

```
/plugin marketplace add reedodev/naver-golden-keywords
/plugin install naver-golden-keywords@naver-golden-keywords
```

**폴더만 복사**

```bash
git clone https://github.com/reedodev/naver-golden-keywords.git
cp -r naver-golden-keywords/skills/naver-golden-keywords ~/.claude/skills/
```

필요한 것: Node 18+, 네이버 검색광고·검색 API 키

## 바로 해 보기

```bash
node skills/naver-golden-keywords/scripts/golden-keywords.mjs 캠핑의자 --related 20 --csv result.csv
```

`.env` 파일에 API 키 5개를 넣어 둔 폴더에서 실행해야 합니다(위 「준비 확인」 참고). 키가 없으면 `missing credentials`가 나오고 종료 코드 2로 끝납니다. 키 없이 스크립트 자체만 확인하려면 `node skills/naver-golden-keywords/scripts/golden-keywords.mjs --self-test`를 실행하며, `SELF-TEST OK (9 cases)`가 나오면 정상입니다.


자세한 절차, 옵션, 한계는 [SKILL.md](skills/naver-golden-keywords/SKILL.md)에 있습니다. 에이전트가 이 파일을 읽고 그대로 따릅니다.

## 시험

`node tests/run.mjs`가 SKILL.md 형식을 검사하고 예시 파일로 스크립트를 실제로 실행합니다. 푸시할 때마다 GitHub Actions가 같은 시험을 돌립니다(위 test 배지).

<!-- credits -->
## 출처와 영감

- **데이터 출처**: 네이버 검색광고 API의 키워드 도구([공식 문서](https://naver.github.io/searchad-apidoc/))와 네이버 개발자센터 검색 API의 블로그 검색([공식 문서](https://developers.naver.com/docs/serviceapi/search/blog/blog.md))만 씁니다. 각 API의 이용 약관과 호출 한도를 따릅니다.
- **"황금키워드"라는 생각**: 검색량에 비해 글이 적은 키워드를 고르는 방식은 국내 블로거들 사이에서 널리 쓰이는 일반적인 방법입니다. S·A·B 등급 기준값은 작성자가 정한 것이고 `--rules`로 바꿀 수 있습니다.

## 만든 사람

**리도**, AI 좌충우돌 실습 중인 20년차 엔지니어입니다. 매일 AI에게 일을 맡기다가 막힌 부분을 고쳐 가며 이 스킬을 만들었습니다.
[YouTube @Reedodev](https://www.youtube.com/@Reedodev) · [Threads @reedo.dev](https://www.threads.com/@reedo.dev) · [GitHub @reedodev](https://github.com/reedodev)

같이 만든 다른 스킬: [설명 영상 만들기](https://github.com/reedodev/explainer-studio) · [옵시디언 3D 지식그래프](https://github.com/reedodev/obsidian-brain-3d) · [노트 먼저 찾아보기](https://github.com/reedodev/vault-recall) · [작업 기록 강의자료](https://github.com/reedodev/work-to-lecture-deck) · [한국어 말투 점검](https://github.com/reedodev/korean-tone-check) · [한글 양식 채우기](https://github.com/reedodev/hwpx-form-fill) · [TTS 끝소리 고치기](https://github.com/reedodev/tts-tail-fix) · [카드뉴스 렌더](https://github.com/reedodev/html-card-render) · [영상 비율 검사](https://github.com/reedodev/social-video-gate) · [유튜브 게시 점검](https://github.com/reedodev/youtube-publish-checklist)

---

## English

Score Naver blog keywords by monthly searches vs. blog document count using the official Search Ad and Search APIs, with related-keyword discovery and S/A/B grades.

### New here? A beginner's guide

**What it solves.** When you pick a blog keyword, it is easy to choose one many people search for, but if hundreds of thousands of posts already exist, a new post is hard to notice. This skill compares monthly searches with the number of blog posts and grades keywords S, A or B: "many searchers, few posts".

- Case 1: you have five keyword ideas for today's post and want to know which one is least crowded.
- Case 2: you want related keywords of a broad keyword to find a narrower topic.

**Glossary.**
- **Terminal**: a window where you type commands (PowerShell or Windows Terminal on Windows, Terminal on Mac).
- **Claude Code**: an AI coding agent that runs in the terminal and reads files and runs commands for you.
- **Skill**: a folder with instructions (SKILL.md) and scripts that the agent reads and follows.
- **API key**: a secret value that proves to Naver who you are. Never show it or commit it to a public repository.
- **Competition ratio**: blog document count divided by monthly searches. Smaller means fewer posts per searcher.

**Check your setup.**

1. Node.js 18 or newer:

   ```bash
   node -v
   ```

   If it prints `v18.0.0` or higher, you are set. Otherwise install the LTS version from [nodejs.org](https://nodejs.org), or run `winget install OpenJS.NodeJS.LTS` (Windows, then reopen the terminal) or `brew install node` (Mac).
2. Five Naver API values. Both are free, and the Search Ad API key can be issued without spending on ads.

   | Where | You get | Environment variable |
   |---|---|---|
   | [searchad.naver.com](https://searchad.naver.com): Tools → API management | API key, secret key, customer ID | `NAVER_AD_API_KEY`, `NAVER_AD_SECRET`, `NAVER_AD_CUSTOMER_ID` |
   | [developers.naver.com](https://developers.naver.com): register an application, choose the Search API | Client ID, Client Secret | `NAVER_CLIENT_ID`, `NAVER_CLIENT_SECRET` |

   Put them in a `.env` file in the folder where you run the script (one `NAME=value` per line). You can copy `skills/naver-golden-keywords/.env.example` as a template. `.env` is listed in this repository's `.gitignore`, and the script never prints key values.

**Three steps.**

1. Install it inside Claude Code (or skip this and run the script directly, see "바로 해 보기" above):

   ```
   /plugin marketplace add reedodev/naver-golden-keywords
   /plugin install naver-golden-keywords@naver-golden-keywords
   ```

2. Tell the agent, copying these sentences and replacing the keyword:
   - `Use the naver-golden-keywords skill to check monthly searches and blog document count for "캠핑의자" and grade up to 20 related keywords too.`
   - `If no keyword reaches S, A or B, lower the grade thresholds (--rules) and show me again.`
   - `Save the result to result.csv and list the top 3 lowest-competition keywords in a table.`
3. Read the result. Keywords with S, A or B in the grade column are topic candidates. `-` means it did not reach any grade.

**Real sample output.** Run on 2026-10-22 with real API keys. Numbers change at every lookup.

```bash
node skills/naver-golden-keywords/scripts/golden-keywords.mjs 캠핑의자 --related 5 --csv result.csv
```

```
등급  월검색량   문서수      경쟁률   키워드
-        65700     263745     4.014   에어텐트 (연관)
-        83400    1481925    17.769   캠핑의자
-        48300    1688635    34.961   캠핑카 (연관)
-        52000    2069566    39.799   캠핑테이블 (연관)
-        79500   26211065   329.699   의자 (연관)
-        45480   15938109   350.442   캠핑 (연관)

0/6 개가 등급 안에 들었습니다(기준: S 2000+ / 0.1, A 1000+ / 0.5, B 500+ / 2). 검색량은 최근 30일 기준, 문서수는 조회 시점 기준입니다.
```

How to read it:
- Columns are grade, monthly searches, blog documents, competition ratio and keyword. Rows with the lowest ratio come first. `(연관)` marks a related keyword suggested by Naver rather than one you typed.
- Here every keyword has a huge number of posts (ratio above 4), so none gets a grade. That is a result, not an error: these broad keywords are hard for a new post. Try a more specific keyword.
- `result.csv` holds the same data for a spreadsheet.

**Troubleshooting.**

| Symptom | Cause | Fix |
|---|---|---|
| `missing credentials: NAVER_AD_API_KEY, ...` (exit 2) | API keys are not set | Create the `.env` file described above in the folder you run from and fill in all five values |
| `error: searchad HTTP 403 (check the Search Ad API key, ...)` (exit 1) | One of the Search Ad API key, secret or customer ID is wrong | Copy the values again from the Search Ad screen and check for stray spaces or line breaks |
| `usage: node golden-keywords.mjs <keyword...> ...` (exit 2) | No keyword was given | Add a keyword after the command; wrap keywords that contain spaces in quotes |
| All grades are `-` and the last line says `0/6 개가 등급 안에 들었습니다` | Not an error: no keyword meets the S/A/B thresholds | Use a more specific keyword or lower the thresholds with `--rules` |

**Credits.**
- Data comes only from the official Naver Search Ad API ([docs](https://naver.github.io/searchad-apidoc/)) and Naver Search API for blogs ([docs](https://developers.naver.com/docs/serviceapi/search/blog/blog.md)). The "golden keyword" idea is common practice among Korean bloggers; the grade thresholds are the author's own and configurable.

Install: `/plugin marketplace add reedodev/naver-golden-keywords` then `/plugin install naver-golden-keywords@naver-golden-keywords`, or copy `skills/naver-golden-keywords` into `~/.claude/skills/`. See [SKILL.md](skills/naver-golden-keywords/SKILL.md) for the full procedure. MIT License; third-party models, fonts and libraries keep their own licenses.
