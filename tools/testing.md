# 🧪 Testing de Performance

Estrategias completas de testing para aplicaciones e-commerce Angular, incluyendo tests de rendimiento, visuales, iconos y Core Web Vitals.

## 🎯 Tipos de Testing

### 1. Tests de Iconos y SVG

```typescript
// libs/icons/src/lib/components/icon.component.spec.ts
import { ComponentFixture, TestBed } from "@angular/core/testing";
import { IconComponent } from "./icon.component";
import { ICON_CONFIG } from "../config/icon.config";

describe("IconComponent", () => {
  let component: IconComponent;
  let fixture: ComponentFixture<IconComponent>;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [IconComponent],
      providers: [
        {
          provide: ICON_CONFIG,
          useValue: {
            cdnUrl: "https://test.cdn.com/icons",
            fallbackEnabled: true,
          },
        },
      ],
    });

    fixture = TestBed.createComponent(IconComponent);
    component = fixture.componentInstance;
  });

  it("should create", () => {
    expect(component).toBeTruthy();
  });

  it("should generate correct sprite URL", () => {
    component.name = "user";
    component.section = "core";

    expect(component.spriteUrl).toBe("https://test.cdn.com/icons/sprite-core");
  });

  it("should apply custom classes correctly", () => {
    component.name = "user";
    component.className = "custom-class";
    component.clickable = true;

    expect(component.iconClass).toBe(
      "icon icon-user custom-class icon-clickable"
    );
  });

  it("should handle missing icons gracefully", () => {
    const consoleSpy = spyOn(console, "error");
    component.name = "non-existent" as any;

    const event = new Event("error");
    component.onError(event);

    expect(consoleSpy).toHaveBeenCalledWith(
      "Failed to load icon: non-existent from section: core"
    );
  });

  it("should preload sprite only once per section", () => {
    const createElementSpy = spyOn(document, "createElement").and.callThrough();

    // First component
    component.section = "core";
    component.ngOnInit();

    // Second component same section
    const component2 = TestBed.createComponent(IconComponent).componentInstance;
    component2.section = "core";
    component2.ngOnInit();

    // Should only create one link element for preloading
    expect(createElementSpy).toHaveBeenCalledTimes(1);
  });
});
```

### 2. Tests de Performance de Iconos

```typescript
// libs/icons/src/lib/services/icon-performance.service.spec.ts
import { TestBed } from "@angular/core/testing";
import { IconPerformanceService } from "./icon-performance.service";

describe("IconPerformanceService", () => {
  let service: IconPerformanceService;

  beforeEach(() => {
    TestBed.configureTestingModule({});
    service = TestBed.inject(IconPerformanceService);
  });

  it("should track icon load times", () => {
    const startTime = performance.now() - 100; // Simulate 100ms load

    service.trackIconLoad("core", startTime);

    expect(service.getAverageLoadTime()).toBeGreaterThan(90);
    expect(service.getAverageLoadTime()).toBeLessThan(110);
  });

  it("should return 0 for average when no loads tracked", () => {
    expect(service.getAverageLoadTime()).toBe(0);
  });

  it("should calculate correct average for multiple loads", () => {
    service.trackIconLoad("core", performance.now() - 100);
    service.trackIconLoad("home", performance.now() - 200);

    const average = service.getAverageLoadTime();
    expect(average).toBeGreaterThan(140);
    expect(average).toBeLessThan(160);
  });
});
```

### 3. Tests E2E con Playwright

