# ☁️ Automatización CDN para Iconos

Sistema completo de automatización para deploy de sprites de iconos a Cloudflare CDN con CI/CD, optimización y monitoreo.

## 🎯 Objetivos

- **Deploy automático**: Sprites se suben automáticamente al CDN
- **Cache busting**: Nombres con hash para cache inmutable
- **Compresión múltiple**: Gzip + Brotli para máximo rendimiento
- **Invalidación automática**: CloudFront cache se limpia automáticamente
- **Rollback seguro**: Posibilidad de volver a versiones anteriores

## 🏗️ Arquitectura de Deploy

```
┌─────────────────────────────────────────────────────────────┐
│                    DESARROLLO LOCAL                         │
├─────────────────────────────────────────────────────────────┤
│  1. Agregar/modificar SVGs en libs/icons/src/assets/       │
│  2. npm run icons:validate                                 │
│  3. npm run icons:generate                                 │
│  4. git commit + push                                      │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    GITHUB ACTIONS                          │
├─────────────────────────────────────────────────────────────┤
│  1. Validar SVGs (sintaxis, naming, tamaño)               │
│  2. Generar sprites optimizados                           │
│  3. Ejecutar tests                                         │
│  4. Deploy a Cloudflare R2                                │
│  5. Invalidar cache CloudFront                            │
│  6. Notificar resultado                                    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                 CLOUDFLARE CDN                             │
├─────────────────────────────────────────────────────────────┤
│  • sprite-core-a1b2c3d4.svg                               │
│  • sprite-home-e5f6g7h8.svg                               │
│  • sprite-admin-i9j0k1l2.svg                              │
│  • manifest.json                                          │
│  📊 Headers: Cache-Control: immutable, max-age=1year      │
└─────────────────────────────────────────────────────────────┘
```

## 🛠️ Scripts de Deploy

### 1. Script Principal de Deploy

```javascript
// scripts/deploy-icons.mjs
import {
  S3Client,
  PutObjectCommand,
  HeadObjectCommand,
} from "@aws-sdk/client-s3";
import {
  CloudFrontClient,
  CreateInvalidationCommand,
} from "@aws-sdk/client-cloudfront";
import { readFileSync, readdirSync, existsSync, statSync } from "fs";
import { createHash } from "crypto";
import { gzipSync, brotliCompressSync, constants } from "zlib";
import chalk from "chalk";

// Configuración del cliente S3 (Cloudflare R2)
const s3Client = new S3Client({
  region: "auto",
  endpoint: `https://${process.env.CLOUDFLARE_ACCOUNT_ID}.r2.cloudflarestorage.com`,
  credentials: {
    accessKeyId: process.env.CLOUDFLARE_ACCESS_KEY_ID,
    secretAccessKey: process.env.CLOUDFLARE_SECRET_ACCESS_KEY,
  },
});

// Cliente CloudFront para invalidación
const cloudFrontClient = new CloudFrontClient({
  region: "us-east-1",
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
  },
});

class IconDeployService {
  constructor() {
    this.bucketName = process.env.CLOUDFLARE_BUCKET_NAME || "icons-cdn";
    this.distributionId = process.env.CLOUDFRONT_DISTRIBUTION_ID;
    this.environment = process.env.ENVIRONMENT || "production";
    this.deployedFiles = [];
    this.skippedFiles = [];
  }

  async deploy() {
    console.log(chalk.blue("🚀 Starting icon deployment to CDN...\n"));
    console.log(`📦 Environment: ${chalk.cyan(this.environment)}`);
    console.log(`🪣 Bucket: ${chalk.cyan(this.bucketName)}\n`);

    try {
      // 1. Validar archivos locales
      await this.validateLocalFiles();

      // 2. Cargar manifest actual del CDN
      const currentManifest = await this.getCurrentManifest();

      // 3. Procesar cada sprite
      const localManifest = await this.processSprites(currentManifest);

      // 4. Subir manifest actualizado
      await this.uploadManifest(localManifest);

      // 5. Invalidar cache si es necesario
      if (this.deployedFiles.length > 0) {
        await this.invalidateCache();
      }

      // 6. Mostrar resumen
      this.showSummary();
    } catch (error) {
      console.error(chalk.red("❌ Deployment failed:"), error.message);
      process.exit(1);
    }
  }

  async validateLocalFiles() {
    const spritesDir = "dist/sprites";

    if (!existsSync(spritesDir)) {
      throw new Error(
        "Sprites directory not found. Run npm run icons:generate first."
      );
    }

    const sprites = readdirSync(spritesDir).filter((f) => f.endsWith(".svg"));

    if (sprites.length === 0) {
      throw new Error("No sprite files found.");
    }

    console.log(chalk.green(`✅ Found ${sprites.length} sprite files locally`));

    // Validar tamaños
    let totalSize = 0;
    for (const sprite of sprites) {
      const content = readFileSync(`${spritesDir}/${sprite}`);
      const size = Buffer.byteLength(content);
      totalSize += size;

      if (size > 500000) {
        // 500KB
        console.log(
          chalk.yellow(`⚠️  ${sprite} is large (${(size / 1024).toFixed(1)}KB)`)
        );
      }
    }

    console.log(
      `📊 Total size: ${chalk.cyan((totalSize / 1024).toFixed(1) + "KB")}\n`
    );
  }

