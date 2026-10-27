---
name: saas-redesign
description: Redesign a Vue 3 application's UI into a modern SaaS-style interface with a vertical left sidebar instead of a top nav bar, consistent spacing, and a polished professional look. Use when asked to redesign, modernize, restyle, or add a sidebar to a Vue 3 + Vue Router app.
---

# SaaS-Style UI Redesign

Turn a Vue 3 + Vue Router app with a top navigation bar into a modern SaaS interface: fixed left sidebar, a clean content column, one spacing scale, and consistent components. This is a **visual change only**. Routes, data flow, filters, i18n keys, and behavior must not change.

Reference files (read them when you reach the matching step):
- `references/design-tokens.md` - copy-ready CSS variables
- `references/sidebar-layout.md` - sidebar markup, CSS, and responsive behavior

## Rules

- Visual change only. Do not rename routes, props, emits, API calls, or i18n keys.
- No new dependencies, no CDN assets, no icon libraries. Use inline SVG icons.
- No emojis anywhere in the UI.
- Keep the app's existing palette and status colors. Add tokens around them, do not replace them. If the app has none, use the defaults in `references/design-tokens.md`.
- Reuse existing global class names (`.card`, `.stats-grid`, `.badge`, `.page-header`, etc.) and restyle them, so views need few or no edits.
- Never touch `package-lock.json`, `.npmrc`, or registry settings.
- If the project's `CLAUDE.md` requires delegating `.vue` work to a specialist agent (for example `vue-expert`), do that for every created or significantly modified `.vue` file.

## Workflow

### 1. Audit (read-only)

Before editing, find and read:
- The app shell (`App.vue`): current header/nav markup, layout wrapper, global `<style>` block.
- The router (`main.js` or `router/`): the full route list and any route `meta`.
- Nav labels and where they come from (i18n keys, hard-coded strings).
- Shared chrome that lives in the header: language switcher, profile menu, filter bar, search.
- Modals and overlays: note their `z-index` values so the sidebar does not cover them.
- Global classes used across views: `grep` for the most common class names to decide what to restyle globally.

Write a short list of what you found before changing anything.

### 2. Define design tokens

Add CSS variables to `:root` in the existing global style block, using `references/design-tokens.md`. Then replace repeated hard-coded colors, radii, and spacing in the shell and global classes with the variables. Do not mass-rewrite per-view scoped styles unless they visibly clash.

### 3. Build the sidebar shell

Follow `references/sidebar-layout.md`. Requirements:
- Fixed left `aside` (about 240px), content column fills the rest and scrolls independently.
- Nav items come from the router's route list or a single array, not copy-pasted links, so new routes appear automatically.
- Each item: inline SVG icon + label (reuse the existing i18n keys) + clear active state.
- Brand at the top. Language switcher and profile menu move to the bottom of the sidebar.
- Collapsible to an icon rail on desktop, and an off-canvas drawer with a toggle button below about 768px.
- Accessibility: `<nav aria-label="Main">`, `aria-current="page"` on the active link, visible `:focus-visible` rings, toggle button with `aria-expanded` and a label.
- Remove the old top nav styles once the sidebar replaces them. Leave no dead CSS.

Components that previously sat in the top bar (filter bar, search) move into the content column, above the page content, in a consistent toolbar row.

### 4. Normalize the content area

- Wrap routed content in a container with a max width and the standard page padding.
- One page header pattern: title, optional description, optional right-aligned actions.
- Restyle globally: cards, tables, badges, buttons, form controls, empty and loading states.
- Use only the spacing scale for padding, margins, and gaps. No one-off pixel values.
- Tables: sticky or clearly separated header, row hover, right-aligned numeric columns, consistent cell padding.
- Make sure charts and grids still fit when the content column is narrower than the old full-width layout.

### 5. Verify in the browser

Use the project's browser tooling (Playwright MCP if configured) against the running dev server.
- Visit every route at 1440px and at 390px wide. Take a screenshot of each.
- Confirm: no horizontal page scroll, active state matches the current route, drawer opens and closes on mobile, modals appear above the sidebar, no new console errors.
- Confirm filters, language switching, and profile menu still work.
- Compare against the "before" screenshots taken during the audit.

Then run a code review pass (for example the `code-reviewer` agent) over the changed files.

### 6. Report

Summarize: files changed, the token set introduced, before and after screenshots, and any view that needed per-view fixes and why. State anything you could not verify.

## Quality checklist

- [ ] Sidebar replaces the top nav on every route
- [ ] All spacing comes from the token scale
- [ ] One accent color, status colors unchanged in meaning
- [ ] Text contrast meets WCAG AA on sidebar and content
- [ ] Keyboard: Tab reaches every nav item, focus ring visible, Escape closes the mobile drawer
- [ ] Reduced motion respected (`prefers-reduced-motion`)
- [ ] No horizontal scroll at 390px
- [ ] No emojis, no new dependencies, no console errors
- [ ] Old top-nav CSS removed
