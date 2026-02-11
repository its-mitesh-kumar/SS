# Speaker Scripts by Slide — Simple & Descriptive

This file gives you a **simple, descriptive, and engaging speaker script for every slide** of the Design Systems at Scale presentation, plus a **transition** to the next slide. Scripts are written to be easy to understand, interesting, and attention-catching—plain language, concrete examples, and clear signposting. Each slide has: (1) **Speaker script**—what to say; (2) **Transition**—a short lead-in to the next slide. Use the script as-is or adapt it to your style.

**Source:** Based on [speaker-notes-detailed.md](speaker-notes-detailed.md).  
**Deck:** 39 slides. **Total time:** ~40 minutes + Q&A.

---

## Part 1: Opening and Agenda (Slides 1–2)

### SLIDE 1: Title (1 minute)

**Speaker script:**

Good morning (or afternoon). Have you ever opened a product and thought, "This part looks like it's from a different app"? That's what we're here to fix. I'm Mitesh Kumar from Red Hat Developer Hub. This session is a practical guide: how to keep your UI consistent when dozens—or hundreds—of plugins plug into one app. Whether you're building an IDE, a developer portal, or any extensible product, the same challenges show up. And the same strategies work.

**Transition:** So that's the intro. Next, here's what we'll cover in this session.

---

### SLIDE 2: What You'll Learn (1 minute)

**Speaker script:**

Here's the map. On the left: the **challenge**—why your UI goes off the rails when many teams ship many plugins. On the right: five **solutions** that actually work, with patterns you can use tomorrow. I'll walk you through Red Hat Developer Hub: we run 100+ plugins in production. Same ideas apply whether you have ten plugins or a thousand.

**Transition:** First, let's align on what we mean by a plugin architecture.

---

## Part 2: The Challenge (Slides 3–6)

### SLIDE 3: What is Plugin Architecture? (1 minute)

**Speaker script:**

First, what do we mean by a plugin architecture? Picture one **core app**—the host. It’s the shell: layout, navigation, the frame. Now other people add **plugins** that slot in. One plugin does CI/CD. Another does security. Another does something else. Each adds a capability. But here’s the key: they all run **inside** the same host. So your user sees one app. They don’t see “twenty different tools”—they see one product with many features. But those features can be built by your team, the community, or third-party vendors. Different people, one product. That’s the power—and that’s also why keeping it consistent is hard.

**Transition:** So why are plugin architectures everywhere? Let's talk about that.

---

### SLIDE 4: Why Plugin Architectures? (2 minutes)

**Speaker script:**

Quick show of hands—how many of you work on an application that has some kind of plugin or extension system? [Pause.] Quite a few. And there's a good reason. **Plugin architectures are everywhere—and there are many of them.** VS Code has tens of thousands of extensions. Shopify has 8,000-plus apps. Figma has thousands of plugins. WordPress has tens of thousands. Different products use different words—extensions, apps, plugins—but it's the same idea: a host platform and add-ons that extend it. In this talk I'll call them all plugins. So why are there so many? Because plugins solve a core business problem: **you can extend the product without everything going through one central team**. Many third parties can build features. Many teams can ship at their own speed. You get an ecosystem—many pieces from many people—instead of one big monolith. But that power comes with a challenge, and that's what we'll tackle next.

**Transition:** That challenge is consistency. Here's the problem.

---

### SLIDE 5: The Consistency Problem (2 minutes)

**Speaker script:**

Here's the tension. **Your users don't care that 50 plugins built your UI. They expect one app.** So what breaks? Different teams pick different blues and spacing. Plugins ignore your guidelines. One plugin uses React 18.1, another 18.2—versions drift. CSS from one plugin overrides another. Accessibility is all over the place. The cost? Confused users. More support tickets. A brand that looks patched together. And the worst part: **users blame you—the platform—not the plugin.** So getting consistency right is in your interest.

**Transition:** Let me show you the difference visually—before and after.

---

### SLIDE 6: Before vs After (1 minute)

**Speaker script:**

