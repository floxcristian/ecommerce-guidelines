# 📊 Performance Monitoring - Monitoreo Avanzado Frontend

Guía completa para implementar monitoreo de rendimiento en aplicaciones ecommerce Angular con métricas RUM, alertas automáticas y dashboards de performance.

## 🎯 Objetivos del Monitoreo

- **RUM (Real User Monitoring)**: Métricas de usuarios reales vs sintéticas
- **Core Web Vitals**: FCP, LCP, CLS, FID en producción
- **Business Metrics**: Conversión, abandono de carrito, revenue
- **Alertas Proactivas**: Detección temprana de degradación
- **Performance Budget**: Prevención de regresiones

## 📈 Arquitectura de Monitoreo

El siguiente diagrama muestra cómo se integra el sistema de monitoreo de performance en una aplicación Angular ecommerce, desde la captura de métricas hasta las alertas automáticas.

```mermaid
graph TB
    A[Angular App<br/>Frontend] --> B[Web Vitals API<br/>Performance Observer]
    A --> C[Performance Observer<br/>Custom Metrics]
    A --> D[Business Events<br/>Ecommerce Analytics]

    B --> E[Analytics Service<br/>Aggregation Layer]
    C --> E
    D --> E

    E --> F[Google Analytics 4<br/>Enhanced Ecommerce]
    E --> G[DataDog RUM<br/>Real User Monitoring]
    E --> H[Custom API<br/>Internal Analytics]

    F --> I[Performance Dashboard<br/>Real-time Metrics]
    G --> J[Alerting System<br/>Automated Notifications]
    H --> K[Business Intelligence<br/>Decision Making]

    I --> L[Stakeholder Reports<br/>Executive Dashboard]
    J --> M[DevOps Notifications<br/>Slack/Email/PagerDuty]
    K --> N[Product Decisions<br/>Performance Budget]

    classDef frontend fill:#e1f5fe
    classDef analytics fill:#f3e5f5
    classDef external fill:#e8f5e8
    classDef reporting fill:#fff3e0
    classDef actions fill:#fce4ec

    class A,B,C,D frontend
    class E analytics
    class F,G,H external
    class I,K reporting
    class J,L,M,N actions
```

#### 📋 Descripción del Flujo de Monitoreo

**🎨 Frontend Data Collection (Azul)**

1. **Angular App**: Aplicación principal que genera métricas de performance
2. **Web Vitals API**: Captura automática de Core Web Vitals (LCP, FCP, CLS, FID)
3. **Performance Observer**: Monitoreo personalizado de recursos, navegación y componentes
4. **Business Events**: Tracking de eventos específicos del ecommerce (add to cart, purchase)

**⚙️ Analytics Processing (Púrpura)**

5. **Analytics Service**: Capa de agregación que procesa, filtra y enruta las métricas

**🌐 External Services (Verde)**

6. **Google Analytics 4**: Enhanced ecommerce tracking con métricas de performance
7. **DataDog RUM**: Real User Monitoring con dashboards avanzados
8. **Custom API**: API interna para métricas específicas del negocio

**📊 Reporting Layer (Amarillo)**

9. **Performance Dashboard**: Dashboard en tiempo real dentro de la aplicación
10. **Business Intelligence**: Análisis de impacto en métricas de negocio

**🚨 Action Layer (Rosa)**

11. **Alerting System**: Sistema automatizado de alertas basado en thresholds
12. **Stakeholder Reports**: Reportes ejecutivos y de producto
13. **DevOps Notifications**: Notificaciones inmediatas para el equipo técnico
14. **Product Decisions**: Información para toma de decisiones de producto

#### ⏱️ Flujo de Datos en Tiempo Real

- **Captura**: 10-50ms después del evento
- **Procesamiento**: Agregación cada 5 segundos
- **Alertas**: 1-2 segundos para alertas críticas
- **Dashboards**: Actualización cada 30 segundos
- **Reportes**: Generación diaria/semanal automática

## 🔧 Implementación Core

### 1. Servicio Base de Analytics

