# 🏗️ Arquitectura NX para E-commerce

Guía completa de organización y estructura NX optimizada para aplicaciones e-commerce Angular con sistema de iconos escalable y performance máxima.

## 🎯 Filosofía de Arquitectura

### Principios Fundamentales

1. **Separation of Concerns**: Cada librería tiene una responsabilidad específica
2. **Domain-Driven Design**: Organización por dominios de negocio
3. **Performance First**: Optimización desde la arquitectura
4. **Reusabilidad**: Componentes y servicios compartidos eficientemente

## 📁 Estructura Completa del Workspace

```
ecommerce-workspace/
├── apps/
│   ├── ecommerce-angular/              # 🛍️ App principal del cliente
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── home/               # Página home con iconos críticos inline
│   │   │   │   ├── products/           # Catálogo de productos
│   │   │   │   ├── checkout/           # Proceso de compra
│   │   │   │   ├── shared/             # Componentes específicos del cliente
│   │   │   │   └── app.config.ts       # Configuración Angular 17+
│   │   │   ├── assets/                 # Assets específicos del cliente
│   │   │   └── index.html              # HTML con iconos críticos inline
│   │   ├── project.json
│   │   └── tsconfig.app.json
│   │
│   ├── admin-dashboard/                # 🔧 Panel de administración
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── products-management/
│   │   │   │   ├── orders-management/
│   │   │   │   ├── analytics/
│   │   │   │   └── shared/
│   │   │   └── assets/
│   │   └── project.json
│   │
│   └── cms-dashboard/                  # 📝 Sistema de gestión de contenidos
│       ├── src/
│       │   ├── app/
│       │   │   ├── content-editor/
│       │   │   ├── media-management/
│       │   │   └── shared/
│       │   └── assets/
│       └── project.json
│
├── libs/
│   ├── icons/                          # 🎨 Sistema de iconos centralizado
│   │   ├── src/
│   │   │   ├── lib/
│   │   │   │   ├── components/
│   │   │   │   │   ├── icon.component.ts           # Wrapper para sprites
│   │   │   │   │   └── icon.component.spec.ts
│   │   │   │   ├── config/
│   │   │   │   │   └── icon.config.ts              # Config CDN y fallbacks
│   │   │   │   ├── types/
│   │   │   │   │   └── icon.types.ts               # Types generados automáticamente
│   │   │   │   ├── providers/
│   │   │   │   │   └── icon.providers.ts           # Angular providers
│   │   │   │   └── references/
│   │   │   │       └── critical-icons.md           # Referencia SVGs críticos
│   │   │   ├── assets/
│   │   │   │   └── individual/                     # SVGs fuente organizados
│   │   │   │       ├── core/                       # Iconos base (user, settings, etc)
│   │   │   │       ├── home/                       # Iconos específicos del cliente
│   │   │   │       ├── admin/                      # Iconos del panel admin
│   │   │   │       └── cms/                        # Iconos del CMS
│   │   │   └── tools/                              # Scripts de generación
│   │   │       ├── generate-sprites.mjs
│   │   │       ├── generate-types.mjs
│   │   │       ├── validate-icons.mjs
│   │   │       └── preview-icons.mjs
│   │   ├── README.md
│   │   └── project.json
│   │
│   ├── ui/                             # 🧩 Componentes UI reutilizables
│   │   ├── src/
│   │   │   ├── lib/
│   │   │   │   ├── button/
│   │   │   │   │   ├── button.component.ts
│   │   │   │   │   ├── button.component.spec.ts
│   │   │   │   │   └── button.stories.ts           # Storybook
│   │   │   │   ├── card/
│   │   │   │   ├── modal/
│   │   │   │   ├── form-controls/
│   │   │   │   └── layout/
│   │   │   └── index.ts                            # Barrel exports
│   │   └── project.json
│   │
│   ├── shared/                         # 🔄 Utilidades compartidas
│   │   ├── data-access/                # Services y state management
│   │   │   ├── src/
│   │   │   │   ├── lib/
│   │   │   │   │   ├── services/
│   │   │   │   │   │   ├── api.service.ts
│   │   │   │   │   │   ├── cart.service.ts
│   │   │   │   │   │   └── auth.service.ts
│   │   │   │   │   ├── state/                      # NgRx o signals
│   │   │   │   │   │   ├── cart/
│   │   │   │   │   │   ├── user/
│   │   │   │   │   │   └── products/
│   │   │   │   │   └── interceptors/
│   │   │   │   └── index.ts
│   │   │   └── project.json
│   │   │
│   │   ├── utils/                      # Utilidades puras
│   │   │   ├── src/
│   │   │   │   ├── lib/
│   │   │   │   │   ├── formatters/
│   │   │   │   │   ├── validators/
│   │   │   │   │   ├── constants/
│   │   │   │   │   └── helpers/
│   │   │   │   └── index.ts
│   │   │   └── project.json
│   │   │
│   │   └── types/                      # Interfaces TypeScript compartidas
│   │       ├── src/
│   │       │   ├── lib/
│   │       │   │   ├── api.types.ts
│   │       │   │   ├── user.types.ts
│   │       │   │   ├── product.types.ts
│   │       │   │   └── cart.types.ts
│   │       │   └── index.ts
│   │       └── project.json
│   │
│   ├── analytics/                      # 📊 Sistema de métricas y performance
│   │   ├── src/
│   │   │   ├── lib/
│   │   │   │   ├── web-vitals.service.ts
│   │   │   │   ├── performance-observer.service.ts
│   │   │   │   ├── business-metrics.service.ts
│   │   │   │   └── providers/
│   │   │   │       └── analytics.providers.ts
│   │   │   └── index.ts
│   │   └── project.json
│   │
│   └── feature/                        # 🚀 Features específicas por dominio
│       ├── products/                   # Feature de productos
│       │   ├── src/
│       │   │   ├── lib/
│       │   │   │   ├── components/
│       │   │   │   │   ├── product-card/
│       │   │   │   │   ├── product-grid/
│       │   │   │   │   └── product-filter/
│       │   │   │   ├── pages/
│       │   │   │   │   ├── product-list.component.ts
│       │   │   │   │   └── product-detail.component.ts
│       │   │   │   └── services/
│       │   │   │       └── products.service.ts
│       │   │   └── index.ts
│       │   └── project.json
│       │
│       ├── cart/                       # Feature del carrito
│       │   ├── src/
│       │   │   ├── lib/
│       │   │   │   ├── components/
│       │   │   │   ├── pages/
│       │   │   │   └── services/
│       │   │   └── index.ts
│       │   └── project.json
│       │
│       └── checkout/                   # Feature de checkout
│           ├── src/
│           │   ├── lib/
│           │   │   ├── components/
│           │   │   ├── pages/
│           │   │   └── services/
│           │   └── index.ts
│           └── project.json
│
├── tools/
│   ├── scripts/                        # Scripts de automatización
│   │   ├── deploy-icons.mjs
│   │   ├── generate-manifest.mjs
│   │   └── performance-audit.mjs
│   │
│   ├── generators/                     # Generadores personalizados NX
│   │   ├── icon-component/
│   │   ├── feature-module/
│   │   └── performance-page/
│   │
│   └── executors/                      # Executors personalizados
│       ├── icon-build/
│       └── performance-test/
│
├── dist/                               # Builds generados
│   ├── apps/
│   └── libs/
│
├── docs/                               # Documentación
│   ├── architecture/
│   ├── deployment/
│   └── guides/
│
├── nx.json                             # Configuración NX
├── package.json
├── tsconfig.base.json                  # TypeScript base config
├── .eslintrc.json
└── README.md
```

