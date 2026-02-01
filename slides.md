---
marp: true
theme: default
paginate: true
backgroundColor: #fff
style: |
  section {
    font-family: 'Red Hat Display', 'Helvetica Neue', sans-serif;
  }
  h1 {
    color: #EE0000;
  }
  h2 {
    color: #151515;
  }
  code {
    background: #f5f5f5;
  }
  .columns {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
  }
  .highlight {
    background: #FDF1F1;
    padding: 1rem;
    border-left: 4px solid #EE0000;
  }
  .architecture-diagram {
    font-family: monospace;
    font-size: 14px;
    background: #1a1a2e;
    color: #0f0;
    padding: 1rem;
    border-radius: 8px;
  }
  table {
    font-size: 0.85em;
  }
---

<!-- _class: lead -->
<!-- _backgroundColor: #151515 -->
<!-- _color: white -->

# Design Systems at Scale

## Maintaining UI Consistency in Dynamic Plugin Architectures

**DevConf.IN 2026 - Track 4: User Experience and Design Engineering**

Mitesh Kumar
Frontend Contributor, Red Hat Developer Hub

<!--
SPEAKER NOTES:
- Welcome everyone
- This is a practical guide for anyone building plugin-based platforms
- We'll cover patterns that work at scale
-->

---

# What You'll Learn Today

A practical guide for organizations building plugin-based platforms

<div class="columns">

<div>

### The Challenge
- Why UI consistency is hard in plugin systems
- Common failure patterns

</div>

<div>

### The Solutions
- 5 proven strategies
- Real implementation patterns
- Code you can adapt

</div>

</div>

**Case Study**: Red Hat Developer Hub (100+ plugins)

<!--
SPEAKER NOTES:
- This isn't theory - these are battle-tested patterns
- You'll leave with actionable strategies
- RHDH is our proof that this works
-->

---

<!-- _class: lead -->
<!-- _backgroundColor: #EE0000 -->
<!-- _color: white -->

# Part 1
## The Universal Challenge

---

# Why Plugin Architectures?

<div class="columns">

<div>

### Benefits
- Extensibility without core changes
- Third-party ecosystem growth
- Independent team velocity
- Separation of concerns

</div>

<div>

### Who Uses Them?
- **VS Code** - 40,000+ extensions
- **Figma** - 2,000+ plugins
- **Shopify** - 8,000+ apps
- **WordPress** - 60,000+ plugins
- **Backstage** - 100+ plugins

</div>

</div>

**You're probably building one too.**

<!--
SPEAKER NOTES:
- Plugin architectures are everywhere
- They enable ecosystems to grow without bottlenecking on core team
- But they come with a challenge...
-->

---

# The Consistency Problem

> "Your users don't care that your UI is composed of 50 different plugins. They expect it to feel like **ONE application**."

<div class="columns">

<div>

### What Goes Wrong
- Different teams = different decisions
- Third-party plugins ignore guidelines
- Version drift in dependencies
- CSS conflicts between plugins
- Inconsistent a11y, theming

</div>

<div>

### The Cost
- High cognitive load for users
- Increased support burden
- Brand dilution
- Maintenance nightmare
- Slow plugin onboarding

</div>

</div>

<!--
SPEAKER NOTES:
- This is the fundamental tension
- More plugins = more inconsistency risk
- Users blame YOUR platform, not the plugins
-->

---

# Before vs After: The Visual Impact

<div class="columns">

<div>

### Without Strategy

- Buttons look different everywhere
- Forms behave inconsistently
- Navigation patterns vary
- Colors don't match
- Spacing is random

**Result**: Users feel lost

</div>

<div>

### With Strategy

- Unified component library
- Consistent interactions
- Predictable navigation
- Cohesive visual language
- Professional feel

**Result**: Users feel confident

</div>

</div>

<!--
SPEAKER NOTES:
- This isn't hypothetical
- I've seen platforms where each plugin feels like a different app
- Your goal: make plugins invisible to users
-->

