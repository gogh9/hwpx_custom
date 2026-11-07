---
name: hwpx-form-preserve
description: 한글 문서(.hwpx)를 서식 그대로 살려 작성·편집. 사용자가 준 서식(계획서·학습지·평가지·공문·가정통신문 등)의 그림·글자 모양·문단 모양·줄간격·내어쓰기·표·수식을 유지한 채 내용 채우기, 문항·항목 추가, 한 단어 넘김 자간 조정.
---

# HWPX 서식 보존 작성·편집

한글(.hwpx) 문서를 **서식 파일 그대로** 살려서 만들고 고치는 스킬이다. 먼저 서식 파일의 구조를 파악한다. 그다음 그림, 글자 모양, 문단 모양, 줄 간격, 내어쓰기, 표, 수식, 글상자, 다단, 바탕쪽을 모두 원본 그대로 둔 채 작업한다. 바꾸는 것은 세 가지뿐이다.

- 글자와 수식 내용
- 기존 부분을 복제해 늘리기
- 줄 맞춤을 위한 자간 조정

Word(.docx)는 docx 스킬을 쓴다.

## 어떤 서식을 쓰나

1. 사용자가 .hwpx를 올렸으면 그 파일을 쓴다.
2. 올린 파일이 없으면 사용자의 **기본 서식**을 찾는다.
   - 위치: Google Drive 커넥터가 연결돼 있으면 `search_files`로 `title contains '기본서식'`을 검색해 문서 종류에 맞는 파일을 찾는다(예: `계획서_기본서식.hwpx`, `가정통신문_기본서식.hwpx`).
   - 받기: `download_file_content`로 받은 base64를 디코딩해 `/home/claude/`에 저장한다.
   - 찾은 파일 이름은 사용자에게 한 줄로 알린다.
3. 그래도 없으면 서식 파일을 올려 달라고 요청한다.
   - 다음부터 자동으로 쓰려면 드라이브에 "○○_기본서식.hwpx" 이름으로 넣어 두면 된다고 안내한다.
   - `HwpxDocument.new()`로 빈 문서를 만들지 않는다. 서식 없는 한글 문서는 모양이 깨지기 쉽다.
4. .hwp(구형)는 처리하지 못하므로, 한글에서 .hwpx로 다시 저장해 올려 달라고 안내한다.

## 원칙

1. 서식 파일 위에서 작업한다. `HwpxDocument.open()/save()`나 정규식으로 XML 문자열을 직접 고치는 방식은 쓰지 않는다. 아래 도구만 쓴다.
2. **새 모양을 만들지 않는다.** 글자는 그 자리에 이미 있는 글자 모양에 넣는다. 문단, 문항, 표 행, 대단원이 더 필요하면 서식에 있는 것을 **복제**한다. 예외는 자간 조정용으로 기존 모양과 자간만 다른 모양을 추가할 때뿐이다(`fit_lines`가 자동 처리).
3. 그림, 바탕쪽, 배경 채우기, 기존 모양 정의는 건드리지 않는다.
4. **한글이 이미 배치해 둔 문단(손대지 않은 원본 문단)과 사용자가 손으로 조정한 자간은 바꾸지 않는다.**
5. 수식은 원본 표기를 따른다(`÷`/`DIVIDE`, `rm`, `{a} over {b}`, 대분수 `1 {4} over {5}`).
6. 원본은 복사해서 작업하고, 결과는 새 이름으로 `/mnt/user-data/outputs/`에 저장한다.
7. 할 수 없는 작업은 억지로 하지 않고 사용자에게 알린다. 예: 세로 병합이 있는 표의 행 추가, 그림 교체, 머리말·꼬리말·누름틀 편집.

## 줄 맞춤 규칙

1. **한 단어만 다음 줄로 넘어가면** 그 줄 본문의 자간을 줄여 한 줄로 만든다.
   - 번호나 기호 머리("1. ", "• ")는 자간 0 그대로 두고 본문 run만 줄인다.
   - 두 단어 이상이 넘어가는 경우는 그대로 둔다(사용자가 원하면 그때 맞춘다).
   - `fit_all(d)`가 새로 쓴 문단만 골라 자동으로 처리한다. 예측 폭이 98~101%인 "경계" 줄도 조금 줄여 여유를 둔다.
   - 자간 -12로도 안 되면 자간을 더 줄이지 말고 문장을 줄인다. 내가 쓴 문장이면 뜻을 유지한 채 직접 줄인다. 사용자가 준 문장이면 줄인 안을 제안한다. 그런 뒤 `fit_all`을 다시 돌린다.
2. **내어쓰기 + Shift+Enter 목록**: 서식에 "￭ 제목 ↵ • 항목 ↵ • 항목", "1. 제목 ↵ 가. … ↵ 나. …" 같은 문단이 있으면 같은 방식으로 쓴다.
   - `set_lines(d, p, ["￭ 제목", "• 항목1", "• 항목2"])`를 쓴다.
   - 둘째 줄부터의 들여쓰기는 문단 모양의 내어쓰기가 맡는다. 공백으로 흉내 내지 않는다.
3. **긴 문장이 다음 줄로 넘어가면** 내어쓰기 문단 모양이 둘째 줄을 들여 준다. 손으로 줄을 바꾸거나 공백을 넣지 않는다.

줄 나눔 예측은 글자 폭 모델로 계산한다. 한글 1em, 공백 0.42em, 영문은 표 값을 쓴다. 함초롬바탕·함초롬돋움 문서의 실제 줄 나눔으로 보정했고, 시험한 문서 100여 문단에서 줄 수가 모두 일치했다. 다른 글꼴(HY헤드라인M, 휴먼명조 등)은 오차가 더 클 수 있다. 저장된 원본으로 정확도를 확인하는 방법은 작업 순서 2단계에 있다. 최종 확인은 한글에서 해야 하므로 결과를 전할 때 한 줄로 알린다.

## 작업 순서