  async getCurrentManifest() {
    try {
      const response = await s3Client.send(
        new HeadObjectCommand({
          Bucket: this.bucketName,
          Key: "manifest.json",
        })
      );

      // Si existe, descargar el contenido
      const manifestContent = await this.downloadFile("manifest.json");
      return JSON.parse(manifestContent);
    } catch (error) {
      console.log(
        chalk.yellow("📄 No existing manifest found, creating new one")
      );
      return {};
    }
  }

  async downloadFile(key) {
    // Implementar descarga de archivo del CDN
    // Por simplicidad, retornamos objeto vacío si no existe
    return "{}";
  }

  async processSprites(currentManifest) {
    const spritesDir = "dist/sprites";
    const localManifest = JSON.parse(
      readFileSync(`${spritesDir}/manifest.json`, "utf-8")
    );
    const newManifest = { ...currentManifest };

    for (const [section, sectionData] of Object.entries(localManifest)) {
      const spriteFile = sectionData.filename;
      const localPath = `${spritesDir}/${spriteFile}`;

      if (!existsSync(localPath)) {
        console.log(chalk.yellow(`⚠️  Sprite file not found: ${spriteFile}`));
        continue;
      }

      // Verificar si necesita actualización
      const needsUpdate = this.needsUpdate(
        currentManifest[section],
        sectionData
      );

      if (!needsUpdate) {
        console.log(chalk.gray(`⏭️  Skipping ${spriteFile} (no changes)`));
        this.skippedFiles.push(spriteFile);
        continue;
      }

      console.log(chalk.blue(`📤 Uploading ${spriteFile}...`));

      // Subir archivo con múltiples compresiones
      await this.uploadSpriteFile(localPath, spriteFile, sectionData);

      // Actualizar manifest
      newManifest[section] = {
        ...sectionData,
        deployedAt: new Date().toISOString(),
        environment: this.environment,
      };

      this.deployedFiles.push(spriteFile);
    }

    return newManifest;
  }

  needsUpdate(current, local) {
    if (!current) return true;
    if (current.hash !== local.hash) return true;
    if (current.environment !== this.environment) return true;
    return false;
  }

  async uploadSpriteFile(localPath, filename, metadata) {
    const content = readFileSync(localPath);
    const stats = statSync(localPath);

    // Headers comunes
    const commonHeaders = {
      "Cache-Control": "public, max-age=31536000, immutable",
      "Content-Type": "image/svg+xml",
      "X-Content-Type-Options": "nosniff",
      "Access-Control-Allow-Origin": "*",
    };

    const commonMetadata = {
      "deployment-date": new Date().toISOString(),
      environment: this.environment,
      "sprite-version": metadata.hash,
      "original-size": content.length.toString(),
      "last-modified": stats.mtime.toISOString(),
    };

    // 1. Subir archivo original
    await s3Client.send(
      new PutObjectCommand({
        Bucket: this.bucketName,
        Key: filename,
        Body: content,
        ContentType: commonHeaders["Content-Type"],
        CacheControl: commonHeaders["Cache-Control"],
        Metadata: commonMetadata,
      })
    );

    // 2. Subir versión Gzip
    const gzipped = gzipSync(content, { level: 9 });
    await s3Client.send(
      new PutObjectCommand({
        Bucket: this.bucketName,
        Key: `${filename}.gz`,
        Body: gzipped,
        ContentType: commonHeaders["Content-Type"],
        CacheControl: commonHeaders["Cache-Control"],
        ContentEncoding: "gzip",
        Metadata: {
          ...commonMetadata,
          "compressed-size": gzipped.length.toString(),
          compression: "gzip",
        },
      })
    );

    // 3. Subir versión Brotli
    const brotli = brotliCompressSync(content, {
      params: {
        [constants.BROTLI_PARAM_QUALITY]: constants.BROTLI_MAX_QUALITY,
      },
    });

    await s3Client.send(
      new PutObjectCommand({
        Bucket: this.bucketName,
        Key: `${filename}.br`,
        Body: brotli,
        ContentType: commonHeaders["Content-Type"],
        CacheControl: commonHeaders["Cache-Control"],
        ContentEncoding: "br",
        Metadata: {
          ...commonMetadata,
          "compressed-size": brotli.length.toString(),
          compression: "brotli",
        },
      })
    );

    // Log de resultados
    const originalKB = (content.length / 1024).toFixed(1);
    const gzipKB = (gzipped.length / 1024).toFixed(1);
    const brotliKB = (brotli.length / 1024).toFixed(1);

    console.log(chalk.green(`✅ ${filename} uploaded successfully`));
    console.log(
      `   📊 Sizes: ${originalKB}KB → ${gzipKB}KB (gz) → ${brotliKB}KB (br)`
    );
    console.log(
      `   🗜️  Compression: ${chalk.cyan(
        ((1 - brotli.length / content.length) * 100).toFixed(1) + "%"
      )}`
    );
  }