---

<!-- _class: lead -->
<!-- _backgroundColor: #EE0000 -->
<!-- _color: white -->

# Part 2
## The Solution Framework

---

# Five Strategies for UI Consistency

```
┌─────────────────────────────────────────────────────────────────┐
│                    UI CONSISTENCY FRAMEWORK                      │
│                                                                  │
│   1. CENTRAL THEME SYSTEM                                        │
│      Single source of truth for colors, fonts, spacing           │
│                                                                  │
│   2. RUNTIME DEPENDENCY SHARING                                  │
│      Module Federation for single React/UI library instance      │
│                                                                  │
│   3. STYLE ISOLATION                                             │
│      Prevent CSS conflicts between plugins                       │
│                                                                  │
│   4. EXTENSION CONTRACTS                                         │
│      Typed APIs, mount points, component interfaces              │
│                                                                  │
│   5. CONFIGURATION-DRIVEN CUSTOMIZATION                          │
│      Theme without code changes                                  │
└─────────────────────────────────────────────────────────────────┘
```

<!--
SPEAKER NOTES:
- These 5 strategies work together
- Each solves a different aspect of the problem
- Let's dive into each one
-->

---

# Case Study: Red Hat Developer Hub

<div class="columns">

<div>

### What is RHDH?
- Internal Developer Portal
- Built on Backstage (Spotify)
- 100+ plugins available
- Enterprise deployments

</div>

<div>

### Plugin Sources
- Red Hat teams
- Backstage community
- Third-party vendors
- Customer custom plugins

</div>

</div>

**Challenge**: Make all plugins look like ONE product

We'll use RHDH examples throughout this talk.

<!--
SPEAKER NOTES:
- RHDH is our proof these patterns work
- Real production system, not a demo
- Everything I show you is from real code
-->

---

<!-- _class: lead -->
<!-- _backgroundColor: #EE0000 -->
<!-- _color: white -->

# Strategy 1
## Central Theme System

---

# The Principle

<div class="highlight">

**Key Idea**: Create a single package that controls ALL visual styles

</div>

### What It Should Control

| Category | Examples |
|----------|----------|
| **Colors** | Primary, secondary, error, warning, success |
| **Typography** | Font family, sizes, weights, line heights |
| **Spacing** | Margins, padding, gaps (4px, 8px, 16px...) |
| **Components** | Buttons, cards, tables, forms appearance |
| **Modes** | Light theme, dark theme variants |

<!--
SPEAKER NOTES:
- This is your FOUNDATION
- Get this right first, everything else builds on it
- One package, one source of truth
-->

---

# Why Centralize?

<div class="columns">

<div>

### Without Central Theme

```
Plugin A: primaryColor = "#007bff"
Plugin B: primaryColor = "#0066cc"  
Plugin C: primaryColor = "blue"
Platform: primaryColor = "#0052cc"
```

**Result**: 4 different "blues"

</div>

<div>

### With Central Theme

```typescript
// theme-package/index.ts
export const palette = {
  primary: "#0052cc",
  // ... all colors defined once
};

// Every plugin imports from here
import { palette } from '@my-org/theme';
```

**Result**: One blue everywhere

</div>

</div>

<!--
SPEAKER NOTES:
- Seems obvious, but many teams skip this
- They let plugins define their own colors
- Disaster waiting to happen
-->

---

# Implementation Pattern

```typescript
// Pattern: Theme Configuration Type
interface ThemePalette {
  // Core colors
  primary: { main: string; light: string; dark: string; };
  secondary: { main: string; light: string; dark: string; };
  
  // Semantic colors
  error: string;
  warning: string;
  success: string;
  info: string;
  
  // Component-specific tokens (20-30 total)
  sidebar: { background: string; selectedItem: string; };
  card: { background: string; border: string; };
  table: { headerBackground: string; rowHover: string; };
  appBar: { background: string; foreground: string; };
  // ...
}
```