Picture before and after. **Without a strategy:** buttons change from page to page. Forms behave differently. Navigation feels random. Users feel lost. **With a strategy:** one set of components, one behavior, one look. Users feel at home. Your goal? Make plugins **invisible**. Users should never think, "This bit feels like another app." It's one product, period.

**Transition:** So how do we get there? Here are the five strategies that work together.

---

## Part 3: The Five Strategies Overview and Case Study (Slides 7–8)

### SLIDE 7: Five Strategies (2 minutes)

**Speaker script:**

Five strategies—and they work together. **One: Central theme.** One place for colors, fonts, spacing. Your foundation. **Two: Share dependencies at runtime.** So every plugin uses the *same* React and UI library—one instance, not four. We'll use Module Federation. **Three: Isolate styles.** So one plugin's CSS doesn't break another's. Class prefixes or Shadow DOM. **Four: Extension contracts.** Clear rules: where plugins plug in, how they register routes and APIs. Mount points, types. **Five: Configuration-driven.** Theming and features from config, not code—change look without redeploying. We'll go through each.

**Transition:** I'll ground all of this in a real case study—Red Hat Developer Hub.

---

### SLIDE 8: Case Study Introduction (1 minute)

**Speaker script:**

Everything I show you is real. Red Hat Developer Hub—RHDH—is our internal developer portal, built on Backstage. We run 100+ plugins: our teams, the Backstage community, and third-party vendors. The challenge? Make all of them feel like **one** product. No toy examples. This is production.

**Transition:** Let's start with Strategy One: the central theme system.

---

## Part 4: Strategy 1 – Central Theme System (Slides 9–13)

### SLIDE 9: Strategy 1 – Central Theme System: The Principle (2 minutes)

**Speaker script:**

Strategy One: **one place that controls how everything looks.** One package or module. What's in it? Colors—primary, secondary, error, success. Typography—fonts, sizes, weights. Spacing—your 4, 8, 16px rhythm so nobody invents their own. How buttons, cards, and tables look. Light and dark mode. This is your foundation. Get it right first. Everything else sits on top.

**Transition:** Why does centralizing matter so much? Here's what happens when you don't.

---

### SLIDE 10: Why Centralize (2 minutes)

**Speaker script:**

What happens without it? Plugin A picks a blue. Plugin B picks another. Plugin C says "blue." You use a fourth. Result: four blues on one screen. Users notice. With a central theme, there's **one** "primary blue." Every plugin uses it. One blue everywhere. Sounds obvious—but many teams skip it. Each plugin does its own thing. That's the disaster we're avoiding.

**Transition:** Here's the concrete implementation pattern.

---

### SLIDE 11: Implementation Pattern – Theme (2 minutes)

**Speaker script:**

The pattern in practice. Define a **TypeScript interface** for your theme—one shape, one contract. Core colors first: primary, secondary. Then semantic: error, warning, success, info—so plugins never hardcode red or green. Then **component tokens**: sidebar background, card border, table header. Start with 20–30 tokens; grow as you need. One place, one definition. The compiler and your IDE keep everyone in sync.

**Transition:** In RHDH we put this into practice—here's how it looks.

---

### SLIDE 12: RHDH Example – Theme (2 minutes)

**Speaker script:**

In RHDH we have a theme plugin with 30+ color tokens. The interesting bit: **ThemeConfigOptions** lets you pick PatternFly or MUI **per component**. Buttons can be PatternFly; tables can be MUI. The platform decides—not the plugins. One hook—**useThemes()**—and everyone gets theme data from the same place.

**Transition:** If you remember one thing from the theme section, it's this takeaway.

---

### SLIDE 13: Takeaway – Theme (30 seconds)

**Speaker script:**

One takeaway: **invest in a theme package early. It's your foundation.** Checklist: one package, 20–30 tokens, component tokens, light and dark, TypeScript types, one central hook. Do that first.

**Transition:** Now Strategy Two: runtime dependency sharing—and the number one cause of plugin bugs.

---

## Part 5: Strategy 2 – Runtime Dependency Sharing (Slides 14–18)

### SLIDE 14: Strategy 2 – Runtime Dependency Sharing: The Problem with Bundling (2 minutes)

