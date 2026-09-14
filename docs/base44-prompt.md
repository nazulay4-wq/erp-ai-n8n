# פרומפט להדבקה ב-Base44 (סבב תיקונים אחרון)

להדביק כבלוק אחד בשורת הצ'אט של Base44.

---

Apply all of the following. Keep existing functionality — this is polish only.

## 1. Data / logic fixes
- Remove the "משימות" (Tasks) page and its sidebar item entirely. That table is unused.
- Dashboard tile "סך ההכנסות" currently shows "...". Compute it in the frontend:
  sum of `record.fields.Total` for invoices where `Status` is "Issued" or "Paid".
  Format as ₪ with thousands separators.
- Add a "מסמך" column to the invoices table: if `record.fields.PdfUrl` exists render a
  link "פתח מסמך" opening it in a new tab, otherwise "—".

## 2. Hebrew
Translate status values everywhere they are DISPLAYED (tables, badges, charts).
Keep sending the English values to the API.
- Leads: New→חדש, Contacted→נוצר קשר, Replied→השיב
- Invoices: Queued→ממתינה, Issued→הופקה, Paid→שולמה, Invalid→פסולה

Translate all column headers:
InvoiceNumber→מספר מסמך, CustomerId→לקוח, Amount→סכום, VatAmount→מע"מ,
Total→סה"כ, Status→סטטוס, Created→נוצר, Name→שם, Email→אימייל,
Company→חברה, Category→קטגוריה, Price→מחיר, InStock→במלאי

## 3. Layout
Tables overflow horizontally and amount columns get cut off. Make every table fit its
container on desktop without horizontal scrolling — reduce padding, size columns to content.

## 4. Visual design
Redesign the visual language. Styling only.

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
- status badges: soft tinted background with darker text of the same hue — not solid saturated pills.
- Replace the dark navy sidebar with white + a right border. Active item = light gray
  background with a blue right-edge marker.

LAYOUT
- Cards: white, 1px #e6e4e0 border, 10px radius, shadow 0 1px 2px rgba(0,0,0,.05), 20px padding.
  No heavy shadows, no gradients.
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

Overall feel: calm, editorial, precise — a well-made financial report, not a colorful SaaS template.

---

## תזכורת API (לא לשנות)
POST `https://hna.app.n8n.cloud/webhook/33543bff-1acb-4971-9d75-517aa49f351d`
קריאה ישירה מהדפדפן, בלי backend function ובלי מפתחות.