**Tip**: Start with 20-30 tokens. Add more as needed.

<!--
SPEAKER NOTES:
- Define types for your theme
- Component-specific tokens are key
- Don't just do primary/secondary - go deeper
-->

---

# RHDH Example: Theme Plugin

```typescript
// @red-hat-developer-hub/backstage-plugin-theme
// Real code from RHDH

interface ThemeConfigOptions {
  // Choose base component styling
  components?: 'rhdh' | 'backstage' | 'mui';
  
  // Per-component overrides - mix and match!
  buttons?: 'patternfly' | 'mui';
  tables?: 'patternfly' | 'mui';
  cards?: 'patternfly' | 'mui';
  inputs?: 'patternfly' | 'mui';
  sidebars?: 'patternfly' | 'mui';
  dialogs?: 'patternfly' | 'mui';
}

// Usage in platform
const themes = useThemes(); // Central hook provides all themes
```

**Key Insight**: Platform decides styling, not plugins!

<!--
SPEAKER NOTES:
- RHDH theme plugin has 30+ color tokens
- Can mix PatternFly and MUI styles per component
- Plugins just use what's provided
-->

---

# Theme System Takeaway

<div class="highlight">

**Invest in a robust theme package early. It's your foundation.**

</div>

### Checklist

- [ ] Single package for all theme values
- [ ] 20-30 color/spacing tokens minimum
- [ ] Component-specific tokens (sidebar, card, table, etc.)
- [ ] Light and dark mode support
- [ ] TypeScript types for enforcement
- [ ] Central hook for consuming theme

<!--
SPEAKER NOTES:
- If you do nothing else, do this
- Everything else is easier with a solid theme system
-->

---

<!-- _class: lead -->
<!-- _backgroundColor: #EE0000 -->
<!-- _color: white -->

# Strategy 2
## Runtime Dependency Sharing

---

# The Problem with Bundling

<div class="columns">

<div>

### What Happens

```
Plugin A bundles: React 18.2.0
Plugin B bundles: React 18.3.0
Plugin C bundles: React 18.2.0
Platform bundles: React 18.3.1
```

</div>

<div>

### The Result

- 4 copies of React loaded
- Multiple React instances = bugs
- Huge bundle sizes
- Version conflicts
- "Invalid hook call" errors

</div>

</div>

**This is the #1 cause of plugin bugs.**

<!--
SPEAKER NOTES:
- React MUST be a singleton
- Multiple instances cause cryptic errors
- Same problem with any stateful library
-->

---

# The Solution: Module Federation

<div class="highlight">

**Key Idea**: Share core libraries at RUNTIME, not build time

</div>

### How It Works

```
┌─────────────────────────────────────────────┐
│              HOST APPLICATION               │
│                                             │
│  Provides: React, ReactDOM, MUI, Router     │
│            (loaded ONCE)                    │
└─────────────────────────────────────────────┘
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
   ┌────────┐   ┌────────┐   ┌────────┐
   │Plugin A│   │Plugin B│   │Plugin C│
   │        │   │        │   │        │
   │Uses    │   │Uses    │   │Uses    │
   │host's  │   │host's  │   │host's  │
   │React   │   │React   │   │React   │
   └────────┘   └────────┘   └────────┘
```

<!--
SPEAKER NOTES:
- Host provides shared libraries
- Plugins consume at runtime
- Single instance of everything
-->

---

# What to Share vs Bundle

| Share (Singleton from Host) | Bundle (Per Plugin) |
|-----------------------------|---------------------|
| React, ReactDOM | Plugin-specific libs |
| UI library (MUI, Chakra) | Visualization (D3, Chart.js) |
| Router (react-router) | Heavy utilities |
| State management (Redux) | Rare dependencies |
| Core platform components | |

<div class="highlight">

**Rule of Thumb**: Share anything that must be a singleton OR is used by >50% of plugins

