# 🚦 Lighthouse Optimization - Auditoría Completa de Performance

Guía específica para optimizar todas las métricas de Lighthouse en aplicaciones ecommerce Angular, con enfoque en automatización y CI/CD.

## 🎯 Métricas de Lighthouse

### Core Web Vitals

- **LCP (Largest Contentful Paint)**: < 2.5s
- **FCP (First Contentful Paint)**: < 1.8s
- **CLS (Cumulative Layout Shift)**: < 0.1
- **FID (First Input Delay)**: < 100ms

### Performance Score

- **Performance**: > 90
- **Accessibility**: > 95
- **Best Practices**: > 90
- **SEO**: > 95
- **PWA**: > 90

## 📊 Lighthouse CI Integration

### GitHub Actions Setup

```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build:prod

      - name: Serve application
        run: |
          npm install -g http-server
          http-server dist/frontend -p 8080 &
          sleep 5

      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v9
        with:
          configPath: "./lighthouserc.json"
          uploadArtifacts: true
          temporaryPublicStorage: true
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}

      - name: Performance Budget Check
        run: npm run lighthouse:budget-check

      - name: Comment PR with Results
        uses: actions/github-script@v6
        if: github.event_name == 'pull_request'
        with:
          script: |
            const fs = require('fs');
            const results = JSON.parse(fs.readFileSync('.lighthouseci/lhr-*.json'));
            const score = Math.round(results.categories.performance.score * 100);

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `🚦 Lighthouse Results: Performance Score **${score}/100**`
            });
```

### Lighthouse Configuration

```javascript
// lighthouserc.json
{
  "ci": {
    "collect": {
      "numberOfRuns": 3,
      "startServerCommand": "npm run serve:ssr",
      "url": [
        "http://localhost:4000",
        "http://localhost:4000/productos",
        "http://localhost:4000/producto/ejemplo",
        "http://localhost:4000/carrito",
        "http://localhost:4000/checkout"
      ],
      "settings": {
        "preset": "desktop",
        "chromeFlags": "--no-sandbox --disable-dev-shm-usage",
        "extraHeaders": {
          "Cookie": "test-mode=true"
        }
      }
    },
    "assert": {
      "assertions": {
        "categories:performance": ["error", {"minScore": 0.9}],
        "categories:accessibility": ["error", {"minScore": 0.95}],
        "categories:best-practices": ["error", {"minScore": 0.9}],
        "categories:seo": ["error", {"minScore": 0.95}],
        "categories:pwa": ["warn", {"minScore": 0.8}]
      }
    },
    "upload": {
      "target": "lhci",
      "serverBaseUrl": "https://lighthouse-server.company.com"
    }
  }
}
```

## 🔧 Optimizaciones Específicas

### 1. Performance Optimizations

```typescript
// Performance optimizations based on Lighthouse audits
export class LighthouseOptimizationService {
  // Eliminate render-blocking resources
  optimizeRenderBlocking() {
    // Inline critical CSS
    this.inlineCriticalCSS();

    // Defer non-critical JavaScript
    this.deferNonCriticalJS();

    // Optimize fonts loading
    this.optimizeFonts();
  }

  private inlineCriticalCSS() {
    // Extract above-the-fold CSS and inline it
    const criticalCSS = this.extractCriticalCSS();
    this.inlineCSS(criticalCSS);
  }

  private deferNonCriticalJS() {
    // Lazy load non-critical components
    const nonCriticalModules = ["analytics", "chat-widget", "marketing-popup"];

    nonCriticalModules.forEach((module) => {
      this.lazyLoadModule(module);
    });
  }

  private optimizeFonts() {
    // Use font-display: swap
    // Preload critical fonts
    // Use system fonts as fallback
  }
}
```

### 2. Image Optimization

```typescript
// Optimize images for better LCP scores
export class ImageOptimizationService {
  optimizeImages() {
    // Use next-gen formats
    this.implementWebP();

    // Properly size images
    this.implementResponsiveImages();

    // Lazy load off-screen images
    this.implementLazyLoading();

    // Preload LCP image
    this.preloadLCPImage();
  }

  private implementWebP() {
    // Convert images to WebP format
    // Implement fallback for unsupported browsers
  }

  private preloadLCPImage() {
    // Identify and preload the largest contentful paint image
    const lcpImage = this.identifyLCPImage();
    if (lcpImage) {
      this.preloadResource(lcpImage);
    }
  }
}
```

### 3. Bundle Optimization

```typescript
// Webpack optimizations for better performance scores
// webpack.config.js optimizations
module.exports = {
  optimization: {
    splitChunks: {
      chunks: "all",
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: "vendors",
          chunks: "all",
        },
        common: {
          minChunks: 2,
          chunks: "all",
          enforce: true,
        },
      },
    },
    usedExports: true,
    providedExports: true,
    sideEffects: false,
  },

  plugins: [
    // Tree shaking and dead code elimination
    new webpack.optimize.ModuleConcatenationPlugin(),

    // Compression
    new CompressionPlugin({
      algorithm: "gzip",
      test: /\.(js|css|html|svg)$/,
      threshold: 8192,
      minRatio: 0.8,
    }),
  ],
};
```

## 📱 PWA Optimization

### Service Worker Implementation