```typescript
// libs/frontend/analytics/src/lib/performance-analytics.service.ts
import { Injectable, Inject, PLATFORM_ID } from "@angular/core";
import { isPlatformBrowser } from "@angular/common";
import {
  BehaviorSubject,
  Observable,
  debounceTime,
  distinctUntilChanged,
} from "rxjs";

export interface PerformanceMetric {
  name: string;
  value: number;
  rating: "good" | "needs-improvement" | "poor";
  timestamp: number;
  url: string;
  userAgent: string;
  connection?: NetworkInformation;
  sessionId: string;
  userId?: string;
}

export interface BusinessMetric {
  event: string;
  category: "ecommerce" | "navigation" | "interaction";
  value?: number;
  metadata?: Record<string, any>;
  timestamp: number;
  sessionId: string;
  userId?: string;
}

@Injectable({
  providedIn: "root",
})
export class PerformanceAnalyticsService {
  private isBrowser: boolean;
  private sessionId: string;
  private userId?: string;

  private metricsSubject = new BehaviorSubject<PerformanceMetric[]>([]);
  private businessEventsSubject = new BehaviorSubject<BusinessMetric[]>([]);

  public metrics$ = this.metricsSubject.asObservable();
  public businessEvents$ = this.businessEventsSubject.asObservable();

  private performanceObserver?: PerformanceObserver;
  private navigationObserver?: PerformanceObserver;

  constructor(@Inject(PLATFORM_ID) platformId: Object) {
    this.isBrowser = isPlatformBrowser(platformId);
    this.sessionId = this.generateSessionId();

    if (this.isBrowser) {
      this.initializeMonitoring();
      this.setupPerformanceObservers();
      this.trackNavigationTiming();
    }
  }

  private generateSessionId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }

  private initializeMonitoring() {
    // Web Vitals monitoring
    this.setupWebVitals();

    // Custom performance monitoring
    this.setupCustomMetrics();

    // Error tracking
    this.setupErrorTracking();

    // Network monitoring
    this.setupNetworkMonitoring();
  }

  private setupWebVitals() {
    import("web-vitals").then(({ getCLS, getFCP, getFID, getLCP, getTTFB }) => {
      getCLS((metric) =>
        this.recordMetric({
          name: "CLS",
          value: metric.value,
          rating: this.getRating("CLS", metric.value),
          timestamp: Date.now(),
          url: window.location.href,
          userAgent: navigator.userAgent,
          connection: (navigator as any).connection,
          sessionId: this.sessionId,
          userId: this.userId,
        })
      );

      getFCP((metric) =>
        this.recordMetric({
          name: "FCP",
          value: metric.value,
          rating: this.getRating("FCP", metric.value),
          timestamp: Date.now(),
          url: window.location.href,
          userAgent: navigator.userAgent,
          connection: (navigator as any).connection,
          sessionId: this.sessionId,
          userId: this.userId,
        })
      );

      getFID((metric) =>
        this.recordMetric({
          name: "FID",
          value: metric.value,
          rating: this.getRating("FID", metric.value),
          timestamp: Date.now(),
          url: window.location.href,
          userAgent: navigator.userAgent,
          connection: (navigator as any).connection,
          sessionId: this.sessionId,
          userId: this.userId,
        })
      );

      getLCP((metric) =>
        this.recordMetric({
          name: "LCP",
          value: metric.value,
          rating: this.getRating("LCP", metric.value),
          timestamp: Date.now(),
          url: window.location.href,
          userAgent: navigator.userAgent,
          connection: (navigator as any).connection,
          sessionId: this.sessionId,
          userId: this.userId,
        })
      );

      getTTFB((metric) =>
        this.recordMetric({
          name: "TTFB",
          value: metric.value,
          rating: this.getRating("TTFB", metric.value),
          timestamp: Date.now(),
          url: window.location.href,
          userAgent: navigator.userAgent,
          connection: (navigator as any).connection,
          sessionId: this.sessionId,
          userId: this.userId,
        })
      );
    });
  }

  private setupCustomMetrics() {
    // Angular-specific metrics
    this.trackAngularBootstrap();
    this.trackRouteChanges();
    this.trackComponentMounts();
  }

  private setupPerformanceObservers() {
    if ("PerformanceObserver" in window) {
      // Navigation timing
      this.navigationObserver = new PerformanceObserver((list) => {
        list.getEntries().forEach((entry) => {
          if (entry.entryType === "navigation") {
            this.processNavigationEntry(entry as PerformanceNavigationTiming);
          }
        });
      });

      this.navigationObserver.observe({ entryTypes: ["navigation"] });

      // Resource timing
      this.performanceObserver = new PerformanceObserver((list) => {
        list.getEntries().forEach((entry) => {
          if (entry.entryType === "resource") {
            this.processResourceEntry(entry as PerformanceResourceTiming);
          }
        });
      });

      this.performanceObserver.observe({ entryTypes: ["resource"] });
    }
  }

  private processNavigationEntry(entry: PerformanceNavigationTiming) {
    const metrics = {
      "DNS Lookup": entry.domainLookupEnd - entry.domainLookupStart,
      "TCP Connect": entry.connectEnd - entry.connectStart,
      "SSL Handshake": entry.connectEnd - entry.secureConnectionStart,
      Request: entry.responseStart - entry.requestStart,
      Response: entry.responseEnd - entry.responseStart,
      "DOM Processing": entry.domComplete - entry.responseEnd,
      "Load Complete": entry.loadEventEnd - entry.loadEventStart,
    };

    Object.entries(metrics).forEach(([name, value]) => {
      if (value > 0) {
        this.recordMetric({
          name: `Navigation.${name}`,
          value,
          rating: this.getRatingForNavigationMetric(name, value),
          timestamp: Date.now(),
          url: window.location.href,
          userAgent: navigator.userAgent,
          sessionId: this.sessionId,
          userId: this.userId,
        });
      }
    });
  }

  private processResourceEntry(entry: PerformanceResourceTiming) {
    // Track slow resources
    const duration = entry.responseEnd - entry.startTime;

    if (duration > 1000) {
      // Resources taking more than 1s
      this.recordMetric({
        name: "Slow Resource",
        value: duration,
        rating: "poor",
        timestamp: Date.now(),
        url: entry.name,
        userAgent: navigator.userAgent,
        sessionId: this.sessionId,
        userId: this.userId,
      });
    }

    // Track icon sprite loads specifically
    if (entry.name.includes("sprite-") && entry.name.endsWith(".svg")) {
      this.recordMetric({
        name: "Icon Sprite Load",
        value: duration,
        rating:
          duration < 200
            ? "good"
            : duration < 500
            ? "needs-improvement"
            : "poor",
        timestamp: Date.now(),
        url: entry.name,
        userAgent: navigator.userAgent,
        sessionId: this.sessionId,
        userId: this.userId,
      });
    }
  }

  private trackAngularBootstrap() {
    // Measure Angular bootstrap time
    const navigationStart = performance.timeOrigin;
    const bootstrapEnd = performance.now();

    this.recordMetric({
      name: "Angular Bootstrap",
      value: bootstrapEnd,
      rating:
        bootstrapEnd < 1000
          ? "good"
          : bootstrapEnd < 2000
          ? "needs-improvement"
          : "poor",
      timestamp: Date.now(),
      url: window.location.href,
      userAgent: navigator.userAgent,
      sessionId: this.sessionId,
      userId: this.userId,
    });
  }

  private trackRouteChanges() {
    // Hook into Router events for SPA navigation timing
    let routeStartTime = 0;

    if (typeof window !== "undefined" && window.addEventListener) {
      window.addEventListener("beforeunload", () => {
        routeStartTime = performance.now();
      });

      window.addEventListener("load", () => {
        if (routeStartTime > 0) {
          const routeTime = performance.now() - routeStartTime;
          this.recordMetric({
            name: "Route Change",
            value: routeTime,
            rating:
              routeTime < 500
                ? "good"
                : routeTime < 1000
                ? "needs-improvement"
                : "poor",
            timestamp: Date.now(),
            url: window.location.href,
            userAgent: navigator.userAgent,
            sessionId: this.sessionId,
            userId: this.userId,
          });
        }
      });
    }
  }

  private trackComponentMounts() {
    // Track component lifecycle metrics
    // This would integrate with Angular's lifecycle hooks
  }

  private setupErrorTracking() {
    window.addEventListener("error", (event) => {
      this.trackBusinessEvent({
        event: "JavaScript Error",
        category: "interaction",
        metadata: {
          message: event.message,
          filename: event.filename,
          lineno: event.lineno,
          colno: event.colno,
          stack: event.error?.stack,
        },
        timestamp: Date.now(),
        sessionId: this.sessionId,
        userId: this.userId,
      });
    });

    window.addEventListener("unhandledrejection", (event) => {
      this.trackBusinessEvent({
        event: "Promise Rejection",
        category: "interaction",
        metadata: {
          reason: event.reason?.toString(),
          stack: event.reason?.stack,
        },
        timestamp: Date.now(),
        sessionId: this.sessionId,
        userId: this.userId,
      });
    });
  }

  private setupNetworkMonitoring() {
    if ("navigator" in window && "connection" in navigator) {
      const connection = (navigator as any).connection;

      if (connection) {
        this.recordMetric({
          name: "Network.EffectiveType",
          value: this.networkTypeToValue(connection.effectiveType),
          rating: "good",
          timestamp: Date.now(),
          url: window.location.href,
          userAgent: navigator.userAgent,
          connection,
          sessionId: this.sessionId,
          userId: this.userId,
        });

        connection.addEventListener("change", () => {
          this.recordMetric({
            name: "Network.Change",
            value: this.networkTypeToValue(connection.effectiveType),
            rating: "good",
            timestamp: Date.now(),
            url: window.location.href,
            userAgent: navigator.userAgent,
            connection,
            sessionId: this.sessionId,
            userId: this.userId,
          });
        });
      }
    }
  }

  private networkTypeToValue(type: string): number {
    const typeMap: Record<string, number> = {
      "slow-2g": 1,
      "2g": 2,
      "3g": 3,
      "4g": 4,
      "5g": 5,
    };
    return typeMap[type] || 0;
  }

  private getRating(
    metric: string,
    value: number
  ): "good" | "needs-improvement" | "poor" {
    const thresholds: Record<string, [number, number]> = {
      CLS: [0.1, 0.25],
      FCP: [1800, 3000],
      FID: [100, 300],
      LCP: [2500, 4000],
      TTFB: [800, 1800],
    };

    const [good, poor] = thresholds[metric] || [1000, 2000];

    if (value <= good) return "good";
    if (value <= poor) return "needs-improvement";
    return "poor";
  }

  private getRatingForNavigationMetric(
    name: string,
    value: number
  ): "good" | "needs-improvement" | "poor" {
    const thresholds: Record<string, [number, number]> = {
      "DNS Lookup": [50, 200],
      "TCP Connect": [100, 300],
      "SSL Handshake": [200, 500],
      Request: [100, 300],
      Response: [200, 500],
      "DOM Processing": [500, 1500],
      "Load Complete": [100, 300],
    };

    const [good, poor] = thresholds[name] || [200, 1000];

    if (value <= good) return "good";
    if (value <= poor) return "needs-improvement";
    return "poor";
  }

  // Public API methods
  recordMetric(metric: PerformanceMetric) {
    const currentMetrics = this.metricsSubject.value;
    this.metricsSubject.next([...currentMetrics, metric]);

    // Send to external services
    this.sendToAnalytics(metric);
  }

  trackBusinessEvent(event: BusinessMetric) {
    const currentEvents = this.businessEventsSubject.value;
    this.businessEventsSubject.next([...currentEvents, event]);

    // Send to external services
    this.sendBusinessEventToAnalytics(event);
  }

  setUserId(userId: string) {
    this.userId = userId;
  }

  // E-commerce specific tracking methods
  trackAddToCart(productId: string, price: number, quantity: number = 1) {
    this.trackBusinessEvent({
      event: "add_to_cart",
      category: "ecommerce",
      value: price * quantity,
      metadata: {
        productId,
        price,
        quantity,
        currency: "USD",
      },
      timestamp: Date.now(),
      sessionId: this.sessionId,
      userId: this.userId,
    });
  }

  trackPurchase(transactionId: string, value: number, items: any[]) {
    this.trackBusinessEvent({
      event: "purchase",
      category: "ecommerce",
      value,
      metadata: {
        transactionId,
        items,
        currency: "USD",
      },
      timestamp: Date.now(),
      sessionId: this.sessionId,
      userId: this.userId,
    });
  }

  trackSearchPerformance(
    query: string,
    resultsCount: number,
    searchTime: number
  ) {
    this.trackBusinessEvent({
      event: "search",
      category: "interaction",
      value: searchTime,
      metadata: {
        query,
        resultsCount,
        searchTime,
      },
      timestamp: Date.now(),
      sessionId: this.sessionId,
      userId: this.userId,
    });
  }

  private sendToAnalytics(metric: PerformanceMetric) {
    // Google Analytics 4
    if (typeof gtag !== "undefined") {
      gtag("event", "performance_metric", {
        custom_parameter_metric_name: metric.name,
        custom_parameter_metric_value: metric.value,
        custom_parameter_metric_rating: metric.rating,
        custom_parameter_session_id: metric.sessionId,
      });
    }

    // DataDog RUM
    if (typeof (window as any).DD_RUM !== "undefined") {
      (window as any).DD_RUM.addRumGlobalContext("performance_metric", {
        name: metric.name,
        value: metric.value,
        rating: metric.rating,
        sessionId: metric.sessionId,
      });
    }

    // Custom API
    this.sendToCustomAPI("metrics", metric);
  }

  private sendBusinessEventToAnalytics(event: BusinessMetric) {
    // Google Analytics 4
    if (typeof gtag !== "undefined") {
      gtag("event", event.event, {
        event_category: event.category,
        value: event.value,
        ...event.metadata,
      });
    }

    // DataDog RUM
    if (typeof (window as any).DD_RUM !== "undefined") {
      (window as any).DD_RUM.addUserAction(event.event, event.metadata);
    }

    // Custom API
    this.sendToCustomAPI("events", event);
  }

  private sendToCustomAPI(endpoint: string, data: any) {
    // Batch and send to custom analytics API
    if (typeof fetch !== "undefined") {
      // Debounce and batch requests
      this.batchedSend(endpoint, data);
    }
  }

  private batchQueue: Record<string, any[]> = {};
  private batchTimeout: Record<string, any> = {};

  private batchedSend(endpoint: string, data: any) {
    if (!this.batchQueue[endpoint]) {
      this.batchQueue[endpoint] = [];
    }

    this.batchQueue[endpoint].push(data);

    // Clear existing timeout
    if (this.batchTimeout[endpoint]) {
      clearTimeout(this.batchTimeout[endpoint]);
    }

    // Set new timeout
    this.batchTimeout[endpoint] = setTimeout(() => {
      this.flushBatch(endpoint);
    }, 5000); // Send every 5 seconds

    // Send immediately if batch is full
    if (this.batchQueue[endpoint].length >= 10) {
      this.flushBatch(endpoint);
    }
  }

  private flushBatch(endpoint: string) {
    if (!this.batchQueue[endpoint] || this.batchQueue[endpoint].length === 0) {
      return;
    }

    const batch = [...this.batchQueue[endpoint]];
    this.batchQueue[endpoint] = [];

    fetch(`/api/analytics/${endpoint}`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        batch,
        timestamp: Date.now(),
        sessionId: this.sessionId,
      }),
    }).catch((error) => {
      console.error(`Failed to send analytics batch for ${endpoint}:`, error);
    });
  }

  // Performance budget checking
  checkPerformanceBudgets() {
    const budgets = {
      LCP: 2500,
      FCP: 1800,
      CLS: 0.1,
      FID: 100,
    };

    const violations: string[] = [];
    const currentMetrics = this.metricsSubject.value;

    Object.entries(budgets).forEach(([metric, budget]) => {
      const latestMetric = currentMetrics
        .filter((m) => m.name === metric)
        .sort((a, b) => b.timestamp - a.timestamp)[0];

      if (latestMetric && latestMetric.value > budget) {
        violations.push(`${metric}: ${latestMetric.value} > ${budget}`);
      }
    });

    if (violations.length > 0) {
      this.trackBusinessEvent({
        event: "performance_budget_violation",
        category: "interaction",
        metadata: {
          violations,
          url: window.location.href,
        },
        timestamp: Date.now(),
        sessionId: this.sessionId,
        userId: this.userId,
      });
    }

    return violations;
  }

  // Get performance summary
  getPerformanceSummary(): Observable<any> {
    return this.metrics$.pipe(
      debounceTime(1000),
      distinctUntilChanged(),
      map((metrics) => {
        const summary: Record<string, any> = {};

        ["LCP", "FCP", "CLS", "FID", "TTFB"].forEach((metricName) => {
          const metricValues = metrics
            .filter((m) => m.name === metricName)
            .map((m) => m.value);

          if (metricValues.length > 0) {
            summary[metricName] = {
              latest: metricValues[metricValues.length - 1],
              average:
                metricValues.reduce((a, b) => a + b, 0) / metricValues.length,
              count: metricValues.length,
            };
          }
        });

        return summary;
      })
    );
  }
}
```

