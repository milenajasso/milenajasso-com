# Notas de desarrollo — milenajasso.com

Documento de traspaso para maquetar páginas nuevas desde **Figma → HTML/CSS estático**.
Léelo antes de empezar una sesión nueva: recoge la estructura del proyecto, el flujo de
trabajo con Figma y los **bugs recurrentes** que ya nos costaron tiempo, para no repetirlos.

Última actualización: 2026-08-04.

---

## 1. Qué es este proyecto

- Sitio **estático**: solo HTML + CSS + JS vanilla. **No hay build system, ni framework, ni npm.**
- Se edita directamente y se sirve tal cual (GitHub Pages, ver `.github/workflows/static.yml`).
- Todo el sitio vive en `web/`.

### Estructura
```
DEV-NOTES.md            # este archivo (raíz del repo, FUERA de web/ — web/ es lo que se despliega)
.github/workflows/
  static.yml            # deploy (GitHub Pages) — sirve el contenido de web/
web/
  index.html            # Home (Milena Jasso)
  priordei.html         # Case study Priordei
  centro-medico.html    # Case study Centro Médico Catalunya
  ponte-guapo.html      # Case study Ponte Guapo (branding Baja California)
  japri-books.html      # Case study Japri Books (UX/UI editorial, app de bienestar)
  chivo-grunon.html     # Case study Chivo Gruñón (packaging cerveza artesanal + branding)
  work.html             # "All work" — grid de carátulas + filtro (Branding/Websites)
  css/
    styles.css          # Compartido por TODAS las páginas (header, footer, menú, tokens)
    priordei.css        # Estilos SOLO de priordei.html
    centro-medico.css   # Estilos SOLO de centro-medico.html
    ponte-guapo.css     # Estilos SOLO de ponte-guapo.html
    japri-books.css     # Estilos SOLO de japri-books.html
    chivo-grunon.css    # Estilos SOLO de chivo-grunon.html
    work.css            # Estilos SOLO de work.html (grid 2 col + filtro)
  assets/
    fonts/              # .woff2 (ver §3)
    priordei/           # imágenes y vídeos del case study Priordei
    centro-medico/      # imágenes/vídeo del case study Centro Médico Catalunya
    ponte-guapo/        # slides del branding (webp) + gif-terra.gif (animado)
    japri-books/        # imágenes + prototype.mp4 (vídeo del prototipo, comprimido 1280w) del case study Japri Books
    chivo-grunon/       # hero + galería (png) + final-gif.gif (animado, 4 frames) del case study Chivo Gruñón
    logo-mj.png, wordmark.png, badge.gif, ...
```
> Este documento vive en la **raíz del repo**, no en `web/`, porque `web/` es exactamente
> lo que se sube al servidor y no debe contener notas de desarrollo.

- `index.html` carga **solo** `styles.css`.
- `priordei.html` carga `styles.css` **y** `priordei.css` (en ese orden).
- **Página nueva** = normalmente `nombre.html` + `nombre.css`, y reutiliza `styles.css`
  para header/footer/menú. Copia el `<header>`, el overlay `#primary-menu` y los dos
  `<script>` del final desde una página existente.

### Cache-busting (IMPORTANTE)
Cada `<link>` de CSS lleva `?v=N`. **Cada vez que edites un CSS, sube el número** o el
navegador (y el preview) sirven la versión cacheada. Estado actual:
- `styles.css?v=27`
- `priordei.css?v=55`
- `centro-medico.css?v=36`
- `ponte-guapo.css?v=6`
- `japri-books.css?v=8`
- `chivo-grunon.css?v=2`
- `work.css?v=11`

---

## 2. Flujo de trabajo con Figma (MCP `figma-desktop`)

Herramientas: `get_design_context`, `get_screenshot`, `get_metadata`, `get_variable_defs`.
Se leen con `ToolSearch` (`select:mcp__figma-desktop__...`) porque son deferred.

**Cómo trabajamos:** el usuario **selecciona un frame en Figma** y luego pides el contexto
sin `nodeId` (usa la selección actual). Pasos:
1. `get_design_context` (sin nodeId) → devuelve React+Tailwind + node-id + colores + assets.
   **Hay que traducirlo** a nuestro HTML/CSS (no usamos React ni Tailwind).
2. `get_screenshot` con el `nodeId` que aparezca → para ver el diseño real.

