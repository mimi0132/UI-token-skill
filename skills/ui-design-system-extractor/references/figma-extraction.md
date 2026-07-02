# Figma Extraction Reference

This file documents how to use the Figma AI Bridge MCP to extract design data. Load when
the user provides a Figma link.

---

## MCP Tools Available

| Tool                          | Purpose                                              |
|-------------------------------|------------------------------------------------------|
| `mcp_Figma_AI_Bridge.get_figma_data` | Fetch node data from a Figma URL/file key     |
| `mcp_Figma_AI_Bridge.download_figma_images` | Download exported images from a Figma URL |

---

## When to Use

- User provides a Figma URL (`figma.com/(file|design)/...`)
- User provides a Figma file key (a 22-char alphanumeric string)
- User provides a specific node URL (with `node-id=...` query param)

If the user gives a local image (PNG / JPG), you cannot extract precise Figma metadata.
In that case, either:

1. Ask the user to upload the image to Figma and provide a Figma URL, OR
2. Ask the user to describe the key visual values (primary color, font family, etc.)
3. Use visual inference (the agent's own image understanding) — but this is less reliable

---

## Step-by-Step: Extracting from a Figma URL

### 1. Parse the URL

Given: `https://www.figma.com/design/abc123/MyApp?node-id=12-34`

Extract:
- `file_key` = `abc123` (the segment after `/design/` or `/file/`)
- `node_id`  = `12:34` (the value of `node-id` query param, dashes → colons)

### 2. Call `get_figma_data`

```json
{
  "fileKey": "abc123",
  "nodeId": "12:34"
}
```

### 3. Interpret the response

The response typically contains:

```jsonc
{
  "document": {
    "id": "12:34",
    "name": "Button / Primary",
    "type": "COMPONENT",
    "children": [/* ... */],
    "fills": [
      { "type": "SOLID", "color": { "r": 0.388, "g": 0.4, "b": 0.949, "a": 1 } }
    ],
    "strokes": [
      { "type": "SOLID", "color": { "r": 0, "g": 0, "b": 0, "a": 0.1 } }
    ],
    "cornerRadius": 6,
    "paddingLeft": 16, "paddingRight": 16,
    "paddingTop": 10,  "paddingBottom": 10,
    "itemSpacing": 8,
    "fontSize": 14,
    "fontWeight": 500,
    "fontName": { "family": "Inter", "style": "Medium" },
    "lineHeight": { "value": 20, "unit": "PIXELS" },
    "letterSpacing": { "value": 0, "unit": "PIXELS" },
    "effects": [
      { "type": "DROP_SHADOW", "color": { "r": 0, "g": 0, "b": 0, "a": 0.1 },
        "offset": { "x": 0, "y": 1 }, "radius": 2, "spread": 0 }
    ]
  }
}
```

### 4. Convert Figma RGBA to HEX

Figma colors are 0-1 normalized. Convert to HEX:

```js
function figmaToHex({ r, g, b, a = 1 }) {
  const to255 = v => Math.round(v * 255).toString(16).padStart(2, '0')
  const hex = `#${to255(r)}${to255(g)}${to255(b)}`
  if (a < 1) {
    return `${hex}${to255(a)}`
  }
  return hex
}
```

Example: `{ r: 0.388, g: 0.4, b: 0.949, a: 1 }` → `#6366F1`

### 5. Extract design tokens

Map Figma properties to token file entries:

| Figma Property                | Token File     | Variable Name                       |
|-------------------------------|----------------|-------------------------------------|
| `fills[0].color`              | `colors.css`   | `--color-primary` (EP / 中创)       |
| `fills[0].color`              | `colors.css`   | `--color-primary-6` (AntD 10-stop)  |
| `cornerRadius`                | `radius.css`   | `--radius-md` (6)                   |
| `paddingLeft/Right`           | `spacing.css`  | `--space-4` (16)                    |
| `paddingTop/Bottom`           | `spacing.css`  | `--space-2` (8)                     |
| `itemSpacing`                 | `spacing.css`  | `--space-2` (8)                     |
| `fontSize`                    | `typography.css` | `--font-size-md` (16)             |
| `fontWeight`                  | `typography.css` | `--font-weight-medium` (500)      |
| `fontName.family`             | `typography.css` | `--font-family-body`              |
| `effects[0]` (DROP_SHADOW)    | `theme.css`    | `--shadow-sm`                       |

