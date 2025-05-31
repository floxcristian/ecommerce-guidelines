# ⚡ Optimización Lighthouse 100/100

Guía completa para conseguir puntuaciones perfectas en Lighthouse con enfoque específico en e-commerce y sistema de iconos.

## 🎯 Objetivos de Performance

| Métrica         | Objetivo | Crítico | Estrategia Principal                       |
| --------------- | -------- | ------- | ------------------------------------------ |
| **Performance** | 100/100  | 95+     | Iconos críticos inline + sprites CDN       |
| **FCP**         | < 1.0s   | < 1.5s  | SSR + iconos above-the-fold inline         |
| **LCP**         | < 2.5s   | < 4.0s  | Preload crítico + lazy loading inteligente |
| **CLS**         | < 0.1    | < 0.25  | Dimensiones fijas + skeleton loading       |
| **TTI**         | < 3.0s   | < 5.0s  | Code splitting + defer no crítico          |

## 🔍 Auditoría Inicial

### Script de Análisis Rápido

```javascript
// tools/quick-audit.mjs
import lighthouse from "lighthouse";
import chromeLauncher from "chrome-launcher";
import chalk from "chalk";

class QuickAudit {
  async run(url = "http://localhost:4200") {
    console.log(chalk.blue(`🔍 Running quick audit on ${url}...\n`));

    const chrome = await chromeLauncher.launch({ chromeFlags: ["--headless"] });

    try {
      const result = await lighthouse(url, {
        port: chrome.port,
        onlyCategories: ["performance"],
        settings: {
          onlyAudits: [
            "first-contentful-paint",
            "largest-contentful-paint",
            "cumulative-layout-shift",
            "total-blocking-time",
          ],
        },
      });

      this.printResults(result.lhr);
      return this.analyzeIssues(result.lhr);
    } finally {
      await chrome.kill();
    }
  }

  printResults(lhr) {
    const audits = lhr.audits;
    const score = Math.round(lhr.categories.performance.score * 100);

    console.log(chalk.blue("📊 Performance Results:"));
    console.log("─".repeat(40));

    const scoreColor =
      score >= 90 ? chalk.green : score >= 70 ? chalk.yellow : chalk.red;
    console.log(`Performance Score: ${scoreColor(score + "/100")}`);

    console.log(
      `FCP: ${this.formatTime(audits["first-contentful-paint"].numericValue)}`
    );
    console.log(
      `LCP: ${this.formatTime(audits["largest-contentful-paint"].numericValue)}`
    );
    console.log(
      `CLS: ${audits["cumulative-layout-shift"].numericValue.toFixed(3)}`
    );
    console.log(
      `TBT: ${Math.round(audits["total-blocking-time"].numericValue)}ms\n`
    );
  }

  formatTime(ms) {
    const seconds = (ms / 1000).toFixed(1);
    const color =
      ms < 1000 ? chalk.green : ms < 2500 ? chalk.yellow : chalk.red;
    return color(seconds + "s");
  }

  analyzeIssues(lhr) {
    const issues = [];
    const audits = lhr.audits;

    // Check Core Web Vitals
    if (audits["first-contentful-paint"].numericValue > 1000) {
      issues.push({
        type: "FCP",
        severity: "high",
        current: audits["first-contentful-paint"].numericValue,
        target: 1000,
        suggestions: [
          "Ensure critical icons are inline in components",
          "Optimize above-the-fold content",
          "Remove render-blocking resources",
        ],
      });
    }

    if (audits["largest-contentful-paint"].numericValue > 2500) {
      issues.push({
        type: "LCP",
        severity: "critical",
        current: audits["largest-contentful-paint"].numericValue,
        target: 2500,
        suggestions: [
          "Preload LCP image/element",
          "Optimize critical rendering path",
          "Use CDN for static assets",
        ],
      });
    }

    if (audits["cumulative-layout-shift"].numericValue > 0.1) {
      issues.push({
        type: "CLS",
        severity: "high",
        current: audits["cumulative-layout-shift"].numericValue,
        target: 0.1,
        suggestions: [
          "Set explicit dimensions on images and icons",
          "Reserve space for dynamic content",
          "Avoid inserting content above existing content",
        ],
      });
    }

    return issues;
  }
}

// Execute
const audit = new QuickAudit();
audit
  .run()
  .then((issues) => {
    if (issues.length > 0) {
      console.log(chalk.red("❌ Issues found:"));
      issues.forEach((issue) => {
        console.log(
          `\n${issue.type}: ${issue.current}ms (target: ${issue.target}ms)`
        );
        issue.suggestions.forEach((suggestion) => {
          console.log(`  • ${suggestion}`);
        });
      });
    } else {
      console.log(chalk.green("✅ No issues found!"));
    }
  })
  .catch(console.error);
```