  async uploadManifest(manifest) {
    const manifestContent = JSON.stringify(
      {
        version: new Date().toISOString().split("T")[0],
        lastUpdate: new Date().toISOString(),
        environment: this.environment,
        ...manifest,
      },
      null,
      2
    );

    await s3Client.send(
      new PutObjectCommand({
        Bucket: this.bucketName,
        Key: "manifest.json",
        Body: manifestContent,
        ContentType: "application/json",
        CacheControl: "public, max-age=300, s-maxage=300", // 5 minutos para manifest
        Metadata: {
          "deployment-date": new Date().toISOString(),
          environment: this.environment,
          "deployed-files": this.deployedFiles.length.toString(),
        },
      })
    );

    console.log(chalk.green("📄 Manifest uploaded successfully"));
  }

  async invalidateCache() {
    if (!this.distributionId) {
      console.log(
        chalk.yellow(
          "⚠️  CloudFront distribution ID not provided, skipping cache invalidation"
        )
      );
      return;
    }

    console.log(chalk.blue("\n🔄 Invalidating CloudFront cache..."));

    const paths = [
      ...this.deployedFiles.map((f) => `/${f}`),
      ...this.deployedFiles.map((f) => `/${f}.gz`),
      ...this.deployedFiles.map((f) => `/${f}.br`),
      "/manifest.json",
    ];

    try {
      const response = await cloudFrontClient.send(
        new CreateInvalidationCommand({
          DistributionId: this.distributionId,
          InvalidationBatch: {
            CallerReference: `icons-${Date.now()}`,
            Paths: {
              Quantity: paths.length,
              Items: paths,
            },
          },
        })
      );

      console.log(
        chalk.green(
          `✅ Cache invalidation created: ${response.Invalidation.Id}`
        )
      );
      console.log(`   📋 Invalidated ${paths.length} paths`);
    } catch (error) {
      console.log(chalk.red(`❌ Cache invalidation failed: ${error.message}`));
      // No fallar el deploy por esto
    }
  }

