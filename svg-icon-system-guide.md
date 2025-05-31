# 🧭 Guía Técnica: Sistema de Iconos SVG con Sprites y Cloudflare CDN

## 🎯 Objetivo

Sistema escalable de iconos SVG optimizado para rendimiento máximo (Lighthouse 100/100) usando sprites servidos desde Cloudflare CDN, compatible con Angular SSR y arquitectura NX.

## 📋 Tabla de Contenidos

1. [Conceptos Clave](#conceptos-clave)
2. [Arquitectura del Sistema](#arquitectura-del-sistema)
3. [Estrategia de Carga](#estrategia-de-carga)
4. [Implementación](#implementación)
5. [Uso en Componentes](#uso-en-componentes)
6. [Automatización CDN](#automatización-cdn)
7. [Comandos y Scripts](#comandos-y-scripts)
8. [Métricas y Beneficios](#métricas-y-beneficios)

## 🔑 Conceptos Clave

### Above-the-fold vs Below-the-fold

**Above-the-fold** = Todo el contenido que es **visible en la pantalla SIN hacer scroll**. Es lo PRIMERO que ve el usuario al cargar la página.

**Below-the-fold** = Contenido que requiere hacer scroll para verlo.

```
┌─────────────────────────────────┐ ◄── Viewport del navegador
│  🏠 Logo    🔍 Search   🛒 Cart │ ◄── ABOVE THE FOLD
│                                 │     (Visible sin scroll)
│      OFERTA BLACK FRIDAY        │
│     [🛍️ COMPRAR AHORA]         │
│                                 │
│   ⭐ Producto 1  ⭐ Producto 2   │
│                                 │
├─────────────────────────────────┤ ◄── El "fold" (límite de pantalla)
│                                 │
│   ⭐ Producto 3  ⭐ Producto 4   │ ◄── BELOW THE FOLD
│                                 │     (Requiere scroll)
│   📦 Envío     🛡️ Garantía     │
│                                 │
└─────────────────────────────────┘
```

### ⚡ Estrategia de Rendimiento

Para lograr Lighthouse 100/100, especialmente en métricas como **LCP** y **FCP**:

1. **Iconos Above-the-fold (Críticos)**: SIEMPRE inline directamente en el template HTML
2. **Iconos Below-the-fold (No Críticos)**: Sprites desde CDN con lazy loading

**❌ EVITAR**: Archivos TypeScript separados para iconos críticos (añaden latencia)  
**✅ PREFERIR**: SVG inline directo en el template del componente

## 🏗️ Arquitectura del Sistema

```
apps/
  ecommerce-angular/
    src/
      app/
        home/
          home.component.ts         # Iconos críticos inline en template
        shared/
          layout/
            layout.component.ts     # Header con iconos críticos inline
libs/
  icons/                           # 📦 Librería de iconos NO críticos
    src/
      assets/
        individual/                # SVGs individuales fuente
          core/
          home/
          admin/
          cms/
      lib/
        components/
          icon.component.ts        # Wrapper Angular para sprites
          icon.component.spec.ts   # Tests unitarios
        types/
          icon.types.ts            # Tipado de iconos
        config/
          icon.config.ts           # Configuración CDN
        providers/
          icon.providers.ts        # Provider functions
        references/
          critical-icons.md        # Referencia de SVGs críticos (NO código)
    tools/
      generate-sprites.mjs         # Script de generación
      generate-types.mjs           # Generador de tipos
      preview-icons.mjs            # Preview local
      validate-icons.mjs           # Validación de SVGs
scripts/
  deploy-icons.mjs                 # Subida a Cloudflare
```

## 🚀 Estrategia de Carga

### 📊 Comparación de Rendimiento

| Método                 | FCP            | LCP            | TTI         | Complejidad | Cuándo Usar                    |
| ---------------------- | -------------- | -------------- | ----------- | ----------- | ------------------------------ |
| SVG inline en template | ⚡ Instantáneo | ⚡ Instantáneo | ✅ Óptimo   | 🟢 Simple   | Iconos críticos above-the-fold |
| Archivo TS separado    | ⏱️ +50-100ms   | ⏱️ +50-100ms   | ⚠️ Más JS   | 🟡 Media    | ❌ Nunca para críticos         |
| Sprite externo         | ⏱️ +100-200ms  | ⏱️ +100-200ms  | ✅ Menos JS | 🟡 Media    | ✅ Iconos below-the-fold       |

### 1. Iconos Críticos Inline (MEJOR RENDIMIENTO)

**IMPORTANTE**: Los iconos críticos deben estar directamente en el template HTML del componente, NO en archivos separados.

```typescript
// ✅ CORRECTO - layout.component.ts
@Component({
  selector: "app-layout",
  standalone: true,
  template: `
    <header class="header">
      <nav class="nav">
        <!-- Logo crítico inline directo - Se renderiza INMEDIATAMENTE con SSR -->
        <a routerLink="/" class="logo" aria-label="Ir al inicio">
          <svg viewBox="0 0 120 40" width="120" height="40" class="logo-svg">
            <path d="M12 0L24 12L12 24L0 12Z" fill="currentColor" />
            <text
              x="30"
              y="28"
              font-family="Arial, sans-serif"
              font-size="24"
              font-weight="bold"
              fill="currentColor"
            >
              ACME
            </text>
          </svg>
        </a>

        <!-- Menu hamburguesa crítico inline -->
        <button
          class="menu-btn"
          (click)="toggleMenu()"
          [attr.aria-expanded]="isMenuOpen"
          aria-label="Abrir menú de navegación"
        >
          <svg viewBox="0 0 24 24" width="24" height="24">
            <path
              d="M3 12h18M3 6h18M3 18h18"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
            />
          </svg>
        </button>

        <!-- Búsqueda crítica inline -->
        <div class="search-box" role="search">
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
          <input
            type="search"
            placeholder="Buscar productos..."
            aria-label="Buscar productos"
          />
        </div>

        <!-- Carrito crítico inline -->
        <button class="cart-btn" routerLink="/cart" aria-label="Ver carrito">
          <svg viewBox="0 0 24 24" width="24" height="24">
            <path
              d="M7 4V2C7 1.45 7.45 1 8 1H16C16.55 1 17 1.45 17 2V4H20C20.55 4 21 4.45 21 5S20.55 6 20 6H19V19C19 20.1 18.1 21 17 21H7C5.9 21 5 20.1 5 19V6H4C3.45 6 3 5.55 3 5S3.45 4 4 4H7Z"
              fill="currentColor"
            />
          </svg>
          <span class="cart-count" *ngIf="cartCount > 0">{{ cartCount }}</span>
        </button>
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
      }

      .logo-svg {
        /* Definir dimensiones evita reflow y CLS */
        width: 120px;
        height: 40px;
        color: var(--primary-color);
      }

      .menu-btn svg,
      .cart-btn svg {
        width: 24px;
        height: 24px;
      }

      .search-icon {
        width: 20px;
        height: 20px;
        flex-shrink: 0;
      }

      .cart-btn {
        position: relative;
      }

      .cart-count {
        position: absolute;
        top: -8px;
        right: -8px;
        background: var(--primary-color);
        color: white;
        border-radius: 50%;
        width: 20px;
        height: 20px;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 12px;
        font-weight: bold;
      }
    `,
  ],
})
export class LayoutComponent {
  isMenuOpen = false;
  cartCount = 0;

  toggleMenu() {
    this.isMenuOpen = !this.isMenuOpen;
  }
}
```

### 2. Referencia de Iconos Críticos (Solo Documentación)

````markdown
<!-- libs/icons/src/lib/references/critical-icons.md -->

# Referencia de Iconos Críticos

Esta es una referencia de los SVGs críticos para copiar/pegar en los componentes.
**NO importar como código**, usar directamente en los templates.

## Logo

```svg
<svg viewBox="0 0 120 40" width="120" height="40">
  <path d="M12 0L24 12L12 24L0 12Z" fill="currentColor"/>
  <text x="30" y="28" font-family="Arial, sans-serif" font-size="24" font-weight="bold" fill="currentColor">ACME</text>
</svg>
```
````

## Menú Hamburguesa

```svg
<svg viewBox="0 0 24 24" width="24" height="24">
  <path d="M3 12h18M3 6h18M3 18h18" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
</svg>
```

## Carrito

```svg
<svg viewBox="0 0 24 24" width="24" height="24">
  <path d="M7 4V2C7 1.45 7.45 1 8 1H16C16.55 1 17 1.45 17 2V4H20C20.55 4 21 4.45 21 5S20.55 6 20 6H19V19C19 20.1 18.1 21 17 21H7C5.9 21 5 20.1 5 19V6H4C3.45 6 3 5.55 3 5S3.45 4 4 4H7Z" fill="currentColor"/>
</svg>
```

## Búsqueda

```svg
<svg viewBox="0 0 24 24" width="20" height="20">
  <circle cx="11" cy="11" r="8" stroke="currentColor" stroke-width="2" fill="none"/>
  <path d="M21 21l-4.35-4.35" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
</svg>
```

````

### 3. Sprites para Iconos No Críticos (CDN)

```typescript
// libs/icons/src/lib/config/icon.config.ts
import { InjectionToken } from '@angular/core';

export interface IconConfig {
  cdnUrl: string;
  fallbackEnabled: boolean;
  preloadStrategies?: PreloadStrategy[];
  cacheControl?: string;
}

export interface PreloadStrategy {
  section: SpriteSection;
  priority: 'high' | 'low';
}

export const ICON_CONFIG = new InjectionToken<IconConfig>('ICON_CONFIG');

export const defaultIconConfig: IconConfig = {
  cdnUrl: 'https://cdn.miempresa.com/icons',
  fallbackEnabled: true,
  preloadStrategies: [
    { section: 'core', priority: 'high' }
  ],
  cacheControl: 'public, max-age=31536000, immutable'
};

export type SpriteSection = 'core' | 'home' | 'admin' | 'cms';
````

## 🛠️ Implementación

### Script de Generación de Sprites

```javascript
// libs/icons/tools/generate-sprites.mjs
import { SVGSpriter } from "svg-sprite";
import { glob } from "glob";
import { readFileSync, writeFileSync, mkdirSync, statSync } from "fs";
import { createHash } from "crypto";

const config = {
  mode: {
    symbol: {
      dest: "dist/sprites",
      sprite: "sprite-{{section}}.svg",
      inline: true,
      example: false,
    },
  },
  shape: {
    id: {
      generator: "icon-%s",
    },
    transform: [
      {
        svgo: {
          plugins: [
            "preset-default",
            {
              name: "removeViewBox",
              active: false,
            },
            {
              name: "removeDimensions",
              active: true,
            },
          ],
        },
      },
    ],
  },
};

async function generateSprites() {
  const sections = ["core", "home", "admin", "cms"];
  const manifest = {};

  console.log("🚀 Generating sprites for non-critical icons...\n");

  for (const section of sections) {
    console.log(`📦 Processing ${section}...`);

    const spriter = new SVGSpriter(config);
    const icons = await glob(
      `libs/icons/src/assets/individual/${section}/*.svg`
    );

    if (icons.length === 0) {
      console.warn(`⚠️  No icons found for section: ${section}`);
      continue;
    }

    const sectionIcons = [];

    for (const iconPath of icons) {
      const content = readFileSync(iconPath, "utf-8");
      const name = iconPath.split("/").pop().replace(".svg", "");
      const stats = statSync(iconPath);

      spriter.add(iconPath, null, content);

      sectionIcons.push({
        name,
        size: stats.size,
        lastModified: stats.mtime,
      });
    }

    const { result } = await spriter.compile();
    const spriteContent = result.symbol.sprite.contents;

    // Generate hash for cache busting
    const hash = createHash("sha256")
      .update(spriteContent)
      .digest("hex")
      .substring(0, 8);

    const filename = `sprite-${section}-${hash}.svg`;

    mkdirSync("dist/sprites", { recursive: true });
    writeFileSync(`dist/sprites/${filename}`, spriteContent);

    manifest[section] = {
      filename,
      hash,
      icons: sectionIcons,
      size: Buffer.byteLength(spriteContent, "utf8"),
    };

    console.log(
      `✅ Generated ${filename} (${sectionIcons.length} icons, ${(
        manifest[section].size / 1024
      ).toFixed(1)}KB)`
    );
  }

  // Write manifest
  writeFileSync(
    "dist/sprites/manifest.json",
    JSON.stringify(manifest, null, 2)
  );
  console.log("\n📄 Manifest generated successfully!");
}

generateSprites().catch(console.error);
```

### Componente Angular para Sprites (Solo iconos no críticos)

```typescript
// libs/icons/src/lib/components/icon.component.ts
import {
  Component,
  Input,
  inject,
  OnInit,
  ChangeDetectionStrategy,
} from "@angular/core";
import { CommonModule } from "@angular/common";
import { ICON_CONFIG } from "../config/icon.config";
import type { IconName, SpriteSection } from "../types/icon.types";

@Component({
  selector: "ui-icon",
  standalone: true,
  imports: [CommonModule],
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <!-- Solo para iconos NO críticos below-the-fold -->
    <svg
      [attr.class]="iconClass"
      [attr.width]="size"
      [attr.height]="size"
      [attr.aria-label]="ariaLabel || name"
      role="img"
      (error)="onError($event)"
    >
      <use [attr.href]="spriteUrl + '#icon-' + name"></use>
    </svg>
  `,
  styles: [
    `
      :host {
        display: inline-flex;
        align-items: center;
        justify-content: center;
      }

      svg {
        fill: currentColor;
        transition: transform 0.2s ease;
      }

      .icon-clickable:hover {
        transform: scale(1.1);
      }
    `,
  ],
})
export class IconComponent implements OnInit {
  @Input({ required: true }) name!: IconName;
  @Input() section: SpriteSection = "core";
  @Input() size: string = "24";
  @Input() className = "";
  @Input() ariaLabel?: string;
  @Input() clickable = false;

  private config = inject(ICON_CONFIG);
  private static loadedSprites = new Set<string>();

  get iconClass() {
    const classes = ["icon", `icon-${this.name}`];
    if (this.className) classes.push(this.className);
    if (this.clickable) classes.push("icon-clickable");
    return classes.join(" ");
  }

  get spriteUrl() {
    return `${this.config.cdnUrl}/sprite-${this.section}`;
  }

  ngOnInit() {
    if (!IconComponent.loadedSprites.has(this.section)) {
      this.preloadSprite();
    }
  }

  private preloadSprite() {
    if (typeof window === "undefined") return; // Skip en SSR

    const link = document.createElement("link");
    link.rel = "prefetch";
    link.as = "image";
    link.href = `${this.spriteUrl}.svg`;
    link.type = "image/svg+xml";

    link.onload = () => {
      IconComponent.loadedSprites.add(this.section);
    };

    document.head.appendChild(link);
  }

  onError(event: Event) {
    console.error(
      `Failed to load icon: ${this.name} from section: ${this.section}`
    );
  }
}
```

### Validador de Iconos

```javascript
// libs/icons/tools/validate-icons.mjs
import { readFileSync, readdirSync } from "fs";
import { join } from "path";

const CRITICAL_ICONS = ["logo", "menu", "search", "cart", "chevron-down"];

function validateIcons() {
  console.log("🔍 Validating icon setup...\n");

  let hasErrors = false;

  // 1. Verificar que no hay iconos críticos en las carpetas de sprites
  const sections = ["core", "home", "admin", "cms"];

  for (const section of sections) {
    const path = join("libs/icons/src/assets/individual", section);
    try {
      const icons = readdirSync(path).filter((f) => f.endsWith(".svg"));

      for (const icon of icons) {
        const iconName = icon.replace(".svg", "");
        if (CRITICAL_ICONS.includes(iconName)) {
          console.error(
            `❌ Critical icon "${iconName}" found in ${section} folder!`
          );
          console.error(
            `   Critical icons should be inline in component templates, not in sprite folders.`
          );
          hasErrors = true;
        }
      }
    } catch (e) {
      // Folder doesn't exist, skip
    }
  }

  // 2. Verificar nombres de archivos
  for (const section of sections) {
    const path = join("libs/icons/src/assets/individual", section);
    try {
      const icons = readdirSync(path).filter((f) => f.endsWith(".svg"));

      for (const icon of icons) {
        if (icon !== icon.toLowerCase()) {
          console.error(
            `❌ Icon "${icon}" in ${section} has uppercase letters!`
          );
          hasErrors = true;
        }

        if (!/^[a-z0-9-]+\.svg$/.test(icon)) {
          console.error(
            `❌ Icon "${icon}" in ${section} has invalid characters!`
          );
          hasErrors = true;
        }
      }
    } catch (e) {
      // Folder doesn't exist, skip
    }
  }

  // 3. Verificar tamaño de archivos
  for (const section of sections) {
    const path = join("libs/icons/src/assets/individual", section);
    try {
      const icons = readdirSync(path).filter((f) => f.endsWith(".svg"));

      for (const icon of icons) {
        const content = readFileSync(join(path, icon), "utf-8");
        const size = Buffer.byteLength(content);

        if (size > 5000) {
          // 5KB
          console.warn(
            `⚠️  Icon "${icon}" in ${section} is large (${(size / 1024).toFixed(
              1
            )}KB)`
          );
        }
      }
    } catch (e) {
      // Folder doesn't exist, skip
    }
  }

  if (!hasErrors) {
    console.log("✅ All validations passed!");
  } else {
    console.log("\n❌ Validation failed! Fix the errors above.");
    process.exit(1);
  }
}

validateIcons();
```

## 🎨 Uso en Componentes

### Criterios para Clasificar Iconos

#### ✅ Iconos Críticos (Inline en el template)

- Logo principal
- Iconos del header (búsqueda, carrito, menú)
- CTAs principales del hero/banner
- Cualquier icono above-the-fold
- Iconos de navegación principal

#### ❌ Iconos No Críticos (Sprites desde CDN)

- Iconos del footer
- Iconos de features/beneficios
- Iconos de redes sociales
- Iconos de productos (excepto primeros 2-4)
- Iconos decorativos
- Cualquier icono below-the-fold

### Home Component

```typescript
// home/home.component.ts
import {
  Component,
  ChangeDetectionStrategy,
  AfterViewInit,
} from "@angular/core";
import { IconComponent } from "@acme/icons";
import { CommonModule } from "@angular/common";

@Component({
  selector: "app-home",
  standalone: true,
  imports: [IconComponent, CommonModule],
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <!-- Hero section con iconos críticos inline -->
    <section class="hero">
      <h1>Black Friday: Hasta 70% OFF</h1>

      <!-- CTA principal con icono crítico inline -->
      <button class="cta-primary" (click)="shopNow()">
        <svg viewBox="0 0 24 24" width="20" height="20" class="cta-icon">
          <path
            d="M19 7h-3V6a4 4 0 0 0-8 0v1H5a1 1 0 0 0-1 1v11a3 3 0 0 0 3 3h10a3 3 0 0 0 3-3V8a1 1 0 0 0-1-1zM10 6a2 2 0 0 1 4 0v1h-4V6z"
            fill="currentColor"
          />
        </svg>
        Comprar Ahora
      </button>

      <!-- Features above-the-fold con iconos inline -->
      <div class="hero-features">
        <div class="hero-feature">
          <svg viewBox="0 0 24 24" width="24" height="24">
            <path
              d="M12 2L2 7v10c0 5.55 3.84 10.74 9 12 5.16-1.26 9-6.45 9-12V7l-10-5z"
              fill="currentColor"
            />
          </svg>
          <span>Compra Segura</span>
        </div>
        <div class="hero-feature">
          <svg viewBox="0 0 24 24" width="24" height="24">
            <path
              d="M19 3h-1V1h-2v2H8V1H6v2H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm0 16H5V8h14v11z"
              fill="currentColor"
            />
          </svg>
          <span>Envío en 24h</span>
        </div>
      </div>
    </section>

    <!-- Productos destacados (parcialmente above-the-fold) -->
    <section class="featured-products">
      <h2>Productos Destacados</h2>
      <div class="product-grid">
        <div
          class="product-card"
          *ngFor="let product of featuredProducts; let i = index"
        >
          <img [src]="product.image" [alt]="product.name" loading="lazy" />
          <h3>{{ product.name }}</h3>
          <p class="price">{{ product.price | currency }}</p>

          <!-- Primeros 2 productos: iconos inline (pueden ser above-the-fold) -->
          <button
            class="add-to-cart"
            *ngIf="i < 2"
            (click)="addToCart(product)"
          >
            <svg viewBox="0 0 24 24" width="16" height="16">
              <path
                d="M19 7h-3V6a4 4 0 0 0-8 0v1H5a1 1 0 0 0-1 1v11a3 3 0 0 0 3 3h10a3 3 0 0 0 3-3V8a1 1 0 0 0-1-1z"
                fill="currentColor"
              />
            </svg>
            Agregar
          </button>

          <!-- Resto: usar componente icon (below-the-fold) -->
          <button
            class="add-to-cart"
            *ngIf="i >= 2"
            (click)="addToCart(product)"
          >
            <ui-icon name="cart" section="core" size="16" />
            Agregar
          </button>
        </div>
      </div>
    </section>

    <!-- ========= BELOW THE FOLD ========= -->

    <!-- Features secundarias con sprites (lazy loading) -->
    <section class="features" *ngIf="showFeatures">
      <div class="feature" *ngFor="let feature of features">
        <ui-icon
          [name]="feature.icon"
          section="home"
          size="48"
          [ariaLabel]="feature.title"
        />
        <h3>{{ feature.title }}</h3>
        <p>{{ feature.description }}</p>
      </div>
    </section>

    <!-- Newsletter con iconos no críticos -->
    <section class="newsletter" *ngIf="showNewsletter">
      <ui-icon name="mail" section="core" size="32" />
      <h2>Suscríbete y obtén 10% OFF</h2>
      <form>
        <input type="email" placeholder="Tu email" />
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
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        color: white;
        padding: 4rem 2rem;
        text-align: center;
      }

      .cta-primary {
        display: inline-flex;
        align-items: center;
        gap: 0.5rem;
        padding: 1rem 2rem;
        background: white;
        color: #667eea;
        border: none;
        border-radius: 0.5rem;
        font-size: 1.125rem;
        font-weight: 600;
        cursor: pointer;
        transition: transform 0.2s;
      }

      .cta-primary:hover {
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
      }

      .cta-icon {
        width: 20px;
        height: 20px;
      }

      .hero-features {
        display: flex;
        justify-content: center;
        gap: 2rem;
        margin-top: 2rem;
      }

      .hero-feature {
        display: flex;
        align-items: center;
        gap: 0.5rem;
      }

      .product-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
        gap: 2rem;
        padding: 2rem;
      }

      .add-to-cart {
        display: flex;
        align-items: center;
        gap: 0.5rem;
        width: 100%;
        justify-content: center;
        padding: 0.75rem;
        background: var(--primary-color);
        color: white;
        border: none;
        border-radius: 0.25rem;
        cursor: pointer;
      }

      .features {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
        gap: 2rem;
        padding: 4rem 2rem;
        background: #f7fafc;
      }

      .feature {
        text-align: center;
        padding: 2rem;
      }
    `,
  ],
})
export class HomeComponent implements AfterViewInit {
  showFeatures = false;
  showNewsletter = false;

  featuredProducts = [
    { id: 1, name: "Producto 1", price: 29.99, image: "/assets/p1.jpg" },
    { id: 2, name: "Producto 2", price: 39.99, image: "/assets/p2.jpg" },
    { id: 3, name: "Producto 3", price: 49.99, image: "/assets/p3.jpg" },
    { id: 4, name: "Producto 4", price: 59.99, image: "/assets/p4.jpg" },
  ];

  features = [
    {
      icon: "star",
      title: "Mejor valorado",
      description: "Productos con las mejores reseñas",
    },
    {
      icon: "heart",
      title: "Favoritos",
      description: "Los más queridos por nuestros clientes",
    },
    {
      icon: "clock",
      title: "Nuevos",
      description: "Últimas novedades en el catálogo",
    },
    {
      icon: "gift",
      title: "Regalos",
      description: "Ideas perfectas para regalar",
    },
  ];

  ngAfterViewInit() {
    // Cargar contenido below-the-fold después del contenido crítico
    setTimeout(() => {
      this.showFeatures = true;
    }, 100);

    setTimeout(() => {
      this.showNewsletter = true;
    }, 200);
  }

  shopNow() {
    // Navegar a la tienda
  }

  addToCart(product: any) {
    // Lógica para agregar al carrito
  }
}
```

### Product Component

```typescript
// product/product.component.ts
import { Component, Input, ChangeDetectionStrategy } from "@angular/core";
import { IconComponent } from "@acme/icons";
import { CommonModule, CurrencyPipe } from "@angular/common";

@Component({
  selector: "app-product",
  standalone: true,
  imports: [IconComponent, CommonModule, CurrencyPipe],
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="product-card">
      <div class="product-image">
        <img [src]="product.image" [alt]="product.name" loading="lazy" />

        <!-- Badge con icono inline si es nuevo -->
        <span class="badge new" *ngIf="product.isNew">
          <svg viewBox="0 0 16 16" width="16" height="16">
            <path
              d="M8 0l2.5 5.3 5.5.8-4 4.1.9 5.8L8 13.3 3.1 16l.9-5.8-4-4.1 5.5-.8z"
              fill="currentColor"
            />
          </svg>
          Nuevo
        </span>
      </div>

      <div class="product-info">
        <h3>{{ product.name }}</h3>
        <p class="price">{{ product.price | currency }}</p>

        <!-- Si el producto está above-the-fold, usar inline -->
        <button
          class="add-to-cart-btn"
          (click)="addToCart()"
          *ngIf="isAboveTheFold"
        >
          <svg viewBox="0 0 24 24" width="20" height="20">
            <path
              d="M19 7h-3V6a4 4 0 0 0-8 0v1H5a1 1 0 0 0-1 1v11a3 3 0 0 0 3 3h10a3 3 0 0 0 3-3V8a1 1 0 0 0-1-1zM10 6a2 2 0 0 1 4 0v1h-4V6z"
              fill="currentColor"
            />
          </svg>
          Agregar al carrito
        </button>

        <!-- Si está below-the-fold, usar sprite -->
        <button
          class="add-to-cart-btn"
          (click)="addToCart()"
          *ngIf="!isAboveTheFold"
        >
          <ui-icon name="cart" section="core" size="20" />
          Agregar al carrito
        </button>

        <!-- Acciones secundarias siempre con sprites (no críticas) -->
        <div class="product-actions">
          <button
            class="action-btn"
            (click)="toggleFavorite()"
            [class.active]="product.isFavorite"
          >
            <ui-icon name="heart" section="home" size="20" />
          </button>
          <button class="action-btn" (click)="share()">
            <ui-icon name="share" section="home" size="20" />
          </button>
          <button class="action-btn" (click)="compare()">
            <ui-icon name="compare" section="home" size="20" />
          </button>
        </div>
      </div>
    </div>
  `,
  styles: [
    `
      .product-card {
        background: white;
        border-radius: 0.5rem;
        overflow: hidden;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        transition: transform 0.2s;
      }

      .product-card:hover {
        transform: translateY(-4px);
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
      }

      .product-image {
        position: relative;
        padding-top: 100%; /* Aspect ratio 1:1 */
      }

      .product-image img {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        object-fit: cover;
      }

      .badge {
        position: absolute;
        top: 0.5rem;
        right: 0.5rem;
        background: var(--accent-color);
        color: white;
        padding: 0.25rem 0.75rem;
        border-radius: 0.25rem;
        font-size: 0.875rem;
        display: flex;
        align-items: center;
        gap: 0.25rem;
      }

      .product-info {
        padding: 1rem;
      }

      .add-to-cart-btn {
        width: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        gap: 0.5rem;
        padding: 0.75rem;
        background: var(--primary-color);
        color: white;
        border: none;
        border-radius: 0.25rem;
        font-weight: 600;
        cursor: pointer;
        transition: background 0.2s;
      }

      .add-to-cart-btn:hover {
        background: var(--primary-dark);
      }

      .product-actions {
        display: flex;
        justify-content: center;
        gap: 1rem;
        margin-top: 1rem;
      }

      .action-btn {
        padding: 0.5rem;
        background: transparent;
        border: 1px solid #e2e8f0;
        border-radius: 0.25rem;
        cursor: pointer;
        transition: all 0.2s;
      }

      .action-btn:hover {
        border-color: var(--primary-color);
        color: var(--primary-color);
      }

      .action-btn.active {
        background: var(--primary-color);
        color: white;
        border-color: var(--primary-color);
      }
    `,
  ],
})
export class ProductComponent {
  @Input() product!: any;
  @Input() isAboveTheFold = false; // Determinar si el producto está above-the-fold

  addToCart() {
    console.log("Producto añadido al carrito:", this.product);
  }

  toggleFavorite() {
    this.product.isFavorite = !this.product.isFavorite;
  }

  share() {
    // Lógica para compartir
  }

  compare() {
    // Lógica para comparar
  }
}
```

### Footer Component

```typescript
// footer.component.ts
import { Component } from "@angular/core";
import { IconComponent } from "@acme/icons";

@Component({
  selector: "app-footer",
  standalone: true,
  imports: [IconComponent],
  template: `
    <!-- Footer siempre está below-the-fold, usar sprites -->
    <footer class="footer">
      <div class="footer-content">
        <!-- Features del footer -->
        <div class="footer-features">
          <div class="footer-feature">
            <ui-icon name="truck" section="core" size="32" />
            <div>
              <h4>Envío Gratis</h4>
              <p>En compras mayores a $50</p>
            </div>
          </div>
          <div class="footer-feature">
            <ui-icon name="shield" section="core" size="32" />
            <div>
              <h4>Compra Segura</h4>
              <p>SSL 256-bit encryption</p>
            </div>
          </div>
          <div class="footer-feature">
            <ui-icon name="return" section="core" size="32" />
            <div>
              <h4>Devoluciones</h4>
              <p>30 días de garantía</p>
            </div>
          </div>
          <div class="footer-feature">
            <ui-icon name="support" section="core" size="32" />
            <div>
              <h4>Soporte 24/7</h4>
              <p>Chat en vivo disponible</p>
            </div>
          </div>
        </div>

        <!-- Links del footer -->
        <div class="footer-links">
          <div class="footer-column">
            <h4>Productos</h4>
            <ul>
              <li><a href="#">Novedades</a></li>
              <li><a href="#">Más vendidos</a></li>
              <li><a href="#">Ofertas</a></li>
              <li><a href="#">Categorías</a></li>
            </ul>
          </div>
          <div class="footer-column">
            <h4>Ayuda</h4>
            <ul>
              <li><a href="#">Centro de ayuda</a></li>
              <li><a href="#">Seguimiento de pedidos</a></li>
              <li><a href="#">Envíos</a></li>
              <li><a href="#">Devoluciones</a></li>
            </ul>
          </div>
          <div class="footer-column">
            <h4>Empresa</h4>
            <ul>
              <li><a href="#">Sobre nosotros</a></li>
              <li><a href="#">Carreras</a></li>
              <li><a href="#">Prensa</a></li>
              <li><a href="#">Sostenibilidad</a></li>
            </ul>
          </div>
        </div>

        <!-- Social y pagos -->
        <div class="footer-bottom">
          <div class="social-links">
            <a href="#" aria-label="Facebook">
              <ui-icon name="facebook" section="core" size="24" />
            </a>
            <a href="#" aria-label="Twitter">
              <ui-icon name="twitter" section="core" size="24" />
            </a>
            <a href="#" aria-label="Instagram">
              <ui-icon name="instagram" section="core" size="24" />
            </a>
            <a href="#" aria-label="YouTube">
              <ui-icon name="youtube" section="core" size="24" />
            </a>
          </div>

          <div class="payment-methods">
            <ui-icon name="visa" section="core" size="40" />
            <ui-icon name="mastercard" section="core" size="40" />
            <ui-icon name="paypal" section="core" size="40" />
            <ui-icon name="amex" section="core" size="40" />
          </div>
        </div>
      </div>
    </footer>
  `,
  styles: [
    `
      .footer {
        background: #1a202c;
        color: white;
        padding: 3rem 0 1rem;
        margin-top: 4rem;
      }

      .footer-content {
        max-width: 1200px;
        margin: 0 auto;
        padding: 0 1rem;
      }

      .footer-features {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
        gap: 2rem;
        padding: 2rem 0;
        border-bottom: 1px solid rgba(255, 255, 255, 0.1);
      }

      .footer-feature {
        display: flex;
        align-items: center;
        gap: 1rem;
      }

      .footer-links {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 2rem;
        padding: 2rem 0;
      }

      .footer-column h4 {
        margin-bottom: 1rem;
      }

      .footer-column ul {
        list-style: none;
        padding: 0;
      }

      .footer-column a {
        color: rgba(255, 255, 255, 0.7);
        text-decoration: none;
        line-height: 2;
      }

      .footer-column a:hover {
        color: white;
      }

      .footer-bottom {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding-top: 2rem;
        border-top: 1px solid rgba(255, 255, 255, 0.1);
      }

      .social-links {
        display: flex;
        gap: 1rem;
      }

      .social-links a {
        color: rgba(255, 255, 255, 0.7);
        transition: color 0.2s;
      }

      .social-links a:hover {
        color: white;
      }

      .payment-methods {
        display: flex;
        gap: 1rem;
        align-items: center;
      }
    `,
  ],
})
export class FooterComponent {}
```

## ☁️ Automatización CDN

### Script de Deploy con Validación y Optimización

```javascript
// scripts/deploy-icons.mjs
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";
import {
  CloudFrontClient,
  CreateInvalidationCommand,
} from "@aws-sdk/client-cloudfront";
import { readFileSync, readdirSync, existsSync } from "fs";
import { gzipSync } from "zlib";
import { minify } from "terser";

const s3 = new S3Client({
  region: "auto",
  endpoint: `https://${process.env.CLOUDFLARE_ACCOUNT_ID}.r2.cloudflarestorage.com`,
  credentials: {
    accessKeyId: process.env.CLOUDFLARE_ACCESS_KEY_ID,
    secretAccessKey: process.env.CLOUDFLARE_SECRET_ACCESS_KEY,
  },
});

const cloudfront = new CloudFrontClient({
  region: "us-east-1",
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
  },
});

async function validateSprites() {
  const spritesDir = "dist/sprites";

  if (!existsSync(spritesDir)) {
    throw new Error(
      "❌ Sprites directory not found. Run npm run icons:generate first."
    );
  }

  const sprites = readdirSync(spritesDir).filter((f) => f.endsWith(".svg"));

  if (sprites.length === 0) {
    throw new Error("❌ No sprite files found.");
  }

  // Validar tamaño de archivos
  const warnings = [];
  for (const sprite of sprites) {
    const content = readFileSync(`${spritesDir}/${sprite}`);
    const size = Buffer.byteLength(content);

    if (size > 500000) {
      // 500KB
      warnings.push(
        `⚠️  ${sprite} is large (${(size / 1024).toFixed(
          1
        )}KB). Consider splitting.`
      );
    }
  }

  if (warnings.length > 0) {
    console.log("\n⚠️  Warnings:");
    warnings.forEach((w) => console.log(w));
    console.log("");
  }

  return sprites;
}

async function deploySprites() {
  console.log("🚀 Starting deployment to Cloudflare CDN...\n");

  try {
    const sprites = await validateSprites();
    const manifest = JSON.parse(
      readFileSync("dist/sprites/manifest.json", "utf-8")
    );
    const deployedFiles = [];

    for (const sprite of sprites) {
      console.log(`📤 Deploying ${sprite}...`);

      const content = readFileSync(`dist/sprites/${sprite}`);
      const gzipped = gzipSync(content, { level: 9 });

      // Headers comunes
      const headers = {
        "Cache-Control": "public, max-age=31536000, immutable",
        "Content-Type": "image/svg+xml",
        "X-Content-Type-Options": "nosniff",
      };

      // Upload original
      await s3.send(
        new PutObjectCommand({
          Bucket: "icons-cdn",
          Key: sprite,
          Body: content,
          ContentType: headers["Content-Type"],
          CacheControl: headers["Cache-Control"],
          Metadata: {
            "deployment-date": new Date().toISOString(),
            "sprite-version": manifest[sprite.split("-")[1]]?.hash || "unknown",
            "original-size": content.length.toString(),
          },
        })
      );

      // Upload versión gzipped
      await s3.send(
        new PutObjectCommand({
          Bucket: "icons-cdn",
          Key: `${sprite}.gz`,
          Body: gzipped,
          ContentType: headers["Content-Type"],
          CacheControl: headers["Cache-Control"],
          ContentEncoding: "gzip",
          Metadata: {
            "compressed-size": gzipped.length.toString(),
          },
        })
      );

      // Upload versión Brotli si está disponible
      try {
        const { brotliCompressSync } = await import("zlib");
        const brotli = brotliCompressSync(content, {
          params: {
            [constants.BROTLI_PARAM_QUALITY]: constants.BROTLI_MAX_QUALITY,
          },
        });

        await s3.send(
          new PutObjectCommand({
            Bucket: "icons-cdn",
            Key: `${sprite}.br`,
            Body: brotli,
            ContentType: headers["Content-Type"],
            CacheControl: headers["Cache-Control"],
            ContentEncoding: "br",
          })
        );

        console.log(
          `✅ ${sprite} deployed (${(content.length / 1024).toFixed(1)}KB → ${(
            gzipped.length / 1024
          ).toFixed(1)}KB gz → ${(brotli.length / 1024).toFixed(1)}KB br)`
        );
      } catch (e) {
        console.log(
          `✅ ${sprite} deployed (${(content.length / 1024).toFixed(1)}KB → ${(
            gzipped.length / 1024
          ).toFixed(1)}KB gz)`
        );
      }

      deployedFiles.push(sprite);
    }

    // Upload manifest con cache más corto
    await s3.send(
      new PutObjectCommand({
        Bucket: "icons-cdn",
        Key: "manifest.json",
        Body: JSON.stringify(manifest, null, 2),
        ContentType: "application/json",
        CacheControl: "public, max-age=300, s-maxage=300", // 5 minutos
      })
    );

    console.log("📄 Manifest uploaded");

    // Invalidar caché de CloudFront si está configurado
    if (process.env.CLOUDFRONT_DISTRIBUTION_ID) {
      console.log("\n🔄 Invalidating CloudFront cache...");

      await cloudfront.send(
        new CreateInvalidationCommand({
          DistributionId: process.env.CLOUDFRONT_DISTRIBUTION_ID,
          InvalidationBatch: {
            CallerReference: Date.now().toString(),
            Paths: {
              Quantity: deployedFiles.length + 1,
              Items: [...deployedFiles.map((f) => `/${f}`), "/manifest.json"],
            },
          },
        })
      );

      console.log("✅ CloudFront cache invalidated");
    }

    console.log(
      `\n✨ Deployment complete! ${deployedFiles.length} sprites deployed to CDN.`
    );
    console.log(`📊 CDN URL: https://cdn.miempresa.com/icons/`);
  } catch (error) {
    console.error("\n❌ Deployment failed:", error.message);
    process.exit(1);
  }
}

// Ejecutar
deploySprites();
```

### GitHub Actions CI/CD

```yaml
# .github/workflows/deploy-icons.yml
name: Deploy Icons to CDN

on:
  push:
    paths:
      - "libs/icons/src/assets/**"
      - "libs/icons/tools/**"
      - "scripts/deploy-icons.mjs"
  workflow_dispatch:
    inputs:
      environment:
        description: "Deployment environment"
        required: true
        default: "production"
        type: choice
        options:
          - production
          - staging

jobs:
  validate:
    name: Validate Icons
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

      - name: Validate SVG files
        run: |
          npm install -g svglint
          echo "Validating SVG syntax..."
          svglint 'libs/icons/src/assets/**/*.svg' --config .svglint.yml || true

      - name: Check icon naming conventions
        run: |
          echo "Checking naming conventions..."
          find libs/icons/src/assets -name "*.svg" | while read file; do
            basename=$(basename "$file")
            if [[ "$basename" =~ [A-Z] ]]; then
              echo "❌ ERROR: $file contains uppercase letters"
              exit 1
            fi
            if [[ ! "$basename" =~ ^[a-z0-9-]+\.svg$ ]]; then
              echo "❌ ERROR: $file has invalid characters"
              exit 1
            fi
          done
          echo "✅ All icons follow naming conventions"

      - name: Run custom validation
        run: node libs/icons/tools/validate-icons.mjs

  build-and-test:
    name: Build Sprites & Test
    needs: validate
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

      - name: Generate sprites
        run: |
          echo "🎨 Generating sprite files..."
          node libs/icons/tools/generate-sprites.mjs

      - name: Generate TypeScript types
        run: |
          echo "📝 Generating TypeScript types..."
          node libs/icons/tools/generate-types.mjs

      - name: Run tests
        run: |
          echo "🧪 Running icon component tests..."
          npm run test:icons -- --coverage

      - name: Check sprite sizes
        run: |
          echo "📊 Sprite file sizes:"
          ls -lah dist/sprites/*.svg

      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: sprite-files
          path: dist/sprites/
          retention-days: 7

  deploy:
    name: Deploy to CDN
    needs: build-and-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' || github.event_name == 'workflow_dispatch'
    environment:
      name: ${{ github.event.inputs.environment || 'production' }}
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Download artifacts
        uses: actions/download-artifact@v3
        with:
          name: sprite-files
          path: dist/sprites/

      - name: Deploy to Cloudflare R2
        run: node scripts/deploy-icons.mjs
        env:
          CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          CLOUDFLARE_ACCESS_KEY_ID: ${{ secrets.CLOUDFLARE_ACCESS_KEY_ID }}
          CLOUDFLARE_SECRET_ACCESS_KEY: ${{ secrets.CLOUDFLARE_SECRET_ACCESS_KEY }}
          CLOUDFRONT_DISTRIBUTION_ID: ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      - name: Notify deployment
        if: success()
        run: |
          echo "✅ Icons deployed successfully to CDN!"
          echo "📊 View sprites at: https://cdn.miempresa.com/icons/"

      - name: Create deployment annotation
        if: success()
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.repos.createDeployment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              ref: context.sha,
              environment: '${{ github.event.inputs.environment || 'production' }}',
              description: 'Icon sprites deployed to CDN',
              auto_merge: false,
              required_contexts: []
            });
```

## 🎯 Setup en la Aplicación

### Provider Configuration

```typescript
// main.ts
import { bootstrapApplication } from "@angular/platform-browser";
import { provideIcons } from "@acme/icons";
import { AppComponent } from "./app/app.component";

bootstrapApplication(AppComponent, {
  providers: [
    provideIcons({
      cdnUrl: environment.production
        ? "https://cdn.miempresa.com/icons"
        : "/assets/icons",
      fallbackEnabled: !environment.production,
      preloadStrategies: [{ section: "core", priority: "high" }],
    }),
    // otros providers...
  ],
});
```

### Provider Implementation

```typescript
// libs/icons/src/lib/providers/icon.providers.ts
import { Provider, APP_INITIALIZER, inject } from "@angular/core";
import { DOCUMENT } from "@angular/common";
import {
  ICON_CONFIG,
  IconConfig,
  defaultIconConfig,
} from "../config/icon.config";

export function provideIcons(config?: Partial<IconConfig>): Provider[] {
  return [
    {
      provide: ICON_CONFIG,
      useValue: { ...defaultIconConfig, ...config },
    },
    {
      provide: APP_INITIALIZER,
      useFactory: () => {
        const iconConfig = inject(ICON_CONFIG);
        const document = inject(DOCUMENT);

        return () => {
          // Preload sprites de alta prioridad
          if (iconConfig.preloadStrategies && typeof window !== "undefined") {
            iconConfig.preloadStrategies
              .filter((s) => s.priority === "high")
              .forEach((strategy) => {
                const link = document.createElement("link");
                link.rel = "preload";
                link.as = "image";
                link.href = `${iconConfig.cdnUrl}/sprite-${strategy.section}.svg`;
                link.type = "image/svg+xml";
                link.crossOrigin = "anonymous";
                document.head.appendChild(link);
              });
          }
        };
      },
      multi: true,
    },
  ];
}
```

### Barrel Exports

```typescript
// libs/icons/src/index.ts
// Componentes
export { IconComponent } from "./lib/components/icon.component";

// Configuración
export { ICON_CONFIG, defaultIconConfig } from "./lib/config/icon.config";
export { provideIcons } from "./lib/providers/icon.providers";

// Tipos
export type {
  IconName,
  SpriteSection,
  CoreIcons,
  HomeIcons,
  AdminIcons,
  CmsIcons,
} from "./lib/types/icon.types";

// Constantes
export { ICON_SECTIONS } from "./lib/types/icon.types";
```

## 📋 Comandos y Scripts

### Package.json Scripts

```json
{
  "scripts": {
    "icons:generate": "node libs/icons/tools/generate-sprites.mjs",
    "icons:types": "node libs/icons/tools/generate-types.mjs",
    "icons:validate": "node libs/icons/tools/validate-icons.mjs",
    "icons:preview": "node libs/icons/tools/preview-icons.mjs",
    "icons:build": "npm run icons:validate && npm run icons:generate && npm run icons:types",
    "icons:deploy": "npm run icons:build && node scripts/deploy-icons.mjs",
    "icons:deploy:staging": "ENVIRONMENT=staging npm run icons:deploy",
    "test:icons": "nx test icons --watch=false"
  }
}
```

### Comandos NX

```bash
# Generar sprites localmente
nx run icons:generate

# Validar iconos
nx run icons:validate

# Generar tipos TypeScript
nx run icons:types

# Build completo (validar + generar + tipos)
nx run icons:build

# Desplegar a CDN
nx run icons:deploy

# Preview de iconos disponibles
nx run icons:preview

# Ejecutar tests
nx test icons

# Ver dependencias
nx graph --focus=icons
```

## 📊 Métricas y Beneficios

### ✅ Beneficios Conseguidos

1. **Lighthouse 100/100**

   - FCP < 1s: Iconos críticos visibles inmediatamente
   - LCP < 2.5s: Sin bloqueos por JavaScript
   - CLS = 0: Dimensiones definidas en todos los SVGs
   - TBT mínimo: Menos JavaScript en el bundle principal

2. **Rendimiento Optimizado**

   - Bundle inicial reducido: Solo HTML con SVGs críticos inline
   - Lazy loading inteligente: Sprites cargados on-demand
   - Compresión eficiente: Gzip + Brotli en CDN
   - Cache inmutable: Sprites con hash para cache eterno

3. **Developer Experience**

   - Type safety completo: Autocompletado de nombres de iconos
   - Validación en build time: Errores detectados antes del deploy
   - Preview visual: Herramienta para ver todos los iconos disponibles
   - Documentación inline: Referencias claras para iconos críticos

4. **Escalabilidad**

   - Fácil agregar nuevos iconos sin impacto en bundle
   - Sprites organizados por contexto/sección
   - CI/CD automatizado: De commit a CDN en minutos
   - Versionado con hash: Actualizaciones sin problemas de cache

5. **SSR Compatible**
   - Iconos críticos renderizados en servidor
   - No hay FOUC (Flash of Unstyled Content)
   - SEO optimizado: SVGs accesibles desde el HTML inicial

### 📈 Métricas de Rendimiento Esperadas

| Métrica         | Antes | Después | Mejora      |
| --------------- | ----- | ------- | ----------- |
| FCP             | 2.1s  | 0.8s    | **62%** ⬇️  |
| LCP             | 3.2s  | 1.9s    | **41%** ⬇️  |
| CLS             | 0.05  | 0       | **100%** ⬇️ |
| Bundle Size     | 320KB | 245KB   | **23%** ⬇️  |
| Icons Load Time | 450ms | 120ms   | **73%** ⬇️  |

## 🔧 Herramientas de Desarrollo

### Preview de Iconos

```javascript
// libs/icons/tools/preview-icons.mjs
import { readFileSync, readdirSync, writeFileSync } from "fs";
import { join } from "path";
import { createServer } from "http";
import open from "open";

function generatePreviewHTML() {
  const sections = ["core", "home", "admin", "cms"];
  let html = `<!DOCTYPE html>
<html>
<head>
  <title>Icon Preview - ACME Icons</title>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      padding: 2rem;
      background: #f5f5f5;
    }
    .header {
      text-align: center;
      margin-bottom: 3rem;
    }
    .section {
      background: white;
      border-radius: 8px;
      padding: 2rem;
      margin-bottom: 2rem;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
    .section h2 {
      margin-bottom: 1.5rem;
      color: #333;
      border-bottom: 2px solid #eee;
      padding-bottom: 0.5rem;
    }
    .icon-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
      gap: 1.5rem;
    }
    .icon-item {
      text-align: center;
      padding: 1rem;
      border-radius: 4px;
      transition: all 0.2s;
      cursor: pointer;
    }
    .icon-item:hover {
      background: #f0f0f0;
      transform: translateY(-2px);
    }
    .icon-item svg {
      width: 48px;
      height: 48px;
      margin-bottom: 0.5rem;
      color: #667eea;
    }
    .icon-name {
      font-size: 0.875rem;
      color: #666;
      word-break: break-all;
    }
    .critical-section {
      background: #fff3cd;
      border: 1px solid #ffeaa7;
    }
    .critical-section h2 {
      color: #856404;
    }
    .stats {
      display: flex;
      justify-content: center;
      gap: 2rem;
      margin-bottom: 2rem;
    }
    .stat {
      text-align: center;
    }
    .stat-value {
      font-size: 2rem;
      font-weight: bold;
      color: #667eea;
    }
    .stat-label {
      color: #666;
      font-size: 0.875rem;
    }
    .search {
      margin-bottom: 2rem;
      text-align: center;
    }
    .search input {
      padding: 0.75rem 1.5rem;
      font-size: 1rem;
      border: 2px solid #e2e8f0;
      border-radius: 4px;
      width: 300px;
    }
    .search input:focus {
      outline: none;
      border-color: #667eea;
    }
    .copy-notification {
      position: fixed;
      bottom: 2rem;
      right: 2rem;
      background: #48bb78;
      color: white;
      padding: 1rem 2rem;
      border-radius: 4px;
      transform: translateY(100px);
      transition: transform 0.3s;
    }
    .copy-notification.show {
      transform: translateY(0);
    }
  </style>
</head>
<body>
  <div class="header">
    <h1>🎨 ACME Icon System</h1>
    <p>Click any icon to copy its name to clipboard</p>
  </div>
  
  <div class="stats">
    <div class="stat">
      <div class="stat-value" id="totalIcons">0</div>
      <div class="stat-label">Total Icons</div>
    </div>
    <div class="stat">
      <div class="stat-value" id="totalSections">0</div>
      <div class="stat-label">Sections</div>
    </div>
  </div>
  
  <div class="search">
    <input 
      type="text" 
      id="searchInput" 
      placeholder="Search icons..." 
      onkeyup="filterIcons()"
    >
  </div>
  
  <!-- Critical Icons Reference -->
  <div class="section critical-section">
    <h2>⚡ Critical Icons (Inline in Components)</h2>
    <div class="icon-grid">
      <div class="icon-item" onclick="copyToClipboard('logo')">
        <svg viewBox="0 0 120 40">
          <path d="M12 0L24 12L12 24L0 12Z" fill="currentColor"/>
          <text x="30" y="28" font-family="Arial, sans-serif" font-size="24" font-weight="bold" fill="currentColor">ACME</text>
        </svg>
        <div class="icon-name">logo</div>
      </div>
      <div class="icon-item" onclick="copyToClipboard('menu')">
        <svg viewBox="0 0 24 24">
          <path d="M3 12h18M3 6h18M3 18h18" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
        </svg>
        <div class="icon-name">menu</div>
      </div>
      <div class="icon-item" onclick="copyToClipboard('search')">
        <svg viewBox="0 0 24 24">
          <circle cx="11" cy="11" r="8" stroke="currentColor" stroke-width="2" fill="none"/>
          <path d="M21 21l-4.35-4.35" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
        </svg>
        <div class="icon-name">search</div>
      </div>
      <div class="icon-item" onclick="copyToClipboard('cart')">
        <svg viewBox="0 0 24 24">
          <path d="M7 4V2C7 1.45 7.45 1 8 1H16C16.55 1 17 1.45 17 2V4H20C20.55 4 21 4.45 21 5S20.55 6 20 6H19V19C19 20.1 18.1 21 17 21H7C5.9 21 5 20.1 5 19V6H4C3.45 6 3 5.55 3 5S3.45 4 4 4H7Z" fill="currentColor"/>
        </svg>
        <div class="icon-name">cart</div>
      </div>
    </div>
  </div>
`;

  let totalIcons = 4; // Critical icons
  let totalSections = 1; // Critical section

  // Generate sections for sprite icons
  for (const section of sections) {
    const path = join("libs/icons/src/assets/individual", section);
    try {
      const icons = readdirSync(path)
        .filter((f) => f.endsWith(".svg"))
        .sort();

      if (icons.length === 0) continue;

      totalSections++;
      totalIcons += icons.length;

      html += `
  <div class="section" data-section="${section}">
    <h2>${section.charAt(0).toUpperCase() + section.slice(1)} Icons (${
        icons.length
      })</h2>
    <div class="icon-grid">`;

      for (const icon of icons) {
        const iconName = icon.replace(".svg", "");
        const svgContent = readFileSync(join(path, icon), "utf-8");

        html += `
      <div class="icon-item" onclick="copyToClipboard('${iconName}')" data-icon="${iconName}">
        ${svgContent}
        <div class="icon-name">${iconName}</div>
      </div>`;
      }

      html += `
    </div>
  </div>`;
    } catch (e) {
      console.warn(`Skipping section ${section}: ${e.message}`);
    }
  }

  html += `
  <div class="copy-notification" id="copyNotification">
    Icon name copied to clipboard!
  </div>
  
  <script>
    // Update stats
    document.getElementById('totalIcons').textContent = ${totalIcons};
    document.getElementById('totalSections').textContent = ${totalSections};
    
    function copyToClipboard(iconName) {
      navigator.clipboard.writeText(iconName).then(() => {
        const notification = document.getElementById('copyNotification');
        notification.textContent = \`"\${iconName}" copied to clipboard!\`;
        notification.classList.add('show');
        setTimeout(() => {
          notification.classList.remove('show');
        }, 2000);
      });
    }
    
    function filterIcons() {
      const searchTerm = document.getElementById('searchInput').value.toLowerCase();
      const iconItems = document.querySelectorAll('.icon-item');
      
      iconItems.forEach(item => {
        const iconName = item.getAttribute('data-icon') || item.querySelector('.icon-name').textContent;
        if (iconName.toLowerCase().includes(searchTerm)) {
          item.style.display = 'block';
        } else {
          item.style.display = 'none';
        }
      });
    }
    
    // Add keyboard shortcut for search (Cmd/Ctrl + K)
    document.addEventListener('keydown', (e) => {
      if ((e.metaKey || e.ctrlKey) && e.key === 'k') {
        e.preventDefault();
        document.getElementById('searchInput').focus();
      }
    });
  </script>
</body>
</html>`;

  return html;
}

// Generate and serve preview
const html = generatePreviewHTML();
writeFileSync("dist/icon-preview.html", html);

// Create simple HTTP server
const server = createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/html" });
  res.end(html);
});

const port = 4200;
server.listen(port, () => {
  console.log(`🎨 Icon preview server running at http://localhost:${port}`);
  console.log("📋 Click any icon to copy its name to clipboard");
  console.log("🔍 Press Ctrl/Cmd + K to focus search\n");

  // Open in browser
  open(`http://localhost:${port}`);
});
```

## 🏆 Best Practices y Recomendaciones

### 1. **Decisión de Iconos Críticos**

- Analiza tu aplicación con herramientas como Lighthouse
- Identifica qué iconos aparecen en el viewport inicial
- Monitorea las métricas de Core Web Vitals
- Revisa periódicamente basándote en analytics

### 2. **Optimización de SVGs**

- Usa herramientas como SVGO para optimizar
- Elimina metadatos innecesarios
- Simplifica paths cuando sea posible
- Mantén viewBox para escalabilidad

### 3. **Accesibilidad**

- Siempre incluye `aria-label` o `aria-labelledby`
- Usa `role="img"` en SVGs decorativos
- Proporciona texto alternativo significativo
- Asegura contraste adecuado

### 4. **Testing**

```typescript
// Ejemplo de test para el componente Icon
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
          useValue: { cdnUrl: "https://test.cdn.com/icons" },
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

  it("should apply custom classes", () => {
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
});
```

### 5. **Monitoreo y Análisis**

```typescript
// Servicio para tracking de rendimiento
import { Injectable } from "@angular/core";

@Injectable({ providedIn: "root" })
export class IconPerformanceService {
  private loadTimes = new Map<string, number>();

  trackIconLoad(section: string, startTime: number) {
    const loadTime = performance.now() - startTime;
    this.loadTimes.set(section, loadTime);

    // Enviar a analytics si es necesario
    if (typeof gtag !== "undefined") {
      gtag("event", "icon_sprite_load", {
        event_category: "performance",
        event_label: section,
        value: Math.round(loadTime),
      });
    }
  }

  getAverageLoadTime(): number {
    if (this.loadTimes.size === 0) return 0;

    const total = Array.from(this.loadTimes.values()).reduce(
      (sum, time) => sum + time,
      0
    );

    return total / this.loadTimes.size;
  }
}
```

## 🎯 Checklist de Implementación

- [ ] Identificar iconos críticos above-the-fold
- [ ] Mover iconos críticos a inline en componentes
- [ ] Configurar estructura de carpetas para sprites
- [ ] Implementar scripts de generación
- [ ] Configurar CDN (Cloudflare R2)
- [ ] Implementar componente Icon para sprites
- [ ] Agregar validación de iconos
- [ ] Configurar CI/CD pipeline
- [ ] Agregar tests unitarios
- [ ] Implementar preview de iconos
- [ ] Documentar proceso para el equipo
- [ ] Monitorear métricas de rendimiento
- [ ] Optimizar basándose en analytics

## 📚 Recursos Adicionales

- [SVGO Configuration](https://github.com/svg/svgo)
- [Cloudflare R2 Documentation](https://developers.cloudflare.com/r2/)
- [Angular Performance Best Practices](https://angular.io/guide/performance-best-practices)
- [Web.dev Icon Optimization](https://web.dev/optimize-cls/)
- [SVG Accessibility](https://css-tricks.com/accessible-svgs/)

---

**Última actualización**: Mayo 2024  
**Mantenido por**: Equipo de Frontend  
**Versión**: 2.0.0