</div>

<!--
SPEAKER NOTES:
- Don't share everything
- Plugin-specific stuff can be bundled
- Focus on the critical singletons
-->

---

# Implementation: Webpack Module Federation

```javascript
// webpack.config.js (Host Application)
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      shared: {
        react: { 
          singleton: true, 
          requiredVersion: '^18.0.0' 
        },
        'react-dom': { singleton: true },
        '@mui/material': { singleton: true },
        'react-router-dom': { singleton: true },
      },
    }),
  ],
};
```

<!--
SPEAKER NOTES:
- Webpack 5 built-in feature
- singleton: true is crucial
- requiredVersion for safety
-->

---

# RHDH Example: Scalprum

```typescript
// Scalprum = Runtime Module Federation wrapper
// From packages/app/src/components/DynamicRoot/ScalprumRoot.tsx

<ScalprumProvider
  config={scalprumConfig}
  pluginSDKOptions={{
    pluginLoaderOptions: {
      transformPluginManifest: manifest => ({
        ...manifest,
        loadScripts: manifest.loadScripts.map(
          script => `${baseUrl}/api/scalprum/${manifest.name}/${script}`
        ),
      }),
    },
  }}
>
  {/* Plugins loaded dynamically at runtime */}
  {children}
</ScalprumProvider>
```

**Plugins are served from backend API, not bundled.**

<!--
SPEAKER NOTES:
- Scalprum wraps Module Federation
- Plugins loaded on-demand
- Can update plugins without rebuilding host
-->

---

<!-- _class: lead -->
<!-- _backgroundColor: #EE0000 -->
<!-- _color: white -->

# Strategy 3
## Style Isolation

---

# The CSS Problem

**CSS is global by default.**

```css
/* Plugin A */
.MuiButton-root { background: red; }

/* Plugin B */  
.MuiButton-root { background: blue; }

/* Result: One wins, unpredictably */
```

### Common Conflicts
- Same class names from UI library
- Global styles bleeding between plugins
- Specificity wars
- !important abuse

<!--
SPEAKER NOTES:
- Even with shared MUI, CSS can conflict
- Different plugins, different overrides
- Global CSS = chaos
-->

---

# Isolation Techniques

<div class="columns">

<div>

### Technique 1: Class Prefixing

```typescript
// MUI ClassNameGenerator
import { unstable_ClassNameGenerator } 
  from '@mui/material/className';

ClassNameGenerator.configure(
  name => `plugin-tekton-${name}`
);

// Result: .plugin-tekton-MuiButton-root
```

</div>

<div>

### Technique 2: Shadow DOM

```typescript
// Encapsulate in shadow root
const shadow = element.attachShadow({ 
  mode: 'open' 
});
// Plugin renders inside
// Styles don't leak out
```

</div>

</div>

<!--
SPEAKER NOTES:
- Class prefixing is most common
- Shadow DOM is more isolated but harder
- Choose based on your needs
-->

---

# More Isolation Techniques

<div class="columns">

<div>

### Technique 3: CSS Modules

```css
/* styles.module.css */
.button { 
  background: blue; 
}
/* Becomes: .button_abc123 */
```

```typescript
import styles from './styles.module.css';
<button className={styles.button} />
```

</div>

<div>

### Technique 4: Naming Conventions

```css
/* BEM with plugin prefix */
.tekton__button--primary { }
.tekton__card--highlighted { }
.rbac__table--striped { }
```

**Discipline required!**

</div>

</div>

<!--
SPEAKER NOTES:
- CSS Modules work at build time
- BEM requires discipline but works
- Pick one and stick with it
-->

---

# RHDH Approach: Plugin Wrappers