```typescript
// tests/e2e/icon-performance.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Icon Performance Tests", () => {
  test("critical icons should be inline in HTML", async ({ page }) => {
    await page.goto("/");

    // Check that critical icons are inline SVGs
    const logoSvg = page.locator('header svg[viewBox="0 0 120 40"]');
    await expect(logoSvg).toBeVisible();

    const cartSvg = page.locator('.cart-btn svg[viewBox="0 0 24 24"]');
    await expect(cartSvg).toBeVisible();

    const searchSvg = page.locator('.search-icon svg[viewBox="0 0 24 24"]');
    await expect(searchSvg).toBeVisible();
  });

  test("sprite icons should load from CDN", async ({ page }) => {
    await page.goto("/");

    // Scroll to footer to trigger sprite loading
    await page.locator("footer").scrollIntoViewIfNeeded();

    // Wait for sprite to load
    await page.waitForFunction(() => {
      const spriteUse = document.querySelector("footer ui-icon svg use");
      return spriteUse && spriteUse.getAttribute("href")?.includes("sprite-");
    });

    const spriteIcons = page.locator("footer ui-icon svg use");
    const count = await spriteIcons.count();
    expect(count).toBeGreaterThan(0);
  });

  test("icon loading should not block FCP", async ({ page }) => {
    const startTime = Date.now();

    await page.goto("/");

    // Check that FCP includes critical icons
    const firstPaint = await page.evaluate(() => {
      return new Promise((resolve) => {
        new PerformanceObserver((list) => {
          for (const entry of list.getEntries()) {
            if (entry.name === "first-contentful-paint") {
              resolve(entry.startTime);
            }
          }
        }).observe({ entryTypes: ["paint"] });
      });
    });

    expect(firstPaint).toBeLessThan(1500); // FCP should be under 1.5s
  });

  test("sprite preloading should work correctly", async ({ page }) => {
    await page.goto("/");

    // Check that sprites are preloaded
    const preloadLinks = await page
      .locator('link[rel="preload"][as="image"]')
      .count();
    expect(preloadLinks).toBeGreaterThan(0);

    // Check that sprite URLs are correct
    const spritePreload = page.locator('link[href*="sprite-core"]');
    await expect(spritePreload).toHaveAttribute("type", "image/svg+xml");
  });
});
```

### 4. Tests Visuales con Percy/Chromatic

```typescript
// tests/visual/icon-regression.spec.ts
import { test } from "@playwright/test";
import { percySnapshot } from "@percy/playwright";

test.describe("Icon Visual Regression Tests", () => {
  test("header icons should render consistently", async ({ page }) => {
    await page.goto("/");

    // Ensure all critical icons are loaded
    await page.waitForSelector('header svg[viewBox="0 0 120 40"]');
    await page.waitForSelector(".cart-btn svg");
    await page.waitForSelector(".search-icon svg");

    await percySnapshot(page, "Header Icons");
  });

  test("footer sprite icons should render consistently", async ({ page }) => {
    await page.goto("/");

    // Scroll to footer and wait for sprites to load
    await page.locator("footer").scrollIntoViewIfNeeded();
    await page.waitForTimeout(1000); // Allow sprites to load

    await percySnapshot(page, "Footer Sprite Icons");
  });

  test("product grid icons should render consistently", async ({ page }) => {
    await page.goto("/productos");

    // Wait for product grid to load
    await page.waitForSelector(".product-grid .product-card");
    await page.waitForTimeout(500);

    await percySnapshot(page, "Product Grid Icons");
  });

  test("icon preview page should render all icons", async ({ page }) => {
    await page.goto("/icon-preview");

    // Wait for all sections to load
    await page.waitForSelector('.section[data-section="core"]');
    await page.waitForSelector('.section[data-section="home"]');

    await percySnapshot(page, "Icon Preview - All Sections");
  });
});
```

## 🚀 Performance Testing

### 1. Lighthouse CI Testing

