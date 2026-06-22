# Auditoría de Conversión — AURA Audífonos (aurahead.shop)

> Fecha: 2026-06-21 · Repo: Unknowblck/AURA-AUDIFONOS (rama main, conectada a Shopify por GitHub) · Tema: Savor 3.5.1 · Producto único: `audifonos`

## Resumen ejecutivo

El landing está **bien construido a nivel técnico** (performance móvil optimizada, animaciones con fallbacks, identidad visual sólida, narrativa correcta). El problema **no es el código**, son **decisiones de UX y de flujo de compra** que están costando ventas — sobre todo en **móvil** y en el **flujo de pago contra entrega (COD)**, que es justo donde está la mayoría del tráfico de ads en Colombia.

Todo el landing vive en `sections/aura-landing.liquid` (996 líneas). El historial de git muestra **6 cambios seguidos** del flujo COD (página de producto → modal → iframe → forms nativos → /cart → checkout directo): nunca quedó estable. Esto se nota en la experiencia.

---

## 🔴 CRÍTICO — afecta ventas directamente

### 1. En móvil NO hay botón de compra visible hasta el final del hero
- El "Comprar" del nav está **oculto en móvil** (`.nav-cta{display:none}` hasta 880px).
- El primer CTA visible aparece en la **etapa 3 del hero**, tras scrollear ~1.5 pantallas (hero de 150vh en móvil).
- **No existe barra fija (sticky)** de compra.
- → El usuario móvil que quiere comprar rápido no tiene dónde. **Fix:** barra sticky inferior con precio + "Pedir contra entrega" siempre visible.

### 2. El "pago contra entrega" no está consolidado ni es explícito
- Los 5 botones hacen lo mismo en código (form → `/cart/add` → JS → `/checkout`), pero terminan en el **checkout completo de Shopify**, donde COD es solo una opción más, escondida. En la página de producto el experience es distinto (header Savor + add to cart nativo).
- En **ningún lugar** del landing se dice "Paga cuando recibas / Contra entrega" — que es **EL** argumento de conversión #1 en Colombia. Solo dice "Pago seguro".
- **Fix:** unificar TODOS los botones a un único flujo COD simple + hacer el COD explícito (badges "Paga al recibir" arriba y junto a cada botón).

### 3. Precio inconsistente
- Landing: **$119.900** (hardcoded en `index.json`). Producto real en Shopify: **$109.900**.
- Genera desconfianza/confusión. **Fix:** leer el precio real del producto dinámicamente, no escribirlo a mano.

### 4. Links legales y de políticas muertos (todos `href="#"`)
- Envíos, Devoluciones, Garantía, Contacto, Términos, Privacidad y todas las redes sociales apuntan a `#`.
- Mata confianza **y puede tumbar los anuncios** (Meta exige política de privacidad/devoluciones reales y accesibles).
- **Fix:** crear las páginas de políticas en Shopify y enlazarlas; WhatsApp/redes reales.

---

## 🟠 IMPORTANTE — confianza y conversión

### 5. Falta WhatsApp
En COD Colombia, un botón flotante de WhatsApp sube mucho la conversión (la gente confirma antes de pedir). Hoy no existe.

### 6. Falta FAQ que resuelva objeciones COD
¿Cuándo llega? ¿Puedo devolver? ¿Es original? ¿Pago al recibir sin adelantar nada? Sin esta sección, el cliente dudoso se va.

### 7. Prueba social genérica / no verificable
Testimonios con nombres y cargos inventados, "+25.000 vendidos", "4.9/5" sin fuente. Riesgo de credibilidad y de políticas de ads. **Fix:** reseñas reales (capturas, fotos de clientes, o app de reseñas).

### 8. Urgencia estática
"Quedan 17 unidades" es un número fijo. Si nunca cambia, pierde efecto. **Fix:** stock/temporizador real o rotación.

### 9. Copy de CTAs disperso
"Comprar" / "Comprar ahora" / "Lo quiero ahora". **Fix:** unificar a uno orientado a COD ("Lo quiero — pago al recibir").

---

## 🟡 ANIMACIONES / UX

### 10. Hero demasiado largo (200vh desktop / 150vh móvil)
Obliga a scrollear 1.5–2 pantallas antes de llegar al contenido y al CTA. **Fix:** reducir altura y/o CTA persistente.

### 11. Secuencia de 56 frames en canvas atada al scroll
Elegante en gama alta, pero puede ir a tirones en Android económico (gran parte del tráfico COD). Tiene throttling rAF (bien), pero conviene QA real en un móvil de gama media.

### 12. El CTA solo aparece en la etapa final del hero
Las etapas 0–2 no tienen botón. Combinado con #1, deja al móvil sin acción durante todo el hero.

---

## ✅ Lo que está BIEN (no tocar)
- Performance móvil: preload de poster + frame 1, lazy de frames, video diferido. Muy bien hecho.
- Fallbacks: IntersectionObserver con timeout de seguridad, `prefers-reduced-motion`, `overflow-x:clip`.
- Identidad visual coherente (Anton + Inter, paleta cream/ink/orange).
- Estructura narrativa correcta: problema → solución → funciones → ciencia → testimonios → precio.

---

## Plan de acción propuesto (por prioridad)

| # | Acción | Impacto | Esfuerzo |
|---|--------|---------|----------|
| 1 | Barra sticky de compra en móvil + CTA visible antes | 🔥 Alto | Bajo |
| 2 | Unificar TODOS los botones a un solo flujo COD + COD explícito | 🔥 Alto | Medio |
| 3 | Corregir precio (dinámico desde el producto) | Alto | Bajo |
| 4 | Crear y enlazar páginas de políticas + WhatsApp | Alto | Bajo |
| 5 | FAQ de objeciones COD | Medio | Bajo |
| 6 | Reseñas reales / honestas | Medio | Medio |
| 7 | Acortar hero / suavizar animación en gama baja | Medio | Medio |

---

**Decisiones pendientes con el cliente:** (1) qué experiencia COD exacta queremos, (2) cómo desplegar de forma segura (branch + tema borrador vs push a main), (3) alcance de esta primera ronda.
