<template>
  <div class="app app-shell" :class="{ collapsed: railMode, 'drawer-open': drawerOpen }">
    <div class="mobile-topbar" aria-hidden="true"></div>
    <button
      class="sidebar-toggle-mobile"
      type="button"
      :aria-expanded="drawerOpen ? 'true' : 'false'"
      aria-controls="app-sidebar"
      aria-label="Toggle navigation"
      @click="drawerOpen = !drawerOpen"
    >
      <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" aria-hidden="true"><path d="M4 6h16M4 12h16M4 18h16" /></svg>
    </button>

    <aside id="app-sidebar" class="sidebar">
      <div class="sidebar-brand">
        <span class="brand-mark" aria-hidden="true">{{ brandInitial }}</span>
        <span class="brand-name">{{ t('nav.companyName') }}</span>
        <span class="brand-sub">{{ t('nav.subtitle') }}</span>
      </div>

      <nav class="sidebar-nav" aria-label="Main">
        <router-link
          v-for="item in navItems"
          :key="item.to"
          :to="item.to"
          class="nav-item"
          :title="item.label"
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
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
        <button
          class="collapse-btn"
          type="button"
          :aria-pressed="railMode ? 'true' : 'false'"
          :aria-label="railMode ? 'Expand sidebar' : 'Collapse sidebar'"
          :title="railMode ? 'Expand sidebar' : 'Collapse sidebar'"
          @click="toggleCollapse"
        >
          <svg class="collapse-icon" viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M11 17l-5-5 5-5M18 17l-5-5 5-5" />
          </svg>
        </button>
      </div>
    </aside>

    <div class="drawer-backdrop" @click="drawerOpen = false"></div>

    <div class="content-column">
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    // Shell state (visual only)
    const STORAGE_KEY = 'sidebar-collapsed'
    const readStored = () => {
      try {
        return window.localStorage.getItem(STORAGE_KEY) === '1'
      } catch (e) {
        return false
      }
    }
    const writeStored = (value) => {
      try {
        window.localStorage.setItem(STORAGE_KEY, value ? '1' : '0')
      } catch (e) {
        // storage unavailable: keep in-memory state only
      }
    }

    const collapsed = ref(readStored()) // manual state, applies above 1024px
    const midExpanded = ref(false) // user expanded the rail at 769-1024px
    const drawerOpen = ref(false)

    const midQuery = typeof window !== 'undefined' && window.matchMedia
      ? window.matchMedia('(min-width: 769px) and (max-width: 1024px)')
      : null
    const isMid = ref(midQuery ? midQuery.matches : false)
    const onMidChange = (e) => { isMid.value = e.matches }

    const railMode = computed(() => (isMid.value ? !midExpanded.value : collapsed.value))
    const brandInitial = computed(() => (t('nav.companyName') || '').trim().charAt(0).toUpperCase())

    const toggleCollapse = () => {
      if (isMid.value) {
        midExpanded.value = !midExpanded.value
      } else {
        collapsed.value = !collapsed.value
        writeStored(collapsed.value)
      }
    }

    const navItems = computed(() => [
      { to: '/', label: t('nav.overview'), icon: 'M3 3h7v7H3zM14 3h7v7h-7zM14 14h7v7h-7zM3 14h7v7H3z' },
      { to: '/inventory', label: t('nav.inventory'), icon: 'M21 8l-9-5-9 5v8l9 5 9-5V8zM3 8l9 5 9-5M12 13v8' },
      { to: '/orders', label: t('nav.orders'), icon: 'M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 012-2h2a2 2 0 012 2 2 2 0 01-2 2h-2a2 2 0 01-2-2zM9 12h6M9 16h6' },
      { to: '/spending', label: t('nav.finance'), icon: 'M12 1v22M17 5H9.5a3.5 3.5 0 000 7h5a3.5 3.5 0 010 7H6' },
      { to: '/demand', label: t('nav.demandForecast'), icon: 'M3 17l6-6 4 4 8-8M15 7h6v6' },
      { to: '/reports', label: 'Reports', icon: 'M14 3H7a2 2 0 00-2 2v14a2 2 0 002 2h10a2 2 0 002-2V8zM14 3v5h5M9 13h6M9 17h6' }
    ])

    const onKey = (e) => {
      if (e.key === 'Escape') drawerOpen.value = false
    }
    onMounted(() => {
      window.addEventListener('keydown', onKey)
      if (midQuery) midQuery.addEventListener('change', onMidChange)
    })
    onBeforeUnmount(() => {
      window.removeEventListener('keydown', onKey)
      if (midQuery) midQuery.removeEventListener('change', onMidChange)
    })

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      railMode,
      brandInitial,
      toggleCollapse,
      drawerOpen,
      navItems
    }
  }
}
</script>

<style>
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

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', var(--font-sans);
  background: var(--color-bg);
  color: var(--color-text);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

:focus-visible {
  outline: 2px solid var(--color-accent);
  outline-offset: 2px;
}

