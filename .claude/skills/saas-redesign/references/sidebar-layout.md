# Sidebar Layout Reference

A starting point. Adapt names, routes, and i18n keys to the app you audited.

## Markup (App.vue template)

```vue
<div class="app-shell" :class="{ collapsed: collapsed, 'drawer-open': drawerOpen }">
  <button
    class="sidebar-toggle-mobile"
    type="button"
    :aria-expanded="drawerOpen"
    aria-controls="app-sidebar"
    aria-label="Toggle navigation"
    @click="drawerOpen = !drawerOpen"
  >
    <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M4 6h16M4 12h16M4 18h16"/></svg>
  </button>

  <aside id="app-sidebar" class="sidebar">
    <div class="sidebar-brand">
      <span class="brand-name">{{ t('nav.companyName') }}</span>
      <span class="brand-sub">{{ t('nav.subtitle') }}</span>
    </div>

    <nav class="sidebar-nav" aria-label="Main">
      <router-link
        v-for="item in navItems"
        :key="item.to"
        :to="item.to"
        class="nav-item"
        :aria-current="$route.path === item.to ? 'page' : undefined"
        @click="drawerOpen = false"
      >
        <svg class="nav-icon" viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <path :d="item.icon" />
        </svg>
        <span class="nav-label">{{ item.label }}</span>
      </router-link>
    </nav>

    <div class="sidebar-footer">
      <LanguageSwitcher />
      <ProfileMenu />
      <button class="collapse-btn" type="button" :aria-pressed="collapsed" @click="collapsed = !collapsed">
        <span class="nav-label">Collapse</span>
      </button>
    </div>
  </aside>

  <div class="drawer-backdrop" @click="drawerOpen = false"></div>

  <div class="content-column">
    <FilterBar />   <!-- keep any toolbar here, above the page -->
    <main class="main-content">
      <router-view />
    </main>
  </div>
</div>
```

Script additions:

```js
const collapsed = ref(false)
const drawerOpen = ref(false)

// Single source of truth for nav. Labels reuse existing i18n keys.
const navItems = computed(() => [
  { to: '/',          label: t('nav.overview'),  icon: 'M3 12l9-9 9 9M5 10v10h14V10' },
  { to: '/inventory', label: t('nav.inventory'), icon: 'M21 8l-9-5-9 5v8l9 5 9-5V8zM3 8l9 5 9-5M12 13v8' },
  // one entry per route found in the audit
])

// Close the drawer on Escape
const onKey = (e) => { if (e.key === 'Escape') drawerOpen.value = false }
onMounted(() => window.addEventListener('keydown', onKey))
onBeforeUnmount(() => window.removeEventListener('keydown', onKey))
```

Pick simple, consistent 24px stroke icons per route. Keep every icon inline SVG.

## CSS

```css
.app-shell {
  display: grid;
  grid-template-columns: var(--sidebar-width) 1fr;
  min-height: 100vh;
  background: var(--color-bg);
  transition: grid-template-columns var(--transition);
}
.app-shell.collapsed { grid-template-columns: var(--sidebar-width-collapsed) 1fr; }

.sidebar {
  position: sticky; top: 0; height: 100vh;
  display: flex; flex-direction: column;
  background: var(--sidebar-bg); color: var(--sidebar-text);
  padding: var(--space-5) var(--space-3);
  overflow-y: auto;
  z-index: 40;           /* keep below modal overlays */
}
.sidebar-brand { padding: 0 var(--space-3) var(--space-5); }
.brand-name { display: block; color: var(--sidebar-text-active); font-weight: var(--weight-semibold); font-size: var(--text-lg); }
.brand-sub { font-size: var(--text-xs); color: var(--sidebar-text); }

.sidebar-nav { display: flex; flex-direction: column; gap: var(--space-1); flex: 1; }
.nav-item {
  display: flex; align-items: center; gap: var(--space-3);
  padding: var(--space-3);
  border-radius: var(--radius-md);
  color: var(--sidebar-text); text-decoration: none;
  font-size: var(--text-sm); font-weight: var(--weight-medium);
  transition: background var(--transition), color var(--transition);
}
.nav-item:hover { background: var(--sidebar-hover); color: var(--sidebar-text-active); }
.nav-item.router-link-exact-active,
.nav-item[aria-current="page"] { background: var(--sidebar-active); color: var(--sidebar-text-active); }
.nav-item:focus-visible,
.collapse-btn:focus-visible { outline: 2px solid var(--color-accent); outline-offset: 2px; }
.nav-icon { flex: none; }

.sidebar-footer { display: flex; flex-direction: column; gap: var(--space-3); padding-top: var(--space-4); border-top: 1px solid var(--sidebar-hover); }

.app-shell.collapsed .nav-label,
.app-shell.collapsed .brand-sub { display: none; }
.app-shell.collapsed .nav-item { justify-content: center; }

.content-column { min-width: 0; display: flex; flex-direction: column; }
.main-content { width: 100%; max-width: 80rem; margin: 0 auto; padding: var(--space-6) var(--space-5); }

.sidebar-toggle-mobile, .drawer-backdrop { display: none; }

@media (max-width: 768px) {
  .app-shell, .app-shell.collapsed { grid-template-columns: 1fr; }
  .sidebar {
    position: fixed; inset: 0 auto 0 0; width: var(--sidebar-width);
    transform: translateX(-100%); transition: transform var(--transition);
  }
  .app-shell.collapsed .nav-label,
  .app-shell.collapsed .brand-sub { display: inline; }
  .app-shell.drawer-open .sidebar { transform: translateX(0); }
  .sidebar-toggle-mobile {
    display: inline-flex; position: fixed; top: var(--space-3); left: var(--space-3); z-index: 50;
    padding: var(--space-2); border: 1px solid var(--color-border); border-radius: var(--radius-md);
    background: var(--color-surface); color: var(--color-text); cursor: pointer;
  }
  .app-shell.drawer-open .drawer-backdrop {
    display: block; position: fixed; inset: 0; background: rgba(15, 23, 42, 0.5); z-index: 30;
  }
  .collapse-btn { display: none; }
  .main-content { padding: var(--space-7) var(--space-4) var(--space-5); } /* room for the toggle */
}
```

## Notes

- Check modal and dropdown `z-index` values from the audit. The sidebar (40) and toggle (50) must sit below modal overlays; raise the modals if needed.
- Components that open menus upward or to the right (profile menu, language switcher) may need their popover direction changed now that they live at the bottom-left.
- Delete the old `.top-nav`, `.nav-container`, and `.nav-tabs` rules after the sidebar works.
