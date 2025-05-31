# 🛠️ Scripts y Automatización

Scripts NX automatizados para generación de iconos, validación, deploy y herramientas de desarrollo para aplicaciones e-commerce Angular.

## 📋 Scripts Principales

```json
{
  "scripts": {
    "icons:build": "npm run icons:validate && npm run icons:generate && npm run icons:types",
    "icons:generate": "node libs/icons/tools/generate-sprites.mjs",
    "icons:validate": "node libs/icons/tools/validate-icons.mjs",
    "icons:types": "node libs/icons/tools/generate-types.mjs",
    "icons:deploy": "node scripts/deploy-icons.mjs",
    "icons:preview": "node libs/icons/tools/preview-icons.mjs",
    "icons:deploy:staging": "ENVIRONMENT=staging npm run icons:deploy"
  }
}
```

## 🎨 Scripts de Iconos

### 1. Generador de Sprites

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

### 2. Validador de Iconos

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

### 3. Generador de Tipos TypeScript

```javascript
// libs/icons/tools/generate-types.mjs
import { readdirSync, writeFileSync } from "fs";
import { join } from "path";

function generateTypes() {
  console.log("📝 Generating TypeScript types for icons...\n");

  const sections = ["core", "home", "admin", "cms"];
  const types = {
    core: [],
    home: [],
    admin: [],
    cms: [],
  };

  // Collect icon names from each section
  for (const section of sections) {
    const path = join("libs/icons/src/assets/individual", section);
    try {
      const icons = readdirSync(path)
        .filter((f) => f.endsWith(".svg"))
        .map((f) => f.replace(".svg", ""))
        .sort();

      types[section] = icons;
      console.log(`📂 ${section}: ${icons.length} icons`);
    } catch (e) {
      console.warn(`⚠️  Section ${section} not found, skipping...`);
    }
  }

  // Generate TypeScript type definitions
  const typeDefinitions = `// Auto-generated icon types - DO NOT EDIT MANUALLY
// Generated on: ${new Date().toISOString()}

export type SpriteSection = ${sections.map((s) => `"${s}"`).join(" | ")};

// Core icons (common across the app)
export type CoreIcons = ${
    types.core.length > 0
      ? types.core.map((i) => `"${i}"`).join(" | ")
      : "never"
  };

// Home page specific icons
export type HomeIcons = ${
    types.home.length > 0
      ? types.home.map((i) => `"${i}"`).join(" | ")
      : "never"
  };

// Admin dashboard icons
export type AdminIcons = ${
    types.admin.length > 0
      ? types.admin.map((i) => `"${i}"`).join(" | ")
      : "never"
  };

// CMS specific icons
export type CmsIcons = ${
    types.cms.length > 0 ? types.cms.map((i) => `"${i}"`).join(" | ") : "never"
  };

// Union of all icon names
export type IconName = CoreIcons | HomeIcons | AdminIcons | CmsIcons;

// Icon sections constant for runtime use
export const ICON_SECTIONS: readonly SpriteSection[] = [${sections
    .map((s) => `"${s}"`)
    .join(", ")}] as const;

// Icon inventories for each section
export const ICON_INVENTORIES = {
  core: [${types.core.map((i) => `"${i}"`).join(", ")}] as const,
  home: [${types.home.map((i) => `"${i}"`).join(", ")}] as const,
  admin: [${types.admin.map((i) => `"${i}"`).join(", ")}] as const,
  cms: [${types.cms.map((i) => `"${i}"`).join(", ")}] as const,
} as const;

// Type guards
export function isCoreIcon(icon: string): icon is CoreIcons {
  return ICON_INVENTORIES.core.includes(icon as CoreIcons);
}

export function isHomeIcon(icon: string): icon is HomeIcons {
  return ICON_INVENTORIES.home.includes(icon as HomeIcons);
}

export function isAdminIcon(icon: string): icon is AdminIcons {
  return ICON_INVENTORIES.admin.includes(icon as AdminIcons);
}

export function isCmsIcon(icon: string): icon is CmsIcons {
  return ICON_INVENTORIES.cms.includes(icon as CmsIcons);
}

export function isValidIcon(icon: string): icon is IconName {
  return isCoreIcon(icon) || isHomeIcon(icon) || isAdminIcon(icon) || isCmsIcon(icon);
}

// Get section for an icon
export function getIconSection(icon: IconName): SpriteSection {
  if (isCoreIcon(icon)) return "core";
  if (isHomeIcon(icon)) return "home";
  if (isAdminIcon(icon)) return "admin";
  if (isCmsIcon(icon)) return "cms";
  throw new Error(\`Unknown icon: \${icon}\`);
}
`;

  // Write types file
  writeFileSync("libs/icons/src/lib/types/icon.types.ts", typeDefinitions);

  console.log("\n✅ TypeScript types generated successfully!");
  console.log(`📊 Total icons: ${Object.values(types).flat().length}`);
  console.log("📁 File: libs/icons/src/lib/types/icon.types.ts");
}

generateTypes();
```

### 4. Preview de Iconos

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
  </div>`;

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

## 🚀 Script de Deploy a CDN

```javascript
// scripts/deploy-icons.mjs
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";
import {
  CloudFrontClient,
  CreateInvalidationCommand,
} from "@aws-sdk/client-cloudfront";
import { readFileSync, readdirSync, existsSync } from "fs";
import { gzipSync } from "zlib";

const s3 = new S3Client({
  region: "auto",
  endpoint: `https://${process.env.CLOUDFLARE_ACCOUNT_ID}.r2.cloudflarestorage.com`,
  credentials: {
    accessKeyId: process.env.CLOUDFLARE_ACCESS_KEY_ID,
    secretAccessKey: process.env.CLOUDFLARE_SECRET_ACCESS_KEY,
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

      // Upload original
      await s3.send(
        new PutObjectCommand({
          Bucket: "icons-cdn",
          Key: sprite,
          Body: content,
          ContentType: "image/svg+xml",
          CacheControl: "public, max-age=31536000, immutable",
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
          ContentType: "image/svg+xml",
          CacheControl: "public, max-age=31536000, immutable",
          ContentEncoding: "gzip",
          Metadata: {
            "compressed-size": gzipped.length.toString(),
          },
        })
      );

      console.log(
        `✅ ${sprite} deployed (${(content.length / 1024).toFixed(1)}KB → ${(
          gzipped.length / 1024
        ).toFixed(1)}KB gz)`
      );

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

    console.log(
      `\n✨ Deployment complete! ${deployedFiles.length} sprites deployed to CDN.`
    );
    console.log(`📊 CDN URL: https://cdn.miempresa.com/icons/`);
  } catch (error) {
    console.error("\n❌ Deployment failed:", error.message);
    process.exit(1);
  }
}

deploySprites();
```

## 📋 Comandos NX

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

## 🎯 Checklist de Scripts

- [ ] **Script de generación configurado**
- [ ] **Validación automática funcionando**
- [ ] **Tipos TypeScript generándose**
- [ ] **Preview visual disponible**
- [ ] **Deploy a CDN automatizado**
- [ ] **CI/CD pipeline integrado**
- [ ] **Comandos NX configurados**
- [ ] **Monitoreo de tamaños de archivo**

---

**Siguiente**: [Testing de Performance](./testing.md)
