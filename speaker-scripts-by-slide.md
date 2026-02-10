# Speaker Scripts by Slide — Simple & Descriptive

This file gives you a **simple, descriptive speaker script for each slide** of the Design Systems at Scale presentation, plus a **transition** line to move smoothly to the next slide. Use it when you want to say things in plain language, with a bit more explanation built in. Each slide has: (1) **Speaker script**—what to say for that slide; (2) **Transition**—a short phrase or sentence to lead into the next slide. You can read the script aloud and use the transition when you advance.

**Source:** Based on [speaker-notes-detailed.md](speaker-notes-detailed.md).  
**Deck:** 39 slides. **Total time:** ~40 minutes + Q&A.

---

## Part 1: Opening and Agenda (Slides 1–2)

### SLIDE 1: Title (1 minute)

**Speaker script:**

Good morning (or afternoon), everyone. Welcome to this session: "Design Systems at Scale: Maintaining UI Consistency in Dynamic Plugin Architectures." I'm Mitesh Kumar, and I work on the frontend for Red Hat Developer Hub. Today I want to give you a practical guide for anyone building a platform that uses plugins—where lots of separate pieces plug into one app. Whether you're building an IDE, a developer portal, or any app that can be extended with plugins, the challenges we'll talk about are the same. So this is for you.

**Transition:** So that's the intro. Next, here's what we'll cover in this session.

---

### SLIDE 2: What You'll Learn (1 minute)

**Speaker script:**

Here's what we'll cover. On the left side, the **challenge**: why it's so hard to keep the UI consistent when you have dozens or even hundreds of plugins built by different teams. On the right side, the **solutions**: five strategies that really work, with patterns you can take back and use with your own teams. I'll use Red Hat Developer Hub as our main example—we run more than a hundred plugins in production—but the same ideas apply to any plugin-based system you're building.

**Transition:** First, let's align on what we mean by a plugin architecture.

---

## Part 2: The Challenge (Slides 3–6)

### SLIDE 3: What is Plugin Architecture? (1 minute)

**Speaker script:**

First, what do we mean by a plugin architecture? Think of it like this. You have one **core application**—we call it the host. It’s the shell: the layout, the navigation, the place where everything lives. Then other people—other teams, or even other companies—build **plugins** that slot into that shell. Plugin A might add CI/CD. Plugin B might add security scanning. Plugin N might do something else. Each plugin adds a capability. But here’s the key: they all run **inside** the same host. So your user sees one application. They don’t see “twenty different tools”—they see one product with many features. The twist is that those features can be built by completely different people. Your team, the community, third-party vendors. That’s the power—and that’s also why keeping it consistent is hard.

**Transition:** So why are plugin architectures everywhere? Let's talk about that.

---

### SLIDE 4: Why Plugin Architectures? (2 minutes)

**Speaker script:**

Quick show of hands—how many of you work on an application that has some kind of plugin or extension system? [Pause.] Quite a few. And there's a good reason. Plugin architectures are everywhere. Think about it: VS Code has tens of thousands of extensions. Figma has thousands of plugins. WordPress has tens of thousands. Why? Because plugins solve a core business problem: **you can extend the product without everything going through one central team**. Third parties can build features. Different teams can ship at their own speed. You get an ecosystem instead of one big monolith. But that power comes with a challenge, and that's what we'll tackle next.

**Transition:** That challenge is consistency. Here's the problem.

---

### SLIDE 5: The Consistency Problem (2 minutes)

**Speaker script:**

Here's the fundamental tension: **Your users don't care that your UI is built from 50 different plugins. They expect it to feel like one application.** So what goes wrong? On the left: different teams make different design choices—different blues, different spacing. Third-party plugins might ignore your guidelines. Dependencies get out of sync—one plugin uses one version of the UI library, another uses a different one. CSS from one plugin clashes with another. Accessibility is inconsistent. On the right, the cost: users get confused, support tickets go up, your brand looks inconsistent, and maintenance becomes a nightmare. And here's the worst part: **users blame your platform, not the plugins.** So it's in your interest to get this right.

**Transition:** Let me show you the difference visually—before and after.

---

### SLIDE 6: Before vs After (1 minute)

**Speaker script:**

Picture the difference. Without a strategy: buttons look different in different places, forms behave differently, navigation doesn't feel the same. Users feel lost. With a strategy: one set of components, consistent behavior, one visual language. Users feel confident. Your goal is to make plugins **invisible** to users. They should never think, "This part feels like a different app." It should all feel like one product.