## 🚀 Estrategias de Optimización

### 1. Above-the-fold Optimization

```typescript
// app/layout/header.component.ts
import { Component, ChangeDetectionStrategy } from "@angular/core";

@Component({
  selector: "app-header",
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <!-- ✅ Header crítico con iconos inline - Renderizado inmediato -->
    <header class="header">
      <nav class="nav-container">
        <!-- Logo inline - CRÍTICO para FCP -->
        <a routerLink="/" class="logo" [attr.aria-label]="'Ir al inicio'">
          <svg
            viewBox="0 0 120 40"
            width="120"
            height="40"
            class="logo-svg"
            [attr.aria-label]="'Logo ACME'"
          >
            <path d="M12 0L24 12L12 24L0 12Z" fill="currentColor" />
            <text
              x="30"
              y="28"
              font-size="24"
              font-weight="bold"
              fill="currentColor"
            >
              ACME
            </text>
          </svg>
        </a>

        <!-- Navegación crítica -->
        <ul class="nav-menu" role="menubar">
          <li role="none">
            <a routerLink="/productos" role="menuitem">Productos</a>
          </li>
          <li role="none">
            <a routerLink="/ofertas" role="menuitem">Ofertas</a>
          </li>
        </ul>

        <!-- Acciones críticas con iconos inline -->
        <div class="nav-actions">
          <!-- Búsqueda crítica -->
          <button
            class="search-btn"
            (click)="toggleSearch()"
            [attr.aria-expanded]="isSearchOpen"
            aria-label="Abrir búsqueda"
          >
            <svg viewBox="0 0 24 24" width="20" height="20" aria-hidden="true">
              <circle
                cx="11"
                cy="11"
                r="8"
                stroke="currentColor"
                stroke-width="2"
                fill="none"
              />
              <path
                d="M21 21l-4.35-4.35"
                stroke="currentColor"
                stroke-width="2"
              />
            </svg>
          </button>

          <!-- Carrito crítico -->
          <a
            routerLink="/carrito"
            class="cart-btn"
            [attr.aria-label]="'Ver carrito (' + cartCount + ' productos)'"
          >
            <svg viewBox="0 0 24 24" width="24" height="24" aria-hidden="true">
              <path
                d="M7 4V2C7 1.45 7.45 1 8 1H16C16.55 1 17 1.45 17 2V4H20C20.55 4 21 4.45 21 5S20.55 6 20 6H19V19C19 20.1 18.1 21 17 21H7C5.9 21 5 20.1 5 19V6H4C3.45 6 3 5.55 3 5S3.45 4 4 4H7Z"
                fill="currentColor"
              />
            </svg>
            <span
              class="cart-count"
              *ngIf="cartCount > 0"
              [attr.aria-hidden]="true"
            >
              {{ cartCount }}
            </span>
          </a>
        </div>
      </nav>
    </header>
  `,
  styles: [
    `
      .header {
        /* Critical CSS inline para evitar render blocking */
        position: sticky;
        top: 0;
        z-index: 1000;
        background: white;
        border-bottom: 1px solid #e2e8f0;
        box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
      }

      .nav-container {
        max-width: 1200px;
        margin: 0 auto;
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 1rem;
        /* Evitar CLS con height fijo */
        min-height: 64px;
      }

      .logo-svg {
        /* Dimensiones explícitas para evitar CLS */
        width: 120px;
        height: 40px;
        color: var(--primary-color);
        /* Optimización de rendering */
        shape-rendering: geometricPrecision;
      }

      /* ...existing code... */
    `,
  ],
})
export class HeaderComponent {
  isSearchOpen = false;
  cartCount = 0; // Esto vendría del servicio