```javascript
// .lighthouserc.js
module.exports = {
  ci: {
    collect: {
      url: [
        "http://localhost:4200/",
        "http://localhost:4200/productos",
        "http://localhost:4200/producto/1",
      ],
      settings: {
        preset: "desktop",
        chromeFlags: "--no-sandbox --disable-dev-shm-usage",
        skipAudits: ["uses-http2"],
        onlyCategories: [
          "performance",
          "accessibility",
          "best-practices",
          "seo",
        ],
      },
      numberOfRuns: 3,
    },
    assert: {
      assertions: {
        "categories:performance": ["error", { minScore: 0.95 }],
        "categories:accessibility": ["error", { minScore: 0.9 }],
        "categories:best-practices": ["error", { minScore: 0.9 }],
        "categories:seo": ["error", { minScore: 0.9 }],
        "first-contentful-paint": ["error", { maxNumericValue: 1000 }],
        "largest-contentful-paint": ["error", { maxNumericValue: 2500 }],
        "cumulative-layout-shift": ["error", { maxNumericValue: 0.1 }],
        "total-blocking-time": ["error", { maxNumericValue: 300 }],
        // Icon-specific audits
        "unused-css-rules": ["warn", { maxLength: 5 }],
        "render-blocking-resources": ["error", { maxLength: 2 }],
      },
    },
    upload: {
      target: "lhci",
      serverBaseUrl: process.env.LHCI_SERVER_BASE_URL,
      token: process.env.LHCI_BUILD_TOKEN,
    },
  },
};
```

### 2. Custom Performance Tests

```typescript
// tests/performance/icon-metrics.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Icon Performance Metrics", () => {
  test("should measure icon sprite load times", async ({ page }) => {
    const spriteLoadTimes: Record<string, number> = {};

    // Listen for sprite requests
    page.on("response", (response) => {
      const url = response.url();
      if (url.includes("sprite-") && url.endsWith(".svg")) {
        const timing = response.timing();
        spriteLoadTimes[url] = timing.responseEnd - timing.requestStart;
      }
    });

    await page.goto("/");
    await page.locator("footer").scrollIntoViewIfNeeded();
    await page.waitForTimeout(2000);

    // Verify sprite load times are acceptable
    Object.entries(spriteLoadTimes).forEach(([url, time]) => {
      console.log(`Sprite ${url}: ${time}ms`);
      expect(time).toBeLessThan(500); // Sprites should load in under 500ms
    });
  });

  test("should measure critical icon rendering time", async ({ page }) => {
    const startTime = Date.now();

    await page.goto("/");

    // Wait for critical icons to be visible
    await page.waitForSelector('header svg[viewBox="0 0 120 40"]', {
      state: "visible",
    });
    await page.waitForSelector(".cart-btn svg", { state: "visible" });

    const renderTime = Date.now() - startTime;
    console.log(`Critical icons rendered in: ${renderTime}ms`);

    expect(renderTime).toBeLessThan(800); // Critical icons should render quickly
  });

  test("should verify no layout shift from icons", async ({ page }) => {
    await page.goto("/");

    // Measure CLS
    const cls = await page.evaluate(() => {
      return new Promise<number>((resolve) => {
        import("web-vitals").then(({ getCLS }) => {
          let clsValue = 0;

          getCLS((metric) => (clsValue += metric.value));

          setTimeout(() => resolve(clsValue), 3000);
        });
      });
    });

    console.log(`Cumulative Layout Shift: ${cls}`);
    expect(cls).toBeLessThan(0.1); // CLS should be minimal
  });
});
```

### 3. Bundle Size Testing