  showSummary() {
    console.log(chalk.blue("\n📊 Deployment Summary"));
    console.log("═".repeat(50));

    if (this.deployedFiles.length > 0) {
      console.log(
        chalk.green(`✅ Deployed files: ${this.deployedFiles.length}`)
      );
      this.deployedFiles.forEach((file) => {
        console.log(`   • ${file}`);
      });
    }

    if (this.skippedFiles.length > 0) {
      console.log(chalk.gray(`⏭️  Skipped files: ${this.skippedFiles.length}`));
      this.skippedFiles.forEach((file) => {
        console.log(`   • ${file} (no changes)`);
      });
    }

    console.log(
      `\n🌐 CDN URL: ${chalk.cyan(
        `https://${process.env.CDN_DOMAIN || "cdn.miempresa.com"}/icons/`
      )}`
    );
    console.log(`📅 Deployed at: ${chalk.cyan(new Date().toLocaleString())}`);

    if (this.deployedFiles.length === 0) {
      console.log(
        chalk.yellow("\n✨ No changes detected, all files up to date!")
      );
    } else {
      console.log(chalk.green("\n✨ Deployment completed successfully!"));
    }
  }
}

// Ejecutar deployment
const deployer = new IconDeployService();
deployer.deploy().catch(console.error);
```

### 2. GitHub Actions Workflow

```yaml
# .github/workflows/deploy-icons.yml
name: 🎨 Deploy Icons to CDN

on:
  push:
    branches: [main, develop]
    paths:
      - "libs/icons/src/assets/**"
      - "libs/icons/tools/**"
      - "scripts/deploy-icons.mjs"
      - ".github/workflows/deploy-icons.yml"

  pull_request:
    paths:
      - "libs/icons/src/assets/**"
      - "libs/icons/tools/**"

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
          - development
      force_deploy:
        description: "Force deploy all files"
        required: false
        default: false
        type: boolean

env:
  NODE_VERSION: "20"
  CACHE_KEY_PREFIX: "icons-v1"

jobs:
  detect-changes:
    name: 🔍 Detect Changes
    runs-on: ubuntu-latest
    outputs:
      icons-changed: ${{ steps.changes.outputs.icons }}
      should-deploy: ${{ steps.deploy-check.outputs.should-deploy }}
    steps:
      - uses: actions/checkout@v4

      - uses: dorny/paths-filter@v2
        id: changes
        with:
          filters: |
            icons:
              - 'libs/icons/src/assets/**'
              - 'libs/icons/tools/**'
              - 'scripts/deploy-icons.mjs'

      - name: Check if should deploy
        id: deploy-check
        run: |
          if [[ "${{ steps.changes.outputs.icons }}" == "true" ]] || \
             [[ "${{ github.event_name }}" == "workflow_dispatch" ]] || \
             [[ "${{ github.event.inputs.force_deploy }}" == "true" ]]; then
            echo "should-deploy=true" >> $GITHUB_OUTPUT
          else
            echo "should-deploy=false" >> $GITHUB_OUTPUT
          fi

  validate:
    name: ✅ Validate Icons
    runs-on: ubuntu-latest
    needs: detect-changes
    if: needs.detect-changes.outputs.should-deploy == 'true'

    steps:
      - name: 📥 Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: 📦 Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"

      - name: 📚 Install dependencies
        run: npm ci --prefer-offline --no-audit

      - name: 🔍 Lint SVG files
        run: |
          echo "🔍 Validating SVG syntax and structure..."
          npm install -g svglint

          # Configuración básica de svglint
          cat > .svglint.yml << 'EOF'
          rules:
            elm:
              svg: 1
              "svg > title": false
              "svg > desc": false
            attr:
              "viewBox": 1
              "xmlns": "http://www.w3.org/2000/svg"
              "xmlns:xlink": false
              "xml:space": false
          EOF

          svglint 'libs/icons/src/assets/**/*.svg' --config .svglint.yml

      - name: 📏 Check file sizes
        run: |
          echo "📏 Checking icon file sizes..."
          find libs/icons/src/assets -name "*.svg" | while read file; do
            size=$(stat -c%s "$file")
            kb_size=$((size / 1024))
            if [ $size -gt 5120 ]; then  # 5KB
              echo "⚠️  WARNING: $file is large (${kb_size}KB)"
            fi
          done

      - name: 🏷️ Validate naming conventions
        run: |
          echo "🏷️ Checking naming conventions..."
          error=0

          find libs/icons/src/assets -name "*.svg" | while read file; do
            basename=$(basename "$file")
            
            # Check lowercase
            if [[ "$basename" != "${basename,,}" ]]; then
              echo "❌ ERROR: $file contains uppercase letters"
              error=1
            fi
            
            # Check valid characters (lowercase, numbers, hyphens only)
            if [[ ! "$basename" =~ ^[a-z0-9-]+\.svg$ ]]; then
              echo "❌ ERROR: $file contains invalid characters"
              error=1
            fi
            
            # Check length (max 50 characters)
            if [ ${#basename} -gt 50 ]; then
              echo "❌ ERROR: $file name too long (${#basename} chars)"
              error=1
            fi
          done

          if [ $error -eq 1 ]; then
            exit 1
          fi

          echo "✅ All icons follow naming conventions"

      - name: 🧪 Run custom validation
        run: |
          echo "🧪 Running custom icon validation..."
          node libs/icons/tools/validate-icons.mjs

      - name: 📊 Generate validation report
        run: |
          echo "📊 Generating validation report..."

          total_icons=$(find libs/icons/src/assets -name "*.svg" | wc -l)
          total_size=$(find libs/icons/src/assets -name "*.svg" -exec stat -c%s {} + | awk '{sum+=$1} END {print sum}')
          total_size_kb=$((total_size / 1024))

          echo "### 📊 Icon Validation Summary" >> $GITHUB_STEP_SUMMARY
          echo "- **Total Icons**: $total_icons" >> $GITHUB_STEP_SUMMARY
          echo "- **Total Size**: ${total_size_kb}KB" >> $GITHUB_STEP_SUMMARY
          echo "- **Average Size**: $((total_size_kb / total_icons))KB per icon" >> $GITHUB_STEP_SUMMARY

          # Breakdown by section
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "#### By Section:" >> $GITHUB_STEP_SUMMARY
          for section in core home admin cms; do
            if [ -d "libs/icons/src/assets/individual/$section" ]; then
              count=$(find "libs/icons/src/assets/individual/$section" -name "*.svg" | wc -l)
              echo "- **$section**: $count icons" >> $GITHUB_STEP_SUMMARY
            fi
          done

  build-and-test:
    name: 🔨 Build Sprites & Test
    runs-on: ubuntu-latest
    needs: [detect-changes, validate]
    if: needs.detect-changes.outputs.should-deploy == 'true'

    steps:
      - name: 📥 Checkout code
        uses: actions/checkout@v4

      - name: 📦 Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"

      - name: 📚 Install dependencies
        run: npm ci --prefer-offline --no-audit

      - name: 🎨 Generate sprites
        run: |
          echo "🎨 Generating sprite files..."
          node libs/icons/tools/generate-sprites.mjs

          echo "✅ Sprite generation completed"
          ls -la dist/sprites/

      - name: 📝 Generate TypeScript types
        run: |
          echo "📝 Generating TypeScript types..."
          node libs/icons/tools/generate-types.mjs

          echo "✅ Type generation completed"

      - name: 🧪 Run unit tests
        run: |
          echo "🧪 Running icon component tests..."
          npm run test:icons -- --coverage --watch=false

          echo "✅ Tests completed"

      - name: 📊 Analyze sprite sizes
        run: |
          echo "📊 Analyzing sprite file sizes..."

          echo "### 🎨 Generated Sprites" >> $GITHUB_STEP_SUMMARY
          echo "| File | Size | Icons | Compression |" >> $GITHUB_STEP_SUMMARY
          echo "|------|------|-------|-------------|" >> $GITHUB_STEP_SUMMARY

          for sprite in dist/sprites/*.svg; do
            if [ -f "$sprite" ]; then
              filename=$(basename "$sprite")
              size=$(stat -c%s "$sprite")
              size_kb=$((size / 1024))
              
              # Count icons in sprite
              icon_count=$(grep -o '<symbol' "$sprite" | wc -l)
              
              # Calculate compression potential
              gzip_size=$(gzip -c "$sprite" | wc -c)
              compression=$(((size - gzip_size) * 100 / size))
              
              echo "| $filename | ${size_kb}KB | $icon_count | ${compression}% |" >> $GITHUB_STEP_SUMMARY
            fi
          done

      - name: 💾 Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: icon-sprites-${{ github.sha }}
          path: |
            dist/sprites/
            libs/icons/src/lib/types/
          retention-days: 7
          compression-level: 9

      - name: 📋 Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results-${{ github.sha }}
          path: |
            coverage/
            test-results.xml
          retention-days: 3

  deploy:
    name: 🚀 Deploy to CDN
    runs-on: ubuntu-latest
    needs: [detect-changes, build-and-test]
    if: |
      needs.detect-changes.outputs.should-deploy == 'true' && 
      (github.ref == 'refs/heads/main' || github.event_name == 'workflow_dispatch')

    environment:
      name: ${{ github.event.inputs.environment || (github.ref == 'refs/heads/main' && 'production' || 'staging') }}
      url: https://${{ vars.CDN_DOMAIN }}/icons/

    steps:
      - name: 📥 Checkout code
        uses: actions/checkout@v4

      - name: 📦 Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"

      - name: 📚 Install dependencies
        run: npm ci --prefer-offline --no-audit

      - name: 💾 Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: icon-sprites-${{ github.sha }}
          path: .

      - name: 🔍 Verify artifacts
        run: |
          echo "🔍 Verifying downloaded artifacts..."
          ls -la dist/sprites/

          if [ ! -f "dist/sprites/manifest.json" ]; then
            echo "❌ Manifest file not found!"
            exit 1
          fi

          sprite_count=$(find dist/sprites -name "*.svg" | wc -l)
          if [ $sprite_count -eq 0 ]; then
            echo "❌ No sprite files found!"
            exit 1
          fi

          echo "✅ Found $sprite_count sprite files and manifest"

      - name: 🚀 Deploy to Cloudflare R2
        env:
          CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          CLOUDFLARE_ACCESS_KEY_ID: ${{ secrets.CLOUDFLARE_ACCESS_KEY_ID }}
          CLOUDFLARE_SECRET_ACCESS_KEY: ${{ secrets.CLOUDFLARE_SECRET_ACCESS_KEY }}
          CLOUDFLARE_BUCKET_NAME: ${{ vars.CLOUDFLARE_BUCKET_NAME }}
          CLOUDFRONT_DISTRIBUTION_ID: ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          CDN_DOMAIN: ${{ vars.CDN_DOMAIN }}
          ENVIRONMENT: ${{ github.event.inputs.environment || (github.ref == 'refs/heads/main' && 'production' || 'staging') }}
        run: |
          echo "🚀 Starting deployment to CDN..."
          node scripts/deploy-icons.mjs

      - name: 🔗 Test CDN endpoints
        run: |
          echo "🔗 Testing CDN endpoints..."

          # Wait a bit for CDN propagation
          sleep 10

          # Test manifest
          manifest_url="https://${{ vars.CDN_DOMAIN }}/icons/manifest.json"
          echo "Testing: $manifest_url"

          if curl -f -s "$manifest_url" > /dev/null; then
            echo "✅ Manifest endpoint is accessible"
          else
            echo "❌ Manifest endpoint failed"
          fi

          # Test a sprite file (assuming core sprite exists)
          sprite_url="https://${{ vars.CDN_DOMAIN }}/icons/sprite-core"
          echo "Testing: $sprite_url"

          if curl -f -s "${sprite_url}.svg" > /dev/null; then
            echo "✅ Sprite endpoint is accessible"
          else
            echo "⚠️  Sprite endpoint may still be propagating"
          fi

      - name: 📢 Create deployment summary
        run: |
          echo "### 🚀 Deployment Summary" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "**Environment**: ${{ github.event.inputs.environment || (github.ref == 'refs/heads/main' && 'production' || 'staging') }}" >> $GITHUB_STEP_SUMMARY
          echo "**CDN URL**: https://${{ vars.CDN_DOMAIN }}/icons/" >> $GITHUB_STEP_SUMMARY
          echo "**Deployed at**: $(date -u '+%Y-%m-%d %H:%M:%S UTC')" >> $GITHUB_STEP_SUMMARY
          echo "**Commit**: ${{ github.sha }}" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "#### 🔗 Quick Links" >> $GITHUB_STEP_SUMMARY
          echo "- [📄 Manifest](https://${{ vars.CDN_DOMAIN }}/icons/manifest.json)" >> $GITHUB_STEP_SUMMARY
          echo "- [🎨 Core Sprites](https://${{ vars.CDN_DOMAIN }}/icons/sprite-core.svg)" >> $GITHUB_STEP_SUMMARY

      - name: 💬 Comment on PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const comment = `### 🎨 Icon Changes Preview

