# Design Systems at Scale: Detailed Speaker Notes and Guide

This document expands every section of the speaker script ([speaker.md](speaker.md)) with full context, explanations, technical detail, and talking points. Use it for deep preparation, backup notes, or as a standalone reference for the presentation content.

---

## Presentation Metadata

- **Event:** DevConf.IN 2026
- **Track:** Track 4: User Experience and Design Engineering
- **Title:** Design Systems at Scale: Maintaining UI Consistency in Dynamic Plugin Architectures
- **Duration:** 40 minutes presentation + 5 minutes Q&A (45 minutes total)
- **Speaker:** Mitesh Kumar (Frontend Contributor, Red Hat Developer Hub)
- **Deck:** 39 slides ([slides-diagram-heavy.md](slides-diagram-heavy.md))

---

## Pre-Talk Checklist (Detailed)

| Item | Why it matters |
|------|----------------|
| **Test slides on presentation system** | Different projectors and aspect ratios can change layout; Marp/PDF may render differently. Verify diagrams (Mermaid), fonts, and slide numbers display correctly. |
| **Have RHDH running locally for demos (optional)** | If you plan to show a live developer portal, ensure the app and plugin load without network or CORS issues in the venue. |
| **Backup slides as PDF** | If the primary system fails, a PDF avoids dependency on Marp or browser. Export from Marp before the talk. |
| **Water bottle ready** | 40 minutes of speaking is demanding; staying hydrated helps clarity and reduces throat strain. |
| **Timer visible** | Keeps you on pace. Aim to hit the Key Takeaways (Slide 36) by ~38 minutes so you have buffer for Final Thought, Resources, and Q&A. |

---

## Part 1: Opening and Agenda (Slides 1–2)

### SLIDE 1: Title (1 minute)

**What you say (script):**  
Good [morning/afternoon] everyone, and welcome to "Design Systems at Scale: Maintaining UI Consistency in Dynamic Plugin Architectures." I'm Mitesh Kumar, and I'm a frontend contributor to Red Hat Developer Hub. Today, I want to share a practical guide for anyone building plugin-based platforms. Whether you're working on an IDE, a developer portal, or any extensible application—the challenges we'll discuss are universal.

**Detailed context:**

- **Purpose of the opening:** Set the frame: this is a *practical* guide, not theory. The phrase "dynamic plugin architectures" signals that the focus is on systems where many independent pieces (plugins) are composed at runtime.
- **Audience framing:** "IDE, developer portal, or any extensible application" broadens relevance—attendees from VS Code–style tools, internal portals, SaaS with app marketplaces, or headless CMS with plugins all face similar design-system and consistency challenges.
- **"Universal" challenges:** Implies that the patterns you’ll show (theme system, Module Federation, style isolation, contracts, configuration) transfer across stacks and domains.
- **Delivery note:** Walk to center stage, pause, make eye contact before speaking. Adjust "morning/afternoon" to the actual slot.

---

### SLIDE 2: What You'll Learn (1 minute)

**What you say (script):**  
Here's what we'll cover. On the left—the challenge: why UI consistency is hard when you have dozens or hundreds of plugins from different teams. On the right—the solutions: five proven strategies with real implementation patterns you can take back to your teams. We'll use Red Hat Developer Hub as our case study—a production system with 100+ plugins—but the patterns apply to any plugin architecture.

**Detailed context:**

- **Left (challenge):** The core tension is *many owners, one product*. When each plugin team ships independently, they make different choices for colors, spacing, components, and behavior. Without a deliberate strategy, the product feels fragmented.
- **Right (solutions):** The five strategies (Central Theme, Runtime Dependency Sharing, Style Isolation, Extension Contracts, Configuration-Driven Customization) are introduced here at a high level; each gets a dedicated section later.
- **RHDH as case study:** Red Hat Developer Hub is built on Backstage and runs 100+ plugins in production. Citing it gives credibility and sets the expectation that code and patterns are real, not toy examples.
- **"Take back to your teams":** Emphasizes actionable, reusable patterns rather than a one-off solution.

---

## Part 2: The Challenge (Slides 3–6)

### SLIDE 3: What is Plugin Architectures? (1 minute)

**What you say (script):**  
A plugin architecture is a core or host application that others extend with plugins. The host provides the foundation. Plugins add features—Plugin A, Plugin B, Plugin N—without changing the core. You get one application with many capabilities, built by different teams or vendors, all running together.

**Detailed context:**

- **Definition:** A *host* (or core) application exposes extension points; *plugins* (or extensions) register and run inside that host. The host is responsible for loading, lifecycle, and often shared dependencies; plugins provide UI, APIs, or behavior.
- **Why this slide:** It grounds the rest of the talk. Without a shared understanding of "plugin architecture," the consistency problem and the five strategies are harder to motivate.
- **Diagram reference:** The slide shows Host/Core application with Plugin A, B, N attached—use it to point at "one app, many capabilities" and "different teams or vendors."
- **Contrast with monolith:** In a monolith, one codebase and one team own the UI. In a plugin architecture, many codebases and many teams contribute; the host must coordinate them so the product still feels like one app.

---

### SLIDE 4: Part 1: Why Plugin Architectures? (2 minutes)

**What you say (script):**  
Let's start with the universal challenge. Quick show of hands—how many of you work on applications with some kind of plugin or extension system? [Pause.] That's a lot! And there's a good reason. Plugin architectures are everywhere. Look at the success stories: VS Code has over 40,000 extensions, Figma has 2,000+ plugins, WordPress has 60,000+ plugins. Why? Because plugin architectures solve a fundamental business problem: **extensibility without bottlenecking on the core team**. Third parties can build features. Independent teams can move at their own pace. You get an ecosystem instead of a monolith. But this power comes with a challenge...

**Detailed context:**