1. 아래 도구 코드를 `/home/claude/hwpx_tools.py`로 저장한다(`lxml` 필요, 없으면 `pip install lxml --break-system-packages`). 서식 파일을 작업 폴더로 복사한다.
2. **서식 파악**
   - `python3 /home/claude/hwpx_tools.py 파일.hwpx`로 구조를 출력한다.
   - 출력 내용: 용지·여백·다단, 최상위 문단 `[n]`, 문단 모양 `p번호`, 글자 모양 `<c번호>`, Shift+Enter `↵`, 표 칸 주소·병합·`★빈칸`, 수식·글상자·그림 id, 사용된 글자·문단 모양 요약(글꼴, 크기, 색, 자간, 장평, 정렬, 줄간격, 여백, 내어쓰기, 줄나눔).
   - `Preview/PrvImage.png`를 unzip해서 Read로 보면 첫 쪽 배치를 확인할 수 있다.
   - 파악한 결과로 정리한다: 어떤 문단이 어떤 역할인지(제목, 대단원 막대, 번호 항목, 표, 문항 등), 문항이나 항목이 어느 문단 범위인지, 채울 빈칸, 바꿀 수식, 손으로 자간을 조정한 곳. 작업이 크면 이 정리를 사용자에게 한두 줄로 확인받는다.
   - 줄 나눔 예측 정확도 확인: 원본에서 각 문단의 `predict` 결과와 저장된 `hp:linesegarray/hp:lineseg` 개수를 비교한다.
   - 원본에 이미 한 단어 넘김이 있는지는 `orphans_in_saved(d)`로 찾는다.
3. **편집**

| 작업 | 함수 |
|---|---|
| 문단 글자 새로 쓰기(Shift+Enter 줄 목록, 머리/본문 run 분리) | `set_lines(d, 문단, ["1. …"])` |
| 글자 일부 바꾸기(글자 모양 유지, 필드·개체 있는 문단도 가능) | `replace_text(d, 옛, 새, count=None, within=문단목록)` |
| 수식 바꾸기 | `set_equation(d, 수식id 또는 요소, "스크립트")` |
| run별 글자(문항 번호 글상자 등) | `set_run_texts(문단, ["0", "6"])` |
| 표 빈칸 채우기 | `fill_cell(find_obj(d, 표id), 행, 열, "글자")` |
| 표 칸 문단 얻기 | `cell(표, 행, 열).findall("hp:subList/hp:p", NS)` |
| 최상위 문단 목록 | `top_paragraphs(d)` (구조 출력의 `[n]` 번호와 같음) |
| 문항·항목·대단원 복제 | `clone_paragraphs(d, 처음, 끝, after=위치)` (새 문단 목록 반환) |
| 표 행 추가 | `clone_row(표, 행, d)` (세로 병합 있으면 불가) |
| **한 단어 넘김 자동 조정** | `fit_all(d)` 또는 `fit_lines(d, 문단)` |
| 줄 나눔 예측 보기 | `predict(d, 문단)` → `[(줄, [(글자, 폭비율)])]` |
| 단어 단위 줄나눔으로 바꾸기 | `p.set("paraPrIDRef", str(clone_parapr(d, p.get("paraPrIDRef"))))` |
| 저장 | `d.save(결과경로)` |

   - 하이퍼링크 같은 필드나 표·수식이 든 문단은 `set_lines`를 쓸 수 없다. `replace_text`로 고친다.
   - 문항 번호가 글상자 안에 있으면 `new[0].find(".//hp:rect/hp:drawText/hp:subList/hp:p", NS)`로 찾는다.
   - 줄간격·여백 같은 문단 모양은 바꾸지 않는다. 사용자가 요청하면 `clone_parapr`로 복제본을 만들어 바꾼다. 이때 `hp:switch` 안의 `hp:case`와 `hp:default` 값을 **둘 다** 고친다.
4. **줄 맞춤**: 내용을 모두 쓴 뒤 `fit_all(d)`를 실행한다. "문장 다듬기 필요"가 나온 줄은 위 규칙대로 처리하고 `fit_all`을 다시 실행한다.
5. **전후 비교**: `verify(원본, 결과)`가 "통과"인지 확인하고, 바뀐 글자 목록이 의도와 같은지 본다. verify는 다음을 점검한다.
   - 기존 모양 정의·그림·바탕쪽 보존(자간 조정용 새 모양만 추가 허용)
   - 없는 모양을 참조하는 곳
   - 개체 id 중복
   - mimetype 위치
6. **전달**: 결과 파일을 보내고 한 줄로 알린다.
   - 자간을 조정한 줄과 줄인 문장
   - 한글에서 확인할 점: 줄 나눔·쪽 넘김·수식 크기·늘어난 표 높이는 한글이 열 때 다시 계산한다. 이 환경에서는 HWPX를 그려 볼 수 없다.
   - 글꼴은 문서에 들어 있지 않으므로 열람 PC에 있어야 한다.

## 예시 - 계획서 채우기

```python
import sys; sys.path.insert(0, "/home/claude")
from hwpx_tools import *
src, out = "/home/claude/계획서_기본서식.hwpx", "/mnt/user-data/outputs/OO_운영계획.hwpx"
d = Hwpx(src); ps = top_paragraphs(d)
replace_text(d, "서식의 기존 제목", "새 제목")                    # 제목 칸(컨트롤 포함 가능)
for p, t in zip(ps[5:8], ["1. …", "2. …", "3. …"]):          # 번호 항목 문단(구조 출력에서 확인한 번호)
    set_lines(d, p, [t])
tbl = find_obj(d, 일정표_id)                                   # 구조 출력의 [표 … id=…]
set_lines(d, cell(tbl, 1, 3).find("hp:subList/hp:p", NS), ["￭ 1차시 제목", "• 활동 1", "• 활동 2"])
fit_all(d)
d.save(out); verify(src, out)
```

## 예시 - 평가지(수식·문항 복제)

```python
d = Hwpx(src)
set_equation(d, 수식_id, "2.8÷0.4")
new = clone_paragraphs(d, 23, 26, after=26)                   # 문항 하나(번호 문단~다음 번호 직전) 복제
set_run_texts(new[0].find(".//hp:rect/hp:drawText/hp:subList/hp:p", NS), ["0", "6"])
replace_text(d, "주스", "우유", within=new)                    # 복제한 문항 안에서만
fill_cell(find_obj(d, 답칸_표_id), 0, 1, "2000")
d.save(out); verify(src, out)
```

## 공문서 작성 기준 (서식 안 글자를 쓸 때)

