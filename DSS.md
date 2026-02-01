# Speaker Script: Design Systems at Scale

**DevConf.IN 2026 - Track 4: User Experience and Design Engineering**
**Duration**: 40 minutes presentation + 5 minutes Q&A
**Speaker**: Mitesh Kumar

---

## Pre-Talk Checklist

- [ ] Test slides on presentation system
- [ ] Have RHDH running locally for demos (optional)
- [ ] Backup slides as PDF
- [ ] Water bottle ready
- [ ] Timer visible

---

## SLIDE 1: Title (1 minute)

**[CLICK TO SLIDE 1]**

*[Walk to center stage, pause, make eye contact]*

Good [morning/afternoon] everyone, and welcome to "Design Systems at Scale: Maintaining UI Consistency in Dynamic Plugin Architectures."

I'm Mitesh Kumar, and I'm a frontend contributor to Red Hat Developer Hub.

Today, I want to share a practical guide for anyone building plugin-based platforms. Whether you're working on an IDE, a developer portal, or any extensible application - the challenges we'll discuss are universal.

---

## SLIDE 2: What You'll Learn (1 minute)

**[CLICK TO SLIDE 2]**

Here's what we'll cover:

On the left - the challenge. Why UI consistency is hard when you have dozens or hundreds of plugins from different teams.

On the right - the solutions. Five proven strategies with real implementation patterns you can take back to your teams.

We'll use Red Hat Developer Hub as our case study - a production system with 100+ plugins - but the patterns apply to any plugin architecture.

---

## SLIDE 3: Part 1 Header

**[CLICK TO SLIDE 3]**

*[Pause for transition]*

Let's start with the universal challenge.

---

## SLIDE 4: Why Plugin Architectures? (2 minutes)

**[CLICK TO SLIDE 4]**

Quick show of hands - how many of you work on applications with some kind of plugin or extension system?

*[Pause for hands]*

That's a lot! And there's a good reason. Plugin architectures are everywhere.

Look at the success stories:
- VS Code has over 40,000 extensions
- Figma has 2,000+ plugins
- WordPress has 60,000+ plugins

Why? Because plugin architectures solve a fundamental business problem: **extensibility without bottlenecking on the core team**.

Third parties can build features. Independent teams can move at their own pace. You get an ecosystem instead of a monolith.

But this power comes with a challenge...

---

## SLIDE 5: The Consistency Problem (2 minutes)

**[CLICK TO SLIDE 5]**

*[Read the quote with emphasis]*

> "Your users don't care that your UI is composed of 50 different plugins. They expect it to feel like ONE application."

This is the fundamental tension.

On the left - what goes wrong:
- Different teams make different design decisions
- Third-party plugins ignore your guidelines
- Dependencies drift out of sync
- CSS conflicts between plugins
- Inconsistent accessibility

On the right - the cost:
- Users get confused
- Support tickets go up
- Your brand gets diluted
- Maintenance becomes a nightmare

And here's the worst part: **users blame YOUR platform, not the plugins**.

---

## SLIDE 6: Before vs After (1 minute)

**[CLICK TO SLIDE 6]**

*[Gesture to left side, then right side]*

Visualize the difference.

Without a strategy: buttons look different everywhere, forms behave inconsistently, navigation varies. Users feel lost.

With a strategy: unified components, consistent interactions, cohesive visual language. Users feel confident.

Your goal is to make plugins invisible to users. They should never think "this feels like a different app."

---

## SLIDE 7: Part 2 Header

**[CLICK TO SLIDE 7]**

*[Pause]*

Now let's look at the solution framework.

---

## SLIDE 8: Five Strategies (2 minutes)

**[CLICK TO SLIDE 8]**

*[Walk through each strategy]*

Here are the five strategies that work together to solve UI consistency:

**One: Central Theme System** - A single source of truth for colors, fonts, spacing. This is your foundation.

**Two: Runtime Dependency Sharing** - Using Module Federation so all plugins share ONE React instance, ONE UI library instance.

**Three: Style Isolation** - Preventing CSS from one plugin from breaking another.