### 2. Integración con Google Analytics 4

```typescript
// libs/frontend/analytics/src/lib/ga4-integration.service.ts
import { Injectable } from "@angular/core";
import { PerformanceAnalyticsService } from "./performance-analytics.service";

@Injectable({
  providedIn: "root",
})
export class GA4IntegrationService {
  constructor(private performanceService: PerformanceAnalyticsService) {
    this.setupGA4();
  }

  private setupGA4() {
    // Enhanced E-commerce tracking
    this.setupEcommerceTracking();

    // Custom performance events
    this.setupPerformanceTracking();

    // User journey tracking
    this.setupUserJourneyTracking();
  }

  private setupEcommerceTracking() {
    this.performanceService.businessEvents$.subscribe((events) => {
      events.forEach((event) => {
        switch (event.event) {
          case "add_to_cart":
            this.trackGA4Event("add_to_cart", {
              currency: event.metadata?.currency || "USD",
              value: event.value,
              items: [
                {
                  item_id: event.metadata?.productId,
                  price: event.metadata?.price,
                  quantity: event.metadata?.quantity,
                },
              ],
            });
            break;

          case "purchase":
            this.trackGA4Event("purchase", {
              transaction_id: event.metadata?.transactionId,
              currency: event.metadata?.currency || "USD",
              value: event.value,
              items: event.metadata?.items,
            });
            break;

          case "search":
            this.trackGA4Event("search", {
              search_term: event.metadata?.query,
              custom_parameter_results_count: event.metadata?.resultsCount,
              custom_parameter_search_time: event.metadata?.searchTime,
            });
            break;
        }
      });
    });
  }

  private setupPerformanceTracking() {
    this.performanceService.metrics$.subscribe((metrics) => {
      metrics.forEach((metric) => {
        // Only send poor performing metrics to reduce data volume
        if (metric.rating === "poor") {
          this.trackGA4Event("poor_performance", {
            custom_parameter_metric_name: metric.name,
            custom_parameter_metric_value: metric.value,
            custom_parameter_page_url: metric.url,
          });
        }
      });
    });
  }

  private setupUserJourneyTracking() {
    // Track critical user paths
    const criticalPaths = [
      "/",
      "/productos",
      "/producto/",
      "/cart",
      "/checkout",
    ];

    let previousPath = "";

    // Listen to route changes
    if (typeof window !== "undefined") {
      const observer = new MutationObserver(() => {
        const currentPath = window.location.pathname;

        if (currentPath !== previousPath) {
          // Check if this is a critical path transition
          const fromCritical = criticalPaths.some((path) =>
            previousPath.startsWith(path)
          );
          const toCritical = criticalPaths.some((path) =>
            currentPath.startsWith(path)
          );

          if (fromCritical && toCritical) {
            this.trackGA4Event("critical_path_navigation", {
              custom_parameter_from_path: previousPath,
              custom_parameter_to_path: currentPath,
              custom_parameter_session_id: this.performanceService["sessionId"],
            });
          }

          previousPath = currentPath;
        }
      });

      observer.observe(document, { subtree: true, childList: true });
    }
  }

  private trackGA4Event(eventName: string, parameters: any) {
    if (typeof gtag !== "undefined") {
      gtag("event", eventName, parameters);
    }
  }

  // Public methods for manual tracking
  trackPageView(path: string, title: string) {
    this.trackGA4Event("page_view", {
      page_title: title,
      page_location: window.location.origin + path,
    });
  }

  trackCustomEvent(eventName: string, parameters: Record<string, any>) {
    this.trackGA4Event(eventName, parameters);
  }

  // Enhanced E-commerce methods
  trackViewItem(
    productId: string,
    productName: string,
    category: string,
    price: number
  ) {
    this.trackGA4Event("view_item", {
      currency: "USD",
      value: price,
      items: [
        {
          item_id: productId,
          item_name: productName,
          item_category: category,
          price: price,
          quantity: 1,
        },
      ],
    });
  }

  trackBeginCheckout(value: number, items: any[]) {
    this.trackGA4Event("begin_checkout", {
      currency: "USD",
      value: value,
      items: items,
    });
  }
}
```

### 3. Dashboard de Performance en Tiempo Real

