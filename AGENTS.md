# AGENTS.md — Omniluxe Shopify Theme

Panduan teknis dan standar kerja untuk AI Agent & developer yang beroperasi pada repositori **omniluxe_theme**.

---

## 1. Project Overview

- **Project Name:** Omniluxe Theme (`omniluxe_theme`)
- **Platform:** Shopify Online Store 2.0 (OS 2.0)
- **Base Architecture:** Maestrooo Focal Theme (v13.0.0)
- **Business Domain:** Luxury Jewelry, Lab-Grown Diamonds, and Bespoke Accessories
- **Primary Languages:** Liquid, JavaScript (ES6+ / Web Components), CSS3, JSON (Schemas & Templates)

---

## 2. Directory Structure

```text
omniluxe_theme/
├── assets/         # Static assets (CSS, JS, SVGs, vendor libraries)
│   ├── custom.js   # Dedicated custom JS hooks & business logic
│   ├── theme.css   # Main stylesheet compiled for the theme
│   └── theme.js    # Core theme JavaScript bundle
├── config/         # Theme customizer configurations
│   ├── settings_schema.json  # Customizer settings fields schema
│   └── settings_data.json    # Active merchant settings values (DO NOT CORRUPT)
├── layout/         # Base layout files
│   └── theme.liquid # Global HTML shell, analytics, fonts, and meta tags
├── locales/        # Internationalization dictionaries
│   ├── en.default.json # Primary English strings
│   ├── id.json         # Indonesian translation strings
│   └── *.json          # Other supported languages
├── sections/       # Modular Shopify sections with {% schema %}
├── snippets/       # Reusable Liquid components & partials
└── templates/      # OS 2.0 JSON templates (index, product, collection, pages)
    └── customers/  # Customer account Liquid templates
```

---

## 3. Core Development Principles for AI Agents

### 3.1 Liquid Templating Rules
- **Modern Partials:** Selalu gunakan `{% render 'snippet-name', param: value %}`. DILARANG menggunakan tag lawas `{% include %}`.
- **No Hardcoded Strings:** Jangan pernah memasukkan teks atau copy langsung di dalam template Liquid. Gunakan locale filter: `{{ 'section.key.title' | t }}` dan daftarkan key tersebut di `locales/en.default.json` (serta `locales/id.json`).
- **Currency & Money Filters:** Format seluruh harga dengan filter Shopify: `{{ product.price | money }}` atau `{{ product.price | money_without_trailing_zeros }}`.
- **Safe Object Checks:** Selalu lakukan defensive check sebelum mengakses properti objek (misal `{% if product.selected_or_first_available_variant != blank %}`).
- **Performance:** Hindari nested loops mendalam pada Liquid collection atau product list. Gunakan pagination atau `limit` parameter.

### 3.2 JSON Templates & Schemas Integrity
- **JSON Validity:** File pada `templates/*.json`, `config/*.json`, dan block `{% schema %}` di dalam `sections/*.liquid` HARUS selalu merupakan format JSON yang valid.
- **Section Schema Standards:**
  - Gunakan tipe data schema standar Shopify: `text`, `richtext`, `image_picker`, `range`, `select`, `checkbox`, `color`, `url`, `collection`, `product`.
  - Berikan label yang jelas dan deskriptif serta default value yang aman.
  - Sediakan `presets` pada section baru agar dapat ditambahkan oleh merchant via Shopify Theme Customizer.
- **Protect `settings_data.json`:** Jangan menimpa atau mereset seluruh file `config/settings_data.json` karena menyimpan konfigurasi live merchant. Lakukan pengeditan terarah jika diperlukan.

### 3.3 Focal Theme JavaScript Architecture & Events
Tema ini berbasis arsitektur **Focal by Maestrooo** yang menggunakan Event-Driven Architecture dan HTML Custom Elements (Web Components).