## 🔧 Configuración NX

### 1. nx.json Principal

```json
<!-- filepath: c:\Users\crist\Desktop\repositorios\estudio\ecommerce-guidelines\architecture\nx-structure.md -->
{
  "$schema": "./node_modules/nx/schemas/nx-schema.json",
  "npmScope": "acme",
  "affected": {
    "defaultBase": "origin/main"
  },
  "cli": {
    "defaultCollection": "@nx/angular"
  },
  "defaultProject": "ecommerce-angular",
  "generators": {
    "@nx/angular:application": {
      "style": "scss",
      "linter": "eslint",
      "unitTestRunner": "jest",
      "e2eTestRunner": "playwright",
      "strict": true,
      "standalone": true
    },
    "@nx/angular:library": {
      "style": "scss",
      "linter": "eslint",
      "unitTestRunner": "jest",
      "strict": true,
      "standalone": true
    },
    "@nx/angular:component": {
      "style": "scss",
      "changeDetection": "OnPush",
      "standalone": true
    }
  },
  "targetDefaults": {
    "build": {
      "dependsOn": ["^build"],
      "inputs": ["production", "^production"],
      "cache": true
    },
    "test": {
      "inputs": ["default", "^production", "{workspaceRoot}/jest.preset.js"],
      "cache": true
    },
    "lint": {
      "inputs": [
        "default",
        "{workspaceRoot}/.eslintrc.json",
        "{workspaceRoot}/.eslintignore"
      ],
      "cache": true
    },
    "e2e": {
      "inputs": ["default", "^production"],
      "cache": true
    }
  },
  "namedInputs": {
    "default": ["{projectRoot}/**/*", "sharedGlobals"],
    "production": [
      "default",
      "!{projectRoot}/**/?(*.)+(spec|test).[jt]s?(x)?(.snap)",
      "!{projectRoot}/tsconfig.spec.json",
      "!{projectRoot}/jest.config.[jt]s",
      "!{projectRoot}/.eslintrc.json",
      "!{projectRoot}/src/test-setup.[jt]s",
      "!{projectRoot}/test-setup.[jt]s"
    ],
    "sharedGlobals": []
  },
  "plugins": [
    {
      "plugin": "@nx/webpack/plugin",
      "options": {
        "buildTargetName": "build",
        "serveTargetName": "serve",
        "previewTargetName": "preview"
      }
    },
    {
      "plugin": "@nx/eslint/plugin",
      "options": {
        "targetName": "lint"
      }
    },
    {
      "plugin": "@nx/jest/plugin",
      "options": {
        "targetName": "test"
      }
    }
  ]
}
```

### 2. tsconfig.base.json con Path Mapping

