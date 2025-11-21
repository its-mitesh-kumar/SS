# Dynamic Frontend Plugin Architecture in Kubernetes-Native Developer Portals

**DevConf.IN 2026 - Track 2: Cloud, Edge, and Sustainable Computing**  
**Duration**: 45 minutes (40 mins presentation + 5 mins Q&A)  
**Speaker**: [Your Name], Frontend Contributor, Red Hat Developer Hub

---

## Slide 1: Title Slide (1 min)

**Visual**: RHDH logo with dynamic plugin architecture diagram overlay

**Talking Points**:
- Welcome and introduction
- Frontend contributor to Red Hat Developer Hub (RHDH)
- Today's topic: How we made our frontend as dynamic as our backend microservices
- Bridging frontend development with cloud-native principles

---

## Slide 2-3: Introduction - The Traditional Frontend Problem (4 mins)

**Visual**: Timeline showing traditional monolithic frontend deployment cycle

**Talking Points**:

### The Old Way: Monolithic Frontends
```bash
# Traditional deployment cycle
1. Developer makes change to single feature
2. Rebuild ENTIRE frontend application
3. Run full test suite (30+ minutes)
4. Deploy complete bundle to production
5. All users download 2MB+ JavaScript
6. One bug? Repeat entire process
```

**Problems**:
- **Coupling**: One team's change requires everyone to redeploy
- **Slow feedback**: Deploy cycles measured in hours or days
- **Bundle bloat**: Users download code they never use
- **Risk**: Single deployment can break entire application
- **Scalability**: Doesn't scale across multiple teams

**The Question**: "Our backend has microservices, containers, and Kubernetes. Why is our frontend still a monolith?"

---

## Slide 4-5: Vision - Dynamic Frontend Plugins (3 mins)

**Visual**: Side-by-side comparison diagram: Monolithic vs Dynamic Plugin Architecture

**Talking Points**:

### What if Frontend Worked Like Backend?

**Backend (Already Dynamic)**:
- Services deployed independently
- Containers in OCI registries
- Kubernetes orchestrates at runtime
- Teams ship when ready
- Scale independently

**Frontend (Vision)**:
- Plugins deployed independently ✅
- Packaged as container images ✅
- Loaded dynamically at runtime ✅
- Teams ship when ready ✅
- Composed on-demand ✅

**Key Insight**: "We needed frontend plugins that behave like microservices, not just feel like them."

---

## Slide 6-8: Architecture Overview (5 mins)

**Visual**: High-level architecture diagram showing all components

**Talking Points**:

### RHDH Dynamic Plugin Architecture Components

**1. Core Platform (Base Container)**
```dockerfile
# Minimal platform with plugin loader
FROM registry.redhat.io/nodejs-18
COPY app/ /opt/app/
# NO plugins bundled - loaded at runtime
```

**2. Plugin Containers (OCI Images)**
```dockerfile
# Each plugin is a separate container
FROM scratch
COPY dist/ /
LABEL plugin.id="my-plugin"
LABEL plugin.version="1.0.0"
```

**3. Plugin Registry**
- Standard OCI-compliant container registry
- Same infrastructure as backend services
- `quay.io/rhdh-plugins/catalog-plugin:v1.2.3`

**4. Dynamic Plugin Loader**
- Runs in RHDH platform at startup
- Discovers configured plugins
- Downloads and loads at runtime
- No rebuild required

### Flow Overview
```
Startup → Read Config → Download Plugins → Load Modules → Render UI
```

**Key Benefit**: Platform team ships once, plugin teams iterate independently

---

## Slide 9-13: Module Federation Deep Dive (10 mins)

**Visual**: Webpack Module Federation diagram with host and remotes

**Talking Points**:

### What is Module Federation?

**Traditional JavaScript Modules**:
- All code bundled together at build time
- Import paths resolved statically
- One big bundle

**Module Federation**:
- Load JavaScript modules from remote sources at runtime
- Share dependencies across boundaries
- Multiple independent builds working together

### RHDH Implementation

**1. Platform as Host**
```javascript
// webpack.config.js (Platform)
new ModuleFederationPlugin({
  name: 'platform',
  shared: {
    react: { singleton: true, requiredVersion: '^18.0.0' },
    'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
    '@backstage/core-plugin-api': { singleton: true },
  },
});
```

**Key Configuration**:
- **singleton**: Only one version loaded (prevents React duplicate issues)
- **requiredVersion**: Ensures compatibility
- **shared**: Dependencies provided by platform, not duplicated in plugins

