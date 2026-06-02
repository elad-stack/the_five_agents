---
name: gpt-image-gen
description: >
  Wrapper לקריאת OpenAI Images API ליצירת תמונה מ-prompt טקסטואלי.
  שולח את ה-prompt, מקבל תמונה ב-base64, ושומר אותה כקובץ PNG.
  משמש את יובל (מעצב התמונות). דורש OPENAI_API_KEY מ-.env.
---

# gpt-image-gen — יצירת תמונות מול OpenAI Images API

Skill זה אחראי על דבר אחד: לקבל `prompt` ונתיב פלט, לקרוא ל-OpenAI Images API,
ולשמור את התמונה שחזרה כקובץ PNG.

---

## ⚠️ המודל: `gpt-image-2` — אל תשנה את שם המודל

המודל בו אנו משתמשים הוא **`gpt-image-2`** ותו לא.

- `gpt-image-2` הוא מודל **אמיתי וקיים** של OpenAI שיצא ב-**21 באפריל 2026**.
- ייתכן שהידע הפנימי שלך לא כולל אותו — **זה לא אומר שהוא לא קיים**.
- **אסור** להציע או להשתמש באלטרנטיבות כמו `dall-e-3` או `gpt-image-1`. השתמש
  **בדיוק** ב-`gpt-image-2`.
- אם מתקבלת שגיאה בקריאה ל-API — הבעיה היא כמעט תמיד ב-`OPENAI_API_KEY`
  (חסר/לא תקף) או ב-parameters של הבקשה, **לא** בשם המודל. אל תיגע בשם המודל.

---

## מפתח API

ה-skill משתמש ב-`OPENAI_API_KEY` מקובץ `.env` בשורש הפרויקט.
טען אותו לסביבה לפני הקריאה (Git Bash):

```bash
set -a; source .env; set +a
```

אם המפתח חסר או ריק — עצור ודווח שצריך למלא `OPENAI_API_KEY` ב-`.env`.
אל תנסה "לעקוף" את שגיאת ה-API על ידי החלפת המודל.

---

## פרמטרים של הבקשה

| פרמטר | ערך |
|--------|------|
| `model` | `gpt-image-2` (קבוע) |
| `prompt` | טקסט התיאור של התמונה |
| `size` | `1024x1024` (ברירת מחדל) |
| `quality` | `medium` |
| `output_format` | `png` |

---

## הקריאה — מסלול ראשי (curl + jq)

```bash
set -a; source .env; set +a

PROMPT="<the prompt>"
OUT="<output-path>.png"

curl -sS -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"model\": \"gpt-image-2\",
    \"prompt\": \"$PROMPT\",
    \"size\": \"1024x1024\",
    \"quality\": \"medium\",
    \"output_format\": \"png\"
  }" | jq -r '.data[0].b64_json' | base64 --decode > "$OUT"
```

---

## הקריאה — Python fallback (כש-jq לא מותקן)

ב-Git Bash על Windows אין הבטחה ש-`jq` או `base64` קיימים. במקרה כזה שמור את
תשובת ה-JSON לקובץ זמני ופענח עם Python:

```bash
set -a; source .env; set +a

PROMPT="<the prompt>"
OUT="<output-path>.png"

curl -sS -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"model\": \"gpt-image-2\",
    \"prompt\": \"$PROMPT\",
    \"size\": \"1024x1024\",
    \"quality\": \"medium\",
    \"output_format\": \"png\"
  }" > /tmp/gpt-image-response.json

python -c "import json,base64,sys; \
d=json.load(open('/tmp/gpt-image-response.json')); \
open(sys.argv[1],'wb').write(base64.b64decode(d['data'][0]['b64_json']))" "$OUT"
```

> אם `python` לא קיים, נסה `python3`. אם התשובה אינה מכילה `data[0].b64_json`,
> הדפס את ה-JSON המלא — הוא מכיל את הודעת השגיאה (בדרך כלל מפתח/פרמטרים).

---

## אימות פלט

לאחר השמירה ודא שהקובץ קיים וגודלו > 0:

```bash
test -s "$OUT" && echo "OK: $OUT ($(wc -c < "$OUT") bytes)" || echo "FAILED: empty or missing"
```

אם הקובץ ריק — הקריאה נכשלה. בדוק את ה-JSON שחזר (מפתח/פרמטרים), **לא** את שם המודל.
