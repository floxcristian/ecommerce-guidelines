# 🚀 Angular SSR Optimization para E-commerce

Guía completa para implementar Server-Side Rendering optimizado en aplicaciones e-commerce Angular con iconos críticos, hydration y métricas de performance.

## 🎯 Objetivos del SSR

- **FCP < 1s**: First Contentful Paint inmediato con contenido desde el servidor
- **LCP < 2.5s**: Largest Contentful Paint optimizado con critical CSS e iconos inline
- **SEO Excellence**: Contenido completamente indexable por crawlers
- **Performance**: Hydration no-blocking y lazy loading inteligente
- **UX**: Transición suave de SSR a CSR sin flickering

## 📊 Comparativa SSR vs CSR

| Métrica       | CSR (Client-Side) | SSR (Server-Side) | Mejora     |
| ------------- | ----------------- | ----------------- | ---------- |
| **FCP**       | 2.1s              | 0.8s              | **62%** ⬇️ |
| **LCP**       | 3.5s              | 1.9s              | **46%** ⬇️ |
| **TTI**       | 4.2s              | 2.8s              | **33%** ⬇️ |
| **SEO Score** | 60/100            | 98/100            | **63%** ⬆️ |
| **CLS**       | 0.15              | 0.02              | **87%** ⬇️ |

## 🏗️ Configuración Base Angular SSR

### 1. Setup Inicial con Angular Universal

```bash
# Agregar Angular Universal al proyecto NX
nx g @nguniversal/express-engine:ng-add ecommerce-angular
```

### 2. Configuración del App Module

```typescript
// apps/ecommerce-angular/src/app/app.config.ts
import { ApplicationConfig, mergeApplicationConfig } from "@angular/core";
import { provideRouter } from "@angular/router";
import { provideClientHydration } from "@angular/platform-browser";
import { provideHttpClient, withFetch } from "@angular/common/http";
import { provideIcons } from "@acme/icons";
import { provideAnalytics } from "@acme/analytics";
import { routes } from "./app.routes";

const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideClientHydration(),
    provideHttpClient(withFetch()),
    provideIcons({
      cdnUrl: "https://cdn.miempresa.com/icons",
      fallbackEnabled: true,
      preloadStrategies: [{ section: "core", priority: "high" }],
    }),
    provideAnalytics({
      enableWebVitals: true,
      enableUserTiming: true,
    }),
  ],
};

export default appConfig;
```

### 3. Configuración del Server

```typescript
// apps/ecommerce-angular/src/app/app.config.server.ts
import { ApplicationConfig, mergeApplicationConfig } from "@angular/core";
import { provideServerRendering } from "@angular/platform-server";
import { provideHttpClient, withFetch } from "@angular/common/http";
import appConfig from "./app.config";

const serverConfig: ApplicationConfig = {
  providers: [
    provideServerRendering(),
    provideHttpClient(withFetch()),
    // Server-specific providers
  ],
};

export default mergeApplicationConfig(appConfig, serverConfig);
```

## 🎨 Optimización de Iconos en SSR

### 1. Iconos Críticos Inline en SSR

