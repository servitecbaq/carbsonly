# Carbsonly — BFIC Fuel Systems · sitio web

Tienda de carburadores en Glendale (Los Ángeles), California, desde 1965. Este repositorio contiene **el sitio reconstruido** (2026-10): HTML/CSS/JS puro, 100 % estático, sin dependencias ni herramientas de build.

> ⚠️ **Si abres este proyecto en un editor/IA:** la carpeta hermana `C:\Downloaded Web Sites\www.carbsonly.com` es el **sitio original de 2003-2010, solo de lectura**. Nunca se edita, ni se borra, ni se publica. Todo el trabajo va a `carbsonly-new` (generación) y de ahí al clon Git. Y **las páginas de catálogo de este repo son generadas**: editarlas a mano se pierde al re-ejecutar el pipeline (ver §1 y §2).

- **URL publicada:** https://servitecbaq.github.io/carbsonly/
- **Dominio previsto:** https://www.carbsonly.com (aún no conectado)
- **Contacto real del negocio:** 820 Thompson Ave, Unit #4, Glendale, CA 91201 · 818-845-9000 · mail@carbsonly.com · lun–vie 8:00–17:00 hora del Pacífico · *Se Habla Español*

---

## 1. Cómo se construye y publica este sitio (lean esto antes de tocar nada)

El sitio **no se edita a mano de forma aislada**: se genera con un pipeline de 4 scripts Node que viven **fuera de este repo**, en `C:\Downloaded Web Sites\` (en la máquina de David Fragozo):

| Script | Qué hace | Claves |
|---|---|---|
| `migrate-carbsonly.js` | Regenera las ~200 páginas del catálogo desde el sitio viejo | Lee de `C:\Downloaded Web Sites\www.carbsonly.com` (SOLO lectura, nunca se modifica). Re-ejecutable. |
| `seo-carbsonly.js` | Escribe el `<head>` de cada página (bloque `ANALYTICS` + bloque `SEO`), genera `sitemap.xml`, `robots.txt`, `404.html` y `js/search-index.js` | Idempotente. **Vuelve a poner el dominio `www.carbsonly.com`**, por eso siempre va seguido de `set-domain.js`. |
| `check-links.js` | Verifica enlaces e imágenes rotas en los 221 archivos HTML | Debe dar `0` antes de publicar. |
| `set-domain.js` | Cambia la base de dominio en canonical/og/JSON-LD/`_next`/sitemap/robots | Reversible. `node set-domain.js https://servitecbaq.github.io/carbsonly` o `https://www.carbsonly.com`. |

**Orden obligatorio:** `migrate` → `seo` → `check-links` → **`set-domain` al final**.

```bash
cd "C:/Downloaded Web Sites"
node migrate-carbsonly.js && node seo-carbsonly.js && node check-links.js
node set-domain.js https://servitecbaq.github.io/carbsonly
node serve-carbsonly.js 8080      # vista previa en http://localhost:8080
```

Hay un quinto script, `serve-carbsonly.js`: servidor estático local sobre `carbsonly-new` (puerto 8080 por defecto).

### Las tres carpetas (no las confundas)

1. `C:\Downloaded Web Sites\www.carbsonly.com` → **el sitio original descargado (2003-2010)**. Es la fuente de datos. **NUNCA se modifica ni se borra.**
2. `C:\Downloaded Web Sites\carbsonly-new` → **área de construcción** (esta es la fuente de verdad del contenido).
3. `D:\02-Clouds\GoogleDrive\GitHub\carbsonly` → clon Git que se sincroniza y se pushea a `github.com/servitecbaq/carbsonly` (rama `main`, GitHub Pages desde la raíz).

### Publicar

```bash
MSYS_NO_PATHCONV=1 robocopy "C:\Downloaded Web Sites\carbsonly-new" "D:\02-Clouds\GoogleDrive\GitHub\carbsonly" /E /XD .git /NFL /NDL /NJH /NJS /NP /R:1 /W:1
# exit code 3 = correcto. Luego, dentro del clon: git add -u && git commit && git push origin main
```

⚠️ **`/XD .git` no es opcional.** `carbsonly-new` contiene un `.git` oculto de 86 MB con historia huerfana (root `4cf847a`, un clon viejo). En 2026-10-19 se copió encima del `.git` del clon y rompió el historial local (push rechazado por *unrelated histories*). Recuperación usada: `git branch backup <commit>`, `git reset --hard origin/main`, re-copiar excluyendo `.git`.