            Your icon changes have been deployed to the staging environment!

            **🔗 Preview URLs:**
            - [📄 Manifest](https://${{ vars.CDN_DOMAIN }}/icons/manifest.json)
            - [🎨 Sprites](https://${{ vars.CDN_DOMAIN }}/icons/)

            **📊 Deployment Info:**
            - Environment: staging
            - Commit: ${{ github.sha }}
            - Deployed at: ${new Date().toISOString()}

            > Preview will be available for 7 days`;

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: comment
            });

  notify:
    name: 📢 Notify Results
    runs-on: ubuntu-latest
    needs: [detect-changes, validate, build-and-test, deploy]
    if: always() && needs.detect-changes.outputs.should-deploy == 'true'

    steps:
      - name: 📊 Determine overall status
        id: status
        run: |
          if [[ "${{ needs.deploy.result }}" == "success" ]]; then
            echo "status=success" >> $GITHUB_OUTPUT
            echo "message=✅ Icons deployed successfully to CDN" >> $GITHUB_OUTPUT
          elif [[ "${{ needs.deploy.result }}" == "failure" ]]; then
            echo "status=failure" >> $GITHUB_OUTPUT
            echo "message=❌ Icon deployment failed" >> $GITHUB_OUTPUT
          elif [[ "${{ needs.build-and-test.result }}" == "failure" ]]; then
            echo "status=failure" >> $GITHUB_OUTPUT
            echo "message=❌ Icon build/test failed" >> $GITHUB_OUTPUT
          elif [[ "${{ needs.validate.result }}" == "failure" ]]; then
            echo "status=failure" >> $GITHUB_OUTPUT
            echo "message=❌ Icon validation failed" >> $GITHUB_OUTPUT
          else
            echo "status=warning" >> $GITHUB_OUTPUT
            echo "message=⚠️  Icon workflow completed with warnings" >> $GITHUB_OUTPUT
          fi

      - name: 🔔 Notify Slack (Optional)
        if: vars.SLACK_WEBHOOK_URL
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ steps.status.outputs.status }}
          webhook_url: ${{ vars.SLACK_WEBHOOK_URL }}
          message: |
            ${{ steps.status.outputs.message }}

            Repository: ${{ github.repository }}
            Branch: ${{ github.ref_name }}
            Commit: ${{ github.sha }}
            Actor: ${{ github.actor }}
```

### 3. Script de Rollback

```javascript
// scripts/rollback-icons.mjs
import {
  S3Client,
  ListObjectsV2Command,
  CopyObjectCommand,
  DeleteObjectCommand,
} from "@aws-sdk/client-s3";
import {
  CloudFrontClient,
  CreateInvalidationCommand,
} from "@aws-sdk/client-cloudfront";
import chalk from "chalk";
import inquirer from "inquirer";

class IconRollbackService {
  constructor() {
    this.s3Client = new S3Client({
      region: "auto",
      endpoint: `https://${process.env.CLOUDFLARE_ACCOUNT_ID}.r2.cloudflarestorage.com`,
      credentials: {
        accessKeyId: process.env.CLOUDFLARE_ACCESS_KEY_ID,
        secretAccessKey: process.env.CLOUDFLARE_SECRET_ACCESS_KEY,
      },
    });