**Speaker script:**

Strategy Two: share dependencies at runtime. Why does it matter? This is the **number one cause** of plugin bugs. If every plugin bundles its own React, you get four Reacts on the page. Hooks break. You see "Invalid hook call." Bundle size explodes. **React must be a singleton**—one instance for the whole app. Same for Redux, the router, anything stateful.

**Transition:** The fix is Module Federation—sharing at runtime.

---

### SLIDE 15: The Solution – Module Federation (2 minutes)

**Speaker script:**

The fix: **Module Federation.** Share at runtime, not at build time. The host loads React, ReactDOM, MUI, the router—once. Plugins don't bundle their own; they take them from the host when they run. One React. One MUI. One router. Done.

**Transition:** What should you actually share? And what should plugins keep in their own bundle?

---

### SLIDE 16: What to Share (1 minute)

**Speaker script:**

What to share? React, ReactDOM, the UI library, the router, state management, core platform components. What do plugins keep? Whatever only they use—D3, chart libs, heavy utils. Rule of thumb: share what **must** be a singleton, or what more than half your plugins use.

**Transition:** Here's how you set it up with Webpack.

---

### SLIDE 17: Implementation – Webpack Module Federation (2 minutes)

**Speaker script:**

How to set it up? Add the **ModuleFederationPlugin** to your webpack config. List shared deps: React, ReactDOM, MUI, router. **singleton: true** is crucial—it forces one instance at runtime. **requiredVersion** catches mismatches: plugin built on React 17, host on 18? You get a warning, not a silent bug. Built into Webpack 5. No extra libs.

**Transition:** In RHDH we use Scalprum on top of that—quick look.

---

### SLIDE 18: RHDH Scalprum (1 minute)

**Speaker script:**

In RHDH we use **Scalprum** on top of Module Federation. It does something neat: we rewrite plugin manifests so scripts load from our **backend API**, not a static CDN. Plugins are served dynamically. Update a plugin? Deploy just that plugin. No host rebuild. Faster and safer.

**Transition:** Strategy Three is style isolation—because even with one MUI, CSS can still fight.

---

## Part 6: Strategy 3 – Style Isolation (Slides 19–22)

### SLIDE 19: Strategy 3 – Style Isolation: The CSS Problem (2 minutes)

**Speaker script:**

Strategy Three: **style isolation.** CSS is global. Plugin A styles the same button class red. Plugin B styles it blue. Who wins? Load order—and that can change. Unpredictable. Even with one shared MUI, plugins still add overrides. They clash. So sharing the library isn't enough. You need isolation at the **style** level.

**Transition:** Here are two main techniques that work.

---

### SLIDE 20: Isolation Techniques (2 minutes)

**Speaker script:**

Two main techniques. **One: Class name prefixing.** MUI's ClassNameGenerator lets you add a prefix. So Plugin A gets `.plugin-tekton-MuiButton-root`, Plugin B gets a different prefix. No more clashes. **Two: Shadow DOM.** Render the plugin inside a shadow root. Styles don't leak in or out. Strongest isolation—but theming and debugging are trickier. Pick based on how much you need.

**Transition:** A few more options we use in practice.

---

### SLIDE 21: More Techniques (1 minute)

**Speaker script:**

Other options: **CSS Modules** (unique hashes at build time—good inside a plugin, but you still need a plan for MUI). **BEM + plugin prefix**—tekton-button, rbac-table—works if everyone follows it; easy to break under pressure. In RHDH we use **thin wrappers** that set the ClassNameGenerator prefix and re-export the plugin. 14+ plugins, no forking upstream.

**Transition:** Here's that wrapper pattern in a bit more detail.

---

### SLIDE 22: RHDH Wrapper Example (1 minute)

**Speaker script:**

The wrapper in practice: import ClassNameGenerator, set a prefix for that plugin, re-export the original. No changes to upstream. You get isolation—and you can bring in third-party or community plugins without forking. Your design system stays in control.

**Transition:** Strategy Four is extension contracts—the rules of engagement between platform and plugins.

---

## Part 7: Strategy 4 – Extension Contracts (Slides 23–27)