**Transition:** So how do we get there? Here are the five strategies that work together.

---

## Part 3: The Five Strategies Overview and Case Study (Slides 7–8)

### SLIDE 7: Five Strategies (2 minutes)

**Speaker script:**

Now let's look at the solution—five strategies that work together. **One: Central Theme System.** One place that defines colors, fonts, spacing. That's your foundation. **Two: Runtime Dependency Sharing.** Use something like Module Federation so every plugin uses the *same* React and the *same* UI library at runtime—one instance of each, not many. **Three: Style Isolation.** Stop one plugin's CSS from breaking another's—with things like class prefixes or Shadow DOM. **Four: Extension Contracts.** Clear rules for where and how plugins plug in: mount points, routes, typed APIs. **Five: Configuration-Driven Customization.** Let theming and features be controlled by config, not code, so you can change look and behavior without redeploying. We'll go deep on each of these.

**Transition:** I'll ground all of this in a real case study—Red Hat Developer Hub.

---

### SLIDE 8: Case Study Introduction (1 minute)

**Speaker script:**

Through this talk I'll use Red Hat Developer Hub as our case study. RHDH is an internal developer portal built on Backstage—the open platform from Spotify. We have more than a hundred plugins: from Red Hat teams, from the Backstage community, and from third-party vendors. Our challenge was making all of them look and feel like **one** product. Everything I show you comes from real production code we run today.

**Transition:** Let's start with Strategy One: the central theme system.

---

## Part 4: Strategy 1 – Central Theme System (Slides 9–13)

### SLIDE 9: Strategy 1 – Central Theme System: The Principle (2 minutes)

**Speaker script:**

Strategy One is the Central Theme System. This is where you should start. The idea is simple: **create one package—or one module—that controls all visual styles.** What does it control? Colors: primary, secondary, status colors like error and success. Typography: font family, sizes, weights. Spacing: your 4px, 8px, 16px system so everyone uses the same rhythm. How components look: buttons, cards, tables. And support for both light and dark mode. This is your foundation. Get this right first; everything else builds on it.

**Transition:** Why does centralizing matter so much? Here's what happens when you don't.

---

### SLIDE 10: Why Centralize (2 minutes)

**Speaker script:**

Here's what happens if you don't centralize. Plugin A uses one shade of blue. Plugin B uses another. Plugin C just picks "blue." The platform uses a fourth shade. Result? Four different blues on the same screen. Users notice. With a central theme package, there's **one** definition of "primary blue." Every plugin gets it from the same place. One blue everywhere. It sounds obvious, but a lot of teams skip it and let each plugin define its own colors. That's a disaster waiting to happen.

**Transition:** Here's the concrete implementation pattern.

---

### SLIDE 11: Implementation Pattern – Theme (2 minutes)

**Speaker script:**

Here's the pattern. Define a **TypeScript interface** for your theme—so the shape of your palette and tokens is fixed and typed. Start with core colors: primary, secondary. Add semantic colors: error, warning, success, info—so plugins don't hardcode red or green. Then add **component-specific tokens**: sidebar background, card borders, table headers. Aim for around 20 to 30 tokens to start; you can add more later. The important thing is: define everything in one place. That way the compiler and your IDE keep everyone in sync.

**Transition:** In RHDH we put this into practice—here's how it looks.

---

### SLIDE 12: RHDH Example – Theme (2 minutes)

**Speaker script:**

In RHDH we have a dedicated theme plugin with 30-plus color tokens. What's interesting is the **ThemeConfigOptions** type. It lets you choose between PatternFly and MUI styling **per component**. So buttons can be PatternFly style and tables MUI style. The platform decides—not the plugins. That gives us flexibility while keeping control. We use a **useThemes()** hook so the platform and plugins all get theme data from one place.

**Transition:** If you remember one thing from the theme section, it's this takeaway.

---

### SLIDE 13: Takeaway – Theme (30 seconds)

**Speaker script:**

If you take one thing from this talk, make it this: **invest in a solid theme package early. It's your foundation.** Quick checklist: one package, at least 20–30 tokens, component-specific tokens, light and dark mode, TypeScript types, and a central hook or API. Do that first.

**Transition:** Now Strategy Two: runtime dependency sharing—and the number one cause of plugin bugs.

---

## Part 5: Strategy 2 – Runtime Dependency Sharing (Slides 14–18)

### SLIDE 14: Strategy 2 – Runtime Dependency Sharing: The Problem with Bundling (2 minutes)