  toggleSearch() {
    this.isSearchOpen = !this.isSearchOpen;
  }
}
```

### 2. Hero Section Optimizado

```typescript
// home/hero.component.ts
import { Component, ChangeDetectionStrategy, OnInit } from "@angular/core";

@Component({
  selector: "app-hero",
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <!-- Hero optimizado para LCP -->
    <section class="hero" [style.background-image]="heroBackgroundUrl">
      <div class="hero-content">
        <!-- Contenido crítico above-the-fold -->
        <h1 class="hero-title">
          Black Friday
          <span class="hero-highlight">Hasta 70% OFF</span>
        </h1>

        <p class="hero-subtitle">
          Los mejores descuentos del año en tecnología, moda y hogar
        </p>

        <!-- CTAs críticos con iconos inline -->
        <div class="hero-actions">
          <button class="cta-primary" (click)="shopNow()" type="button">
            <!-- ✅ Icono crítico inline para above-the-fold -->
            <svg viewBox="0 0 24 24" width="20" height="20" aria-hidden="true">
              <path
                d="M19 7h-3V6a4 4 0 0 0-8 0v1H5a1 1 0 0 0-1 1v11a3 3 0 0 0 3 3h10a3 3 0 0 0 3-3V8a1 1 0 0 0-1-1zM10 6a2 2 0 0 1 4 0v1h-4V6z"
                fill="currentColor"
              />
            </svg>
            Comprar Ahora
          </button>

          <a routerLink="/ofertas" class="cta-secondary">
            <svg viewBox="0 0 24 24" width="20" height="20" aria-hidden="true">
              <path
                d="M12 2L3.09 8.26L12 14.1l8.91-5.84L12 2z"
                fill="currentColor"
              />
            </svg>
            Ver Ofertas
          </a>
        </div>

        <!-- Features above-the-fold con iconos inline -->
        <div class="hero-features">
          <div class="feature-item">
            <svg viewBox="0 0 24 24" width="24" height="24" aria-hidden="true">
              <path
                d="M12 2L2 7v10c0 5.55 3.84 10.74 9 12 5.16-1.26 9-6.45 9-12V7l-10-5z"
                fill="currentColor"
              />
            </svg>
            <span>Compra Segura</span>
          </div>

          <div class="feature-item">
            <svg viewBox="0 0 24 24" width="24" height="24" aria-hidden="true">
              <path
                d="M19 3h-1V1h-2v2H8V1H6v2H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm0 16H5V8h14v11z"
                fill="currentColor"
              />
            </svg>
            <span>Envío 24h</span>
          </div>

          <div class="feature-item">
            <svg viewBox="0 0 24 24" width="24" height="24" aria-hidden="true">
              <path
                d="M12 2L3.09 8.26L12 14.1l8.91-5.84L12 2z"
                fill="currentColor"
              />
            </svg>
            <span>Mejor Precio</span>
          </div>
        </div>
      </div>
    </section>
  `,
  styles: [
    `
      .hero {
        /* Evitar CLS con dimensiones fijas */
        min-height: 60vh;
        display: flex;
        align-items: center;
        justify-content: center;
        text-align: center;
        color: white;
        position: relative;
        background-size: cover;
        background-position: center;
        background-repeat: no-repeat;
      }

      /* Overlay para mejorar legibilidad */
      .hero::before {
        content: "";
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background: rgba(0, 0, 0, 0.4);
        z-index: 1;
      }

      .hero-content {
        position: relative;
        z-index: 2;
        max-width: 800px;
        padding: 2rem;
      }

      .hero-title {
        font-size: clamp(2rem, 5vw, 4rem);
        font-weight: 800;
        margin-bottom: 1rem;
        line-height: 1.2;
      }

      .hero-highlight {
        display: block;
        color: #ffd700;
        font-size: 1.2em;
      }

      .hero-subtitle {
        font-size: clamp(1rem, 2.5vw, 1.5rem);
        margin-bottom: 2rem;
        opacity: 0.9;
      }

      .hero-actions {
        display: flex;
        gap: 1rem;
        justify-content: center;
        flex-wrap: wrap;
        margin-bottom: 3rem;
      }

      .cta-primary,
      .cta-secondary {
        display: inline-flex;
        align-items: center;
        gap: 0.5rem;
        padding: 1rem 2rem;
        font-size: 1.125rem;
        font-weight: 600;
        border-radius: 0.5rem;
        transition: all 0.3s ease;
        text-decoration: none;
        border: 2px solid transparent;
        /* Evitar CLS con min-width */
        min-width: 180px;
        justify-content: center;
      }

      .cta-primary {
        background: #ff6b35;
        color: white;
        border-color: #ff6b35;
      }

      .cta-primary:hover {
        background: #e55a2b;
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(255, 107, 53, 0.4);
      }

      .cta-secondary {
        background: transparent;
        color: white;
        border-color: white;
      }

      .cta-secondary:hover {
        background: white;
        color: #333;
      }

      .hero-features {
        display: flex;
        justify-content: center;
        gap: 2rem;
        flex-wrap: wrap;
      }

      .feature-item {
        display: flex;
        align-items: center;
        gap: 0.5rem;
        font-weight: 500;
        /* Evitar CLS */
        min-width: 120px;
      }

      .feature-item svg {
        flex-shrink: 0;
      }

      /* Responsive */
      @media (max-width: 768px) {
        .hero-actions {
          flex-direction: column;
          align-items: center;
        }

        .hero-features {
          gap: 1rem;
        }

        .feature-item {
          min-width: 100px;
          font-size: 0.875rem;
        }
      }
    `,
  ],
})
export class HeroComponent implements OnInit {
  heroBackgroundUrl = "";

  ngOnInit() {
    // Preload hero image crítico
    this.preloadHeroImage();
  }

  private preloadHeroImage() {
    const imageUrl = "url(/assets/hero-bg.webp)";
    this.heroBackgroundUrl = imageUrl;

    // Preload additional format fallbacks
    if (typeof window !== "undefined") {
      const link = document.createElement("link");
      link.rel = "preload";
      link.as = "image";
      link.href = "/assets/hero-bg.webp";
      link.type = "image/webp";
      document.head.appendChild(link);
    }
  }

  shopNow() {
    // Navigate to shop with tracking
    console.log("CTA: Shop Now clicked");
  }
}
```

### 3. Preloading Strategy

```typescript
// app.config.ts
import { ApplicationConfig } from "@angular/core";
import {
  provideRouter,
  withPreloading,
  PreloadAllModules,
} from "@angular/router";
import { provideClientHydration } from "@angular/platform-browser";

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withPreloading(PreloadAllModules)),
    provideClientHydration(),
    // Otros providers...
  ],
};
```

### 4. Resource Hints en index.html

```html
<!-- src/index.html -->
<!DOCTYPE html>
<html lang="es">
  <head>
    <meta charset="utf-8" />
    <title>ACME E-commerce</title>
    <base href="/" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link rel="icon" type="image/x-icon" href="/favicon.ico" />

    <!-- ✅ Preconnect to critical domains -->
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://cdn.miempresa.com" />
    <link rel="preconnect" href="https://analytics.google.com" />

    <!-- ✅ Preload critical resources -->
    <link
      rel="preload"
      href="/assets/fonts/Inter-Regular.woff2"
      as="font"
      type="font/woff2"
      crossorigin
    />
    <link
      rel="preload"
      href="/assets/hero-bg.webp"
      as="image"
      type="image/webp"
    />

    <!-- ✅ Critical CSS inline -->
    <style>
      /* Critical above-the-fold styles */
      body {
        margin: 0;
        font-family: Inter, system-ui, sans-serif;
        line-height: 1.6;
        color: #333;
      }

      /* Loading skeleton to prevent CLS */
      .loading-skeleton {
        background: linear-gradient(
          90deg,
          #f0f0f0 25%,
          #e0e0e0 50%,
          #f0f0f0 75%
        );
        background-size: 200% 100%;
        animation: loading 1.5s infinite;
      }

      @keyframes loading {
        0% {
          background-position: 200% 0;
        }
        100% {
          background-position: -200% 0;
        }
      }

      /* Header critical styles */
      .header {
        position: sticky;
        top: 0;
        z-index: 1000;
        background: white;
        border-bottom: 1px solid #e2e8f0;
        box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
      }
    </style>
  </head>
  <body>
    <app-root>
      <!-- ✅ Loading state to prevent CLS -->
      <div
        style="min-height: 100vh; display: flex; align-items: center; justify-content: center;"
      >
        <div
          class="loading-skeleton"
          style="width: 100px; height: 40px; border-radius: 4px;"
        ></div>
      </div>
    </app-root>

    <!-- ✅ Non-critical CSS deferred -->
    <link
      rel="stylesheet"
      href="/assets/non-critical.css"
      media="print"
      onload="this.media='all'"
    />
  </body>
</html>
```

## 📊 Monitoreo Automatizado

### Script de Lighthouse CI

```javascript
// tools/lighthouse-ci.mjs
import { execSync } from "child_process";
import { writeFileSync, readFileSync } from "fs";
import chalk from "chalk";

class LighthouseCI {
  constructor() {
    this.urls = [
      { name: "Home", url: "http://localhost:4200/", weight: 3 },
      { name: "Productos", url: "http://localhost:4200/productos", weight: 2 },
      { name: "Producto", url: "http://localhost:4200/producto/1", weight: 2 },
      { name: "Carrito", url: "http://localhost:4200/carrito", weight: 1 },
    ];

    this.thresholds = {
      performance: 95,
      fcp: 1000,
      lcp: 2500,
      cls: 0.1,
      tbt: 300,
    };
  }

  async runCI() {
    console.log(chalk.blue("🚀 Running Lighthouse CI...\n"));

    const results = [];
    let totalScore = 0;
    let totalWeight = 0;

    for (const { name, url, weight } of this.urls) {
      console.log(chalk.gray(`Testing ${name}...`));

      const result = await this.runSingleAudit(url);
      result.name = name;
      result.weight = weight;
      results.push(result);

      totalScore += result.performance * weight;
      totalWeight += weight;

      console.log(chalk.green(`✅ ${name}: ${result.performance}/100`));
    }

    const averageScore = Math.round(totalScore / totalWeight);

    // Generate report
    this.generateReport(results, averageScore);

    // Check if CI should pass
    const ciPassed = this.validateThresholds(results, averageScore);

    if (ciPassed) {
      console.log(chalk.green("\n✅ Lighthouse CI PASSED"));
    } else {
      console.log(chalk.red("\n❌ Lighthouse CI FAILED"));
      process.exit(1);
    }
  }

  async runSingleAudit(url) {
    try {
      const command = `lighthouse ${url} --chrome-flags="--headless" --output=json --quiet`;
      const output = execSync(command, { encoding: "utf8" });
      const report = JSON.parse(output);

      return {
        performance: Math.round(report.categories.performance.score * 100),
        fcp: report.audits["first-contentful-paint"].numericValue,
        lcp: report.audits["largest-contentful-paint"].numericValue,
        cls: report.audits["cumulative-layout-shift"].numericValue,
        tbt: report.audits["total-blocking-time"].numericValue,
      };
    } catch (error) {
      console.error(`Failed to audit ${url}:`, error.message);
      return {
        performance: 0,
        fcp: 9999,
        lcp: 9999,
        cls: 1,
        tbt: 9999,
      };
    }
  }

  validateThresholds(results, averageScore) {
    let passed = true;

    // Check average performance
    if (averageScore < this.thresholds.performance) {
      console.log(
        chalk.red(
          `❌ Average performance ${averageScore} < ${this.thresholds.performance}`
        )
      );
      passed = false;
    }

    // Check individual metrics
    for (const result of results) {
      if (result.fcp > this.thresholds.fcp) {
        console.log(
          chalk.red(
            `❌ ${result.name} FCP ${Math.round(result.fcp)}ms > ${
              this.thresholds.fcp
            }ms`
          )
        );
        passed = false;
      }

      if (result.lcp > this.thresholds.lcp) {
        console.log(
          chalk.red(
            `❌ ${result.name} LCP ${Math.round(result.lcp)}ms > ${
              this.thresholds.lcp
            }ms`
          )
        );
        passed = false;
      }

      if (result.cls > this.thresholds.cls) {
        console.log(
          chalk.red(
            `❌ ${result.name} CLS ${result.cls.toFixed(3)} > ${
              this.thresholds.cls
            }`
          )
        );
        passed = false;
      }
    }

    return passed;
  }

  generateReport(results, averageScore) {
    const report = {
      timestamp: new Date().toISOString(),
      averageScore,
      results,
      thresholds: this.thresholds,
    };

    writeFileSync(
      "reports/lighthouse-ci.json",
      JSON.stringify(report, null, 2)
    );

    // Generate HTML report
    const html = this.generateHTMLReport(report);
    writeFileSync("reports/lighthouse-ci.html", html);

    console.log(chalk.blue("\n📊 Reports generated:"));
    console.log("  📄 reports/lighthouse-ci.json");
    console.log("  🌐 reports/lighthouse-ci.html");
  }

  generateHTMLReport(report) {
    return `<!DOCTYPE html>
<html>
<head>
  <title>Lighthouse CI Report</title>
  <style>
    body { font-family: system-ui, sans-serif; margin: 2rem; }
    .header { text-align: center; margin-bottom: 2rem; }
    .score { font-size: 3rem; font-weight: bold; margin: 1rem 0; }
    .score.good { color: #0cce6b; }
    .score.average { color: #ffa400; }
    .score.poor { color: #ff4e42; }
    table { width: 100%; border-collapse: collapse; margin: 2rem 0; }
    th, td { padding: 1rem; text-align: left; border-bottom: 1px solid #eee; }
    th { background: #f8f9fa; }
    .metric { text-align: center; }
    .metric.good { color: #0cce6b; }
    .metric.average { color: #ffa400; }
    .metric.poor { color: #ff4e42; }
  </style>
</head>
<body>
  <div class="header">
    <h1>📊 Lighthouse CI Report</h1>
    <div class="score ${
      report.averageScore >= 90
        ? "good"
        : report.averageScore >= 70
        ? "average"
        : "poor"
    }">
      ${report.averageScore}/100
    </div>
    <p>Generated: ${new Date(report.timestamp).toLocaleString()}</p>
  </div>
  
  <table>
    <thead>
      <tr>
        <th>Page</th>
        <th>Performance</th>
        <th>FCP</th>
        <th>LCP</th>
        <th>CLS</th>
        <th>TBT</th>
      </tr>
    </thead>
    <tbody>
      ${report.results
        .map(
          (result) => `
        <tr>
          <td>${result.name}</td>
          <td class="metric ${
            result.performance >= 90
              ? "good"
              : result.performance >= 70
              ? "average"
              : "poor"
          }">
            ${result.performance}
          </td>
          <td class="metric ${
            result.fcp <= 1000
              ? "good"
              : result.fcp <= 2500
              ? "average"
              : "poor"
          }">
            ${(result.fcp / 1000).toFixed(1)}s
          </td>
          <td class="metric ${
            result.lcp <= 2500
              ? "good"
              : result.lcp <= 4000
              ? "average"
              : "poor"
          }">
            ${(result.lcp / 1000).toFixed(1)}s
          </td>
          <td class="metric ${
            result.cls <= 0.1 ? "good" : result.cls <= 0.25 ? "average" : "poor"
          }">
            ${result.cls.toFixed(3)}
          </td>
          <td class="metric ${
            result.tbt <= 300 ? "good" : result.tbt <= 600 ? "average" : "poor"
          }">
            ${Math.round(result.tbt)}ms
          </td>
        </tr>
      `
        )
        .join("")}
    </tbody>
  </table>
</body>
</html>`;
  }
}

// Execute
const ci = new LighthouseCI();
ci.runCI().catch(console.error);
```

## 🎯 Checklist de Optimización

- [ ] **Above-the-fold optimizado**

  - [ ] Iconos críticos inline en templates
  - [ ] CSS crítico inline en `<head>`
  - [ ] Preload de recursos críticos
  - [ ] Dimensiones fijas para evitar CLS

- [ ] **Below-the-fold eficiente**

  - [ ] Lazy loading de imágenes
  - [ ] Sprites de iconos desde CDN
  - [ ] Code splitting por rutas
  - [ ] Defer scripts no críticos

- [ ] **Core Web Vitals**

  - [ ] FCP < 1s
  - [ ] LCP < 2.5s
  - [ ] CLS < 0.1
  - [ ] FID < 100ms

- [ ] **Recursos optimizados**
  - [ ] Imágenes en WebP/AVIF
  - [ ] Fonts con font-display: swap
  - [ ] Minificación CSS/JS
  - [ ] Compresión Gzip/Brotli

---

**Siguiente**: [Monitoreo y Analytics](./monitoring.md)