**Four: Extension Contracts** - Defining clear rules for how plugins integrate. Mount points, typed APIs.

**Five: Configuration-Driven Customization** - Enabling theming without code changes.

We'll dive deep into each of these.

---

## SLIDE 9: Case Study Introduction (1 minute)

**[CLICK TO SLIDE 9]**

Throughout this talk, I'll use Red Hat Developer Hub as our case study.

RHDH is an internal developer portal built on Backstage - Spotify's open platform. We have 100+ plugins from Red Hat teams, the Backstage community, and third-party vendors.

Our challenge was making all these plugins look like ONE product. Everything I'll show you comes from real production code.

---

## SLIDE 10: Strategy 1 Header

**[CLICK TO SLIDE 10]**

*[Pause]*

Strategy One: Central Theme System. This is where you should start.

---

## SLIDE 11: The Principle (2 minutes)

**[CLICK TO SLIDE 11]**

*[Emphasize the key idea]*

The key idea is simple: **create a single package that controls ALL visual styles**.

What should it control?
- Colors - primary, secondary, status colors
- Typography - font family, sizes, weights
- Spacing - your 4px, 8px, 16px system
- Component appearance - how buttons, cards, tables look
- Light and dark mode variants

This is your foundation. Get this right first.

---

## SLIDE 12: Why Centralize (2 minutes)

**[CLICK TO SLIDE 12]**

*[Show the contrast]*

Here's what happens without a central theme:

Plugin A uses one shade of blue. Plugin B uses another. Plugin C just says "blue". The platform uses a fourth shade.

Result? Four different "blues" in your UI. Users notice.

With a central theme package, there's one definition of "primary blue." Every plugin imports from the same source. One blue everywhere.

This seems obvious, but many teams skip it. They let plugins define their own colors. It's a disaster waiting to happen.

---

## SLIDE 13: Implementation Pattern (2 minutes)

**[CLICK TO SLIDE 13]**

*[Walk through the code]*

Here's the pattern. Define a TypeScript interface for your theme palette.

Start with core colors - primary, secondary.

Add semantic colors - error, warning, success, info.

Then - and this is crucial - add **component-specific tokens**. Sidebar background, card borders, table headers.

Aim for 20-30 tokens to start. You'll add more as needed.

The key is defining everything in one place.

---

## SLIDE 14: RHDH Example (2 minutes)

**[CLICK TO SLIDE 14]**

In RHDH, we have a dedicated theme plugin with 30+ color tokens.

What's interesting is the `ThemeConfigOptions` type. See how it lets you choose between 'patternfly' and 'mui' styling **per component**?

Buttons can be PatternFly style. Tables can be MUI style. The platform decides - not the plugins.

This gives us flexibility while maintaining control.

The `useThemes()` hook is how the platform accesses all theme data centrally.

---

## SLIDE 15: Takeaway (30 seconds)

**[CLICK TO SLIDE 15]**

*[Emphasize]*

If you take nothing else from this talk: **invest in a robust theme package early. It's your foundation.**

Here's a quick checklist. Single package. 20-30 tokens minimum. Component-specific tokens. Light and dark mode. TypeScript types. Central hook.

---

## SLIDE 16: Strategy 2 Header

**[CLICK TO SLIDE 16]**

*[Pause]*

Strategy Two: Runtime Dependency Sharing.

---

## SLIDE 17: The Problem with Bundling (2 minutes)

**[CLICK TO SLIDE 17]**

*[Show urgency]*

This is the number one cause of plugin bugs.

If each plugin bundles its own React, you end up with multiple React instances running at the same time.

The result? Four copies of React loaded. Hooks don't work across instances. You get cryptic "Invalid hook call" errors. Bundle sizes explode.

React MUST be a singleton. Same with any stateful library.

---

## SLIDE 18: The Solution (2 minutes)

**[CLICK TO SLIDE 18]**

The solution is Module Federation - sharing libraries at RUNTIME, not build time.

*[Point to diagram]*

The host application provides React, ReactDOM, MUI, Router - loaded once.