```json
<!-- filepath: c:\Users\crist\Desktop\repositorios\estudio\ecommerce-guidelines\architecture\nx-structure.md -->
{
  "compileOnSave": false,
  "compilerOptions": {
    "rootDir": ".",
    "sourceMap": true,
    "declaration": false,
    "moduleResolution": "node",
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "importHelpers": true,
    "target": "ES2022",
    "module": "ES2022",
    "lib": ["ES2022", "dom"],
    "skipLibCheck": true,
    "skipDefaultLibCheck": true,
    "useDefineForClassFields": false,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "paths": {
      // ============ CORE LIBRARIES ============
      "@acme/icons": ["libs/icons/src/index.ts"],
      "@acme/ui": ["libs/ui/src/index.ts"],
      "@acme/analytics": ["libs/analytics/src/index.ts"],

      // ============ SHARED LIBRARIES ============
      "@acme/shared/data-access": ["libs/shared/data-access/src/index.ts"],
      "@acme/shared/utils": ["libs/shared/utils/src/index.ts"],
      "@acme/shared/types": ["libs/shared/types/src/index.ts"],

      // ============ FEATURE LIBRARIES ============
      "@acme/feature/products": ["libs/feature/products/src/index.ts"],
      "@acme/feature/cart": ["libs/feature/cart/src/index.ts"],
      "@acme/feature/checkout": ["libs/feature/checkout/src/index.ts"],
      "@acme/feature/auth": ["libs/feature/auth/src/index.ts"],
      "@acme/feature/user-profile": ["libs/feature/user-profile/src/index.ts"],

      // ============ APP-SPECIFIC PATHS ============
      "@ecommerce/shared": ["apps/ecommerce-angular/src/app/shared"],
      "@admin/shared": ["apps/admin-dashboard/src/app/shared"],
      "@cms/shared": ["apps/cms-dashboard/src/app/shared"]
    }
  },
  "exclude": ["node_modules", "tmp"]
}
```

### 3. Configuración de Apps

#### App Principal E-commerce

```json
<!-- filepath: c:\Users\crist\Desktop\repositorios\estudio\ecommerce-guidelines\architecture\nx-structure.md -->
// apps/ecommerce-angular/project.json
{
  "name": "ecommerce-angular",
  "$schema": "../../node_modules/nx/schemas/project-schema.json",
  "projectType": "application",
  "prefix": "app",
  "sourceRoot": "apps/ecommerce-angular/src",
  "tags": ["scope:ecommerce", "type:app"],
  "targets": {
    "build": {
      "executor": "@angular-devkit/build-angular:browser",
      "outputs": ["{options.outputPath}"],
      "options": {
        "outputPath": "dist/apps/ecommerce-angular",
        "index": "apps/ecommerce-angular/src/index.html",
        "main": "apps/ecommerce-angular/src/main.ts",
        "polyfills": ["zone.js"],
        "tsConfig": "apps/ecommerce-angular/tsconfig.app.json",
        "assets": [
          "apps/ecommerce-angular/src/favicon.ico",
          "apps/ecommerce-angular/src/assets",
          {
            "glob": "**/*",
            "input": "libs/icons/src/assets",
            "output": "/assets/icons/"
          }
        ],
        "styles": ["apps/ecommerce-angular/src/styles.scss"],
        "scripts": [],
        "vendorChunk": true,
        "extractLicenses": false,
        "buildOptimizer": false,
        "sourceMap": true,
        "optimization": false,
        "namedChunks": true
      },
      "configurations": {
        "production": {
          "budgets": [
            {
              "type": "initial",
              "maximumWarning": "500kb",
              "maximumError": "1mb"
            },
            {
              "type": "anyComponentStyle",
              "maximumWarning": "2kb",
              "maximumError": "4kb"
            }
          ],
          "outputHashing": "all",
          "sourceMap": false,
          "namedChunks": false,
          "extractLicenses": true,
          "vendorChunk": false,
          "buildOptimizer": true,
          "optimization": true,
          "serviceWorker": true,
          "ngswConfigPath": "apps/ecommerce-angular/ngsw-config.json"
        },
        "development": {
          "buildOptimizer": false,
          "optimization": false,
          "vendorChunk": true,
          "extractLicenses": false,
          "sourceMap": true,
          "namedChunks": true
        }
      },
      "defaultConfiguration": "production"
    },
    "serve": {
      "executor": "@angular-devkit/build-angular:dev-server",
      "configurations": {
        "production": {
          "buildTarget": "ecommerce-angular:build:production"
        },
        "development": {
          "buildTarget": "ecommerce-angular:build:development"
        }
      },
      "defaultConfiguration": "development"
    },
    "extract-i18n": {
      "executor": "@angular-devkit/build-angular:extract-i18n",
      "options": {
        "buildTarget": "ecommerce-angular:build"
      }
    },
    "lint": {
      "executor": "@nx/eslint:lint",
      "outputs": ["{options.outputFile}"],
      "options": {
        "lintFilePatterns": [
          "apps/ecommerce-angular/**/*.ts",
          "apps/ecommerce-angular/**/*.html"
        ]
      }
    },
    "test": {
      "executor": "@nx/jest:jest",
      "outputs": ["{workspaceRoot}/coverage/{projectRoot}"],
      "options": {
        "jestConfig": "apps/ecommerce-angular/jest.config.ts"
      }
    },
    "serve-static": {
      "executor": "@nx/web:file-server",
      "options": {
        "buildTarget": "ecommerce-angular:build",
        "port": 4200,
        "staticFilePath": "dist/apps/ecommerce-angular"
      }
    },
    // ============ CUSTOM TARGETS ============
    "performance-audit": {
      "executor": "@acme/tools:performance-audit",
      "options": {
        "url": "http://localhost:4200",
        "outputPath": "reports/lighthouse"
      }
    },
    "icons-precheck": {
      "executor": "@acme/tools:icons-precheck",
      "options": {
        "sourceDir": "apps/ecommerce-angular/src"
      }
    }
  }
}
```

