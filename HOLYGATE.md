# 성문교회 주보 영문 번역 → index.html 갱신 → GitHub push

Claude Code에서 이 저장소(`dashpapa/holygate-bulletin`)의 루트에서 실행한다.
이 문서를 `CLAUDE.md`에 넣거나, "HOLYGATE.md를 읽고 그대로 실행해"라고 지시하면 된다.

## 목표

성문교회(Holy Gate Church) 최신 주보(표지~5면)를 영어로 번역해서, 저장소의 `index.html`(모바일 웹 주보 페이지)을 새 호수 내용으로 갱신하고 `main`에 push 한다.

디자인·구조는 그대로 두고 **내용만** 바꾼다.

## 실행 순서

### 1. 최신 주보 확인
- https://holygatechurch.org/184 를 열어 게시판 맨 위 글(제목 형식: `N월 N일 주보`)의 링크를 찾는다.
- 글 본문은 텍스트가 아니라 **이미지 8장**이다. 주소는 `https://cdn.imweb.me/upload/S202505022995f7c9f1d17/<해시>.jpg` 형태이고, 페이지 HTML 안에 `<img src>`로 들어 있다.
- 현재 `index.html`에 반영된 호수(`No. 29xx`)와 날짜를 확인한다(아래 "index.html 구조" 참고).
  새 글의 호수·날짜가 이미 반영돼 있으면 아무것도 바꾸지 말고 "이미 최신"이라고 알리고 끝낸다.

### 2. 이미지 저장
- 이미지를 작업 폴더(예: `./work/YYMMDD/`)에 `curl -O`로 내려받는다. 이 폴더는 `.gitignore`에 넣거나 저장소 밖에 둔다(push 금지).
- 각 이미지를 열어서 내용과 쪽번호 배지로 페이지를 판별한다. 게시된 순서가 뒤집혀 있을 수 있다.
  - 표지: `제N호`, 교회창립 1963년 4월 7일, 설교 제목·성구
  - 1면 예배안내: 주일예배표, 교독문, 다음주 기도순서, 봉사위원
  - 2면 영혼의 양식: 설교 칼럼
  - 3·4면 교회소식
  - 5면: 특별집회 안내(특별새벽부흥회, 토요비전새벽예배 등. 주마다 다름)
- **표지~5면만** 사용한다. 6면 이후(약도, 교역자 안내 등)는 제외한다.

### 3. 번역 규칙
자연스러운 영어로 번역한다. 직역하지 않는다. 공지는 격식체, 설교는 따뜻한 어조.