/* ===== App shell ===== */
.app-shell {
  display: grid;
  grid-template-columns: var(--sidebar-width) minmax(0, 1fr);
  min-height: 100vh;
  background: var(--color-bg);
  transition: grid-template-columns var(--transition);
}

.app-shell.collapsed {
  grid-template-columns: var(--sidebar-width-collapsed) minmax(0, 1fr);
}

.sidebar {
  position: sticky;
  top: 0;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: var(--sidebar-bg);
  color: var(--sidebar-text);
  padding: var(--space-5) var(--space-3);
  z-index: 1500; /* above FilterBar/popovers, below 2000 modals */
}

.sidebar-brand {
  padding: 0 var(--space-3) var(--space-5);
  min-height: 4rem;
}

.brand-name {
  display: block;
  color: var(--sidebar-text-active);
  font-weight: var(--weight-semibold);
  font-size: var(--text-lg);
  letter-spacing: -0.025em;
}

.brand-sub {
  display: block;
  font-size: var(--text-xs);
  color: var(--sidebar-text);
}

.sidebar-nav {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  flex: 1;
  min-height: 0;
  overflow-y: auto;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-3);
  border-radius: var(--radius-md);
  color: var(--sidebar-text);
  text-decoration: none;
  font-size: var(--text-sm);
  font-weight: var(--weight-medium);
  transition: background var(--transition), color var(--transition);
}

.nav-item:hover {
  background: var(--sidebar-hover);
  color: var(--sidebar-text-active);
}

.nav-item[aria-current="page"] {
  background: var(--sidebar-active);
  color: var(--sidebar-text-active);
}

.sidebar :focus-visible {
  outline: 2px solid #93c5fd;
  outline-offset: 2px;
}

.nav-icon {
  flex: none;
}

.sidebar-footer {
  display: flex;
  flex-direction: column;
  gap: var(--space-3);
  padding-top: var(--space-4);
  border-top: 1px solid var(--sidebar-active);
}

.sidebar-footer .language-button,
.sidebar-footer .profile-button {
  width: 100%;
}

.collapse-btn {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  padding: var(--space-3);
  background: transparent;
  border: none;
  border-radius: var(--radius-md);
  color: var(--sidebar-text);
  cursor: pointer;
  transition: background var(--transition), color var(--transition);
}

.collapse-btn:hover {
  background: var(--sidebar-hover);
  color: var(--sidebar-text-active);
}

.collapse-icon {
  transition: transform var(--transition);
}

.app-shell.collapsed .collapse-icon {
  transform: rotate(180deg);
}

/* Collapsed icon rail (desktop/tablet). Labels stay in the DOM, visually hidden. */
.brand-mark {
  display: none;
}

@media (min-width: 769px) {
  .app-shell.collapsed .nav-label,
  .app-shell.collapsed .brand-name,
  .app-shell.collapsed .brand-sub,
  .app-shell.collapsed .sidebar-footer .language-label,
  .app-shell.collapsed .sidebar-footer .profile-name {
    position: absolute;
    width: 1px;
    height: 1px;
    overflow: hidden;
    clip: rect(0 0 0 0);
    clip-path: inset(50%);
    white-space: nowrap;
  }

  .app-shell.collapsed .sidebar-footer .chevron {
    display: none;
  }

  .app-shell.collapsed .sidebar-brand {
    padding-left: 0;
    padding-right: 0;
    display: flex;
    justify-content: center;
  }

  .app-shell.collapsed .brand-mark {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 2.5rem;
    height: 2.5rem;
    border-radius: var(--radius-md);
    background: var(--color-accent);
    color: var(--sidebar-text-active);
    font-weight: var(--weight-semibold);
    font-size: var(--text-lg);
  }

  .app-shell.collapsed .nav-item,
  .app-shell.collapsed .collapse-btn {
    justify-content: center;
  }

  .app-shell.collapsed .sidebar-footer .language-button,
  .app-shell.collapsed .sidebar-footer .profile-button {
    justify-content: center;
    padding-left: var(--space-2);
    padding-right: var(--space-2);
  }
}

.content-column {
  min-width: 0;
  display: flex;
  flex-direction: column;
}

.main-content {
  flex: 1;
  width: 100%;
  max-width: 90rem;
  margin: 0 auto;
  padding: var(--space-5);
}

.sidebar-toggle-mobile,
.drawer-backdrop,
.mobile-topbar {
  display: none;
}