#### Configuración de Icons Library

```json
<!-- filepath: c:\Users\crist\Desktop\repositorios\estudio\ecommerce-guidelines\architecture\nx-structure.md -->
// libs/icons/project.json
{
  "name": "icons",
  "$schema": "../../node_modules/nx/schemas/project-schema.json",
  "sourceRoot": "libs/icons/src",
  "prefix": "ui",
  "tags": ["scope:shared", "type:ui"],
  "projectType": "library",
  "targets": {
    "build": {
      "executor": "@nx/angular:ng-packagr-lite",
      "outputs": ["{workspaceRoot}/dist/libs/icons"],
      "options": {
        "project": "libs/icons/ng-package.json"
      }
    },
    "test": {
      "executor": "@nx/jest:jest",
      "outputs": ["{workspaceRoot}/coverage/libs/icons"],
      "options": {
        "jestConfig": "libs/icons/jest.config.ts"
      }
    },
    "lint": {
      "executor": "@nx/eslint:lint",
      "outputs": ["{options.outputFile}"],
      "options": {
        "lintFilePatterns": ["libs/icons/**/*.ts", "libs/icons/**/*.html"]
      }
    },
    "storybook": {
      "executor": "@storybook/angular:start-storybook",
      "options": {
        "port": 4400,
        "configDir": "libs/icons/.storybook"
      }
    },
    "build-storybook": {
      "executor": "@storybook/angular:build-storybook",
      "outputs": ["{options.outputDir}"],
      "options": {
        "outputDir": "dist/storybook/icons",
        "configDir": "libs/icons/.storybook"
      }
    },
    // ============ CUSTOM ICON TARGETS ============
    "generate-sprites": {
      "executor": "@acme/tools:icon-sprites",
      "outputs": ["dist/sprites"],
      "options": {
        "sourceDir": "libs/icons/src/assets/individual",
        "outputDir": "dist/sprites"
      }
    },
    "generate-types": {
      "executor": "@acme/tools:icon-types",
      "outputs": ["libs/icons/src/lib/types/icon.types.ts"],
      "options": {
        "sourceDir": "libs/icons/src/assets/individual",
        "outputFile": "libs/icons/src/lib/types/icon.types.ts"
      }
    },
    "validate-icons": {
      "executor": "@acme/tools:icon-validator",
      "options": {
        "sourceDir": "libs/icons/src/assets/individual"
      }
    },
    "preview-icons": {
      "executor": "@acme/tools:icon-preview",
      "options": {
        "sourceDir": "libs/icons/src/assets/individual",
        "port": 4201
      }
    },
    "icons-build": {
      "executor": "@nx/workspace:run-commands",
      "options": {
        "commands": [
          "nx validate-icons icons",
          "nx generate-sprites icons",
          "nx generate-types icons"
        ],
        "parallel": false
      }
    },
    "deploy-icons": {
      "executor": "@nx/workspace:run-commands",
      "dependsOn": ["icons-build"],
      "options": {
        "command": "node scripts/deploy-icons.mjs"
      }
    }
  }
}
```

## 🎛️ Generadores Personalizados

### 1. Generador de Feature Module

```typescript
// tools/generators/feature-module/index.ts
import {
  Tree,
  formatFiles,
  installPackagesTask,
  names,
  generateFiles,
  joinPathFragments,
} from "@nx/devkit";
import { libraryGenerator } from "@nx/angular/generators";

interface FeatureModuleGeneratorSchema {
  name: string;
  directory?: string;
  tags?: string;
  hasPages?: boolean;
  hasServices?: boolean;
  hasState?: boolean;
}

export default async function (
  tree: Tree,
  options: FeatureModuleGeneratorSchema
) {
  const normalizedOptions = normalizeOptions(options);

  // Generar la librería base
  await libraryGenerator(tree, {
    name: normalizedOptions.name,
    directory: normalizedOptions.directory,
    tags: normalizedOptions.tags,
    standalone: true,
    prefix: "app",
    style: "scss",
    changeDetection: "OnPush",
  });

  // Agregar estructura de feature
  addFeatureStructure(tree, normalizedOptions);

  // Agregar archivos específicos
  addFeatureFiles(tree, normalizedOptions);

  await formatFiles(tree);
  return () => {
    installPackagesTask(tree);
  };
}

function normalizeOptions(options: FeatureModuleGeneratorSchema) {
  const name = names(options.name).fileName;
  const projectDirectory = options.directory
    ? `${names(options.directory).fileName}/${name}`
    : name;

  return {
    ...options,
    name,
    projectRoot: `libs/feature/${projectDirectory}`,
    projectDirectory,
    tags: options.tags || `scope:feature,domain:${name}`,
  };
}

function addFeatureStructure(tree: Tree, options: any) {
  const templateOptions = {
    ...options,
    ...names(options.name),
    template: "",
  };

  generateFiles(
    tree,
    joinPathFragments(__dirname, "files"),
    options.projectRoot,
    templateOptions
  );
}

function addFeatureFiles(tree: Tree, options: any) {
  // Crear estructura de carpetas
  const basePath = `${options.projectRoot}/src/lib`;

  // Components folder
  if (!tree.exists(`${basePath}/components`)) {
    tree.write(`${basePath}/components/.gitkeep`, "");
  }

  // Pages folder si se requiere
  if (options.hasPages && !tree.exists(`${basePath}/pages`)) {
    tree.write(`${basePath}/pages/.gitkeep`, "");
  }

  // Services folder si se requiere
  if (options.hasServices && !tree.exists(`${basePath}/services`)) {
    tree.write(`${basePath}/services/.gitkeep`, "");
  }

  // State folder si se requiere
  if (options.hasState && !tree.exists(`${basePath}/state`)) {
    tree.write(`${basePath}/state/.gitkeep`, "");
  }
}
```