**2. Plugins as Remotes**
```javascript
// webpack.config.js (Plugin)
new ModuleFederationPlugin({
  name: 'catalogPlugin',
  filename: 'remoteEntry.js',
  exposes: {
    './plugin': './src/plugin.ts',
  },
  shared: {
    react: { singleton: true },
    'react-dom': { singleton: true },
    '@backstage/core-plugin-api': { singleton: true },
  },
});
```

**Exposes**:
- Plugin defines what it exports
- `remoteEntry.js` is the plugin's manifest
- Platform knows how to import plugin

**3. Runtime Loading**
```typescript
// Dynamic import at runtime
const loadPlugin = async (pluginUrl: string) => {
  // Inject script tag
  await loadRemoteModule(pluginUrl);
  
  // Import exposed module
  const plugin = await import('catalogPlugin/plugin');
  
  // Register with platform
  app.registerPlugin(plugin.default);
};
```

### Dependency Resolution

**Scenario: Version Mismatch**
```
Platform: React 18.2.0
Plugin A: React 18.2.0 → ✅ Uses platform version
Plugin B: React 18.1.0 → ✅ Compatible, uses platform version  
Plugin C: React 17.0.0 → ❌ Incompatible, build fails
```

**Solution**: Peer dependency strategy
```json
{
  "peerDependencies": {
    "react": "^18.0.0"
  }
}
```

### Bundle Size Impact

**Traditional Monolith**:
```
Initial Load: 2.5MB
  - Platform: 1.0MB
  - Plugin 1: 300KB (includes React copy)
  - Plugin 2: 400KB (includes React copy)
  - Plugin 3: 350KB (includes React copy)
  - React duplicated 3 times!
```

**Module Federation**:
```
Initial Load: 1.4MB
  - Platform: 1.0MB (includes React once)
  - Plugin 1: 150KB (no React)
  - Plugin 2: 180KB (no React)
  - Plugin 3: 120KB (no React)
  
Lazy Load: Plugin 4: 100KB (when needed)
```

**Savings**: 44% reduction + lazy loading capability

---

## Slide 14-16: Container-Native Frontend Delivery (8 mins)

**Visual**: Container build and delivery pipeline diagram

**Talking Points**:

### Why Package Frontend as Containers?

**Benefits**:
- **Standardization**: Same tooling as backend (Podman, Docker, Buildah)
- **Distribution**: Leverage existing container registries
- **Security**: Vulnerability scanning, signing, SBOM
- **Versioning**: Semantic tagging, immutable artifacts
- **Orchestration**: Kubernetes-native delivery

### Plugin Container Structure

**1. Building the Plugin Container**
```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Package
FROM scratch
COPY --from=builder /app/dist /dist
COPY metadata.json /
LABEL com.redhat.rhdh.plugin.id="my-plugin"
LABEL com.redhat.rhdh.plugin.version="1.2.3"
```

**Key Points**:
- Multi-stage build keeps image small
- `FROM scratch` - no OS, just static files
- Labels for plugin metadata
- Final image: 5-10MB typical

**2. Plugin Metadata**
```json
{
  "name": "my-plugin",
  "version": "1.2.3",
  "backstage": {
    "role": "frontend-plugin"
  },
  "exports": {
    "remoteEntry": "./dist/remoteEntry.js",
    "assets": ["./dist/static/"]
  }
}
```

**3. Publishing to Registry**
```bash
# Build and tag
podman build -t quay.io/rhdh-plugins/my-plugin:1.2.3 .

# Push to registry
podman push quay.io/rhdh-plugins/my-plugin:1.2.3

# Sign image (optional but recommended)
cosign sign quay.io/rhdh-plugins/my-plugin:1.2.3
```

### Platform Plugin Discovery

**4. Configuration**
```yaml
# dynamic-plugins.yaml
plugins:
  - package: quay.io/rhdh-plugins/catalog-plugin:1.2.3
    disabled: false
  - package: quay.io/rhdh-plugins/kubernetes-plugin:2.0.1
    disabled: false
  - package: quay.io/rhdh-plugins/techdocs-plugin:1.5.0
    disabled: true  # Can enable/disable without rebuild
```

**5. Init Container Pattern**
```yaml
# Kubernetes deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rhdh-platform
spec:
  template:
    spec:
      initContainers:
      # Download plugins before app starts
      - name: install-dynamic-plugins
        image: quay.io/rhdh/dynamic-plugin-installer:latest
        volumeMounts:
        - name: dynamic-plugins
          mountPath: /dynamic-plugins
        env:
        - name: PLUGINS_CONFIG
          valueFrom:
            configMapKeyRef:
              name: dynamic-plugins-config
              key: plugins.yaml
      
      containers:
      - name: rhdh-platform
        image: quay.io/rhdh/platform:latest
        volumeMounts:
        - name: dynamic-plugins
          mountPath: /opt/app-root/src/dynamic-plugins
```