```typescript
// libs/frontend/analytics/src/lib/performance-dashboard.component.ts
import {
  Component,
  OnInit,
  OnDestroy,
  ChangeDetectionStrategy,
} from "@angular/core";
import { CommonModule } from "@angular/common";
import {
  PerformanceAnalyticsService,
  PerformanceMetric,
} from "./performance-analytics.service";
import {
  Observable,
  Subject,
  combineLatest,
  map,
  startWith,
  takeUntil,
} from "rxjs";

interface DashboardData {
  currentMetrics: Record<string, number>;
  trends: Record<string, number[]>;
  alerts: string[];
  summary: {
    performanceScore: number;
    issues: number;
    improvements: string[];
  };
}

@Component({
  selector: "performance-dashboard",
  standalone: true,
  imports: [CommonModule],
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="performance-dashboard" *ngIf="dashboardData$ | async as data">
      <!-- Real-time status -->
      <div class="status-header">
        <div
          class="status-item"
          [class]="getStatusClass(data.summary.performanceScore)"
        >
          <h3>Performance Score</h3>
          <div class="score">{{ data.summary.performanceScore }}/100</div>
        </div>

        <div class="status-item" [class.warning]="data.summary.issues > 0">
          <h3>Active Issues</h3>
          <div class="score">{{ data.summary.issues }}</div>
        </div>

        <div class="status-item">
          <h3>Session ID</h3>
          <div class="session-id">{{ sessionId }}</div>
        </div>
      </div>

      <!-- Core Web Vitals Grid -->
      <div class="metrics-grid">
        <div class="metric-card" *ngFor="let metric of coreWebVitals">
          <div class="metric-header">
            <h4>{{ metric.name }}</h4>
            <span class="metric-unit">{{ metric.unit }}</span>
          </div>

          <div
            class="metric-value"
            [class]="
              getMetricClass(metric.name, data.currentMetrics[metric.name])
            "
          >
            {{
              formatMetricValue(metric.name, data.currentMetrics[metric.name])
            }}
          </div>

          <div class="metric-trend" *ngIf="data.trends[metric.name]">
            <div class="trend-chart">
              <div
                class="trend-bar"
                *ngFor="let value of data.trends[metric.name]; let i = index"
                [style.height.%]="
                  getTrendHeight(value, data.trends[metric.name])
                "
                [class]="getMetricClass(metric.name, value)"
              ></div>
            </div>
          </div>

          <div class="metric-threshold">
            <span class="good">Good: &lt; {{ metric.goodThreshold }}</span>
            <span class="poor">Poor: &gt; {{ metric.poorThreshold }}</span>
          </div>
        </div>
      </div>

      <!-- Performance Alerts -->
      <div class="alerts-section" *ngIf="data.alerts.length > 0">
        <h3>🚨 Performance Alerts</h3>
        <div class="alert-list">
          <div class="alert-item" *ngFor="let alert of data.alerts">
            {{ alert }}
          </div>
        </div>
      </div>

      <!-- Improvement Suggestions -->
      <div
        class="improvements-section"
        *ngIf="data.summary.improvements.length > 0"
      >
        <h3>💡 Suggested Improvements</h3>
        <ul class="improvement-list">
          <li *ngFor="let improvement of data.summary.improvements">
            {{ improvement }}
          </li>
        </ul>
      </div>

      <!-- Real-time Network Info -->
      <div class="network-info" *ngIf="networkInfo">
        <h3>📶 Network Information</h3>
        <div class="network-details">
          <span>Type: {{ networkInfo.effectiveType }}</span>
          <span>Downlink: {{ networkInfo.downlink }} Mbps</span>
          <span>RTT: {{ networkInfo.rtt }}ms</span>
        </div>
      </div>

      <!-- Export/Share Options -->
      <div class="actions-section">
        <button class="btn-primary" (click)="exportReport()">
          📊 Export Report
        </button>
        <button class="btn-secondary" (click)="shareMetrics()">
          🔗 Share Metrics
        </button>
        <button class="btn-secondary" (click)="resetMetrics()">
          🔄 Reset Session
        </button>
      </div>
    </div>
  `,
  styles: [
    `
      .performance-dashboard {
        padding: 2rem;
        font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
        background: #f8fafc;
        min-height: 100vh;
      }

      .status-header {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 1rem;
        margin-bottom: 2rem;
      }

      .status-item {
        background: white;
        padding: 1.5rem;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        text-align: center;
      }

      .status-item.good {
        border-left: 4px solid #10b981;
      }
      .status-item.needs-improvement {
        border-left: 4px solid #f59e0b;
      }
      .status-item.poor {
        border-left: 4px solid #ef4444;
      }
      .status-item.warning {
        border-left: 4px solid #f59e0b;
      }

      .score {
        font-size: 2rem;
        font-weight: bold;
        margin-top: 0.5rem;
      }

      .session-id {
        font-family: monospace;
        font-size: 0.875rem;
        background: #f1f5f9;
        padding: 0.25rem 0.5rem;
        border-radius: 4px;
        margin-top: 0.5rem;
      }

      .metrics-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
        gap: 1.5rem;
        margin-bottom: 2rem;
      }

      .metric-card {
        background: white;
        padding: 1.5rem;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
      }

      .metric-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 1rem;
      }

      .metric-header h4 {
        margin: 0;
        font-size: 1.125rem;
      }

      .metric-unit {
        color: #6b7280;
        font-size: 0.875rem;
      }

      .metric-value {
        font-size: 2.5rem;
        font-weight: bold;
        margin-bottom: 1rem;
      }

      .metric-value.good {
        color: #10b981;
      }
      .metric-value.needs-improvement {
        color: #f59e0b;
      }
      .metric-value.poor {
        color: #ef4444;
      }

      .trend-chart {
        display: flex;
        align-items: end;
        height: 60px;
        gap: 2px;
        margin-bottom: 1rem;
      }

      .trend-bar {
        flex: 1;
        min-height: 4px;
        border-radius: 2px;
        transition: height 0.3s ease;
      }

      .trend-bar.good {
        background: #10b981;
      }
      .trend-bar.needs-improvement {
        background: #f59e0b;
      }
      .trend-bar.poor {
        background: #ef4444;
      }

      .metric-threshold {
        display: flex;
        justify-content: space-between;
        font-size: 0.75rem;
        color: #6b7280;
      }

      .metric-threshold .good {
        color: #10b981;
      }
      .metric-threshold .poor {
        color: #ef4444;
      }

      .alerts-section,
      .improvements-section {
        background: white;
        padding: 1.5rem;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        margin-bottom: 2rem;
      }

      .alert-list {
        margin-top: 1rem;
      }

      .alert-item {
        background: #fef2f2;
        border: 1px solid #fecaca;
        padding: 0.75rem;
        border-radius: 4px;
        margin-bottom: 0.5rem;
        color: #991b1b;
      }

      .improvement-list {
        margin-top: 1rem;
        padding-left: 1.5rem;
      }

      .improvement-list li {
        margin-bottom: 0.5rem;
        color: #374151;
      }

      .network-info {
        background: white;
        padding: 1.5rem;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        margin-bottom: 2rem;
      }

      .network-details {
        display: flex;
        gap: 2rem;
        margin-top: 1rem;
        font-family: monospace;
        font-size: 0.875rem;
      }

      .actions-section {
        display: flex;
        gap: 1rem;
        justify-content: center;
      }

      .btn-primary,
      .btn-secondary {
        padding: 0.75rem 1.5rem;
        border: none;
        border-radius: 4px;
        font-weight: 500;
        cursor: pointer;
        transition: all 0.2s;
      }

      .btn-primary {
        background: #3b82f6;
        color: white;
      }

      .btn-primary:hover {
        background: #2563eb;
      }

      .btn-secondary {
        background: #f1f5f9;
        color: #374151;
      }

      .btn-secondary:hover {
        background: #e2e8f0;
      }
    `,
  ],
})
export class PerformanceDashboardComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();

  dashboardData$!: Observable<DashboardData>;
  sessionId: string = "";
  networkInfo: any = null;

  coreWebVitals = [
    { name: "LCP", unit: "ms", goodThreshold: 2500, poorThreshold: 4000 },
    { name: "FCP", unit: "ms", goodThreshold: 1800, poorThreshold: 3000 },
    { name: "CLS", unit: "score", goodThreshold: 0.1, poorThreshold: 0.25 },
    { name: "FID", unit: "ms", goodThreshold: 100, poorThreshold: 300 },
    { name: "TTFB", unit: "ms", goodThreshold: 800, poorThreshold: 1800 },
  ];

  constructor(private performanceService: PerformanceAnalyticsService) {}

  ngOnInit() {
    this.sessionId = this.performanceService["sessionId"];
    this.setupNetworkInfo();
    this.setupDashboardData();
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }

  private setupNetworkInfo() {
    if ("navigator" in window && "connection" in navigator) {
      this.networkInfo = (navigator as any).connection;
    }
  }

  private setupDashboardData() {
    this.dashboardData$ = combineLatest([
      this.performanceService.metrics$,
      this.performanceService.businessEvents$,
    ]).pipe(
      takeUntil(this.destroy$),
      map(([metrics, events]) => this.processDashboardData(metrics, events)),
      startWith(this.getInitialDashboardData())
    );
  }

  private processDashboardData(
    metrics: PerformanceMetric[],
    events: any[]
  ): DashboardData {
    const currentMetrics: Record<string, number> = {};
    const trends: Record<string, number[]> = {};
    const alerts: string[] = [];

    // Process current metrics
    this.coreWebVitals.forEach((vital) => {
      const vitalMetrics = metrics.filter((m) => m.name === vital.name);
      if (vitalMetrics.length > 0) {
        const latest = vitalMetrics[vitalMetrics.length - 1];
        currentMetrics[vital.name] = latest.value;

        // Build trend data (last 10 measurements)
        trends[vital.name] = vitalMetrics.slice(-10).map((m) => m.value);

        // Check for alerts
        if (latest.rating === "poor") {
          alerts.push(
            `${vital.name} is performing poorly: ${this.formatMetricValue(
              vital.name,
              latest.value
            )}`
          );
        }
      }
    });

    // Calculate performance score
    const performanceScore = this.calculatePerformanceScore(currentMetrics);

    // Generate improvement suggestions
    const improvements = this.generateImprovements(currentMetrics, metrics);

    return {
      currentMetrics,
      trends,
      alerts,
      summary: {
        performanceScore,
        issues: alerts.length,
        improvements,
      },
    };
  }

  private getInitialDashboardData(): DashboardData {
    return {
      currentMetrics: {},
      trends: {},
      alerts: [],
      summary: {
        performanceScore: 0,
        issues: 0,
        improvements: [],
      },
    };
  }

  private calculatePerformanceScore(metrics: Record<string, number>): number {
    const weights = {
      LCP: 0.25,
      FCP: 0.25,
      CLS: 0.25,
      FID: 0.25,
    };

    let totalScore = 0;
    let totalWeight = 0;

    Object.entries(weights).forEach(([metric, weight]) => {
      if (metrics[metric] !== undefined) {
        const vital = this.coreWebVitals.find((v) => v.name === metric)!;
        const score = this.getMetricScore(
          metric,
          metrics[metric],
          vital.goodThreshold,
          vital.poorThreshold
        );
        totalScore += score * weight;
        totalWeight += weight;
      }
    });

    return totalWeight > 0 ? Math.round(totalScore / totalWeight) : 0;
  }

  private getMetricScore(
    metric: string,
    value: number,
    good: number,
    poor: number
  ): number {
    if (value <= good) return 100;
    if (value >= poor) return 0;

    // Linear interpolation between good and poor
    return Math.round(100 - ((value - good) / (poor - good)) * 100);
  }

  private generateImprovements(
    currentMetrics: Record<string, number>,
    allMetrics: PerformanceMetric[]
  ): string[] {
    const improvements: string[] = [];

    // LCP improvements
    if (currentMetrics["LCP"] > 2500) {
      improvements.push(
        "Optimize Largest Contentful Paint: consider image optimization and preloading critical resources"
      );
    }

    // FCP improvements
    if (currentMetrics["FCP"] > 1800) {
      improvements.push(
        "Improve First Contentful Paint: reduce render-blocking resources and inline critical CSS"
      );
    }

    // CLS improvements
    if (currentMetrics["CLS"] > 0.1) {
      improvements.push(
        "Reduce Cumulative Layout Shift: set size attributes on images and reserve space for dynamic content"
      );
    }

    // Icon-specific improvements
    const iconMetrics = allMetrics.filter((m) => m.name === "Icon Sprite Load");
    if (iconMetrics.some((m) => m.value > 500)) {
      improvements.push(
        "Optimize icon loading: consider preloading critical icon sprites or using inline SVGs for above-the-fold icons"
      );
    }

    return improvements;
  }

  // Template helper methods
  getStatusClass(score: number): string {
    if (score >= 90) return "good";
    if (score >= 70) return "needs-improvement";
    return "poor";
  }

  getMetricClass(metricName: string, value: number): string {
    if (value === undefined) return "";

    const vital = this.coreWebVitals.find((v) => v.name === metricName);
    if (!vital) return "";

    if (value <= vital.goodThreshold) return "good";
    if (value <= vital.poorThreshold) return "needs-improvement";
    return "poor";
  }

  formatMetricValue(metricName: string, value: number): string {
    if (value === undefined) return "N/A";

    if (metricName === "CLS") {
      return value.toFixed(3);
    }

    return Math.round(value).toString();
  }

  getTrendHeight(value: number, series: number[]): number {
    const max = Math.max(...series);
    const min = Math.min(...series);

    if (max === min) return 50;

    return ((value - min) / (max - min)) * 100;
  }

  // Action methods
  exportReport() {
    this.dashboardData$.pipe(takeUntil(this.destroy$)).subscribe((data) => {
      const report = {
        timestamp: new Date().toISOString(),
        sessionId: this.sessionId,
        performanceScore: data.summary.performanceScore,
        metrics: data.currentMetrics,
        alerts: data.alerts,
        improvements: data.summary.improvements,
        networkInfo: this.networkInfo,
      };

      const blob = new Blob([JSON.stringify(report, null, 2)], {
        type: "application/json",
      });
      const url = URL.createObjectURL(blob);
      const link = document.createElement("a");
      link.href = url;
      link.download = `performance-report-${this.sessionId}.json`;
      link.click();
      URL.revokeObjectURL(url);
    });
  }

  shareMetrics() {
    if (navigator.share) {
      this.dashboardData$.pipe(takeUntil(this.destroy$)).subscribe((data) => {
        navigator.share({
          title: "Performance Metrics",
          text: `Performance Score: ${data.summary.performanceScore}/100`,
          url: window.location.href,
        });
      });
    } else {
      // Fallback: copy to clipboard
      this.dashboardData$.pipe(takeUntil(this.destroy$)).subscribe((data) => {
        const text = `Performance Score: ${data.summary.performanceScore}/100\nSession: ${this.sessionId}`;
        navigator.clipboard.writeText(text);
        alert("Metrics copied to clipboard");
      });
    }
  }

  resetMetrics() {
    if (confirm("Reset current performance metrics session?")) {
      window.location.reload();
    }
  }
}
```

### 4. Sistema de Alertas Automáticas

```typescript
// libs/frontend/analytics/src/lib/performance-alerts.service.ts
import { Injectable } from "@angular/core";
import { PerformanceAnalyticsService } from "./performance-analytics.service";
import { filter, debounceTime, distinctUntilChanged } from "rxjs/operators";

