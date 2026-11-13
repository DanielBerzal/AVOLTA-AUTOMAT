# Contrato CLPBANNER

Tipo y carpeta: `CLPBANNER`. Banner de imagen simple en la parte superior de las
Category Landing Pages (CLP): paginas con el contenido de una categoria Magento
enlazadas desde los megamenus de la home. Es el antes llamado "CLP hero banner".
Contrato de autoria estatico, no prueba de auction/SDK. No anadir atributos,
clases, CSS ni comportamientos que no esten en la documentacion de Topsort o que
el usuario no haya indicado.

## Descubrimiento Desde La Home

Por cada home/store/idioma seleccionado:

1. Inventariar TODOS los megamenus del navbar (nombre y Category ID del wrapper,
   igual que MENUBANNER). Excluir el de Brands/Brand Boutique (Brands, Marcas,
   Boutiques, etc.): son Brand Boutiques, no CLP. Registrarlo como EXCLUIDO con su
   nombre. Si no se sabe si una entrada es de marcas: AMBIGUOUS, preguntar.
   Entradas sin megamenu (enlace directo) se tratan como seccion de un solo
   enlace, con Category ID de wrapper desconocido; nunca inventarlo.
2. Recoger SOLO los enlaces de la lista de enlaces del megamenu, con su nivel:
   1 = enlace principal, 2 = sublink, 3 = subsublink, etc., y su orden. No recoger
   enlaces de `.banner-blocks`, promos ni otros bloques. Agrupar por menu padre.
3. Normalizar cada URL (path + query, sin host ni fragmento) y deduplicar en toda la
   home: una CLP se visita y se implementa una vez. Las repeticiones se anotan
   donde aparecen como "duplicada de [ruta del primer menu]", sin segundo bloque.
   Dos URLs distintas con el mismo Category ID de breadcrumb tambien son una CLP.
4. Enlaces a otro host, no categoria (PDP, buscador, PDF, marca) o no HTTPS:
   reportar sin visitar.

Visita autorizada: abrir secuencialmente solo esas URLs del megamenu, mismo host
que la home seleccionada, HTTPS publico, validando cada redireccion. Es la unica
excepcion a "no crawl": no seguir enlaces dentro de la CLP, banners, paginacion,
filtros ni productos. Redireccion a otro host, home o pagina no categoria:
BLOCKED para esa CLP. Un fallo no elimina el resto del lote.

## Category ID

Ultimo crumb del breadcrumb, clase `category[ID]`:

```html
<li class="item category95" aria-current="page"><strong>Liquor</strong></li>
```

ID = sufijo numerico de la clase del ultimo `li` (`95`). Es el mismo en todos los
idiomas, sirve para correlacionarlos; no es Store ID. Sin breadcrumb, sin clase
`category[ID]` o ultimo crumb que no corresponde a la URL: BLOCKED para esa CLP,
sin inventar ID ni comentario WPP.

## Clasificacion Por CLP

Evaluar el primer bloque de contenido de la categoria, encima del listado de
productos/filtros. Confirmar selectores en la pagina real; no asumirlos. Contenido
bajo el listado no cuenta.

Banner simple: un unico bloque de imagen (normalmente Page Builder Image: `figure`
con imagen desktop `pagebuilder-mobile-hidden` y mobile `pagebuilder-mobile-only`)
con link opcional; sin texto, CTA, boton, caption, overlay, slider, video ni
varias imagenes.

| Estado | Evidencia | Accion |
| --- | --- | --- |
| SKIPPED_ALREADY_TOPSORT | Tags Topsort reales en el bloque superior (incluso parciales o mirror) | "Ya esta implementado", sin HTML; parcial: "Estructura incompleta; sin cambios" |
| SIMPLE_NO_TOPSORT | Banner simple sin tags Topsort | Convertir conservando imagen(es), alt y link |
| NO_BANNER | Sin banner ni bloque de contenido superior | Sin HTML ni implementacion: la CLP se omite; solo se reporta "No hay banner" |
| NON_SIMPLE | Banner complejo u otro bloque (slider, texto/CTA, varias imagenes, video, HTML libre, grid, etc.) | Reportar tipo observado; sin implementacion ni HTML |
| AMBIGUOUS | Bloque no clasificable, simple mas otro contenido, imagen sin identificar | Reportar; sin implementacion |
| ERROR/BLOCKED | Acceso, redireccion, sin Category ID, fuente insuficiente | Reportar motivo; sin implementacion |

Tags en scripts, clases o fuera del bloque superior no cuentan. Check solo: sin
HTML, ni siquiera para SIMPLE_NO_TOPSORT. NO_BANNER nunca genera HTML ni bloque
vacio, ni en check ni en implementacion.