**Init Container Process**:
1. Reads plugin configuration
2. Pulls plugin containers from registry
3. Extracts static files to shared volume
4. Platform container mounts volume
5. Plugin loader reads and imports modules

**Benefits**:
- Plugins available before app starts
- No runtime network dependencies for plugin code
- Atomic updates via Kubernetes rolling deployments

---

## Slide 17-19: Plugin Lifecycle in Kubernetes (8 mins)

**Visual**: State diagram showing plugin lifecycle stages

**Talking Points**:

### Complete Plugin Lifecycle

**1. Development**
```bash
# Create plugin from template
npx @backstage/create-app create-plugin

# Develop locally with hot reload
yarn start

# Build for production
yarn build

# Package as container
yarn export-dynamic-plugin
```

**2. Publishing**
```bash
# Automated CI/CD pipeline
- Build plugin container
- Run tests (unit, integration)
- Security scan (Snyk, Trivy)
- Generate SBOM
- Push to registry
- Update plugin catalog
```

**3. Discovery & Configuration**

**GitOps Approach**:
```yaml
# Platform team maintains config repo
apiVersion: v1
kind: ConfigMap
metadata:
  name: dynamic-plugins-config
data:
  plugins.yaml: |
    plugins:
      - package: quay.io/rhdh-plugins/catalog:v1.2.3
        pluginConfig:
          catalog:
            providers:
              github:
                organization: my-org
```

**4. Runtime Loading**
```typescript
// Platform startup sequence
class DynamicPluginLoader {
  async loadPlugins() {
    // 1. Read configuration
    const config = await this.readPluginConfig();
    
    // 2. Validate plugins
    for (const plugin of config.plugins) {
      await this.validatePlugin(plugin);
    }
    
    // 3. Load modules
    for (const plugin of config.plugins) {
      if (!plugin.disabled) {
        await this.loadPlugin(plugin);
      }
    }
  }
  
  async loadPlugin(pluginConfig) {
    const { package: pluginPath } = pluginConfig;
    
    // Load remote module
    const module = await import(
      `/dynamic-plugins/${pluginPath}/remoteEntry.js`
    );
    
    // Initialize plugin
    await module.init(this.sharedScope);
    
    // Get plugin factory
    const pluginFactory = await module.get('./plugin');
    
    // Register with platform
    this.registerPlugin(pluginFactory());
  }
}
```

**5. Hot Swapping (Advanced)**

**Without Downtime**:
```typescript
// Watch for configuration changes
watchPluginConfig(async (newConfig) => {
  const diff = calculateDiff(currentPlugins, newConfig);
  
  // Unload removed plugins
  for (const plugin of diff.removed) {
    await unloadPlugin(plugin);
  }
  
  // Load new plugins
  for (const plugin of diff.added) {
    await loadPlugin(plugin);
  }
  
  // Reload changed plugins
  for (const plugin of diff.updated) {
    await reloadPlugin(plugin);
  }
});
```

**Reality Check**: Hot swapping is complex and has limitations:
- Stateful plugins may not reload cleanly
- Shared dependencies can't be hot-swapped
- Currently, most production deployments use rolling updates instead

**6. Monitoring & Observability**

**Plugin Metrics**:
```typescript
// Track plugin health
metrics.gauge('plugin.load_time', loadTimeMs, {
  plugin: pluginId,
  version: pluginVersion,
});

metrics.increment('plugin.error', {
  plugin: pluginId,
  error_type: errorType,
});
```

**Logging**:
```json
{
  "level": "info",
  "msg": "Plugin loaded successfully",
  "plugin": "catalog-plugin",
  "version": "1.2.3",
  "loadTime": "234ms"
}
```

---

## Slide 20-22: Production Lessons & Trade-offs (5 mins)

**Visual**: Lessons learned infographic with checkmarks and warning signs

**Talking Points**:

### What Works Well ✅

**1. True Team Independence**
- Plugin teams deploy daily without platform team involvement
- Reduced coordination overhead by 80%
- Faster time-to-production: Days → Hours

**2. Selective Updates**
- Security fix in one plugin? Update only that plugin
- No "deploy all or nothing" forced upgrades
- Reduced blast radius of changes

**3. Resource Optimization**
- User's browser caches core platform
- Only downloads plugins they use
- Lazy loading reduces initial load time

