# Contrato CLPCONTENT

Bloque editorial inferior de una CLP, de dos o tres columnas con imagenes,
titulo/CTA, descripcion y enlaces. Tipo y carpeta: `CLPCONTENT`. No es el
banner superior CLPBANNER, ni TRENDING, ni un carrusel de productos. Solo
transformacion local autorizada; no publicar, instalar SDK ni importar creatives.

## Descubrimiento Y Adquisicion

Reutilizar exclusivamente Descubrimiento Desde La Home y Category ID del
[contrato CLPBANNER](./clp-banner.md): todas las listas de enlaces del navbar,
menos Brands/Brand Boutique, con menu padre y niveles, URL y Category ID
deduplicados. Visitar secuencialmente solo esas CLP HTTPS del mismo host y
store/idioma autorizado, validar redirecciones antes de seguir y no seguir
enlaces de columnas, banners, productos, filtros o paginacion. Resolver tiendas
en DB solo lectura y conservar la seleccion exacta del checker.

El Category ID procede de la clase `category[ID]` del ultimo breadcrumb, nunca
del Store ID ni de un nombre inferido. Fuente incompleta, redireccion insegura,
sin identidad o fallo de acceso: BLOCKED, no ABSENT. Check no escribe HTML.

## Checker: Seleccion Y Estados

Inspeccionar el bloque editorial al final del contenido de categoria. Confirmar
su posicion respecto al listado real y al footer mediante la estructura de la
pagina; no fijar un selector universal ni confundir una fila superior, menu,
footer global, grid de productos, SEO o bloque promocional ajeno. Las clases
`news1`, `news2`, `content-row-1`, `content-row-2` y los IDs documentados son
anclas, no prueba suficiente sin contenido/posicion. Capturar region y limites
en contexto; no modificar la pagina para fingir implementacion.

Por cada candidato registrar columnas originales en orden, variantes desktop y
mobile, src/alt, titulo, descripcion (incluido vacio), todos los enlaces y
atributos funcionales. No deduplicar columnas que compartan imagen. Dos bloques
inequivocos de distinta variante en una CLP se deciden por separado; candidatos
del mismo tipo sin seleccion inequivoca: AMBIGUOUS. Tags Topsort de CLPBANNER,
TRENDING u otra region no implican implementacion de CLPCONTENT.

| Estado | Evidencia y accion al implementar |
| --- | --- |
| ABSENT | Region inferior inspeccionada suficientemente, sin bloque; reportar "No se ha encontrado el bloque"; sin HTML ni slots vacios. |
| PRESENT_NO_TOPSORT | Un bloque identificado con exactamente 2 o 3 columnas originales; convertir preservando todo su contenido. |
| SKIPPED_ALREADY_TOPSORT | Tags Topsort reales dentro del candidato, completos o parciales; omitir solo ese bloque, sin repararlo. |
| NON_SUPPORTED | Otra cantidad de columnas, slider, video, layout o contenido no representable; reportar sin conversion ni truncar/agregar columnas. |
| AMBIGUOUS | Seleccion, variantes, destino o contenido no recuperable con certeza; reportar sin HTML. |
| BLOCKED | Acceso, identidad, permisos, redireccion o fuente insuficiente; sin HTML. |

En tags existentes comprobar estructura, IDs, slots, ranks y bindings; conservar
defectos concretos como pendientes, sin reparar automaticamente. Un nodo con
binding y texto vacio no es defecto por ese motivo. No crear un bloque ausente:
CLPCONTENT nunca crea bloques ausentes (igual que NO_BANNER en CLPBANNER).

## Implementer: Estructura Documentada

Generar un fragmento completo por bloque, con todas las columnas originales.
Conservar titulo, descripcion, imagenes, alt y destinos reales de cada columna;
nunca copiar defaults del ejemplo a una tienda. Los atributos booleanos de la
documentacion se normalizan a `context="true"` y `predefined="true"`.

Dos columnas: wrapper `news1`, banner `content-block-clp`, grupo
`pagebuilder-column-group content-row-1`, linea `pagebuilder-column-line`, dos
`pagebuilder-column`, cada una con su slot. Ranks exactos `[1, 2]`.