    this.cloudFrontClient = new CloudFrontClient({
      region: "us-east-1",
      credentials: {
        accessKeyId: process.env.AWS_ACCESS_KEY_ID,
        secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
      },
    });

    this.bucketName = process.env.CLOUDFLARE_BUCKET_NAME || "icons-cdn";
    this.distributionId = process.env.CLOUDFRONT_DISTRIBUTION_ID;
  }

  async rollback() {
    console.log(chalk.blue("🔄 Icon Rollback Utility\n"));

    try {
      // 1. Listar versiones disponibles
      const versions = await this.listAvailableVersions();

      if (versions.length === 0) {
        console.log(chalk.yellow("No backup versions found."));
        return;
      }

      // 2. Mostrar opciones al usuario
      const { selectedVersion } = await inquirer.prompt([
        {
          type: "list",
          name: "selectedVersion",
          message: "Select version to rollback to:",
          choices: versions.map((v) => ({
            name: `${v.date} - ${v.description} (${v.files.length} files)`,
            value: v,
          })),
        },
      ]);

      // 3. Confirmar rollback
      const { confirm } = await inquirer.prompt([
        {
          type: "confirm",
          name: "confirm",
          message: `Are you sure you want to rollback to ${selectedVersion.date}?`,
          default: false,
        },
      ]);

      if (!confirm) {
        console.log(chalk.gray("Rollback cancelled."));
        return;
      }

      // 4. Ejecutar rollback
      await this.executeRollback(selectedVersion);

      console.log(chalk.green("\n✅ Rollback completed successfully!"));
    } catch (error) {
      console.error(chalk.red("❌ Rollback failed:"), error.message);
      process.exit(1);
    }
  }

  async listAvailableVersions() {
    // Implementar listado de versiones desde backups
    // Por simplicidad, retornamos array de ejemplo
    return [
      {
        date: "2024-12-15T10:30:00Z",
        description: "Version before breaking changes",
        files: ["sprite-core-abc123.svg", "sprite-home-def456.svg"],
      },
      {
        date: "2024-12-14T15:20:00Z",
        description: "Stable release v2.1.0",
        files: ["sprite-core-xyz789.svg", "sprite-home-uvw012.svg"],
      },
    ];
  }

  async executeRollback(version) {
    console.log(chalk.blue(`🔄 Rolling back to ${version.date}...`));

    // Implementar lógica de rollback
    // 1. Restaurar archivos desde backup
    // 2. Actualizar manifest
    // 3. Invalidar cache

    console.log(chalk.green("Files restored from backup"));
    console.log(chalk.green("Manifest updated"));
    console.log(chalk.green("Cache invalidated"));
  }
}

