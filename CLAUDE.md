# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **RTL fork** of Shopify's Horizon theme (v3.2.1). The upstream is `https://github.com/Shopify/horizon.git`. RTL customizations are layered on top of regular upstream updates.

Core principles: server-rendered Liquid, progressive enhancement (works without JS), zero external dependencies, native browser APIs only, WCAG AA accessibility.

## Development Commands

```bash
shopify theme dev          # Local development server
shopify theme check        # Lint Liquid, JSON, and schema files
```

No build step, no package.json. Files in `assets/`, `sections/`, `blocks/`, `snippets/` are served directly.

To sync with upstream:
```bash
git remote add upstream https://github.com/Shopify/horizon.git
git pull upstream main
```

## RTL Implementation

RTL is the key customization in this fork, implemented in three layers:

1. **Layout detection** (`layout/theme.liquid` lines 3-8): checks `request.locale.iso_code` against `ar,he,fa,ur`, sets `dir="rtl"` on `<html>`
2. **CSS logical properties** throughout `base.css`: `padding-inline`, `margin-block`, `text-align: start/end`, `inset-inline` — these handle most RTL automatically
3. **Explicit overrides** (`assets/rtl-overrides.css`): loaded conditionally for RTL locales, handles cases logical properties can't solve:
   - Directional icon mirroring (`scaleX(-1)` on arrows, chevrons, carets, pagination)
   - Drawer slide directions (menu from right, cart from left)
   - Slider/slideshow navigation reversal
   - Float reversals, input prefix/suffix borders

Arabic font (Almarai, 4 weights) is loaded via `snippets/font-arabic.liquid`.

**When editing CSS:** always use logical properties (`padding-inline`, `margin-block`, `inset-inline-start`) instead of physical ones (`padding-left`, `margin-top`, `left`). Only add to `rtl-overrides.css` when logical properties are insufficient (directional icons, animations with explicit transform values).

## JavaScript Architecture

### Import Maps (no bundler)

Native ES module import maps defined in `snippets/scripts.liquid`. All JS uses `@theme/` prefixed imports:

```js
import { Component } from '@theme/component';
import { VariantSelectedEvent } from '@theme/events';
```

### Component Framework (`assets/component.js`)

All interactive elements are Web Components extending `Component`:

- **Refs**: `ref="myElement"` on HTML → `this.refs.myElement`. Array refs: `ref="items[]"` → `this.refs.items`
- **Required refs**: `requiredRefs = ['button']` throws `MissingRefError` if missing
- **Declarative events**: `on:click="/methodName"` on HTML elements (global delegation). Supports: click, change, select, focus, blur, submit, input, keydown, keyup, toggle, pointerenter, pointerleave
- **Event routing**: `on:click="selector/method"` routes to closest matching component
- **Data passing**: `on:click="/method?key=value"` or `on:click="/method/data"`
- **Section re-rendering**: `updatedCallback()` fires when Section Rendering API replaces HTML

### Event System (`assets/events.js`)

Custom events for cross-component communication: `variant:selected`, `variant:update`, `cart:update`, `cart:error`, `quantity-selector:update`, `filter:update`, `media:started-playing`, `slideshow:select`.

### Global Theme Object

Defined in `snippets/scripts.liquid` — provides `Theme.translations`, `Theme.routes` (cart, search URLs), and `Theme.template`.

### Type Checking

`assets/jsconfig.json` enables `checkJs`, `strictNullChecks`, `noImplicitAny`. Path alias `@theme/*` maps to `assets/*`. Types in `assets/global.d.ts`.

## CSS Architecture

- Single main stylesheet: `assets/base.css` (~4000 lines)
- BEM naming at `0 1 0` specificity (single class selectors)
- CSS nesting (native, no preprocessor)
- Color scheme system: `snippets/color-schemes.liquid` generates `.color-{id}` classes with CSS variables (`--color-background`, `--color-foreground`, `--color-primary`, etc.)
- Font system: 4 presets (primary/body, secondary/subheading, tertiary/heading, accent) in `snippets/theme-styles-variables.liquid`
- Instance styles via inline `style` attributes, not `{% style %}` blocks
- Mobile-first with `min-width` media queries
- Animate only `transform` and `opacity`; respect `prefers-reduced-motion`

## Liquid Conventions

- `{% liquid %}` for multiline blocks
- `{% doc %}...{% enddoc %}` with `@param` JSDoc-style annotations on snippets
- Single `{% content_for 'blocks' %}` per file
- `_` prefix on filenames = internal/helper components (not for merchant use in editor)
- Schema translation keys: `t:names.keyname` (must exist in `locales/en.default.schema.json`)

## Naming Conventions

| Context | Convention | Example |
|---------|------------|---------|
| HTML IDs | CamelCase + section ID | `FeaturedCollection-{{ section.id }}` |
| CSS classes | kebab-case BEM | `.product-card__title` |
| CSS variables | kebab-case, component-prefixed | `--product-card-padding` |
| Liquid variables | snake_case | `product_title` |
| Files | kebab-case | `add-to-cart-button.liquid` |
| Translation keys | dot-notation | `t:sections.header.menu` |

## Key Files

- `layout/theme.liquid` — master layout, RTL detection, head includes
- `assets/base.css` — all component styles
- `assets/rtl-overrides.css` — RTL-specific fixes
- `assets/component.js` — Web Component base class
- `assets/events.js` — custom event system
- `assets/utilities.js` — shared utilities (idle callbacks, view transitions, device detection)
- `snippets/scripts.liquid` — import map, module loading, global Theme object
- `snippets/theme-styles-variables.liquid` — CSS custom properties for fonts/spacing
- `snippets/color-schemes.liquid` — color scheme CSS generation
- `config/settings_schema.json` — theme admin settings
- `.cursor/rules/` — 43 MDC files with detailed component and accessibility guidelines