Tres columnas: wrapper `news2`, banner `content-block-clp-2`, grupo
`pagebuilder-column-group content-row-2`, misma linea y tres columnas/slots.
Ranks exactos `[1, 2, 3]`. La primera columna siempre rank 1, la segunda 2 y
la tercera 3; no ordenar por links/nombres ni reordenar el DOM.

Ambas variantes usan `width="600" height="400"`. Envoltorios de referencia:

```html
<div class="news1">
  <topsort-banner context="true" width="600" height="400" id="content-block-clp">
    <div class="pagebuilder-column-group content-row-1">
      <div class="pagebuilder-column-line">
        <!-- Dos columnas completas segun la plantilla de columna; ranks 1 y 2. -->
      </div>
    </div>
  </topsort-banner>
</div>
```

```html
<div class="news2">
  <topsort-banner context="true" width="600" height="400" id="content-block-clp-2">
    <div class="pagebuilder-column-group content-row-2">
      <div class="pagebuilder-column-line">
        <!-- Tres columnas completas segun la plantilla de columna; ranks 1, 2 y 3. -->
      </div>
    </div>
  </topsort-banner>
</div>
```

Plantilla de cada columna (RANK y valores en mayusculas son marcadores de esta
referencia, nunca defaults para entregar). Expandirla completa dos o tres veces
en la salida, sustituyendo todos los marcadores y los comentarios de referencia:

```html
<div class="pagebuilder-column">
  <topsort-banner-slot predefined="true" rank="RANK">
    <figure>
      <a data-ts-field="target:href" href="IMAGE_TARGET">
        <img data-ts-field="mainImage:src, altText: alt" class="pagebuilder-mobile-hidden" src="DESKTOP_IMAGE" alt="DESKTOP_ALT" />
        <img data-ts-field="mobileImage:src, altText: alt" class="pagebuilder-mobile-only" src="MOBILE_IMAGE" alt="MOBILE_ALT" />
      </a>
    </figure>
    <div class="news__item">
      <h3>
        <a data-ts-field="target:href" href="TEXT_TARGET" class="primary-link">
          <span data-ts-field="ctaText:textContent">CTA_TEXT</span><i aria-hidden="true" class="icomoon-icon-Arrow-Forward"></i>
        </a>
      </h3>
      <p data-ts-field="description:textContent" class="d-none d-md-block">DESCRIPTION</p>
    </div>
  </topsort-banner-slot>
</div>
```

Bindings exactos: `target:href` en ambos anchors; `mainImage:src, altText: alt`
y `mobileImage:src, altText: alt` en las imagenes; `ctaText:textContent` en el
span del titulo/CTA; `description:textContent` en el parrafo desktop. Son seis
variables, no los cuatro campos TRENDING. Eliminar la comilla duplicada del
ejemplo original (`href="...""`); no reproducir HTML mal formado.

Si imagen y titulo tienen destinos diferentes, AMBIGUOUS sin HTML: reportar la
discrepancia con la unica variable creativa `target`, sin igualarlos ni perder
destinos silenciosamente. Destino o variantes ausentes/no identificables: AMBIGUOUS, no
inventar `#`, copiar una imagen a la otra ni aplicar la regla de enlace opcional
de CLPBANNER. Descripcion original vacia: crear/conservar el parrafo con su
binding, vacio y sin copy inventado. No descartar srcset, sources, textos o
comportamiento significativo no representable: bloquear ese candidato.

## CSS Documentado Al Final

Cada fragmento incluye una sola vez, despues de cerrar el wrapper, el CSS de su
variante. Si ambos bloques se entregan juntos, incluir cada variante una vez.
No CSS de Trending, no Swiper, no estilos/atributos inventados.

Dos columnas:

```css
.news1 {
    margin-bottom: 20px;
    padding-bottom: 20px
}

.content-row-1 .pagebuilder-column-line {
    display: flex;
}

.content-row-1 .pagebuilder-column {
    padding: 0 20px;
    margin-bottom: 25px;
    width: 50%;
}
```

Tres columnas:

```css
.news2 {
    margin-bottom: 30px;
}

.content-row-2 .pagebuilder-column-line {
    display: flex;
}

.content-row-2 .pagebuilder-column {
    padding: 0 20px;
    margin-bottom: 25px;
    width: 33.33%;
}
```

## URLs, Agrupacion Y Entrega

