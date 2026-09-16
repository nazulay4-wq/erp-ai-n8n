# פרומפט לבניית אפליקציית הניהול

הפרומפט נוסה גם ב-Base44 וגם ב-Lovable. **הגרסה המוגשת נבנתה ב-Lovable** —
אותו פרומפט בדיוק, התוצאה שם נראתה טוב יותר. שם הקובץ נשמר מהמפרט המקורי.

להדביק כבלוק אחד בשורת הצ'אט של הכלי.

---

## 0. API contract — הכי חשוב, אל תנחש

All data comes from one endpoint. No backend function, no database, no keys.
Call it directly from the browser with `fetch`.

```
POST https://<n8n>/webhook/<path>
Content-Type: application/json
```

| action | body | returns |
|---|---|---|
| list | `{ "action":"list", "table":"Invoices" }` | array of records |
| create | `{ "action":"create", "table":"Tasks", "payload":{ ... } }` | the new record |
| update | `{ "action":"update", "table":"Tasks", "payload":{ "id":"rec...", "Status":"Done" } }` | the updated record |
| dashboard | `{ "action":"dashboard" }` | invoice totals grouped by status |
| chat | `{ "action":"chat", "message":"..." }` | `[{ "output":"..." }]` |

Three rules that break the app if ignored:

- fields for create/update go under **`payload`**, never under `fields`
- on update the record id sits **inside** `payload`, next to the fields
- the response is always an **array**, and record fields live under `record.fields`,
  not at the top level

Tables: `Invoices`, `Leads`, `Products`, `Tasks`.
Update currently supports the `Status` field only, on any table.

## 1. Pages
- לוח בקרה — KPIs + charts, from `dashboard` + `list`
- חשבוניות · לידים · מוצרים — one table each, from `list`
- משימות — `list` on `Tasks`, a form that `create`s a task, and a checkbox per row
  that `update`s its `Status` to `Done`. Fields: `Title` (text), `DueAt`
  (datetime, ISO string), `Status` (`Open` / `Done`)
- צ'אט — sends `chat` and renders `[0].output`

## 2. Data / logic
- Dashboard tile "סך ההכנסות": sum of `record.fields.Total` for invoices whose
  `Status` is "Issued" or "Paid". Format as ₪ with thousands separators.
- Invoices table: a "מסמך" column — if `record.fields.PdfUrl` exists render a link
  "פתח מסמך" opening in a new tab, otherwise "—".
- Tasks: sort by `DueAt` ascending, `Open` before `Done`, and show an overdue task
  in the orange series color.

## 3. Hebrew
Translate status values everywhere they are DISPLAYED (tables, badges, charts).
Keep sending the English values to the API.

- Leads: New→חדש, Contacted→נוצר קשר, Replied→השיב
- Invoices: Queued→ממתינה, Issued→הופקה, Paid→שולמה, Invalid→פסולה
- Tasks: Open→פתוחה, Done→בוצעה

Column headers:
InvoiceNumber→מספר מסמך, CustomerId→לקוח, Amount→סכום, VatAmount→מע"מ,
Total→סה"כ, Status→סטטוס, Created→נוצר, Name→שם, Email→אימייל,
Company→חברה, Category→קטגוריה, Price→מחיר, InStock→במלאי,
Title→משימה, DueAt→מועד

## 4. Layout
Every table must fit its container on desktop without horizontal scrolling —
reduce padding, size columns to content. Amount columns must never be cut off.

## 5. Visual design
Styling only.

TYPOGRAPHY
- UI font Heebo. Numbers use tabular-nums so columns align.
- Page title 24px/700 · section title 15px/600 · body 13.5px.
- Table headers 10.5px, uppercase, letter-spacing .08em, muted gray, monospace.
- KPI numbers 30px/700 with a small muted caption beneath.

COLOR — exactly these, nothing else
- background #f7f7f5 · surface #ffffff
- text #1a1a1a · secondary #5c5c5c · muted #8a8a8a
- borders #e6e4e0
- series: blue #2a78d6 · orange #eb6834 · green #1baf7a
- status badges: soft tinted background with darker text of the same hue — not solid
  saturated pills.
- Sidebar: white + a right border. Active item = light gray background with a blue
  right-edge marker.

LAYOUT
- Cards: white, 1px #e6e4e0 border, 10px radius, shadow 0 1px 2px rgba(0,0,0,.05),
  20px padding. No heavy shadows, no gradients.
- KPI row: 4 equal cards in one row → 2 → 1 as the screen narrows.
- 20px gap between cards, 28px between sections.

TABLES
- No vertical borders. One 1px bottom border per row, none on the last row.
- Row height ~44px, cell padding 11px/14px.
- Numeric columns right-aligned, tabular-nums. Subtle row hover #fafaf9.

CHARTS
- Bars: single blue #2a78d6 on a #eeece8 track, 14px tall, 4px radius.
- Donut: thin ring, 3.5% gap between segments, total in the center.
- No 3D, no shadows, no rainbow palettes.

BUTTONS
- Primary: solid #1a1a1a, white text, 8px radius. Secondary: white with a border.
  Not blue, not fully rounded.

Overall feel: calm, editorial, precise — a well-made financial report, not a colorful
SaaS template.