### 6. Derive color scales from the single primary anchor

Use `color-mix(in srgb, ...)` to derive the rest of the lib's scale from the anchor:

**EP / 中创 (7 stops)** — anchor is single color, no numbering:

```css
:root {
  --color-primary: #6366F1;
  --color-primary-light-3: color-mix(in srgb, var(--color-primary) 88%, black);
  --color-primary-light-5: color-mix(in srgb, var(--color-primary) 40%, white);
  --color-primary-light-7: color-mix(in srgb, var(--color-primary) 24%, white);
  --color-primary-light-8: color-mix(in srgb, var(--color-primary) 12%, white);
  --color-primary-light-9: color-mix(in srgb, var(--color-primary)  6%, white);
  --color-primary-dark-2:  color-mix(in srgb, var(--color-primary) 80%, black);
}
```

**Ant Design v5 (10 stops)** — anchor is `--color-primary-6`, the rest is numbered:

```css
:root {
  --color-primary-6: #6366F1;  /* anchor (was -500 in v4) */
  --color-primary-1:  color-mix(in srgb, var(--color-primary-6)  8%, white);
  --color-primary-2:  color-mix(in srgb, var(--color-primary-6) 16%, white);
  --color-primary-3:  color-mix(in srgb, var(--color-primary-6) 28%, white);
  --color-primary-4:  color-mix(in srgb, var(--color-primary-6) 44%, white);
  --color-primary-5:  color-mix(in srgb, var(--color-primary-6) 68%, white);
  --color-primary-7:  color-mix(in srgb, var(--color-primary-6) 88%, black);
  --color-primary-8:  color-mix(in srgb, var(--color-primary-6) 76%, black);
  --color-primary-9:  color-mix(in srgb, var(--color-primary-6) 60%, black);
  --color-primary-10: color-mix(in srgb, var(--color-primary-6) 44%, black);
}
```

> **Do not** emit a 10-stop scale for EP / 中创, and do not emit a 7-stop scale
> for AntD. Match the lib's own structure (see `references/token-spec.md`).

---

## Image Download (`download_figma_images`)

If the user wants to extract more details visually (e.g. shadows on a complex component):

```json
{
  "fileKey": "abc123",
  "nodeId": "12:34",
  "format": "png",
  "scale": 2
}
```

Use the downloaded image as visual reference in the agent's image analysis.

---

## Common Pitfalls

1. **Don't trust Figma's default "Auto" layout properties** — they often map to 0 or
   undefined. Always check `paddingLeft/Right/Top/Bottom` and `itemSpacing` directly.

2. **Text in Figma may have "Auto" line height** (`{ unit: "AUTO" }`). Compute it as
   `fontSize * 1.2` for headings or `fontSize * 1.5` for body text.

3. **`opacity` is not `alpha`** — Figma uses a separate `opacity` field. Multiply
   `color.a * node.opacity` to get the effective alpha.

4. **Gradient fills need a separate token** — convert to a `linear-gradient` string and
   store in a `gradient.css` file (add to the 7 token files as an 8th file if needed).

5. **Variables / styles in Figma are referenced, not inlined** — the response may
   contain `boundVariables` with style IDs. Resolve these via the variables API before
   generating tokens.

---

## Fallback: When MCP Is Unavailable

If `mcp_Figma_AI_Bridge.get_figma_data` is not available or returns an error:

1. Ask the user to export the Figma frames as a ZIP and provide the path.
2. Use the `download_figma_images` tool with a higher scale (3x or 4x) to get visual
   references.
3. Read the user's description of the design (colors, fonts, etc.) and generate tokens
   from that.
4. Offer the user a "designer's-eye" mode: agent visually infers from screenshot and
   outputs best-guess tokens marked as `/* estimated */` for the user to refine.