interface AlertRule {
  metric: string;
  threshold: number;
  condition: "greater" | "less" | "equal";
  severity: "low" | "medium" | "high" | "critical";
  description: string;
}

@Injectable({
  providedIn: "root",
})
export class PerformanceAlertsService {
  private alertRules: AlertRule[] = [
    {
      metric: "LCP",
      threshold: 4000,
      condition: "greater",
      severity: "critical",
      description: "Largest Contentful Paint is critically slow",
    },
    {
      metric: "FCP",
      threshold: 3000,
      condition: "greater",
      severity: "high",
      description: "First Contentful Paint is too slow",
    },
    {
      metric: "CLS",
      threshold: 0.25,
      condition: "greater",
      severity: "high",
      description: "Cumulative Layout Shift is causing poor UX",
    },
    {
      metric: "FID",
      threshold: 300,
      condition: "greater",
      severity: "medium",
      description: "First Input Delay is affecting interactivity",
    },
  ];

  constructor(private performanceService: PerformanceAnalyticsService) {
    this.setupAlerts();
  }

  private setupAlerts() {
    this.performanceService.metrics$
      .pipe(
        debounceTime(2000), // Wait 2 seconds to avoid spam
        distinctUntilChanged(
          (prev, curr) => JSON.stringify(prev) === JSON.stringify(curr)
        )
      )
      .subscribe((metrics) => {
        this.checkAlerts(metrics);
      });
  }

  private checkAlerts(metrics: any[]) {
    this.alertRules.forEach((rule) => {
      const relevantMetrics = metrics.filter((m) => m.name === rule.metric);

      if (relevantMetrics.length > 0) {
        const latestMetric = relevantMetrics[relevantMetrics.length - 1];

        if (this.shouldTriggerAlert(latestMetric.value, rule)) {
          this.triggerAlert(rule, latestMetric);
        }
      }
    });
  }

  private shouldTriggerAlert(value: number, rule: AlertRule): boolean {
    switch (rule.condition) {
      case "greater":
        return value > rule.threshold;
      case "less":
        return value < rule.threshold;
      case "equal":
        return value === rule.threshold;
      default:
        return false;
    }
  }

