# חוזה ה-API — WF13

נקודת הכניסה היחידה מהאפליקציה למערכת.

```
POST https://<n8n-instance>/webhook/<path>
Content-Type: application/json
```

כל בקשה מכילה שדה `action` שקובע את הניתוב.

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
      "PdfUrl": "https://drive.google.com/..."
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
  "table": "Leads",
  "payload": {
    "Name": "רון אלמוג",
    "Email": "ron@example.co.il",
    "Company": "סקיילין",
    "Status": "New"
  }
}
```

מפתחות ה-`payload` חייבים להיות **זהים בדיוק** לשמות העמודות ב-Airtable.
מחזיר את הרשומה שנוצרה.

---

## update — עדכון סטטוס

```json
{
  "action": "update",
  "table": "Leads",
  "payload": { "id": "recXXXXXXXXXXXXXX", "Status": "Contacted" }
}
```

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

## הערות למימוש באפליקציה

- התשובה היא תמיד **מערך**, גם כשיש פריט אחד
- שדות הרשומה יושבים תחת `fields` ולא ברמה העליונה
- שמור את כתובת ה-webhook כ-**secret** בצד השרת, לא בקוד שרץ בדפדפן
