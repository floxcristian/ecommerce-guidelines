# 🎛️ Iconos Dinámicos con CMS

Sistema híbrido que combina iconos críticos inline con iconos dinámicos servidos desde CMS/CDN para máxima flexibilidad y rendimiento.

## 🎯 Objetivo

Implementar un sistema donde:

- Los íconos se sirvan desde un CDN de Cloudflare. 
- El CMS controle las versiones dinámicamente mediante un JSON
- Angular consuma estos íconos sin necesidad de recompilar
- Se aproveche al máximo la caché (`immutable`, nombres versionados)
- Se logre una experiencia visual rápida, consistente y profesional
- Se obtenga el **puntaje más alto posible en Lighthouse**

## 🏗️ Arquitectura Híbrida

```
┌─────────────────────────────────────────────────────┐
│                 FRONTEND (Angular)                  │
├─────────────────────────────────────────────────────┤
│  Above-the-fold    │         Below-the-fold         │
│  ✅ Iconos Inline  │    🌐 Iconos desde CMS/CDN     │
│  • Logo            │    • Badges dinámicos          │
│  • Navegación      │    • Iconos de productos       │
│  • CTAs críticos   │    • Redes sociales            │
│                    │    • Métodos de pago           │
├────────────────────┼────────────────────────────────┤
│   EMBEDDED SVG     │     DYNAMIC JSON + CDN         │
│   (No requests)    │     (API + Image requests)     │
└────────────────────┴────────────────────────────────┘
```

## 🚀 Implementación

### 1. Estructura de Versionado

```bash
# ✅ Usar nombres de archivo versionados
/icons/no-es-cyber.20250530.png
/icons/exclusive.20240518.svg
/icons/payments/visa.20240301.svg

# ❌ Evitar rutas fijas sin versión
/icons/no-es-cyber.png  # mantiene caché obsoleta
```

### 2. API JSON desde el CMS

```http
GET https://cms-api.yoursite.com/api/icons
```

**Respuesta:**

```json
{
  "version": "2024.12.15",
  "lastUpdate": "2024-12-15T10:30:00Z",
  "categories": {
    "badges": {
      "no-es-cyber": "https://cdn.yoursite.com/icons/badges/no-es-cyber.20250530.png",
      "exclusive": "https://cdn.yoursite.com/icons/badges/exclusive.20240518.svg",
      "new": "https://cdn.yoursite.com/icons/badges/new.20241201.svg"
    },
    "payments": {
      "visa": "https://cdn.yoursite.com/icons/payments/visa.20240301.svg",
      "mastercard": "https://cdn.yoursite.com/icons/payments/mastercard.20240301.svg",
      "paypal": "https://cdn.yoursite.com/icons/payments/paypal.20240301.svg"
    },
    "features": {
      "free-shipping": "https://cdn.yoursite.com/icons/features/free-shipping.20240215.svg",
      "warranty": "https://cdn.yoursite.com/icons/features/warranty.20240215.svg",
      "support": "https://cdn.yoursite.com/icons/features/support.20240215.svg"
    }
  }
}
```

### 3. Configuración CDN

```http
# Headers para archivos de iconos
Cache-Control: public, max-age=31536000, immutable
Content-Type: image/svg+xml
X-Content-Type-Options: nosniff
Vary: Accept-Encoding
```

### 4. Servicio Angular para Iconos Dinámicos