  private triggerAlert(rule: AlertRule, metric: any) {
    const alert = {
      rule,
      metric,
      timestamp: Date.now(),
      severity: rule.severity,
      message: `${rule.description}: ${metric.value} ${this.getUnit(
        rule.metric
      )}`,
    };

    // Send to different channels based on severity
    switch (rule.severity) {
      case "critical":
        this.sendCriticalAlert(alert);
        this.sendSlackAlert(alert, "#critical-performance");
        break;
      case "high":
        this.sendHighPriorityAlert(alert);
        break;
      case "medium":
        this.sendMediumPriorityAlert(alert);
        break;
      case "low":
        this.sendLowPriorityAlert(alert);
        break;
    }

    // Track the alert as a business event
    this.performanceService.trackBusinessEvent({
      event: "performance_alert",
      category: "interaction",
      metadata: {
        rule: rule.metric,
        severity: rule.severity,
        value: metric.value,
        threshold: rule.threshold,
      },
      timestamp: Date.now(),
      sessionId: metric.sessionId,
    });
  }

  private sendCriticalAlert(alert: any) {
    // Send to Slack, PagerDuty, email
    this.sendSlackAlert(alert, "#critical-alerts");
    this.sendEmailAlert(alert, [
      "dev-team@company.com",
      "ops-team@company.com",
    ]);

    if (typeof window !== "undefined") {
      // Show browser notification for critical alerts
      this.showBrowserNotification(alert);
    }
  }

  private sendHighPriorityAlert(alert: any) {
    this.sendSlackAlert(alert, "#performance-alerts");
    this.sendEmailAlert(alert, ["dev-team@company.com"]);
  }

  private sendMediumPriorityAlert(alert: any) {
    this.sendSlackAlert(alert, "#performance-monitoring");
  }

  private sendLowPriorityAlert(alert: any) {
    // Just log for now
    console.warn("Performance Alert:", alert);
  }

  private sendSlackAlert(alert: any, channel: string) {
    const webhookUrl = process.env["SLACK_WEBHOOK_URL"];

    if (webhookUrl && typeof fetch !== "undefined") {
      const payload = {
        channel,
        username: "Performance Monitor",
        icon_emoji: ":warning:",
        attachments: [
          {
            color: this.getSeverityColor(alert.severity),
            title: `Performance Alert - ${alert.rule.metric}`,
            text: alert.message,
            fields: [
              {
                title: "Metric",
                value: alert.rule.metric,
                short: true,
              },
              {
                title: "Value",
                value: `${alert.metric.value} ${this.getUnit(
                  alert.rule.metric
                )}`,
                short: true,
              },
              {
                title: "Threshold",
                value: `${alert.rule.threshold} ${this.getUnit(
                  alert.rule.metric
                )}`,
                short: true,
              },
              {
                title: "Page",
                value: alert.metric.url,
                short: false,
              },
            ],
            timestamp: alert.timestamp / 1000,
          },
        ],
      };

      fetch(webhookUrl, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(payload),
      }).catch((error) => {
        console.error("Failed to send Slack alert:", error);
      });
    }
  }

  private sendEmailAlert(alert: any, recipients: string[]) {
    // Send email via API
    if (typeof fetch !== "undefined") {
      fetch("/api/alerts/email", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          recipients,
          subject: `Performance Alert: ${alert.rule.metric}`,
          body: this.generateEmailBody(alert),
        }),
      }).catch((error) => {
        console.error("Failed to send email alert:", error);
      });
    }
  }

  private showBrowserNotification(alert: any) {
    if ("Notification" in window && Notification.permission === "granted") {
      new Notification(`Performance Alert: ${alert.rule.metric}`, {
        body: alert.message,
        icon: "/assets/icons/warning.png",
        badge: "/assets/icons/performance.png",
      });
    }
  }

  private getSeverityColor(severity: string): string {
    const colors = {
      critical: "#ff0000",
      high: "#ff8c00",
      medium: "#ffa500",
      low: "#ffff00",
    };
    return colors[severity as keyof typeof colors] || "#cccccc";
  }

  private getUnit(metric: string): string {
    const units = {
      LCP: "ms",
      FCP: "ms",
      FID: "ms",
      TTFB: "ms",
      CLS: "score",
    };
    return units[metric as keyof typeof units] || "";
  }

  private generateEmailBody(alert: any): string {
    return `
      <h2>Performance Alert</h2>
      <p><strong>Metric:</strong> ${alert.rule.metric}</p>
      <p><strong>Current Value:</strong> ${alert.metric.value} ${this.getUnit(
      alert.rule.metric
    )}</p>
      <p><strong>Threshold:</strong> ${alert.rule.threshold} ${this.getUnit(
      alert.rule.metric
    )}</p>
      <p><strong>Severity:</strong> ${alert.severity.toUpperCase()}</p>
      <p><strong>Page:</strong> ${alert.metric.url}</p>
      <p><strong>Time:</strong> ${new Date(alert.timestamp).toISOString()}</p>
      <p><strong>Session ID:</strong> ${alert.metric.sessionId}</p>
      
      <h3>Description</h3>
      <p>${alert.rule.description}</p>
      
      <h3>Recommendations</h3>
      <ul>
        ${this.getRecommendations(alert.rule.metric)
          .map((rec) => `<li>${rec}</li>`)
          .join("")}
      </ul>
    `;
  }

  private getRecommendations(metric: string): string[] {
    const recommendations: Record<string, string[]> = {
      LCP: [
        "Optimize images and use next-gen formats (WebP, AVIF)",
        "Preload critical resources",
        "Reduce server response times",
        "Remove render-blocking JavaScript",
      ],
      FCP: [
        "Eliminate render-blocking resources",
        "Inline critical CSS",
        "Reduce server response times",
        "Use a CDN",
      ],
      CLS: [
        "Set size attributes on images and videos",
        "Reserve space for dynamic content",
        "Avoid inserting content above existing content",
        "Use CSS aspect-ratio for responsive images",
      ],
      FID: [
        "Reduce JavaScript execution time",
        "Remove unused JavaScript",
        "Use code splitting",
        "Optimize third-party scripts",
      ],
    };

    return recommendations[metric] || ["Review performance best practices"];
  }

  // Public API for custom alerts
  addCustomAlert(rule: AlertRule) {
    this.alertRules.push(rule);
  }

  removeAlert(metric: string) {
    this.alertRules = this.alertRules.filter((rule) => rule.metric !== metric);
  }

  updateAlertThreshold(metric: string, newThreshold: number) {
    const rule = this.alertRules.find((r) => r.metric === metric);
    if (rule) {
      rule.threshold = newThreshold;
    }
  }
}
```

## 🎯 Integración con Business Metrics

### Tracking de Conversión

```typescript
// Ejemplo de integración con métricas de negocio
export class EcommerceTrackingService {
  constructor(private analytics: PerformanceAnalyticsService) {}

  trackConversionFunnel(step: string, metadata?: any) {
    this.analytics.trackBusinessEvent({
      event: `conversion_${step}`,
      category: "ecommerce",
      metadata: {
        ...metadata,
        performanceScore: this.getLatestPerformanceScore(),
      },
      timestamp: Date.now(),
      sessionId: this.analytics["sessionId"],
    });
  }

  private getLatestPerformanceScore(): number {
    // Correlate performance with conversion
    // ...implementation...
  }
}
```

## 📊 Performance Budget

### Definición de Presupuestos

```typescript
// Performance budgets específicos para ecommerce
export const PERFORMANCE_BUDGETS = {
  // Page-specific budgets
  homepage: {
    LCP: 2000,
    FCP: 1500,
    CLS: 0.05,
    FID: 50,
  },
  productPage: {
    LCP: 2500,
    FCP: 1800,
    CLS: 0.1,
    FID: 100,
  },
  checkout: {
    LCP: 2000,
    FCP: 1500,
    CLS: 0.01, // Very strict for checkout
    FID: 50,
  },
};
```

## 🔗 Integración con CI/CD

### GitHub Actions para Performance

```yaml
# .github/workflows/performance-monitoring.yml
name: Performance Monitoring

on:
  deployment_status:
    types: [success]

jobs:
  performance-check:
    runs-on: ubuntu-latest
    steps:
      - name: Performance Audit
        uses: treosh/lighthouse-ci-action@v9
        with:
          configPath: "./lighthouserc.json"
          uploadArtifacts: true

      - name: Send Performance Report
        run: |
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -H 'Content-Type: application/json' \
            -d '{"text":"Performance audit completed for deployment"}'
```

## 🚀 Lighthouse CI Integration

### Configuración Lighthouse CI

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

### Automatización de Auditorías

```typescript
// tools/performance-audit.ts
import { exec } from "child_process";
import { promisify } from "util";

const execAsync = promisify(exec);