```javascript
// tests/bundle/icon-bundle-size.spec.mjs
import { statSync } from "fs";
import { glob } from "glob";

describe("Icon Bundle Size Tests", () => {
  test("main bundle should not include sprite icons", async () => {
    const mainBundle = glob.sync("dist/apps/ecommerce-angular/main.*.js")[0];
    const bundleContent = readFileSync(mainBundle, "utf-8");

    // Check that sprite icon names are not in main bundle
    const spriteIconPattern = /#icon-[a-z-]+/g;
    const matches = bundleContent.match(spriteIconPattern);

    expect(matches).toBeNull(); // No sprite references in main bundle
  });

  test("sprite files should be under size limits", async () => {
    const sprites = glob.sync("dist/sprites/*.svg");

    sprites.forEach((spritePath) => {
      const stats = statSync(spritePath);
      const sizeKB = stats.size / 1024;

      console.log(`${spritePath}: ${sizeKB.toFixed(1)}KB`);
      expect(sizeKB).toBeLessThan(100); // Sprites should be under 100KB
    });
  });

  test("total icon assets should be under budget", async () => {
    const allSprites = glob.sync("dist/sprites/*.svg");
    const totalSize = allSprites.reduce((total, file) => {
      return total + statSync(file).size;
    }, 0);

    const totalSizeKB = totalSize / 1024;
    console.log(`Total sprite size: ${totalSizeKB.toFixed(1)}KB`);

    expect(totalSizeKB).toBeLessThan(500); // Total under 500KB
  });
});
```

## 🔧 Testing Utilities

### 1. Icon Testing Helper

```typescript
// libs/icons/src/testing/icon-testing.module.ts
import { NgModule } from "@angular/core";
import { ICON_CONFIG } from "../lib/config/icon.config";

@NgModule({
  providers: [
    {
      provide: ICON_CONFIG,
      useValue: {
        cdnUrl: "http://localhost:4200/assets/sprites",
        fallbackEnabled: true,
        preloadStrategies: [],
      },
    },
  ],
})
export class IconTestingModule {}

// Mock sprite response for tests
export function mockSpriteResponse(section: string, icons: string[]) {
  const spriteContent = `
    <svg xmlns="http://www.w3.org/2000/svg">
      ${icons
        .map(
          (icon) => `
        <symbol id="icon-${icon}" viewBox="0 0 24 24">
          <circle cx="12" cy="12" r="10"/>
        </symbol>
      `
        )
        .join("")}
    </svg>
  `;

  return spriteContent;
}
```

### 2. Performance Testing Helper

```typescript
// libs/testing/src/lib/performance-helper.ts
export class PerformanceTestHelper {
  static async measureRenderTime(
    page: any,
    selector: string,
    description: string
  ): Promise<number> {
    const startTime = Date.now();

    await page.waitForSelector(selector, { state: "visible" });

    const renderTime = Date.now() - startTime;
    console.log(`${description}: ${renderTime}ms`);

    return renderTime;
  }

  static async getWebVitals(page: any) {
    return await page.evaluate(() => {
      return new Promise((resolve) => {
        import("web-vitals").then(
          ({ getCLS, getFCP, getFID, getLCP, getTTFB }) => {
            const vitals: any = {};

            getCLS((metric) => (vitals.cls = metric.value));
            getFCP((metric) => (vitals.fcp = metric.value));
            getFID((metric) => (vitals.fid = metric.value));
            getLCP((metric) => (vitals.lcp = metric.value));
            getTTFB((metric) => (vitals.ttfb = metric.value));

            setTimeout(() => resolve(vitals), 3000);
          }
        );
      });
    });
  }

  static async measureNetworkRequests(page: any, urlPattern: string) {
    const requests: any[] = [];

    page.on("request", (request: any) => {
      if (request.url().includes(urlPattern)) {
        requests.push({
          url: request.url(),
          method: request.method(),
          timestamp: Date.now(),
        });
      }
    });

    page.on("response", (response: any) => {
      if (response.url().includes(urlPattern)) {
        const request = requests.find((r) => r.url === response.url());
        if (request) {
          request.responseTime = Date.now() - request.timestamp;
          request.status = response.status();
          request.size = response.headers()["content-length"];
        }
      }
    });

    return requests;
  }
}
```

### 3. Icon Validation Helper