### SLIDE 23: Strategy 4 – Extension Contracts: The Principle (2 minutes)

**Speaker script:**

Strategy Four: **extension contracts.** Without them, plugins can do anything—render anywhere, replace layout, use internal APIs. Upgrades break things. No clear boundary. With contracts, the platform defines **slots**—"entity overview," "settings page." Plugins say: "I fill this slot with this component." Platform owns layout. Plugins fill slots. Stable, predictable. Contracts are your **rules of engagement.**

**Transition:** The main idea is mount points—named slots in the layout.

---

### SLIDE 24: Mount Points (2 minutes)

**Speaker script:**

**Mount points** are named slots. Page header. Sidebar. Entity overview. Plugins say: "Put my card in entity.overview." You control where the slots are. Plugins only supply content. So they can't break the layout—they can't move the sidebar or kill the header. They just fill the blanks.

**Transition:** Beyond UI slots, you have routes and API extensions.

---

### SLIDE 25: More Extension Points (1 minute)

**Speaker script:**

The same contract idea goes beyond UI slots. **Routes first.** Plugins don't ask you to add their page to your router by hand. Instead they declare: "Here's my path—for example slash tekton. Here's the component that renders the page. And optionally, here's a menu item so it shows up in the sidebar." The host's router reads those declarations and wires the routes. So you never hardcode "if path is tekton, show TektonPage." The contract drives it. **Then API extensions.** Plugins can provide new APIs—like a Tekton API for CI/CD data—or extend existing ones. They do it by registering a factory: a function that, when the platform needs the API, creates the implementation. The platform exposes one stable, typed layer so the host and other plugins can get and use these APIs in a consistent way. So: routes and APIs are extension points too. All through the same contract—declare, don't hack.

**Transition:** TypeScript makes these contracts enforceable at build time.

---

### SLIDE 26: TypeScript Enforcement (2 minutes)

**Speaker script:**

How do you make sure plugins actually follow the contract? **TypeScript.** The platform defines a type—something like MountPointComponent—that says exactly what a plugin must provide: the component to render, optional layout hints like grid column, and optional conditions like "only show this card when the entity is a Component." When a plugin registers for a slot, it has to pass an object that matches this type. If they forget the component, or spell something wrong, or pass the wrong shape—TypeScript reports an error **when they build**, not when the app is already running in production. So broken integrations get caught in the editor and in CI, before anything ships. And because the contract lives in the type definition, it doubles as documentation: the type is always up to date, because the compiler won't let the code drift from it. Types are the contract, and they're docs that never go stale.

**Transition:** In RHDH, plugin integration is driven entirely by YAML—no code changes.

---

### SLIDE 27: YAML Configuration (2 minutes)

**Speaker script:**

In RHDH, plugin integration is **all YAML.** Mount points, layout, conditions, routes, menu—all in config. Add a plugin? Edit YAML. Remove it? Delete lines. No code. No rebuild. Ops can turn plugins on or off, or move them around, without developers. Huge for multi-tenant and gradual rollouts.

**Transition:** Strategy Five is configuration-driven customization—theming and features without code.

---

## Part 8: Strategy 5 – Configuration-Driven Customization (Slides 28–30)

### SLIDE 28: Strategy 5 – Configuration-Driven: The Principle (1 minute)

**Speaker script:**

Strategy Five: **configuration-driven.** Why config over code? Code means dev, PR, build, deploy. Config can be changed by ops or product—often at runtime. Code is hard to A/B test; config is easy. For multi-tenant, config is essential: different logos, colors, features—no separate codebases.

**Transition:** What should actually go in that config?

---

### SLIDE 29: What Should Be Configurable (2 minutes)

**Speaker script:**

What goes in the config? **Logos**—light and dark variants. **Theme colors**—primary, sidebar, for light and dark. **Feature flags**—dark mode, compact sidebar. YAML or JSON. No code. The app reads it and applies it through your theme and feature APIs.

**Transition:** On the platform side, hooks make that config easy to consume.

---

### SLIDE 30: Theme Hooks (1 minute)

**Speaker script:**

