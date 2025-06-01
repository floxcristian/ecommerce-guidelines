# 🎨 Optimización Frontend Angular - Guía Completa

Guía detallada para optimizar aplicaciones Angular de ecommerce, incluyendo performance, SEO, UX, y mejores prácticas específicas para comercio electrónico.

## 🎯 Objetivos

- Performance superior (Core Web Vitals)
- SEO optimizado para ecommerce
- UX excepcional en desktop y mobile
- Código mantenible y escalable
- Conversión optimizada

## 🏗️ Arquitectura Frontend

### Estructura NX Monorepo

```bash
apps/
├── frontend/                    # App principal cliente
│   ├── src/
│   │   ├── app/
│   │   │   ├── core/           # Services, guards, interceptors
│   │   │   ├── shared/         # Componentes compartidos
│   │   │   ├── features/       # Módulos por feature
│   │   │   │   ├── home/
│   │   │   │   ├── products/
│   │   │   │   ├── cart/
│   │   │   │   ├── checkout/
│   │   │   │   └── account/
│   │   │   └── layout/         # Header, footer, etc
│   │   ├── assets/
│   │   └── environments/
│   └── project.json
│
├── admin/                      # Panel administrativo
│   └── src/...
│
└── mobile/                     # App móvil (Ionic/Capacitor)
    └── src/...

libs/
├── ui/                         # Design system
│   ├── components/
│   ├── directives/
│   └── pipes/
├── shared/
│   ├── data-access/           # Services, state management
│   ├── utils/
│   └── types/
└── feature/
    ├── auth/
    ├── products/
    └── orders/
```

## ⚡ Performance Optimization

### 1. Lazy Loading Estratégico

```typescript
// apps/frontend/src/app/app-routing.module.ts
const routes: Routes = [
  {
    path: "",
    loadChildren: () =>
      import("./features/home/home.module").then((m) => m.HomeModule),
    data: { preload: true }, // Precargar módulo crítico
  },
  {
    path: "products",
    loadChildren: () =>
      import("./features/products/products.module").then(
        (m) => m.ProductsModule
      ),
    data: { preload: true },
  },
  {
    path: "cart",
    loadChildren: () =>
      import("./features/cart/cart.module").then((m) => m.CartModule),
    data: { preload: false }, // Cargar bajo demanda
  },
  {
    path: "checkout",
    loadChildren: () =>
      import("./features/checkout/checkout.module").then(
        (m) => m.CheckoutModule
      ),
    data: { preload: false },
  },
  {
    path: "account",
    loadChildren: () =>
      import("./features/account/account.module").then((m) => m.AccountModule),
    data: { preload: false },
  },
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      preloadingStrategy: SelectivePreloadingStrategy,
      initialNavigation: "enabledBlocking",
    }),
  ],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

### 2. Preloading Strategy Personalizada

```typescript
// apps/frontend/src/app/core/strategies/selective-preloading.strategy.ts
@Injectable()
export class SelectivePreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    if (route.data && route.data["preload"]) {
      // Precargar después de un delay para no bloquear la navegación inicial
      return timer(2000).pipe(switchMap(() => load()));
    }
    return of(null);
  }
}
```

### 3. OnPush Change Detection

```typescript
// libs/ui/components/product-card/product-card.component.ts
@Component({
  selector: "app-product-card",
  templateUrl: "./product-card.component.html",
  styleUrls: ["./product-card.component.scss"],
  changeDetection: ChangeDetectionStrategy.OnPush, // Optimización crítica
})
export class ProductCardComponent {
  @Input() product!: Product;
  @Output() addToCart = new EventEmitter<Product>();

  constructor(private cdr: ChangeDetectorRef) {}

  onAddToCart(): void {
    this.addToCart.emit(this.product);
    // Marcar para detección de cambios solo cuando sea necesario
    this.cdr.markForCheck();
  }

  trackByProductId(index: number, product: Product): string {
    return product.id; // TrackBy para ngFor performance
  }
}
```

### 4. Virtual Scrolling para Listas Grandes

```typescript
// apps/frontend/src/app/features/products/components/product-list.component.ts
@Component({
  selector: "app-product-list",
  template: `
    <cdk-virtual-scroll-viewport itemSize="300" class="product-viewport">
      <app-product-card
        *cdkVirtualFor="let product of products$ | async; trackBy: trackByFn"
        [product]="product"
        (addToCart)="onAddToCart($event)"
      >
      </app-product-card>
    </cdk-virtual-scroll-viewport>
  `,
  styleUrls: ["./product-list.component.scss"],
})
export class ProductListComponent {
  products$ = this.store.select(selectProducts);