### Gotchas de Figma (ya nos pasaron)
- **"Nothing is selected":** si el usuario cree que seleccionó pero el MCP dice que no,
  pídele que seleccione de nuevo y reintenta. No inventes el diseño.
- **`get_metadata` de una página entera (`0:1`) explota** (>300k tokens, se guarda a archivo).
  No lo uses para explorar todo; trabaja siempre con el frame **seleccionado** o un `nodeId` concreto.
- **Servidor de assets `localhost:3845`**: las imágenes/SVG del diseño se sirven ahí
  (`http://localhost:3845/assets/xxx.png`). **En algún momento empezó a devolver 500**
  ("Error getting image"). No dependas de descargar assets desde ahí: pídele al usuario
  el asset, o recrea el SVG a mano (p.ej. el icono X de cerrar lo dibujé con `<line>`).
- **Conector `plugin:figma:figma`** (OAuth) **no funciona** en sesión no interactiva. Ignóralo.
- Las medidas de Figma vienen en px absolutos de un frame (p.ej. iPhone 390×812). Para
  maquetar responsive conviene convertir a **%** o `clamp()`, no clavar px fijos.

---

## 3. Fuentes

Todas están ya convertidas a `.woff2` en `assets/fonts/` y declaradas con `@font-face`.
El usuario tiene las originales instaladas en `~/Library/Fonts`; se convirtieron con
`fonttools`/`brotli` (otf/ttf → woff2). **No sustituyas por fuentes "parecidas"** (una vez
usé Fraunces como sustituto y estaba mal — el usuario quiere las originales exactas).

| Familia (CSS)     | Uso                              | Archivos |
|-------------------|----------------------------------|----------|
| `Editor's Note`   | Titulares serif (nombre, H1)     | `EditorsNote-Regular/Italic.woff2` |
| `Aktiv Grotesk`   | UI / cuerpo (grotesca)           | `AktivGrotesk-Regular(400)/Medium(500)/Bold(700).woff2` |
| `Newsreader`      | Cita en cursiva (brandquote)     | `Newsreader-Italic.woff2` |
| `Satoshi`, `Lato` | Home (index)                     | `Satoshi-*`, `Lato-*` |

**Ojo:** `styles.css` y `priordei.css` declaran sus `@font-face` por separado. Si usas un
peso nuevo (p.ej. Aktiv Grotesk **Bold**) en una página que solo carga `styles.css`,
**declara ese `@font-face` en `styles.css`** o saldrá faux-bold. (Ya pasó con el menú.)

---

## 4. Previsualización y verificación (limitaciones)

Se usa el **Browser pane** (`mcp__Claude_Browser__*`) apuntando a `file://.../web/xxx.html`.
Los `file://` se renderizan como **snapshot estático**: útil para ver CSS, con límites.

### Qué funciona y qué no
- ✅ Screenshot, `read_page`, `get_page_text`.
- ✅ `javascript_tool` para **inspeccionar** (`getBoundingClientRect`, `getComputedStyle`)
  y para **forzar estados** (p.ej. `document.body.classList.add('menu-open')` para ver un
  overlay sin poder hacer click real).
- ⚠️ **`resize_window` es poco fiable con snapshots**: a veces el pane renderiza a un ancho
  distinto al pedido. **Comprueba siempre `innerWidth` por JS** antes de fiarte del layout.
  El preset `mobile` (375) es el más consistente. Tras `resize`, **vuelve a `navigate`**
  (con un `?r=N` distinto para evitar caché) para que recalcule.
- ❌ Los **handlers de click/JS interactivo** pueden no ejecutarse en el snapshot → prueba
  los estados forzando la clase/estilo por JS.
- ❌ `window.scrollTo`/`scrollBy` **no mueven** la vista renderizada. Sí funcionan
  `element.scrollIntoView()` y cambiar el viewport.
- Para vídeos: `v.pause(); v.currentTime = X` para fijar un frame concreto.

### Patrón "verify" (para ver una sección aislada)
Extraer una sección de la página a un `_verify.html` temporal, envolverla con los `<link>`
de CSS (¡con la versión `?v=N` correcta!) y screenshot a distintos anchos. Ejemplo:
```python
src = open('priordei.html').read()
s = src.find('<!-- Sección X'); e = src.find('</section>', s) + len('</section>')
sec = src[s:e]
html = ('<!DOCTYPE html><meta charset="UTF-8">'
        '<meta name="viewport" content="width=device-width,initial-scale=1">'
        '<link rel="stylesheet" href="css/styles.css?v=6">'
        '<link rel="stylesheet" href="css/priordei.css?v=55">'
        '<body class="page-priordei"><main class="case" style="padding:0">' + sec + '</main>')
open('_verify.html', 'w').write(html)
```
**Borra `_verify.html` al terminar.** (Cuidado al recortar por `</section>` si hay
`</section>` anidados; ajusta el índice de búsqueda.)

