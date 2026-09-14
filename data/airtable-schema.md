# סכימת Airtable

ארבע טבלאות. **שמות השדות חייבים להיות זהים בדיוק** — כל התהליכים
מסתמכים עליהם, וחוסר התאמה נכשל בשקט בלי הודעת שגיאה.

## Invoices

| שדה | סוג | הערות |
|---|---|---|
| `InvoiceNumber` | Single line text | שדה ראשי. פורמט `INV-0001` |
| `CustomerId` | Single line text | מפתח זר כטקסט |
| `Amount` | Number | סכום לפני מע"מ |
| `VatAmount` | Number | מחושב ב-WF1 |
| `Total` | Number | מחושב ב-WF1 |
| `Status` | Single line text | `New` → `Queued` → `Issued`, או `Invalid` |
| `PdfUrl` | URL | נכתב ב-WF8 |
| `Created` | **Created time** | הטריגר של WF1 מסתמך עליו |

## Leads

| שדה | סוג |
|---|---|
| `Name` | Single line text |
| `Email` | Email |
| `Company` | Single line text |
| `Status` | Single line text — `New` → `Contacted` → `Replied` |
| `Created` | **Created time** |

## Products

| שדה | סוג |
|---|---|
| `Name` | Single line text |
| `Category` | Single line text |
| `Price` | Currency |
| `Description` | Long text |
| `InStock` | Checkbox |

## Tasks

| שדה | סוג |
|---|---|
| `Title` | Single line text |
| `Status` | Single line text |

---

## למה `InvoiceNumber` הוא טקסט

- שדה Number אינו יכול להחזיק אפסים מובילים או קידומת — `INV-0001` היה נשמר כ-`1`
- זהו השדה הראשי, ולכן שם הרשומה בכל מקום: n8n, האפליקציה, ושם הקובץ בדרייב
- המסמך דורש להבחין בין חשבונית מס, קבלה וחשבונית — קידומת עושה זאת בשדה אחד
- זהו מזהה ולא כמות; הוא לעולם לא מסוכם
- המיון נשמר: עם ריפוד אפסים ברוחב קבוע, מיון אלפביתי זהה למיון מספרי

**לא להשתמש ב-Autonumber** — n8n אינו יכול לכתוב לשדה כזה, והתכנון בנוי
על כך ש-WF1 מחשב את המספר בעצמו.
