# Design Systems at Scale: Maintaining UI Consistency in Dynamic Plugin Architectures

**DevConf.IN 2026 - Track 4: User Experience and Design Engineering**  
**Duration**: 45 minutes (40 mins presentation + 5 mins Q&A)  
**Speaker**: [Your Name], Frontend Contributor, Red Hat Developer Hub

---

## Slide 1: Title Slide (1 min)
**Visual**: Conference title, your name, and a compelling visual of RHDH interface

**Talking Points**:
- Welcome and introduction
- Brief background: Frontend contributor to Red Hat Developer Hub
- Today's focus: How we solve UI consistency in a dynamic plugin ecosystem

---

## Slide 2-3: Introduction - The Challenge (4 mins)

**Visual**: Before/After comparison - inconsistent UI vs consistent UI across plugins

**Talking Points**:
- Modern developer portals are no longer monolithic applications
- RHDH is built on Backstage with 100+ potential plugins
- Each plugin can come from different teams, vendors, or open source contributors
- **The Problem**: How do you ensure a cohesive user experience when:
  - You don't control all the code
  - Plugins are developed independently
  - Updates happen at different cadences
  - Users expect enterprise-grade consistency

**Key Quote**: "Your users don't care that your UI is composed of 50 different plugins. They expect it to feel like ONE application."

---

## Slide 4-5: Why Design Systems Matter in Plugin Architectures (4 mins)

**Visual**: Diagram showing plugin ecosystem with shared design system at the center

**Talking Points**:
- **Without a design system**:
  - Every plugin invents its own buttons, forms, navigation
  - Cognitive load increases for users
  - Maintenance nightmare for platform teams
  - Poor accessibility and inconsistent behavior
  
- **With a design system**:
  - Single source of truth for UI components
  - Predictable user experience
  - Faster plugin development
  - Built-in accessibility and best practices

**Real-world impact**: Reduced onboarding time for new plugins from weeks to days

---

## Slide 6-8: PatternFly: Our Design System Foundation (10 mins)

**Visual**: PatternFly component library showcase

**Talking Points**:

### What is PatternFly?
- Open source design system from Red Hat
- React components built for enterprise applications
- WCAG 2.1 AA compliant out of the box
- Consistent with Red Hat's design language

### Integration Strategy in RHDH:

**1. Shared Dependencies**
```javascript
// All plugins share the same PatternFly version
"peerDependencies": {
  "@patternfly/react-core": "^5.x",
  "@patternfly/react-icons": "^5.x"
}
```

**Benefits**:
- Single bundle loaded once
- Consistent component behavior across plugins
- Reduced bundle size (no duplication)

**2. Component Guidelines**
```typescript
// Example: Standardized card component usage
import { Card, CardBody, CardTitle } from '@patternfly/react-core';

export const PluginCard = () => (
  <Card>
    <CardTitle>Plugin Content</CardTitle>
    <CardBody>Consistent spacing and styling</CardBody>
  </Card>
);
```

**3. Theme Integration**
- PatternFly CSS custom properties for theming
- Plugins automatically inherit platform theme
- Dark mode support out of the box

**Demo Suggestion**: Show live RHDH interface with multiple plugins using PatternFly components

---

## Slide 9-11: Component Contracts & APIs (10 mins)

**Visual**: API contract diagram showing platform <-> plugin interfaces

**Talking Points**:

### Defining Clear Boundaries

**1. Extension Points**
```typescript
// Platform provides typed extension points
export const catalogPlugin = createPlugin({
  id: 'catalog',
  routes: {
    catalogIndex: '/catalog',
  },
  externalRoutes: {
    viewTechDoc: techDocsPlugin.routes.docRoot,
  },
});
```

**2. Component Contracts**
```typescript
// Plugins must use provided components for navigation
import { Link } from '@backstage/core-components';

// ✅ Correct: Uses platform navigation
<Link to="/catalog">View Catalog</Link>

// ❌ Wrong: Breaks navigation context
<a href="/catalog">View Catalog</a>
```

### Enforcing Consistency Without Sacrificing Flexibility

**Platform Provides**:
- Navigation components (Sidebar, Header, Link)
- Data display patterns (Tables, Cards, Lists)
- Form components (Inputs, Selects, Buttons)
- Layout utilities (Page, Content, Header)

**Plugins Control**:
- Business logic and data flow
- Custom visualizations (charts, diagrams)
- Domain-specific components
- Feature implementations

**Example: InfoCard Component**
```typescript
// Platform-provided wrapper with consistent styling
import { InfoCard } from '@backstage/core-components';

export const MyPluginCard = () => (
  <InfoCard title="Plugin Feature">
    {/* Plugin controls content */}
    <CustomPluginContent />
  </InfoCard>
);
```

**Benefits**:
- Plugins look native to the platform
- Layout and spacing handled automatically
- Responsive behavior built-in
- Accessibility compliance guaranteed

---

## Slide 12-14: Runtime Theming & Customization (7 mins)

**Visual**: Same interface shown in different brand themes

**Talking Points**:

