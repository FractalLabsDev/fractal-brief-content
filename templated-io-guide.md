# Templated.io Complete Guide

*For programmatic design generation at Fractal Labs*

---

## What Templated.io Does

Templated.io is an API-first design automation platform. You create or import templates with editable layers, then generate variations programmatically by swapping text, colors, and images.

**Pricing:** 50 free trial renders → $29/month for 1,000 renders (~$0.029/image)

---

## What Ruk Can Do (Via API)

### 1. Create Templates from Scratch

Ruk defines templates as JSON with layers. Example:

```json
{
  "name": "Question Card",
  "width": 420,
  "height": 720,
  "layers": [
    {"layer": "background", "type": "shape", "fill": "#FDF8F3", ...},
    {"layer": "question", "type": "text", "text": "Your question here?", ...},
    {"layer": "accent", "type": "shape", "fill": "#C67B5C", ...}
  ]
}
```

**Workflow:** Describe what you want → Ruk translates to layer spec → template created via API.

### 2. Render Images with Modifications

Once a template exists, Ruk can generate variations by overriding any layer property:

```bash
echo '{"question": {"text": "New question?"}, "accent": {"fill": "#3D5A80"}}' \
  | node TOOLS/templated/render.js <templateId>
```

Output: PNG, JPG, PDF, or MP4 (for animated templates).

### 3. List Templates and Gallery

- `node TOOLS/templated/list-templates.js` — your templates
- `node TOOLS/templated/list-gallery.js --query "quote"` — browse 100+ public templates

---

## What Ruk Cannot Do

### Import from Canva

**This requires a human in a browser.** Ruk cannot access the Templated.io web interface.

---

## How to Import from Canva (Human Steps)

### Step 1: In Canva
1. Open your design
2. If multi-page, close other pages (import is single-page only)
3. Click **Share** → **Anyone with the link** → Copy link

### Step 2: In Templated.io
1. Go to https://app.templated.io
2. Click **Create Template** → **Import from Canva**
3. Paste the Canva share link
4. Wait for import (preserves all layers: text, images, shapes)

### Step 3: Note the Template ID
After import, the template appears in your dashboard. Get its ID from the URL or run:
```bash
node TOOLS/templated/list-templates.js
```

### Step 4: Ruk Takes Over
Now Ruk can render variations programmatically:
```bash
echo '{"headline": {"text": "New headline"}}' \
  | node TOOLS/templated/render.js <templateId>
```

---

## Layer Types Ruk Can Create/Modify

### Text
```json
{
  "layer": "headline",
  "type": "text",
  "text": "Hello World",
  "fontFamily": "Playfair Display",
  "fontSize": 32,
  "fontWeight": "bold",
  "color": "#000000",
  "textAlign": "center"
}
```

### Shape (rectangles, lines, accents)
```json
{
  "layer": "accent-bar",
  "type": "shape",
  "fill": "#C67B5C",
  "x": 40, "y": 580, "width": 340, "height": 4
}
```

### Image
```json
{
  "layer": "photo",
  "type": "image",
  "image_url": "https://example.com/photo.jpg"
}
```

---

## Existing Template: Dina's Connection Cards

**Template ID:** `9fcc1640-2d59-48dd-9a4b-4caa3c0ac893`

**Layers:**
- `question-text` — the Russian question
- `category-text` — category label (РЕФЛЕКСИЯ, СВЯЗЬ, etc.)
- `accent-line` — colored bar at bottom

**Category Color Palette:**
| Category | Russian | Color |
|----------|---------|-------|
| Reflection | Рефлексия | `#C67B5C` (terracotta) |
| Connection | Связь | `#3D5A80` (indigo) |
| Dreams | Мечты | `#C5A572` (gold) |
| Inspiration | Вдохновение | `#E07A5F` (coral) |
| Transformation | Трансформация | `#722F37` (burgundy) |

**To render a card:**
```bash
echo '{
  "question-text": {"text": "Что бы ты создал(а), если бы знал(а), что не можешь потерпеть неудачу?"},
  "category-text": {"text": "МЕЧТЫ"},
  "accent-line": {"fill": "#C5A572"}
}' | node TOOLS/templated/render.js 9fcc1640-2d59-48dd-9a4b-4caa3c0ac893
```

---

## Workflow Summary

| Task | Who Does It |
|------|-------------|
| Design template in Canva | Human (Dina) |
| Get Canva share link | Human |
| Import into Templated.io | Human (one-time per template) |
| Generate all variations | Ruk (automated via API) |
| Create templates from scratch (no Canva) | Ruk (describe → JSON → API) |

---

## API Reference

- Docs: https://templated.io/docs/
- Create render: `POST https://api.templated.io/v1/render`
- List templates: `GET https://api.templated.io/v1/templates`
- Create template: `POST https://api.templated.io/v1/template`
- Gallery: `GET https://api.templated.io/v1/templates/gallery`

---

## Ruk's Tools

Location: `TOOLS/templated/`

| Tool | Purpose |
|------|---------|
| `list-templates.js` | List your templates |
| `list-gallery.js` | Browse public gallery |
| `create-template.js` | Create template from JSON |
| `render.js` | Generate image from template |
| `lib.js` | Shared API client |

---

## Trial Status

- **50 free renders** (used 5 so far)
- **$29/month** for 1,000 renders after trial