- **Show of hands:** Engages the room and surfaces how many people are in the same situation. It also sets up "we're all in this together."
- **Success stories:** Concrete numbers (40k, 2k, 60k) make the trend tangible. You can add Shopify (8k+ apps), JetBrains IDEs, Slack apps, etc., if time allows.
- **Business problem:** "Extensibility without bottlenecking" means the core team doesn’t have to implement every feature; external or internal teams can ship value without waiting for a central roadmap. That scales both delivery and ecosystem.
- **Ecosystem vs monolith:** Monolith = one team, one release train. Ecosystem = many teams, many release trains, but one product experience. The rest of the talk is about how to preserve "one product experience" despite many teams.
- **"But this power comes with a challenge":** Direct transition to Slide 5 (The Consistency Problem).

---

### SLIDE 5: The Consistency Problem (2 minutes)

**What you say (script):**  
"Your users don't care that your UI is composed of 50 different plugins. They expect it to feel like ONE application." This is the fundamental tension. On the left—what goes wrong: different teams make different design decisions; third-party plugins ignore your guidelines; dependencies drift out of sync; CSS conflicts between plugins; inconsistent accessibility. On the right—the cost: users get confused; support tickets go up; your brand gets diluted; maintenance becomes a nightmare. And here's the worst part: **users blame YOUR platform, not the plugins**.

**Detailed context:**

- **Quote:** This line should be delivered with emphasis. It captures the product and design imperative: the system may be modular, but the experience must be unified.
- **What goes wrong (left):**
  - **Different design decisions:** Team A uses 8px spacing, Team B uses 12px; Team A uses rounded buttons, Team B uses square—no single source of truth.
  - **Third-party plugins ignore guidelines:** External vendors may not have access to or incentive to follow your design system.
  - **Dependencies drift:** Plugin 1 uses React 18.1, Plugin 2 uses 18.2; UI library versions diverge, leading to visual and behavioral differences.
  - **CSS conflicts:** Global CSS from one plugin overrides another; specificity and load order become unpredictable.
  - **Inconsistent accessibility:** Contrast, focus management, and ARIA may vary across plugins, hurting compliance and usability.
- **The cost (right):** Confusion and support load increase; brand perception suffers when the product looks patched together; maintenance becomes reactive (fixing one plugin breaks another).
- **"Users blame YOUR platform":** Critical point. End users don’t distinguish "plugin bug" vs "platform bug"—they blame the product and the team that owns it. So the platform team has a strong incentive to enforce consistency even for third-party code.

---

### SLIDE 6: Before vs After (1 minute)

**What you say (script):**  
Visualize the difference. Without a strategy: buttons look different everywhere, forms behave inconsistently, navigation varies. Users feel lost. With a strategy: unified components, consistent interactions, cohesive visual language. Users feel confident. Your goal is to make plugins invisible to users. They should never think "this feels like a different app."

**Detailed context:**

- **Before (no strategy):** Inconsistent controls, layouts, and patterns force users to relearn the UI in different areas. Cognitive load goes up; trust and perceived quality go down.
- **After (with strategy):** One design language, predictable patterns, and consistent behavior. Users can transfer learning from one part of the app to another.
- **"Plugins invisible":** Success means the user doesn’t need to know or care that the screen is built from 20 plugins; it should feel like one product. This is the design and UX goal that the five strategies support.

---

## Part 3: The Five Strategies Overview and Case Study (Slides 7–8)

### SLIDE 7: Part 2: Five Strategies (2 minutes)

**What you say (script):**  
Now let's look at the solution framework. Here are the five strategies that work together to solve UI consistency: One—Central Theme System: a single source of truth for colors, fonts, spacing. This is your foundation. Two—Runtime Dependency Sharing: using Module Federation so all plugins share ONE React instance, ONE UI library instance. Three—Style Isolation: preventing CSS from one plugin from breaking another. Four—Extension Contracts: defining clear rules for how plugins integrate—mount points, typed APIs. Five—Configuration-Driven Customization: enabling theming without code changes. We'll dive deep into each of these.

**Detailed context:**

- **Framework, not silver bullet:** The word "framework" signals that these five work as a system. Skipping one (e.g., theme but no isolation) can leave gaps.
- **One: Central Theme:** Single package or module that defines colors, typography, spacing, and component tokens. All plugins consume from it so there are no competing definitions.
- **Two: Runtime Dependency Sharing:** Ensures React, ReactDOM, and the UI library are loaded once by the host and consumed by plugins at runtime (e.g., via Webpack Module Federation). Avoids duplicate instances and "Invalid hook call"–type bugs.
- **Three: Style Isolation:** Keeps one plugin’s CSS from affecting another (e.g., class-name prefixing, Shadow DOM, or CSS-in-JS with scoping).
- **Four: Extension Contracts:** The platform defines *where* and *how* plugins can plug in (mount points, routes, APIs) and enforces those contracts (e.g., with TypeScript and runtime checks).
- **Five: Configuration-Driven Customization:** Theming, feature flags, and branding are driven by config (e.g., YAML/JSON) so ops or product can change look and behavior without code deploys.

---

### SLIDE 8: Case Study Introduction (1 minute)

**What you say (script):**  
Throughout this talk, I'll use Red Hat Developer Hub as our case study. RHDH is an internal developer portal built on Backstage—Spotify's open platform. We have 100+ plugins from Red Hat teams, the Backstage community, and third-party vendors. Our challenge was making all these plugins look like ONE product. Everything I'll show you comes from real production code.

**Detailed context:**

- **RHDH:** Red Hat Developer Hub is an internal developer portal (IDP) that helps teams discover, manage, and operate services and apps. It is built on Backstage (open source).
- **100+ plugins:** Mix of first-party (Red Hat), community (Backstage ecosystem), and third-party or partner plugins. This variety is exactly where consistency is hardest.
- **"Making all these plugins look like ONE product":** The design and engineering goal—one visual and interaction language despite many sources.
- **"Real production code":** Builds trust; the patterns you describe are not theoretical but in use at scale.

---

## Part 4: Strategy 1 – Central Theme System (Slides 9–13)

### SLIDE 9: Strategy 1: Central Theme System – The Principle (2 minutes)

