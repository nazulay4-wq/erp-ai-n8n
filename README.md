<div align="center">

# AI-ERP

### מערכת ניהול עסק חכמה · פרויקט גמר · ג'ון ברייס

מערכת ERP קטנה לעסק אלקטרוניקה ישראלי, שבה סוכני בינה מלאכותית
ותהליכי אוטומציה מבצעים את רוב העבודה התפעולית.

![n8n](https://img.shields.io/badge/n8n-10%20workflows-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![Airtable](https://img.shields.io/badge/Airtable-4%20tables-18BFFF?style=for-the-badge&logo=airtable&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-3%20agents-412991?style=for-the-badge&logo=openai&logoColor=white)
![Telegram](https://img.shields.io/badge/Telegram-2%20bots-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)

![Zero code](https://img.shields.io/badge/lines_of_code-0-success?style=flat-square)
![RAG](https://img.shields.io/badge/RAG-191_chunks-blue?style=flat-square)
![VAT](https://img.shields.io/badge/VAT-18%25_automated-blue?style=flat-square)

**[↗ לוח הבקרה החי](https://nazulay4-wq.github.io/erp-ai-n8n/dashboard.html)**

</div>

---

<div align="center">

![לוח הבקרה](screenshots/dashboard-hero.png)

</div>

---

## מה המערכת עושה

| תחום | מה קורה אוטומטית |
|:---|:---|
| **לידים** | קליטה מטופס · סינון כפילויות · מייל קר · זיהוי תשובה |
| **שירות לקוחות** | בוט טלגרם שעונה מתוך מסמכי המדיניות והקטלוג (RAG) |
| **מסמכי מס** | שליפת מוצר מהקטלוג · תמחור · מע"מ 18% · מספור רץ · חשבונית מעוצבת |
| **הנהלה** | בוט טלגרם לבעלים עם נתוני הכנסות, מוגן בשער זהות |
| **משימות** | מעקב אחר משימות ופגישות, נקרא ונכתב מהאפליקציה |
| **ממשק** | לוח בקרה לבעלים · אפליקציית ניהול — שניהם דרך webhook יחיד |

**אפס שורות קוד.** כל הלוגיקה מיושמת בצמתים מוכנים של n8n.

---

## ארכיטקטורה

```mermaid
flowchart RL
    TG["בוטי טלגרם<br/>לקוחות · מנהל"] ==> N8N
    FORM["טופס לידים"] ==> N8N
    GM["Gmail נכנס"] ==> N8N
    UI["לוח בקרה<br/>אפליקציית ניהול"] ==> N8N
    CLK["לוחות זמנים"] ==> N8N

    N8N["n8n Cloud<br/>10 Workflows<br/>3 סוכני AI<br/>2 מאגרים וקטוריים"]

    N8N ==> AT["Airtable<br/>4 טבלאות"]
    N8N ==> GD["Google Drive<br/>חשבוניות"]
    N8N ==> GM2["Gmail יוצא<br/>מיילי מכירות"]
    N8N ==> OA["OpenAI<br/>מודלים"]

    style N8N fill:#16213e,stroke:#3987e5,stroke-width:3px,color:#fff
    style AT fill:#0f1419,stroke:#199e70,color:#fff
    style GD fill:#0f1419,stroke:#199e70,color:#fff
    style GM2 fill:#0f1419,stroke:#199e70,color:#fff
    style OA fill:#0f1419,stroke:#199e70,color:#fff
    style TG fill:#0f1419,stroke:#d95926,color:#fff
    style FORM fill:#0f1419,stroke:#d95926,color:#fff
    style GM fill:#0f1419,stroke:#d95926,color:#fff
    style UI fill:#0f1419,stroke:#d95926,color:#fff
    style CLK fill:#0f1419,stroke:#d95926,color:#fff
```

שלוש שכבות: **ממשק** (לוח בקרה + אפליקציה) · **לוגיקה** (n8n) · **נתונים** (Airtable).
שתי שכבות הממשק מדברות עם Webhook יחיד ולא נוגעות ב-Airtable ישירות,
כך שמפתח ה-API לעולם לא מגיע לדפדפן.

פירוט מלא: [`docs/architecture.md`](docs/architecture.md)

---

## עשרת ה-Workflows

| WF | מה עושה | טריגר |
|:---:|:---|:---|
| **1** | אימות מסמכי מס, שליפת מוצר, מע"מ ומספור רץ | חשבונית חדשה |
| **3** | קליטת לידים וסינון כפילויות | Webhook |
| **4a** | מיילים קרים ללידים | כל 3 שעות |
| **4b** | זיהוי תשובה ועדכון סטטוס | Gmail · כל 30 דק' |
| **5** | סוכן שירות לקוחות | בוט טלגרם |
| **6** | מדיניות ← מאגר וקטורי | ידני |
| **7** | מוצרים ← מאגר וקטורי | ידני |
| **8** | הפקת חשבונית והעלאה לדרייב | כל שעה |
| **9** | סוכן המנהל | בוט טלגרם |
| **13** | שער ה-API לממשקים | Webhook |

המספור נשמר יציב לפי מפרט הפרויקט — 2, 10, 11 ו-12 הושמטו בכוונה.

> **WF6 ו-WF7 נשארים ידניים לתמיד.** המאגר הווקטורי יושב בזיכרון של n8n
> ונמחק בכל הפעלה מחדש; יש להריץ אותם שוב לפני כל שימוש בסוכנים.

---

## מחזור הלידים

שלושה תהליכים נפרדים ששרשרו דרך שדה `Status` אחד:

```mermaid
flowchart RL
    F["טופס"] --> W3["WF3<br/>סינון כפילויות"]
    W3 --> N(["New"])
    N --> W4a["WF4a<br/>מייל קר"]
    W4a --> C(["Contacted"])
    C --> W4b["WF4b<br/>זיהוי תשובה"]
    W4b --> R(["Replied"])

    style N fill:#1e3a5f,stroke:#3987e5,color:#fff
    style C fill:#4a2617,stroke:#d95926,color:#fff
    style R fill:#0f3d2e,stroke:#199e70,color:#fff
    style W3 fill:#16213e,stroke:#6b7280,color:#fff
    style W4a fill:#16213e,stroke:#6b7280,color:#fff
    style W4b fill:#16213e,stroke:#6b7280,color:#fff
    style F fill:#0f1419,stroke:#6b7280,color:#fff
```

## מסלול מסמכי המס

מזינים **לקוח · מק"ט · כמות** — וזהו. כל השאר מחושב.

```mermaid
flowchart RL
    IN["לקוח + מק&quot;ט + כמות"] --> W1
    subgraph W1["WF1"]
        direction TB
        L["שליפת מוצר מהקטלוג"] --> P["מחיר × כמות"] --> V["מע&quot;מ 18%"] --> NUM["מספר רץ"]
    end
    W1 --> Q(["Queued"])
    Q --> W8["WF8<br/>HTML ← Drive ← Google Doc"]
    W8 --> I(["Issued + קישור למסמך"])

    style W1 fill:#16213e,stroke:#3987e5,stroke-width:2px,color:#fff
    style Q fill:#4a2617,stroke:#d95926,color:#fff
    style I fill:#0f3d2e,stroke:#199e70,color:#fff
    style W8 fill:#16213e,stroke:#6b7280,color:#fff
    style IN fill:#0f1419,stroke:#6b7280,color:#fff
```

זה הקשר היחיד במערכת שבו שתי טבלאות באמת מדברות ביניהן: WF1 מחפש
ב-`Products` לפי המק"ט ומביא משם שם ומחיר. אי אפשר להוציא חשבונית על
מוצר שאינו בקטלוג, ואי אפשר לטעות במחיר.

---

## הקטלוג

34 מוצרים ב-13 קטגוריות — מוצרים פיזיים ו-12 שירותים.

<div align="center">

<img src="https://images.unsplash.com/photo-1641048930621-ab5d225ae5b0?w=260&h=170&fit=crop&q=70" width="150"> <img src="https://images.unsplash.com/photo-1585792180666-f7347c490ee2?w=260&h=170&fit=crop&q=70" width="150"> <img src="https://images.unsplash.com/photo-1626958390898-162d3577f293?w=260&h=170&fit=crop&q=70" width="150"> <img src="https://images.unsplash.com/photo-1608043152269-423dbba4e7e1?w=260&h=170&fit=crop&q=70" width="150">

<sub>אוזניות · מסכים · מקלדות · רמקולים — ועוד תשע קטגוריות</sub>

</div>

כל מוצר נושא `ImageUrl`, וכך התמונה מגיעה מהנתונים ולא מהקוד —
בלי לגעת באף Workflow.

---

## מבנה הריפו

```
workflows/        10 קובצי JSON לייבוא ל-n8n
knowledge-base/   12 מסמכי מדיניות + קובץ מאוחד למאגר הווקטורי
data/             קטלוג המוצרים וסכימת Airtable
docs/             ארכיטקטורה · התקנה · חוזה API · העברה · תסריט הסרטון
screenshots/      צילומי מסך להגשה
dashboard.html    לוח הבקרה לבעלים
```

## תיעוד

| קובץ | עונה על |
|:---|:---|
| [`docs/architecture.md`](docs/architecture.md) | איך המערכת בנויה, ולמה כל החלטה התקבלה |
| [`docs/setup.md`](docs/setup.md) | איך מקימים אותה מאפס |
| [`docs/api-contract.md`](docs/api-contract.md) | איך מדברים איתה |
| [`docs/migration-checklist.md`](docs/migration-checklist.md) | איך מעבירים למופע n8n אחר |
| [`docs/app-prompt.md`](docs/app-prompt.md) | הפרומפט שבנה את אפליקציית הניהול |

---

<details>
<summary><strong>מגבלות מוכרות</strong> — בחירות תכנון מודעות, לא באגים</summary>

<br>

- המאגר הווקטורי בזיכרון — נמחק בכל restart של n8n
- החשבונית היא מסמך Google ולא PDF; ייצוא נעשה מתוך המסמך בעת הצורך
- טריגרי Airtable סורקים כל דקה לפחות ומסתמכים על שדה `Created time`
- מספור חשבוניות עלול להתנגש אם שתיים נוצרות באותה דקה
- Google OAuth במצב Testing — טוקן הרענון פג אחרי 7 ימים
- אין טיפול בשגיאות ואין ניסיונות חוזרים; כשל נראה אדום ב-Executions
- סוכן המנהל רואה עד 100 חשבוניות ואינו מקבל כלים
- WF4a שולח ליד אחד בכל הרצה, כדי שטעות לא תשלח עשרות מיילים
- ענף העדכון ב-WF13 מעדכן את שדה `Status` בלבד
- משימות נוצרות ידנית מהאפליקציה; אין עדיין תהליך שיוצר אותן אוטומטית

</details>

<details>
<summary><strong>אבטחה</strong></summary>

<br>

קובצי ה-workflow שבריפו **אינם מכילים מפתחות או סיסמאות** — רק שמות
ה-credentials כפי שהם מוגדרים ב-n8n. את החיבורים עצמם יש להגדיר מחדש
בכל מופע, לפי [`docs/setup.md`](docs/setup.md).

ה-Webhook של WF13 פתוח וללא אימות — בחירה מודעת לצורך הדגמה. במערכת
אמיתית היה נוסף לו בדיקת `x-api-key` בצומת ה-Switch.

בוט המנהל מוגן בשער זהות לפי `chat.id`; הודעה מכל מזהה אחר נחסמת.

</details>