---

## 5. Bugs de CSS RECURRENTES (esto es lo que más tiempo nos costó)

> La mayoría son el mismo tema: **flexbox en `flex-direction:column` + tamaños heredados**.

### 5.1 `flex: 1 1 Npx` se interpreta como ALTURA en columnas
En un contenedor `flex-direction:column`, la "flex-basis" es el eje **principal = altura**.
Así que `flex:1 1 348px` sobre un hijo hace que **348px sea su alto**, estirándolo.
- **Síntoma:** bloques con muchísimo espacio de más, "marcos" grises alargados.
- **Fix móvil:** override a `flex:none` (y `max-width:none`) en la media query.
- Ya pasó en `Legacy audit` y en `.cols3`.

### 5.2 `aspect-ratio` + flex-item con hijos absolutos → altura 0
Un flex-item que solo contiene hijos `position:absolute` y depende de `aspect-ratio` para
su alto puede **colapsar a `height:0`** (bug de Chromium). En cambio funciona si el
elemento está **anidado** en otro flex con `align-items:stretch`.
- **Fix:** no dependas de `aspect-ratio` en el flex-item de primer nivel. Opciones:
  usar una imagen en **flujo normal** que dé el alto, o poner `flex:none` + `height:auto`.
- Ya pasó con `.composition__phone`.

### 5.3 `height:100%` heredado en móvil → dependencia circular que aplasta hermanos
Al apilar en móvil (`flex-direction:column`), si un hijo conserva del desktop
`height:100%` + `flex-shrink:0`, fuerza al contenedor a una altura fija y **encoge a 0**
a los hermanos con `flex-shrink:1`.
- **Síntoma:** elementos "encimados"/solapados; uno de ellos con alto 0.
- **Fix:** en la media query resetea `height:auto` y `flex:none` en los hijos que apilas.
- Ya pasó en `.composition` (la `.composition__col` conservaba `height:100%`).

### 5.5 Apóstrofo escapado en una CUSTOM PROPERTY rompe el parseo
Definir `--serif:'Editor\'s Note', ...` (apóstrofo escapado con `\'` dentro de un valor
de variable) **rompe el parser CSS** y descarta silenciosamente TODAS las reglas que van
después del bloque (aunque `--maxw`, etc. sí resuelvan). Síntoma: la mitad del CSS "no
aplica" sin error visible. **Fix:** usa comillas dobles: `--serif:"Editor's Note", ...`.
(En `@font-face` el `'Editor\'s Note'` sí funciona; el problema es solo en custom properties.)
Ya pasó al crear `centro-medico.css`.

### 5.6 `white-space:nowrap` en columnas estrechas → OVERFLOW horizontal (corta hero/about)
En layouts lado-a-lado que encogen en anchos intermedios (tablet / escritorio estrecho),
un texto con `white-space:nowrap` dentro de una columna flex que se estrecha (p.ej.
`.project__tag` "Branding & Fundraising Campaign", o `.project__name` largo) **no cabe y
desborda a la derecha**, dando **scroll horizontal a toda la página**. Síntoma engañoso:
como el hero y las bandas de color miden solo el ancho del *viewport* (no el ancho de
scroll), sus **fondos se ven "cortados por la derecha"** — parece un bug del hero/about
cuando en realidad el culpable es otro elemento (la sección Work).
- **Diagnóstico:** comparar `document.documentElement.scrollWidth` vs `innerWidth`; recorrer
  `body *` y listar los que tengan `getBoundingClientRect().right > innerWidth`.
- **Aparece al INSPECCIONAR** (ancho exacto de dispositivo o DevTools acoplado que encoge el
  viewport) y **no al redimensionar** la ventana (que no baja tanto y el scrollbar disimula).
