<div align="center">

# AI-ERP

**מערכת ניהול עסק חכמה**

פרויקט גמר · ג'ון ברייס

מערכת ERP קטנה לעסק אלקטרוניקה ישראלי, שבה סוכני בינה מלאכותית
ותהליכי אוטומציה מבצעים את רוב העבודה התפעולית.

<br>

![n8n](https://img.shields.io/badge/n8n-10%20workflows-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![Airtable](https://img.shields.io/badge/Airtable-4%20tables-18BFFF?style=for-the-badge&logo=airtable&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-3%20agents-412991?style=for-the-badge&logo=openai&logoColor=white)
![Telegram](https://img.shields.io/badge/Telegram-2%20bots-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)

<br>

### [↗ פתח את לוח הבקרה החי](https://nazulay4-wq.github.io/erp-ai-n8n/dashboard.html)

<br>

![לוח הבקרה](screenshots/dashboard-hero.png)

</div>

<br>

<div align="center">

| | | | | |
|:---:|:---:|:---:|:---:|:---:|
| **0** | **10** | **34** | **191** | **18%** |
| שורות קוד | Workflows | מוצרים בקטלוג | קטעים ב-RAG | מע"מ אוטומטי |

</div>

<br>

---

## מה המערכת עושה

<table>
<tr>
<td width="50%" valign="top">

### לידים
קליטה מטופס, סינון כפילויות, מייל קר אוטומטי, וזיהוי תשובה מהג'ימייל — בלי נגיעה אנושית.

</td>
<td width="50%" valign="top">

### שירות לקוחות
בוט טלגרם שעונה **רק** מתוך מסמכי המדיניות והקטלוג. לא יודע? מפנה לנציג.

</td>
</tr>
<tr>
<td valign="top">

### מסמכי מס
מזינים לקוח, מק"ט וכמות. המערכת שולפת מחיר מהקטלוג, מחשבת מע"מ, ממספרת, ומפיקה חשבונית מעוצבת.

</td>
<td valign="top">

### הנהלה
בוט טלגרם לבעלים עם נתוני הכנסות חיים, מוגן בשער זהות לפי `chat.id`.

</td>
</tr>
<tr>
<td valign="top">

### משימות
מעקב אחר משימות ופגישות. נקרא ונכתב מהאפליקציה דרך אותו webhook.

</td>
<td valign="top">

### ממשק
לוח בקרה לבעלים ואפליקציית ניהול — שניהם מדברים עם כתובת אחת בלבד.

</td>
</tr>
</table>

> **אפס שורות קוד.** כל הלוגיקה מיושמת בצמתים מוכנים של n8n.

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

שלוש שכבות: **ממשק** · **לוגיקה** · **נתונים**. שתי שכבות הממשק מדברות עם
Webhook יחיד ולא נוגעות ב-Airtable ישירות, כך שמפתח ה-API לעולם לא מגיע לדפדפן.

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

שלושה תהליכים נפרדים, ששרשרו זה לזה דרך שדה `Status` אחד:

```mermaid
flowchart RL
    F["טופס לידים"] -->|ליד נכנס| W3["WF3 · סינון כפילויות"]
    W3 -->|New| W4a["WF4a · מייל קר<br/>כל 3 שעות"]
    W4a -->|Contacted| W4b["WF4b · זיהוי תשובה<br/>Gmail, כל 30 דקות"]
    W4b -->|Replied| HOT["ליד חם<br/>מוכן לטיפול אנושי"]

    style F fill:#0f1419,stroke:#6b7280,color:#fff
    style W3 fill:#16213e,stroke:#3987e5,stroke-width:2px,color:#fff
    style W4a fill:#16213e,stroke:#3987e5,stroke-width:2px,color:#fff
    style W4b fill:#16213e,stroke:#3987e5,stroke-width:2px,color:#fff
    style HOT fill:#0f3d2e,stroke:#199e70,stroke-width:2px,color:#fff
```

אף תהליך לא יודע על האחרים. כל אחד מסתכל על `Status`, עושה את שלו, ומקדם אותו.

## מסלול מסמכי המס

מזינים **לקוח · מק"ט · כמות** — וזהו. כל השאר מחושב.

```mermaid
flowchart RL
    IN["לקוח · מק״ט · כמות"] --> W1["WF1<br/>שליפת המוצר מהקטלוג<br/>מחיר × כמות · מע״מ 18%<br/>מספר רץ"]
    W1 --> Q(["Queued"])
    Q --> W8["WF8<br/>HTML ← Drive<br/>← Google Doc"]
    W8 --> I(["Issued<br/>+ קישור למסמך"])

    style IN fill:#0f1419,stroke:#6b7280,color:#fff
    style W1 fill:#16213e,stroke:#3987e5,stroke-width:2px,color:#fff
    style W8 fill:#16213e,stroke:#3987e5,stroke-width:2px,color:#fff
    style Q fill:#4a2617,stroke:#d95926,stroke-width:2px,color:#fff
    style I fill:#0f3d2e,stroke:#199e70,stroke-width:2px,color:#fff
```

זה הקשר היחיד במערכת שבו שתי טבלאות באמת מדברות ביניהן: WF1 מחפש
ב-`Products` לפי המק"ט ומביא משם שם ומחיר. אי אפשר להוציא חשבונית על
מוצר שאינו בקטלוג, ואי אפשר לטעות במחיר.

---

## הקטלוג

**34 מוצרים ב-13 קטגוריות** — 22 מוצרים פיזיים ו-12 שירותים.

<div align="center">
<table>
<tr>
<td align="center" width="25%"><img src="https://images.unsplash.com/photo-1641048930621-ab5d225ae5b0?w=400&h=260&fit=crop&q=75" width="100%"><br><b>אוזניות</b></td>
<td align="center" width="25%"><img src="https://images.unsplash.com/photo-1585792180666-f7347c490ee2?w=400&h=260&fit=crop&q=75" width="100%"><br><b>מסכים</b></td>
<td align="center" width="25%"><img src="https://images.unsplash.com/photo-1626958390898-162d3577f293?w=400&h=260&fit=crop&q=75" width="100%"><br><b>מקלדות</b></td>
<td align="center" width="25%"><img src="https://images.unsplash.com/photo-1605773527852-c546a8584ea3?w=400&h=260&fit=crop&q=75" width="100%"><br><b>עכברים</b></td>
</tr>
<tr>
<td align="center"><img src="https://images.unsplash.com/photo-1608043152269-423dbba4e7e1?w=400&h=260&fit=crop&q=75" width="100%"><br><b>רמקולים</b></td>
<td align="center"><img src="https://images.unsplash.com/photo-1636569826709-8e07f6104992?w=400&h=260&fit=crop&q=75" width="100%"><br><b>מצלמות רשת</b></td>
<td align="center"><img src="https://images.unsplash.com/photo-1628557118391-56cd62c9f2cb?w=400&h=260&fit=crop&q=75" width="100%"><br><b>אחסון</b></td>
<td align="center"><img src="https://images.unsplash.com/photo-1615774925655-a0e97fc85c14?w=400&h=260&fit=crop&q=75" width="100%"><br><b>שירותים</b></td>
</tr>
</table>
</div>

כל מוצר נושא שדה `ImageUrl`, וכך התמונה מגיעה **מהנתונים ולא מהקוד** —
הוספנו יכולת שלמה למערכת בלי לגעת באף Workflow.

---

## אפליקציית הניהול

שישה מסכים, ונקודת קצה אחת. האפליקציה אינה מכירה את Airtable ואין בה
אף מפתח — כל קריאה עוברת ב-Webhook של WF13.

<div align="center">
<table>
<tr>
<td align="center" width="50%"><img src="screenshots/app-products.png" width="100%"><br><b>מוצרים</b><br><sub>רשת כרטיסים עם תמונה מ-<code>ImageUrl</code></sub></td>
<td align="center" width="50%"><img src="screenshots/app-dashboard.png" width="100%"><br><b>לוח בקרה</b><br><sub>ארבעה מדדים, היקף עסקאות וסטטוס חשבוניות</sub></td>
</tr>
<tr>
<td align="center"><img src="screenshots/app-tasks.png" width="100%"><br><b>משימות</b><br><sub>יצירה וסימון כבוצעה — כתיבה חוזרת ל-Airtable</sub></td>
<td align="center"><img src="screenshots/app-chat.png" width="100%"><br><b>צ'אט</b><br><sub>אותו סוכן RAG שעונה גם בטלגרם</sub></td>
</tr>
</table>
</div>

---

## מבנה הריפו

```
workflows/        10 קובצי JSON לייבוא ל-n8n
knowledge-base/   12 מסמכי מדיניות + קובץ מאוחד למאגר הווקטורי
data/             קטלוג המוצרים וסכימת Airtable
docs/             ארכיטקטורה · התקנה · חוזה API · העברה · פרומפט האפליקציה
screenshots/      צילומי מסך להגשה
dashboard.html    לוח הבקרה לבעלים
```

## תיעוד

| קובץ | עונה על השאלה |
|:---|:---|
| [`docs/architecture.md`](docs/architecture.md) | איך המערכת בנויה, ולמה כל החלטה התקבלה |
| [`docs/setup.md`](docs/setup.md) | איך מקימים אותה מאפס |
| [`docs/api-contract.md`](docs/api-contract.md) | איך מדברים איתה |
| [`docs/migration-checklist.md`](docs/migration-checklist.md) | איך מעבירים למופע n8n אחר |
| [`docs/app-prompt.md`](docs/app-prompt.md) | איך נבנתה אפליקציית הניהול |

---

<details>
<summary><b>מגבלות מוכרות</b> — בחירות תכנון מודעות, לא באגים</summary>

<br>

| מגבלה | למה זו בחירה |
|:---|:---|
| המאגר הווקטורי בזיכרון | פשטות; המחיר הוא הרצה ידנית של WF6/WF7 אחרי כל restart |
| החשבונית היא Google Doc ולא PDF | הדרייב אינו מציג HTML; ייצוא ל-PDF נעשה מתוך המסמך |
| טריגרי Airtable סורקים במרווחים | אין webhook יוצא ב-Airtable בתוכנית החינמית |
| מספור עלול להתנגש בשתי חשבוניות באותה דקה | Autonumber אינו ניתן לכתיבה מ-n8n |
| Google OAuth במצב Testing | טוקן הרענון פג אחרי 7 ימים |
| אין טיפול בשגיאות וניסיונות חוזרים | כשל נראה אדום ב-Executions — מספיק להדגמה |
| סוכן המנהל רואה עד 100 חשבוניות | המספרים מחושבים לפניו, לא על ידו |
| WF4a שולח ליד אחד בכל הרצה | כדי שטעות לא תשלח עשרות מיילים |
| ענף העדכון ב-WF13 מעדכן `Status` בלבד | נוד HTTP אחד משרת את כל הטבלאות |
| משימות נוצרות ידנית | אין עדיין תהליך שיוצר אותן אוטומטית |

</details>

<details>
<summary><b>אבטחה</b></summary>

<br>

קובצי ה-workflow שבריפו **אינם מכילים מפתחות או סיסמאות** — רק שמות
ה-credentials כפי שהם מוגדרים ב-n8n. את החיבורים עצמם יש להגדיר מחדש
בכל מופע, לפי [`docs/setup.md`](docs/setup.md).

מפתח ה-API של Airtable נשאר בתוך n8n ולעולם אינו מגיע לדפדפן — זו הסיבה
שהממשקים מדברים עם WF13 ולא עם Airtable ישירות.

ה-Webhook של WF13 פתוח וללא אימות — בחירה מודעת לצורך הדגמה. במערכת
אמיתית היה נוסף לו בדיקת `x-api-key` בצומת ה-Switch.

בוט המנהל מוגן בשער זהות לפי `chat.id`; הודעה מכל מזהה אחר נחסמת.

</details>