- 항목 기호는 `1. → 가. → 1) → 가) → (1) → (가) → ① → ㉮` 순서다. 항목이 하나뿐이면 기호를 붙이지 않는다.
- 날짜는 `2026. 2. 13.(금)`처럼 쓴다. 월·일 앞의 0은 생략한다.
- 시간은 24시각제로 `15:20`처럼 쓴다.
- 본문 끝은 2타 띄우고 `끝.`이다. "붙임" 뒤에는 쌍점을 붙이지 않는다.

## 도구 코드 (`/home/claude/hwpx_tools.py`로 저장)

`Hwpx.save()`는 원래 ZIP 순서와 압축 방식을 그대로 쓰고, lxml이 원래 접두어를 유지한다. 그래서 네임스페이스 후처리가 필요 없다.

```python
"""서식 보존형 HWPX 도구 (lxml 사용, 원본 접두어·속성 순서 유지)."""
import copy, os, re, zipfile
from lxml import etree

NS = {
    "hp": "http://www.hancom.co.kr/hwpml/2011/paragraph",
    "hh": "http://www.hancom.co.kr/hwpml/2011/head",
    "hc": "http://www.hancom.co.kr/hwpml/2011/core",
    "hs": "http://www.hancom.co.kr/hwpml/2011/section",
}
HP = "{%s}" % NS["hp"]


def _ln(e):
    return etree.QName(e).localname if isinstance(e.tag, str) else ""


# ---------------------------------------------------------------- 열기/저장
class Hwpx:
    """ZIP 전체를 메모리에 올려 두고, 고친 XML 파트만 다시 쓴다."""

    def __init__(self, path):
        self.path = path
        with zipfile.ZipFile(path) as z:
            self.infos = z.infolist()
            self.raw = {i.filename: z.read(i.filename) for i in self.infos}
        self._trees = {}

    def tree(self, part):
        if part not in self._trees:
            self._trees[part] = etree.fromstring(self.raw[part])
        return self._trees[part]

    @property
    def header(self):
        return self.tree("Contents/header.xml")

    def sections(self):
        names = sorted(n for n in self.raw if re.match(r"Contents/section\d+\.xml$", n))
        return [(n, self.tree(n)) for n in names]

    def save(self, dst):
        for part, root in self._trees.items():
            self.raw[part] = etree.tostring(root, xml_declaration=True,
                                            encoding="UTF-8", standalone=True)
        tmp = dst + ".tmp"
        with zipfile.ZipFile(tmp, "w") as z:
            for info in self.infos:            # 원래 순서·압축 방식 그대로 (mimetype 무압축 유지)
                z.writestr(info, self.raw[info.filename])
        os.replace(tmp, dst)


# ---------------------------------------------------------------- 서식 정의 요약
def _font_names(h):
    out = {}
    for ff in h.iterfind(".//hh:fontface", NS):
        lang = ff.get("lang")
        for f in ff.iterfind("hh:font", NS):
            out[(lang, f.get("id"))] = f.get("face")
    return out


def charpr_summary(h, cid):
    fonts = _font_names(h)
    c = h.find(f".//hh:charPr[@id='{cid}']", NS)
    if c is None:
        return f"charPr {cid}: 없음"
    fr = c.find("hh:fontRef", NS)
    face = fonts.get(("HANGUL", fr.get("hangul"))) if fr is not None else "?"
    sp = c.find("hh:spacing", NS)
    ra = c.find("hh:ratio", NS)
    flags = [n for n in ("bold", "italic", "underline", "strikeout", "outline", "shadow",
                         "emboss", "engrave", "supscript", "subscript")
             if c.find(f"hh:{n}", NS) is not None and
             (n != "underline" or c.find("hh:underline", NS).get("type") != "NONE") and
             (n != "strikeout" or c.find("hh:strikeout", NS).get("shape") != "NONE") and
             (n != "outline" or c.find("hh:outline", NS).get("type") != "NONE") and
             (n != "shadow" or c.find("hh:shadow", NS).get("type") != "NONE")]
    return (f"charPr {cid}: {face} {int(c.get('height'))/100:g}pt 색{c.get('textColor')}"
            f" 자간{sp.get('hangul') if sp is not None else '?'}"
            f" 장평{ra.get('hangul') if ra is not None else '?'}"
            + (f" {'/'.join(flags)}" if flags else ""))


def parapr_summary(h, pid):
    p = h.find(f".//hh:paraPr[@id='{pid}']", NS)
    if p is None:
        return f"paraPr {pid}: 없음"
    al = p.find("hh:align", NS)
    # lineSpacing / margin 은 hp:switch 안에 있을 수 있음 -> 첫 번째 것 사용
    ls = p.find(".//hh:lineSpacing", NS)
    m = p.find(".//hh:margin", NS)
    def mv(tag):
        e = m.find(f"hc:{tag}", NS) if m is not None else None
        return e.get("value") if e is not None else "0"
    br = p.find("hh:breakSetting", NS)
    return (f"paraPr {pid}: 정렬{al.get('horizontal') if al is not None else '?'}"
            f" 줄간격{ls.get('type') if ls is not None else '?'}:{ls.get('value') if ls is not None else '?'}"
            f" 왼{mv('left')} 오{mv('right')} 들여{mv('intent')}"
            f" 위{mv('prev')} 아래{mv('next')}"
            f" 한글줄나눔{br.get('breakNonLatinWord') if br is not None else '?'}")


# ---------------------------------------------------------------- 구조 조사
def _obj_desc(e):
    n = _ln(e)
    if n == "tbl":
        return f"[표 {e.get('rowCnt')}x{e.get('colCnt')} id={e.get('id')}]"
    if n == "equation":
        s = e.find("hp:script", NS)
        return f"[수식 '{s.text if s is not None else ''}' id={e.get('id')}]"
    if n == "pic":
        img = e.find(".//hc:img", NS)
        sz = e.find("hp:sz", NS)
        return (f"[그림 {img.get('binaryItemIDRef') if img is not None else '?'}"
                f" {sz.get('width')}x{sz.get('height')} id={e.get('id')}]")
    if n in ("rect", "ellipse", "polygon", "curve", "arc", "line", "container"):
        txt = "".join(e.xpath(".//hp:drawText//hp:t//text()", namespaces=NS)).strip()
        return f"[{ {'rect':'글상자','line':'선','container':'묶음'}.get(n, n)} id={e.get('id')}" + (f" '{txt}'" if txt else "") + "]"
    return None


def _para_line(p, depth):
    parts = []
    for run in p.iterfind("hp:run", NS):
        seg = []
        for ch in run:
            n = _ln(ch)
            if n == "t":
                t = ch.text or ""
                for sub in ch:
                    t += {"lineBreak": "↵", "tab": "⇥"}.get(_ln(sub), "") + (sub.tail or "")
                seg.append(t)
            elif n == "ctrl":
                seg.append("[ctrl]")
            else:
                d = _obj_desc(ch)
                if d:
                    seg.append(d)
        parts.append(f"<c{run.get('charPrIDRef')}>{''.join(seg)}")
    flag = " 쪽나눔" if p.get("pageBreak") == "1" else ""
    flag += " 단나눔" if p.get("columnBreak") == "1" else ""
    return f"{'  '*depth}¶ p{p.get('paraPrIDRef')} s{p.get('styleIDRef')}{flag}: " + " ".join(parts)


def dump_structure(path, show_styles=True, max_depth=6):
    """서식 파일의 구조를 사람이 읽을 수 있게 출력한다.
    용지·여백·단, 바탕쪽, 그림 목록, 문단(모양 번호)·표(칸 주소)·수식·글상자, 사용된 모양 정의."""
    d = Hwpx(path)
    h = d.header
    used_c, used_p = set(), set()
    for name, sec in d.sections():
        print(f"===== {name}")
        pp = sec.find(".//hp:pagePr", NS)
        if pp is not None:
            mg = pp.find("hp:margin", NS)
            print(f"용지 {int(pp.get('width'))/283.46:.0f}x{int(pp.get('height'))/283.46:.0f}mm "
                  f"방향{pp.get('landscape')} | 여백(mm) " +
                  " ".join(f"{k}{int(mg.get(k))/283.46:.1f}" for k in
                           ("top", "bottom", "left", "right", "header", "footer")))
        for cp in sec.iter(HP + "colPr"):
            if cp.get("colCount", "1") != "1":
                print(f"다단: {cp.get('colCount')}단 간격{cp.get('sameGap')} type={cp.get('type')}")
        print("--- 본문")

        def walk(container, depth):
            for idx, p in enumerate(container.iterfind("hp:p", NS)):
                used_p.add(p.get("paraPrIDRef"))
                for r in p.iterfind("hp:run", NS):
                    used_c.add(r.get("charPrIDRef"))
                line = _para_line(p, depth)
                print(f"[{idx}] {line}" if depth == 0 else line)
                if depth >= max_depth:
                    continue
                for obj in p.iterfind("hp:run/*", NS):
                    n = _ln(obj)
                    if n == "tbl":
                        for tc in obj.iterfind("hp:tr/hp:tc", NS):
                            a = tc.find("hp:cellAddr", NS)
                            sp = tc.find("hp:cellSpan", NS)
                            span = (f" 병합{sp.get('rowSpan')}x{sp.get('colSpan')}"
                                    if sp is not None and (sp.get('rowSpan'), sp.get('colSpan')) != ("1", "1") else "")
                            txt = "".join(tc.xpath(".//hp:t//text()", namespaces=NS)).strip() or \
                                any(_ln(x) in OBJ_TAGS for x in tc.iter())
                            print(f"{'  '*(depth+1)}칸({a.get('rowAddr')},{a.get('colAddr')}){span}"
                                  f" bf{tc.get('borderFillIDRef')}" + ("" if txt else " ★빈칸"))
                            walk(tc.find("hp:subList", NS), depth + 2)
                    elif n in ("rect", "ellipse", "polygon", "container"):
                        for sl in obj.iterfind("hp:drawText/hp:subList", NS):
                            walk(sl, depth + 1)
        walk(sec, 0)

    for mp in sorted(n for n in d.raw if n.startswith("Contents/masterpage")):
        root = d.tree(mp)
        objs = [_obj_desc(e) for e in root.iter() if _obj_desc(e)]
        txt = "".join(root.xpath(".//hp:t//text()", namespaces=NS)).strip()
        print(f"--- 바탕쪽 {mp}: {objs} {('글자: ' + txt) if txt else ''}")
    print("--- 그림 파일")
    hpf = d.raw.get("Contents/content.hpf", b"").decode("utf-8")
    for m in re.finditer(r'<opf:item id="([^"]+)" href="(BinData/[^"]+)"', hpf):
        print(f"{m.group(1)} -> {m.group(2)} ({len(d.raw.get(m.group(2), b''))} bytes)")
    if show_styles:
        print("--- 사용된 글자 모양")
        for c in sorted(used_c, key=int):
            print(" ", charpr_summary(h, c))
        print("--- 사용된 문단 모양")
        for p in sorted(used_p, key=int):
            print(" ", parapr_summary(h, p))


# ================================================================ 편집 (서식 그대로)
OBJ_TAGS = ("tbl", "equation", "rect", "pic", "line", "container", "ellipse",
            "polygon", "arc", "curve", "ole", "textart")


def find_obj(doc, obj_id):
    """dump_structure에 나온 id로 표·수식·글상자·그림을 찾는다."""
    for _, sec in doc.sections():
        for e in sec.iter():
            if _ln(e) in OBJ_TAGS and e.get("id") == str(obj_id):
                return e
    raise KeyError(f"id {obj_id} 없음")


def _para_of(e):
    while e is not None and _ln(e) != "p":
        e = e.getparent()
    return e


def drop_lineseg(p):
    """글자가 바뀐 문단의 저장된 줄 배치 정보를 지워 한글이 다시 배치하게 한다."""
    for ls in p.findall("hp:linesegarray", NS):
        p.remove(ls)


def set_run_texts(p, texts):
    """문단 p의 run들에 글자를 순서대로 넣는다 (각 run의 글자 모양 유지).
    texts 길이는 hp:t를 가진 run 수와 같아야 한다. 예: 문항 번호 글상자 ['0','6']"""
    runs = [r for r in p.iterfind("hp:run", NS) if r.find("hp:t", NS) is not None]
    if len(runs) != len(texts):
        raise ValueError(f"run {len(runs)}개 / 글자 {len(texts)}개 불일치")
    for r, s in zip(runs, texts):
        ts = r.findall("hp:t", NS)
        ts[0].text = s
        for extra in ts[1:]:
            if extra.find("*") is None:
                r.remove(extra)
    drop_lineseg(p)


def cell(tbl, row, col):
    for tc in tbl.iterfind("hp:tr/hp:tc", NS):
        a = tc.find("hp:cellAddr", NS)
        if a.get("rowAddr") == str(row) and a.get("colAddr") == str(col):
            return tc
    raise KeyError(f"칸({row},{col}) 없음")


def fill_cell(tbl, row, col, text, para_index=0):
    """빈칸에 글자를 넣는다. 그 칸 문단에 이미 있는 글자 모양(첫 run)을 그대로 쓴다."""
    p = cell(tbl, row, col).findall("hp:subList/hp:p", NS)[para_index]
    run = p.find("hp:run", NS)
    t = run.find("hp:t", NS)
    if t is not None and not (t.text or "").strip() and t.find("*") is None:
        t.text = text
    else:
        t = etree.SubElement(run, HP + "t")
        t.text = text
    drop_lineseg(p)


def replace_text(doc, old, new, count=None, within=None):
    """본문·표·글상자 안의 글자(hp:t)에서만 바꾼다. 글자 모양·개체는 그대로.
    within: 문단(또는 문단 목록)을 주면 그 안에서만 바꾼다 (복제한 문항만 고칠 때).
    한 hp:t 안에 old가 통째로 있을 때만 바뀐다 (run을 걸치면 dump로 확인 후 set_run_texts 사용)."""
    n = 0
    if within is None:
        roots = [sec for _, sec in doc.sections()]
    else:
        roots = within if isinstance(within, list) else [within]
    for root in roots:
        for t in root.iter(HP + "t"):
            for holder, attr in [(t, "text")] + [(c, "tail") for c in t]:
                s = getattr(holder, attr)
                if s and old in s and (count is None or n < count):
                    k = s.count(old) if count is None else min(s.count(old), count - n)
                    setattr(holder, attr, s.replace(old, new, k))
                    n += k
                    drop_lineseg(_para_of(t))
    return n


def set_equation(doc, eq_id, script):
    """수식 내용 바꾸기. eq_id는 id 숫자 또는 수식 요소 자체.
    수식 문법은 한글 수식 편집기 문법(over, TIMES, DIVIDE, rm 등) - 원본 수식의 표기를 그대로 따른다."""
    eq = eq_id if not isinstance(eq_id, (str, int)) else find_obj(doc, eq_id)
    eq.find("hp:script", NS).text = script
    drop_lineseg(_para_of(eq))


def _renumber(doc, elems):
    """복제한 부분(요소 목록)의 개체 id/instid를 문서에서 겹치지 않는 새 번호로 한 번에 매긴다."""
    if not isinstance(elems, list):
        elems = [elems]
    used = set()
    for _, sec in doc.sections():
        for e in sec.iter():
            for k in ("id", "instid"):
                v = e.get(k) if isinstance(e.tag, str) else None
                if v and v.isdigit():
                    used.add(int(v))
    nxt = max([v for v in used if v < 2147483648] or [1000]) + 1
    for elem in elems:
        for e in elem.iter():
            if _ln(e) in OBJ_TAGS:
                e.set("id", str(nxt)); nxt += 1
                if e.get("instid"):
                    e.set("instid", str(nxt)); nxt += 1


def top_paragraphs(doc, section=0):
    return doc.sections()[section][1].findall("hp:p", NS)


def clone_paragraphs(doc, first, last, after, section=0):
    """최상위 문단 first..last(번호, dump 순서)를 서식째 복제해 after 문단 뒤에 넣는다.
    문항 하나를 통째로 늘릴 때 쓴다. 새 문단 목록을 반환한다."""
    ps = top_paragraphs(doc, section)
    if any(p.find("hp:run/hp:secPr", NS) is not None or p.find("hp:run/hp:ctrl", NS) is not None
           for p in ps[first:last + 1]):
        raise ValueError("구역·단 설정(ctrl/secPr)이 든 문단은 복제하지 않는다 - 범위를 다시 잡는다")
    anchor = ps[after]
    new = []
    for p in ps[first:last + 1]:
        c = copy.deepcopy(p)
        c.set("pageBreak", "0"); c.set("columnBreak", "0")
        new.append(c)
    _renumber(doc, new)
    for c in reversed(new):
        anchor.addnext(c)
    return new


def clone_row(tbl, src_row, doc):
    """표의 src_row 행을 서식째 복제해 바로 아래에 넣는다 (행 걸침 병합이 없는 표만)."""
    rows = tbl.findall("hp:tr", NS)
    for tc in tbl.iterfind("hp:tr/hp:tc", NS):
        if tc.find("hp:cellSpan", NS).get("rowSpan") != "1":
            raise ValueError("세로 병합이 있는 표는 행 복제 불가 - 사용자에게 알린다")
    newtr = copy.deepcopy(rows[src_row])
    _renumber(doc, newtr)
    rows[src_row].addnext(newtr)
    for i, tr in enumerate(tbl.findall("hp:tr", NS)):
        for tc in tr.iterfind("hp:tc", NS):
            tc.find("hp:cellAddr", NS).set("rowAddr", str(i))
    tbl.set("rowCnt", str(int(tbl.get("rowCnt")) + 1))
    sz = tbl.find("hp:sz", NS)
    h = sum(int(tr.find("hp:tc/hp:cellSz", NS).get("height")) for tr in tbl.findall("hp:tr", NS))
    sz.set("height", str(max(int(sz.get("height")), h)))
    return newtr


# ================================================================ 자간·줄나눔 (모양 복제본 추가)
def _clone_def(doc, tag, list_tag, base_id):
    h = doc.header
    base = h.find(f".//hh:{tag}[@id='{base_id}']", NS)
    if base is None:
        raise KeyError(f"{tag} {base_id} 없음")
    parent = h.find(f".//hh:{list_tag}", NS)
    new_id = max(int(e.get("id")) for e in parent.iterfind(f"hh:{tag}", NS)) + 1
    c = copy.deepcopy(base)
    c.set("id", str(new_id))
    parent.append(c)
    parent.set("itemCnt", str(len(parent.findall(f"hh:{tag}", NS))))
    return c, new_id


def clone_charpr(doc, base_id, spacing=None, ratio=None):
    """글자 모양 base_id를 복제해 자간/장평만 바꾼 새 모양을 추가하고 새 id를 반환.
    기존 모양은 건드리지 않는다. 적용은 run.set('charPrIDRef', str(new_id))."""
    c, nid = _clone_def(doc, "charPr", "charProperties", base_id)
    keys = ("hangul", "latin", "hanja", "japanese", "other", "symbol", "user")
    for tag, v in (("spacing", spacing), ("ratio", ratio)):
        if v is not None:
            e = c.find(f"hh:{tag}", NS)
            for k in keys:
                e.set(k, str(v))
    return nid


def clone_parapr(doc, base_id, keep_word=True):
    """문단 모양 base_id를 복제해 한글 줄나눔을 단어 단위(KEEP_WORD)로 바꾼 새 모양을 추가.
    적용은 p.set('paraPrIDRef', str(new_id))."""
    c, nid = _clone_def(doc, "paraPr", "paraProperties", base_id)
    if keep_word:
        c.find("hh:breakSetting", NS).set("breakNonLatinWord", "KEEP_WORD")
    return nid


# ================================================================ 전후 비교
def verify(before_path, after_path):
    """서식 보존 점검: 기존 모양 정의·그림·바탕쪽이 그대로인지, 바뀐 글자는 무엇인지 출력.
    header.xml은 '기존 정의가 그대로이고 새 정의만 뒤에 붙은 경우'만 허용한다."""
    a, b = Hwpx(before_path), Hwpx(after_path)
    ok = True
    for part in sorted(set(a.raw) | set(b.raw)):
        if part.startswith("Contents/section") or part.startswith("Preview/"):
            continue
        if part == "Contents/header.xml" and a.raw.get(part) != b.raw.get(part):
            for tag in ("charPr", "paraPr", "borderFill", "font", "style", "tabPr"):
                old = {e.get("id"): etree.tostring(e) for e in a.header.iter(f"{{{NS['hh']}}}{tag}")}
                new = {e.get("id"): etree.tostring(e) for e in b.header.iter(f"{{{NS['hh']}}}{tag}")}
                changed = [i for i in old if old[i] != new.get(i)]
                if changed:
                    print(f"!! 기존 {tag} 바뀜: {changed}"); ok = False
                added = [i for i in new if i not in old]
                if added:
                    print(f"   새 {tag} 추가: {added}")
            continue
        if a.raw.get(part) != b.raw.get(part):
            print(f"!! 바뀜: {part}"); ok = False
    with zipfile.ZipFile(after_path) as z:
        first = z.infolist()[0]
        if first.filename != "mimetype" or first.compress_type != 0:
            print("!! mimetype 위치/압축 이상"); ok = False
        for n in z.namelist():
            if n.endswith(".xml"):
                etree.fromstring(z.read(n))
    hc = {c.get("id") for c in b.header.iterfind(".//hh:charPr", NS)}
    hp_ = {p.get("id") for p in b.header.iterfind(".//hh:paraPr", NS)}
    for _, sec in b.sections():
        for r in sec.iter(HP + "run"):
            if r.get("charPrIDRef") not in hc:
                print("!! 없는 글자 모양 참조", r.get("charPrIDRef")); ok = False
        for p in sec.iter(HP + "p"):
            if p.get("paraPrIDRef") not in hp_:
                print("!! 없는 문단 모양 참조", p.get("paraPrIDRef")); ok = False
        ids = [e.get("id") for e in sec.iter() if _ln(e) in OBJ_TAGS]
        before_dup = len([1 for _, s in a.sections() for e in s.iter() if _ln(e) in OBJ_TAGS]) - \
                     len({e.get("id") for _, s in a.sections() for e in s.iter() if _ln(e) in OBJ_TAGS})
        if len(ids) - len(set(ids)) > before_dup:
            print("!! 개체 id 중복 증가"); ok = False

    def texts(d):
        out = []
        for _, sec in d.sections():
            for e in sec.iter():
                if _ln(e) == "t":
                    s = "".join(e.itertext()).strip()
                    if s: out.append(s)
                elif _ln(e) == "script" and e.text:
                    out.append("수식:" + e.text)
        return out
    import difflib
    diff = [l for l in difflib.unified_diff(texts(a), texts(b), lineterm="", n=0)
            if l[:1] in "+-" and l[:3] not in ("+++", "---")]
    print("--- 바뀐 글자/수식")
    print("\n".join(diff) if diff else "(없음)")
    print("서식 보존 점검:", "통과" if ok else "문제 있음")
    return ok


# ================================================================ 줄 나눔 예측·한 단어 넘김 조정
# 글자 폭 모델(em): 한글·전각 1.0, 공백 0.42, 영문·숫자는 아래 표.
# 함초롬바탕 14pt / 함초롬돋움 13pt 문서의 실제 줄 나눔으로 보정함. 오차 약 ±2% (다른 글꼴은 오차가 더 클 수 있음).
_ASCII = dict(zip((chr(c) for c in range(33, 127)), (
    0.29, 0.353, 0.54, 0.517, 0.868, 0.732, 0.184, 0.342, 0.342, 0.449, 0.545, 0.307, 0.325, 0.307, 0.331,
    0.524, 0.443, 0.525, 0.524, 0.522, 0.525, 0.525, 0.516, 0.525, 0.528, 0.307, 0.307, 0.545, 0.545, 0.545,
    0.406, 0.857, 0.675, 0.63, 0.65, 0.724, 0.613, 0.591, 0.697, 0.802, 0.381, 0.379, 0.687, 0.587, 0.916,
    0.748, 0.721, 0.603, 0.721, 0.672, 0.533, 0.619, 0.749, 0.671, 0.99, 0.658, 0.646, 0.569, 0.323, 0.331,
    0.323, 0.545, 0.529, 0.409, 0.524, 0.6, 0.506, 0.591, 0.514, 0.365, 0.532, 0.623, 0.312, 0.292, 0.571,
    0.314, 0.916, 0.621, 0.56, 0.6, 0.571, 0.434, 0.445, 0.345, 0.614, 0.515, 0.788, 0.53, 0.519, 0.468,
    0.352, 0.297, 0.352, 0.545)))
_ASCII.update({" ": 0.42, "·": 0.327, "•": 0.336, "￭": 0.5})
FIT_OK, FIT_OVER = 0.98, 1.01      # 98% 이하: 들어감 / 101% 초과: 넘침 / 사이: 경계(한글에서 확인)
CLOSE_PUNCT = set(")]}」』〉》,.!?:;%’”·、。")
PREFIX_RE = re.compile(r"^\s*((\d+|[가-힣])\.|\(\d+\)|\d+\)|[-•￭■□○◦※▪·])\s*")


def char_em(c):
    if c in _ASCII:
        return _ASCII[c]
    o = ord(c)
    return 1.0 if o >= 0x1100 and not (0x2000 <= o <= 0x206F) else 0.5


def _charpr(doc, cid):
    return doc.header.find(f".//hh:charPr[@id='{cid}']", NS)


def _size_spacing(doc, cid):
    c = _charpr(doc, cid)
    return int(c.get("height")) / 100, int(c.find("hh:spacing", NS).get("hangul"))


def text_width(doc, pieces):
    """pieces: [(글자, charPrID 또는 (크기pt, 자간%))] -> HWPUNIT 폭 예측."""
    w = 0.0
    for text, st in pieces:
        size, sp = st if isinstance(st, tuple) else _size_spacing(doc, st)
        w += sum(size * 100 * (char_em(c) + sp / 100) for c in text)
    return w


def avail_width(doc, p):
    """(첫 줄 폭, 둘째 줄부터 폭) HWPUNIT. 표 칸·본문 폭과 문단 여백·내어쓰기를 반영."""
    par = p.getparent()
    if _ln(par) == "subList" and _ln(par.getparent()) == "tc":
        tc = par.getparent()
        tbl = tc.getparent().getparent()
        src = tc.find("hp:cellMargin", NS) if tc.get("hasMargin") == "1" else tbl.find("hp:inMargin", NS)
        W = int(tc.find("hp:cellSz", NS).get("width")) - int(src.get("left")) - int(src.get("right"))
    else:
        sec = p.getroottree().getroot()
        pp = sec.find(".//hp:pagePr", NS)
        mg = pp.find("hp:margin", NS)
        W = int(pp.get("width")) - int(mg.get("left")) - int(mg.get("right"))
    W -= 3                                         # 한글 반올림 오차
    pr = doc.header.find(f".//hh:paraPr[@id='{p.get('paraPrIDRef')}']", NS)
    m = pr.find(".//hh:margin", NS)
    g = lambda t: int(m.find(f"hc:{t}", NS).get("value"))
    W -= g("left") + g("right")
    ind = g("intent")
    return (W, W + ind) if ind < 0 else (W - ind, W)   # 내어쓰기(음수): 둘째 줄부터 좁아짐


def logical_lines(p):
    """문단을 Shift+Enter(lineBreak) 기준 줄로 나눈다. 각 줄 = [(글자, charPrID)].
    표·수식·필드 같은 개체가 있으면 None (자동 조정 대상 아님)."""
    lines, cur = [], []
    for r in p.iterfind("hp:run", NS):
        cid = r.get("charPrIDRef")
        for ch in r:
            n = _ln(ch)
            if n == "t":
                if ch.text:
                    cur.append((ch.text, cid))
                for sub in ch:
                    if _ln(sub) == "lineBreak":
                        lines.append(cur); cur = []
                    elif _ln(sub) == "tab":
                        cur.append(("    ", cid))
                    if sub.tail:
                        cur.append((sub.tail, cid))
            elif n in OBJ_TAGS or n == "ctrl":
                return None
    lines.append(cur)
    return lines


def wrap(doc, pieces, first_w, rest_w):
    """한글 '단어 단위 줄나눔'을 흉내 내어 물리적 줄 목록 [(글자, 폭비율)]을 예측."""
    chars = [(c, cid) for t, cid in pieces for c in t]
    out, start, width = [], 0, first_w
    while start < len(chars):
        w, last_sp, i = 0.0, None, start
        while i < len(chars):
            c, cid = chars[i]
            w += text_width(doc, [(c, cid)])
            if c == " ":
                last_sp = i
            if w > width and c != " " and c not in CLOSE_PUNCT:   # 금칙: 닫는 부호는 앞줄에 붙임
                break
            i += 1
        if i >= len(chars):
            seg = chars[start:]
        else:
            cut = last_sp + 1 if last_sp is not None and last_sp > start else i
            seg = chars[start:cut]
            i = cut
        txt = "".join(c for c, _ in seg)
        ww = text_width(doc, [(c, cid) for c, cid in seg]) - \
            sum(text_width(doc, [(c, cid)]) for c, cid in seg[len(txt.rstrip()):])
        out.append((txt, ww / width))
        start, width = i, rest_w
    return out or [("", 0.0)]


def predict(doc, p):
    """문단의 줄별 예측. [(줄번호, [(물리줄 글자, 폭비율)])]"""
    ll = logical_lines(p)
    if ll is None:
        return None
    fw, rw = avail_width(doc, p)
    return [(k, wrap(doc, pieces, fw if k == 0 else rw, rw)) for k, pieces in enumerate(ll)]


def _charpr_key(c):
    c = copy.deepcopy(c)
    c.attrib.pop("id", None)
    c.remove(c.find("hh:spacing", NS))
    return etree.tostring(c)


def charpr_with_spacing(doc, base_id, spacing):
    """base_id와 자간만 다른 기존 모양이 있으면 재사용, 없으면 복제해 만든다."""
    base = _charpr(doc, base_id)
    key = _charpr_key(base)
    for c in doc.header.iter(f"{{{NS['hh']}}}charPr"):
        if int(c.find("hh:spacing", NS).get("hangul")) == spacing and _charpr_key(c) == key \
                and len(set(c.find("hh:spacing", NS).attrib.values())) == 1:
            return int(c.get("id"))
    return clone_charpr(doc, base_id, spacing=spacing)


def base_charpr(doc, cid):
    """손으로 자간을 줄인 모양이면 자간 0인 같은 모양을 찾아 돌려준다 (없으면 그대로)."""
    base = _charpr(doc, cid)
    if int(base.find("hh:spacing", NS).get("hangul")) == 0:
        return int(cid)
    key = _charpr_key(base)
    for c in doc.header.iter(f"{{{NS['hh']}}}charPr"):
        if _charpr_key(c) == key and all(v == "0" for v in c.find("hh:spacing", NS).attrib.values()):
            return int(c.get("id"))
    return int(cid)


def set_lines(doc, p, lines, cid=None):
    """문단 글자를 줄 목록으로 새로 쓴다. 줄 사이는 Shift+Enter(lineBreak).
    각 줄은 '머리(1. / 가. / • / ￭ / - 등)' run과 '본문' run으로 나뉘어 줄마다 자간을 따로 조정할 수 있다.
    글자 모양은 cid(없으면 그 문단 첫 run의 기본 모양)를 쓴다. 문단 모양(내어쓰기 등)은 그대로."""
    if logical_lines(p) is None:
        raise ValueError("개체·필드가 있는 문단 - replace_text로 고친다")
    runs = p.findall("hp:run", NS)
    cid = cid or base_charpr(doc, runs[0].get("charPrIDRef"))
    for r in runs:
        p.remove(r)
    anchor = p.find("hp:linesegarray", NS)
    new_runs = []
    for k, line in enumerate(lines):
        m = PREFIX_RE.match(line)
        parts = [line[:m.end()], line[m.end():]] if m and m.end() < len(line) else [line]
        for j, part in enumerate(parts):
            r = etree.Element(HP + "run"); r.set("charPrIDRef", str(cid))
            t = etree.SubElement(r, HP + "t"); t.text = part
            if j == len(parts) - 1 and k < len(lines) - 1:
                etree.SubElement(t, HP + "lineBreak")
            new_runs.append(r)
    for r in new_runs:
        if anchor is not None:
            anchor.addprevious(r)
        else:
            p.append(r)
    drop_lineseg(p)


def fit_lines(doc, p, max_tighten=-12, report=True):
    """줄마다 줄 나눔을 예측해, 마지막 줄에 한 단어만 넘어가거나(또는 경계라 넘어갈 수 있으면)
    그 줄 본문 run의 자간을 -1씩 줄여 한 줄 덜 쓰게 만든다. 머리(1. 등) run은 그대로.
    max_tighten보다 더 줄여야 하면 바꾸지 않고 '문장 다듬기 필요'로 보고한다.
    한글이 이미 배치해 둔 문단(linesegarray 있음 = 손대지 않은 원본)은 건드리지 않는다."""
    if p.find("hp:linesegarray", NS) is not None or logical_lines(p) is None:
        return []
    fw, rw = avail_width(doc, p)
    groups, cur = [], []
    for r in p.iterfind("hp:run", NS):
        cur.append(r)
        if r.find("hp:t/hp:lineBreak", NS) is not None:
            groups.append(cur); cur = []
    groups.append(cur)
    rtext = lambda r: "".join(t.text or "" for t in r.iterfind("hp:t", NS)) + \
        "".join(x.tail or "" for t in r.iterfind("hp:t", NS) for x in t)
    results = []
    for k, runs in enumerate(groups):
        w0 = fw if k == 0 else rw
        body = [r for r in runs if not PREFIX_RE.fullmatch(rtext(r))] or runs
        def pieces(sp=None):
            out = []
            for r in runs:
                size, cur_sp = _size_spacing(doc, r.get("charPrIDRef"))
                out.append((rtext(r), (size, sp if (sp is not None and r in body) else cur_sp)))
            return out
        lines = wrap(doc, pieces(), w0, rw)
        text = "".join(rtext(r) for r in runs)
        orphan = len(lines) >= 2 and " " not in lines[-1][0].strip() and " " in text.strip()
        border = FIT_OK < lines[-1][1] <= FIT_OVER
        if not (orphan or border):
            continue
        target = len(lines) - 1 if orphan else len(lines)
        done = None
        for sp in range(-1, max_tighten - 1, -1):
            tl = wrap(doc, pieces(sp), w0, rw)
            if len(tl) <= target and tl[-1][1] <= FIT_OK:
                done = sp; break
        if done is None:
            results.append((text, len(lines), None, f"자간 {max_tighten}로도 부족 - 문장 다듬기 필요"))
            continue
        for r in body:
            b = base_charpr(doc, r.get("charPrIDRef"))
            r.set("charPrIDRef", str(charpr_with_spacing(doc, b, done)))
        results.append((text, len(lines), done, "한 단어 넘김 → 한 줄로" if orphan else "경계 → 여유 확보"))
    if report:
        for text, n, sp, msg in results:
            print(f"  [{msg}] 자간 {sp} | {n}줄 | {text[:45]}")
    return results


def fit_all(doc, max_tighten=-12):
    """문서 전체 문단에 fit_lines 적용 (개체·필드가 있는 문단은 건너뜀)."""
    out = []
    for _, sec in doc.sections():
        for p in list(sec.iter(HP + "p")):
            out += fit_lines(doc, p, max_tighten)
    return out


def orphans_in_saved(doc):
    """한글이 저장한 줄 배치(lineseg)로, 이미 '한 단어만 다음 줄'인 곳을 정확히 찾는다.
    (사용자가 올린 원본 확인용 - 우리가 고친 문단은 lineseg가 없어 해당 없음)"""
    found = []
    for _, sec in doc.sections():
        for p in sec.iter(HP + "p"):
            segs = p.findall("hp:linesegarray/hp:lineseg", NS)
            if len(segs) < 2:
                continue
            s = []
            for r in p.iterfind("hp:run", NS):
                for ch in r:
                    n = _ln(ch)
                    if n == "t":
                        s += list(ch.text or "")
                        for sub in ch:
                            s.append("\n" if _ln(sub) == "lineBreak" else "\t")
                            s += list(sub.tail or "")
                    elif n in OBJ_TAGS or n == "ctrl":
                        s += ["\x00"] * 8
            pos = [int(x.get("textpos")) for x in segs] + [len(s)]
            full = "".join(s)
            for a, b in zip(pos, pos[1:]):
                line = "".join(s[a:b]).replace("\x00", "")
                prev_end = full[a - 1] if a else "\n"
                ls = full.rfind("\n", 0, a) + 1                     # 이 줄이 속한 Shift+Enter 줄의 시작
                logical = full[ls:full.find("\n", a) if "\n" in full[a:] else len(full)]
                if prev_end != "\n" and line.strip() and " " not in line.strip() and " " in logical.strip():
                    found.append(("".join(s).replace("\x00", "")[:50], line.strip()))
    return found


if __name__ == "__main__":
    import sys
    dump_structure(sys.argv[1])
```