**4. Multi-tenancy Flexibility**
- Different customers get different plugin sets
- Same platform, customized experience
- No custom builds per customer

### Challenges & Trade-offs ⚠️

**1. Complexity**

**Before (Monolith)**:
```bash
yarn build   # One command
yarn deploy  # One deployment
```

**After (Dynamic)**:
```bash
# Platform team
deploy platform → manage plugin registry → maintain loader

# Plugin teams  
build plugin → publish to registry → update config → validate compatibility
```

**Mitigation**: Invest heavily in tooling and automation

**2. Dependency Management**

**The Nightmare Scenario**:
```
Platform: React 18.2, Library X 2.0
Plugin A: Works with React 18.2, Library X 2.0 ✅
Plugin B: Needs React 18.2, Library X 1.5 ❌ Conflict!
```

**Solutions**:
- Strict peer dependency requirements
- Automated compatibility checking in CI
- Deprecation schedules for breaking changes
- Version compatibility matrix

**3. Debugging is Harder**

**Challenges**:
- Stack traces span multiple bundles
- Source maps from different builds
- Harder to reproduce locally

**Solutions**:
```typescript
// Enhanced error context
window.addEventListener('error', (event) => {
  enhancedError({
    message: event.message,
    plugin: getCurrentPlugin(),
    pluginVersion: getPluginVersion(),
    platformVersion: getPlatformVersion(),
    sourceMap: event.filename,
  });
});
```

**4. Testing Complexity**

**Integration Testing**:
```typescript
// Must test plugin with actual platform
describe('Plugin Integration', () => {
  it('loads in production platform', async () => {
    const platform = await loadPlatform();
    const plugin = await loadPlugin('my-plugin:1.2.3');
    
    await platform.installPlugin(plugin);
    expect(platform.getPlugin('my-plugin')).toBeDefined();
  });
});
```

**5. Performance Overhead**

**Runtime Costs**:
- Initial module federation setup: ~200ms
- Per-plugin load: ~50-100ms
- Memory overhead: ~10-20MB per plugin

**When It's Worth It**:
- ✅ Large applications (10+ plugins)
- ✅ Multiple teams
- ✅ Frequent deployments
- ❌ Small applications (3-5 plugins)
- ❌ Single team
- ❌ Infrequent updates

---

## Slide 23: Architecture Decision Framework (3 mins)

**Visual**: Decision tree flowchart

**Talking Points**:

### Should YOU Use Dynamic Plugins?

**Consider Dynamic Plugins When**:
- Multiple teams contributing to one frontend
- Need independent deployment cycles
- Plugin marketplace or ecosystem planned
- Large monolith causing deployment bottlenecks
- Multi-tenant with varying feature sets

**Stick with Monolith When**:
- Small team (< 5 developers)
- Tightly coupled features
- Deploy everything together anyway
- Complexity overhead not justified
- Performance is critical (every millisecond counts)

**Hybrid Approach**:
- Core features in monolith
- Extensions as dynamic plugins
- Best of both worlds

**Quote**: "Dynamic plugins are powerful, but they're not free. Make sure you're solving a real problem, not just using cool technology."

---

## Slide 24-25: Future Directions (2 mins)

**Visual**: Roadmap timeline

**Talking Points**:

### What's Next for Dynamic Frontends?

**1. Native ES Modules**
- Browser-native module loading
- No bundler required
- Import maps for dependency management

```html
<script type="importmap">
{
  "imports": {
    "react": "/node_modules/react/index.js",
    "catalog-plugin": "/plugins/catalog/plugin.js"
  }
}
</script>
```

**2. WebAssembly Plugins**
- Language-agnostic plugins
- Write plugins in Rust, Go, etc.
- Performance benefits

**3. Edge Computing Integration**
- Serve plugins from CDN edge
- Geo-distributed plugin delivery
- Reduced latency

**4. AI-Assisted Plugin Composition**
- Automatically suggest plugin combinations
- Detect conflicts before deployment
- Optimize bundle loading order

---

## Slide 26: Key Takeaways (2 mins)

**Visual**: Summary slide with numbered key points

**Key Takeaways**:

1. **Frontend can be as dynamic as backend** - Module federation + containers enable true runtime composition

2. **Kubernetes-native delivery is possible** - Use the same infrastructure and patterns for frontend as you do for backend

3. **Independence has a cost** - Complexity increases, but scales better for large teams

4. **Tooling is critical** - Without good automation, dynamic plugins become a burden

5. **Know when to use it** - Powerful pattern, but not appropriate for every application

**Final Thought**: "We're just beginning to explore what's possible when frontend architecture embraces cloud-native principles. The monolithic frontend is dead—long live dynamic plugins!"