**Speaker script:**

Strategy Two is Runtime Dependency Sharing. This is the number one cause of plugin bugs. If each plugin bundles its **own** React, you end up with multiple React instances on the page at the same time. So you might have four copies of React loaded. Hooks don't work across those instances—you get those cryptic "Invalid hook call" errors. Bundle size blows up. **React has to be a singleton**—one instance for the whole app. Same for any stateful library like Redux or the router.

**Transition:** The fix is Module Federation—sharing at runtime.

---

### SLIDE 15: The Solution – Module Federation (2 minutes)

**Speaker script:**

The solution is Module Federation—sharing libraries at **runtime**, not at build time. The host application loads React, ReactDOM, MUI, the router—once. Plugins don't bundle their own copies; they get these from the host when they run. So you have a single instance of everything. One React, one MUI, one router. Problem solved.

**Transition:** What should you actually share? And what should plugins keep in their own bundle?

---

### SLIDE 16: What to Share (1 minute)

**Speaker script:**

What should you share? React and ReactDOM—absolutely. The UI library—yes. The router—yes. State management—yes. Core platform components—yes. What should plugins bundle themselves? Things only they use: plugin-specific libraries, things like D3 for charts, heavy utilities. Rule of thumb: share anything that **must** be a singleton, or that more than half your plugins use.

**Transition:** Here's how you set it up with Webpack.

---

### SLIDE 17: Implementation – Webpack Module Federation (2 minutes)

**Speaker script:**

Here's how you set it up with Webpack Module Federation. In your webpack config you add the **ModuleFederationPlugin**. You list your shared dependencies—React, ReactDOM, MUI, router. The **singleton: true** flag is crucial: it forces exactly one instance at runtime. **requiredVersion** adds safety: if a plugin was built for React 17 and the host has React 18, you get a warning instead of silent bugs. This is built into Webpack 5—no extra libraries needed.

**Transition:** In RHDH we use Scalprum on top of that—quick look.

---

### SLIDE 18: RHDH Scalprum (1 minute)

**Speaker script:**

In RHDH we use Scalprum, which sits on top of Module Federation and adds some useful features. For example, we transform plugin manifests so that script URLs point to our **backend API** instead of a static CDN. So plugins are served dynamically. We can update a plugin by deploying just that plugin—we don't have to rebuild the host application. That makes iteration faster and safer.

**Transition:** Strategy Three is style isolation—because even with one MUI, CSS can still fight.

---

## Part 6: Strategy 3 – Style Isolation (Slides 19–22)

### SLIDE 19: Strategy 3 – Style Isolation: The CSS Problem (2 minutes)

**Speaker script:**

Strategy Three is Style Isolation. CSS is global by default. So if Plugin A styles `.MuiButton-root` with red and Plugin B styles the same class with blue, which wins? It depends on load order—and that can change. It's unpredictable. Even when you share MUI through Module Federation, different plugins can still add their own overrides. Those overrides conflict. So sharing the library is necessary but not enough; you need isolation at the **style** level too.

**Transition:** Here are two main techniques that work.

---

### SLIDE 20: Isolation Techniques (2 minutes)

**Speaker script:**

Two main techniques. **One: Class name prefixing.** MUI has a ClassNameGenerator. You configure it to add a prefix. So instead of every plugin using `.MuiButton-root`, Plugin A gets `.plugin-tekton-MuiButton-root` and Plugin B gets something different. No more conflicts. **Two: Shadow DOM.** You create a shadow root and render the plugin inside it. Styles inside don't leak out; outside styles don't leak in. Shadow DOM gives you the strongest isolation but is more involved—theming and debugging can be trickier. Choose based on how much isolation you need.

**Transition:** A few more options we use in practice.

---

### SLIDE 21: More Techniques (1 minute)

**Speaker script:**

A few more options. **CSS Modules** give your classes unique hashes at build time—good for avoiding clashes inside a plugin, but you still need a plan for shared libraries like MUI. **BEM with a plugin prefix**—like tekton-button, rbac-table—works if everyone follows the convention, but it's easy to break when people are in a hurry. In RHDH we use the **ClassNameGenerator approach in thin wrappers** around each plugin. More than 14 plugins use this pattern, and we get isolation without forking the upstream code.

**Transition:** Here's that wrapper pattern in a bit more detail.

---

### SLIDE 22: RHDH Wrapper Example (1 minute)

**Speaker script:**