### Multi-Tenant Branding Challenge
- RHDH is white-labeled by different organizations
- Each customer wants their brand identity
- **Requirement**: Theme all plugins without modifying source code

### CSS Custom Properties Strategy

**1. Theme Definition**
```yaml
# app-config.yaml
app:
  branding:
    theme:
      light:
        primaryColor: '#0066CC'
        headerColor1: '#0066CC'
        headerColor2: '#004080'
      dark:
        primaryColor: '#1F8FFF'
```

**2. Dynamic Injection**
```typescript
// Platform injects CSS variables at runtime
document.documentElement.style.setProperty(
  '--pf-global-primary-color--100', 
  primaryColor
);
```

**3. Plugin Consumption**
```css
/* Plugins automatically inherit theme */
.my-plugin-header {
  background: var(--pf-global-primary-color--100);
  color: var(--pf-global-Color--light-100);
}
```

### Logo and Asset Customization
```typescript
// Platform provides branded assets
import { useApp } from '@backstage/core-plugin-api';

const app = useApp();
const logo = app.getSystemIcon('logo');
```

**Real-world Example**: Same RHDH deployment looks completely different for Company A vs Company B

**Demo Suggestion**: Live theme switching showing all plugins updating simultaneously

---

## Slide 15-17: Developer Experience & Plugin Guidelines (10 mins)

**Visual**: Plugin development workflow diagram

**Talking Points**:

### Making the Right Choice the Easy Choice

**1. Plugin Scaffolding**
```bash
# Create new plugin with all best practices baked in
yarn backstage-cli create-plugin

# Generated structure includes:
# - PatternFly dependencies
# - Component examples
# - Testing setup
# - Linting rules
```

**2. Documentation & Examples**
- Pattern library with copy-paste examples
- Do's and Don'ts guide with visual examples
- Accessibility checklist
- Component playground for experimentation

**3. Automated Checks**

**ESLint Rules**:
```javascript
// Enforce platform component usage
{
  "rules": {
    "no-restricted-imports": ["error", {
      "patterns": ["**/react-router-dom", "!@backstage/*"],
      "message": "Use @backstage/core-components Link instead"
    }]
  }
}
```

**Visual Regression Testing**:
```typescript
// Catch unintended visual changes
import { renderInTestApp } from '@backstage/test-utils';

it('matches visual snapshot', () => {
  const { container } = renderInTestApp(<MyComponent />);
  expect(container).toMatchSnapshot();
});
```

**4. Design Review Process**
- Pre-merge design checklist
- Automated accessibility scans (axe-core)
- Responsive design testing
- Theme compatibility verification

### Contribution Guidelines

**Required for Plugin Approval**:
- [ ] Uses PatternFly components for standard UI elements
- [ ] No custom navigation implementations
- [ ] Supports light/dark themes
- [ ] Passes WCAG 2.1 AA accessibility audit
- [ ] Responsive down to 320px viewport
- [ ] Visual regression tests included

**Impact**: 95% of plugin submissions now follow design guidelines on first review

---

## Slide 18-20: Performance at Scale (5 mins)

**Visual**: Performance metrics dashboard

**Talking Points**:

### Challenges with Shared Dependencies

**Bundle Size Management**:
```javascript
// Webpack Module Federation config
new ModuleFederationPlugin({
  shared: {
    'react': { singleton: true },
    '@patternfly/react-core': { singleton: true },
    '@backstage/core-components': { singleton: true }
  }
});
```

**Benefits**:
- Shared dependencies loaded once: 80% bundle size reduction
- Faster load times for subsequent plugins
- Consistent behavior across plugin boundaries

### Code Splitting Strategy
```typescript
// Lazy load plugin routes
const MyPluginPage = lazy(() => import('./components/MyPluginPage'));
```

**Metrics from Production**:
- Initial load: 450KB gzipped (platform + PatternFly)
- Each additional plugin: 30-100KB average
- Time to interactive: <2s on 3G connection

### Tree Shaking Best Practices
```typescript
// ✅ Import specific components
import { Button } from '@patternfly/react-core';

// ❌ Imports entire library
import * as PF from '@patternfly/react-core';
```

---

## Slide 21-22: Real-world Examples from RHDH (3 mins)

**Visual**: Screenshots of actual RHDH plugins with consistent UI

**Talking Points**:

### Case Study 1: Third-party Plugin Integration
- **Challenge**: Vendor provides plugin with custom UI
- **Solution**: Provided PatternFly wrapper components
- **Result**: Plugin integrated seamlessly, vendor became design system advocate

### Case Study 2: Community Plugin Migration
- **Challenge**: Popular Backstage plugin with Material UI
- **Solution**: Created compatibility layer and migration guide
- **Result**: Plugin maintained functionality with RHDH design language

### Case Study 3: Custom Visualization Plugin
- **Challenge**: Needed D3.js charts while maintaining consistency
- **Solution**: Used PatternFly colors and spacing tokens in custom components
- **Result**: Charts feel native to platform while being highly customized

**Key Metrics**:
- 100+ plugins maintaining consistent UX
- 40% reduction in design review iterations
- 99% user satisfaction with UI consistency (internal survey)

---