export class PerformanceAuditService {
  async runLighthouseAudit(url: string, options: any = {}) {
    const lighthouse = await import("lighthouse");
    const chromeLauncher = await import("chrome-launcher");

    const chrome = await chromeLauncher.launch({
      chromeFlags: ["--headless", "--no-sandbox"],
    });

    const auditOptions = {
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
      ...options,
    };

    try {
      const runnerResult = await lighthouse(url, auditOptions);
      await chrome.kill();

      return this.parseAuditResults(runnerResult.lhr);
    } catch (error) {
      await chrome.kill();
      throw error;
    }
  }

  private parseAuditResults(results: any) {
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
        ttfb: results.audits["server-response-time"].numericValue,
      },
      opportunities: this.extractOpportunities(results.audits),
      diagnostics: this.extractDiagnostics(results.audits),
    };
  }

  private extractOpportunities(audits: any) {
    const opportunities = [];

    // Critical opportunities that save the most bytes/time
    const criticalAudits = [
      "unused-css-rules",
      "unused-javascript",
      "modern-image-formats",
      "efficiently-encode-images",
      "render-blocking-resources",
      "unminified-css",
      "unminified-javascript",
    ];

    criticalAudits.forEach((auditKey) => {
      const audit = audits[auditKey];
      if (audit && audit.score < 1 && audit.details) {
        opportunities.push({
          title: audit.title,
          description: audit.description,
          score: audit.score,
          numericValue: audit.numericValue,
          displayValue: audit.displayValue,
          details: audit.details,
        });
      }
    });

    return opportunities.sort((a, b) => b.numericValue - a.numericValue);
  }

  private extractDiagnostics(audits: any) {
    const diagnostics = [];

    const diagnosticAudits = [
      "main-thread-tasks",
      "bootup-time",
      "uses-long-cache-ttl",
      "total-byte-weight",
      "dom-size",
    ];

    diagnosticAudits.forEach((auditKey) => {
      const audit = audits[auditKey];
      if (audit && audit.score < 1) {
        diagnostics.push({
          title: audit.title,
          description: audit.description,
          score: audit.score,
          numericValue: audit.numericValue,
          displayValue: audit.displayValue,
        });
      }
    });

    return diagnostics;
  }

  // Batch audit multiple URLs
  async auditMultiplePages(urls: string[]) {
    const results = [];

    for (const url of urls) {
      try {
        console.log(`Auditing ${url}...`);
        const result = await this.runLighthouseAudit(url);
        results.push({
          url,
          timestamp: new Date().toISOString(),
          ...result,
        });

        // Wait between audits to avoid overwhelming the server
        await new Promise((resolve) => setTimeout(resolve, 2000));
      } catch (error) {
        console.error(`Failed to audit ${url}:`, error);
        results.push({
          url,
          timestamp: new Date().toISOString(),
          error: error.message,
        });
      }
    }

    return results;
  }

  // Generate performance report
  generateReport(auditResults: any[]) {
    const report = {
      summary: this.generateSummary(auditResults),
      details: auditResults,
      recommendations: this.generateRecommendations(auditResults),
      timestamp: new Date().toISOString(),
    };

    return report;
  }

  private generateSummary(results: any[]) {
    const validResults = results.filter((r) => !r.error);

    if (validResults.length === 0) {
      return { error: "No valid audit results" };
    }

    const averages = {
      performance:
        validResults.reduce((sum, r) => sum + r.performance, 0) /
        validResults.length,
      accessibility:
        validResults.reduce((sum, r) => sum + r.accessibility, 0) /
        validResults.length,
      bestPractices:
        validResults.reduce((sum, r) => sum + r.bestPractices, 0) /
        validResults.length,
      seo:
        validResults.reduce((sum, r) => sum + r.seo, 0) / validResults.length,
      pwa:
        validResults.reduce((sum, r) => sum + r.pwa, 0) / validResults.length,
    };

    const metricsAverages = {
      fcp:
        validResults.reduce((sum, r) => sum + r.metrics.fcp, 0) /
        validResults.length,
      lcp:
        validResults.reduce((sum, r) => sum + r.metrics.lcp, 0) /
        validResults.length,
      cls:
        validResults.reduce((sum, r) => sum + r.metrics.cls, 0) /
        validResults.length,
      fid:
        validResults.reduce((sum, r) => sum + r.metrics.fid, 0) /
        validResults.length,
      ttfb:
        validResults.reduce((sum, r) => sum + r.metrics.ttfb, 0) /
        validResults.length,
    };

    return {
      totalPages: results.length,
      successfulAudits: validResults.length,
      failedAudits: results.length - validResults.length,
      averageScores: averages,
      averageMetrics: metricsAverages,
      overallScore:
        Object.values(averages).reduce((sum, score) => sum + score, 0) / 5,
    };
  }

  private generateRecommendations(results: any[]) {
    const allOpportunities = results
      .filter((r) => !r.error && r.opportunities)
      .flatMap((r) => r.opportunities);

    // Group similar opportunities
    const groupedOpportunities = new Map();

    allOpportunities.forEach((opp) => {
      const key = opp.title;
      if (!groupedOpportunities.has(key)) {
        groupedOpportunities.set(key, {
          title: opp.title,
          description: opp.description,
          totalSavings: 0,
          pageCount: 0,
          avgScore: 0,
        });
      }

      const group = groupedOpportunities.get(key);
      group.totalSavings += opp.numericValue || 0;
      group.pageCount += 1;
      group.avgScore += opp.score;
    });

    // Convert to array and sort by total savings
    const recommendations = Array.from(groupedOpportunities.values())
      .map((group) => ({
        ...group,
        avgScore: group.avgScore / group.pageCount,
        avgSavings: group.totalSavings / group.pageCount,
      }))
      .sort((a, b) => b.totalSavings - a.totalSavings)
      .slice(0, 10); // Top 10 recommendations

    return recommendations;
  }
}

// CLI Usage
export async function runPerformanceAudit() {
  const auditService = new PerformanceAuditService();

  const urls = [
    "http://localhost:4200",
    "http://localhost:4200/productos",
    "http://localhost:4200/producto/ejemplo",
    "http://localhost:4200/carrito",
    "http://localhost:4200/checkout",
  ];

  console.log("🚀 Starting performance audit...");

  const results = await auditService.auditMultiplePages(urls);
  const report = auditService.generateReport(results);

  console.log("📊 Audit completed!");
  console.log("📋 Summary:", report.summary);
  console.log("💡 Top Recommendations:", report.recommendations);

  // Save report to file
  const fs = await import("fs");
  const path = await import("path");

  const reportPath = path.join(
    process.cwd(),
    `performance-report-${Date.now()}.json`
  );
  fs.writeFileSync(reportPath, JSON.stringify(report, null, 2));

  console.log(`📄 Report saved to: ${reportPath}`);

  return report;
}

// If running directly
if (require.main === module) {
  runPerformanceAudit().catch(console.error);
}
```

## 🎯 Performance Budget Monitoring

### Budget Configuration

```typescript
// performance-budget.config.ts
export const PERFORMANCE_BUDGET = {
  // Global budgets
  global: {
    performance: 90,
    accessibility: 95,
    bestPractices: 90,
    seo: 95,
    pwa: 80,
  },

  // Page-specific budgets
  pages: {
    homepage: {
      performance: 95,
      lcp: 2000,
      fcp: 1500,
      cls: 0.05,
      fid: 50,
      budgets: [
        { resourceType: "script", budget: 300 },
        { resourceType: "image", budget: 800 },
        { resourceType: "stylesheet", budget: 50 },
      ],
    },
    productPage: {
      performance: 90,
      lcp: 2500,
      fcp: 1800,
      cls: 0.1,
      fid: 100,
      budgets: [
        { resourceType: "script", budget: 400 },
        { resourceType: "image", budget: 1200 },
        { resourceType: "stylesheet", budget: 80 },
      ],
    },
    checkout: {
      performance: 95,
      lcp: 2000,
      fcp: 1500,
      cls: 0.01, // Very strict for checkout
      fid: 50,
      budgets: [
        { resourceType: "script", budget: 250 },
        { resourceType: "image", budget: 400 },
        { resourceType: "stylesheet", budget: 40 },
      ],
    },
  },

  // Alert thresholds
  alerts: {
    performance: {
      warning: 85,
      critical: 75,
    },
    lcp: {
      warning: 2500,
      critical: 4000,
    },
    fcp: {
      warning: 1800,
      critical: 3000,
    },
    cls: {
      warning: 0.1,
      critical: 0.25,
    },
    fid: {
      warning: 100,
      critical: 300,
    },
  },
};