### 2. Generador de Componente Icon

```typescript
// tools/generators/icon-component/index.ts
import {
  Tree,
  formatFiles,
  names,
  generateFiles,
  joinPathFragments,
} from "@nx/devkit";

interface IconComponentGeneratorSchema {
  name: string;
  project: string;
  section: "core" | "home" | "admin" | "cms";
  critical?: boolean;
  export?: boolean;
}

export default async function (
  tree: Tree,
  options: IconComponentGeneratorSchema
) {
  const normalizedOptions = normalizeOptions(options);

  if (options.critical) {
    // Para iconos críticos, generar solo referencia
    generateCriticalIconReference(tree, normalizedOptions);
  } else {
    // Para iconos no críticos, generar SVG en assets
    generateIconAsset(tree, normalizedOptions);
  }

  await formatFiles(tree);
}

function normalizeOptions(options: IconComponentGeneratorSchema) {
  return {
    ...options,
    ...names(options.name),
  };
}

function generateCriticalIconReference(tree: Tree, options: any) {
  const referencePath = "libs/icons/src/lib/references/critical-icons.md";
  const existingContent = tree.read(referencePath)?.toString() || "";

  const newIconReference = `
## ${options.className}

\`\`\`svg
<!-- TODO: Agregar SVG para ${options.name} -->
<svg viewBox="0 0 24 24" width="24" height="24">
  <!-- SVG paths aquí -->
</svg>
\`\`\`

**Uso**: Copiar directamente en el template del componente.
**Ubicación**: Above-the-fold / Crítico para FCP
`;

  tree.write(referencePath, existingContent + newIconReference);

  console.log(`✅ Referencia de icono crítico agregada para: ${options.name}`);
  console.log(`📝 Edita: ${referencePath}`);
}

function generateIconAsset(tree: Tree, options: any) {
  const assetPath = `libs/icons/src/assets/individual/${options.section}/${options.fileName}.svg`;

  const svgTemplate = `<svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
  <!-- TODO: Agregar paths SVG para ${options.name} -->
  <path d="M12 2L2 7v10c0 5.55 3.84 10.74 9 12 5.16-1.26 9-6.45 9-12V7l-10-5z" fill="currentColor"/>
</svg>`;

  tree.write(assetPath, svgTemplate);

  console.log(`✅ Asset de icono creado: ${assetPath}`);
  console.log(`🔧 Ejecuta: nx icons-build icons para regenerar sprites`);
}
```

## 🚀 Executors Personalizados

### 1. Executor de Performance Audit

```typescript
// tools/executors/performance-audit/executor.ts
import { ExecutorContext } from "@nx/devkit";
import { exec } from "child_process";
import { promisify } from "util";
import { writeFileSync, mkdirSync } from "fs";
import chalk from "chalk";

const execAsync = promisify(exec);

export interface PerformanceAuditExecutorSchema {
  url: string;
  outputPath: string;
  device?: "mobile" | "desktop";
  throttling?: "slow-3g" | "4g" | "none";
  thresholds?: {
    performance?: number;
    fcp?: number;
    lcp?: number;
    cls?: number;
  };
}

export default async function runExecutor(
  options: PerformanceAuditExecutorSchema,
  context: ExecutorContext
) {
  console.log(chalk.blue("🚀 Starting performance audit..."));

  const { url, outputPath, device = "desktop", throttling = "none" } = options;
  const thresholds = {
    performance: 90,
    fcp: 1500,
    lcp: 2500,
    cls: 0.1,
    ...options.thresholds,
  };

  try {
    // Crear directorio de salida
    mkdirSync(outputPath, { recursive: true });

    // Configurar flags de Chrome
    const chromeFlags = [
      "--headless",
      "--no-sandbox",
      "--disable-dev-shm-usage",
    ];

    if (device === "mobile") {
      chromeFlags.push("--window-size=375,667");
    }

    // Ejecutar Lighthouse
    const lighthouseCommand = [
      "lighthouse",
      url,
      `--chrome-flags="${chromeFlags.join(" ")}"`,
      "--output=json",
      "--output=html",
      `--output-path=${outputPath}/lighthouse-report`,
      `--preset=${device}`,
      "--quiet",
    ].join(" ");

    console.log(chalk.gray(`Running: ${lighthouseCommand}`));

    const { stdout } = await execAsync(lighthouseCommand);
    const report = JSON.parse(stdout);

    // Analizar resultados
    const results = analyzeResults(report, thresholds);

    // Generar reporte
    generateReport(results, outputPath);

    // Mostrar resultados en consola
    displayResults(results);

    return {
      success: results.passed,
      results,
    };
  } catch (error) {
    console.error(chalk.red("❌ Performance audit failed:"), error);
    return {
      success: false,
      error: error.message,
    };
  }
}