```typescript
// Service Worker for PWA score improvement
// sw.js
self.addEventListener("install", (event) => {
  event.waitUntil(
    caches.open("ecommerce-v1").then((cache) => {
      return cache.addAll([
        "/",
        "/productos",
        "/manifest.json",
        "/assets/icons/icon-192x192.png",
        "/assets/icons/icon-512x512.png",
      ]);
    })
  );
});

self.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request);
    })
  );
});
```

### Web App Manifest

```json
{
  "name": "Ecommerce App",
  "short_name": "Ecommerce",
  "description": "Modern ecommerce application",
  "start_url": "/",
  "display": "standalone",
  "theme_color": "#667eea",
  "background_color": "#ffffff",
  "icons": [
    {
      "src": "/assets/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/assets/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable any"
    }
  ]
}
```

## 🔍 Accessibility Improvements

```typescript
// Accessibility optimizations for better a11y scores
export class AccessibilityService {
  improveAccessibility() {
    // Add proper ARIA labels
    this.addAriaLabels();

    // Improve color contrast
    this.validateColorContrast();

    // Add focus management
    this.implementFocusManagement();

    // Add semantic HTML
    this.implementSemanticHTML();
  }

  private addAriaLabels() {
    // Add ARIA labels to interactive elements
    const buttons = document.querySelectorAll("button");
    buttons.forEach((button) => {
      if (!button.getAttribute("aria-label")) {
        button.setAttribute("aria-label", this.generateAriaLabel(button));
      }
    });
  }

  private implementFocusManagement() {
    // Implement proper focus indicators
    // Skip links for navigation
    // Focus trapping in modals
  }
}
```

## 📊 Performance Monitoring Dashboard

```typescript
// Real-time Lighthouse scores monitoring
export class LighthouseMonitoringService {
  private readonly lighthouseEndpoint = "/api/lighthouse";

  async runLighthouseAudit(url: string) {
    const lighthouse = await import("lighthouse");
    const chromeLauncher = await import("chrome-launcher");

    const chrome = await chromeLauncher.launch({ chromeFlags: ["--headless"] });
    const options = {
      logLevel: "info",
      output: "json",
      onlyCategories: [
        "performance",
        "accessibility",
        "best-practices",
        "seo",
        "pwa",
      ],
      port: chrome.port,
    };

    const runnerResult = await lighthouse(url, options);
    await chrome.kill();

    return this.parseResults(runnerResult.lhr);
  }

  private parseResults(results: any) {
    return {
      performance: Math.round(results.categories.performance.score * 100),
      accessibility: Math.round(results.categories.accessibility.score * 100),
      bestPractices: Math.round(
        results.categories["best-practices"].score * 100
      ),
      seo: Math.round(results.categories.seo.score * 100),
      pwa: Math.round(results.categories.pwa.score * 100),
      metrics: {
        fcp: results.audits["first-contentful-paint"].numericValue,
        lcp: results.audits["largest-contentful-paint"].numericValue,
        cls: results.audits["cumulative-layout-shift"].numericValue,
        fid: results.audits["max-potential-fid"].numericValue,
      },
    };
  }
}
```

## 🎯 Performance Budget

```javascript
// performance-budget.json
{
  "budgets": [
    {
      "path": "/*",
      "timings": [
        {
          "metric": "first-contentful-paint",
          "budget": 1800
        },
        {
          "metric": "largest-contentful-paint",
          "budget": 2500
        },
        {
          "metric": "cumulative-layout-shift",
          "budget": 0.1
        }
      ],
      "resourceSizes": [
        {
          "resourceType": "script",
          "budget": 400
        },
        {
          "resourceType": "stylesheet",
          "budget": 100
        },
        {
          "resourceType": "image",
          "budget": 1000
        }
      ]
    }
  ]
}
```

## 🚀 Checklist de Optimización

### ✅ Performance (Target: 90+)

- [ ] Optimize images (WebP, lazy loading, proper sizing)
- [ ] Eliminate render-blocking resources
- [ ] Minimize main thread work
- [ ] Reduce JavaScript execution time
- [ ] Serve images in next-gen formats
- [ ] Preload key requests
- [ ] Use efficient cache policy

### ✅ Accessibility (Target: 95+)

- [ ] Proper color contrast ratios
- [ ] ARIA labels on interactive elements
- [ ] Semantic HTML structure
- [ ] Focus indicators and management
- [ ] Alt text for images
- [ ] Keyboard navigation support

### ✅ Best Practices (Target: 90+)

- [ ] HTTPS implementation
- [ ] No browser errors in console
- [ ] Proper aspect ratios for images
- [ ] Avoid deprecated APIs
- [ ] CSP (Content Security Policy)

### ✅ SEO (Target: 95+)

- [ ] Meta descriptions
- [ ] Valid HTML structure
- [ ] Crawlable links
- [ ] Proper heading hierarchy
- [ ] Structured data
- [ ] Mobile-friendly design

### ✅ PWA (Target: 90+)

- [ ] Web app manifest
- [ ] Service worker implementation
- [ ] Offline functionality
- [ ] Installable app
- [ ] Proper icons and theme colors

---

**🎯 Próximo paso**: Ejecuta `npm run lighthouse:audit` para obtener un baseline y comienza las optimizaciones prioritarias.
