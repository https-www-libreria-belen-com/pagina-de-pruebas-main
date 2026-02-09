# LIBRERIA BELEN - Sitio Web Estatico

Sitio web estatico de Libreria Belen (portada, catalogo y pagina de nosotros), preparado para deploy en Cloudflare Pages sin proceso de build.

## Resumen rapido

- Stack: HTML + CSS + JavaScript vanilla.
- Sin backend, sin Node.js, sin npm.
- Datos de productos en `data/products.js` (expuestos como `window.PRODUCTS`).
- Integraciones frontend: WhatsApp, Google Maps embebido y generacion de PDF (jsPDF local).
- Deploy recomendado: Cloudflare Pages con `_headers` y `_redirects`.

## Estructura del proyecto

- `index.html`: portada y carrusel de novedades.
- `catalog.html`: grilla de productos, filtros, carrito, checkout por WhatsApp y boleta PDF.
- `about.html`: informacion institucional.
- `plantilla-formulario.html`: plantilla de formulario.
- `css/style.css`: estilos globales.
- `js/app.js`: logica principal (catalogo, busqueda, filtros, carrito, tema, modales, PDF).
- `data/products.js`: catalogo maestro (`window.PRODUCTS`) y ofertas (`window.OFFERS`).
- `img/` y `images/`: imagenes del sitio y productos.
- `vendor/fontawesome/`: iconografia local.
- `vendor/jspdf/jspdf.umd.min.js`: libreria para PDF local.
- `_headers`: politicas de seguridad y cache para Cloudflare Pages.
- `_redirects`: rutas amigables (`/about`, `/catalog`, etc.).

## Inicio rapido local

No abras los HTML con `file://` porque el catalogo carga `data/products.js` por `<script>`.
Usa un servidor local:

```bash
python3 -m http.server 8080
```

Luego abre:

- `http://localhost:8080/`
- `http://localhost:8080/catalog.html`

## Flujo funcional (como opera hoy)

1. `js/app.js` espera `DOMContentLoaded`.
2. Carga productos desde `data/products.js` con `ensureProductsLoaded()`.
3. Convierte `window.PRODUCTS` a arreglo interno para renderizar tarjetas.
4. Aplica busqueda, filtros (categoria/subcategoria/marca/precio), orden y paginacion.
5. Maneja estado persistente en `localStorage`:
   - `libreriaBelenCart`
   - `libreriaBelenFavorites`
   - `libreriaBelenTheme`
   - `libreriaBelenSearchHistory`

Nota actual de negocio:
- En `js/app.js`, `PRICES_PENDING = true`; por eso el frontend muestra precios como `0` y deshabilita acciones de compra/pago reales.

## Edicion de productos

Fuente de verdad:
- `data/products.js` -> objeto `PRODUCTS` con clave `slug`.

Campos usados por el frontend:
- `title`, `short`, `description`, `longDescription`
- `price`, `rating`, `stock`, `tag`, `category`
- `image`, `gallery`, `details`, `usage`, `reviews`, `link`

Ejemplo minimo:

```js
"mi-producto-slug": {
  "title": "Nombre del producto",
  "short": "Resumen corto",
  "description": "Descripcion corta",
  "longDescription": "Descripcion extendida",
  "price": "S/ 9.90",
  "rating": "4.5",
  "stock": 10,
  "tag": "nuevo",
  "category": "papeleria",
  "link": "https://...",
  "details": ["Punto 1", "Punto 2"],
  "usage": ["Uso 1", "Uso 2"],
  "reviews": [{ "name": "Cliente", "text": "Buen producto" }],
  "image": "assets/img/products/mi-producto.jpg",
  "gallery": []
}
```

Importante:
- Si `image` viene como `assets/img/...`, `app.js` lo normaliza automaticamente a `img/...`.
- Si una imagen falla, el frontend aplica fallback visual.

## Publicar en GitHub

```bash
git init
git add .
git commit -m "Actualizar sitio estatico Libreria Belen"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/TU_REPO.git
git push -u origin main
```

## Deploy en Cloudflare Pages

1. Cloudflare Dashboard -> Workers & Pages -> Create -> Pages -> Connect to Git.
2. Selecciona el repositorio.
3. Configura:
   - Framework preset: `None`
   - Build command: vacio
   - Build output directory: `/`
4. Ejecuta `Deploy`.

Archivos especiales que Cloudflare aplicara automaticamente:
- `_headers` (seguridad y cache).
- `_redirects` (rutas amigables y canonical home).

## Cache y cabeceras configuradas

En `_headers`:
- HTML: `max-age=0, must-revalidate`.
- Assets (`/css`, `/js`, `/data`, `/images`, `/img`, `/vendor`): `max-age=604800`.
- Seguridad: `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`.

## Rutas amigables actuales

En `_redirects`:
- `/about` -> `/about.html` (200)
- `/catalog` -> `/catalog.html` (200)
- `/formulario` -> `/plantilla-formulario.html` (200)
- `/index` -> `/` (301)

## Troubleshooting rapido

- El catalogo no carga:
  - Verifica que exista `data/products.js`.
  - Revisa consola por error al cargar script.
  - Ejecuta desde servidor local o deploy, no desde `file://`.

- Cambios no se ven en produccion:
  - Purga cache en Cloudflare.
  - Fuerza recarga del navegador (`Ctrl+F5`).

- El carrito/favoritos se comporta raro en pruebas:
  - Limpia `localStorage` del navegador para este dominio.

## Checklist breve antes de deploy

- `index.html`, `catalog.html` y `about.html` abren sin errores.
- `catalog.html` renderiza productos y filtros.
- WhatsApp y enlaces de mapas responden correctamente.
- Imagenes clave cargan sin 404.
- `_headers` y `_redirects` estan en la raiz del repo.
