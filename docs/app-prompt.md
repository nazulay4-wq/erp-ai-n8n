# פרומפט לבניית אפליקציית הניהול

הפרומפט נוסה גם ב-Base44 וגם ב-Lovable. **הגרסה המוגשת נבנתה ב-Lovable** —
אותו פרומפט בדיוק, התוצאה שם נראתה טוב יותר.

להדביק כבלוק אחד בשורת הצ'אט של הכלי.

---

## 0. API contract — the part that breaks everything if guessed

All data comes from one endpoint. No backend function, no database, no keys.
Call it directly from the browser with `fetch`.

```
POST https://n4n.app.n8n.cloud/webhook/33543bff-1acb-4971-9d75-517aa49f351d
Content-Type: application/json
```

| action | body | returns |
|---|---|---|
| list | `{ "action":"list", "table":"Invoices" }` | array of records |
| create | `{ "action":"create", "table":"Tasks", "payload":{ ... } }` | the new record |
| update | `{ "action":"update", "table":"Tasks", "payload":{ "id":"rec...", "Status":"Done" } }` | the updated record |
| dashboard | `{ "action":"dashboard" }` | invoice totals grouped by status |
| chat | `{ "action":"chat", "message":"..." }` | `[{ "output":"..." }]` |

Four rules that break the app if ignored:

- fields for create/update go under **`payload`**, never under `fields`
- on update the record id sits **inside** `payload`, next to the fields
- the response is always an **array**, and record fields live under `record.fields`,
  not at the top level
- update currently supports the `Status` field only, on any table

Tables: `Invoices`, `Leads`, `Products`, `Tasks`.

### Field reference

```
Invoices   InvoiceNumber, CustomerId, Sku, Qty, ProductName,
           Amount, VatAmount, Total, Status, PdfUrl, Created
Leads      Name, Email, Company, Status, Created
Products   Name, Sku, Category, Price, Description, InStock, ImageUrl
Tasks      Title, DueAt, Status
```

## 1. Pages

- **לוח בקרה** — KPI row + charts, from `dashboard` and `list`
- **חשבוניות** — table, from `list` on Invoices
- **לידים** — table, from `list` on Leads
- **מוצרים** — a **card grid, not a table**: product photo, name, SKU, price,
  stock badge. Filter chips by `Category` across the top, plus a search box
  matching name or SKU.
- **משימות** — `list` on Tasks, a form that `create`s one, and a checkbox per row
  that `update`s `Status` to `Done`. Fields: `Title` (text), `DueAt` (datetime,
  ISO string), `Status` (`Open` / `Done`). Sort by `DueAt` ascending, `Open`
  before `Done`, and show an overdue task in the orange series colour.
- **צ'אט** — sends `chat`, renders `[0].output`

## 2. Product images

Every product carries an `ImageUrl` — one photo per category, served from a CDN.

- Products grid: the photo fills a 16:10 tile at the top of the card,
  `object-fit: cover`, with a `linear-gradient(to top, rgba(0,0,0,.55), transparent 55%)`
  overlay so text stays readable against any photo.
- Invoices table: a 36px rounded thumbnail to the right of `ProductName`,
  matched from the product whose `Sku` equals the invoice's `Sku`.
- If `ImageUrl` is missing, render nothing — never a broken image icon,
  never a grey "no image" box.
- Every one of these images is decorative next to its own label, so `alt=""`.

## 3. Data / logic

- Dashboard tile "סך ההכנסות": sum of `record.fields.Total` for invoices whose
  `Status` is "Issued" or "Paid". Format as ₪ with thousands separators.
- Invoices table columns: מספר · לקוח · **מוצר** (thumbnail + name) · **כמות** ·
  סכום · מע"מ · סה"כ · סטטוס · מסמך.
- "מסמך" column: if `record.fields.PdfUrl` exists render a link "פתח מסמך"
  opening in a new tab, otherwise "—".
- Nothing is computed that the API already returns. `Amount`, `VatAmount` and
  `Total` are authoritative — never recalculate them in the frontend.

## 4. Hebrew

Translate status values everywhere they are DISPLAYED (tables, badges, charts).
Keep sending the English values to the API.

- Leads: New→חדש, Contacted→נוצר קשר, Replied→השיב
- Invoices: Queued→ממתינה, Issued→הופקה, Paid→שולמה, Invalid→פסולה
- Tasks: Open→פתוחה, Done→בוצעה

