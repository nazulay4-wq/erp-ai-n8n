# חוזה ה-API — WF13

נקודת הכניסה היחידה מלוח הבקרה ומאפליקציית הניהול אל המערכת.

```
POST https://<n8n-instance>/webhook/<path>
Content-Type: application/json
```

**הכתובת הפעילה כרגע:**

```
https://n4n.app.n8n.cloud/webhook/33543bff-1acb-4971-9d75-517aa49f351d
```

כל בקשה מכילה שדה `action` שקובע את הניתוב ב-Switch.

---

## list — שליפת רשומות

```json
{ "action": "list", "table": "Invoices" }
```

מחזיר מערך של רשומות:

```json
[
  {
    "id": "recXXXXXXXXXXXXXX",
    "createdTime": "2026-09-10T11:54:16.000Z",
    "fields": {
      "InvoiceNumber": "INV-0002",
      "CustomerId": "CUST-0002",
      "Amount": 3000,
      "VatAmount": 540,
      "Total": 3540,
      "Status": "Issued",
      "PdfUrl": "https://docs.google.com/document/d/.../preview"
    }
  }
]
```

טבלאות אפשריות: `Invoices`, `Leads`, `Products`, `Tasks`.

---

## create — יצירת רשומה

```json
{
  "action": "create",
  "table": "Tasks",
  "payload": {
    "Title": "לחזור לדנה כהן",
    "DueAt": "2026-09-20T10:00:00.000Z",
    "Status": "Open"
  }
}
```

מפתחות ה-`payload` חייבים להיות **זהים בדיוק** לשמות העמודות ב-Airtable.
מחזיר את הרשומה שנוצרה.

---

## update — עדכון רשומה

**מזהה הרשומה נמצא בתוך `payload`, לא לצדו.**

```json
{
  "action": "update",
  "table": "Tasks",
  "payload": { "id": "recXXXXXXXXXXXXXX", "Status": "Done" }
}
```

**מגבלה נוכחית:** הענף מעדכן את שדה `Status` בלבד. הוא עובד על כל
טבלה — `table` קובע לאן — אבל שדות אחרים יידרשו הרחבה של נוד ה-HTTP.

מימוש: הענף אינו משתמש בנוד Airtable אלא בנוד **HTTP Request** שפונה
ישירות ל-API. נוד Airtable אינו יכול לעדכן טבלה שנקבעת בזמן ריצה, כי
הוא דורש סכימת עמודות ידועה מראש. ה-HTTP Request לא דורש סכימה, ולכן
נוד אחד משרת את כל הטבלאות.

```
PATCH https://api.airtable.com/v0/<baseId>/<table>/<recordId>
{ "fields": { "Status": "Done" } }
```

**דורש** שהקרדנציאל של Airtable ירשה את הדומיין `api.airtable.com`
תחת Allowed HTTP Request Domains.

---

## dashboard — סיכומים

```json
{ "action": "dashboard" }
```

מחזיר סיכום חשבוניות מקובץ לפי סטטוס:

```json
[ { "Status": "Issued", "count_Status": 4, "sum_Total": 8024 } ]
```

---

## chat — שאלה לסוכן

```json
{ "action": "chat", "message": "מה מדיניות ההחזרות?" }
```

מחזיר:

```json
[ { "output": "מדיניות ההחזרות שלנו: בעסקת מכר מרחוק..." } ]
```

הסוכן זהה לזה של WF5 — אותו System Message ואותם שני כלי RAG.

---

## הערות למימוש בממשק

- התשובה היא תמיד **מערך**, גם כשיש פריט אחד
- שדות הרשומה יושבים תחת `fields` ולא ברמה העליונה
- `create` ו-`update` מקבלים את השדות תחת `payload`, לא תחת `fields`
- הקריאה נעשית **ישירות מהדפדפן**. זה מכוון: מפתח ה-API של Airtable
  נשאר בתוך n8n ולעולם אינו מגיע לקוד שרץ אצל המשתמש
- ה-Webhook עצמו פתוח וללא אימות — בחירה מודעת לצורך הדגמה. במערכת
  אמיתית הייתה נוספת בדיקת `x-api-key` בצומת ה-Switch