```typescript
// dynamic-plugins/wrappers/backstage-community-plugin-tekton/src/index.ts
// Used in 14+ plugin wrappers

import { unstable_ClassNameGenerator as ClassNameGenerator } 
  from '@mui/material/className';

// Prefix all MUI class names for this plugin
ClassNameGenerator.configure(componentName => 
  componentName.startsWith('v5-') 
    ? componentName 
    : `v5-${componentName}`
);

// Re-export the original plugin
export * from '@backstage-community/plugin-tekton';
```

**Wrapper pattern**: Add isolation without modifying original plugin.

<!--
SPEAKER NOTES:
- Wrappers are thin layers
- Add CSS isolation
- Don't modify upstream code
-->

---

<!-- _class: lead -->
<!-- _backgroundColor: #EE0000 -->
<!-- _color: white -->

# Strategy 4
## Extension Contracts

---

# The Principle

<div class="highlight">

**Key Idea**: Define clear boundaries for how plugins integrate

</div>

<div class="columns">

<div>

### Without Contracts
- Plugins modify anything
- Breaking changes common
- No predictable structure
- Integration is fragile

</div>

<div>

### With Contracts
- Defined extension points
- Platform controls layout
- Stable integration
- Plugins are predictable

</div>

</div>

<!--
SPEAKER NOTES:
- Contracts = rules of engagement
- Platform defines what's possible
- Plugins work within those boundaries
-->

---

# Extension Point Types

### 1. Mount Points (Slots)

```typescript
// Platform defines named slots
<PageLayout>
  <Slot name="page.header" />
  <Slot name="page.sidebar" />
  <Slot name="page.main">
    <Slot name="entity.overview" />  {/* Plugins inject here */}
    <Slot name="entity.details" />
  </Slot>
</PageLayout>

// Plugin registers for a slot
registerComponent('entity.overview', MyOverviewCard);
```

**Platform controls structure. Plugins provide content.**

<!--
SPEAKER NOTES:
- Named slots = predictable locations
- Plugins can't break layout
- Easy to add/remove plugins
-->

---

# More Extension Points

### 2. Dynamic Routes

```yaml
# Plugin declares routes via config
dynamicRoutes:
  - path: /my-plugin
    importName: MyPluginPage
    menuItem:
      icon: MyIcon
      text: "My Plugin"
```

### 3. API Extensions

```typescript
// Plugin provides or extends APIs
createApiFactory({
  api: myApiRef,
  deps: { configApi: configApiRef },
  factory: ({ configApi }) => new MyApiImpl(configApi),
})
```

<!--
SPEAKER NOTES:
- Routes declared, not hardcoded
- APIs extend platform capabilities
- All via defined contracts
-->

---

# TypeScript for Enforcement

```typescript
// Platform defines the contract type
interface MountPointComponent {
  Component: React.ComponentType<MountPointProps>;
  config?: {
    layout?: { gridColumn?: string };
    if?: (entity: Entity) => boolean;
  };
}

// Plugins MUST conform to this type
const MyCard: MountPointComponent = {
  Component: TektonOverview,
  config: { 
    layout: { gridColumn: '1 / -1' },
    if: entity => entity.kind === 'Component',
  },
};
```

**TypeScript catches violations at build time.**

<!--
SPEAKER NOTES:
- Types enforce contracts
- Compile-time errors, not runtime
- Documentation as code
-->

---

# RHDH Example: YAML Configuration

```yaml
# app-config.yaml - No code changes needed!
dynamicPlugins:
  frontend:
    backstage-community.plugin-tekton:
      mountPoints:
        - mountPoint: entity.page.ci-cd
          importName: TektonCI
          config:
            layout: { gridColumn: "1 / -1" }
            if:
              allOf:
                - isKind: component
      dynamicRoutes:
        - path: /tekton
          importName: TektonPage
          menuItem:
            icon: TektonIcon
            text: Tekton
```

**Add or remove plugins by editing YAML.**

<!--
SPEAKER NOTES:
- Declarative plugin configuration
- Operations can manage plugins
- No rebuilds required
-->

---

