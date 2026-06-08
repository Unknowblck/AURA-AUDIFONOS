# AURA Shopify — Referencia del proyecto

> Snapshot del proyecto AURA al 2026-06-08. Sirve como base para próximas landings premium con animaciones scroll-driven en Shopify.

## 🎯 Stack

- **Repo GitHub:** https://github.com/Unknowblck/AURA-AUDIFONOS (rama `main`)
- **Tienda Shopify:** `aurahead.shop` / `wmzdsd-ux.myshopify.com`
- **Tema base:** Savor 3.5.1 (Online Store 2.0)
- **Producto único:** `audifonos` (handle exacto)
- **Método de pago:** Releasit COD Form (Cash on Delivery) + checkout estándar Shopify

## 📂 Arquitectura

```
aura-shopify-theme/
├── assets/
│   ├── aura-frame-001.webp ... aura-frame-056.webp  ← 56 frames hero (1600px WebP q90)
│   ├── aura-hero-poster.webp                        ← LCP preload (~50KB)
│   ├── aura-final-frame.webp                        ← poster sección Funciones
│   ├── aura-transition.mp4                          ← video original 1080p (loop en Funciones)
│   └── [resto del tema Savor sin tocar]
├── sections/
│   ├── aura-landing.liquid    ← LA SECCIÓN GIGANTE: TODO el landing aquí
│   └── [resto del tema Savor sin tocar]
├── templates/
│   ├── index.json             ← Simplificado: solo renderiza aura-landing
│   └── [resto del tema Savor sin tocar]
├── layout/
│   └── theme.liquid           ← Modificado para ocultar header/footer Savor en home
└── [config, locales, snippets, blocks: Savor original]
```

## 🛠️ Workflow git → ZIP → Shopify

```bash
# 1. Hacer cambios en sections/aura-landing.liquid
# 2. Commit con identity inline (importante en Windows)
git -c user.email="juanjoseco589@gmail.com" -c user.name="Juan Jose" commit -m "..." --quiet

# 3. Push
git push origin main

# 4. Generar ZIP con PowerShell
$src = "C:\Users\USUARIO\Claude_code\desarrollo\aura-shopify-theme"
$dst = "C:\Users\USUARIO\Downloads\aura-theme-vXX.zip"
Compress-Archive -Path "$src\assets","$src\blocks","$src\config","$src\layout","$src\locales","$src\sections","$src\snippets","$src\templates" -DestinationPath $dst -CompressionLevel Optimal

# 5. El cliente sube el ZIP en Shopify Admin → Online Store → Themes → Add theme → Upload zip file
# 6. Vista previa → Publicar
```

### Si push falla por divergencia (Shopify GitHub sync hizo commit):
```bash
git -c user.email="juanjoseco589@gmail.com" -c user.name="Juan Jose" pull --rebase origin main
# Resolver conflictos si hay
git push origin main
```

### Si quedas en detached HEAD:
```bash
git branch backup HEAD          # respaldo del commit
git checkout main
git fetch origin
git reset --hard origin/main
git cherry-pick backup          # aplicar tu commit limpio
git push origin main
```

## 🎬 Stack de animaciones implementadas

| Animación | Tipo | Trigger |
|---|---|---|
| Hero scroll-driven frame sequence (56 frames) | Canvas + rAF | Scroll progress 0→1 |
| Hero cross-fade text stages (4) | CSS grid stack + opacity | Scroll progress segmentado |
| Reveals con stagger | IntersectionObserver | Element enters viewport |
| Navbar scrolled (backdrop blur) | Scroll listener | window.scrollY > 24 |
| Mobile menu burger → X | CSS transforms | Click toggle |
| Urgency pulse naranja | `@keyframes` infinito | Mount |
| Scroll cue (barra vertical) | `@keyframes` infinito | Mount |
| Button hover lift + arrow slide | CSS transitions | :hover |
| Card hover elevation | CSS transitions | :hover |
| Nav link underline | `::after` width transition | :hover |
| Func video loop (cuando visible) | IntersectionObserver | Element near viewport |

**Fallbacks críticos:**
- IntersectionObserver con timeout 2.5s de seguridad para reveals (iOS Safari bug).
- `overflow-x: clip` en `.aura-landing` (NO `hidden` — rompe sticky).
- `prefers-reduced-motion` en pulse, reveals, cue.

## 🛒 Flow de compra (validado)