Here's the wrapper pattern. We import the ClassNameGenerator, configure it with a prefix for that plugin, then re-export the original plugin unchanged. The wrapper adds isolation without touching the upstream code. That's a big deal—you can integrate third-party plugins without forking them. You stay in control of the design system while still using community or vendor plugins.

**Transition:** Strategy Four is extension contracts—the rules of engagement between platform and plugins.

---

## Part 7: Strategy 4 – Extension Contracts (Slides 23–27)

### SLIDE 23: Strategy 4 – Extension Contracts: The Principle (2 minutes)

**Speaker script:**

Strategy Four is Extension Contracts. Without contracts, plugins can do anything—render anywhere, replace layout, depend on internal APIs. Upgrading the host can break plugins; changing a plugin can break the shell. There's no clear boundary. With contracts, the platform defines **extension points**—named slots like "entity overview" or "settings page." Plugins register what they provide for which slot. The platform controls layout and navigation; plugins only fill slots. Integration becomes stable and predictable. Contracts are your **rules of engagement** between platform and plugins.

**Transition:** The main idea is mount points—named slots in the layout.

---

### SLIDE 24: Mount Points (2 minutes)

**Speaker script:**

Mount points are **named slots** where plugins can inject UI. The platform defines the slots: page header, sidebar, main content, entity overview. Plugins say things like: "Put my overview card in entity.overview." The platform controls the structure—where the slots are. Plugins only provide the content. So plugins can't break the layout, because they can only use the slots you've defined. They can't move the sidebar or remove the header; they just fill in the blanks.

**Transition:** Beyond UI slots, you have routes and API extensions.

---

### SLIDE 25: More Extension Points (1 minute)

**Speaker script:**

You can extend the idea beyond UI slots. **Routes** can be declared by plugins too—path, component, and optionally a menu item. The host's router uses these so navigation and deep links work without the host hardcoding every plugin route. **API extensions** let plugins provide or extend platform APIs using a factory pattern. The platform exposes a stable API layer so other plugins or the host can use them in a typed way. All of this goes through the same contract model.

**Transition:** TypeScript makes these contracts enforceable at build time.

---

### SLIDE 26: TypeScript Enforcement (2 minutes)

**Speaker script:**

TypeScript enforces contracts at **compile time**. The platform defines something like **MountPointComponent**—what a plugin must provide: the component, optional layout config, optional conditions like "only show for this entity kind." When plugins implement this interface, TypeScript catches mistakes right away—missing or wrong properties—when you build, not when the app is running. So the types are your contract, and they double as documentation that stays in sync with the code.

**Transition:** In RHDH, plugin integration is driven entirely by YAML—no code changes.

---

### SLIDE 27: YAML Configuration (2 minutes)

**Speaker script:**

In RHDH, plugin integration is **entirely declarative**. This YAML configures the Tekton plugin: mount points, layout, conditions, routes, menu items. Want to add a plugin? Edit the YAML. Want to remove it? Delete the lines. No code changes. No rebuilds. Operations teams can enable or disable plugins, or change where they show up, without involving developers. That's a big win for multi-tenant or gradual rollouts.

**Transition:** Strategy Five is configuration-driven customization—theming and features without code.

---

## Part 8: Strategy 5 – Configuration-Driven Customization (Slides 28–30)

### SLIDE 28: Strategy 5 – Configuration-Driven: The Principle (1 minute)

**Speaker script:**

Strategy Five is Configuration-Driven Customization. Why config over code? Code changes need developers, a PR, a build, a deploy. Config can often be changed by ops or product, sometimes at runtime. Code is hard to A/B test; config makes experiments easy. For multi-tenant platforms, configuration is essential—different tenants can get different logos, colors, and features without separate codebases.

**Transition:** What should actually go in that config?

---

### SLIDE 29: What Should Be Configurable (2 minutes)

**Speaker script:**

What should be in that config? **Logos**—with light and dark variants so they work on different backgrounds. **Theme colors**—primary, sidebar background, for both light and dark modes. **Feature flags**—like enable dark mode, compact sidebar. All of that in a YAML or JSON file. No code. The app reads the config and applies it through your theme system and feature APIs.

**Transition:** On the platform side, hooks make that config easy to consume.

---

### SLIDE 30: Theme Hooks (1 minute)

**Speaker script:**

On the platform side, **hooks** hide where the config lives. Something like `useSidebarBackground` reads from the theme and returns the right value. Components use the hook instead of reading raw config. You can also respect the user's OS preference—light or dark—so the app follows system theme automatically. One place for config, one set of hooks to use it.