```typescript
// apps/ecommerce-angular/src/app/layout/layout.component.ts
import { Component, inject, OnInit, Inject, PLATFORM_ID } from "@angular/core";
import { isPlatformBrowser, isPlatformServer } from "@angular/common";
import { MetaService } from "@angular/platform-browser";

@Component({
  selector: "app-layout",
  standalone: true,
  template: `
    <header class="header" [class.ssr-ready]="!isBrowser">
      <nav class="nav">
        <!-- Logo crítico - SIEMPRE inline para SSR -->
        <a routerLink="/" class="logo" [attr.aria-label]="logoAriaLabel">
          <svg viewBox="0 0 120 40" width="120" height="40" class="logo-svg">
            <path d="M12 0L24 12L12 24L0 12Z" fill="currentColor" />
            <text x="30" y="28" class="logo-text">ACME</text>
          </svg>
        </a>

        <!-- Navigation crítica inline -->
        <div class="nav-actions">
          <!-- Search inline para SSR -->
          <button
            class="search-btn"
            (click)="toggleSearch()"
            [attr.aria-expanded]="isSearchOpen"
            aria-label="Abrir búsqueda"
          >
            <svg viewBox="0 0 24 24" width="20" height="20" class="search-icon">
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
                stroke-linecap="round"
              />
            </svg>
          </button>

          <!-- Cart inline para SSR -->
          <button
            class="cart-btn"
            routerLink="/cart"
            aria-label="Ver carrito de compras"
          >
            <svg viewBox="0 0 24 24" width="24" height="24" class="cart-icon">
              <path
                d="M7 4V2C7 1.45 7.45 1 8 1H16C16.55 1 17 1.45 17 2V4H20C20.55 4 21 4.45 21 5S20.55 6 20 6H19V19C19 20.1 18.1 21 17 21H7C5.9 21 5 20.1 5 19V6H4C3.45 6 3 5.55 3 5S3.45 4 4 4H7Z"
                fill="currentColor"
              />
            </svg>
            <span
              class="cart-count"
              *ngIf="cartCount > 0"
              [textContent]="cartCount"
            ></span>
          </button>
        </div>
      </nav>
    </header>
  `,
  styles: [
    `
      .header {
        position: sticky;
        top: 0;
        z-index: 1000;
        background: white;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        transition: opacity 0.3s ease;
      }

      /* Evitar flickering durante hydration */
      .header.ssr-ready {
        opacity: 1;
      }

      .logo-svg {
        width: 120px;
        height: 40px;
        color: var(--primary-color, #667eea);
        display: block;
      }

      .logo-text {
        font-family: "Inter", -apple-system, BlinkMacSystemFont, sans-serif;
        font-weight: 700;
        font-size: 24px;
        fill: currentColor;
      }

      .nav-actions {
        display: flex;
        align-items: center;
        gap: 1rem;
      }

      .search-btn,
      .cart-btn {
        display: flex;
        align-items: center;
        justify-content: center;
        padding: 0.5rem;
        background: transparent;
        border: none;
        border-radius: 0.375rem;
        cursor: pointer;
        color: var(--text-color, #374151);
        transition: all 0.2s ease;
      }

      .cart-btn {
        position: relative;
      }

      .cart-count {
        position: absolute;
        top: -4px;
        right: -4px;
        background: var(--accent-color, #ef4444);
        color: white;
        border-radius: 50%;
        width: 20px;
        height: 20px;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 12px;
        font-weight: 600;
        line-height: 1;
      }
    `,
  ],
})
export class LayoutComponent implements OnInit {
  isBrowser: boolean;
  isSearchOpen = false;
  cartCount = 0;
  logoAriaLabel = "ACME - Ir al inicio";

  constructor(
    @Inject(PLATFORM_ID) private platformId: Object,
    private meta: MetaService
  ) {
    this.isBrowser = isPlatformBrowser(this.platformId);
  }

  ngOnInit() {
    if (this.isBrowser) {
      this.loadCartCount();
    }
    this.preloadCriticalData();
  }

  toggleSearch() {
    this.isSearchOpen = !this.isSearchOpen;
  }

  private loadCartCount() {
    const savedCount = localStorage.getItem("cartCount");
    if (savedCount) {
      this.cartCount = parseInt(savedCount, 10);
    }
  }

  private preloadCriticalData() {
    this.meta.updateTag({
      name: "description",
      content: "ACME E-commerce - Los mejores productos al mejor precio",
    });
    this.meta.updateTag({ property: "og:title", content: "ACME E-commerce" });
  }
}
```

### 2. Critical CSS Strategy

```scss
// apps/ecommerce-angular/src/styles.scss

/* =================== CRITICAL STYLES (Above-the-fold) =================== */

:root {
  --primary-color: #667eea;
  --primary-dark: #5a67d8;
  --accent-color: #ef4444;
  --text-color: #374151;
  --border-color: #e5e7eb;
  --background: #ffffff;
}

/* Reset y base críticos */
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html {
  line-height: 1.5;
  -webkit-text-size-adjust: 100%;
  font-family: "Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
}

body {
  background-color: var(--background);
  color: var(--text-color);
  font-size: 16px;
  line-height: 1.6;
  font-display: swap;
}

/* Header crítico */
.header {
  position: sticky;
  top: 0;
  z-index: 1000;
  background: var(--background);
  border-bottom: 1px solid var(--border-color);
  contain: layout style paint;
}

/* Hero section crítico */
.hero {
  background: linear-gradient(
    135deg,
    var(--primary-color) 0%,
    var(--primary-dark) 100%
  );
  color: white;
  padding: 4rem 2rem;
  text-align: center;
  min-height: 60vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

/* Loading states para SSR */
.loading-skeleton {
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
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

/* =================== NON-CRITICAL STYLES =================== */
@import "non-critical.scss" print, (min-width: 1px);
```

## 🚀 Optimización de Hydration

### 1. Hydration Strategy

```typescript
// apps/ecommerce-angular/src/app/app.component.ts
import { Component, OnInit, inject, PLATFORM_ID } from "@angular/core";
import { isPlatformBrowser } from "@angular/common";
import { Router, NavigationEnd } from "@angular/router";
import { filter } from "rxjs/operators";

@Component({
  selector: "app-root",
  standalone: true,
  imports: [RouterOutlet, LayoutComponent],
  template: `
    <app-layout>
      <router-outlet></router-outlet>
    </app-layout>
  `,
  styles: [
    `
      :host {
        display: block;
        min-height: 100vh;
      }
    `,
  ],
})
export class AppComponent implements OnInit {
  private platformId = inject(PLATFORM_ID);
  private router = inject(Router);

  ngOnInit() {
    if (isPlatformBrowser(this.platformId)) {
      this.setupHydrationOptimizations();
      this.setupPerformanceMonitoring();
    }
  }

  private setupHydrationOptimizations() {
    setTimeout(() => {
      document.body.classList.add("hydrated");
      this.enableInteractiveFeatures();
    }, 0);
  }

  private setupPerformanceMonitoring() {
    if (process.env["NODE_ENV"] === "development") {
      import("web-vitals").then(({ getCLS, getFCP, getLCP }) => {
        getCLS((metric) => console.log("CLS:", metric.value));
        getFCP((metric) => console.log("FCP:", metric.value));
        getLCP((metric) => console.log("LCP:", metric.value));
      });
    }
  }

  private enableInteractiveFeatures() {
    this.setupLazyLoading();
    document.documentElement.style.scrollBehavior = "smooth";
  }

  private setupLazyLoading() {
    if ("IntersectionObserver" in window) {
      const imageObserver = new IntersectionObserver((entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            const img = entry.target as HTMLImageElement;
            img.src = img.dataset["src"] || "";
            img.classList.remove("lazy");
            imageObserver.unobserve(img);
          }
        });
      });

      document.querySelectorAll("img[data-src]").forEach((img) => {
        imageObserver.observe(img);
      });
    }
  }
}
```

### 2. Home Component con Progressive Enhancement

```typescript
// apps/ecommerce-angular/src/app/home/home.component.ts
import { Component, AfterViewInit, inject, PLATFORM_ID } from "@angular/core";
import { isPlatformBrowser } from "@angular/common";
import { CommonModule } from "@angular/common";
import { IconComponent } from "@acme/icons";

@Component({
  selector: "app-home",
  standalone: true,
  imports: [CommonModule, IconComponent],
  template: `
    <!-- Hero section con iconos críticos inline -->
    <section class="hero">
      <h1>{{ heroTitle }}</h1>
      <p>{{ heroSubtitle }}</p>

      <a href="/productos" class="cta-primary">
        <svg viewBox="0 0 24 24" width="20" height="20" class="cta-icon">
          <path
            d="M19 7h-3V6a4 4 0 0 0-8 0v1H5a1 1 0 0 0-1 1v11a3 3 0 0 0 3 3h10a3 3 0 0 0 3-3V8a1 1 0 0 0-1-1z"
            fill="currentColor"
          />
        </svg>
        Comprar Ahora
      </a>
    </section>

    <!-- Productos destacados -->
    <section class="featured-products">
      <h2>Productos Destacados</h2>

      <div class="product-grid" *ngIf="!productsLoaded && !isBrowser">
        <div
          class="product-card loading-skeleton"
          *ngFor="let item of [1, 2, 3, 4]"
        >
          <div class="product-image skeleton"></div>
          <div class="product-info">
            <div class="skeleton-text"></div>
            <div class="skeleton-button"></div>
          </div>
        </div>
      </div>

      <div class="product-grid" *ngIf="productsLoaded || isBrowser">
        <div
          class="product-card"
          *ngFor="let product of featuredProducts; let i = index"
        >
          <img
            [src]="i < 2 ? product.image : ''"
            [attr.data-src]="i >= 2 ? product.image : null"
            [alt]="product.name"
            [class.lazy]="i >= 2"
            [loading]="i < 2 ? 'eager' : 'lazy'"
          />

          <div class="product-info">
            <h3>{{ product.name }}</h3>
            <p class="price">{{ product.price | currency }}</p>

            <button class="add-to-cart-btn" (click)="addToCart(product)">
              <svg *ngIf="i < 2" viewBox="0 0 24 24" width="20" height="20">
                <path
                  d="M19 7h-3V6a4 4 0 0 0-8 0v1H5a1 1 0 0 0-1 1v11a3 3 0 0 0 3 3h10a3 3 0 0 0 3-3V8a1 1 0 0 0-1-1z"
                  fill="currentColor"
                />
              </svg>

              <ui-icon *ngIf="i >= 2" name="cart" section="core" size="20" />

              Agregar al Carrito
            </button>
          </div>
        </div>
      </div>
    </section>

    <!-- Newsletter (solo en browser) -->
    <section class="newsletter" *ngIf="showNewsletter && isBrowser">
      <ui-icon name="mail" section="core" size="32" />
      <h2>Suscríbete y obtén 10% OFF</h2>
      <form (ngSubmit)="subscribeNewsletter()">
        <input
          type="email"
          placeholder="Tu email"
          [(ngModel)]="newsletterEmail"
        />
        <button type="submit">
          <ui-icon name="send" section="core" size="20" />
          Suscribir
        </button>
      </form>
    </section>
  `,
  styles: [
    `
      .hero {
        background: linear-gradient(
          135deg,
          var(--primary-color) 0%,
          var(--primary-dark) 100%
        );
        color: white;
        padding: 4rem 2rem;
        text-align: center;
        min-height: 60vh;
        display: flex;
        flex-direction: column;
        justify-content: center;
      }

      .cta-primary {
        display: inline-flex;
        align-items: center;
        gap: 0.5rem;
        padding: 1rem 2rem;
        background: white;
        color: var(--primary-color);
        border: none;
        border-radius: 0.5rem;
        font-size: 1.125rem;
        font-weight: 600;
        text-decoration: none;
        cursor: pointer;
        transition: transform 0.2s ease;
      }

      .cta-primary:hover {
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
      }

      .product-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
        gap: 2rem;
        padding: 2rem;
      }

      .product-card {
        background: white;
        border-radius: 0.5rem;
        overflow: hidden;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        transition: transform 0.2s ease;
      }

      .loading-skeleton {
        animation: loading 1.5s infinite;
      }

      .skeleton {
        background: #f0f0f0;
        border-radius: 0.25rem;
      }

      .product-image.skeleton {
        width: 100%;
        height: 200px;
      }

      .skeleton-text {
        height: 1rem;
        background: #f0f0f0;
        border-radius: 0.25rem;
        margin-bottom: 0.5rem;
      }

      .skeleton-button {
        height: 40px;
        background: #f0f0f0;
        border-radius: 0.25rem;
        margin-top: 1rem;
      }

      @keyframes loading {
        0% {
          opacity: 1;
        }
        50% {
          opacity: 0.4;
        }
        100% {
          opacity: 1;
        }
      }
    `,
  ],
})
export class HomeComponent implements AfterViewInit {
  private platformId = inject(PLATFORM_ID);

  isBrowser: boolean;
  productsLoaded = false;
  showNewsletter = false;
  newsletterEmail = "";

  heroTitle = "Black Friday: Hasta 70% OFF en toda la tienda";
  heroSubtitle =
    "Descubre ofertas increíbles en miles de productos. Envío gratis en compras mayores a $50.";

  featuredProducts = [
    {
      id: 1,
      name: "Auriculares Bluetooth Premium",
      price: 89.99,
      image: "/assets/images/product-1.jpg",
    },
    {
      id: 2,
      name: "Smartwatch Deportivo",
      price: 159.99,
      image: "/assets/images/product-2.jpg",
    },
    {
      id: 3,
      name: "Cámara Digital 4K",
      price: 299.99,
      image: "/assets/images/product-3.jpg",
    },
    {
      id: 4,
      name: "Laptop Gaming Ultra",
      price: 899.99,
      image: "/assets/images/product-4.jpg",
    },
  ];

  constructor() {
    this.isBrowser = isPlatformBrowser(this.platformId);
  }

  ngAfterViewInit() {
    if (this.isBrowser) {
      setTimeout(() => {
        this.productsLoaded = true;
        this.showNewsletter = true;
      }, 100);
    } else {
      this.productsLoaded = true;
    }
  }

  addToCart(product: any) {
    console.log("Adding to cart:", product);
  }

  subscribeNewsletter() {
    console.log("Newsletter subscription:", this.newsletterEmail);
  }
}
```

## 📊 Performance Monitoring Service

```typescript
// libs/analytics/src/lib/ssr-performance.service.ts
import { Injectable, Inject, PLATFORM_ID } from "@angular/core";
import { isPlatformBrowser } from "@angular/common";

export interface SSRMetrics {
  serverRenderTime?: number;
  hydrationTime?: number;
  fcp?: number;
  lcp?: number;
  cls?: number;
}

@Injectable({
  providedIn: "root",
})
export class SSRPerformanceService {
  private metrics: SSRMetrics = {};
  private isBrowser: boolean;

  constructor(@Inject(PLATFORM_ID) platformId: Object) {
    this.isBrowser = isPlatformBrowser(platformId);
  }

  recordWebVitals() {
    if (!this.isBrowser) return;

    import("web-vitals").then(({ getCLS, getFCP, getLCP }) => {
      getCLS((metric) => {
        this.metrics.cls = metric.value;
        this.sendMetrics("cls", metric.value);
      });

      getFCP((metric) => {
        this.metrics.fcp = metric.value;
        this.sendMetrics("fcp", metric.value);
      });

      getLCP((metric) => {
        this.metrics.lcp = metric.value;
        this.sendMetrics("lcp", metric.value);
      });
    });
  }

  private sendMetrics(metric: string, value: number) {
    console.log(`SSR Metric - ${metric}:`, value);

    if (typeof fetch !== "undefined") {
      fetch("/api/analytics/performance", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          metric,
          value,
          timestamp: Date.now(),
          page: window.location.pathname,
        }),
      }).catch((err) => console.error("Failed to send metrics:", err));
    }
  }

  getMetrics(): SSRMetrics {
    return { ...this.metrics };
  }
}
```

## 🎯 Checklist de SSR

- [ ] **Angular Universal configurado**
- [ ] **Iconos críticos inline en templates**
- [ ] **Critical CSS implementado**
- [ ] **Hydration optimizada**
- [ ] **Progressive enhancement activo**
- [ ] **Performance monitoring configurado**
- [ ] **Lazy loading implementado**
- [ ] **Meta tags y SEO configurados**

---

**Siguiente**: [Deployment y CI/CD](./deployment.md)