**What you say (script):**  
Strategy One: Central Theme System. This is where you should start. The key idea is simple: **create a single package that controls ALL visual styles**. What should it control? Colors—primary, secondary, status colors. Typography—font family, sizes, weights. Spacing—your 4px, 8px, 16px system. Component appearance—how buttons, cards, tables look. Light and dark mode variants. This is your foundation. Get this right first.

**Detailed context:**

- **Single package:** One npm package (or monorepo package) that every plugin and the host depend on. That package exports tokens, theme objects, and optionally component overrides.
- **Colors:** Include primary/secondary brand colors and semantic colors (error, warning, success, info) so plugins don’t invent their own.
- **Typography:** Font family, scale (e.g., 12/14/16/20/24px), weights. Ensures readable, consistent text across plugins.
- **Spacing:** A shared scale (e.g., 4, 8, 16, 24, 32) for margins and padding avoids arbitrary values.
- **Component appearance:** How buttons, cards, tables, inputs look (and optionally behave) so that no plugin ships a completely different button style.
- **Light and dark mode:** Tokens or variants for both so the whole app can switch coherently.
- **"Get this right first":** Theme is the base; Module Federation and isolation build on top of a consistent token set.

---

### SLIDE 10: Why Centralize (2 minutes)

**What you say (script):**  
Here's what happens without a central theme: Plugin A uses one shade of blue. Plugin B uses another. Plugin C just says "blue". The platform uses a fourth shade. Result? Four different "blues" in your UI. Users notice. With a central theme package, there's one definition of "primary blue." Every plugin imports from the same source. One blue everywhere. This seems obvious, but many teams skip it. They let plugins define their own colors. It's a disaster waiting to happen.

**Detailed context:**