⚠️ En Git Bash el flag `/E` se corrompe a `E:/` si no se pone `MSYS_NO_PATHCONV=1` delante.

⚠️ **GitHub Pages no publica al instante.** Medido: entre 2 y 45 minutos (los runners gratuitos de `pages build and deployment` a veces están en cola). Verifica con `curl` + cache-buster antes de dar nada por vivo:

```bash
curl -s "https://servitecbaq.github.io/carbsonly/?v=$(date +%s)" | grep -o "<title>[^<]*"
```

---

## 2. Inventario real (al 2026-10-23)

- **221 archivos HTML** = 21 `.html` + 200 `.htm`. **Ojo: la extensión NO distingue página a mano / página generada** (son a mano los 12 `.html` de la raíz; hay 9 `.html` generados en subcarpetas, herencia del sitio viejo). **93 MB** (el `.git` de `carbsonly-new` añade 86 MB extra que NO se publica). GitHub Pages límite 1 GB → holgado.
- **209 URLs en `sitemap.xml`** y **209 páginas con el tag de Clarity**.
- **12 páginas sin tag de analítica, a propósito:** 9 stubs de redirección (`frames/*.htm` ×5, `index.htm`, `index-1.htm`, `inforequest/*` ×2, que solo hacen meta-refresh a su página nueva) + `404.html`, `search.html`, `thank-you.html` (las tres con `noindex`).

### Páginas hechas a mano (raíz, `.html`) — edítalas directamente

`index.html` · `cars-trucks.html` · `marine.html` · `weber.html` · `industrial-farm.html` · `company.html` · `contact.html` · `faq.html` · `map-directions.html` · `search.html` · `thank-you.html` · `404.html`

### Páginas generadas (subcarpetas `.htm`) — **no las edites a mano, se pisotean**

`carbs/` `marine/` `weber/` `industrialandZenithcarburetors/` `Zenithfloatandparts/` `zenithmodel228/` `zenithmodel33/` `zenithmodel87/` `zenith28ada/` `zenithlpg/` `TLCParts/` `Spacers/` `tbiinjection/`

Para cambiar algo de esas 200 páginas hay que cambiar la **plantilla o los mapas dentro de `migrate-carbsonly.js`** y re-ejecutar el pipeline.

---

## 3. Reglas de marca y contenido (las corrigió el dueño; no volver a romperlas)

- La marca se escribe **`Carbsonly`**, una sola palabra y con C mayúscula. **Nunca `Carbs Only` separado.** En minúsculas solo donde debe ser minúscula: URLs (`carbsonly.com`, `/carbsonly/`), el email `mail@carbsonly.com` y el logo (la imagen ya dice "carbsonly"). Al hacer reemplazos globales, cuidado con mayusculizar rutas: GitHub Pages distingue mayúsculas.
- `servitecbaq` (la empresa de desarrollo de David) **siempre en minúsculas y pegado**, en la firma del footer.
- Audiencia: **personas mayores**. Tipografía base 18 px (`1.125rem`), blancos táctiles grandes (`.btn` = `min-height:54px`, CTA de navegación = 48 px, campo de búsqueda = 54 px), contraste alto, navegación simple, teléfono siempre clicable.
- Idioma: inglés (como el original) con nota *Se Habla Español*.
- Copy local: "Los Angeles" / "Glendale (Los Angeles)" en títulos y ledes de las páginas clave; footers con *"Serving Los Angeles and shipping worldwide"*. El envío mundial es **cierto** (lo dice el sitio original).
- **Los ~200 títulos de catálogo se dejan limpios, sin ciudad**: no hacer keyword stuffing.

## 4. Decisiones de alcance (no re-abrir sin preguntar)

- **Videos: eliminados del sitio** (2026-10-17). El original tiene 44 archivos en `videos/` (1.3 GB, el mayor de 138 MB). Si algún día se quieren, subirlos a YouTube e incrustar. `temp_videos/` en el repo solo quedó con 9 imágenes sueltas heredadas.
- **Páginas de fotos sueltas de `Graphics/`: eliminadas** (116 páginas). Las imágenes sí se conservan porque las usan las fichas de producto.
- **Sin formulario de tarjeta de crédito**: enviar números por correo viola PCI. La página original `testfolder/testsecureorderfrom.htm` era una prueba sin form real y no se recreó; en su lugar hay aviso *"Paying by credit card? Call us…"*.
- **Sin SPA ni framework**: el prototipo "suave" original era una SPA de una página; se descartó la arquitectura (rompería SEO y buscador) y solo se adoptó su lenguaje visual.
- Los testimonios de `index.html` (4) y `company.html` (14) son **reales**, copiados del sitio original (`carbs/companyprofile.htm`). No son inventados; no añadir schema.org `Review`/`AggregateRating`.