#### Event Hooks Penting (`assets/custom.js`):
1. **Variant Changed:**
   ```javascript
   document.addEventListener('variant:changed', function(event) {
     const variant = event.detail.variant; // Detail objek variant
     const form = event.target;           // Form pemicu event
   });
   ```
2. **Variant Added to Cart:**
   ```javascript
   document.addEventListener('variant:added', function(event) {
     const variant = event.detail.variant; // Variant yang baru ditambahkan
   });
   ```
3. **Cart Updated:**
   ```javascript
   document.addEventListener('cart:updated', function(event) {
     const cart = event.detail.cart; // Objek data cart terbaru
   });
   ```
4. **Trigger Cart Re-render (Refresh):**
   ```javascript
   document.documentElement.dispatchEvent(new CustomEvent('cart:refresh', {
     bubbles: true
   }));
   ```

#### Web Components & Custom Elements:
- Tema memanfaatkan custom elements seperti drawer dan popover (misalnya `#mini-cart`).
- Buka atau tutup modal/drawer menggunakan atribut boolean `.open`:
  ```javascript
  // Membuka cart drawer
  const miniCart = document.getElementById('mini-cart');
  if (miniCart) miniCart.open = true;

  // Menutup cart drawer
  if (miniCart) miniCart.open = false;
  ```

### 3.4 Styling & CSS Standards
- Gunakan variabel tema CSS yang sudah didefinisikan (lihat `theme.liquid` / `theme.css`).
- Hindari mengubah file vendor secara langsung (`vendor.min.js`, `photoswipe.min.js`, dsb).
- Untuk kustomisasi styling baru:
  - Gunakan penamaan class BEM (`.omniluxe-card`, `.omniluxe-card__title`, `.omniluxe-card--active`).
  - Masukkan CSS terisolasi atau integrasikan ke stylesheet custom yang terkelola rapi.
- Desain harus responsif (Mobile First), ramah sentuhan, dan mempertahankan estetika mewah (*luxury minimalist, subtle shadows, refined typography*).

### 3.5 Asset Optimization & Core Web Vitals
- **Responsive Images:** Selalu gunakan filter gambar modern Shopify:
  ```liquid
  {{ image | image_url: width: 1200 | image_tag:
    loading: 'lazy',
    sizes: '(min-width: 1000px) 50vw, 100vw',
    widths: '300, 600, 900, 1200',
    alt: image.alt | escape
  }}
  ```
- **Above-the-Fold (Hero) Media:** Untuk banner LCP utama di atas lipatan layar, ubah `loading: 'lazy'` menjadi `loading: 'eager'` atau `fetchpriority: 'high'`.

---

## 4. Local Development & Shopify CLI

Gunakan Shopify CLI untuk pengujian dan sinkronisasi tema:

```bash
# Menjalankan preview server lokal
shopify theme dev --store <store-subdomain>

# Memeriksa kepatuhan kode dan linting Liquid
shopify theme check

# Menarik perubahan terbaru dari Shopify remote
shopify theme pull

# Mengunggah perubahan ke tema development
shopify theme push --development
```

---

## 5. Agent Workflow & Quality Checklist

Sebelum menyelesaikan tugas atau membuat commit:
1. [ ] **Validasi Sintaks Liquid:** Pastikan tag `{% %}` dan `{{ }}` tertutup sempurna tanpa syntax error.
2. [ ] **Validasi JSON:** Periksa validitas JSON pada setiap file `templates/*.json`, `locales/*.json`, dan block `{% schema %}`.
3. [ ] **Cek Transparansi i18n:** Pastikan tidak ada copy teks statis yang lupa didaftarkan ke locale file.
4. [ ] **Responsivitas & a11y:** Pastikan elemen interaktif memiliki `aria-*` tags yang sesuai, kontras warna memadai, dan berfungsi baik di tampilan layar ponsel.
5. [ ] **No Regression:** Pastikan custom script tidak merusak pelacakan analitik yang ada (GTM, Google Ads, WhatsApp tracker).