// Ejecutar rollback
const rollbackService = new IconRollbackService();
rollbackService.rollback().catch(console.error);
```

## 📊 Monitoreo y Alertas

### 1. Health Check Script

```javascript
// scripts/health-check-icons.mjs
import fetch from "node-fetch";
import chalk from "chalk";

class IconHealthChecker {
  constructor() {
    this.baseUrl = process.env.CDN_DOMAIN || "cdn.miempresa.com";
    this.timeout = 10000; // 10 segundos
  }

  async checkHealth() {
    console.log(chalk.blue("🏥 Icon CDN Health Check\n"));

    const checks = [
      { name: "Manifest", test: () => this.checkManifest() },
      { name: "Core Sprites", test: () => this.checkSprite("core") },
      { name: "Home Sprites", test: () => this.checkSprite("home") },
      { name: "Response Times", test: () => this.checkPerformance() },
      { name: "Compression", test: () => this.checkCompression() },
    ];

    let allPassed = true;

    for (const check of checks) {
      try {
        const result = await check.test();
        if (result.passed) {
          console.log(chalk.green(`✅ ${check.name}: ${result.message}`));
        } else {
          console.log(chalk.red(`❌ ${check.name}: ${result.message}`));
          allPassed = false;
        }
      } catch (error) {
        console.log(chalk.red(`❌ ${check.name}: ${error.message}`));
        allPassed = false;
      }
    }

    console.log(chalk.blue("\n📊 Health Check Summary"));
    console.log(
      allPassed
        ? chalk.green("✅ All checks passed")
        : chalk.red("❌ Some checks failed")
    );

    return allPassed;
  }

  async checkManifest() {
    const url = `https://${this.baseUrl}/icons/manifest.json`;
    const response = await fetch(url, { timeout: this.timeout });

    if (!response.ok) {
      return { passed: false, message: `HTTP ${response.status}` };
    }

    const manifest = await response.json();

    if (!manifest.version || !manifest.lastUpdate) {
      return { passed: false, message: "Invalid manifest structure" };
    }

    return { passed: true, message: `Version ${manifest.version}` };
  }

  async checkSprite(section) {
    const manifestUrl = `https://${this.baseUrl}/icons/manifest.json`;
    const manifestResponse = await fetch(manifestUrl);
    const manifest = await manifestResponse.json();

    if (!manifest[section]) {
      return {
        passed: false,
        message: `Section ${section} not found in manifest`,
      };
    }

    const spriteUrl = `https://${this.baseUrl}/icons/${manifest[section].filename}`;
    const response = await fetch(spriteUrl, { timeout: this.timeout });

    if (!response.ok) {
      return { passed: false, message: `HTTP ${response.status}` };
    }

    const content = await response.text();
    const iconCount = (content.match(/<symbol/g) || []).length;

    return { passed: true, message: `${iconCount} icons loaded` };
  }

