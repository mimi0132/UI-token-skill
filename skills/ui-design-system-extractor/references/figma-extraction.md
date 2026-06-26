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
| `fills[0].color`              | `colors.css`   | `--color-primary-500`               |
| `cornerRadius`                | `radius.css`   | `--radius-md` (6)                   |
| `paddingLeft/Right`           | `spacing.css`  | `--space-4` (16)                    |
| `paddingTop/Bottom`           | `spacing.css`  | `--space-2` (8)                     |
| `itemSpacing`                 | `spacing.css`  | `--space-2` (8)                     |
| `fontSize`                    | `typography.css` | `--font-size-md` (16)             |
| `fontWeight`                  | `typography.css` | `--font-weight-medium` (500)      |
| `fontName.family`             | `typography.css` | `--font-family-body`              |
| `effects[0]` (DROP_SHADOW)    | `theme.css`    | `--shadow-sm`                       |

### 6. Derive color scales from a single primary

If Figma only gives you the 500/600 stops, derive the rest of the scale using
`color-mix(in srgb, ...)`:

```css
:root {
  --color-primary-500: #6366F1;
  --color-primary-50:  color-mix(in srgb, var(--color-primary-500) 8%,  white);
  --color-primary-100: color-mix(in srgb, var(--color-primary-500) 16%, white);
  --color-primary-200: color-mix(in srgb, var(--color-primary-500) 30%, white);
  --color-primary-300: color-mix(in srgb, var(--color-primary-500) 50%, white);
  --color-primary-400: color-mix(in srgb, var(--color-primary-500) 75%, white);
  --color-primary-600: color-mix(in srgb, var(--color-primary-500) 87%, black);
  --color-primary-700: color-mix(in srgb, var(--color-primary-500) 75%, black);
  --color-primary-800: color-mix(in srgb, var(--color-primary-500) 60%, black);
  --color-primary-900: color-mix(in srgb, var(--color-primary-500) 45%, black);
}
```

Same approach for the dark mode: invert the white/black arguments.

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
   store in a `gradient.css` file (add to the 5 token files as a 6th file if needed).

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
