# התקנה מאפס

## 0. חשבונות

n8n Cloud · Airtable · OpenAI · שני בוטי טלגרם מ-BotFather · חשבון גוגל
(Gmail + Drive) · GitHub (לאירוח לוח הבקרה). אין תוכנה להתקין.

## 1. Airtable

צור בסיס עם ארבע הטבלאות שב-[`../data/airtable-schema.md`](../data/airtable-schema.md),
**עם שמות השדות בדיוק כפי שהם שם**.

ארבע מלכודות שחוסכות שעה של תקלות:

1. `Created` חייב להיות מסוג **Created time** ב-Invoices וב-Leads.
   בלעדיו הטריגרים לא יופעלו — בלי הודעת שגיאה.
2. `Status` חייב להיות **טקסט רגיל** ולא Single select — ב-Invoices
   וב-Leads. ב-`Tasks` דווקא Single select עם `Open` ו-`Done`.
3. `InvoiceNumber` חייב להיות **טקסט** ולא Number.
4. `DueAt` בטבלת `Tasks` חייב להיות **Date עם `Include time` דלוק**.

ייבא את `data/products.csv` לטבלת Products (Import data ← CSV).
מחק את שורת הכותרת אם היא נקלטה כרשומה.

## 2. Credentials ב-n8n

שישה חיבורים תחת **Credentials ← Add credential**:

Airtable · OpenAI · טלגרם (בוט מנהל) · טלגרם (בוט לקוחות) · Gmail · Google Drive

**שני חיבורים דורשים הרשאת דומיין.** n8n חוסם כברירת מחדל שימוש
בקרדנציאל מתוך נוד HTTP Request. פתח את הקרדנציאל ← **Details** ←
**Allowed HTTP Request Domains** ← `Specific Domains`:

| קרדנציאל | דומיין | למה |
|---|---|---|
| Google Drive | `googleapis.com` | WF8 ממיר את החשבונית למסמך Google |
| Airtable | `api.airtable.com` | WF13 מעדכן רשומות |

בלי זה שני התהליכים נכשלים בהרשאה, בלי רמז ברור מה חסר.

## 3. ייבוא ה-Workflows

לכל קובץ ב-`workflows/`: ב-n8n **Create Workflow** ← תפריט `...` ←
**Import from File**.

בכל תהליך שתי החלפות חובה:

- בכל צומת Airtable — לבחור מהרשימה את הבסיס שלך, **ואז לבחור מחדש גם את הטבלה**
- בכל צומת — לבחור את ה-credential שחיברת

## 4. מילוי המאגר הווקטורי

הרץ ידנית **WF6** ואחריו **WF7** (Execute workflow).
העלה את `knowledge-base/all-policies-merged.md` ואת `data/products.csv`.

צפוי: 157 קטעים ב-WF6, 34 ב-WF7.

**שני אלה נשארים ידניים לתמיד.** אחרי כל הפעלה מחדש של n8n — להריץ שוב.

## 5. הפעלה

Publish ל-WF1, WF3, WF4a, WF4b, WF5, WF8, WF9, WF13.
**לא** ל-WF6 ו-WF7.

ב-WF8 ודא שה-Schedule Trigger מכוון ל**שעה** ולא לדקה — טריגר דקתי
צורך כ-1,440 executions ביום ומרוקן מכסה חינמית בפחות משבוע.

## 6. שער הזהות של סוכן המנהל

שלח הודעה לבוט המנהל, פתח את ההרצה ב-Executions, וקח את
`message.chat.id`. הזן אותו בתנאי ה-IF של WF9.

בלעדיו כל אדם שימצא את הבוט יוכל לשאול "מה ההכנסות?".

## 7. לוח הבקרה

ב-WF13 העתק את **Production URL** (לא Test URL) — הכתובת עם `/webhook/`
ולא `/webhook-test/`.

1. ב-`dashboard.html`, בראש הקובץ, הצב אותה בקבוע `API`
2. `git commit` ו-`git push`
3. ב-GitHub: **Settings ← Pages ← Deploy from branch ← main / root**
4. הכתובת שמתקבלת היא זו שהבוט שולח לבעלים

הכתובת מוטמעת בקוד שרץ בדפדפן, וזו בחירה מודעת: ה-Webhook הוא הדבר
היחיד שנחשף, ומפתח ה-API של Airtable נשאר בתוך n8n.

## 8. אפליקציית הניהול

נבנתה ב-Lovable מהפרומפט שב-[`app-prompt.md`](app-prompt.md).
היא פונה לאותו Production URL של WF13, לפי החוזה שב-[`api-contract.md`](api-contract.md).

## 9. בדיקה מקצה לקצה

- לבוט הלקוחות: `מה מדיניות ההחזרות?` → תשובה עם 14 ימים
- לבוט הלקוחות: `מה מזג האוויר?` → סירוב מנומס
- לבוט המנהל: `מה ההכנסות?` → סכום שתואם לחישוב ידני, עם קישור ללוח הבקרה
- הודעה לבוט המנהל מ-Chat ID אחר → נחסמת
- חשבונית חדשה ב-Airtable → מע"מ, מספר, ותוך שעה קישור למסמך Google
- ליד חדש ב-webhook של WF3 → נוצר; אותו ליד שוב → נחסם ככפילות
- לוח הבקרה → האריחים מתמלאים
- אפליקציה: יצירת משימה, ואז סימון שלה כ-`Done` → משתנה ב-Airtable
