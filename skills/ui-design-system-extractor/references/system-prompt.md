# System Prompt Reference

This file is the "design philosophy" that the agent should keep in mind while extracting
tokens and writing components. Load it when the user asks "why this token?" or "is this
choice correct?".

---

## Design Analysis Process

### Step 1: Design Token Extraction

From the source, extract these visual features with **concrete values**, never vague
descriptions:

| Token Type    | Extract                                                        | Example                                  |
|---------------|----------------------------------------------------------------|------------------------------------------|
| **Colors**    | Primary, background, text, border, semantic (success/warning/danger/info) | Primary: `#6366F1` / Bg: `#FFFFFF`     |
| **Radius**    | sm / md / lg / full in concrete px                            | Button: `6px` / Card: `12px` / Input: `8px` |
| **Shadow**    | Precise `box-shadow` values (light/medium/heavy/overlay)      | `0 1px 3px rgba(0,0,0,0.1), 0 1px 2px rgba(0,0,0,0.06)` |
| **Spacing**   | Component padding, element gap                                | Button padding: `10px 16px` / Gap: `8px` |
| **Typography**| Size, weight, family, line-height                             | Body: `14px/500` / Heading: `16px/600`   |
| **Glass**     | `backdrop-blur` value, opacity                                 | `backdrop-blur(12px)` / `bg-white/80`    |
| **Motion**    | Transition duration, easing                                    | `transition: all 200ms cubic-bezier(0.4, 0, 0.2, 1)` |

### Step 2: Component Inference

A complete component library is **more than** what's visible in the design. Common
inference rules:

| Visible in Design       | Must Generate                                             |
|-------------------------|-----------------------------------------------------------|
| Any button              | Button (all variants) + IconButton                        |
| Input field             | Input (all states) + Textarea                             |
| Card container          | Card (all variants) + Paper                               |
| Tag / Badge             | Badge (all variants)                                      |
| User avatar             | Avatar (all variants) + AvatarGroup                       |
| Divider                 | Divider                                                   |
| List item               | List, ListItem                                            |
| Modal / Dialog          | Modal (all variants)                                      |
| Tooltip                 | Tooltip (all placements)                                  |
| Navigation              | Navbar, Tabs                                              |
| Form                    | Checkbox, Switch, Radio, Select                           |
| Table                   | Table (pagination + sort + filter)                        |
| Loading                 | Spinner, Skeleton                                         |
| Empty state             | Empty (all variants)                                      |

### Step 3: File Output

Each component goes into its own file, marked with delimiters for parsing:

```
<!-- FILE_START: Button.vue -->
<!-- FILE_END: Button.vue -->
```

**NEVER** wrap in markdown code fences (` ``` `).

### Step 4: Browser Preview

Components go into `src/components/ui/{name}.vue` or `.tsx`, plus a `preview.html` that
renders:

1. **Color palette** (colors.css) — swatches with HEX + click-to-copy
2. **Typography spec** (typography.css) — sample text per size/weight
3. **Spacing & radius** (spacing.css + radius.css) — visual bars + rounded rects
4. **Grid system** (theme.css) — 12-col overlay, 5 breakpoints
5. **Component list** — one card per component, each card embeds its `<DEMO>` block

---

## Design Principles

1. **Token-first, never hard-code.** Every color, size, gap, radius in a component file
   must reference a CSS variable from the 5 token files. This is what makes the library
   overridable.

2. **DEMO block per component.** A component without a demo block showing all its
   variants is incomplete. The preview page parses these blocks; they're also the
   spec for what the component must support.

3. **Match the base library's API.** If the user wants Element Plus override, generate
   `<el-button>`-shaped components, not bespoke `<MyButton>`. Override, don't replace.

4. **Semantic naming over visual naming.** Use `--color-primary-500`, not `--blue-500`.
   Use `--button-bg`, not `--button-blue-bg`. The rename happens at the consumer side.

5. **Dark mode is mandatory.** Every token must have a `[data-theme="dark"]` variant. Use
   `color-mix(in srgb, var(--color-*) var(--alpha), transparent)` for tints/shades
   instead of hand-rolled HEX variations.

6. **No silent framework choice.** Always ask at Step 0. Default if user is unclear:
   Vue 3 + Element Plus (most common), then React + shadcn (most common).

---

## Quality Gate (run before declaring done)

```bash
# 1. No hard-coded hex in components
! grep -rE "#[0-9a-fA-F]{3,8}" components/ tokens/

# 2. No hard-coded px in components
! grep -rE "[0-9]+px" components/

# 3. Every component has a DEMO block
for f in components/*.{vue,tsx}; do
  grep -q "DEMO_START" "$f" || echo "MISSING DEMO: $f"
done
```

If any check fails, fix before outputting.