<!-- _class: lead -->
<!-- _backgroundColor: #EE0000 -->
<!-- _color: white -->

# Strategy 5
## Configuration-Driven Customization

---

# The Principle

<div class="highlight">

**Key Idea**: Enable theming and customization WITHOUT code changes

</div>

### Why Configuration Over Code?

| Code Changes | Configuration |
|--------------|---------------|
| Requires developers | Ops teams can do it |
| Needs rebuild/deploy | Runtime changes |
| Hard to A/B test | Easy to experiment |
| One version | Multi-tenant friendly |

<!--
SPEAKER NOTES:
- Configuration = flexibility
- Different customers, different brands
- No code changes = faster iteration
-->

---

# What Should Be Configurable?

```yaml
# app-config.yaml
app:
  branding:
    # Logos (light/dark variants)
    fullLogo:
      light: "data:image/svg+xml,..."
      dark: "data:image/svg+xml,..."
    
    # Theme colors
    theme:
      light:
        primaryColor: "#2A61A7"
        sidebarBackground: "#ffffff"
      dark:
        primaryColor: "#DC6ED9"
        sidebarBackground: "#1a1a1a"
    
    # Feature flags
    features:
      darkMode: true
      compactSidebar: false
```

<!--
SPEAKER NOTES:
- Logos, colors, features
- Light and dark variants
- Feature flags for gradual rollout
-->

---

# Runtime Theme Application

```typescript
// Hooks for accessing themed values
export const useSidebarBackground = () => {
  const theme = useTheme();
  return theme?.palette?.sidebar?.background ?? '#ffffff';
};

// System theme detection
export const useSystemTheme = () => {
  const [mode, setMode] = useState<'light' | 'dark'>('light');
  
  useEffect(() => {
    const query = window.matchMedia('(prefers-color-scheme: dark)');
    setMode(query.matches ? 'dark' : 'light');
    query.addEventListener('change', e => setMode(e.matches ? 'dark' : 'light'));
  }, []);
  
  return mode;
};
```

**Hooks abstract configuration access.**

<!--
SPEAKER NOTES:
- Components use hooks
- Hooks read from config
- System theme detection built-in
-->

---

<!-- _class: lead -->
<!-- _backgroundColor: #EE0000 -->
<!-- _color: white -->

# Putting It All Together

---

# Complete Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      YOUR PLATFORM                               │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                 THEME PACKAGE                            │    │
│  │  • Color tokens    • Typography    • Component styles    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              MODULE FEDERATION HOST                      │    │
│  │  • Shared: React, UI lib, Router, Core components        │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              EXTENSION POINT SYSTEM                      │    │
│  │  • Mount points    • Routes    • API factories           │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │ Plugin A │  │ Plugin B │  │ Plugin C │  │ Plugin D │        │
│  │ Wrapper  │  │ Wrapper  │  │ Wrapper  │  │ Wrapper  │        │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │
└─────────────────────────────────────────────────────────────────┘
```

<!--
SPEAKER NOTES:
- Theme at top - foundation
- Module federation - sharing
- Extension points - contracts
- Wrappers - isolation
-->

---

# The Plugin Wrapper Pattern

```
plugin-wrapper/
├── package.json       # Declares shared deps, exports
├── src/
│   └── index.ts       # CSS isolation + re-export
└── dist/              # Built assets
```

```typescript
// src/index.ts
import { ClassNameGenerator } from '@mui/material/className';

// Add CSS isolation
ClassNameGenerator.configure(name => `my-plugin-${name}`);