```typescript
// libs/icons/src/testing/icon-validator.ts
export class IconValidator {
  static validateSpriteStructure(spriteContent: string): ValidationResult {
    const errors: string[] = [];
    const warnings: string[] = [];

    // Check if it's valid SVG
    if (!spriteContent.includes("<svg")) {
      errors.push("Not a valid SVG file");
    }

    // Check for symbols
    const symbolMatches = spriteContent.match(/<symbol[^>]*id="icon-([^"]+)"/g);
    if (!symbolMatches || symbolMatches.length === 0) {
      errors.push("No icon symbols found");
    }

    // Check for proper IDs
    symbolMatches?.forEach((match) => {
      const idMatch = match.match(/id="icon-([^"]+)"/);
      if (idMatch) {
        const iconName = idMatch[1];
        if (!/^[a-z0-9-]+$/.test(iconName)) {
          errors.push(`Invalid icon name: ${iconName}`);
        }
      }
    });

    // Check file size
    const sizeKB = Buffer.byteLength(spriteContent, "utf8") / 1024;
    if (sizeKB > 100) {
      warnings.push(`Large sprite file: ${sizeKB.toFixed(1)}KB`);
    }

    return {
      isValid: errors.length === 0,
      errors,
      warnings,
      iconCount: symbolMatches?.length || 0,
      sizeKB,
    };
  }

  static validateCriticalIcon(svgContent: string, iconName: string): boolean {
    // Check if SVG is inline-ready
    if (!svgContent.includes("viewBox")) {
      console.warn(`Critical icon ${iconName} missing viewBox`);
      return false;
    }

    if (svgContent.includes("width=") || svgContent.includes("height=")) {
      console.warn(`Critical icon ${iconName} has fixed dimensions`);
      return false;
    }

    return true;
  }
}

interface ValidationResult {
  isValid: boolean;
  errors: string[];
  warnings: string[];
  iconCount: number;
  sizeKB: number;
}
```

## 📊 Continuous Testing

### 1. GitHub Actions Test Pipeline

```yaml
# .github/workflows/icon-tests.yml
name: Icon Tests

on:
  push:
    paths:
      - "libs/icons/**"
      - "apps/**/src/**"
  pull_request:
    paths:
      - "libs/icons/**"

jobs:
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run icon unit tests
        run: npm run test:icons -- --coverage --watch=false

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/libs/icons/lcov.info

  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright
        run: npx playwright install

      - name: Build application
        run: npm run build

      - name: Start application
        run: npm run serve &

      - name: Wait for application
        run: npx wait-on http://localhost:4200

      - name: Run E2E tests
        run: npx playwright test tests/e2e/icon-performance.spec.ts

      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/

  performance-tests:
    name: Performance Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build:production

      - name: Start application
        run: npm run serve:production &

      - name: Run Lighthouse CI
        run: npx lhci autorun
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}

      - name: Run custom performance tests
        run: npx playwright test tests/performance/
```

### 2. Test Scripts

```json
{
  "scripts": {
    "test:icons": "nx test icons",
    "test:icons:watch": "nx test icons --watch",
    "test:icons:coverage": "nx test icons --coverage",
    "test:e2e:icons": "playwright test tests/e2e/icon-performance.spec.ts",
    "test:performance": "playwright test tests/performance/",
    "test:visual": "playwright test tests/visual/",
    "test:bundle": "node tests/bundle/icon-bundle-size.spec.mjs",
    "test:all": "npm run test:icons && npm run test:e2e:icons && npm run test:performance"
  }
}
```

## 🎯 Testing Checklist

- [ ] **Unit tests para componente Icon**
- [ ] **Tests de carga de sprites**
- [ ] **Tests de iconos críticos inline**
- [ ] **Tests de performance (FCP, LCP, CLS)**
- [ ] **Tests visuales de regresión**
- [ ] **Tests de bundle size**
- [ ] **Tests E2E de funcionalidad**
- [ ] **Lighthouse CI configurado**
- [ ] **Tests de accesibilidad de iconos**
- [ ] **Pipeline de CI/CD con tests**

---

**Siguiente**: [Arquitectura NX](../architecture/nx-structure.md)