Plugins consume these from the host at runtime. They don't bundle their own copies.

Single instance of everything. Problem solved.

---

## SLIDE 19: What to Share (1 minute)

**[CLICK TO SLIDE 19]**

*[Go through the table]*

What should you share?

React, ReactDOM - absolutely.
UI library - yes.
Router - yes.
State management - yes.
Core platform components - yes.

What should plugins bundle themselves? Plugin-specific libraries. Visualization tools like D3. Heavy utilities only they use.

Rule of thumb: share anything that must be a singleton OR is used by more than half your plugins.

---

## SLIDE 20: Implementation (2 minutes)

**[CLICK TO SLIDE 20]**

*[Walk through code]*

Here's how to set it up with Webpack Module Federation.

In your webpack config, add the ModuleFederationPlugin. Define your shared dependencies. The `singleton: true` flag is crucial - it forces a single instance.

`requiredVersion` adds safety - it'll warn if a plugin expects an incompatible version.

This is Webpack 5's built-in feature. No extra libraries needed.

---

## SLIDE 21: RHDH Scalprum (1 minute)

**[CLICK TO SLIDE 21]**

RHDH uses Scalprum, which wraps Module Federation with some nice features.

See how we transform plugin manifests to load scripts from our backend API? This means plugins are served dynamically - we can update them without rebuilding the host application.

---

## SLIDE 22: Strategy 3 Header

**[CLICK TO SLIDE 22]**

*[Pause]*

Strategy Three: Style Isolation.

---

## SLIDE 23: The CSS Problem (1 minute)

**[CLICK TO SLIDE 23]**

CSS is global by default.

*[Point to code]*

Plugin A defines `.MuiButton-root` with red. Plugin B defines the same class with blue. Which wins? Depends on load order. Unpredictable.

Even when you share MUI via Module Federation, different plugins might add different style overrides. They conflict.

---

## SLIDE 24: Isolation Techniques (2 minutes)

**[CLICK TO SLIDE 24]**

*[Cover both techniques]*

Technique one: Class name prefixing.

MUI has a ClassNameGenerator. You configure it to add a prefix. Now Plugin A has `.plugin-tekton-MuiButton-root`. No more conflicts.

Technique two: Shadow DOM.

More isolated. Create a shadow root, render the plugin inside. Styles can't leak out.

Shadow DOM is more complex but more isolated. Choose based on your needs.

---

## SLIDE 25: More Techniques (1 minute)

**[CLICK TO SLIDE 25]**

*[Quick overview]*

CSS Modules - your styles get unique hashes at build time.

BEM naming conventions with plugin prefixes - works but requires discipline.

In RHDH, we use the ClassNameGenerator approach in plugin wrappers. 14+ plugins use this pattern.

---

## SLIDE 26: RHDH Wrapper Example (1 minute)

**[CLICK TO SLIDE 26]**

*[Walk through the code]*

Here's the wrapper pattern.

We import the ClassNameGenerator. Configure it with a prefix. Then re-export the original plugin unchanged.

The wrapper adds isolation without modifying the upstream code. This is huge - you can integrate third-party plugins without forking them.

---

## SLIDE 27: Strategy 4 Header

**[CLICK TO SLIDE 27]**

*[Pause]*

Strategy Four: Extension Contracts.

---

## SLIDE 28: The Principle (2 minutes)

**[CLICK TO SLIDE 28]**

*[Emphasize the contrast]*

Without contracts: plugins can modify anything. Breaking changes are common. No predictable structure.

With contracts: plugins use defined extension points. The platform controls layout. Integration is stable and predictable.

Contracts are your rules of engagement.

---

## SLIDE 29: Mount Points (2 minutes)

**[CLICK TO SLIDE 29]**

*[Walk through code]*

Mount points are named slots where plugins can inject UI.

The platform defines the slots: page header, sidebar, main content, entity overview.

Plugins register components for specific slots: "Put my overview card in entity.overview."

The platform controls structure. Plugins provide content. Plugins can't break the layout because they can only use predefined slots.

---

## SLIDE 30: More Extension Points (1 minute)