## 5. Formulario de contacto (FormSubmit.co)

`contact.html` hace POST a `https://formsubmit.co/soporte@servitecbaq.com` con `_cc=mail@carbsonly.com`, `_captcha=false`, `_template=table`, `_subject="New inquiry from carbsonly.com website"`, `_honey` (honeypot anti-bot, con `display:none`) y `_next=<dominio>/thank-you.html`. Campos visibles: name, email, phone, make, model, year, carbnumber, country, applicationtype, inforequested, description. Está **activado** (verificado 2026-10-17).

- El target es el correo de David porque él no tenía acceso a `mail@carbsonly.com`; la copia llega al negocio por `_cc`. Para invertirlo hace falta re-activar el nuevo target.
- **FormSubmit rechaza envíos sin `Referer` de página web**: probar con `curl` siempre falla. Hay que probar desde navegador con la página servida por HTTP (no `file://`).
- `_next` lo actualiza `set-domain.js`.

## 6. Analítica: Microsoft Clarity

- **Project ID `z2axvbs0ld`**, gratis, sin cuenta publicitaria.
- El snippet **no está escrito a mano en las páginas**: lo genera `seo-carbsonly.js` (constantes `CLARITY_ID` y `CLARITY_JS`) dentro de `<!-- ANALYTICS:START --> … <!-- ANALYTICS:END -->`, insertado justo antes del bloque SEO. Es idempotente (el script borra el bloque previo antes de re-insertarlo). Añadirlo a mano en un `.htm` de catálogo se pierde al re-ejecutar `migrate`.
- Verificado funcionando: el tag descarga `scripts.clarity.ms` y hace POST a `t.clarity.ms/collect`.
- Clarity escribe cookies `_clck` y `_clsk`. El sitio **no lo oculta**: el item 11 del FAQ dice explícitamente que no se usan cookies publicitarias ni de redes sociales y que se usa Clarity. Si se prefiriera sesión sin cookies, es un interruptor del panel de Clarity (Settings → Setup → Advanced settings → Cookies OFF), no un cambio de código.
- Recomendado en el panel (lo hace David, es configuración externa): bloquear la captura de lo que se teclea en los formularios.

## 7. SEO implementado

En cada página indexable, generado por `seo-carbsonly.js`:

- `meta description` (si falta, se sintetiza del `<h>`/`<p>`), `canonical`, Open Graph + Twitter card, `theme-color #1F2A3C`.
- Geo metas `geo.region US-CA`, `geo.placename`, `geo.position`, `ICBM` (34.1425, -118.2551).
- JSON-LD **`LocalBusiness`** en todas: `foundingDate 1965`, `areaServed [Los Angeles, Southern California, United States, Worldwide]`, `hasMap`, `geo`, dirección, teléfonos, horario.
- JSON-LD **`BreadcrumbList`** en páginas de subcarpeta; **`FAQPage`** en `faq.html` (se construye desde los `<details><summary>` — añadir una pregunta = añadirla en el HTML y re-ejecutar el script); **`WebSite` con `SearchAction`** solo en `index.html` (apunta a `search.html?q=`).
- Un solo `<h1>` por página (el script degrada los `<h1>` extra a `<h2>`), `loading="lazy"` y `width`/`height` en imágenes.
- `robots.txt` permite todo y apunta al sitemap. `404.html`, `search.html`, `thank-you.html` y los stubs de redirección llevan `noindex, nofollow`.

**Buscador del sitio:** estático, sin backend. `seo-carbsonly.js` genera `js/search-index.js` (`window.SEARCH_INDEX`, 209 entradas con título/descripción/ruta) y `search.html` filtra con JS vainilla; acepta `?q=` para deeplinks.

## 8. Diseño

