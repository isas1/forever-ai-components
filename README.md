# Forever AI Components

[![600+ components](https://img.shields.io/badge/components-600%2B-blueviolet?style=flat)](https://forever-components.vercel.app/infinite/)
[![Themes](https://img.shields.io/badge/themes-57-blue?style=flat)](#component-index-by-theme)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat)](LICENSE)
[![Zero deps](https://img.shields.io/badge/dependencies-zero-brightgreen?style=flat)](#)

---

## Identity

Forever AI Components is a structured design knowledge base and retrieval layer for AI coding agents.

The components are the database. The retrieval layer is the product.

The primary consumer is an AI agent building or improving a visual interface. The registry contains 600+ curated interface components, each structured with rich metadata across many facets and made retrievable by intent. The value is in the structure: every component is labelled, related, scored, and indexed so an agent can consistently retrieve the right one for a given design task, rather than generating UI blind.

Every component is a single `.html` file. Every component is a URL. Built with CSS, Canvas 2D, and SVG: zero external dependencies, no build step, no framework.

**Live canvas:** https://forever-components.vercel.app/infinite/
**Agent entry point:** https://raw.githubusercontent.com/isas1/forever-ai-components/main/agents.json
**Full index:** https://raw.githubusercontent.com/isas1/forever-ai-components/main/infinite/components.index.json

Updated continuously: see [ROADMAP.md](ROADMAP.md).

---

## Capabilities

An agent using this registry can:

- **Search**: filter 600+ components across many structured facets
- **Select**: choose the smallest useful set for a given task
- **Retrieve**: fetch complete, ready-to-embed component source via URL
- **Compose**: assemble multiple components into a coherent interface
- **Adapt**: modify retrieved components to match project style, palette, and spacing
- **Explain**: describe selection rationale and remaining work to the user

---

## Decision strategy

When solving a UI task, follow this order:

1. Understand the requested outcome: what does the interface need to accomplish?
2. Infer the interface type: single component, page section, or full layout?
3. Search the index, filtering by `useCases`, `technique`, `perfTier`, `style`, `mood`
4. Prefer existing components over generating new UI from scratch
5. Prefer `perfTier: cheap` unless the task explicitly requires heavy animation
6. Prefer `labels: accessible` and `labels: mobile-ready` by default
7. Prefer low `complexity` when the component will be adapted
8. Only generate new UI if no suitable component exists in the registry

---

## Retrieval protocol

The registry exposes three lean surfaces. Filter on the small ones, fetch the heavy one only for the components you actually pick.

### Step 1: learn the filter vocab (optional, tiny)

```
GET https://raw.githubusercontent.com/isas1/forever-ai-components/main/infinite/facets.json
```

A few KB: every facet you can filter on, with its values and counts. Fetch this first if you want to know the schema before pulling the index.

### Step 2: load the discovery index

```
GET https://raw.githubusercontent.com/isas1/forever-ai-components/main/infinite/components.index.json
```

A flat array of all 600+ components with their discovery facets (title, description, file, role, technique, perfTier, style, mood, tags, labels, …). Lean by design: the heavy adaptation prose is NOT here. Load once, filter in CODE. Do not paste the whole file into a model context.

### Step 3: filter by facet

Filter the array by any combination of structured facets before retrieving source:

```js
components
  .filter(c => c.useCases?.includes('loader'))
  .filter(c => c.perfTier === 'cheap')
  .filter(c => c.technique === 'css')
  .filter(c => c.labels?.includes('mobile-ready'))
```

### Step 4: retrieve component source (and adaptation detail)

```
GET https://raw.githubusercontent.com/isas1/forever-ai-components/main/infinite/{file}
```

The response is the complete component: self-contained HTML with inline CSS and JS. No dependencies to resolve. No build step required.

The component's adaptation detail travels INSIDE this file, as an inert JSON block:

```html
<script type="application/json" id="forever-agent-meta">
{ "useWhen": …, "avoidWhen": …, "agentPrompt": …, "editableAreas": …, "preserveElements": …, "knownLimitations": … }
</script>
```

Parse that block for guidance on how to safely modify the component. One fetch returns code + instructions together, so this prose never bloats the discovery index.

### Compact manifest

```
GET https://raw.githubusercontent.com/isas1/forever-ai-components/main/agents.json
```

Returns: registry name, component count, index URL, fetch URL pattern, and full facet schema. Use as the agent entry point before loading the full index.

---

## Composition rules

Retrieve solutions, not individual components. When the task is a page or layout, retrieve a coordinated set:

| Task | Retrieve |
|---|---|
| Creative portfolio | backgrounds + cards + text effects + navigation |
| Artist showcase | artist-inspired hero + kinetic typography + organic transitions |
| Generative art page | generative geometry + particle systems + cosmic backgrounds |
| Motion-heavy hero | kinetic type + CSS 3D + particle effects + loaders |
| Editorial layout | swiss typography + micro-dataviz + minimal navigation |
| Interactive experience | cursor effects + UI micro-interactions + feedback states |

Choose the smallest set that solves the task. Avoid retrieving components that will not be used.

---

## Adaptation rules

When adapting a retrieved component:

- Inherit the project's colour palette: replace hardcoded hex values
- Inherit the project's spacing scale: replace hardcoded px values where practical
- Inherit the project's typography: replace font families if Google Fonts are used
- Preserve `prefers-reduced-motion` handling: do not remove it
- Preserve `document.hidden` pause logic: do not remove it
- Preserve CSS/JS namespace prefixes: do not globalise component styles
- Preserve semantic HTML structure
- Minimise additional dependencies introduced during adaptation

---

## Component schema

`components.index.json` (schemaVersion 3) is the lean discovery surface. Each record carries the Identity and Discovery facets below. The Agentic adaptation fields are NOT in the index: they travel embedded in each tile (see the `forever-agent-meta` block in Retrieval step 4).

**Identity**

| Field | Type | Description |
|---|---|---|
| `id` | string | stable unique component id |
| `file` | string | path relative to registry root: use as fetch URL suffix |
| `repoFile` | string | path within the source repository |
| `themeSlug` | string | machine slug for the theme (e.g. `kinetic-type`) |
| `theme` | string | thematic family (e.g. "Kinetic Typography") |
| `title` | string | component name |
| `loc` | number | lines of code |
| `kb` | number | file size in kilobytes |

**Discovery facets**

| Field | Type | Description |
|---|---|---|
| `role` | string | primary UI role (e.g. `hero`, `loader`, `background`) |
| `technique` | enum | `css` `canvas` `svg` `css+canvas` |
| `style` | string | visual style descriptor |
| `motion` | enum | `none` `subtle` `moderate` `intense` |
| `intensity` | enum | relative animation intensity |
| `interaction` | enum | `passive` `hover` `click` `pointer` |
| `a11y` | string[] | `reduced-motion` `keyboard` `screen-reader` |
| `perfTier` | enum | `cheap` `moderate` `expensive` |
| `palette` | string[] | dominant colours (hex) |
| `deps` | string[] | external dependencies (normally empty) |
| `useCases` | string[] | UI roles this component serves |
| `mood` | string[] | emotional register: `calm` `energetic` `elegant` `playful` … |
| `description` | string | one-line summary of the component |
| `aiEnriched` | boolean | whether metadata was enriched by an AI pass |
| `tier` | enum | quality tier (e.g. `ready`, `needs-review`) |
| `labels` | string[] | `agent-ready` `easy-to-modify` `mobile-ready` `accessible` `copy-paste-safe` |
| `tags` | string[] | freeform semantic keywords |

**Agentic adaptation** (embedded in each tile, not in the index)

Found in the tile's `<script type="application/json" id="forever-agent-meta">` block. These tell an agent how to safely reuse and modify a component.

| Field | Type | Description |
|---|---|---|
| `useWhen` | string[] | situations this component is a good fit for |
| `avoidWhen` | string[] | situations to avoid this component in |
| `agentPrompt` | string | an example adaptation instruction for an agent |
| `editableAreas` | string[] | regions that are safe to change |
| `preserveElements` | string[] | code that must be kept (e.g. reduced-motion, namespacing) |
| `knownLimitations` | string[] | caveats and known constraints |

---

## Output contract

When returning components to a user, include:

1. **Reason**: why this component was selected over alternatives
2. **Selected components**: file paths and titles
3. **Adaptations made**: what was changed from the source and why
4. **Remaining work**: what the component does not yet cover
5. **Recommended next**: what to retrieve or build next to complete the interface

---

## Quality gates

Do not retrieve or recommend a component that:

- Has `tier: needs-review` unless no alternative exists
- Has `perfTier: expensive` for a mobile-primary context
- Has `interaction: pointer` for a touch-only context without a fallback

---

## Registry structure

```
forever-ai-components/
├── catalog.json                    # source of truth: full structured metadata
├── agents.json                     # compact agent entry point
├── llms.txt                        # plain-text agent protocol (llms.txt convention)
├── ROADMAP.md                      # what's being built next
│
└── infinite/
    ├── facets.json                 # filter vocab: facets + values + counts (tiny)
    ├── components.index.json       # lean discovery index (flat array, filter in code)
    └── <theme>/
        └── NN-<kebab-name>.html    # self-contained component + embedded adaptation detail
```

---

## Component index by theme

600+ components across 57 themes:

### UI Components

| Theme | Count |
|---|---|
| Buttons & Toggles | 11 |
| Cards | 11 |
| Loaders | 16 |
| Navigation | 10 |
| Backgrounds | 10 |
| Inputs & Switches | 19 |
| Cursor & Pointer | 6 |
| Progress & Status | 12 |
| Feedback & States | 12 |
| Drag & Reorder | 10 |

### Animation & Motion

| Theme | Count |
|---|---|
| Kinetic Typography | 20 |
| Micro Data-Viz | 20 |
| CSS 3D & Perspective | 20 |
| Particles & Physics | 20 |
| Organic & Nature | 18 |
| Generative Geometry | 20 |
| UI Micro-interactions | 21 |
| Retro & Glitch | 20 |
| Cosmic & Space | 20 |
| Text Effects | 6 |
| SVG Animations | 8 |
| Audio Visualizers | 8 |
| Charts & Flow | 10 |
| Fractals & Automata | 10 |

### Generative & Geometric Styles

| Theme | Count |
|---|---|
| Op Art & Moire | 12 |
| Swiss Typographic | 10 |
| De Stijl Neoplastic | 8 |
| Neo-Brutalism | 6 |
| Neumorphism | 6 |
| Terminal | 6 |
| Blueprint Technical | 1 |
| Wabi-Sabi | 1 |

### Design Movement Revival

| Theme | Count |
|---|---|
| Bauhaus Geometric | 12 |
| Art Deco | 12 |
| Art Nouveau | 12 |
| Memphis 80s | 10 |
| Psychedelic | 10 |
| Vaporwave Y2K | 10 |
| Constructivist | 10 |
| Pointillist | 10 |
| Stained Glass | 2 |
| Clockwork & Mechanical | 1 |

### Artist-Inspired

| Theme | Count |
|---|---|
| Post-Impressionist Night Studies | 8 |
| Abstract Geometric Improvisations | 10 |
| Orphist Colour Discs | 8 |
| Playful Geometric Notation | 8 |
| Symbolic Pastel Abstraction | 8 |
| Gold-Leaf Ornamental Studies | 8 |
| Ukiyo-e Wave & Mountain | 9 |
| Ukiyo-e Weather & Road | 8 |
| Pointillist Dot-Field Studies | 8 |
| Art Nouveau Ornamental Panels | 8 |
| Primary Grid Compositions | 8 |
| Suprematist Geometric Forms | 8 |
| Arts & Crafts Pattern Repeats | 8 |
| Ukiyo-e Woodblock | 10 |
| Infinitude | 8 |


---

## License

MIT. Some components are inspired by public-domain artwork: see [`NOTICE.md`](NOTICE.md).