function analyzeResults(report: any, thresholds: any) {
  const scores = report.categories;
  const audits = report.audits;

  const results = {
    performance: Math.round(scores.performance.score * 100),
    fcp: audits["first-contentful-paint"].numericValue,
    lcp: audits["largest-contentful-paint"].numericValue,
    cls: audits["cumulative-layout-shift"].numericValue,
    tbt: audits["total-blocking-time"].numericValue,
    passed: true,
    issues: [],
  };

  // Verificar thresholds
  if (results.performance < thresholds.performance) {
    results.passed = false;
    results.issues.push(
      `Performance score ${results.performance} < ${thresholds.performance}`
    );
  }

  if (results.fcp > thresholds.fcp) {
    results.passed = false;
    results.issues.push(
      `FCP ${Math.round(results.fcp)}ms > ${thresholds.fcp}ms`
    );
  }

  if (results.lcp > thresholds.lcp) {
    results.passed = false;
    results.issues.push(
      `LCP ${Math.round(results.lcp)}ms > ${thresholds.lcp}ms`
    );
  }

  if (results.cls > thresholds.cls) {
    results.passed = false;
    results.issues.push(`CLS ${results.cls.toFixed(3)} > ${thresholds.cls}`);
  }

  return results;
}

function generateReport(results: any, outputPath: string) {
  const reportData = {
    timestamp: new Date().toISOString(),
    ...results,
  };

  writeFileSync(
    `${outputPath}/performance-summary.json`,
    JSON.stringify(reportData, null, 2)
  );
}

function displayResults(results: any) {
  console.log(chalk.blue("\n📊 Performance Results:"));
  console.log("─".repeat(50));

  const performanceColor =
    results.performance >= 90
      ? chalk.green
      : results.performance >= 70
      ? chalk.yellow
      : chalk.red;

  console.log(`Performance: ${performanceColor(results.performance + "/100")}`);
  console.log(`FCP: ${formatTime(results.fcp)}`);
  console.log(`LCP: ${formatTime(results.lcp)}`);
  console.log(`CLS: ${results.cls.toFixed(3)}`);
  console.log(`TBT: ${Math.round(results.tbt)}ms`);

  if (results.passed) {
    console.log(chalk.green("\n✅ All thresholds passed!"));
  } else {
    console.log(chalk.red("\n❌ Issues found:"));
    results.issues.forEach((issue: string) => {
      console.log(chalk.red(`  • ${issue}`));
    });
  }
}

function formatTime(ms: number): string {
  const seconds = (ms / 1000).toFixed(1);
  const color = ms < 1000 ? chalk.green : ms < 2500 ? chalk.yellow : chalk.red;
  return color(seconds + "s");
}
```

### 2. Executor de Icon Build

```typescript
// tools/executors/icon-build/executor.ts
import { ExecutorContext } from "@nx/devkit";
import { exec } from "child_process";
import { promisify } from "util";
import chalk from "chalk";

const execAsync = promisify(exec);

export interface IconBuildExecutorSchema {
  sourceDir: string;
  outputDir: string;
  validate?: boolean;
  generateTypes?: boolean;
  deploy?: boolean;
}