**Transition:** Let's see how all five strategies fit together in one architecture.

---

## Part 9: Putting It Together and Roadmap (Slides 31–34)

### SLIDE 31: Putting It Together – Complete Architecture (2 minutes)

**Speaker script:**

Let's see how it all fits. At the top: the **theme package**—colors, typography, component styles. Your foundation. Next: the **Module Federation host**—it shares React, the UI library, the router. Then the **extension point system**—mount points, routes, API factories. Below that: **plugins**, each with a thin wrapper for CSS isolation. At the bottom: **configuration**—theme values, plugin config, feature flags. This is the full picture. All five strategies working together.

**Transition:** The wrapper pattern in a bit more detail.

---

### SLIDE 32: Wrapper Pattern (1 minute)

**Speaker script:**

The wrapper in a bit more detail. It's just a thin layer: your package.json declares the exports, and your index file sets up CSS isolation—like the ClassNameGenerator prefix—and re-exports the original plugin. That way you can integrate third-party plugins without forking them. You get isolation without modifying their code.

**Transition:** If you're starting from scratch, here's a phased roadmap.

---

### SLIDE 33: Implementation Roadmap (2 minutes)

**Speaker script:**

If you're starting from scratch, think in phases. **Phase 1—Foundation:** Theme package, Module Federation, and a few mount points. **Phase 2—Isolation:** Your CSS isolation strategy, a wrapper template, and documented contracts. **Phase 3—Configuration:** YAML theming, hooks, light and dark. **Phase 4—Enforcement:** TypeScript contracts, linting, maybe visual regression tests. Don't try to do everything at once. Start with the foundation; the rest builds on it.

**Transition:** And a few common pitfalls to avoid.

---

### SLIDE 34: Common Pitfalls (1 minute)

**Speaker script:**

Learn from common mistakes. No theme system leads to inconsistent colors. Bundling shared deps in every plugin leads to multiple React instances. No CSS isolation leads to style conflicts. No contracts mean plugins break things in unpredictable ways. Code-only config means every change needs a developer and a deploy. Each of these has a solution—and we've covered them all in the five strategies.

**Transition:** What does this get you in practice? Here are some real numbers.

---

## Part 10: Results and Closing (Slides 35–39)

### SLIDE 35: Results – Metrics (2 minutes)

**Speaker script:**

Here are some real results. **Plugin onboarding:** from two or three weeks down to two or three days. That's a big change. **UI inconsistency bugs:** from about 15 per month to about 2. Users notice the consistency. **Bundle size** for five plugins: from 2.8 MB down to 850 KB—shared dependencies really pay off. **Design review rounds:** from four or five down to one or two, because plugins align with the system faster. **Theme changes:** from days of code changes to minutes of editing config. We have over a hundred plugins in production keeping a consistent UX. These strategies work.

**Transition:** Five things to take away.

---

### SLIDE 36: Key Takeaways (2 minutes)

**Speaker script:**

Five things to take away. **One:** Invest in a central theme system early; it's your foundation. **Two:** Use Module Federation for runtime sharing—one React instance. **Three:** Implement style isolation; otherwise CSS conflicts are inevitable. **Four:** Define clear extension contracts—TypeScript and mount points. **Five:** Make it configuration-driven so operations teams can change behavior without code. They'll thank you.

**Transition:** One final thought before we wrap up.

---

### SLIDE 37: Final Thought (30 seconds)

**Speaker script:**

I'll leave you with this: A good design system in a plugin architecture is **invisible**. Users see one cohesive application. Developers build features without constantly fighting consistency. That invisibility is the goal. When you get there, everyone wins—users, developers, and ops.

**Transition:** Here are some resources to go deeper.

---

### SLIDE 38: Resources (30 seconds)

**Speaker script:**

For more: Backstage.io for the open source platform. RHDH on GitHub for our implementation—theme, Scalprum, wrappers, config. Module Federation docs for the technical details. I'm happy to connect—my handles are on the slide. Slides and examples will be available at the GitHub link. Thank you.

**Transition:** Thank you—and over to you for questions.

---

### SLIDE 39: Thank You & Q&A (5 minutes)

**Speaker script:**

Thank you for your time and attention. I'd love to hear your questions—about implementation, challenges you're facing, or anything else. Let's discuss. [Repeat questions for the room, keep answers concise, offer to go deeper offline or via the repo.]

**Transition:** [None—open the floor for Q&A.]

---

**End of speaker scripts by slide.**