- **Without central theme:** Each plugin (or team) picks its own hex for "primary" or "blue." Even small differences (e.g., #0052CC vs #0066CC) are visible side by side and feel broken.
- **With central theme:** One palette object (e.g., `theme.palette.primary.main`) is imported by all. Design and code stay in sync; design reviews can reference the same tokens.
- **"Users notice":** Non-technical users may not say "inconsistent primaries," but they perceive the product as unpolished or unreliable.
- **"Many teams skip it":** Often due to time pressure or "we’ll fix it later." Later rarely comes; technical debt compounds. Stressing this encourages investing in the theme package early.

---

### SLIDE 11: Implementation Pattern (theme) (2 minutes)

**What you say (script):**  
Here's the pattern. Define a TypeScript interface for your theme palette. Start with core colors—primary, secondary. Add semantic colors—error, warning, success, info. Then—and this is crucial—add **component-specific tokens**. Sidebar background, card borders, table headers. Aim for 20-30 tokens to start. You'll add more as needed. The key is defining everything in one place.

**Detailed explanation:**

## 1. TypeScript interface

**What it is:** A single type that describes the exact shape of your theme object (all keys and their types).

**Why it matters for host + plugins:**

- **Type checking:** Any code that uses the theme (host app or any plugin) must use the same shape. Typos like `theme.colors.eror` or `theme.sidebar.bg` are caught at compile time.
- **Autocomplete:** IDEs can suggest `theme.colors.primary.main`, `theme.colors.error`, `theme.sidebar.background`, etc., so plugin authors don’t have to remember or guess names.
- **Safer refactors:** If you rename `sidebar.background` to `sidebar.bg` or add a new token, TypeScript shows every consumer that must be updated. You avoid “runtime surprise” when a plugin uses an old key.

So the interface is the **contract**: “everyone gets the same theme shape, and the compiler enforces it.”

---

## 2. Core colors

**What they are:** The main brand and hierarchy colors, usually:

- **Primary** – main brand color (e.g. primary actions, key UI).
- **Secondary** – supporting brand color (e.g. secondary actions, accents).
- **Variants** – typically `main`, `light`, `dark` (and sometimes `contrastText`) so you can use the same “concept” in different contexts (buttons, hover, disabled, etc.).

**Why they matter:** They give a consistent brand look and a clear visual hierarchy. Plugins use `theme.colors.primary.main` instead of a random blue; if the brand color changes, you change it in one place and all plugins follow.

---

## 3. Semantic colors

**What they are:** Colors tied to **meaning**, not to a specific hex:

- **Error** – red, for errors, destructive actions, validation failures.
- **Warning** – amber/orange, for cautions and non-blocking issues.
- **Success** – green, for success states and confirmations.
- **Info** – blue, for neutral information and hints.

**Why plugins should use these instead of hardcoding hex:**

- **Consistency:** All error states look the same across host and plugins.
- **Accessibility:** Semantic colors can be tuned once for contrast and WCAG.
- **Theming:** Light/dark or brand themes can change the actual hex; plugins stay correct because they only reference `theme.colors.error`, etc.
- **Maintainability:** No scattered `#FF0000` or `#d32f2f`; one source of truth.

So: “we have a theme” becomes “we have a shared language for status and feedback.”

---

## 4. Component-specific tokens

**What they are:** Named tokens for **specific UI surfaces**, not just “a color.” Examples from your slides:

- `sidebar.background`, `sidebar.selectedItem`
- `card.background`, `card.border`
- `table.headerBackground`, `table.rowHover`
- `appBar.background`, `appBar.foreground`

**Why they matter:**

- **No guessing:** Plugins don’t have to invent “a gray for the sidebar” or “a border color for cards.” They use `theme.sidebar.background` and `theme.card.border`. The platform defines what “sidebar” and “card” look like.
- **Shared surfaces:** Sidebars, cards, and tables look the same whether rendered by the host or a plugin, because everyone uses the same tokens.
- **Design system vs “just a theme”:** A theme with only primary/secondary/semantic is still generic. Component tokens answer: “What color is the sidebar? The card border? The table header?” That’s what turns a palette into a **design system** – shared, named decisions for concrete parts of the UI.

So: **theme** = “we have colors”; **design system** = “we have colors and rules for where they go (sidebar, card, table, …).”

---

## 5. 20–30 tokens (and growing to 50+)

**What it means:** A practical **starting set** of tokens (e.g. core + semantic + a first wave of component tokens) that lands in the 20–30 range. You can grow to 50+ as you add more components and states (e.g. disabled, focus, skeleton, more surfaces).

**Why not start too small:**

- If you only expose **primary** and **secondary**, plugins still need backgrounds, borders, headers, hover states. They’ll either:
  - Hardcode values (e.g. `#f5f5f5` for background), or
  - Reuse primary/secondary for everything (everything looks the same, or wrong).
- So “too few tokens” doesn’t reduce choices; it pushes **one-off, inconsistent** choices into every plugin.

**Why 20–30 is a good start:**

- Enough to cover: core colors (with variants), semantic colors, and a small set of component tokens (e.g. sidebar, card, table, app bar, maybe buttons/inputs).
- Not so many that you’re designing tokens for components you don’t have yet.
- You can add tokens as you add components (e.g. modal, drawer, tabs) and states (hover, disabled, focus), and grow toward 50+ over time.

So: **20–30 tokens** = “enough to be a real design system from day one”; **grow to 50+** = “expand as the UI and components grow.”

---

## Summary

| Concept | Role |
|--------|------|
| **TypeScript interface** | Single contract for theme shape → type safety, autocomplete, safe refactors for host and all plugins. |
| **Core colors** | Brand and hierarchy (primary, secondary + main/light/dark). |
| **Semantic colors** | Meaning-based colors (error, warning, success, info) so plugins don’t hardcode hex. |
| **Component-specific tokens** | Named values for concrete surfaces (sidebar, card, table, …) so no one invents their own; this is what makes it a design system. |
| **20–30 tokens** | Practical starting set; grow to 50+ as you add components and states; starting too small leads to one-off values in plugins. |

Together, this is the pattern your slides recommend: **one typed theme, core + semantic + component tokens, and a starting set of 20–30 tokens** so the platform (and RHDH’s theme plugin / `useThemes()`) truly controls look and feel while plugins stay consistent and maintainable.

---

### SLIDE 12: RHDH Example (theme) (2 minutes)

**What you say (script):**  
In RHDH, we have a dedicated theme plugin with 30+ color tokens. What's interesting is the `ThemeConfigOptions` type. See how it lets you choose between 'patternfly' and 'mui' styling **per component**? Buttons can be PatternFly style. Tables can be MUI style. The platform decides—not the plugins. This gives us flexibility while maintaining control. The `useThemes()` hook is how the platform accesses all theme data centrally.

**Detailed context:**

- **Dedicated theme plugin:** In Backstage/RHDH, the theme is itself a plugin that provides tokens and optional component overrides. Other plugins and the app shell consume it.
- **ThemeConfigOptions / per-component styling:** RHDH supports both PatternFly and MUI. Instead of forcing one library everywhere, the platform can say "buttons use PatternFly, tables use MUI" so that existing plugins can be gradually aligned without a full rewrite.
- **Platform decides:** Centralizes control. Plugins don’t choose their own component library or theme variant; the platform config does. That keeps the visual language consistent.
- **useThemes() hook:** A single entry point for theme data (palette, typography, component defaults). Components and plugins call this instead of importing raw tokens from multiple places.

---

### SLIDE 13: Takeaway (theme) (30 seconds)

**What you say (script):**  
If you take nothing else from this talk: **invest in a robust theme package early. It's your foundation.** Here's a quick checklist. Single package. 20-30 tokens minimum. Component-specific tokens. Light and dark mode. TypeScript types. Central hook.

**Detailed context:**

- **Single message:** The one thing you want the audience to remember from Strategy 1 is "theme package early."
- **Checklist:** Single package; 20-30 tokens minimum; component-specific tokens; light and dark; TypeScript types; central hook (or similar API). This is a repeatable recipe teams can adopt.

---

## Part 5: Strategy 2 – Runtime Dependency Sharing (Slides 14–18)

### SLIDE 14: Strategy 2: Runtime Dependency Sharing – The Problem with Bundling (2 minutes)

**What you say (script):**  
Strategy Two: Runtime Dependency Sharing. This is the number one cause of plugin bugs. If each plugin bundles its own React, you end up with multiple React instances running at the same time. The result? Four copies of React loaded. Hooks don't work across instances. You get cryptic "Invalid hook call" errors. Bundle sizes explode. React MUST be a singleton. Same with any stateful library.

**Detailed context:**

- **Why multiple instances happen:** Each plugin is often built as its own bundle. If each bundle includes React (and ReactDOM), the browser loads several copies. React’s internals (e.g., the hook dispatcher) assume a single instance; crossing instance boundaries breaks hooks and can cause subtle bugs.
- **"Invalid hook call":** A common React error when a component from one React instance is rendered in a tree owned by another instance. It’s confusing because the code looks correct; the issue is at the runtime/boundary level.
- **Bundle size:** Duplicating React, ReactDOM, and a UI library across 10 plugins can add megabytes. Sharing one copy reduces total size and improves load time.
- **Stateful libraries:** Redux, React Router, or any library that keeps internal state must also be a singleton when used across the host and plugins; otherwise state and context don’t align.

---

### SLIDE 15: The Solution (Module Federation) (2 minutes)

**What you say (script):**  
The solution is Module Federation—sharing libraries at RUNTIME, not build time. The host application provides React, ReactDOM, MUI, Router—loaded once. Plugins consume these from the host at runtime. They don't bundle their own copies. Single instance of everything. Problem solved.

**Detailed context:**

- **Module Federation:** A Webpack 5 feature (and similar concepts in other bundlers) that lets the host expose modules (e.g., `react`, `react-dom`) and plugins declare them as "shared" and consume from the host at runtime. Build time: plugins don’t bundle React. Runtime: one React for the whole app.
- **Loaded once:** The host loads React (and other shared deps) once; plugin chunks reference that same instance when they mount. So you get a single React tree and a single hook dispatcher.
- **"Problem solved":** Hooks work across host and plugins; bundle size drops; version drift is managed in one place (the host).

---

### SLIDE 16: What to Share (1 minute)

**What you say (script):**  
What should you share? React, ReactDOM—absolutely. UI library—yes. Router—yes. State management—yes. Core platform components—yes. What should plugins bundle themselves? Plugin-specific libraries. Visualization tools like D3. Heavy utilities only they use. Rule of thumb: share anything that must be a singleton OR is used by more than half your plugins.

**Detailed context:**

- **Share:** React, ReactDOM (singleton requirement). UI library (MUI, Chakra, etc.) so components look consistent and you don’t duplicate it. Router so navigation is unified. State management (e.g., Redux store) so plugins can participate in app state. Core platform components (e.g., layout, header) so plugins slot into the same shell.
- **Don’t share (plugin bundles):** D3, Chart.js, or other heavy/vendor-specific libs that only one or two plugins use. Plugin-specific business logic or API clients can stay in the plugin bundle.
- **Singleton OR >50% use:** If it must be a singleton (React, router), share it. If most plugins use it (e.g., UI library), sharing reduces size and keeps versions aligned.

---

### SLIDE 17: Implementation (Webpack Module Federation) (2 minutes)

**What you say (script):**  
Here's how to set it up with Webpack Module Federation. In your webpack config, add the ModuleFederationPlugin. Define your shared dependencies. The `singleton: true` flag is crucial—it forces a single instance. `requiredVersion` adds safety—it'll warn if a plugin expects an incompatible version. This is Webpack 5's built-in feature. No extra libraries needed.

**Detailed context:**

- **ModuleFederationPlugin:** In the host’s webpack config, you list `shared: { react: { singleton: true, requiredVersion: '^18.0.0' }, 'react-dom': { singleton: true }, ... }`. In each plugin’s config, you mark the same dependencies as shared so they’re not bundled.
- **singleton: true:** Tells Webpack to resolve to exactly one instance. Without it, you might still get multiple copies in some scenarios.
- **requiredVersion:** Ensures that if a plugin was built against React 17 and the host has React 18, you get a warning or error instead of silent misbehavior.
- **Webpack 5 built-in:** No need for extra plugins or custom loaders; Module Federation is part of Webpack 5.

---

### SLIDE 18: RHDH Scalprum (1 minute)

**What you say (script):**  
RHDH uses Scalprum, which wraps Module Federation with some nice features. See how we transform plugin manifests to load scripts from our backend API? This means plugins are served dynamically—we can update them without rebuilding the host application.

**Detailed context:**

- **Scalprum:** A layer on top of Module Federation used in RHDH. It handles plugin discovery, manifest loading, and script URLs.
- **Backend API:** Plugin assets (JS chunks) can be served from your backend instead of a static CDN. That allows dynamic plugin registration, A/B testing, or tenant-specific plugin sets without redeploying the host.
- **Update plugins without rebuilding host:** Fix or update a plugin by deploying only the plugin bundle; the host stays the same. That speeds up iteration and reduces risk.

---

## Part 6: Strategy 3 – Style Isolation (Slides 19–22)

### SLIDE 19: Strategy 3: Style Isolation – The CSS Problem (2 minutes)

**What you say (script):**  
Strategy Three: Style Isolation. CSS is global by default. Plugin A defines `.MuiButton-root` with red. Plugin B defines the same class with blue. Which wins? Depends on load order. Unpredictable. Even when you share MUI via Module Federation, different plugins might add different style overrides. They conflict.

**Detailed context:**

- **CSS is global:** In traditional CSS (or many CSS-in-JS setups), class names are global. If two plugins target `.MuiButton-root`, the one that loads last or has higher specificity wins. Order can change between builds or routes, so behavior is flaky.
- **Shared MUI, different overrides:** Even with one MUI instance, each plugin might inject its own overrides (e.g., theme or `sx`). Those can still collide if they affect the same DOM nodes or classes. So sharing the library is necessary but not sufficient; you need isolation at the style level too.

---

### SLIDE 20: Isolation Techniques (2 minutes)

**What you say (script):**  
Technique one: Class name prefixing. MUI has a ClassNameGenerator. You configure it to add a prefix. Now Plugin A has `.plugin-tekton-MuiButton-root`. No more conflicts. Technique two: Shadow DOM. More isolated. Create a shadow root, render the plugin inside. Styles can't leak out. Shadow DOM is more complex but more isolated. Choose based on your needs.

**Detailed context:**

- **ClassNameGenerator (MUI):** MUI allows you to set a global prefix (e.g., per plugin) so that generated class names become unique. When each plugin runs in its own "context" with its own prefix (e.g., in a wrapper), their classes don’t clash.
- **Shadow DOM:** Encapsulates a subtree so that external CSS doesn’t penetrate and internal CSS doesn’t leak. Best isolation, but can complicate theming (you may need CSS custom properties or slots to pass theme in) and debugging. Suited for highly isolated widgets or when you have strict boundaries.

---

### SLIDE 21: More Techniques (1 minute)

**What you say (script):**  
CSS Modules—your styles get unique hashes at build time. BEM naming conventions with plugin prefixes—works but requires discipline. In RHDH, we use the ClassNameGenerator approach in plugin wrappers. 14+ plugins use this pattern.

**Detailed context:**

- **CSS Modules:** Each file’s classes are renamed to something like `button_abc123`. Good for avoiding global clashes within a plugin; you still need a strategy for shared component libraries (e.g., MUI) used across plugins.
- **BEM + plugin prefix:** e.g. `tekton-button`, `rbac-table`. Works if all teams follow the convention; easier to break under time pressure.
- **RHDH: ClassNameGenerator in wrappers:** Many community Backstage plugins don’t prefix by default. RHDH wraps them in a thin layer that sets the ClassNameGenerator prefix and re-exports the plugin. So 14+ plugins get isolation without forking upstream code.

---

### SLIDE 22: RHDH Wrapper Example (1 minute)

**What you say (script):**  
Here's the wrapper pattern. We import the ClassNameGenerator. Configure it with a prefix. Then re-export the original plugin unchanged. The wrapper adds isolation without modifying the upstream code. This is huge—you can integrate third-party plugins without forking them.

**Detailed context:**

- **Wrapper = thin package:** A small package that (1) sets the MUI class-name prefix for that plugin, (2) re-exports the original plugin. The host (or app) imports from the wrapper instead of the original. No changes to the upstream plugin repo.
- **Why it matters:** You can adopt community or vendor plugins and still enforce isolation and consistency, without maintaining a fork. Low friction for adding new plugins while keeping the design system under control.

---

## Part 7: Strategy 4 – Extension Contracts (Slides 23–27)

### SLIDE 23: Strategy 4: Extension Contracts – The Principle (2 minutes)

**What you say (script):**  
Strategy Four: Extension Contracts. Without contracts: plugins can modify anything. Breaking changes are common. No predictable structure. With contracts: plugins use defined extension points. The platform controls layout. Integration is stable and predictable. Contracts are your rules of engagement.

**Detailed context:**

- **Without contracts:** Plugins might render anywhere, replace layout, or depend on internal APIs. A host upgrade can break plugins; plugin updates can break the shell. No clear "surface area" for integration.
- **With contracts:** The platform defines named extension points (e.g., "entity overview slot," "settings page"). Plugins register what they provide for which slot. Layout and navigation stay under platform control; plugins only fill slots. Upgrades are easier because the contract is the only boundary.
- **Rules of engagement:** Makes the relationship between platform and plugins explicit and documentable. Easier onboarding and fewer surprises.

---

### SLIDE 24: Mount Points (2 minutes)

**What you say (script):**  
Mount points are named slots where plugins can inject UI. The platform defines the slots: page header, sidebar, main content, entity overview. Plugins register components for specific slots: "Put my overview card in entity.overview." The platform controls structure. Plugins provide content. Plugins can't break the layout because they can only use predefined slots.

**Detailed context:**

- **Named slots:** e.g. `entity.overview`, `page.sidebar`, `app.settings`. The host renders these slots in a fixed layout; each slot can hold one or more plugin-provided components.
- **Registration:** Plugins declare "for slot X I provide component Y." The host (or a registry) collects these and renders the right components in the right slots. Order and filtering (e.g., by entity kind) can be part of the contract.
- **Platform controls structure:** The host owns the DOM structure and the slot positions. Plugins can’t move the sidebar or remove the header; they only supply content. That prevents layout breakage and keeps the shell consistent.

---

### SLIDE 25: More Extension Points (1 minute)

**What you say (script):**  
Routes can also be declared. The plugin specifies the path, the component, and optionally a menu item. API extensions let plugins provide or extend platform APIs using a factory pattern. All through defined contracts.

**Detailed context:**

- **Routes:** Plugins register routes (path + component + optional sidebar/menu entry). The host’s router uses these so that navigation and deep links work without the host hardcoding every plugin route.
- **API extensions:** Plugins can provide new APIs (e.g., "Tekton API") or extend existing ones via a factory. The platform exposes these through a stable API layer so other plugins or the host can consume them in a typed way. All still within the same contract model.

---

### SLIDE 26: TypeScript Enforcement (2 minutes)

**What you say (script):**  
TypeScript enforces contracts at compile time. The platform defines `MountPointComponent`—what a plugin must provide. The Component itself, optional layout config, optional conditions. When plugins implement this interface, TypeScript catches violations immediately. Not at runtime—at build time. This is documentation as code.

**Detailed context:**

- **MountPointComponent (or similar):** An interface that describes the shape of a mount-point contribution: e.g. `Component`, `config.layout`, `config.if` (condition). Plugins implement this interface when they register.
- **Compile-time checks:** Missing or wrong properties are caught when the plugin is built, not when the app runs. That reduces integration bugs and makes the contract the single source of truth.
- **Documentation as code:** The TypeScript types are the contract documentation; they stay in sync with the runtime because both use the same definitions.

---

### SLIDE 27: YAML Configuration (2 minutes)

**What you say (script):**  
In RHDH, plugin integration is entirely declarative. This YAML configures the Tekton plugin. Mount points, layout, conditions, routes, menu items. Want to add a plugin? Edit the YAML. Want to remove it? Delete the lines. No code changes. No rebuilds. Operations teams can manage plugins without developer involvement.

**Detailed context:**

- **Declarative config:** Instead of code that imports and registers each plugin, a YAML (or JSON) file lists which plugins are enabled and how they’re configured (mount points, routes, conditions). The host reads this at runtime (or build time) and wires things up.
- **Add/remove by editing YAML:** Enabling or disabling a plugin doesn’t require a code change or a new deploy of the host—only a config change. Great for ops, multi-tenant, or gradual rollouts.
- **Ops without developers:** Reduces dependency on engineering for simple "turn this plugin on/off" or "add this plugin to this slot" decisions.

---

## Part 8: Strategy 5 – Configuration-Driven Customization (Slides 28–30)

### SLIDE 28: Strategy 5: Configuration-Driven – The Principle (1 minute)

**What you say (script):**  
Strategy Five: Configuration-Driven Customization. Why configuration over code? Code changes require developers. Configuration can be done by ops. Code needs rebuild and deploy. Configuration can change at runtime. Code is hard to A/B test. Configuration makes it easy. For multi-tenant platforms, configuration is essential.

**Detailed context:**

- **Code vs configuration:** Code = change requires a developer, a PR, a build, and a deploy. Configuration = change can be done by ops or product (within guardrails), often at runtime or via a config reload. Faster iteration and lower risk for tweaks like theming or feature flags.
- **A/B testing:** With config-driven theming or feature flags, you can run experiments without multiple code paths or deploys. Essential for product and design experimentation.
- **Multi-tenant:** Different tenants (or customers) can have different logos, colors, and features via config instead of separate codebases or builds.

---

### SLIDE 29: What Should Be Configurable (2 minutes)

**What you say (script):**  
Logos—with light and dark variants. Theme colors—primary, sidebar background, for both light and dark modes. Feature flags—enable dark mode, compact sidebar. All in a YAML file. No code.

**Detailed context:**

- **Logos:** Full logo and optional compact/icon, with light and dark variants so they work on different backgrounds.
- **Theme colors:** At least primary and key surfaces (e.g., sidebar background); both light and dark so the whole app can be themed per tenant or preference.
- **Feature flags:** e.g. dark mode on/off, compact sidebar. Lets you roll out or roll back UI changes without code deploys.
- **YAML (or JSON):** One place for branding and feature config; the app reads it and applies it through the theme system and feature APIs.

---

### SLIDE 30: Theme Hooks (1 minute)

**What you say (script):**  
On the platform side, hooks abstract configuration access. `useSidebarBackground` reads from the theme. Components use it. System theme detection respects user's OS preference. Light or dark, automatically.

**Detailed context:**

- **Hooks abstract config:** Instead of components reading raw config or theme files, they use hooks like `useSidebarBackground()` or `useTheme()`. The hook reads from the central theme/config and returns the right value. Easier to test and change where config lives.
- **System theme:** `prefers-color-scheme` or similar can drive initial light/dark mode so the app respects the OS setting without extra user action.

---

## Part 9: Putting It Together and Roadmap (Slides 31–34)

### SLIDE 31: Putting It Together: Complete Architecture (2 minutes)

**What you say (script):**  
Let's see how it all fits together. At the top: Theme package. Colors, typography, component styles. Your foundation. Next: Module Federation host. Shares React, UI library, router. Then: Extension point system. Mount points, routes, API factories. Below that: Plugins. Each with a thin wrapper for CSS isolation. At the bottom: Configuration. Theme values, plugin config, feature flags. This is the architecture. All five strategies working together.

**Detailed context:**

- **Layers (top to bottom):** Theme → Host (Module Federation) → Extension system → Plugins (with wrappers) → Configuration. Walk the diagram so the audience sees how each strategy sits in the stack and how they depend on each other.
- **Single diagram:** This slide is the "architecture summary"; you can refer back to it when answering "how does X fit in?"

---

### SLIDE 32: Wrapper Pattern (1 minute)

**What you say (script):**  
The wrapper pattern in detail. A wrapper is just a thin layer. Package.json declares the exports. Index.ts adds CSS isolation and re-exports. This lets you integrate third-party plugins without forking. Isolation without modification.

**Detailed context:**

- **Thin layer:** One (or a few) files: set ClassNameGenerator (or equivalent), re-export the original plugin. Package.json points to this wrapper as the entry so the host loads the wrapper, which then loads the real plugin with isolation in place.
- **No forking:** Upstream plugin stays unchanged; you get fixes and features from upstream while adding only isolation and any minor adapters in the wrapper.

---

### SLIDE 33: Implementation Roadmap (2 minutes)

**What you say (script):**  
If you're starting from scratch: Phase 1—Foundation. Theme package, Module Federation, initial mount points. Phase 2—Isolation. CSS isolation strategy, wrapper template, document contracts. Phase 3—Configuration. YAML theming, hooks, light/dark modes. Phase 4—Enforcement. TypeScript contracts, linting, visual regression tests. Don't try to do everything at once. Start with the foundation.

**Detailed context:**

- **Phase 1:** Without theme and shared deps, nothing else holds. Get theme + Module Federation + a few mount points in place first.
- **Phase 2:** Add isolation (prefix or Shadow DOM) and a wrapper pattern; document the extension contract so plugin authors know the rules.
- **Phase 3:** Move theming and feature flags to config so ops and product can iterate without code.
- **Phase 4:** Harden with TypeScript contracts, lint rules (e.g., no inline colors), and optional visual regression tests so regressions are caught early.
- **Order matters:** Doing Phase 4 before Phase 1 doesn’t work; the roadmap is a sequence, not a menu.

---

### SLIDE 34: Common Pitfalls (1 minute)

**What you say (script):**  
Learn from others' mistakes. No theme system → inconsistent colors. Bundling shared deps → multiple React instances. No CSS isolation → style conflicts. No contracts → plugins break things. Code-only config → slow changes. Each has a solution. We've covered them all.

**Detailed context:**

- **Pitfall → solution mapping:** Each pitfall corresponds to one of the five strategies. This slide is a quick "don’t do this / do this instead" recap so the audience can self-check their current setup.

---

## Part 10: Results and Closing (Slides 35–39)

### SLIDE 35: Results: Metrics (2 minutes)

**What you say (script):**  
Let's look at real results. Plugin onboarding: from 2-3 weeks down to 2-3 days. That's transformational. UI inconsistency bugs: from 15 per month to 2. Users notice the consistency. Bundle size for 5 plugins: 2.8 MB down to 850 KB. Shared dependencies pay off. Design review rounds: 4-5 down to 1-2. Plugins get it right faster. Theme changes: from days of code changes to minutes of config editing. 100+ plugins maintaining consistent UX in production. These strategies work.

**Detailed context:**

- **Onboarding:** With a clear theme, shared deps, and documented contracts, new plugins can integrate in days instead of weeks (no more "which React? which theme? where do I mount?").
- **UI bugs:** Fewer inconsistency bugs means less rework and happier users.
- **Bundle size:** One shared React + UI lib vs many copies; the numbers (2.8 MB → 850 KB for 5 plugins) show the impact.
- **Design reviews:** Plugins that follow the system pass review faster because they’re already aligned.
- **Theme changes:** Config-driven theming means changing colors or logos in minutes, not days of code and deploy.
- **100+ plugins:** Proof that the approach scales in production.

---

### SLIDE 36: Key Takeaways (2 minutes)

**What you say (script):**  
Five key takeaways. One: Invest in a central theme system early. It's your foundation. Two: Use Module Federation for runtime sharing. Single React instance. Three: Implement style isolation. CSS conflicts are inevitable otherwise. Four: Define clear extension contracts. TypeScript plus mount points. Five: Make it configuration-driven. Operations teams will thank you.

**Detailed context:**

- **One:** Theme first; everything else builds on it.
- **Two:** Shared deps at runtime avoid the "multiple React" class of bugs and reduce bundle size.
- **Three:** Global CSS will clash; prefix, Shadow DOM, or scoped CSS is necessary.
- **Four:** Contracts (mount points, types) keep integration predictable and upgradeable.
- **Five:** Config for theming and features enables ops and product to move fast without code deploys.

---

### SLIDE 37: Final Thought (30 seconds)

**What you say (script):**  
I want to leave you with this thought: "A good design system in a plugin architecture is invisible. Users see one cohesive application. Developers build features without fighting consistency." That invisibility is the goal. When you achieve it, everyone wins.

**Detailed context:**

- **Invisible:** Users don’t see "plugins" or "design system"—they see one product. Developers don’t fight over colors or layout because the system handles it.
- **Everyone wins:** Product feels polished; engineering scales; ops can configure; plugin authors have clear rules. This is the North Star for the talk.

---

### SLIDE 38: Resources (30 seconds)

**What you say (script):**  
Resources for learning more. Backstage.io for the open source platform. RHDH on GitHub for our implementation. Module Federation docs. I'm happy to connect—handles are on the slide. All slides and examples will be available at the GitHub link.

**Detailed context:**

- **Backstage.io:** Where to learn the platform RHDH is based on.
- **RHDH on GitHub:** Real code and configs for theme, Scalprum, wrappers, and config.
- **Module Federation:** Official or community docs for Webpack Module Federation.
- **Handles and repo:** So the audience can follow up and find the deck and samples.

---

### SLIDE 39: Thank You & Q&A (5 minutes)

**What you say (script):**  
Thank you so much for your time and attention. I'd love to hear your questions. Whether about implementation, challenges you're facing, or anything else—let's discuss. [Manage Q&A: repeat questions for the room, keep answers concise.]

**Detailed context:**

- **Repeat questions:** So the whole room hears the question and the answer.
- **Concise answers:** Leave time for more questions; offer to go deeper offline or via the repo/handles.

---

## Anticipated Q&A (Detailed Answers)

**Q: What if plugins use different frameworks like Vue or Angular?**

**Short answer:** Module Federation can support multiple frameworks. The key is that they all need to respect your theme system through CSS variables. It's harder but possible.

**Detailed:** You can expose a shared runtime that loads Vue and Angular apps in addition to React, or use Web Components / iframes as a boundary. The harder part is theming: React uses one theme object, Vue/Angular another. A practical approach is to drive visual tokens from CSS custom properties (or a shared config) so every framework reads the same colors and spacing from the DOM or a shared config service. Then each framework’s components apply those tokens. You may also need separate shared dependencies per framework (e.g., one React, one Vue) to avoid conflicts.

---

**Q: How do you handle third-party plugins that don't follow guidelines?**

**Short answer:** Wrappers. You create a thin wrapper that adds CSS isolation and any needed adaptations without modifying the original plugin.

**Detailed:** The wrapper (Strategy 3) sets a class-name prefix or wraps the plugin in Shadow DOM so its styles don’t clash. If the plugin uses different design tokens, you can add a small adapter in the wrapper that maps your theme to what the plugin expects, or wrap its root component and inject theme via context or props. You don’t fork the plugin so you can still pull upstream updates; the wrapper stays a thin, maintainable layer.

---

**Q: What's the performance overhead of Module Federation?**

**Short answer:** Initial load is slightly heavier because you're loading shared libraries upfront. But it amortizes quickly—with 3+ plugins, you're ahead. The shared libraries are cached.

**Detailed:** The host loads React (and other shared deps) once. That’s a fixed cost. Each plugin chunk is smaller because it doesn’t bundle React. So after the first few plugins, total bytes and parse time are typically lower than "each plugin with its own React." Shared chunks are cacheable across navigations and plugin updates, which helps repeat visits. The main overhead is the initial resolution of shared modules and any runtime checks; in practice this is small compared to the savings.

---

**Q: How do you version the theme package?**

**Short answer:** Semantic versioning. Breaking changes in tokens are major versions with deprecation periods. We aim for 6-month migration windows for major changes.

**Detailed:** Use semver: patch for fixes, minor for new tokens or backward-compatible changes, major for removed or renamed tokens or signature changes. For major changes, publish a migration guide and keep the old tokens working (with a deprecation warning) for a period (e.g., 6 months) so plugin authors can migrate. Document token lifecycle in your design system docs so consumers know what to expect.

---

**Q: Can this work with micro-frontends?**

**Short answer:** Absolutely. Module Federation is often used for micro-frontends. The same principles apply—shared dependencies, CSS isolation, contracts.

**Detailed:** Micro-frontends are another form of "host + multiple apps." The same strategies apply: a shared theme (or design tokens), shared runtime deps (React, router) where possible, style isolation per micro-frontend, and clear contracts for how they integrate (routes, events, or APIs). Module Federation is commonly used for micro-frontend loading; the main difference from "plugins" is often organizational (separate teams/repos) rather than technical. So the talk’s patterns transfer directly.

---

## Timing Summary (Detailed)

The table in [speaker.md](speaker.md) gives approximate durations and cumulative time. Use it to stay on pace. If you run over in the middle (e.g., Strategy 2 or 4), compress "Putting It Together" or "Pitfalls" rather than rushing Key Takeaways or Q&A. The last 10 minutes (takeaways, final thought, resources, thank you, Q&A) are high value; protect them.

---

**End of detailed speaker notes.** This document is intended to complement [speaker.md](speaker.md) and [slides-diagram-heavy.md](slides-diagram-heavy.md) for full preparation and reference.