**[CLICK TO SLIDE 30]**

Routes can also be declared. The plugin specifies the path, the component, and optionally a menu item.

API extensions let plugins provide or extend platform APIs using a factory pattern.

All through defined contracts.

---

## SLIDE 31: TypeScript Enforcement (2 minutes)

**[CLICK TO SLIDE 31]**

*[Emphasize TypeScript]*

TypeScript enforces contracts at compile time.

The platform defines `MountPointComponent` - what a plugin must provide. The Component itself, optional layout config, optional conditions.

When plugins implement this interface, TypeScript catches violations immediately. Not at runtime - at build time.

This is documentation as code.

---

## SLIDE 32: YAML Configuration (2 minutes)

**[CLICK TO SLIDE 32]**

*[Show the YAML]*

In RHDH, plugin integration is entirely declarative.

This YAML configures the Tekton plugin. Mount points, layout, conditions, routes, menu items.

Want to add a plugin? Edit the YAML. Want to remove it? Delete the lines. No code changes. No rebuilds.

Operations teams can manage plugins without developer involvement.

---

## SLIDE 33: Strategy 5 Header

**[CLICK TO SLIDE 33]**

*[Pause]*

Strategy Five: Configuration-Driven Customization.

---

## SLIDE 34: The Principle (1 minute)

**[CLICK TO SLIDE 34]**

*[Go through the table]*

Why configuration over code?

Code changes require developers. Configuration can be done by ops.
Code needs rebuild and deploy. Configuration can change at runtime.
Code is hard to A/B test. Configuration makes it easy.

For multi-tenant platforms, configuration is essential.

---

## SLIDE 35: What Should Be Configurable (2 minutes)

**[CLICK TO SLIDE 35]**

*[Walk through YAML]*

Logos - with light and dark variants.
Theme colors - primary, sidebar background, for both light and dark modes.
Feature flags - enable dark mode, compact sidebar.

All in a YAML file. No code.

---

## SLIDE 36: Theme Hooks (1 minute)

**[CLICK TO SLIDE 36]**

*[Explain the code]*

On the platform side, hooks abstract configuration access.

`useSidebarBackground` reads from the theme. Components use it.

System theme detection respects user's OS preference. Light or dark, automatically.

---

## SLIDE 37: Putting Together Header

**[CLICK TO SLIDE 37]**

*[Pause]*

Let's see how it all fits together.

---

## SLIDE 38: Complete Architecture (2 minutes)

**[CLICK TO SLIDE 38]**

*[Walk through the diagram top to bottom]*

At the top: Theme package. Colors, typography, component styles. Your foundation.

Next: Module Federation host. Shares React, UI library, router.

Then: Extension point system. Mount points, routes, API factories.

Below that: Plugins. Each with a thin wrapper for CSS isolation.

At the bottom: Configuration. Theme values, plugin config, feature flags.

This is the architecture. All five strategies working together.

---

## SLIDE 39: Wrapper Pattern (1 minute)

**[CLICK TO SLIDE 39]**

*[Quick overview]*

The wrapper pattern in detail.

A wrapper is just a thin layer. Package.json declares the exports. Index.ts adds CSS isolation and re-exports.

This lets you integrate third-party plugins without forking. Isolation without modification.

---

## SLIDE 40: Implementation Roadmap (2 minutes)

**[CLICK TO SLIDE 40]**

*[Go through phases]*

If you're starting from scratch:

Phase 1: Foundation. Theme package, Module Federation, initial mount points.

Phase 2: Isolation. CSS isolation strategy, wrapper template, document contracts.

Phase 3: Configuration. YAML theming, hooks, light/dark modes.

Phase 4: Enforcement. TypeScript contracts, linting, visual regression tests.

Don't try to do everything at once. Start with the foundation.

---

## SLIDE 41: Common Pitfalls (1 minute)

**[CLICK TO SLIDE 41]**

*[Quick warning]*

Learn from others' mistakes.

No theme system → inconsistent colors.
Bundling shared deps → multiple React instances.
No CSS isolation → style conflicts.
No contracts → plugins break things.
Code-only config → slow changes.

