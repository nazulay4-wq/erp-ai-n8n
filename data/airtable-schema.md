# סכימת Airtable

ארבע טבלאות. **שמות השדות חייבים להיות זהים בדיוק** — כל התהליכים
מסתמכים עליהם, וחוסר התאמה נכשל בשקט בלי הודעת שגיאה.

## Invoices

| שדה | סוג | הערות |
|---|---|---|
| `InvoiceNumber` | Single line text | שדה ראשי. פורמט `INV-0001`. **נכתב ב-WF1** |
| `CustomerId` | Single line text | מפתח זר כטקסט. **מוזן ידנית** |
| `Sku` | Single line text | מק"ט המוצר שנרכש. **מוזן ידנית** |
| `Qty` | Number | כמות. **מוזנת ידנית** |
| `ProductName` | Single line text | שם המוצר. **נשלף ב-WF1 מטבלת Products** |
| `Amount` | Number | `Price × Qty`. **מחושב ב-WF1** |
| `VatAmount` | Number | מחושב ב-WF1 |
| `Total` | Number | מחושב ב-WF1 |
| `Status` | Single line text | `New` → `Queued` → `Issued`, או `Invalid` |
| `PdfUrl` | URL | נכתב ב-WF8 |
| `Created` | **Created time** | הטריגר של WF1 מסתמך עליו |

**שלושה שדות בלבד מוזנים ידנית** — `CustomerId`, `Sku` ו-`Qty`. כל
השאר מחושב או נשלף. זה הקשר היחיד במערכת שבו שתי טבלאות מדברות
ביניהן: WF1 מחפש ב-Products לפי `Sku`, ומביא משם `Name` ו-`Price`.

## Leads

| שדה | סוג |
|---|---|
| `Name` | Single line text |
| `Email` | Email |
| `Company` | Single line text |
| `Status` | Single line text — `New` → `Contacted` → `Replied` |
| `Created` | **Created time** |

## Products

| שדה | סוג | הערות |
|---|---|---|
| `Name` | Single line text | שדה ראשי. WF1 מעתיק אותו ל-`ProductName` |
| `Sku` | Single line text | מק"ט, פורמט `TY-MN-27Q`. **המפתח שלפיו WF1 מחפש** |
| `Category` | Single line text | |
| `Price` | Currency | ₪. WF1 מכפיל אותו ב-`Qty` |
| `Description` | Long text | מפרט מלא. נכנס למאגר הווקטורי ב-WF7 |
| `InStock` | Checkbox | |

## Tasks

משימות ופגישות מעקב. נקראת ונכתבת מהאפליקציה דרך WF13.

| שדה | סוג | הערות |
|---|---|---|
| `Title` | Single line text | תיאור המשימה. שדה ראשי |
| `DueAt` | **Date with time** | חובה להפעיל את `Include time` — התאריך והשעה יושבים יחד |
| `Status` | Single select | `Open` → `Done` |

---

## למה `InvoiceNumber` הוא טקסט

- שדה Number אינו יכול להחזיק אפסים מובילים או קידומת — `INV-0001` היה נשמר כ-`1`
- זהו השדה הראשי, ולכן שם הרשומה בכל מקום: n8n, האפליקציה, ושם הקובץ בדרייב
- המסמך דורש להבחין בין חשבונית מס, קבלה וחשבונית — קידומת עושה זאת בשדה אחד
- זהו מזהה ולא כמות; הוא לעולם לא מסוכם
- המיון נשמר: עם ריפוד אפסים ברוחב קבוע, מיון אלפביתי זהה למיון מספרי

**לא להשתמש ב-Autonumber** — n8n אינו יכול לכתוב לשדה כזה, והתכנון בנוי
על כך ש-WF1 מחשב את המספר בעצמו.