  constructor(private store: Store) {}

  trackByFn(index: number, product: Product): string {
    return product.id;
  }

  onAddToCart(product: Product): void {
    this.store.dispatch(addToCart({ product }));
  }
}
```

### 5. Image Optimization

```typescript
// libs/ui/components/optimized-image/optimized-image.component.ts
@Component({
  selector: "app-optimized-image",
  template: `
    <picture>
      <source [srcset]="webpSrcset" type="image/webp" [media]="mediaQuery" />
      <source [srcset]="srcset" [media]="mediaQuery" />
      <img
        [src]="src"
        [alt]="alt"
        [loading]="loading"
        [width]="width"
        [height]="height"
        (load)="onImageLoad()"
        (error)="onImageError()"
      />
    </picture>
  `,
})
export class OptimizedImageComponent {
  @Input() src!: string;
  @Input() alt!: string;
  @Input() width?: number;
  @Input() height?: number;
  @Input() loading: "lazy" | "eager" = "lazy";
  @Input() sizes?: string;

  get srcset(): string {
    return this.generateSrcset(this.src);
  }

  get webpSrcset(): string {
    return this.generateSrcset(this.src, "webp");
  }

  get mediaQuery(): string {
    return this.sizes || "(max-width: 768px) 100vw, 50vw";
  }

  private generateSrcset(baseSrc: string, format?: string): string {
    const sizes = [480, 768, 1024, 1200];
    const extension = format || baseSrc.split(".").pop();

    return sizes
      .map(
        (size) =>
          `${baseSrc.replace(/\.[^/.]+$/, "")}_${size}w.${extension} ${size}w`
      )
      .join(", ");
  }

  onImageLoad(): void {
    // Analytics tracking
  }

  onImageError(): void {
    // Fallback image
  }
}
```

## 🛒 Estado de la Aplicación con NgRx

### 1. Store Structure

```typescript
// libs/shared/data-access/src/lib/store/app.state.ts
export interface AppState {
  auth: AuthState;
  products: ProductsState;
  cart: CartState;
  orders: OrdersState;
  ui: UiState;
}

// Feature States
export interface ProductsState {
  products: Product[];
  categories: Category[];
  filters: ProductFilters;
  pagination: Pagination;
  loading: boolean;
  error: string | null;
}

export interface CartState {
  items: CartItem[];
  total: number;
  shipping: ShippingInfo | null;
  discounts: Discount[];
  loading: boolean;
}
```

### 2. Effects para API Calls

```typescript
// libs/shared/data-access/src/lib/store/products/products.effects.ts
@Injectable()
export class ProductsEffects {
  loadProducts$ = createEffect(() =>
    this.actions$.pipe(
      ofType(ProductsActions.loadProducts),
      debounceTime(300), // Debounce búsquedas
      distinctUntilChanged(),
      switchMap(({ filters, pagination }) =>
        this.productsService.getProducts(filters, pagination).pipe(
          map((response) =>
            ProductsActions.loadProductsSuccess({
              products: response.products,
              pagination: response.pagination,
            })
          ),
          catchError((error) =>
            of(ProductsActions.loadProductsFailure({ error }))
          )
        )
      )
    )
  );

  addToCart$ = createEffect(() =>
    this.actions$.pipe(
      ofType(CartActions.addToCart),
      concatMap(({ product }) =>
        this.cartService.addItem(product).pipe(
          map(() => CartActions.addToCartSuccess({ product })),
          catchError((error) => of(CartActions.addToCartFailure({ error })))
        )
      )
    )
  );

  constructor(
    private actions$: Actions,
    private productsService: ProductsService,
    private cartService: CartService
  ) {}
}
```

### 3. Selectors Optimizados

```typescript
// libs/shared/data-access/src/lib/store/products/products.selectors.ts
export const selectProductsState =
  createFeatureSelector<ProductsState>("products");

export const selectProducts = createSelector(
  selectProductsState,
  (state: ProductsState) => state.products
);

export const selectProductsByCategory = createSelector(
  selectProducts,
  (products: Product[], props: { categoryId: string }) =>
    products.filter((product) => product.categoryId === props.categoryId)
);