Each has a solution. We've covered them all.

---

## SLIDE 42: Results Header

**[CLICK TO SLIDE 42]**

*[Pause]*

Let's look at real results.

---

## SLIDE 43: Metrics (2 minutes)

**[CLICK TO SLIDE 43]**

*[Go through each metric with emphasis]*

Plugin onboarding: from 2-3 weeks down to 2-3 days. That's transformational.

UI inconsistency bugs: from 15 per month to 2. Users notice the consistency.

Bundle size for 5 plugins: 2.8 MB down to 850 KB. Shared dependencies pay off.

Design review rounds: 4-5 down to 1-2. Plugins get it right faster.

Theme changes: from days of code changes to minutes of config editing.

100+ plugins maintaining consistent UX in production. These strategies work.

---

## SLIDE 44: Key Takeaways (2 minutes)

**[CLICK TO SLIDE 44]**

*[Slow down, emphasize each point]*

Five key takeaways:

One: Invest in a central theme system early. It's your foundation.

Two: Use Module Federation for runtime sharing. Single React instance.

Three: Implement style isolation. CSS conflicts are inevitable otherwise.

Four: Define clear extension contracts. TypeScript plus mount points.

Five: Make it configuration-driven. Operations teams will thank you.

---

## SLIDE 45: Final Thought (30 seconds)

**[CLICK TO SLIDE 45]**

*[Pause, deliver with conviction]*

I want to leave you with this thought:

> "A good design system in a plugin architecture is invisible. Users see one cohesive application. Developers build features without fighting consistency."

That invisibility is the goal. When you achieve it, everyone wins.

---

## SLIDE 46: Resources (30 seconds)

**[CLICK TO SLIDE 46]**

Resources for learning more.

Backstage.io for the open source platform. RHDH on GitHub for our implementation. Module Federation docs.

I'm happy to connect - handles are on the slide.

All slides and examples will be available at the GitHub link.

---

## SLIDE 47: Thank You & Q&A (5 minutes)

**[CLICK TO SLIDE 47]**

*[Return to center, open posture]*

Thank you so much for your time and attention.

I'd love to hear your questions. Whether about implementation, challenges you're facing, or anything else - let's discuss.

*[Manage Q&A - repeat questions for the room, keep answers concise]*

---

## Anticipated Q&A

**Q: What if plugins use different frameworks like Vue or Angular?**

A: Module Federation can support multiple frameworks. The key is that they all need to respect your theme system through CSS variables. It's harder but possible.

**Q: How do you handle third-party plugins that don't follow guidelines?**

A: Wrappers. You create a thin wrapper that adds CSS isolation and any needed adaptations without modifying the original plugin.

**Q: What's the performance overhead of Module Federation?**

A: Initial load is slightly heavier because you're loading shared libraries upfront. But it amortizes quickly - with 3+ plugins, you're ahead. The shared libraries are cached.

**Q: How do you version the theme package?**

A: Semantic versioning. Breaking changes in tokens are major versions with deprecation periods. We aim for 6-month migration windows for major changes.

**Q: Can this work with micro-frontends?**

A: Absolutely. Module Federation is often used for micro-frontends. The same principles apply - shared dependencies, CSS isolation, contracts.

---

## Timing Summary

| Section | Duration | Cumulative |
|---------|----------|------------|
| Title + What You'll Learn | 2 min | 2 min |
| Part 1: Universal Challenge | 5 min | 7 min |
| Part 2: Solution Framework | 3 min | 10 min |
| Strategy 1: Theme System | 8 min | 18 min |
| Strategy 2: Module Federation | 6 min | 24 min |
| Strategy 3: Style Isolation | 5 min | 29 min |
| Strategy 4: Extension Contracts | 7 min | 36 min |
| Strategy 5: Configuration | 4 min | 40 min |
| Putting Together + Roadmap | 5 min | 45 min → compress |
| Results + Takeaways | 3 min | 40 min |
| Q&A | 5 min | 45 min |

**Buffer**: Compress "Putting Together" section if running long.