export default async function runExecutor(
  options: IconBuildExecutorSchema,
  context: ExecutorContext
) {
  console.log(chalk.blue("🎨 Building icon system..."));

  try {
    // 1. Validar iconos si está habilitado
    if (options.validate !== false) {
      console.log(chalk.gray("Validating icons..."));
      await execAsync("node libs/icons/tools/validate-icons.mjs");
      console.log(chalk.green("✅ Icons validated"));
    }

    // 2. Generar sprites
    console.log(chalk.gray("Generating sprites..."));
    await execAsync("node libs/icons/tools/generate-sprites.mjs");
    console.log(chalk.green("✅ Sprites generated"));

    // 3. Generar tipos si está habilitado
    if (options.generateTypes !== false) {
      console.log(chalk.gray("Generating TypeScript types..."));
      await execAsync("node libs/icons/tools/generate-types.mjs");
      console.log(chalk.green("✅ Types generated"));
    }

    // 4. Deploy si está habilitado
    if (options.deploy) {
      console.log(chalk.gray("Deploying to CDN..."));
      await execAsync("node scripts/deploy-icons.mjs");
      console.log(chalk.green("✅ Icons deployed to CDN"));
    }

    console.log(chalk.green("\n🎉 Icon build completed successfully!"));

    return { success: true };
  } catch (error) {
    console.error(chalk.red("❌ Icon build failed:"), error.message);
    return {
      success: false,
      error: error.message,
    };
  }
}
```

## 🔄 Dependency Graph y Tags

### 1. Sistema de Tags

```json
<!-- filepath: c:\Users\crist\Desktop\repositorios\estudio\ecommerce-guidelines\architecture\nx-structure.md -->
// Estrategia de tags para control de dependencias
{
  "apps": {
    "ecommerce-angular": ["scope:ecommerce", "type:app"],
    "admin-dashboard": ["scope:admin", "type:app"],
    "cms-dashboard": ["scope:cms", "type:app"]
  },
  "libs": {
    "icons": ["scope:shared", "type:ui"],
    "ui": ["scope:shared", "type:ui"],
    "analytics": ["scope:shared", "type:util"],
    "shared/data-access": ["scope:shared", "type:data-access"],
    "shared/utils": ["scope:shared", "type:util"],
    "shared/types": ["scope:shared", "type:types"],
    "feature/products": ["scope:feature", "domain:products"],
    "feature/cart": ["scope:feature", "domain:cart"],
    "feature/checkout": ["scope:feature", "domain:checkout"]
  }
}
```

### 2. Reglas de Dependencias

```json
<!-- filepath: c:\Users\crist\Desktop\repositorios\estudio\ecommerce-guidelines\architecture\nx-structure.md -->
// .eslintrc.json
{
  "extends": ["@nx/eslint-plugin"],
  "rules": {
    "@nx/enforce-module-boundaries": [
      "error",
      {
        "enforceBuildableLibDependency": true,
        "allow": [],
        "depConstraints": [
          // Apps pueden importar cualquier lib
          {
            "sourceTag": "type:app",
            "onlyDependOnLibsWithTags": ["*"]
          },
          // Libs shared solo pueden depender de otras shared
          {
            "sourceTag": "scope:shared",
            "onlyDependOnLibsWithTags": ["scope:shared"]
          },
          // Features pueden usar shared pero no otras features
          {
            "sourceTag": "scope:feature",
            "onlyDependOnLibsWithTags": ["scope:shared"]
          },
          // UI libs no pueden depender de data-access
          {
            "sourceTag": "type:ui",
            "bannedExternalImports": ["@acme/shared/data-access"]
          },
          // Data-access puede importar types y utils
          {
            "sourceTag": "type:data-access",
            "onlyDependOnLibsWithTags": ["type:types", "type:util"]
          }
        ]
      }
    ]
  }
}
```

## 📝 Comandos NX Útiles

### Scripts de Package.json

```json
<!-- filepath: c:\Users\crist\Desktop\repositorios\estudio\ecommerce-guidelines\architecture\nx-structure.md -->
{
  "scripts": {
    // ============ DESARROLLO ============
    "start": "nx serve ecommerce-angular",
    "start:admin": "nx serve admin-dashboard",
    "start:cms": "nx serve cms-dashboard",

    // ============ BUILD ============
    "build": "nx build ecommerce-angular",
    "build:all": "nx run-many --target=build --all",
    "build:affected": "nx affected --target=build",

    // ============ TESTING ============
    "test": "nx test",
    "test:all": "nx run-many --target=test --all",
    "test:affected": "nx affected --target=test",
    "test:coverage": "nx run-many --target=test --all --coverage",
    "e2e": "nx e2e ecommerce-angular-e2e",

    // ============ LINTING ============
    "lint": "nx run-many --target=lint --all",
    "lint:fix": "nx run-many --target=lint --all --fix",

    // ============ ICONOS ============
    "icons:build": "nx icons-build icons",
    "icons:validate": "nx validate-icons icons",
    "icons:preview": "nx preview-icons icons",
    "icons:deploy": "nx deploy-icons icons",

    // ============ PERFORMANCE ============
    "perf:audit": "nx performance-audit ecommerce-angular",
    "perf:audit:all": "nx run-many --target=performance-audit --projects=ecommerce-angular,admin-dashboard",

    // ============ GENERADORES ============
    "g:feature": "nx g @acme/tools:feature-module",
    "g:icon": "nx g @acme/tools:icon-component",
    "g:component": "nx g @nx/angular:component",

    // ============ ANÁLISIS ============
    "dep-graph": "nx graph",
    "affected:graph": "nx affected:graph",
    "bundle-analyzer": "nx build ecommerce-angular --stats-json && npx webpack-bundle-analyzer dist/apps/ecommerce-angular/stats.json",

    // ============ WORKSPACE ============
    "workspace-generator": "nx workspace-generator",
    "migrate": "nx migrate latest && npm install && nx migrate --run-migrations",
    "reset": "nx reset"
  }
}
```

### Comandos Útiles de NX

```bash
# ============ DESARROLLO ============
# Servir app específica
nx serve ecommerce-angular
nx serve admin-dashboard --port=4201

# Servir con configuración específica
nx serve ecommerce-angular --configuration=development
nx serve ecommerce-angular --configuration=production

# ============ BUILD Y DEPLOY ============
# Build de producción
nx build ecommerce-angular --configuration=production

# Build de todas las apps
nx run-many --target=build --projects=ecommerce-angular,admin-dashboard,cms-dashboard

# Build solo de lo que cambió
nx affected --target=build --base=main

# ============ TESTING ============
# Test con coverage
nx test icons --coverage
nx test ecommerce-angular --watch

# Test E2E
nx e2e ecommerce-angular-e2e
nx e2e ecommerce-angular-e2e --headed