- `css/style.css` — sistema de diseño. Paleta cálida: fondo crema `#F4F1EA`, navy `#1F2A3C`, acento terracota `#7A3B25`, líneas `#D9D3C7`. Tipografías Playfair Display (títulos) + Inter (texto) vía `@import` de Google Fonts. Radios 4 px, sombras suaves. Portada con hero de 2 columnas y foto; trust-strip navy; catálogo en "hoja blanca" (`.legacy-content`).
- `css/legacy.css` — estilos heredados que necesitan las páginas de catálogo antiguas (tablas, `align=center`, etc.).
- `Graphics/Logos/og-image.png` (1792×1024) se regenera a mano si cambia la paleta; `favicon2.ico` y `logo.jpg` son del sitio original.
- No hay plantillas tipo templ/hbs: el header/footer de las 200 páginas de catálogo están **materializados** por `migrate-carbsonly.js` (función de plantilla dentro del script). Cambiar el footer ⇒ editar el script y re-ejecutar.

## 9. Trampas conocidas (esto muerde)

1. **`set-domain.js` va al final.** `seo-carbsonly.js` escribe el dominio de producción (`www.carbsonly.com`); si publicas sin pasar `set-domain.js`, toda la web queda apuntando a un dominio que aún no resuelve a este sitio.
2. **Detector de redirecciones en `seo-carbsonly.js`:** exige retardo numérico o `url=` dentro de `http-equiv="refresh"`. Varias páginas originales traen un `<meta http-equiv="refresh" content="">` **vacío** en el `<body>` (basura de 2003). Si se relaja esa regex, esas páginas se quedan sin SEO, sin analítica y con un `noindex` espurio — exactamente el bug que dejó `marine/zenithmarine.htm` fuera del sitemap hasta 2026-10-23.
3. **`walk()` de `seo-carbsonly.js` excluye `404.html`, `thank-you.html` y `search.html`** y les añade `noindex` si no lo traen. Si se crea una página nueva parecida, hay que decidirla en esa lista.
4. **Editar HTML generado no persiste**: `migrate-carbsonly.js` (200 páginas) y `seo-carbsonly.js` (todos los `<head>`) sobrescriben. Las páginas de la raíz `.html` sí son editables salvo su bloque `ANALYTICS`/`SEO`.
5. **El footer muestra `www.carbsonly.com` como texto** de copyright aunque el sitio corra en github.io: es a propósito (es el dominio del negocio, no una URL canónica).
6. **Avisos `LF will be replaced by CRLF`** en `git diff`: los produce `.gitattributes`, son inocuos.
7. **Caché del navegador:** tras publicar, avisar de Ctrl+F5; Pages sirve `max-age` en algunos assets.
8. **`.nojekyll` NO existe en este repo**, así que Pages pasa los archivos por Jekyll en cada build (más lentitud y riesgo con nombres raros como `CarterBall&Ball1Bl.htm`). Crear un `.nojekyll` vacío en la raíz es una mejora recomendada, pendiente de aprobación del dueño.

## 10. Pendientes (mayoría: solo los puede hacer David)

- [ ] **Google Business Profile** de la ficha en Glendale — es el factor #1 para aparecer en búsquedas locales de Los Ángeles.
- [ ] **Google Search Console + Bing Webmaster Tools**: verificar `servitecbaq.github.io/carbsonly` y enviar `sitemap.xml`.
- [ ] **Dominio de producción**: `node set-domain.js https://www.carbsonly.com` + CNAME/`404.html` en la config de Pages, cuando el dominio apunte. El dominio viejo tiene historial de backlinks: migrar con redirecciones 301 desde `www.carbsonly.com` (no desde github.io).
- [ ] Opcional: borrar el `.git` huérfano de `carbsonly-new` (86 MB) para eliminar el riesgo del robocopy (§1).
- [ ] Opcional: versionar los 4 scripts del pipeline dentro del repo (hoy solo existen en el portátil).
- [ ] Opcional: subir a YouTube los videos del sitio original (44 archivos, 1.3 GB en `www.carbsonly.com/videos`) y re-incrustarlos.

## 11. Verificación antes de dar nada por terminado

1. `node check-links.js` ⇒ `Enlaces/imagenes rotos: 0`.
2. Conteos: `grep -l 'clarity\.ms/tag' $(find . -name "*.htm*") | wc -l` ⇒ 209; URLs en `sitemap.xml` ⇒ 209.
3. Navegador sobre `http://localhost:8080` (no `file://`: FormSubmit y Clarity no funcionan igual): comprobar `<h1>` único, header responsive con menú hamburguesa, buscador, y que en Network aparezcan `clarity.ms/tag/z2axvbs0ld` y `t.clarity.ms/collect`.
4. Tras el push: `curl` con cache-buster hasta ver el cambio servido, y avisar de Ctrl+F5.