@media (max-width: 768px) {
  .app-shell,
  .app-shell.collapsed {
    grid-template-columns: minmax(0, 1fr);
  }

  .sidebar {
    position: fixed;
    inset: 0 auto 0 0;
    width: var(--sidebar-width);
    transform: translateX(-100%);
    visibility: hidden;
    transition: transform var(--transition), visibility var(--transition);
  }

  .app-shell.drawer-open .sidebar {
    transform: translateX(0);
    visibility: visible;
  }

  .collapse-btn {
    display: none;
  }

  .mobile-topbar {
    display: block;
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    height: 3.5rem;
    background: var(--color-surface);
    border-bottom: 1px solid var(--color-border);
    z-index: 1300;
  }

  .sidebar-toggle-mobile {
    display: inline-flex;
    position: fixed;
    top: var(--space-2);
    left: var(--space-3);
    z-index: 1400;
    padding: var(--space-2);
    border: 1px solid var(--color-border);
    border-radius: var(--radius-md);
    background: var(--color-surface);
    color: var(--color-text);
    cursor: pointer;
  }

  .app-shell.drawer-open .drawer-backdrop {
    display: block;
    position: fixed;
    inset: 0;
    background: rgba(15, 23, 42, 0.5);
    z-index: 1450;
  }

  .content-column {
    padding-top: 3.5rem;
  }

  .main-content {
    padding: var(--space-4);
  }
}

/* ===== Global components ===== */
.page-header {
  margin-bottom: var(--space-5);
}

.page-header h2 {
  font-size: var(--text-xl);
  font-weight: var(--weight-semibold);
  color: var(--color-text);
  margin-bottom: var(--space-1);
  letter-spacing: -0.025em;
}

.page-header p {
  color: var(--color-text-muted);
  font-size: var(--text-sm);
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 12rem), 1fr));
  gap: var(--space-4);
  margin-bottom: var(--space-5);
}

.stat-card {
  background: var(--color-surface);
  padding: var(--space-5);
  border-radius: var(--radius-lg);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-sm);
  transition: border-color var(--transition), box-shadow var(--transition);
}

.stat-card:hover {
  border-color: #cbd5e1;
  box-shadow: var(--shadow-md);
}

.stat-label {
  color: var(--color-text-muted);
  font-size: var(--text-xs);
  font-weight: var(--weight-semibold);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: var(--space-2);
}

.stat-value {
  font-size: var(--text-2xl);
  font-weight: var(--weight-semibold);
  color: var(--color-text);
  overflow-wrap: anywhere;
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value {
  color: #ea580c;
}

.stat-card.success .stat-value {
  color: #059669;
}

.stat-card.danger .stat-value {
  color: #dc2626;
}

.stat-card.info .stat-value {
  color: #2563eb;
}

.card {
  background: var(--color-surface);
  border-radius: var(--radius-lg);
  padding: var(--space-5);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-sm);
  margin-bottom: var(--space-4);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: var(--space-2);
  margin-bottom: var(--space-4);
  padding-bottom: var(--space-3);
  border-bottom: 1px solid var(--color-border);
}

.card-title {
  font-size: var(--text-lg);
  font-weight: var(--weight-semibold);
  color: var(--color-text);
  letter-spacing: -0.025em;
}

.table-container {
  overflow-x: auto;
  max-width: 100%;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--color-bg);
  border-top: 1px solid var(--color-border);
  border-bottom: 2px solid var(--color-border);
}

th {
  text-align: left;
  padding: var(--space-3);
  font-weight: var(--weight-semibold);
  color: #475569;
  font-size: var(--text-xs);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: var(--space-3);
  border-top: 1px solid #f1f5f9;
  color: #334155;
  font-size: var(--text-sm);
  white-space: nowrap;
}

tbody tr {
  transition: background-color var(--transition);
}

tbody tr:hover {
  background: var(--color-accent-soft);
}

.badge {
  display: inline-block;
  white-space: nowrap;
  padding: var(--space-1) var(--space-3);
  border-radius: var(--radius-sm);
  font-size: var(--text-xs);
  font-weight: var(--weight-semibold);
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.badge.success {
  background: #d1fae5;
  color: #065f46;
}

.badge.warning {
  background: #fed7aa;
  color: #92400e;
}

.badge.danger {
  background: #fecaca;
  color: #991b1b;
}

.badge.info {
  background: #dbeafe;
  color: #1e40af;
}

.badge.increasing {
  background: #d1fae5;
  color: #065f46;
}

.badge.decreasing {
  background: #fecaca;
  color: #991b1b;
}

.badge.stable {
  background: #e0e7ff;
  color: #3730a3;
}

.badge.high {
  background: #fecaca;
  color: #991b1b;
}

.badge.medium {
  background: #fed7aa;
  color: #92400e;
}

.badge.low {
  background: #dbeafe;
  color: #1e40af;
}

.loading {
  text-align: center;
  padding: var(--space-7);
  color: var(--color-text-muted);
  font-size: var(--text-sm);
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: var(--space-4);
  border-radius: var(--radius-md);
  margin: var(--space-4) 0;
  font-size: var(--text-sm);
}

/* Global buttons and form controls (only affect elements without scoped overrides) */
select,
input[type="text"],
input[type="search"],
input[type="number"],
input[type="date"],
textarea {
  font-family: inherit;
  color: var(--color-text);
}

select:focus-visible,
input:focus-visible,
textarea:focus-visible,
button:focus-visible {
  outline: 2px solid var(--color-accent);
  outline-offset: 2px;
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    transition-duration: 0ms !important;
    animation-duration: 0ms !important;
  }
}
</style>