# ============ GENERADORES ============
# Generar nueva feature library
nx g @acme/tools:feature-module products --hasPages --hasServices --hasState

# Generar componente
nx g @nx/angular:component product-card --project=feature-products --standalone

# Generar icono crítico
nx g @acme/tools:icon-component logo --critical --section=core

# Generar icono no crítico
nx g @acme/tools:icon-component heart --section=home

# ============ ICONOS ============
# Build completo del sistema de iconos
nx icons-build icons

# Solo validar iconos
nx validate-icons icons

# Solo generar sprites
nx generate-sprites icons

# Preview de iconos
nx preview-icons icons

# Deploy a CDN
nx deploy-icons icons

# ============ PERFORMANCE ============
# Audit de performance
nx performance-audit ecommerce-angular --url=http://localhost:4200

# Audit con thresholds personalizados
nx performance-audit ecommerce-angular --url=http://localhost:4200 --device=mobile --thresholds='{"performance": 95, "fcp": 1000}'

# ============ ANÁLISIS ============
# Ver dependency graph
nx graph

# Ver dependency graph solo de iconos
nx graph --focus=icons

# Ver qué se ve afectado por cambios
nx affected:graph

# Analizar bundle size
nx build ecommerce-angular --stats-json
npx webpack-bundle-analyzer dist/apps/ecommerce-angular/stats.json

# ============ WORKSPACE MAINTENANCE ============
# Limpiar cache
nx reset

# Actualizar dependencias NX
nx migrate latest
npm install
nx migrate --run-migrations

# Lint de todo el workspace
nx run-many --target=lint --all --fix

# Verificar reglas de dependencias
nx run-many --target=lint --all --verbose
```

## 📊 Optimizaciones de Build

### 1. Webpack Optimizations

```typescript
// tools/webpack/webpack.config.js
const { composePlugins, withNx } = require("@nx/webpack");
const { BundleAnalyzerPlugin } = require("webpack-bundle-analyzer");

module.exports = composePlugins(withNx(), (config, { options, context }) => {
  // Optimizaciones para producción
  if (options.configuration === "production") {
    // Code splitting optimizado
    config.optimization = {
      ...config.optimization,
      splitChunks: {
        chunks: "all",
        cacheGroups: {
          vendor: {
            test: /[\\/]node_modules[\\/]/,
            name: "vendors",
            chunks: "all",
            priority: 10,
          },
          common: {
            name: "common",
            minChunks: 2,
            chunks: "all",
            priority: 5,
          },
          icons: {
            test: /[\\/]libs[\\/]icons[\\/]/,
            name: "icons",
            chunks: "all",
            priority: 8,
          },
        },
      },
    };

    // Comprimir iconos SVG
    config.module.rules.push({
      test: /\.svg$/,
      include: /assets\/icons/,
      use: [
        {
          loader: "svgo-loader",
          options: {
            plugins: [
              { name: "preset-default" },
              { name: "removeViewBox", active: false },
              { name: "removeDimensions", active: true },
            ],
          },
        },
      ],
    });
  }

  // Bundle analyzer en desarrollo
  if (process.env.ANALYZE) {
    config.plugins.push(
      new BundleAnalyzerPlugin({
        analyzerMode: "server",
        openAnalyzer: true,
      })
    );
  }

  return config;
});
```

### 2. Build Optimizations

```json
<!-- filepath: c:\Users\crist\Desktop\repositorios\estudio\ecommerce-guidelines\architecture\nx-structure.md -->
// angular.json optimizations
{
  "build": {
    "configurations": {
      "production": {
        "optimization": {
          "scripts": true,
          "styles": {
            "minify": true,
            "inlineCritical": true
          },
          "fonts": {
            "inline": true
          }
        },
        "buildOptimizer": true,
        "aot": true,
        "extractLicenses": true,
        "vendorChunk": false,
        "namedChunks": false,
        "outputHashing": "all",
        "sourceMap": false,
        "budgets": [
          {
            "type": "initial",
            "maximumWarning": "500kb",
            "maximumError": "1mb"
          },
          {
            "type": "anyComponentStyle",
            "maximumWarning": "2kb",
            "maximumError": "4kb"
          }
        ]
      }
    }
  }
}
```

## 🎯 Checklist de Setup

- [ ] **Workspace Base**

  - [ ] NX workspace inicializado
  - [ ] tsconfig.base.json configurado con paths
  - [ ] ESLint con reglas de módulos configurado
  - [ ] Tags system implementado

- [ ] **Aplicaciones**

  - [ ] App principal e-commerce configurada
  - [ ] Admin dashboard configurada
  - [ ] CMS dashboard configurada

- [ ] **Libraries**

  - [ ] Icons library con estructura completa
  - [ ] UI library con componentes base
  - [ ] Shared libraries organizadas
  - [ ] Feature libraries por dominio

- [ ] **Herramientas**

  - [ ] Generadores personalizados
  - [ ] Executors personalizados
  - [ ] Scripts de automatización
  - [ ] Performance auditing

- [ ] **CI/CD**
  - [ ] Build matrix configurado
  - [ ] Testing automatizado
  - [ ] Deploy automático
  - [ ] Dependency checking

---

**Siguiente**: [Angular SSR Optimization](./ssr-optimization.md)