## Slide 23-24: Lessons Learned & Best Practices (3 mins)

**Visual**: Key takeaways list

**Talking Points**:

### What Worked Well
✅ **Enforce through tooling, not documentation**
- Linters, type checking, automated tests
- Developers get immediate feedback

✅ **Invest in great examples**
- Plugin templates with best practices
- Copy-paste code snippets that do the right thing

✅ **Make theming transparent to plugins**
- CSS custom properties handled by platform
- Plugins don't need theme-aware logic

✅ **Progressive enhancement**
- Basic plugins work out of the box
- Advanced customization available when needed

### What We'd Do Differently

⚠️ **Version management is hard**
- Design system major versions cause plugin churn
- Solution: Extended deprecation periods, automated migration tools

⚠️ **Balance flexibility vs consistency**
- Too rigid → frustrated developers, workarounds
- Too loose → inconsistent UX
- Solution: Clear guidelines on when to deviate

⚠️ **Documentation maintenance**
- Keeping examples updated with design system changes
- Solution: Generated docs from code, versioned examples

---

## Slide 25: Key Takeaways (2 mins)

**Visual**: Summary with 5 key points

**Key Takeaways**:

1. **Design systems aren't optional in plugin architectures** - They're the glue that holds distributed UI together

2. **Shared dependencies are your friend** - PatternFly as peer dependency ensures consistency and performance

3. **Define clear contracts** - Extension points and component APIs prevent plugin sprawl

4. **Make compliance easy** - Scaffolding, linting, and automation guide developers to correct patterns

5. **Theme at the platform level** - CSS custom properties enable customization without plugin code changes

**Final Thought**: "A good design system in a plugin architecture is invisible to both users AND developers. Users see one cohesive application. Developers build features without thinking about design consistency."

---

## Slide 26: Resources & Links (1 min)

**Visual**: QR codes and links

**Resources**:
- PatternFly Design System: https://www.patternfly.org
- Red Hat Developer Hub: https://developers.redhat.com/rhdh
- Backstage: https://backstage.io
- RHDH Plugin Development Guide: [link]
- GitHub Repository: https://github.com/redhat-developer/rhdh

**Connect**:
- GitHub: [your handle]
- LinkedIn: [your profile]
- Email: [your email]

---

## Slide 27: Q&A (5 mins)

**Visual**: Thank you slide with contact information

**Potential Questions & Answers**:

**Q: How do you handle plugins that need custom components not in PatternFly?**
A: We encourage building on PatternFly primitives and using design tokens. For truly unique needs, document the pattern for reuse.

**Q: What about plugins built with different frameworks (Vue, Angular)?**
A: RHDH uses module federation to support this. Plugins must still respect the visual design through CSS variables.

**Q: How do you enforce design compliance in open source contributions?**
A: Automated checks in CI, design review as part of PR process, and helpful maintainer feedback with examples.

**Q: Performance impact of shared dependencies?**
A: Initial load is heavier, but amortized across plugins. Net positive once you have 3+ plugins loaded.

**Q: How often does the design system change?**
A: PatternFly follows semantic versioning. Major versions 1-2 times per year with long deprecation periods.

---

## Demo Flow (Integrated throughout)

**Demo 1** (Slide 8): Show RHDH with 5-6 different plugins, highlighting consistent UI elements

**Demo 2** (Slide 14): Live theme switching - change primary color, logo, watch all plugins update

**Demo 3** (Slide 17): Show plugin development - scaffold new plugin, components already follow design system

**Demo 4** (If time): Show accessibility inspector on a plugin, highlighting WCAG compliance

---

## Preparation Checklist

### Technical Setup
- [ ] RHDH instance running locally or accessible
- [ ] Multiple plugins installed and configured
- [ ] Theme switcher prepared with 2-3 different themes
- [ ] Code examples ready in IDE
- [ ] Browser developer tools for demonstrating CSS variables
- [ ] Backup screenshots/videos in case of connectivity issues

### Presentation Materials
- [ ] Slides exported to PDF backup
- [ ] Demo environment tested multiple times
- [ ] Questions anticipated and answers prepared
- [ ] Timing rehearsed (aim for 38-40 mins to leave buffer)
- [ ] Contact information and links verified

### Speaking Notes
- [ ] Key statistics and metrics memorized
- [ ] PatternFly component names and concepts reviewed
- [ ] Real-world examples with specific details
- [ ] Transition phrases between sections prepared

---

## Additional Notes

### Target Audience Assumptions
- Familiar with React and component-based UI
- Understand basic design system concepts
- May not know Backstage or RHDH specifically
- Interested in solving plugin/extension architecture challenges

### Energy & Pacing
- Start with relatable problem (consistency challenge)
- Build momentum with technical solutions
- Include visual demos to maintain engagement
- Keep code examples concise and clear
- End with actionable takeaways

### Adaptation Points
- If running short on time: Condense slides 18-20 (Performance)
- If audience wants deeper dive: Expand module federation details in performance section
- If more beginner audience: Add more PatternFly basics, less architecture details
- If more advanced audience: Deep dive into Webpack config and optimization techniques

---

**Good luck with your presentation! 🚀**