```
Click Comprar (cualquier botón en home)
   ↓ (AJAX POST /cart/add.js)
Add Audifonos al cart silenciosamente
   ↓ (window.location.href = '/checkout')
Checkout estándar Shopify
   ↓ (cliente elige "Pago Contra Entrega")
Releasit COD form aparece como método de pago
   ↓
Cliente llena nombre/teléfono/dirección/ciudad
   ↓
Orden creada en Shopify con estado COD
```

**Forma de los 4 botones Comprar:**
```liquid
{%- if aura_variant_id != '' -%}
<form action="/cart/add?return_to=/checkout" method="post" class="aura-buy-form" data-product-handle="audifonos">
  <input type="hidden" name="id" value="{{ aura_variant_id }}">
  <input type="hidden" name="quantity" value="1">
  <button type="submit" name="add" class="btn btn-primary">Comprar ahora</button>
</form>
{%- else -%}
<a href="{{ buy_url }}" class="btn btn-primary">Comprar ahora</a>
{%- endif -%}
```

## ⚡ Performance mobile

| Asset | Antes | Ahora |
|---|---|---|
| Hero poster | 1.2 MB (PNG) | 50 KB (WebP q92) |
| Final frame Funciones | 1.5 MB (PNG) | 24 KB (WebP q92) |
| 56 frames hero (calidad 4K) | N/A | 2.7 MB total |
| Video Funciones (preload="none") | 1.7 MB cargados | 0 KB hasta visible |
| **Initial paint mobile** | 4.1 MB | **~300 KB** |

**Estrategias:**
- `<link rel="preload" as="image" fetchpriority="high">` para hero poster + frame 001.
- Lazy load: primeros 8 frames en `for` loop, resto en `requestIdleCallback`.
- Video con `preload="none"` + IntersectionObserver con `rootMargin:'200px 0px'`.
- Fuentes Google con `display=swap` (no bloquea render).

## 🚫 Lo que NO funciona en Shopify (no repetir)

- ❌ **Iframe a `/products/X` para modal popup** → Shopify manda `X-Frame-Options: DENY`. Solo Shopify Plus puede cambiarlo.
- ❌ **JS `fetch('/cart/add.js') + redirect a /checkout` sin Releasit configurado** → bypassa el COD.
- ❌ **Conjunto release Free + popup mode** → no existe, es feature Plus/Pro.
- ❌ **`overflow-x: hidden` en wrapper con sticky inside** → iOS Safari rompe el sticky. Usar `clip`.
- ❌ **Templates `.json` con comentarios** → Shopify falla en parseo.

## 📋 Configuración Releasit confirmada

- **App embed activado** en theme editor (Customize → App embeds → Releasit toggle ON).
- **Visibility:** "Both cart and product pages" en Settings & Integrations.
- **Hide Buy Now button on product pages:** ON.
- **Disable Releasit on home page:** OFF (sí corre en home, aunque no detecta forms ahí).
- **Plan:** Free (sin Sticky Buy Now popup).

## 🎨 Brand identity referencia rápida

```css
--aura-bg: #F5F3EE;      /* cream */
--aura-ink: #080808;     /* dark */
--aura-accent: #FF5A1F;  /* orange */
--aura-muted: #8A857C;   /* secondary */
--aura-cloud: #FAF8F3;   /* cards */
```

- **Display font:** Anton Regular (UPPERCASE, tracking .005-.14em)
- **Body font:** Inter 300/500/600 (line-height 1.5-1.6)
- **Easing premium:** `cubic-bezier(.22,.61,.36,1)`
- **Border radius cards:** 18-22px
- **Shadow primary button:** `0 12px 30px -12px rgba(255,90,31,.7)`

## 📦 Brand kit completo

Carpeta: `diseno/aura-brand-kit/` — incluye documento HTML editorial, imágenes de marca y producto, recortes para redes (FB cover 1640×624, OG 1200×630, IG square/story), favicons (SVG + ICO + PNG 16-512).

## 📜 Historial de versiones del ZIP

| Versión | Commit | Cambio principal |
|---|---|---|
| v1 | `d1e22e5` | Initial tema Savor |
| v3 | `be92530` | Fixes mobile + animaciones |
| v6 | `8db9c14` | Upscale 4K Topaz frames |
| v7 | `1b70a5f` | Sin eyebrows |
| v8 | `028661d` | 4 text stages cross-fade |
| v9 | `5db08a4` | Releasit redirect |
| v11 | `df06074` | Revert iframe modal |
| v12 | `9a441f1` | Product forms nativos |
| v14 | `3576d52` | **Checkout directo (FINAL)** |

---

**Autores:** Juan José (cliente) + Claude (implementación)
**Última actualización:** 2026-06-08