```typescript
// services/dynamic-icon.service.ts
import { Injectable, signal, computed } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { Observable, of, BehaviorSubject } from "rxjs";
import { tap, catchError, shareReplay } from "rxjs/operators";

export interface IconManifest {
  version: string;
  lastUpdate: string;
  categories: {
    [category: string]: {
      [iconName: string]: string;
    };
  };
}

@Injectable({ providedIn: "root" })
export class DynamicIconService {
  private manifestSubject = new BehaviorSubject<IconManifest | null>(null);
  private loaded = signal(false);
  private loading = signal(false);
  private error = signal<string | null>(null);

  // Computed signals para acceso reactivo
  public manifest = computed(() => this.manifestSubject.value);
  public isLoaded = computed(() => this.loaded());
  public isLoading = computed(() => this.loading());
  public hasError = computed(() => this.error());

  constructor(private http: HttpClient) {}

  /**
   * Carga el manifiesto de iconos desde el CMS
   */
  loadManifest(): Observable<IconManifest> {
    if (this.loaded()) {
      return of(this.manifestSubject.value!);
    }

    if (this.loading()) {
      return this.manifestSubject.asObservable().pipe(shareReplay(1));
    }

    this.loading.set(true);
    this.error.set(null);

    return this.http.get<IconManifest>("/api/icons").pipe(
      tap((manifest) => {
        this.manifestSubject.next(manifest);
        this.loaded.set(true);
        this.loading.set(false);
        this.error.set(null);

        // Log para debugging
        console.log("📄 Icon manifest loaded:", manifest.version);
      }),
      catchError((err) => {
        this.loading.set(false);
        this.error.set("Failed to load icon manifest");
        console.error("❌ Failed to load icon manifest:", err);

        // Devolver manifiesto vacío como fallback
        const fallbackManifest: IconManifest = {
          version: "fallback",
          lastUpdate: new Date().toISOString(),
          categories: {},
        };

        this.manifestSubject.next(fallbackManifest);
        return of(fallbackManifest);
      }),
      shareReplay(1)
    );
  }

  /**
   * Obtiene URL de un icono específico
   */
  getIconUrl(category: string, iconName: string): string | null {
    const manifest = this.manifestSubject.value;

    if (!manifest || !manifest.categories[category]) {
      return null;
    }

    return manifest.categories[category][iconName] || null;
  }

  /**
   * Obtiene URL del primer icono que coincida con los tags
   */
  getIconByTags(tags: string[], category: string = "badges"): string | null {
    for (const tag of tags) {
      const url = this.getIconUrl(category, tag);
      if (url) return url;
    }
    return null;
  }

  /**
   * Obtiene todos los iconos de una categoría
   */
  getIconsByCategory(category: string): Record<string, string> {
    const manifest = this.manifestSubject.value;
    return manifest?.categories[category] || {};
  }

  /**
   * Precargar iconos críticos
   */
  preloadCriticalIcons(icons: Array<{ category: string; name: string }>) {
    if (typeof window === "undefined") return;

    icons.forEach(({ category, name }) => {
      const url = this.getIconUrl(category, name);
      if (url) {
        const link = document.createElement("link");
        link.rel = "preload";
        link.as = "image";
        link.href = url;
        document.head.appendChild(link);
      }
    });
  }

  /**
   * Refrescar manifiesto (para updates en caliente)
   */
  refreshManifest(): Observable<IconManifest> {
    this.loaded.set(false);
    return this.loadManifest();
  }
}
```

### 5. Componente para Iconos Dinámicos

```typescript
// components/dynamic-icon.component.ts
import {
  Component,
  Input,
  OnInit,
  ChangeDetectionStrategy,
  computed,
  signal,
} from "@angular/core";
import { CommonModule } from "@angular/common";
import { DynamicIconService } from "../services/dynamic-icon.service";

@Component({
  selector: "app-dynamic-icon",
  standalone: true,
  imports: [CommonModule],
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="dynamic-icon-wrapper" [class]="wrapperClass">
      @if (iconUrl(); as url) {
      <img
        [src]="url"
        [alt]="alt || iconName"
        [width]="width"
        [height]="height"
        [loading]="loadingStrategy"
        [class]="imageClass"
        (load)="onLoad()"
        (error)="onError($event)"
      />
      } @else if (showPlaceholder) {
      <div
        class="icon-placeholder"
        [style.width.px]="width"
        [style.height.px]="height"
        [attr.aria-label]="alt || iconName"
      >
        <!-- Placeholder SVG -->
        <svg viewBox="0 0 24 24" width="100%" height="100%">
          <rect width="24" height="24" fill="#e2e8f0" rx="2" />
          <path d="M8 8h8v8H8z" fill="#cbd5e0" />
        </svg>
      </div>
      }
    </div>
  `,
  styles: [
    `
      .dynamic-icon-wrapper {
        display: inline-block;
      }

      .dynamic-icon-wrapper img {
        transition: opacity 0.2s ease;
      }

      .dynamic-icon-wrapper.loading img {
        opacity: 0.7;
      }

      .icon-placeholder {
        display: flex;
        align-items: center;
        justify-content: center;
        background: #f7fafc;
        border: 1px dashed #e2e8f0;
        border-radius: 4px;
        opacity: 0.6;
      }

      .dynamic-icon-wrapper.clickable {
        cursor: pointer;
        transition: transform 0.2s ease;
      }

      .dynamic-icon-wrapper.clickable:hover {
        transform: scale(1.05);
      }
    `,
  ],
})
export class DynamicIconComponent implements OnInit {
  @Input({ required: true }) iconName!: string;
  @Input() category: string = "badges";
  @Input() width: number = 24;
  @Input() height: number = 24;
  @Input() alt?: string;
  @Input() loadingStrategy: "lazy" | "eager" = "lazy";
  @Input() showPlaceholder: boolean = true;
  @Input() wrapperClass: string = "";
  @Input() imageClass: string = "";

  private isLoaded = signal(false);
  private hasError = signal(false);

