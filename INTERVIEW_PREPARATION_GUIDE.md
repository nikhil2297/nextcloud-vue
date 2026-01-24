# Nextcloud Interview Preparation Guide

## Based on Analysis of @nextcloud/vue Library

---

## Table of Contents

1. [JavaScript Concepts to Master](#1-javascript-concepts-to-master)
2. [Vue.js Concepts to Master](#2-vuejs-concepts-to-master)
3. [TypeScript Concepts](#3-typescript-concepts)
4. [Coding Standards at Nextcloud](#4-coding-standards-at-nextcloud)
5. [Library Architecture & Entry Points](#5-library-architecture--entry-points)
6. [Dependencies to Study](#6-dependencies-to-study)
7. [Testing Knowledge](#7-testing-knowledge)
8. [Build Tools & Configuration](#8-build-tools--configuration)
9. [How to Setup a Similar Vue Library Project](#9-how-to-setup-a-similar-vue-library-project)

---

## 1. JavaScript Concepts to Master

### ES6+ Features Used Throughout the Codebase

| Feature | Example Usage | File Reference |
|---------|--------------|----------------|
| **ES Modules** | `import`, `export`, `export default` | All files |
| **Arrow Functions** | `(e) => e.target.value` | Components |
| **Destructuring** | `const { props, emit } = defineProps()` | Vue components |
| **Spread Operator** | `{ ...defaultOptions, ...userOptions }` | vite.config.ts |
| **Template Literals** | `` `Component: ${name}` `` | Logging |
| **Optional Chaining** | `props?.value?.nested` | Safe access patterns |
| **Nullish Coalescing** | `value ?? defaultValue` | Default values |
| **Array Methods** | `.map()`, `.filter()`, `.reduce()`, `.find()` | Data transformations |
| **Async/Await** | `async function fetch() { await api.get() }` | API calls |
| **Promises** | `new Promise((resolve, reject) => ...)` | Async operations |
| **Symbol** | `Symbol.for('InjectionKey')` | Vue provide/inject |
| **Proxy/Reflect** | Vue reactivity system | Understanding Vue internals |
| **WeakMap/WeakSet** | Cache implementations | Memory management |
| **Generators** | Iterator patterns | Advanced patterns |

### Key JavaScript Patterns in Use

```javascript
// 1. Module Pattern - All exports follow this
export { default as NcButton } from './NcButton/index.ts'

// 2. Factory Pattern - Component creation
export function spawnDialog(component, props) { ... }

// 3. Observer Pattern - Event handling
window.addEventListener('resize', handleResize)

// 4. Singleton Pattern - Shared state
const isMobile = ref(false)  // Module-level singleton
export function useIsMobile() { return readonly(isMobile) }

// 5. Debounce/Throttle Patterns
import debounce from 'debounce'
const debouncedSearch = debounce(search, 300)
```

### DOM APIs to Know

- `document.querySelector()`, `querySelectorAll()`
- `element.focus()`, `element.blur()`
- `addEventListener()`, `removeEventListener()`
- `MutationObserver`, `ResizeObserver`, `IntersectionObserver`
- `window.matchMedia()` for responsive detection
- Clipboard API: `navigator.clipboard`
- Storage APIs: `localStorage`, `sessionStorage`

---

## 2. Vue.js Concepts to Master

### Vue 3 Core Concepts (The Library Uses Vue 3.5.18)

#### Composition API (Primary Pattern)

```typescript
// Pattern used in NcButton, NcInputField, NcModal, etc.
<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'

// Props with TypeScript
const props = withDefaults(defineProps<{
  disabled?: boolean
  size?: 'small' | 'normal' | 'large'
}>(), {
  disabled: false,
  size: 'normal'
})

// Emits with TypeScript
const emit = defineEmits<{
  click: [event: MouseEvent]
  'update:modelValue': [value: string]
}>()

// Slots with TypeScript
defineSlots<{
  default?: () => VNode[]
  icon?: () => VNode[]
}>()

// v-model with defineModel (Vue 3.4+)
const modelValue = defineModel<string>({ required: true })

// Reactive State
const isHovered = ref(false)
const buttonClasses = computed(() => ({
  'button--disabled': props.disabled,
  'button--hovered': isHovered.value
}))

// Lifecycle
onMounted(() => { console.log('Component mounted') })
onUnmounted(() => { cleanup() })
</script>
```

#### Options API (Legacy Components Still Use This)

```javascript
export default {
  name: 'NcActionButton',
  components: { NcIconSvgWrapper },
  mixins: [ActionTextMixin],
  inject: { isInSemanticMenu: { from: NC_ACTIONS_IS_SEMANTIC_MENU } },
  props: {
    disabled: { type: Boolean, default: false },
    name: { type: String, required: true }
  },
  emits: ['click', 'update:modelValue'],
  data() {
    return { isActive: false }
  },
  computed: {
    classes() { return { 'is-active': this.isActive } }
  },
  methods: {
    handleClick(event) { this.$emit('click', event) }
  }
}
```

### Vue 3 Features to Master

| Feature | Description | Used In |
|---------|-------------|---------|
| **Composition API** | `setup()`, `<script setup>` | Modern components |
| **Reactivity** | `ref()`, `reactive()`, `computed()` | All components |
| **Lifecycle Hooks** | `onMounted`, `onUnmounted`, `onUpdated` | Component lifecycle |
| **Provide/Inject** | Dependency injection | NcActions, NcFormBox |
| **defineModel** | Two-way binding macro | NcInputField, NcCheckbox |
| **defineProps/Emits** | Type-safe props/events | All modern components |
| **defineSlots** | Typed slots | NcButton, NcModal |
| **Teleport** | Portal rendering | NcModal |
| **Suspense** | Async component loading | Lazy loading |
| **Template Refs** | `useTemplateRef()` | DOM access |

### Composables Pattern (Critical to Understand)

```typescript
// src/composables/useIsMobile/index.ts
import { ref, readonly, onUnmounted } from 'vue'
import type { Ref } from 'vue'

const MOBILE_BREAKPOINT = 1024
const isMobile = ref(window.innerWidth < MOBILE_BREAKPOINT)

function handleResize() {
  isMobile.value = window.innerWidth < MOBILE_BREAKPOINT
}
window.addEventListener('resize', handleResize)

export function useIsMobile(): Readonly<Ref<boolean>> {
  return readonly(isMobile)
}
```

### Custom Directives

```typescript
// src/directives/Focus/index.ts
import type { ObjectDirective } from 'vue'

const directive: ObjectDirective<HTMLElement> = {
  mounted(el: HTMLElement) {
    el.focus()
  }
}
export default directive

// Usage: <input v-focus />
```

### Provide/Inject with TypeScript

```typescript
// Define injection key with type
import type { InjectionKey, ComputedRef } from 'vue'

export const NC_ACTIONS_IS_SEMANTIC_MENU: InjectionKey<ComputedRef<boolean>>
  = Symbol.for('NcActions:isSemanticMenu')

// Provider component
import { provide, computed } from 'vue'
provide(NC_ACTIONS_IS_SEMANTIC_MENU, computed(() => true))

// Consumer component
import { inject } from 'vue'
const isSemanticMenu = inject(NC_ACTIONS_IS_SEMANTIC_MENU, computed(() => false))
```

---

## 3. TypeScript Concepts

### Types Used in the Codebase

```typescript
// Union Types
type ButtonSize = 'small' | 'normal' | 'large'
type ButtonVariant = 'primary' | 'secondary' | 'tertiary' | 'tertiary-no-background'

// Interface Definitions
interface NcButtonProps {
  disabled?: boolean
  href?: string
  alignment?: ButtonAlignment
}

// Generics
function spawnDialog<C extends Component, E>(
  dialog: C,
  props: ComponentProps<C>
): Promise<E>

// Utility Types (from src/utils/VueTypes.ts)
export type ComponentProps<C extends Component> = C extends new (...args: any) => any
  ? Omit<InstanceType<C>['$props'], keyof VNodeProps>
  : never

// Type Assertions
const el = event.target as HTMLInputElement

// Mapped Types
type ReadonlyProps<T> = { readonly [K in keyof T]: T[K] }

// Conditional Types
type IsButton<T> = T extends 'button' ? true : false
```

### TypeScript Configuration

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "allowJs": true,
    "allowImportingTsExtensions": true,
    "noImplicitAny": false  // Allows gradual migration
  },
  "vueCompilerOptions": {
    "target": 3.5
  }
}
```

---

## 4. Coding Standards at Nextcloud

### ESLint Configuration

- Extends `@nextcloud/eslint-config` (recommended library preset)
- Flat config format (ESLint 9.x)
- Vue 3 rules enforced
- TypeScript support
- Strict unused disable directive reporting

### Stylelint Configuration

```javascript
// stylelint.config.cjs
module.exports = {
  extends: ['@nextcloud/stylelint-config'],
  rules: {
    'selector-pseudo-class-no-unknown': [true, {
      ignorePseudoClasses: ['global', 'slotted']
    }],
    'scss/load-partial-extension': ['always']
  }
}
```

### EditorConfig Standards

```editorconfig
[*]
charset = utf-8
indent_style = tab          # TABS, not spaces!
indent_size = 4
end_of_line = lf            # Unix line endings
insert_final_newline = true
trim_trailing_whitespace = true

[*.yml]
indent_style = space
indent_size = 2
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase + "Nc" prefix | `NcButton`, `NcModal`, `NcAvatar` |
| Composables | camelCase + "use" prefix | `useIsMobile`, `useHotKey` |
| Directives | PascalCase | `Focus`, `Linkify` |
| Functions | camelCase | `isDarkTheme`, `preloadImage` |
| TypeScript Types | PascalCase | `ButtonVariant`, `ButtonSize` |
| CSS Classes | BEM-like | `.button-vue`, `.button-vue--primary` |
| Files | Component name matches | `NcButton/NcButton.vue` |

### File Structure Convention

```
src/components/NcButton/
├── NcButton.vue    # Main component
├── index.ts        # Re-export for tree-shaking
└── styles.scss     # (optional) External styles
```

### Vue Component Template Order

```vue
<!--
  - SPDX-FileCopyrightText: 2024 Nextcloud GmbH and Nextcloud contributors
  - SPDX-License-Identifier: AGPL-3.0-or-later
-->

<docs>
Usage documentation with examples
</docs>

<script setup lang="ts">
// 1. Imports
// 2. Types
// 3. Props (withDefaults + defineProps)
// 4. Emits (defineEmits)
// 5. Slots (defineSlots)
// 6. Reactive state
// 7. Computed properties
// 8. Methods
// 9. Lifecycle hooks
</script>

<template>
  <!-- Template code -->
</template>

<style lang="scss" scoped>
/* Scoped SCSS styles */
</style>
```

### Commit Message Convention (Conventional Commits)

```
<type>(<scope>): <description>

Types: feat, fix, docs, style, refactor, test, chore
Examples:
- feat(NcButton): add loading state
- fix(NcModal): correct focus trap behavior
- docs(NcAvatar): update usage examples
```

---

## 5. Library Architecture & Entry Points

### Main Entry Point

```typescript
// src/index.ts
export * from './components/index.ts'
export * from './composables/index.js'
export * from './functions/index.ts'
export * from './directives/index.js'
```

### Export Structure

```
@nextcloud/vue
├── /                    → Main bundle (all exports)
├── /components/*        → Individual components
├── /composables/*       → Vue composables
├── /directives/*        → Vue directives
└── /functions/*         → Utility functions
```

### Import Patterns

```typescript
// Full import (not recommended for production)
import { NcButton, NcModal, useIsMobile } from '@nextcloud/vue'

// Tree-shakeable imports (recommended)
import NcButton from '@nextcloud/vue/components/NcButton'
import { useIsMobile } from '@nextcloud/vue/composables/useIsMobile'
```

### Package.json Exports Field

```json
{
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.mjs"
    },
    "./components/*": {
      "types": "./dist/components/*/index.d.ts",
      "import": "./dist/components/*/index.mjs"
    }
  }
}
```

---

## 6. Dependencies to Study

### Core Framework

| Package | Version | What to Study |
|---------|---------|---------------|
| **vue** | ^3.5.18 | Composition API, Reactivity, Lifecycle |
| **vue-router** | ^4.6.4 | Route guards, Navigation, Dynamic routes |
| **@vueuse/core** | ^14.0.0 | Composable utilities, Reactive helpers |

### UI Component Libraries

| Package | Purpose | Study Focus |
|---------|---------|-------------|
| **floating-vue** | Tooltips, Popovers | Positioning strategies |
| **@vuepic/vue-datepicker** | Date picker | Date handling |
| **vue-select** | Dropdown select | Async options, Filtering |
| **emoji-mart-vue-fast** | Emoji picker | Unicode handling |
| **splitpanes** | Split layouts | Resize handling |

### Positioning & Accessibility

| Package | Purpose | Study Focus |
|---------|---------|-------------|
| **@floating-ui/dom** | Floating elements | Positioning algorithms |
| **focus-trap** | Modal focus | Accessibility patterns |
| **tabbable** | Focusable elements | Keyboard navigation |

### Content Processing

| Package | Purpose | Study Focus |
|---------|---------|-------------|
| **unified** | AST processing | Text processing pipelines |
| **remark-*** | Markdown parsing | Remark plugin ecosystem |
| **rehype-*** | HTML processing | Rehype plugin ecosystem |
| **dompurify** | XSS protection | Security best practices |
| **linkifyjs** | URL detection | Regex patterns |

### Nextcloud-Specific

| Package | Purpose |
|---------|---------|
| **@nextcloud/auth** | Authentication, CSRF tokens |
| **@nextcloud/axios** | Pre-configured HTTP client |
| **@nextcloud/router** | URL generation |
| **@nextcloud/l10n** | Translations (gettext) |
| **@nextcloud/event-bus** | Inter-app communication |
| **@nextcloud/initial-state** | Server state injection |

### Utility Libraries

| Package | Purpose | Common Use Case |
|---------|---------|----------------|
| **debounce** | Rate limiting | Search inputs |
| **clone** | Deep cloning | State management |
| **escape-html** | XSS prevention | User content |
| **striptags** | HTML removal | Plain text extraction |
| **ts-md5** | Hashing | Gravatar URLs |
| **blurhash** | Image placeholders | Loading states |

---

## 7. Testing Knowledge

### Unit Testing with Vitest

```typescript
// tests/unit/components/NcButton/NcButton.spec.ts
import { describe, it, expect, vi } from 'vitest'
import { mount } from '@vue/test-utils'
import NcButton from '../../../../src/components/NcButton/NcButton.vue'

describe('NcButton', () => {
  it('renders with default props', () => {
    const wrapper = mount(NcButton, {
      props: { text: 'Click me' }
    })
    expect(wrapper.text()).toContain('Click me')
  })

  it('emits click event', async () => {
    const wrapper = mount(NcButton)
    await wrapper.trigger('click')
    expect(wrapper.emitted('click')).toBeTruthy()
  })

  it('applies disabled state', () => {
    const wrapper = mount(NcButton, {
      props: { disabled: true }
    })
    expect(wrapper.attributes('disabled')).toBeDefined()
  })
})
```

### Component Testing with Playwright

```typescript
// tests/component/NcButton.spec.ts
import { test, expect } from '@playwright/experimental-ct-vue'
import NcButton from '../../src/components/NcButton/NcButton.vue'

test('button visual regression @visual', async ({ mount, page }) => {
  const component = await mount(NcButton, {
    props: { text: 'Primary Button', variant: 'primary' }
  })

  await expect(component).toHaveScreenshot()
})
```

### Testing Commands

```bash
npm run test              # Run unit tests
npm run test:coverage     # With coverage report
npm run test:component    # Run Playwright tests
npm run test:component:gui  # Interactive mode
npm run update:snapshots  # Update visual snapshots
```

---

## 8. Build Tools & Configuration

### Vite Configuration (Primary Build Tool)

```typescript
// vite.config.ts
import { createLibConfig } from '@nextcloud/vite-config'
import { globSync } from 'glob'

// Auto-discover entry points
const entryPoints = {
  ...globSync('src/@(components|composables|directives|functions)/*/index.@(js|ts)')
    .reduce((acc, item) => {
      const name = item.replace('src/', '').replace(/\/index\.(j|t)s$/, '/index')
      acc[name] = join(import.meta.dirname, item)
      return acc
    }, {}),
  index: resolve(import.meta.dirname, 'src/index.ts'),
}

export default defineConfig((env) => {
  return createLibConfig(entryPoints, {
    inlineCSS: true,           // CSS in JS for backwards compat
    libraryFormats: ['es'],    // ESM only
    DTSPluginOptions: {        // TypeScript declarations
      tsconfigPath: 'src/tsconfig.json',
    },
  })(env)
})
```

### SCSS Configuration

```typescript
css: {
  preprocessorOptions: {
    scss: {
      additionalData: '@use "sass:math"; @use "variables" as *;',
      loadPaths: ['src/assets'],
    },
  },
  modules: {
    hashPrefix: '@nextcloud/vue@9',
    generateScopedName: '_[local]_[hash:base64:5]',
  },
}
```

### Key npm Scripts

```bash
npm run build          # Production build
npm run dev            # Development build
npm run watch          # Watch mode
npm run styleguide     # Component playground
npm run lint           # ESLint check
npm run lint:fix       # Auto-fix lint issues
npm run stylelint      # CSS linting
npm run ts:check       # TypeScript type check
npm run l10n:extract   # Extract translations
```

---

## 9. How to Setup a Similar Vue Library Project

### Step 1: Initialize Project

```bash
mkdir my-vue-library
cd my-vue-library
npm init -y
```

### Step 2: Configure package.json

```json
{
  "name": "@myorg/vue-components",
  "version": "1.0.0",
  "type": "module",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.mjs"
    },
    "./components/*": {
      "types": "./dist/components/*/index.d.ts",
      "import": "./dist/components/*/index.mjs"
    }
  },
  "files": ["dist"],
  "scripts": {
    "build": "vite build",
    "dev": "vite build --watch",
    "test": "vitest run",
    "lint": "eslint",
    "ts:check": "vue-tsc --noEmit"
  }
}
```

### Step 3: Install Dependencies

```bash
# Core
npm install vue vue-router

# Development
npm install -D vite typescript vue-tsc @vue/tsconfig
npm install -D vitest @vue/test-utils jsdom
npm install -D eslint @eslint/js
npm install -D sass stylelint
npm install -D @vitejs/plugin-vue
npm install -D vite-plugin-dts
```

### Step 4: Create Directory Structure

```
src/
├── components/
│   └── MyButton/
│       ├── MyButton.vue
│       └── index.ts
├── composables/
│   └── useTheme/
│       └── index.ts
├── directives/
│   └── Focus/
│       └── index.ts
├── assets/
│   └── variables.scss
└── index.ts
```

### Step 5: Configure TypeScript (tsconfig.json)

```json
{
  "extends": "@vue/tsconfig/tsconfig.dom.json",
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "declaration": true,
    "declarationDir": "./dist",
    "outDir": "./dist"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Step 6: Configure Vite (vite.config.ts)

```typescript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import dts from 'vite-plugin-dts'
import { resolve } from 'path'
import { globSync } from 'glob'

const entryPoints = {
  ...globSync('src/components/*/index.ts').reduce((acc, file) => {
    const name = file.replace('src/', '').replace('/index.ts', '/index')
    acc[name] = resolve(__dirname, file)
    return acc
  }, {}),
  index: resolve(__dirname, 'src/index.ts'),
}

export default defineConfig({
  plugins: [
    vue(),
    dts({ tsconfigPath: './tsconfig.json' }),
  ],
  build: {
    lib: {
      entry: entryPoints,
      formats: ['es'],
    },
    rollupOptions: {
      external: ['vue', 'vue-router'],
      output: {
        preserveModules: true,
        entryFileNames: '[name].mjs',
      },
    },
  },
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: '@use "variables" as *;',
        loadPaths: [resolve(__dirname, 'src/assets')],
      },
    },
  },
})
```

### Step 7: Create a Component

```vue
<!-- src/components/MyButton/MyButton.vue -->
<script setup lang="ts">
export type ButtonVariant = 'primary' | 'secondary'

const props = withDefaults(defineProps<{
  text?: string
  variant?: ButtonVariant
  disabled?: boolean
}>(), {
  variant: 'primary',
  disabled: false,
})

const emit = defineEmits<{
  click: [event: MouseEvent]
}>()
</script>

<template>
  <button
    :class="['my-button', `my-button--${variant}`]"
    :disabled="disabled"
    @click="emit('click', $event)"
  >
    <slot>{{ text }}</slot>
  </button>
</template>

<style lang="scss" scoped>
.my-button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;

  &--primary {
    background: $color-primary;
    color: white;
  }

  &--secondary {
    background: $color-secondary;
    color: black;
  }

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}
</style>
```

### Step 8: Create Index Files

```typescript
// src/components/MyButton/index.ts
export type * from './MyButton.vue'
export { default } from './MyButton.vue'

// src/components/index.ts
export { default as MyButton } from './MyButton/index.ts'

// src/index.ts
export * from './components/index.ts'
export * from './composables/index.ts'
```

### Step 9: Create a Composable

```typescript
// src/composables/useTheme/index.ts
import { ref, readonly, type Ref } from 'vue'

const isDark = ref(false)

export function useTheme(): { isDark: Readonly<Ref<boolean>>, toggle: () => void } {
  const toggle = () => {
    isDark.value = !isDark.value
  }

  return {
    isDark: readonly(isDark),
    toggle,
  }
}
```

### Step 10: Configure Testing (vitest.config.ts)

```typescript
import { defineConfig } from 'vitest/config'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  test: {
    environment: 'jsdom',
    include: ['tests/**/*.spec.ts'],
  },
})
```

### Step 11: Add Linting Configs

```javascript
// eslint.config.js
import js from '@eslint/js'
import vue from 'eslint-plugin-vue'
import ts from '@typescript-eslint/eslint-plugin'

export default [
  js.configs.recommended,
  ...vue.configs['flat/recommended'],
]
```

```javascript
// stylelint.config.cjs
module.exports = {
  extends: ['stylelint-config-standard-scss'],
}
```

### Step 12: Build and Publish

```bash
npm run build
npm publish --access public
```

---

## Quick Reference Cheat Sheet

### Most Important Concepts

1. **Vue 3 Composition API** - `<script setup>`, `ref`, `computed`, `watch`
2. **TypeScript with Vue** - `defineProps<T>()`, `defineEmits<T>()`
3. **Composables** - Reusable reactive logic with `use*` naming
4. **Provide/Inject** - Dependency injection with `InjectionKey`
5. **Tree-shaking** - Per-component exports, barrel files
6. **Vite** - Modern build tool, ESM-first
7. **Testing** - Vitest for units, Playwright for components
8. **CSS Modules** - Scoped styles, SCSS preprocessing

### Interview Discussion Points

1. Why Vue 3 over Vue 2? (Composition API, better TS, performance)
2. Options API vs Composition API trade-offs
3. How does reactivity work in Vue 3?
4. How to structure a component library for tree-shaking?
5. Testing strategies for UI components
6. Accessibility in component libraries
7. Handling theming and dark mode
8. Internationalization approaches

---

*Generated from analysis of @nextcloud/vue v9.4.0*