El enlace es opcional: un banner existente sin anchor ni `ctaLink:href` no es
incompleto por ese motivo. Si hay anchor, comprobar su binding. No informar
"falta link" por ausencia de destino, `href="#"` o implementacion vacia.
Los bindings de imagen incorrectos siguen siendo incidencias reales.

## HTML De Autoria

Plantilla oficial de Topsort. Mantener literales `context="true"`,
`predefined="true"`, `rank="1"`, id `banner-image` y los dos bindings de imagen.
El anchor y su `data-ts-field="ctaLink:href"` solo se incluyen al conservar un
enlace original. Plantilla con enlace:

```html
<topsort-banner id="banner-image" context="true">
  <topsort-banner-slot rank="1" predefined="true">
    <a data-ts-field="ctaLink:href" href="LINK">
      <figure>
        <img
          class="pagebuilder-mobile-hidden"
          src="DESKTOP_IMAGE"
          alt="ALT"
          data-ts-field="mainImage:src, altText:alt"
        />
        <img
          class="pagebuilder-mobile-only"
          src="MOBILE_IMAGE"
          alt="ALT"
          data-ts-field="mobileImage:src, altText:alt"
        />
      </figure>
    </a>
  </topsort-banner-slot>
</topsort-banner>
```

Variables: `mainImage` (desktop), `mobileImage` (mobile), `altText`. Sustituir
`DESKTOP_IMAGE`/`MOBILE_IMAGE` siempre, para evitar problemas.

- Imagenes y alt: conservar `src` y `alt` reales. Alt vacio queda `alt=""`
  (documentacion: vacio si el banner no tiene informacion importante). Sin
  imagen mobile distinta identificable: AMBIGUOUS, reportar; no duplicar la
  desktop por iniciativa.
- Link: conservar el destino del banner original. URLs RELATIVAS (ver regla comun
  abajo). Al convertir un banner existente sin Topsort, incluir el anchor con
  `data-ts-field="ctaLink:href"` solo si el original tenia enlace. Si no lo tenia,
  no crear anchor ni binding de enlace: `figure` va directamente dentro de
  `topsort-banner-slot`, sin introducir `href="#"`. No marcarlo como defecto.
- Sin banner (NO_BANNER): no se crea implementacion vacia ni HTML para esa CLP
  (esas paginas no se implementan). El reporte solo indica "No hay banner; sin HTML".
- Sin CSS, `width`/`height` ni ningun atributo que no este en la plantilla oficial.
- Fuente live: usar URLs renderizadas, nunca inventar `{{media}}`/`{{store}}`.
  Fuente local con directivas Magento: preservarlas literalmente sin ejecutarlas.

## URLs Relativas (Regla Comun)

Todo `href` generado (en CLPBANNER, TRENDING y MENUBANNER) que apunte al mismo
host de la home se entrega relativo: path + query + fragmento, empezando por `/`
(`/en/196/liquor.html`), para que un cambio de dominio no rompa los enlaces. Se
conserva el segmento `/idioma/store_id/`. Hosts distintos y directivas Magento
quedan intactos. Imagenes (`src`, `srcset`) pueden quedar absolutas. Al comparar
stores, resolver siempre contra la home: relativo y absoluto equivalentes son
iguales; diferencias de Store ID en el path siguen siendo diferencias reales.
Aplica solo a nuevas generaciones; no reescribir salidas anteriores.

## Entrega Por Store E Idioma

Misma estructura TRENDING: `CLPBANNER-[host]/report.md` unico en la carpeta padre y,
en `[idioma]/`, solo HTML. Los grupos equivalentes entre idiomas usan `shared/`,
tambien solo con HTML. HTML por megamenu (seccion), nunca por CLP:
`[SECCION]-store-[IDs]-[idioma].html`; `SECCION` = nombre del megamenu en
mayusculas ASCII sin acentos, espacios a guion (`LIQUOR`, `SPECIAL-OFFERS`).
En grupos multidioma, `[idioma]` enumera los codigos reales ordenados y separados
por `_`, por ejemplo `ch_en_zh`; los IDs tambien van ordenados y sin duplicados.
Colision de nombres: anadir `-[CategoryID]`. Category ID padre va en el
comentario y reporte, no en el filename. No se genera HTML para Brands.