// Re-export original plugin unchanged
export * from 'original-plugin';
```

**Wrappers let you integrate third-party plugins without forking.**

<!--
SPEAKER NOTES:
- Thin wrapper layer
- CSS isolation
- No changes to original
-->

---

# Implementation Roadmap

<div class="columns">

<div>

### Phase 1: Foundation
- [ ] Create theme package
- [ ] Set up Module Federation
- [ ] Define initial mount points

### Phase 2: Isolation
- [ ] CSS isolation strategy
- [ ] Plugin wrapper template
- [ ] Document contracts

</div>

<div>

### Phase 3: Configuration
- [ ] YAML-based theming
- [ ] Theme hooks
- [ ] Light/dark modes

### Phase 4: Enforcement
- [ ] TypeScript contracts
- [ ] Linting rules
- [ ] Visual regression tests

</div>

</div>

<!--
SPEAKER NOTES:
- Don't do everything at once
- Foundation first
- Iterate and improve
-->

---

# Common Pitfalls to Avoid

| Pitfall | Why It's Bad | Solution |
|---------|--------------|----------|
| No theme system | Inconsistent colors/fonts | Start with theme package |
| Bundling shared deps | Multiple React instances | Use Module Federation |
| No CSS isolation | Style conflicts | Prefix classes or Shadow DOM |
| No contracts | Plugins break things | TypeScript + mount points |
| Code-only config | Slow changes, dev bottleneck | YAML/JSON configuration |

<!--
SPEAKER NOTES:
- Learn from others' mistakes
- These are real problems we've seen
- Each has a solution
-->

---

<!-- _class: lead -->
<!-- _backgroundColor: #EE0000 -->
<!-- _color: white -->

# Results & Closing

---

# Metrics from RHDH

| Metric | Before | After |
|--------|--------|-------|
| Plugin onboarding time | 2-3 weeks | 2-3 days |
| UI inconsistency bugs | ~15/month | ~2/month |
| Bundle size (5 plugins) | 2.8 MB | 850 KB |
| Design review rounds | 4-5 | 1-2 |
| Theme change time | Days (code) | Minutes (config) |

<div class="highlight">

**100+ plugins** maintaining consistent UX in production

</div>

<!--
SPEAKER NOTES:
- Real metrics from production
- Dramatic improvements
- These strategies work
-->

---

# Key Takeaways

<div style="font-size: 0.9em">

### 1. Invest in a Central Theme System early
It's your foundation - colors, fonts, spacing, component tokens

### 2. Use Module Federation for runtime sharing
Single React/UI library instance - no more "invalid hook call"

### 3. Implement Style Isolation
CSS conflicts are inevitable - prefix classes or use Shadow DOM

### 4. Define Clear Extension Contracts
TypeScript + mount points = predictable plugin integration

### 5. Make It Configuration-Driven
Ops teams will thank you - YAML > code for theming

</div>

<!--
SPEAKER NOTES:
- Five strategies, five actions
- Start with theme system
- Build up from there
-->

---

<!-- _class: lead -->
<!-- _backgroundColor: #151515 -->
<!-- _color: white -->

# Final Thought

> "A good design system in a plugin architecture is **invisible**. Users see one cohesive application. Developers build features without fighting consistency."

---

# Resources

<div class="columns">

<div>

### Open Source Examples
- **Backstage**: backstage.io
- **RHDH**: github.com/redhat-developer/rhdh
- **Module Federation**: module-federation.io

### Further Reading
- "Micro Frontends in Action"
- "Design Systems" by Alla Kholmatova

</div>

<div>

### Connect With Me
- **GitHub**: @its-mitesh-kumar
- **LinkedIn**: /in/miteshkumar
- **Twitter/X**: @miteshkumar

### Slides & Code
github.com/its-mitesh-kumar/devconf-2026-design-systems

</div>

</div>

<!--
SPEAKER NOTES:
- Links to learn more
- Happy to connect
- Slides will be available
-->

---

<!-- _class: lead -->
<!-- _backgroundColor: #EE0000 -->
<!-- _color: white -->

# Thank You!

## Questions?

Mitesh Kumar
Frontend Contributor, Red Hat Developer Hub

@its-mitesh-kumar

<!--
SPEAKER NOTES:
- Thank the audience
- Open for questions
- Available after the talk
-->
