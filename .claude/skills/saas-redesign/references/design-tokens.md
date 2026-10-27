# Design Tokens

Add these to `:root` in the app's global style block. If the app already has a palette, keep its values for the color tokens and only add what is missing.

```css
:root {
  /* Spacing: 4px base */
  --space-1: 0.25rem;  /* 4  */
  --space-2: 0.5rem;   /* 8  */
  --space-3: 0.75rem;  /* 12 */
  --space-4: 1rem;     /* 16 */
  --space-5: 1.5rem;   /* 24 */
  --space-6: 2rem;     /* 32 */
  --space-7: 3rem;     /* 48 */

  /* Type */
  --font-sans: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.5rem;
  --text-2xl: 1.875rem;
  --weight-regular: 400;
  --weight-medium: 500;
  --weight-semibold: 600;

  /* Neutrals (slate) */
  --color-bg: #f8fafc;
  --color-surface: #ffffff;
  --color-border: #e2e8f0;
  --color-text: #0f172a;
  --color-text-muted: #64748b;

  /* Sidebar */
  --sidebar-bg: #0f172a;
  --sidebar-text: #cbd5e1;
  --sidebar-text-active: #ffffff;
  --sidebar-hover: rgba(255, 255, 255, 0.06);
  --sidebar-active: rgba(255, 255, 255, 0.12);
  --sidebar-width: 15rem;
  --sidebar-width-collapsed: 4.5rem;

  /* Accent and status */
  --color-accent: #2563eb;
  --color-accent-soft: #eff6ff;
  --color-success: #16a34a;
  --color-info: #2563eb;
  --color-warning: #ca8a04;
  --color-danger: #dc2626;

  /* Shape and depth */
  --radius-sm: 0.375rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  --shadow-sm: 0 1px 2px rgba(15, 23, 42, 0.06);
  --shadow-md: 0 4px 12px rgba(15, 23, 42, 0.08);

  /* Motion */
  --transition: 150ms ease;
}

@media (prefers-reduced-motion: reduce) {
  :root { --transition: 0ms; }
}
```

## Usage rules

- Padding, margin, and gap use `--space-*` only.
- Card: `background: var(--color-surface); border: 1px solid var(--color-border); border-radius: var(--radius-lg); box-shadow: var(--shadow-sm); padding: var(--space-5);`
- Page title uses `--text-xl` and `--weight-semibold`. Section titles use `--text-lg`. Body uses `--text-sm` or `--text-base`.
- Muted text uses `--color-text-muted`. Do not use lighter grays than that for readable text.
- Status badges keep their existing meaning: green success, blue info, yellow warning, red danger.
- Check contrast: sidebar text on `--sidebar-bg` and muted text on `--color-bg` must meet WCAG AA (4.5:1).