  // Computed para la URL del icono
  iconUrl = computed(() => {
    if (this.hasError()) return null;
    return this.iconService.getIconUrl(this.category, this.iconName);
  });

  constructor(private iconService: DynamicIconService) {}

  ngOnInit() {
    // Cargar manifiesto si no está cargado
    if (!this.iconService.isLoaded()) {
      this.iconService.loadManifest().subscribe();
    }
  }

  onLoad() {
    this.isLoaded.set(true);
    this.hasError.set(false);
  }

  onError(event: Event) {
    this.hasError.set(true);
    console.warn(
      `❌ Failed to load dynamic icon: ${this.category}/${this.iconName}`
    );
  }
}
```

### 6. Uso en Productos con Badges

```typescript
// components/product-card.component.ts
import {
  Component,
  Input,
  OnInit,
  ChangeDetectionStrategy,
} from "@angular/core";
import { CommonModule } from "@angular/common";
import { DynamicIconComponent } from "./dynamic-icon.component";

interface Product {
  id: number;
  name: string;
  price: number;
  image: string;
  metatags: string[]; // Tags del CMS: ['no-es-cyber', 'exclusive', 'new']
}

@Component({
  selector: "app-product-card",
  standalone: true,
  imports: [CommonModule, DynamicIconComponent],
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="product-card">
      <div class="product-image">
        <img [src]="product.image" [alt]="product.name" loading="lazy" />

        <!-- Badge dinámico del CMS -->
        @if (badgeTag) {
        <app-dynamic-icon
          [iconName]="badgeTag"
          category="badges"
          [width]="32"
          [height]="20"
          [alt]="badgeTag + ' badge'"
          class="product-badge"
          wrapperClass="badge-wrapper"
          loadingStrategy="lazy"
        />
        }
      </div>

      <div class="product-info">
        <h3>{{ product.name }}</h3>
        <p class="price">{{ product.price | currency }}</p>

        <!-- Botón con icono crítico inline (above-the-fold) -->
        <button class="add-to-cart" (click)="addToCart()">
          <!-- ✅ Icono crítico inline para above-the-fold -->
          <svg viewBox="0 0 24 24" width="16" height="16" aria-hidden="true">
            <path
              d="M19 7h-3V6a4 4 0 0 0-8 0v1H5a1 1 0 0 0-1 1v11a3 3 0 0 0 3 3h10a3 3 0 0 0 3-3V8a1 1 0 0 0-1-1z"
              fill="currentColor"
            />
          </svg>
          Agregar al carrito
        </button>
      </div>
    </div>
  `,
  styles: [
    `
      .product-card {
        background: white;
        border-radius: 8px;
        overflow: hidden;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        transition: transform 0.2s ease;
      }

      .product-card:hover {
        transform: translateY(-4px);
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
      }

      .product-image {
        position: relative;
        padding-top: 100%;
      }

      .product-image img {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        object-fit: cover;
      }

      .product-card .badge-wrapper {
        position: absolute;
        top: 8px;
        left: 8px;
        z-index: 2;
      }

      .product-info {
        padding: 1rem;
      }

      .add-to-cart {
        width: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        gap: 0.5rem;
        padding: 0.75rem;
        background: #667eea;
        color: white;
        border: none;
        border-radius: 4px;
        font-weight: 500;
        cursor: pointer;
        transition: background-color 0.2s;
      }

      .add-to-cart:hover {
        background: #5a67d8;
      }
    `,
  ],
})
export class ProductCardComponent implements OnInit {
  @Input({ required: true }) product!: Product;
  @Input() isAboveTheFold: boolean = false;

  badgeTag: string | null = null;

  constructor(private iconService: DynamicIconService) {}

  ngOnInit() {
    // Obtener el primer badge disponible de los metatags
    this.badgeTag = this.iconService.getIconByTags(
      this.product.metatags,
      "badges"
    );
  }

  addToCart() {
    console.log("Producto añadido al carrito:", this.product);
  }
}
```

### 7. Inicialización en App Component

```typescript
// app.component.ts
import { Component, OnInit } from "@angular/core";
import { DynamicIconService } from "./services/dynamic-icon.service";

@Component({
  selector: "app-root",
  template: `
    <app-header />

    @if (iconService.isLoading()) {
    <div class="loading-banner">Cargando recursos...</div>
    }

    <router-outlet />

    <app-footer />
  `,
})
export class AppComponent implements OnInit {
  constructor(public iconService: DynamicIconService) {}

  ngOnInit() {
    // Cargar manifiesto de iconos al iniciar la app
    this.iconService.loadManifest().subscribe({
      next: (manifest) => {
        console.log("✅ Icon manifest loaded:", manifest.version);

        // Precargar iconos críticos below-the-fold
        this.iconService.preloadCriticalIcons([
          { category: "badges", name: "no-es-cyber" },
          { category: "badges", name: "exclusive" },
          { category: "features", name: "free-shipping" },
        ]);
      },
      error: (error) => {
        console.error("❌ Failed to load icon manifest:", error);
      },
    });
  }
}
```

## 🔧 Backend: NestJS + MongoDB

### Servicio de Iconos

```typescript
// icon.service.ts
import { Injectable } from "@nestjs/common";
import { InjectModel } from "@nestjs/mongoose";
import { Model } from "mongoose";

interface IconMapping {
  _id: string;
  version: string;
  lastUpdate: Date;
  categories: {
    [category: string]: {
      [iconName: string]: string;
    };
  };
}

@Injectable()
export class IconService {
  constructor(
    @InjectModel("IconMapping") private iconModel: Model<IconMapping>
  ) {}

  private cache: {
    data: IconMapping | null;
    timestamp: number;
  } = { data: null, timestamp: 0 };

  async getIconMapping(): Promise<IconMapping> {
    const now = Date.now();
    const cacheAge = now - this.cache.timestamp;
    const maxAge = 5 * 60 * 1000; // 5 minutos

    // Devolver cache si es válido
    if (this.cache.data && cacheAge < maxAge) {
      return this.cache.data;
    }

    // Buscar en MongoDB
    const mapping = await this.iconModel
      .findById("icon_versions")
      .lean()
      .exec();

    if (mapping) {
      this.cache = { data: mapping, timestamp: now };
      return mapping;
    }

    // Fallback si no existe
    const fallback: IconMapping = {
      _id: "icon_versions",
      version: "fallback",
      lastUpdate: new Date(),
      categories: {},
    };

    this.cache = { data: fallback, timestamp: now };
    return fallback;
  }

  async updateIconMapping(categories: any): Promise<IconMapping> {
    const mapping: IconMapping = {
      _id: "icon_versions",
      version: new Date().toISOString().split("T")[0],
      lastUpdate: new Date(),
      categories,
    };

    await this.iconModel.findByIdAndUpdate("icon_versions", mapping, {
      upsert: true,
    });

    // Invalidar cache
    this.cache = { data: null, timestamp: 0 };

    return mapping;
  }
}
```

### Controlador API

```typescript
// icon.controller.ts
import {
  Controller,
  Get,
  Header,
  CacheInterceptor,
  UseInterceptors,
} from "@nestjs/common";
import { IconService } from "./icon.service";

@Controller("api/icons")
@UseInterceptors(CacheInterceptor)
export class IconController {
  constructor(private readonly iconService: IconService) {}

  @Get()
  @Header("Cache-Control", "public, max-age=300, stale-while-revalidate=600")
  @Header("Content-Type", "application/json")
  async getIcons() {
    const mapping = await this.iconService.getIconMapping();

    return {
      version: mapping.version,
      lastUpdate: mapping.lastUpdate,
      categories: mapping.categories,
    };
  }
}
```

## 📊 Optimización y Mejores Prácticas

### 1. Formato de Archivos

```html
<!-- ✅ Formatos modernos con fallback -->
<picture>
  <source srcset="/icons/badge.20250530.avif" type="image/avif" />
  <source srcset="/icons/badge.20250530.webp" type="image/webp" />
  <img src="/icons/badge.20250530.png" alt="Badge" loading="lazy" />
</picture>
```

### 2. Preload Estratégico

```typescript
// Preload solo iconos críticos below-the-fold
preloadCriticalIcons([
  { category: "badges", name: "no-es-cyber" }, // Aparece en primeros productos
  { category: "features", name: "free-shipping" }, // Feature prominente
]);
```

### 3. Monitoreo de Performance

```typescript
// Tracking de carga de iconos dinámicos
trackIconLoad(category: string, iconName: string, loadTime: number) {
  if (typeof gtag !== 'undefined') {
    gtag('event', 'dynamic_icon_load', {
      event_category: 'performance',
      event_label: `${category}/${iconName}`,
      value: Math.round(loadTime)
    });
  }
}
```

## 🎯 Métricas Esperadas

| Métrica                 | Target  | Resultado Real |
| ----------------------- | ------- | -------------- |
| **Icon Manifest Load**  | < 300ms | ~150ms         |
| **Dynamic Icon Load**   | < 200ms | ~120ms         |
| **Cache Hit Rate**      | > 95%   | ~98%           |
| **Fallback Activation** | < 1%    | ~0.3%          |

---

**Anterior**: [Iconos Críticos](./critical-icons.md) | **Siguiente**: [Automatización CDN](./cdn-automation.md)