export class PerformanceBudgetService {
  checkBudget(auditResult: any, pageName: string) {
    const budget =
      PERFORMANCE_BUDGET.pages[pageName] || PERFORMANCE_BUDGET.global;
    const violations = [];

    // Check Lighthouse scores
    if (auditResult.performance < budget.performance) {
      violations.push({
        type: "performance_score",
        expected: budget.performance,
        actual: auditResult.performance,
        severity: this.getSeverity("performance", auditResult.performance),
      });
    }

    // Check Core Web Vitals
    if (budget.lcp && auditResult.metrics.lcp > budget.lcp) {
      violations.push({
        type: "lcp",
        expected: budget.lcp,
        actual: auditResult.metrics.lcp,
        severity: this.getSeverity("lcp", auditResult.metrics.lcp),
      });
    }

    if (budget.fcp && auditResult.metrics.fcp > budget.fcp) {
      violations.push({
        type: "fcp",
        expected: budget.fcp,
        actual: auditResult.metrics.fcp,
        severity: this.getSeverity("fcp", auditResult.metrics.fcp),
      });
    }

    if (budget.cls && auditResult.metrics.cls > budget.cls) {
      violations.push({
        type: "cls",
        expected: budget.cls,
        actual: auditResult.metrics.cls,
        severity: this.getSeverity("cls", auditResult.metrics.cls),
      });
    }

    if (budget.fid && auditResult.metrics.fid > budget.fid) {
      violations.push({
        type: "fid",
        expected: budget.fid,
        actual: auditResult.metrics.fid,
        severity: this.getSeverity("fid", auditResult.metrics.fid),
      });
    }

    return {
      passed: violations.length === 0,
      violations,
      score: this.calculateBudgetScore(violations),
    };
  }

  private getSeverity(
    metric: string,
    value: number
  ): "warning" | "critical" | "ok" {
    const thresholds = PERFORMANCE_BUDGET.alerts[metric];
    if (!thresholds) return "ok";

    if (metric === "performance") {
      if (value < thresholds.critical) return "critical";
      if (value < thresholds.warning) return "warning";
    } else {
      if (value > thresholds.critical) return "critical";
      if (value > thresholds.warning) return "warning";
    }

    return "ok";
  }

  private calculateBudgetScore(violations: any[]): number {
    if (violations.length === 0) return 100;

    const criticalCount = violations.filter(
      (v) => v.severity === "critical"
    ).length;
    const warningCount = violations.filter(
      (v) => v.severity === "warning"
    ).length;

    let score = 100;
    score -= criticalCount * 20; // -20 points per critical violation
    score -= warningCount * 5; // -5 points per warning violation

    return Math.max(0, score);
  }
}
```

## 📈 Regression Testing

### Automated Performance Testing

```typescript
// tools/performance-regression.ts
import { PerformanceAuditService } from "./performance-audit";
import { PerformanceBudgetService } from "./performance-budget";

export class PerformanceRegressionService {
  private auditService = new PerformanceAuditService();
  private budgetService = new PerformanceBudgetService();

  async checkRegression(baselineFile: string, currentUrls: string[]) {
    console.log("🔍 Running regression analysis...");

    // Load baseline data
    const baseline = await this.loadBaseline(baselineFile);

    // Run current audits
    const current = await this.auditService.auditMultiplePages(currentUrls);

    // Compare results
    const regression = this.compareResults(baseline, current);

    // Generate regression report
    const report = this.generateRegressionReport(regression);

    return report;
  }

  private async loadBaseline(filePath: string) {
    const fs = await import("fs");

    if (!fs.existsSync(filePath)) {
      throw new Error(`Baseline file not found: ${filePath}`);
    }

    const content = fs.readFileSync(filePath, "utf8");
    return JSON.parse(content);
  }

  private compareResults(baseline: any[], current: any[]) {
    const comparisons = [];

    current.forEach((currentResult) => {
      const baselineResult = baseline.find((b) => b.url === currentResult.url);

      if (!baselineResult) {
        comparisons.push({
          url: currentResult.url,
          status: "new",
          current: currentResult,
        });
        return;
      }

      const comparison = {
        url: currentResult.url,
        status: "compared",
        baseline: baselineResult,
        current: currentResult,
        diff: this.calculateDifferences(baselineResult, currentResult),
      };

      comparisons.push(comparison);
    });

    return comparisons;
  }

  private calculateDifferences(baseline: any, current: any) {
    const diff = {
      performance: current.performance - baseline.performance,
      accessibility: current.accessibility - baseline.accessibility,
      bestPractices: current.bestPractices - baseline.bestPractices,
      seo: current.seo - baseline.seo,
      pwa: current.pwa - baseline.pwa,
      metrics: {
        fcp: current.metrics.fcp - baseline.metrics.fcp,
        lcp: current.metrics.lcp - baseline.metrics.lcp,
        cls: current.metrics.cls - baseline.metrics.cls,
        fid: current.metrics.fid - baseline.metrics.fid,
        ttfb: current.metrics.ttfb - baseline.metrics.ttfb,
      },
    };

    // Determine if this is a regression
    diff.isRegression =
      diff.performance < -5 || // 5 point drop in performance
      diff.metrics.fcp > 200 || // 200ms increase in FCP
      diff.metrics.lcp > 500 || // 500ms increase in LCP
      diff.metrics.cls > 0.05 || // 0.05 increase in CLS
      diff.metrics.fid > 50; // 50ms increase in FID

    return diff;
  }

  private generateRegressionReport(comparisons: any[]) {
    const regressions = comparisons.filter((c) => c.diff?.isRegression);
    const improvements = comparisons.filter(
      (c) =>
        c.diff &&
        !c.diff.isRegression &&
        (c.diff.performance > 2 ||
          c.diff.metrics.fcp < -100 ||
          c.diff.metrics.lcp < -200)
    );

    const report = {
      summary: {
        total: comparisons.length,
        regressions: regressions.length,
        improvements: improvements.length,
        stable: comparisons.length - regressions.length - improvements.length,
      },
      regressions: regressions.map((r) => ({
        url: r.url,
        performanceDrop: r.diff.performance,
        metricChanges: r.diff.metrics,
        severity: this.calculateRegressionSeverity(r.diff),
      })),
      improvements: improvements.map((i) => ({
        url: i.url,
        performanceGain: i.diff.performance,
        metricImprovements: i.diff.metrics,
      })),
      recommendations: this.generateRegressionRecommendations(regressions),
    };

    return report;
  }

  private calculateRegressionSeverity(
    diff: any
  ): "low" | "medium" | "high" | "critical" {
    if (diff.performance < -15 || diff.metrics.lcp > 1000) return "critical";
    if (diff.performance < -10 || diff.metrics.lcp > 500) return "high";
    if (diff.performance < -5 || diff.metrics.lcp > 200) return "medium";
    return "low";
  }

  private generateRegressionRecommendations(regressions: any[]) {
    const recommendations = [];

    regressions.forEach((regression) => {
      const diff = regression.diff;

      if (diff.metrics.fcp > 200) {
        recommendations.push({
          type: "fcp_regression",
          description: "First Contentful Paint has regressed significantly",
          action:
            "Check for new render-blocking resources or increased server response time",
        });
      }

      if (diff.metrics.lcp > 500) {
        recommendations.push({
          type: "lcp_regression",
          description: "Largest Contentful Paint has regressed",
          action: "Review image optimizations and critical resource loading",
        });
      }

      if (diff.metrics.cls > 0.05) {
        recommendations.push({
          type: "cls_regression",
          description: "Cumulative Layout Shift has increased",
          action:
            "Check for dynamic content insertion or missing size attributes",
        });
      }
    });

    return recommendations;
  }
}
```

## 🎯 Checklist de Implementación Completo

### ✅ **Performance Monitoring Setup**

- [ ] Servicio base de analytics configurado
- [ ] Web Vitals tracking activo
- [ ] Google Analytics 4 integrado
- [ ] Dashboard de performance implementado
- [ ] Sistema de alertas configurado
- [ ] Monitoreo de iconos específico
- [ ] Métricas de negocio (e-commerce) tracked

### ✅ **Lighthouse Integration**

- [ ] Lighthouse CI configurado
- [ ] Performance budgets definidos
- [ ] Regression testing automatizado
- [ ] Reports automáticos en PRs
- [ ] Integration con Slack/Email

### ✅ **Advanced Monitoring**

- [ ] Real User Monitoring (RUM) activo
- [ ] Performance budget monitoring
- [ ] Automated regression detection
- [ ] Cross-browser performance tracking
- [ ] Mobile vs Desktop performance comparison

---

**🎯 Próximo paso**: Implementa el monitoreo gradualmente, empezando por Core Web Vitals y expandiendo a métricas de negocio avanzado.