Aplicar URLs Relativas del contrato CLPBANNER a todos los `href` del mismo host:
path + query + fragmento, conservando idioma y Store ID. No eliminar tracking,
inventar directivas Magento ni confundir esto con modificar `src`; imagenes
pueden quedar absolutas. Otros hosts y directivas existentes se preservan sin
visitarlos. Validar URLs y no copiar protocolos peligrosos ni handlers `on*`.

Agrupar solo dentro de CLPCONTENT y del mismo hostname e idioma, sin comparar
ni agrupar entre idiomas aunque parezcan iguales. Dentro del mismo idioma,
tras comparar Category IDs, estados, variante 2/3, orden, imagenes desktop/mobile,
alt, titulo/CTA, descripcion, todos los links resueltos y estructura funcional.
Textos traducidos diferentes requieren HTML separados. Ignorar solo metadatos
traducidos de categoria, comentarios y formato no significativo; no ignorar
diferencias reales de idioma/Store ID en destinos. No compartir con CLPBANNER.
CLPBANNER mantiene su comparacion entre idiomas; TRENDING y MENUBANNER mantienen
agrupacion por idioma.

Reutilizar Entrega Por Store E Idioma de CLPBANNER, con carpeta
`CLPCONTENT-[host]/`, un `report.md` padre y HTML por megamenu en `[idioma]/`,
sin carpeta `shared/`. IDs ordenados en `[SECCION]-store-[IDs]-[idioma].html`;
un archivo nunca incluye miembros de idiomas distintos. Todos los miembros reales
en comentario y reporte, idioma(s) en mayusculas, sin copias redundantes.

Por cada CLP convertida, `<!-- Categoria <nombre real> -->` inmediatamente sobre
`<!-- WPP Cat[ID] <Nombre general> <dd/mm/aa> Topsort -->`, seguido del fragmento
completo con su CSS. Si hay dos variantes confirmadas en la CLP, una pareja de
comentarios por variante. En un agregado los IDs repetidos son para reemplazos
en paginas distintas: copiar cada fragmento por separado, nunca el archivo
entero como un unico bloque CMS. No renombrar IDs para eludir colisiones reales.

El reporte conserva todas las stores/idiomas, megamenus excluidos y arboles con
nivel, nombre, Category ID, `[Link](URL_ABSOLUTA)`, variante, estado y motivo.
Enlazar al HTML y mostrar su codigo completo identico una sola vez por grupo.
Resumen raiz: CLP/bloques ausentes, ya implementados, no soportados, ambiguos o
bloqueados con Store ID, Category ID y motivo. Chat no escribe archivos. Pedir
MD/chat solo al ejecutar un nuevo lote, no al mantener estas personalizaciones.

## CSV Adjunto Y Limite Del Template

El archivo aportado `trending-banner-carousel.csv` tiene once columnas y cuatro
campos required TRUE: `mainImage`, `target`, `headline`, `description`. No es un
template CLPCONTENT equivalente: faltan `mobileImage`, `altText`, `ctaText`, y
`headline` no es `ctaText`. No renombrarlo como canonico CLPCONTENT, reinterpretar
`headline`, inventar required/defaults para los seis campos ni modificar/importar
el CSV. Para un template creativo CLPCONTENT pedir CSV corregido o confirmacion
explicita del esquema. El HTML se rige por los seis bindings documentados arriba;
informar la discrepancia sin inventar una validacion de template creativo.

## Revision Minima Propia

Revisar con parser ambas variantes: envoltorios, ID, context, width/height,
cantidad de columnas, un slot por columna y ranks exactos consecutivos segun
orden DOM; seis destinos de binding por slot (target en dos nodos e imagenes con
altText), texto dinamico vacio valido, defaults preservados, sin doble wrapping,
sin IDs duplicados en una misma pagina, CSS de la variante exactamente una vez,
URLs relativas y marcadores resueltos. Comprobar ABSENT sin HTML, tags existentes
como NO_OP incluso parciales, y no confundir tags de otro bloque. Verificar
agrupacion solo dentro del mismo idioma, separacion de idiomas incluso cuando
coincida el contenido, y diferencias de texto/link/variante que impiden agrupar,
miembros exactos, comentarios Categoria/WPP, enlaces y codigo del reporte igual
al HTML. Revision estructural local, no prueba CMS, subasta ni SDK.