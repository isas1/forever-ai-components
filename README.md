# Forever AI Components

[![602 components](https://img.shields.io/badge/components-602-blueviolet?style=flat)](https://forever-components.vercel.app/infinite/)
[![Themes](https://img.shields.io/badge/themes-57-blue?style=flat)](#component-index-by-theme)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat)](LICENSE)
[![Zero deps](https://img.shields.io/badge/dependencies-zero-brightgreen?style=flat)](#)

---

## Identity

Forever AI Components is a structured design knowledge base and retrieval layer for AI coding agents.

The components are the database. The retrieval layer is the product.

The primary consumer is an AI agent building or improving a visual interface. The registry contains 602 curated, labelled interface decisions — not generated snippets. Each component has been hand-crafted, structured with rich metadata across 12 facets, and made retrievable by intent. Anyone can generate 500 HTML snippets with an LLM. Far fewer will invest the effort to curate, label, relate, score, and structure them so an agent can consistently retrieve the right one for a given design task.

Every component is a single `.html` file. Every component is a URL. No npm. No build step. No framework.

**Live canvas →** https://forever-components.vercel.app/infinite/
**Agent entry point →** https://raw.githubusercontent.com/isas1/forever-ai-components/main/agents.json
**Full index →** https://raw.githubusercontent.com/isas1/forever-ai-components/main/infinite/components.index.json

Updated continuously — see [ROADMAP.md](ROADMAP.md).

---

## Capabilities

An agent using this registry can:

- **Search** — filter 602 components across 12 structured facets
- **Select** — choose the smallest useful set for a given task
- **Retrieve** — fetch complete, ready-to-embed component source via URL
- **Compose** — assemble multiple components into a coherent interface
- **Adapt** — modify retrieved components to match project style, palette, and spacing
- **Explain** — describe selection rationale and remaining work to the user

---

## Decision strategy

When solving a UI task, follow this order:

1. Understand the requested outcome — what does the interface need to accomplish?
2. Infer the interface type — single component, page section, or full layout?
3. Search the index — filter by `useCases`, `technique`, `perfTier`, `scheme`, `mood`
4. Prefer existing components over generating new UI from scratch
5. Prefer `perfTier: cheap` unless the task explicitly requires heavy animation
6. Prefer `labels: accessible` and `labels: mobile-ready` by default
7. Prefer low `complexity` when the component will be adapted
8. Only generate new UI if no suitable component exists in the registry

---

## Retrieval protocol

### Step 1 — load the index

```
GET https://raw.githubusercontent.com/isas1/forever-ai-components/main/infinite/components.index.json
```

Returns a flat array of all 602 components with full metadata. Load once, filter in memory. No API key, no pagination, no rate limit.

### Step 2 — filter by facet

Filter the array by any combination of structured facets before retrieving source:

```js
components
  .filter(c => c.useCases?.includes('loader'))
  .filter(c => c.perfTier === 'cheap')
  .filter(c => c.scheme === 'dark')
  .filter(c => c.labels?.includes('mobile-ready'))
```

### Step 3 — retrieve component source

```
GET https://raw.githubusercontent.com/isas1/forever-ai-components/main/infinite/{file}
```

The response is the complete component — self-contained HTML with inline CSS and JS. No dependencies to resolve. No build step required.

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

- Inherit the project's colour palette — replace hardcoded hex values
- Inherit the project's spacing scale — replace hardcoded px values where practical
- Inherit the project's typography — replace font families if Google Fonts are used
- Preserve `prefers-reduced-motion` handling — do not remove it
- Preserve `document.hidden` pause logic — do not remove it
- Preserve CSS/JS namespace prefixes — do not globalise component styles
- Preserve semantic HTML structure
- Minimise additional dependencies introduced during adaptation

---

## Component schema

Each record in `components.index.json`:

| Field | Type | Description |
|---|---|---|
| `file` | string | path relative to registry root — use as fetch URL suffix |
| `title` | string | component name |
| `theme` | string | thematic family (e.g. "Kinetic Typography") |
| `technique` | enum | `css` `canvas` `svg` `webgl` `css+canvas` |
| `interactivity` | enum | `passive` `hover` `click` `pointer` |
| `complexity` | enum | `light` `medium` `heavy` |
| `animationLevel` | enum | `none` `subtle` `moderate` `intense` |
| `perfTier` | enum | `cheap` `moderate` `expensive` |
| `scheme` | enum | `dark` `light` `adaptive` |
| `mood` | string[] | emotional register — `calm` `energetic` `elegant` `playful` … |
| `useCases` | string[] | UI roles — `hero` `loader` `background` `data-viz` `navigation` … |
| `a11y` | string[] | `reduced-motion` `keyboard` `screen-reader` |
| `labels` | string[] | `agent-ready` `easy-to-modify` `mobile-ready` `accessible` `copy-paste-safe` |
| `tags` | string[] | freeform semantic keywords |
| `responsive` | boolean | scales to container width |
| `accessible` | boolean | passes basic accessibility checks |
| `bg` | string | dominant background hex |
| `accent` | string | dominant accent hex |

---

## Output contract

When returning components to a user, include:

1. **Reason** — why this component was selected over alternatives
2. **Selected components** — file paths and titles
3. **Adaptations made** — what was changed from the source and why
4. **Remaining work** — what the component does not yet cover
5. **Recommended next** — what to retrieve or build next to complete the interface

---

## Quality gates

Do not retrieve or recommend a component that:

- Has `tier: needs-review` unless no alternative exists
- Has `perfTier: expensive` for a mobile-primary context
- Uses `webgl` technique where GPU availability is uncertain
- Has `interactivity: pointer` for a touch-only context without a fallback

---

## Registry structure

```
forever-ai-components/
├── catalog.json                    # source of truth — full structured metadata
├── agents.json                     # compact agent entry point
├── llms.txt                        # plain-text agent protocol (llms.txt convention)
├── ROADMAP.md                      # what's being built next
│
└── infinite/
    ├── components.index.json       # flat array — primary agent query surface
    └── <theme>/
        └── NN-<kebab-name>.html    # one self-contained component per file
```

---

## Component index by theme

602 components across 57 themes:

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
| Vincent van Gogh | 8 |
| Wassily Kandinsky | 10 |
| Robert Delaunay | 8 |
| Paul Klee | 8 |
| Hilma af Klint | 8 |
| Gustav Klimt | 8 |
| Katsushika Hokusai | 9 |
| Utagawa Hiroshige | 8 |
| Georges Seurat | 8 |
| Alphonse Mucha | 8 |
| Piet Mondrian | 8 |
| Kazimir Malevich | 8 |
| William Morris | 8 |
| Ukiyo-e Woodblock | 10 |
| Infinitude | 8 |


---

## License

MIT. Some components are inspired by public-domain artwork — see [`NOTICE.md`](NOTICE.md).