- **Fix (index):**
  - Escritorio: nombre en `nowrap` + `flex-shrink:0` (una línea) y tag SIN nowrap (envuelve
    dentro de su columna); apilar Work por debajo de **1100px** (donde el lado-a-lado deja de
    tener sitio para el nombre largo). Escritorio ≥1101 sin cambios.
  - Móvil (≤768): el `.project__head` se **apila** (`flex-direction:column`) y se oculta el
    `.project__divider`; así el nombre largo dispone de toda la columna y no desborda. (El
    nombre en `nowrap`+`flex-shrink:0` SÍ desbordaba a 390px: 311px de nombre + tag no caben
    en ~342px, y el tag "Website" es una sola palabra que no envuelve.)
  - **OJO con el pane del Browser:** no baja de ~461px por navegación normal, pero si fijas
    `resize_window` a 360 y luego `navigate`, sí llega a 360 (ahí se reprodujo/verificó).

### 5.4 Regla general al apilar en móvil
Cuando un layout de escritorio (row, con %/aspect-ratio/height:100%) pasa a columna en
móvil, **resetea explícitamente**: `position`, `left/top`, `width`, `height:auto`,
`aspect-ratio` (si hace falta) y `flex:none`. No asumas que se "desactivan" solos.

---

## 6. Patrones útiles ya establecidos

### Banda a ancho completo (full-bleed) rompiendo el gutter del `.case`
```css
.seccion{
  width:100vw; position:relative; left:50%; right:50%;
  margin-left:-50vw; margin-right:-50vw;
}
```

### Intercalar dos contenedores separados (texto en uno, imágenes en otro)
Cuando en móvil quieres alternar hijos de **dos** contenedores distintos (no se puede con
`order` porque `order` solo ordena hermanos del mismo flex): aplana ambos con
`display:contents` sobre el contenedor común y luego ordena los nietos con `order`.
```css
@media (max-width:768px){
  .fila{flex-direction:column}
  .col-textos, .col-imgs{display:contents}
  .paso[data-step="1"]{order:1}
  .img[data-step="1"]{order:2}
  .paso[data-step="2"]{order:3}
  .img[data-step="2"]{order:4}
}
```
Ya usado en la sección "Beyond the product" (`.others`).

### Recortar una imagen ancha para quedarte con una zona (móvil)
Para un montaje ancho (foto de un móvil dentro de una escena) que en desktop se recorta con
un `img` absoluto: en móvil es más robusto **recortar el PNG con ffmpeg** y usarlo en flujo
normal. Se calcula la región desde los % del recorte de desktop. Ejemplo real:
```bash
ffmpeg -y -i montaje-mobile.png -vf "crop=1054:2174:1521:304" montaje-phone.png
```
Se añade un `<img class="...-mobile">` y se hace swap desktop/móvil con `display:none`.

### Vídeos
`autoplay muted loop playsinline`. Comprimir/escalar con `ffmpeg`. Extraer poster con
`ffmpeg -ss T -i in.mp4 -frames:v 1 poster.jpg`. **PNG transparentes NO se pasan a JPG**
(el alpha se aplana a negro → fondos negros no deseados; ya pasó con las tablets).

### Header / Footer / Menú (compartidos en `styles.css`)
- Footer: `.site-footer` (Figma 255:127), mismo en todas las páginas.
- Header: logo + `.header__toggle` (hamburguesa, **2 líneas**) + `.nav` (inline en desktop).
- **Menú móvil a pantalla completa** (Figma 277:393): overlay `#primary-menu` (`.menu`),
  fondo `#ea4934`, ABOUT/WORK/CONTACT + "Let´s work together" + email. Es un elemento
  **aparte** del `.nav` de escritorio (por eso no hay que tocar desktop). Se abre con
  `body.menu-open` dentro del `@media (max-width:768px)`. El JS bloquea scroll, cierra con
  la "×", con Escape y al pulsar un enlace. El logo oscuro se pone blanco con
  `filter:brightness(0) invert(1)`.

---

## 7. Regla de oro repetida por el usuario

> **"No toques nada de la versión de escritorio, solamente en la versión mobile."**

- Haz los cambios **dentro de `@media (max-width:768px)`** (breakpoint móvil del proyecto;
  también hay `1024px` para tablet y el header cambia en `768px`).
- Añade elementos nuevos **ocultos por defecto** (`display:none` en base) y muéstralos solo
  en la media query móvil, para no alterar desktop.
- Verifica **siempre** desktop después: `getComputedStyle` de los elementos clave a ≥1280px
  para confirmar que no cambió (display, layout, links).
- Sube el `?v=N` del CSS afectado.

## 8. Idioma / interlocución
El usuario escribe en **español**; respóndele en español. Los textos del sitio están en
inglés/catalán (contenido de la marca) — respétalos tal cual.
