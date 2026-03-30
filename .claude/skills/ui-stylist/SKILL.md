---
name: ui-stylist
description: Apply, create, or modify color themes for boudreaux-labs front-end apps. Themes are defined in a themes.js file as structured objects with color tokens, accent values, and font assignments. Use this skill when the user wants to add a new theme, switch a site to use the theme system, or adjust colors in an existing theme.
user-invocable: true
---

# UI Stylist

You help manage color themes for boudreaux-labs front-end apps.

## Theme contract

Every theme is an entry in `themes.js` — a plain `const THEMES = [...]` array loaded via `<script src="themes.js">` before the main script block. Each theme object has this shape:

```js
{
  id: 'theme-id',          // kebab-case, unique
  name: 'Display Name',
  dot: '#rrggbb',          // swatch color shown in the picker UI
  fonts: {
    headline: 'Font Name', // serif — used for h1/h2
    body:     'Font Name', // used for prose
    label:    'Font Name', // used for UI labels, tags, metadata
  },
  colors: {
    // Full Material Design 3 surface/primary/secondary/tertiary token set.
    // Keys are kebab-case matching Tailwind color names in the site's config.
    'surface': '#rrggbb',
    'surface-dim': '#rrggbb',
    'surface-bright': '#rrggbb',
    'surface-container-lowest': '#rrggbb',
    'surface-container-low': '#rrggbb',
    'surface-container': '#rrggbb',
    'surface-container-high': '#rrggbb',
    'surface-container-highest': '#rrggbb',
    'background': '#rrggbb',
    'on-surface': '#rrggbb',
    'on-surface-variant': '#rrggbb',
    'on-background': '#rrggbb',
    'primary': '#rrggbb',
    'primary-container': '#rrggbb',
    'on-primary': '#rrggbb',
    // ... full token set matching existing themes
  },
  accents: {
    gold:        '#rrggbb', // primary brand accent — drives CSS --color-gold
    goldBright:  '#rrggbb', // lighter variant
    nav:         '#rrggbb', // header + sidebar background
    navBorder:   'rgba(...)', // sidebar right border
    sidebarText: '#rrggbb', // inactive nav item text
  }
}
```

## Runtime: how themes apply

The site uses CSS custom properties on `:root` for all structural colors:

```css
:root {
  --color-gold:         #d4af37;
  --color-gold-bright:  #f2ca50;
  --color-nav:          #131313;
  --color-nav-border:   rgba(212,175,55,0.05);
  --color-sidebar-text: #ffc293;
}
```

`applyTheme(id)` injects a `<style id="theme-colors">` tag at runtime that:
1. Sets all `--tw-*` CSS vars from `theme.colors`
2. Generates `.text-{token}`, `.bg-{token}`, `.border-{token}` overrides
3. Sets `--color-gold`, `--color-nav`, etc. from `theme.accents`
4. Updates `document.body.style.fontFamily`

The picker is rendered in the sidebar as colored dot buttons. Active theme persists in `localStorage` under key `speakeasy-theme`.

## Accepting color input

The user may provide colors in any format:
- Hex list: `primary: #89d1e6, background: #0c1515`
- Natural language: "deep teal background, electric cyan primary"
- Screenshot or image: sample the dominant colors
- Prose description: "warm terracotta on dark cocoa"
- Tailwind config snippet
- Material Design export

When given sparse input (e.g. just a primary + background), derive the full token set by:
1. Computing surface tiers as lightness steps from the background (each ~6-10% lighter)
2. Deriving `on-surface` as a desaturated near-white matching the primary hue
3. Deriving `on-surface-variant` at ~75% opacity of `on-surface`
4. Setting `primary-container` as a darker/more saturated version of `primary`
5. Deriving secondary/tertiary from analogous or complementary hues of primary
6. Matching `accents.gold` to `primary`, `accents.nav` to `surface`

## Adding a new theme

1. Read the existing `themes.js` to understand current entries
2. Build the full theme object following the contract above
3. Append it to the `THEMES` array in `themes.js`
4. If a new font is used, verify it's loaded in the Google Fonts `<link>` tag in the HTML

## Migrating a new app to use the theme system

If asked to add theme support to a new app in the boudreaux-labs org:
1. Copy `themes.js` from `app-speakeasy/` as the starting point
2. Add `<script src="themes.js"></script>` to the HTML head
3. Add the `:root` CSS vars block to the site's `<style>` tag
4. Add structural CSS classes (`speakeasy-wordmark`, etc.) or adapt equivalents
5. Add `applyTheme()` and `renderThemePicker()` functions from `app-speakeasy/home_bar.html`
6. Add the theme picker `<div id="theme-picker">` to the sidebar

## Fonts available (already loaded in app-speakeasy)

- Noto Serif (headline)
- Newsreader (body)
- Work Sans (label)
- Manrope (body/label)
- Space Grotesk (label)

Add new fonts by appending to the Google Fonts `<link>` tag.