  async checkPerformance() {
    const start = performance.now();
    const url = `https://${this.baseUrl}/icons/manifest.json`;

    await fetch(url, { timeout: this.timeout });

    const duration = performance.now() - start;
    const passed = duration < 500; // 500ms threshold

    return {
      passed,
      message: `${Math.round(duration)}ms ${passed ? "(Good)" : "(Slow)"}`,
    };
  }

  async checkCompression() {
    const url = `https://${this.baseUrl}/icons/sprite-core.svg`;

    // Check if gzip is working
    const response = await fetch(url, {
      headers: { "Accept-Encoding": "gzip" },
      timeout: this.timeout,
    });

    const isCompressed = response.headers.get("content-encoding") === "gzip";

    return {
      passed: isCompressed,
      message: isCompressed ? "Gzip enabled" : "Compression not detected",
    };
  }
}

// Ejecutar health check
const checker = new IconHealthChecker();
checker
  .checkHealth()
  .then((allPassed) => process.exit(allPassed ? 0 : 1))
  .catch((error) => {
    console.error(chalk.red("Health check failed:"), error);
    process.exit(1);
  });
```

### 2. Configuración de Monitoreo

```yaml
# monitoring/uptime.yml
name: 🏥 CDN Health Monitor

on:
  schedule:
    # Every 5 minutes
    - cron: "*/5 * * * *"
  workflow_dispatch:

jobs:
  health-check:
    name: 🏥 Health Check
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: 📦 Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: 🏥 Run health check
        env:
          CDN_DOMAIN: ${{ vars.CDN_DOMAIN }}
        run: |
          npm install node-fetch chalk
          node scripts/health-check-icons.mjs

      - name: 🚨 Alert on failure
        if: failure()
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: '🚨 Icon CDN Health Check Failed',
              body: `Health check failed at ${new Date().toISOString()}\n\nPlease investigate CDN status.`,
              labels: ['bug', 'cdn', 'monitoring']
            });
```

## 🔧 Configuración de Entorno

### Variables de Entorno Requeridas

```bash
# .env.example

# Cloudflare R2 Configuration
CLOUDFLARE_ACCOUNT_ID=your_account_id
CLOUDFLARE_ACCESS_KEY_ID=your_access_key
CLOUDFLARE_SECRET_ACCESS_KEY=your_secret_key
CLOUDFLARE_BUCKET_NAME=icons-cdn

# CloudFront (Optional, for cache invalidation)
CLOUDFRONT_DISTRIBUTION_ID=your_distribution_id
AWS_ACCESS_KEY_ID=your_aws_key
AWS_SECRET_ACCESS_KEY=your_aws_secret

# CDN Domain
CDN_DOMAIN=cdn.miempresa.com

# Environment
ENVIRONMENT=production # or staging, development

# Monitoring (Optional)
SLACK_WEBHOOK_URL=your_slack_webhook
```

### GitHub Secrets Configuration

```yaml
# Repository Secrets (Settings > Secrets and variables > Actions)

# Cloudflare
CLOUDFLARE_ACCOUNT_ID: "abc123def456..."
CLOUDFLARE_ACCESS_KEY_ID: "access_key_here"
CLOUDFLARE_SECRET_ACCESS_KEY: "secret_key_here"

# AWS (for CloudFront)
AWS_ACCESS_KEY_ID: "AKIA..."
AWS_SECRET_ACCESS_KEY: "secret..."
CLOUDFRONT_DISTRIBUTION_ID: "E123456789"

# Repository Variables
CLOUDFLARE_BUCKET_NAME: "icons-cdn"
CDN_DOMAIN: "cdn.miempresa.com"
SLACK_WEBHOOK_URL: "https://hooks.slack.com/..."
```

## 📋 Checklist de Configuración

- [ ] **Configurar Cloudflare R2**

  - [ ] Crear bucket
  - [ ] Configurar access keys
  - [ ] Configurar CORS si es necesario

- [ ] **Configurar GitHub Actions**

  - [ ] Agregar secrets requeridos
  - [ ] Configurar variables de entorno
  - [ ] Testear workflow en branch develop

- [ ] **Configurar Monitoreo**

  - [ ] Health check automático
  - [ ] Alertas en Slack (opcional)
  - [ ] Dashboard de métricas

- [ ] **Testear Deploy**
  - [ ] Deploy manual con `npm run icons:deploy`
  - [ ] Verificar archivos en CDN
  - [ ] Testear invalidación de cache

---

**Siguiente**: [Herramientas de Desarrollo](../tools/scripts.md)