On the platform, **hooks** hide the config. `useSidebarBackground()` reads from the theme and returns the value. Components use the hook—they don't touch raw config. You can also follow the user's OS: light or dark, automatically. One config, one set of hooks.

**Transition:** Let's see how all five strategies fit together in one architecture.

---

## Part 9: Putting It Together and Roadmap (Slides 31–34)

### SLIDE 31: Putting It Together – Complete Architecture (2 minutes)

**Speaker script:**

How it all fits. **Top:** theme package—colors, typography, styles. Foundation. **Next:** Module Federation host—shared React, UI lib, router. **Then:** extension system—mount points, routes, APIs. **Below:** plugins, each with a thin wrapper for CSS isolation. **Bottom:** config—theme, plugin settings, feature flags. Full stack. All five strategies together.

**Transition:** The wrapper pattern in a bit more detail.

---

### SLIDE 32: Wrapper Pattern (1 minute)

**Speaker script:**

Wrapper in detail: a thin layer. package.json points to your entry. The entry sets the class-name prefix and re-exports the real plugin. Third-party plugins, no fork. Isolation without touching their code.

**Transition:** If you're starting from scratch, here's a phased roadmap.

---

### SLIDE 33: Implementation Roadmap (2 minutes)

**Speaker script:**

Starting from zero? Go in phases. **Phase 1:** Theme, Module Federation, a few mount points. **Phase 2:** CSS isolation, wrapper template, document contracts. **Phase 3:** YAML theming, hooks, light/dark. **Phase 4:** TypeScript contracts, linting, maybe visual regression. Don't do it all at once. Foundation first—then build.

**Transition:** And a few common pitfalls to avoid.

---

### SLIDE 34: Common Pitfalls (1 minute)

**Speaker script:**

Pitfalls to avoid. No theme → inconsistent colors. Every plugin bundles React → multiple instances, hooks break. No CSS isolation → style wars. No contracts → plugins break the shell. Config only in code → every tweak needs a deploy. Each has a fix—and we've covered all five.

**Transition:** What does this get you in practice? Here are some real numbers.

---

## Part 10: Results and Closing (Slides 35–39)

### SLIDE 35: Results – Metrics (2 minutes)

**Speaker script:**

Real numbers. **Onboarding:** 2–3 weeks → 2–3 days. **UI bugs:** ~15/month → ~2. Users feel the difference. **Bundle size** (5 plugins): 2.8 MB → 850 KB. Shared deps pay off. **Design reviews:** 4–5 rounds → 1–2. Plugins align faster. **Theme changes:** days of code → minutes of config. 100+ plugins in production, one consistent UX. These strategies work.

**Transition:** Five things to take away.

---

### SLIDE 36: Key Takeaways (2 minutes)

**Speaker script:**

Five takeaways. **One:** Theme early—it's your foundation. **Two:** Module Federation—one React, one UI lib. **Three:** Style isolation—or CSS will fight. **Four:** Extension contracts—TypeScript and mount points. **Five:** Config-driven—so ops can change behavior without code. They'll thank you.

**Transition:** One final thought before we wrap up.

---

### SLIDE 37: Final Thought (30 seconds)

**Speaker script:**

One last thought. A great design system in a plugin world is **invisible**. Users see one app. Developers ship without fighting consistency. That invisibility is the goal. Get there, and everyone wins.

**Transition:** Here are some resources to go deeper.

---

### SLIDE 38: Resources (30 seconds)

**Speaker script:**

Resources: Backstage.io for the platform. RHDH on GitHub for our code—theme, Scalprum, wrappers, config. Module Federation docs for the how-to. I'm happy to connect—handles on the slide. Slides and examples at the GitHub link. Thanks.

**Transition:** Thank you—and over to you for questions.

---

### SLIDE 39: Thank You & Q&A (5 minutes)

**Speaker script:**

Thank you. Questions? Implementation, challenges you're facing, whatever's on your mind—let's talk. [Repeat for the room, keep answers tight, offer to go deeper offline or via the repo.]

**Transition:** [None—open the floor for Q&A.]

---

**End of speaker scripts by slide.**