Cada HTML: comentario informativo (miembros `NOMBRE ID:<store_id> - <URL>`,
`IDIOMA: EN`, `<Seccion> - Category ID:<ID del wrapper>`), linea en blanco y, para
cada CLP convertida en orden del menu, el comentario
`<!-- Categoria <nombre real de la categoria> -->`, inmediatamente encima del
comentario WPP, seguido del bloque sin lineas en blanco intermedias. Una linea
en blanco separa las entradas. Validar nombres para no romper comentarios.
Si se comparte entre idiomas, enumerar todos en `IDIOMA: CH, EN, ZH` y los
nombres reales distintos de la categoria en el comentario, separados por ` / `;
son metadatos, no cambios al contenido del banner.

```html
<!-- Categoria Liquor -->
<!-- WPP Cat95 Barcelona Deliver 06/11/26 Topsort -->
<topsort-banner id="banner-image" context="true">...</topsort-banner>

<!-- Categoria Whisky -->
<!-- WPP Cat97 Barcelona Deliver 06/11/26 Topsort -->
<topsort-banner id="banner-image" context="true">...</topsort-banner>
```

(Los `...` son abreviatura de este documento; la salida lleva el bloque completo.)
`Cat[ID]` es el Category ID del breadcrumb de esa CLP; el nombre general y la fecha
siguen la regla comun del implementer. Se copia a Magento desde el comentario
Categoria, incluyendo WPP y su bloque, nunca el informativo. Solo entran CLP
convertidas (SIMPLE_NO_TOPSORT); NO_BANNER no entra.

`report.md`: una seccion por store/idioma (encabezado, `Idioma`, `Resumen`), la
lista de megamenus excluidos y, por cada megamenu, titulo `NOMBRE (Category ID:N)`
con enlace directo al HTML y el arbol de CLP con su nivel por indentacion
(nivel 1, 2, 3...):

```
LIQUOR (Category ID:95) - HTML
- Liquor - category95 - [Link](https://madrid.shopdutyfree.com/es/18/liquor) - SIMPLE_NO_TOPSORT
  - Whisky - category97 - [Link](https://madrid.shopdutyfree.com/es/18/liquor/whisky) - NO_BANNER
    - Scotch - category101 - [Link](https://madrid.shopdutyfree.com/es/18/liquor/whisky/scotch) - NON_SIMPLE: slider
```

Cada linea: nombre, `category[ID]`, enlace Markdown con texto `Link` y la URL
ABSOLUTA de la categoria (host de la home + path; solo en el reporte, el HTML
sigue con `href` relativos), y estado (con tipo observado para
NON_SIMPLE/AMBIGUOUS, motivo para BLOCKED, "duplicada de ..." si procede). Despues
del arbol, el codigo COMPLETO del HTML de la seccion en un bloque fenced `html`,
identico al archivo. No tablas de implementacion. Comparar tambien entre idiomas
y compartir HTML si coinciden megamenu/Category IDs, estados, orden, imagenes
desktop/mobile, alt, textos del banner, links resueltos y estructura funcional.
El idioma y los nombres traducidos de categoria usados solo como metadatos no
impiden agrupar; diferencias reales en alt/textos/URLs si lo impiden. No eliminar
idioma o Store ID de los destinos para forzar igualdad. Un archivo comun por
grupo, todos sus miembros en el comentario y reporte, codigo completo una sola
vez y enlaces directos desde cada store. Resumen raiz:
CLP omitidas (ya implementadas, NO_BANNER, NON_SIMPLE, AMBIGUOUS, BLOCKED) con Store ID,
Category ID y motivo. Chat: misma estructura sin archivos.

## Template Y Revision Propia

[CSV canonico](../assets/clp-banner.csv): once columnas, tres campos; required TRUE:
mainImage, mobileImage; FALSE: altText. El `ctaLink:href` de la plantilla es un
binding del anchor y no figura en el CSV; no anadirlo sin confirmacion del usuario.
CSV entregado no autoriza importacion.

Check enfocado: megamenu de Brands excluido; links solo de listas de megamenu;
sin URLs/Category IDs repetidos; estado por CLP segun tabla; ID unico por pagina;
dos bindings de imagen y atributos literales de las nuevas salidas; anchor y
`ctaLink:href` solo si habia enlace original; `figure`
directamente en el slot para banners originales sin enlace; `src` sustituidos;
sin HTML para CLP NO_BANNER; links relativos y opcionales en fuentes existentes;
comentario Categoria inmediatamente antes de WPP `Cat[ID]`, con nombre real e
ID del breadcrumb; equivalencia entre stores/idiomas y metadatos de todos sus
miembros; arbol y niveles correctos; codigo del reporte igual al HTML;
segunda conversion omite CLP con tags. No afirmar ejecucion CMS, subasta o SDK.