---

## Slide 27: Resources & Further Reading (1 min)

**Visual**: QR codes and links

**Resources**:
- Red Hat Developer Hub: https://developers.redhat.com/rhdh
- Webpack Module Federation: https://webpack.js.org/concepts/module-federation/
- Backstage Dynamic Plugins: https://backstage.io/docs/plugins/
- RHDH GitHub: https://github.com/redhat-developer/rhdh
- Container Registries: Quay.io, Docker Hub, GitHub Container Registry

**Sample Code**:
- Example plugin repository: [link]
- Module federation starter: [link]
- CI/CD pipeline templates: [link]

**Connect**:
- GitHub: [your handle]
- LinkedIn: [your profile]
- Email: [your email]

---

## Slide 28: Q&A (5 mins)

**Visual**: Thank you slide with contact info

**Anticipated Questions & Answers**:

**Q: How do you handle authentication in dynamic plugins?**
A: Platform provides authentication context through shared API. Plugins receive auth tokens via React context, never handle auth directly.

**Q: What about offline support?**
A: Service workers can cache plugin bundles. Challenge is versioning—need strategies for cache invalidation.

**Q: Performance impact on mobile?**
A: Initial load is heavier. We mitigate with aggressive code splitting and only loading plugins needed for current route.

**Q: Can plugins communicate with each other?**
A: Yes, through platform-provided event bus and shared state management. Direct dependencies are discouraged.

**Q: How do you prevent malicious plugins?**
A: Container image signing, security scanning, allowlist of approved registries, runtime sandboxing (future work).

**Q: What if a plugin crashes?**
A: Error boundaries isolate plugin failures. Platform stays functional, user sees error state for that plugin only.

**Q: Migration strategy from monolith?**
A: Incremental. Start with new features as plugins. Gradually extract existing features. Run both in parallel during transition.

**Q: Licensing concerns with dynamic loading?**
A: Complex topic. SBOM generation helps. Consult legal team for your specific situation.

---

## Demo Flow (Integrated throughout)

**Demo 1** (Slide 8): Show RHDH platform starting up, logs showing plugin discovery and loading

**Demo 2** (Slide 13): Browser DevTools showing module federation in action, shared dependencies

**Demo 3** (Slide 16): Show container registry with plugin images, pull and inspect one

**Demo 4** (Slide 19): Edit dynamic-plugins.yaml in Git, push, show GitOps deploying new plugin

**Demo 5** (If time): Show performance comparison with/without module federation using Chrome DevTools

---

## Preparation Checklist

### Technical Setup
- [ ] RHDH instance with admin access
- [ ] kubectl access to show Kubernetes resources
- [ ] Container registry with plugin images
- [ ] Browser DevTools prepared with relevant tabs
- [ ] Git repository with dynamic-plugins config
- [ ] Performance profiling tools ready
- [ ] Backup screenshots/videos for all demos

### Code Examples
- [ ] Webpack configs syntax-highlighted and tested
- [ ] YAML configs validated
- [ ] Dockerfile builds successfully
- [ ] TypeScript examples compile

### Presentation Materials
- [ ] Slides exported to PDF
- [ ] All code examples tested and working
- [ ] Diagrams clear and readable
- [ ] Timing rehearsed multiple times
- [ ] Transitions smooth between demos and slides

### Speaking Notes
- [ ] Module federation concepts reviewed
- [ ] Kubernetes terminology prepared
- [ ] Performance metrics memorized
- [ ] Trade-offs clearly understood

---

## Additional Notes

### Target Audience Assumptions
- Familiar with React and modern JavaScript
- Understand containers and Kubernetes basics
- May not know Webpack Module Federation
- Interested in platform engineering and scalability

### Energy & Pacing
- Start with relatable pain point (monolithic frontend)
- Build excitement with vision of dynamic plugins
- Deep dive into technical implementation
- Balance complexity with practical examples
- End with honest assessment of trade-offs

### Adaptation Points
- If running short: Condense slides 20-22 (Production Lessons)
- If time available: Deep dive into webpack config in slide 11-13
- If more technical audience: Add more code examples, less conceptual
- If mixed audience: Focus on architecture diagrams, less code

### Story Arc
1. **Problem**: Monolithic frontend doesn't scale
2. **Vision**: What if frontend worked like backend?
3. **Solution**: Module federation + containers + Kubernetes
4. **Reality**: Here's how it actually works
5. **Honesty**: Here are the real trade-offs
6. **Guidance**: Should you do this?

---

**Good luck with your presentation! 🚀**