| 한국어 | 영어 |
|---|---|
| 성문교회 | Holy Gate Church |
| 황정식 목사 (원로목사) | Rev. Hwang Jeong-sik, Pastor Emeritus |
| 고동훈 목사 (담임목사) | Rev. Ko Dong-hoon, Senior Pastor |
| 류요섭 목사 | Rev. Ryu Yo-seop |
| 시온 찬양대 | Zion Choir |
| 할렐루야 찬양대 | Hallelujah Choir |
| 마리아 찬양대 | Maria Choir |
| 홀리하트 찬양팀 | Holy Heart Praise Team |
| 대한예수교장로회 | Presbyterian Church of Korea (Hapdong) |
| 장로 / 집사 / 안수집사 / 권사 | Elder / Deacon / Ord. Deacon / Elder (Women's) |

- 사람 이름: 국어 로마자 표기법(예: 김철수 → Kim Cheol-su). 봉사위원·식당 조원 명단도 모두 로마자로 적는다.
- 날짜: `Sun, Oct 25`, `Lord's Day, Oct 25, 2026`. 호수: `제2912호` → `No. 2912`.
- 금액: `₩10,000`.
- 성구: 반드시 **NIV 본문**으로 바꿔 쓴다(한글 성경 문장을 번역하지 말고 NIV 원문을 쓴다). 형식: `1 Samuel 17:45`. 사무엘상 → 1 Samuel 등 책 이름도 영어로.
- 교독문(인도자/회중): NIV 본문으로.
- 설교 칼럼(2면): **전문 번역**. 요약하지 않는다.
- 전화번호는 그대로.
- 이미지에서 글자가 흐릿해 판독이 불확실하면 추측하지 말고, 마지막 보고에 해당 위치를 적는다.

번역 결과는 `holygate_bulletin_YYMMDD_en.txt`로 작업 폴더에 저장해 둔다(push 불필요).

### 4. index.html 갱신

#### 파일 구조(중요)
`index.html`은 번들러가 만든 파일이다. 보이는 페이지는 이 안에 JSON 문자열로 들어 있다.

- `<script type="__bundler/template">…</script>` : 페이지 전체 HTML이 **JSON 문자열**로 들어 있다.
- `<script type="__bundler/manifest">` : 로고(png), 폰트(woff2), 스크립트(js, 압축)가 UUID로 들어 있다. **건드리지 않는다.**
- 그 밖의 script/style(번들러 로더)도 건드리지 않는다.
- 템플릿 안의 `<img src="UUID">` 등은 manifest의 UUID를 가리킨다. UUID는 그대로 둔다.

#### 읽기/쓰기 방법
템플릿 JSON은 `json.dumps(tpl, ensure_ascii=False).replace('</', '<\\u002F')`로 인코딩돼 있다. 같은 방식으로 다시 써야 한다.

```python
import re, json
src = 'index.html'
s = open(src, encoding='utf-8').read()
m = re.search(r'(<script type="__bundler/template">)(.*?)(</script>)', s, re.S)
raw = m.group(2)
tpl = json.loads(raw)                       # 페이지 HTML 문자열

# 안전 확인: 편집 전에 왕복 인코딩이 원본과 같은지 확인
enc = json.dumps(tpl, ensure_ascii=False).replace('</', '<\\u002F')
assert enc == raw.strip(), '인코딩 방식이 원본과 다름'

# ... tpl 문자열을 편집 ...

enc = json.dumps(tpl, ensure_ascii=False).replace('</', '<\\u002F')
assert json.loads(enc) == tpl
new = s[:m.start(2)] + '\n' + enc + '\n  ' + s[m.end(2):]
open(src, 'w', encoding='utf-8').write(new)
```

실행은 `python3 -I`로 한다. 편집 전 `tpl`을 별도 파일(`template.html`)로 덤프해 보면 구조 파악이 쉽다.

#### 템플릿의 섹션 구조(위에서 아래로)
디자인(인라인 스타일)과 이 구조를 그대로 유지하고 **내용 텍스트만** 새 호수로 교체한다.

1. 상단 고정 헤더: 로고, `Holy Gate Church`, 호수(`No. 29xx`)
2. 목차 메뉴(햄버거): 앵커 id(`toc-cover`, `toc-worship`, `toc-reading`, `toc-prayerteam`, `toc-sermon`, `toc-news`, `toc-member`, `toc-festival`, `toc-giving`)와 라벨
3. 표지 줄: `2026 Vision … (1 Sam 16:7)`, `Founded April 7, 1963`, `No. 29xx · Lord's Day, <날짜>`
4. 표지 카드: 이번 주 설교 제목, 성구
5. 예배 안내(Page 1): 사회자, 주일예배표(1·2·3부 행들), 설교 행, 통역·새가족 안내
6. 교독문(`showReading` 조건 블록): 번호, 시편, 인도자/회중 줄들
7. 다음 주 기도 순서, 주일 중보기도(주일 날짜 2개), 주일오후·수요저녁·새벽·금요 예배 카드
8. 이번 달 봉사위원(안내위원·헌금위원, 1·2부)
9. 영혼의 양식(Page 2, `showReflection` 조건 블록): 칼럼 제목, 성구, 설교자, 본문 전체
10. 교회소식(Page 3–4): 새가족 환영, 번호 항목들(1., 2., …), 식당 운영 표
11. 교우 소식(부고·출산·결혼) 또는 Upcoming Events(행사 예고). 그 주 주보에 있는 쪽만 둔다
12. 5면(`showFestival` 조건 블록 등): 특별새벽부흥회 일정, 토요비전새벽예배 일정 등. 주마다 내용이 다르므로 그 주 구성에 맞게 카드를 늘리거나 줄인다
13. 헌금 계좌, Visit Us, Core Values, 푸터(`No. 29xx · Lord's Day, <날짜>`)

규칙:
- 헌금 계좌·Core Values·Visit Us는 주보에 변경이 없으면 그대로 둔다.
- 이번 주보에 없는 섹션은 제거하고, 목차 메뉴의 링크·라벨도 같이 맞춘다. 목차 앵커 id와 본문 id가 서로 맞아야 한다.
- 지난 호의 날짜·호수·행사가 남아 있으면 안 된다(예: `No. 2911`, `Oct 18`).
- 기존 인라인 스타일을 복사해 새 행·카드를 만든다. 새 CSS를 만들지 않는다.

#### 검증
저장 후 playwright(chromium)로 `file://`로 열어 확인한다.

- 콘솔 오류가 없다.
- 이전 호수·날짜 문자열이 남아 있지 않다.
- 한글(U+AC00–D7A3)이 한 글자도 남아 있지 않다.
- 폭 440px 모바일 스크린샷에서 레이아웃이 깨지지 않았다(가로 스크롤, 겹침 없음).

### 5. GitHub push
- 변경은 `index.html` 한 파일만.
- 커밋 메시지: `Update bulletin No. 29xx (Mon D, 2026)`
- `git fetch origin main` 후 필요하면 rebase 하고 `main`에 push.

### 6. 마지막 보고
호수·날짜, 커밋 해시, 판독이 불확실했던 부분, 이번 주 구성이 지난주와 달라서 조정한 섹션을 짧게 적는다.

## 이전 작업에서 정해 둔 표기(기존 사이트와 일치시킬 것)
- 직함: `Pastor Emeritus` / `Senior Pastor`, 권사는 `Elder (Women's)`, 안수집사는 `Ord. Deacon`.
- 이름 표기는 지난 호 코드의 표기를 그대로 따른다(같은 사람은 같은 철자).
- 교회 비전 문구: `2026 Vision: A Faith That Captures God's Heart! (1 Sam 16:7)`.
- 요일·달 약어: `Mon, Oct 26`, `Sat, Nov 7`.

## 선택: 매주 자동 실행
Claude Code가 컴퓨터에서 도는 경우에는 이미지 서버(`cdn.imweb.me`)에 제한이 없다.
토요일 오전에 실행하려면 예약 작업(스케줄)으로 이 문서의 1~6단계를 그대로 지시하면 된다. 이번 주보는 보통 금요일 오후에 게시되므로 토요일 오전이 안전하다.