// Selector memoizado para productos filtrados
export const selectFilteredProducts = createSelector(
  selectProducts,
  selectProductsState,
  (products: Product[], state: ProductsState) => {
    const { filters } = state;

    return products.filter((product) => {
      if (filters.category && product.categoryId !== filters.category)
        return false;
      if (filters.minPrice && product.price < filters.minPrice) return false;
      if (filters.maxPrice && product.price > filters.maxPrice) return false;
      if (filters.inStock && !product.inStock) return false;

      return true;
    });
  }
);

// Selector para carrito con totales calculados
export const selectCartWithTotals = createSelector(
  selectCartState,
  (cartState: CartState) => ({
    ...cartState,
    subtotal: cartState.items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    ),
    tax: cartState.items.reduce(
      (sum, item) => sum + item.price * item.quantity * 0.1,
      0
    ),
    total: cartState.total,
  })
);
```

## 🎨 Design System y UI

### 1. Design Tokens

```scss
// libs/ui/src/lib/tokens/_colors.scss
:root {
  // Primary Brand Colors
  --color-primary-50: #f0f9ff;
  --color-primary-100: #e0f2fe;
  --color-primary-500: #0ea5e9;
  --color-primary-600: #0284c7;
  --color-primary-900: #0c4a6e;

  // Semantic Colors
  --color-success: #22c55e;
  --color-warning: #f59e0b;
  --color-error: #ef4444;
  --color-info: #3b82f6;

  // Neutral Colors
  --color-gray-50: #f9fafb;
  --color-gray-100: #f3f4f6;
  --color-gray-500: #6b7280;
  --color-gray-900: #111827;

  // E-commerce Specific
  --color-price: #059669;
  --color-discount: #dc2626;
  --color-out-of-stock: #6b7280;
}
```

### 2. Componente Button Reutilizable

```typescript
// libs/ui/components/button/button.component.ts
@Component({
  selector: "app-button",
  template: `
    <button
      [class]="buttonClasses"
      [disabled]="disabled || loading"
      [attr.aria-label]="ariaLabel"
      (click)="onClick($event)"
    >
      <app-loading-spinner *ngIf="loading" [size]="spinnerSize">
      </app-loading-spinner>

      <ng-container *ngIf="!loading">
        <ng-content></ng-content>
      </ng-container>
    </button>
  `,
  styleUrls: ["./button.component.scss"],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ButtonComponent {
  @Input() variant: "primary" | "secondary" | "outline" | "ghost" = "primary";
  @Input() size: "sm" | "md" | "lg" = "md";
  @Input() disabled = false;
  @Input() loading = false;
  @Input() fullWidth = false;
  @Input() ariaLabel?: string;

  @Output() clicked = new EventEmitter<Event>();

  get buttonClasses(): string {
    return [
      "btn",
      `btn--${this.variant}`,
      `btn--${this.size}`,
      this.fullWidth ? "btn--full-width" : "",
      this.loading ? "btn--loading" : "",
      this.disabled ? "btn--disabled" : "",
    ]
      .filter(Boolean)
      .join(" ");
  }

  get spinnerSize(): "sm" | "md" {
    return this.size === "lg" ? "md" : "sm";
  }

  onClick(event: Event): void {
    if (!this.disabled && !this.loading) {
      this.clicked.emit(event);
    }
  }
}
```

### 3. Componente Product Card Optimizado

```typescript
// libs/ui/components/product-card/product-card.component.ts
@Component({
  selector: "app-product-card",
  template: `
    <article class="product-card" [attr.data-product-id]="product.id">
      <div class="product-card__image">
        <app-optimized-image
          [src]="product.image"
          [alt]="product.name"
          loading="lazy"
          [width]="280"
          [height]="280"
        >
        </app-optimized-image>

        <div class="product-card__badges">
          <span *ngIf="product.isNew" class="badge badge--new">Nuevo</span>
          <span *ngIf="product.discount > 0" class="badge badge--discount">
            -{{ product.discount }}%
          </span>
        </div>

        <button
          class="product-card__quick-view"
          (click)="onQuickView()"
          aria-label="Vista rápida de {{ product.name }}"
        >
          <app-icon name="eye"></app-icon>
        </button>
      </div>

      <div class="product-card__content">
        <h3 class="product-card__title">
          <a [routerLink]="['/products', product.slug]">{{ product.name }}</a>
        </h3>

        <div class="product-card__rating">
          <app-star-rating
            [rating]="product.rating"
            [reviewCount]="product.reviewCount"
          >
          </app-star-rating>
        </div>

        <div class="product-card__price">
          <span class="price price--current">{{
            product.price | currency
          }}</span>
          <span *ngIf="product.originalPrice" class="price price--original">
            {{ product.originalPrice | currency }}
          </span>
        </div>

        <app-button
          variant="primary"
          [fullWidth]="true"
          [loading]="addingToCart"
          [disabled]="!product.inStock"
          (clicked)="onAddToCart()"
        >
          {{ product.inStock ? "Agregar al carrito" : "Sin stock" }}
        </app-button>
      </div>
    </article>
  `,
  styleUrls: ["./product-card.component.scss"],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ProductCardComponent {
  @Input() product!: Product;
  @Input() addingToCart = false;

  @Output() addToCart = new EventEmitter<Product>();
  @Output() quickView = new EventEmitter<Product>();

  onAddToCart(): void {
    if (this.product.inStock) {
      this.addToCart.emit(this.product);
    }
  }

  onQuickView(): void {
    this.quickView.emit(this.product);
  }
}
```

## 🔍 SEO y Meta Tags

### 1. SEO Service

```typescript
// libs/shared/data-access/src/lib/services/seo.service.ts
@Injectable({
  providedIn: "root",
})
export class SeoService {
  constructor(
    private title: Title,
    private meta: Meta,
    @Inject(DOCUMENT) private document: Document
  ) {}

  setProductSEO(product: Product): void {
    const title = `${product.name} - Tu Tienda Online`;
    const description = product.description.substring(0, 155);
    const image = product.images[0];
    const url = `${environment.baseUrl}/products/${product.slug}`;

    // Title
    this.title.setTitle(title);

    // Meta tags básicos
    this.meta.updateTag({ name: "description", content: description });
    this.meta.updateTag({ name: "keywords", content: product.tags.join(", ") });

    // Open Graph
    this.meta.updateTag({ property: "og:title", content: title });
    this.meta.updateTag({ property: "og:description", content: description });
    this.meta.updateTag({ property: "og:image", content: image });
    this.meta.updateTag({ property: "og:url", content: url });
    this.meta.updateTag({ property: "og:type", content: "product" });

    // Twitter Cards
    this.meta.updateTag({
      name: "twitter:card",
      content: "summary_large_image",
    });
    this.meta.updateTag({ name: "twitter:title", content: title });
    this.meta.updateTag({ name: "twitter:description", content: description });
    this.meta.updateTag({ name: "twitter:image", content: image });

    // Product específicos
    this.meta.updateTag({
      property: "product:price:amount",
      content: product.price.toString(),
    });
    this.meta.updateTag({ property: "product:price:currency", content: "CLP" });
    this.meta.updateTag({
      property: "product:availability",
      content: product.inStock ? "in stock" : "out of stock",
    });

    // Structured Data
    this.setProductStructuredData(product);
  }

  private setProductStructuredData(product: Product): void {
    const structuredData = {
      "@context": "https://schema.org",
      "@type": "Product",
      name: product.name,
      description: product.description,
      image: product.images,
      sku: product.sku,
      brand: {
        "@type": "Brand",
        name: product.brand,
      },
      offers: {
        "@type": "Offer",
        price: product.price,
        priceCurrency: "CLP",
        availability: product.inStock
          ? "https://schema.org/InStock"
          : "https://schema.org/OutOfStock",
      },
      aggregateRating: {
        "@type": "AggregateRating",
        ratingValue: product.rating,
        reviewCount: product.reviewCount,
      },
    };

    this.insertStructuredData(structuredData);
  }

  private insertStructuredData(data: any): void {
    const script = this.document.createElement("script");
    script.type = "application/ld+json";
    script.text = JSON.stringify(data);
    this.document.head.appendChild(script);
  }
}
```

### 2. Resolver para SEO

```typescript
// apps/frontend/src/app/features/products/resolvers/product-seo.resolver.ts
@Injectable()
export class ProductSeoResolver implements Resolve<Product> {
  constructor(
    private productsService: ProductsService,
    private seoService: SeoService
  ) {}

  resolve(route: ActivatedRouteSnapshot): Observable<Product> {
    const slug = route.params["slug"];

    return this.productsService.getBySlug(slug).pipe(
      tap((product) => {
        if (product) {
          this.seoService.setProductSEO(product);
        }
      })
    );
  }
}
```

## 📱 PWA y Performance

### 1. Service Worker Personalizado

```typescript
// apps/frontend/src/app/core/services/pwa.service.ts
@Injectable({
  providedIn: "root",
})
export class PwaService {
  private promptEvent: any;

  constructor(private swUpdate: SwUpdate) {
    this.initPwa();
  }

  private initPwa(): void {
    // Detectar cuando hay una nueva versión disponible
    if (this.swUpdate.isEnabled) {
      this.swUpdate.available.subscribe(() => {
        if (confirm("Nueva versión disponible. ¿Quieres actualizar?")) {
          window.location.reload();
        }
      });
    }

    // Detectar evento de instalación PWA
    window.addEventListener("beforeinstallprompt", (e) => {
      e.preventDefault();
      this.promptEvent = e;
    });
  }

  installPwa(): void {
    if (this.promptEvent) {
      this.promptEvent.prompt();
      this.promptEvent.userChoice.then((result: any) => {
        this.promptEvent = null;
      });
    }
  }

  canInstall(): boolean {
    return !!this.promptEvent;
  }
}
```

### 2. Web Vitals Tracking

```typescript
// apps/frontend/src/app/core/services/analytics.service.ts
@Injectable({
  providedIn: "root",
})
export class AnalyticsService {
  constructor() {
    this.initWebVitals();
  }

  private initWebVitals(): void {
    import("web-vitals").then(({ getCLS, getFID, getFCP, getLCP, getTTFB }) => {
      getCLS(this.sendToAnalytics);
      getFID(this.sendToAnalytics);
      getFCP(this.sendToAnalytics);
      getLCP(this.sendToAnalytics);
      getTTFB(this.sendToAnalytics);
    });
  }

  private sendToAnalytics = (metric: any) => {
    // Enviar métricas a Google Analytics
    gtag("event", metric.name, {
      value: Math.round(
        metric.name === "CLS" ? metric.value * 1000 : metric.value
      ),
      event_category: "Web Vitals",
      event_label: metric.id,
      non_interaction: true,
    });
  };

  trackPurchase(order: Order): void {
    gtag("event", "purchase", {
      transaction_id: order.id,
      value: order.total,
      currency: "CLP",
      items: order.items.map((item) => ({
        item_id: item.productId,
        item_name: item.name,
        category: item.category,
        quantity: item.quantity,
        price: item.price,
      })),
    });
  }
}
```

## 🔧 Build y Deployment Optimization

### 1. Angular.json Optimizado

```json
{
  "projects": {
    "frontend": {
      "architect": {
        "build": {
          "configurations": {
            "production": {
              "budgets": [
                {
                  "type": "initial",
                  "maximumWarning": "2mb",
                  "maximumError": "5mb"
                },
                {
                  "type": "anyComponentStyle",
                  "maximumWarning": "6kb",
                  "maximumError": "10kb"
                }
              ],
              "outputHashing": "all",
              "optimization": true,
              "sourceMap": false,
              "extractCss": true,
              "namedChunks": false,
              "aot": true,
              "buildOptimizer": true,
              "vendorChunk": false,
              "commonChunk": false
            }
          }
        }
      }
    }
  }
}
```

### 2. Webpack Bundle Analyzer

```json
{
  "scripts": {
    "build:prod": "nx build frontend --prod",
    "build:analyze": "nx build frontend --prod --stats-json && npx webpack-bundle-analyzer dist/apps/frontend/stats.json"
  }
}
```

## 🔗 Próximos Pasos

### Enlaces Relacionados

- [🚀 Guía Completa Full-Stack](./new.md)
- [⚙️ Microservicios con NestJS](./backend-microservices.md)
- [🏗️ Infraestructura y Deployment](./infrastructure-deployment.md)

### Próximos Temas

- [ ] Testing avanzado (Unit, Integration, E2E)
- [ ] Accesibilidad (a11y) para ecommerce
- [ ] Internacionalización (i18n)
- [ ] Micro-frontends con Module Federation
- [ ] Mobile app con Ionic/Capacitor

---

> 💡 **Tip**: Mide siempre el performance antes y después de cada optimización. Usa herramientas como Lighthouse, WebPageTest y Core Web Vitals para validar mejoras reales.