Column headers:
InvoiceNumber→מספר מסמך, CustomerId→לקוח, Amount→סכום, VatAmount→מע"מ,
Total→סה"כ, Status→סטטוס, Created→נוצר, Name→שם, Email→אימייל,
Company→חברה, Category→קטגוריה, Price→מחיר, InStock→במלאי,
Title→משימה, DueAt→מועד, ProductName→מוצר, Sku→מק"ט, Qty→כמות

The whole app is RTL: `<html dir="rtl" lang="he">`. Use logical CSS properties
(`margin-inline-start`, `inset-inline-end`) — never `left` / `right`.

## 5. Visual design — dark, technical, precise

TYPOGRAPHY
- Poppins for Latin, Heebo for Hebrew: `font-family: "Poppins","Heebo",sans-serif`.
  Poppins covers Latin and numerals, Heebo picks up Hebrew through fallback.
- Base 16px / line-height 1.65. Page title 30px/600 with `letter-spacing:-.035em`.
  Section title 16.5px/600. KPI numbers 36px/600, `letter-spacing:-.035em`.
- Table headers 11.5px, uppercase, `letter-spacing:.09em`, monospace, muted.
- `font-variant-numeric: tabular-nums` on every numeric column, never on the
  large standalone KPI figures.

COLOR — exactly these
```
background      #000000
surface         rgba(255,255,255,.028)
surface-2       rgba(255,255,255,.05)
hairline        rgba(255,255,255,.09)
hairline-2      rgba(255,255,255,.14)
text            #ffffff
text-secondary  #9ca3af
text-muted      #6b7280
series 1 2 3    #3987e5  #d95926  #199e70
good / warn / bad   #3ecf77  #fab219  #e66767
```

The three series colours are validated: each clears 3:1 against the black
surface, and every pair clears the colour-blindness separation floor.
**Do not substitute them and do not add a fourth** — a fourth category folds
into `#6b7280`.

LAYOUT
- Page title uses a gradient fill: `linear-gradient(to bottom,#fff,rgba(255,255,255,.62))`
  with `background-clip:text`.
- A soft blue glow behind the top of the page:
  `radial-gradient(60rem 26rem at 70% -8rem, rgba(57,135,229,.14), transparent 70%)`.
- Cards: `surface` background, 1px `hairline` border, 12px radius, 20px padding.
  No heavy shadows, no gradients on cards.
- Sidebar: transparent with a hairline border on its inline-start edge. Active
  item = `surface-2` background plus a 3px blue marker on the sidebar's edge.
- Pills / chips: fully rounded, `surface-2` background, `hairline` border,
  a 6px dot in the status colour before the label.
- Buttons: `linear-gradient(to bottom,#fff,rgba(255,255,255,.88))`, black text,
  10px radius, scale 1.03 on hover and 0.97 on press. Secondary = transparent
  with a `hairline-2` border.

TABLES
- No vertical borders. One hairline bottom border per row, none on the last.
- Row height ~48px, cell padding 13px/15px. Row hover `rgba(255,255,255,.022)`.
- Numeric columns right-aligned.

CHARTS
- Bars: single blue on a `rgba(255,255,255,.07)` track, 12px tall, 4px radius.
- Donut: thin ring, a small gap between segments, total in the centre.
- The legend shows the value as text beside every colour swatch, so meaning is
  never carried by colour alone.
- No 3D, no shadows, no rainbow palettes.

Overall feel: calm, dark, precise — an instrument panel, not a colourful SaaS
template.

## 6. Accessibility — treat these as requirements, not suggestions

- A "דלג לתוכן" skip link, first in tab order, visible on focus.
- Real landmarks: `<nav>`, `<main>`, `<section aria-labelledby>`. One `<h1>` per page,
  headings in order, no level skipped.
- Loading state `role="status"`, error state `role="alert"`.
- Tables get a `<caption>` (visually hidden is fine) and `scope="col"` on headers.
  A horizontally scrolling table is focusable: `tabindex="0"` + `role="region"`.
- Every interactive target is at least 44×44px.
- A visible `:focus-visible` ring everywhere — 2px `#7fb2f0`, 3px offset.
  Never `outline: none` without a replacement.
- Checkboxes and icon-only buttons carry an `aria-label` naming the row they act on.
- Links opening a new tab say so in visually hidden text.
- Charts: `role="img"` with an `aria-label` carrying the headline number, and the
  full breakdown available as text in the legend.
- Honour `prefers-reduced-motion` and `forced-colors`.
- Contrast: `#9ca3af` on black is 9:1, `#6b7280` is 5.3:1 — keep muted text at
  13px or larger and never put it on `surface-2` for body copy.
- Every form input has a real `<label>`, not just a placeholder.
