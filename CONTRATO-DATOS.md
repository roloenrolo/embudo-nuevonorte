# Contrato de datos del dashboard

Este documento describe las fuentes que consume el dashboard público del embudo. El objetivo es que cualquier generador de datos pueda producir archivos compatibles sin conocer la implementación interna.

## Convenciones

- El CSV publicado usa formato largo. La primera fila debe traer estos encabezados: `tipo`, `orden`, `clave`, `etiqueta`, `n`, `extra1`, `extra2`.
- El código normaliza encabezados a minúsculas y ordena cada familia por `orden` numérico.
- Los números pueden venir con punto decimal o coma decimal chilena. Si un valor termina en `,` + 1 o 2 dígitos, se interpreta como decimal con coma.
- Las fechas visibles se formatean con locale `es-CL`; no se muestran ISO crudas cuando el dato pasa por los renderers actuales.
- Los montos USD se muestran como `US$` con decimal chileno. Los porcentajes se muestran con coma decimal cuando tienen decimales.
- Los sellos de ventana declaran `ventana temporal · fuente · actualizado`. Si una tabla mezcla ventanas en una misma fila, la ventana se declara por columna.
- Las pestañas alimentadas por JSON se revelan solo si su fuente carga y valida el mínimo que exige el código. Si falta una fuente o no valida, la pestaña o sección queda oculta.

## CSV publicado

Cada fila debe declarar su familia en `tipo`. Las familias no listadas como consumidas se ignoran.

| `tipo` | Ventana usada por la UI | Columnas leídas | Uso en dashboard |
|---|---|---|---|
| `funnel` | acumulado | `orden`, `etiqueta`, `n` | Embudo histórico. `n` es el conteo por etapa. También alimenta KPIs base si no hay `kpi.leads`. |
| `funnel15` | 15d | `orden`, `etiqueta`, `n` | Embudo de últimos 15 días. Se muestra si existe y la primera etapa tiene conteo mayor que 0. |
| `paso` | acumulado | `orden`, `clave`, `etiqueta`, `n` | Card “Dónde se caen”. Usa `etiqueta` o, si falta, `clave`; `n` es el conteo detenido por paso. |
| `anuncio` | mixta: gasto Meta 30d; agendados/conversaciones CRM histórico | `clave`, `etiqueta`, `n`, `extra1`, `extra2` | Tabla de costo por agendamiento. `clave` es `ad_id`; `etiqueta` nombre del anuncio; `n` agendados; `extra1` gasto USD; `extra2` conversaciones. |
| `dia` | diario | `orden`, `clave`, `n` | Gráfico de volumen por día. `clave` es fecha; `n` leads. |
| `adfun` | acumulado | `orden`, `clave`, `etiqueta`, `n`, `extra1` | Cards “Dónde se caen, por anuncio”. `clave` es `ad_id`; `extra1` nombre del anuncio; `etiqueta` etapa; `n` conteo. |
| `hora` | acumulado | `clave`, `n` | Gráfico por hora. `clave` debe ser hora `0` a `23`; `n` leads. |
| `presu` | acumulado | `clave`, `n` | Split de presupuesto dentro del funnel. Claves esperadas: `califica`, `no_califica`. |
| `dsem` | acumulado | `orden`, `clave`, `etiqueta`, `n` | Gráfico por día de semana. Usa `etiqueta` o `clave` como label; `n` leads. |
| `ver` | acumulado | `orden`, `clave`, `etiqueta`, `n` | Nota de versiones bajo el funnel. `etiqueta` describe la versión; `n` conteo. |
| `meta` | 30d | `clave`, `n` | KPIs Meta Ads. El código usa el objeto resultante por claves como `gasto`, `impresiones`, `clics`, `alcance`, `conversaciones`. |
| `metaad` | 30d + estado snapshot vía join | `clave`, `etiqueta`, `n`, `extra1`, `extra2` | Tabla Meta por anuncio. `etiqueta` nombre; `n` gasto USD; `extra1` pipe string `imp\|cl\|lnk\|rch`; `extra2` conversaciones. |
| `calif` | acumulado | `clave`, `n` | Resultado de calificación. Se convierte a objeto por clave. Claves usadas por el render: `calificados`, `no_califica`, `sin_declarar`, `calif_horario`, `calif_agendo`. |
| `renta` | acumulado | `orden`, `clave`, `etiqueta`, `n` | Distribución de tramo de renta. Usa `etiqueta` o `clave`; `n` conteo. |
| `anunciofecha` | snapshot | `clave`, `etiqueta`, `n`, `extra1`, `extra2` | Tabla de fecha de inicio y join de estado. `clave` es `ad_id`; `etiqueta` nombre; `n` días activo; `extra1` fecha de inicio; `extra2` estado (`ACTIVE`, `PAUSED`, etc.). |
| `formad` | 30d | `clave`, `etiqueta`, `n`, `extra1`, `extra2` | Tabla de formularios por anuncio. `n` leads de formulario; `extra1` gasto USD; `extra2` estado. |
| `formtot` | 30d | `n`, `extra1` | Totales del carril formulario. `n` leads; `extra1` gasto USD. Si falta, el dashboard suma `formad`. |
| `reun` | acumulado | `clave`, `n` | Estado de reuniones. Se convierte a objeto por clave. Claves usadas: `agendadas`, `realizadas`, `no_asistio`, `proximas`, `pendientes`. |
| `reunres` | acumulado | `orden`, `clave`, `etiqueta`, `n` | Resultado comercial. `clave` clasifica; `etiqueta` o `clave` se muestra; `n` conteo. |
| `reunorig` | acumulado | `clave`, `n` | Origen de reuniones. Claves usadas: `antonia`, `rodolfo`. |
| `hxd` | acumulado | `clave`, `n` | Heatmap día x hora. `clave` debe tener formato `D-H`, donde `D` va de `0` a `6` y `H` de `0` a `23`; `n` leads. |
| `kpi` | acumulado | `clave`, `n` | KPIs auxiliares. Claves usadas: `leads`, `spend_total`. |

### Familias no consumidas por esta versión

El código actual no lee `adfun30`, `calif30`, `ventana` ni una familia `tipo` como valor de `tipo`. Si aparecen en el CSV publicado, se ignoran sin error.

## `forecast.json`

Fuente del tab Pronóstico. El tab valida que existan `serie` con al menos 2 puntos, `pron` con al menos 1 punto, y métricas numéricas mínimas en `tot`, `agendas` y `cierres`.

Campos consumidos:

- `generado`: fecha/hora de generación.
- `horizonte`: número informativo de días del horizonte.
- `serie[]`: objetos `{ fecha, leads }` para la serie observada.
- `pron[]`: objetos `{ fecha, p10, p50, p90 }` para pronóstico diario.
- `tot`: `{ p10, p50, p90 }` para leads esperados.
- `agendas`: `{ media, p10, p90 }` para reuniones.
- `cierres`: `{ esperados, p0, p1, p2 }`; el render usa `esperados`, `p1`, `p2`.
- `palancas[]`: objetos `{ clave, nombre, segun, detalle, cierres_esperados, p1 }`.
- `tasas[]`: objetos `{ nombre, valor, origen }`, donde `origen` se compara con `calibrada`.
- `anuncios_activos_generacion`: número usado para ajuste en vivo contra anuncios activos del CSV.
- `nota`: texto mostrado bajo la tabla de tasas.

## `noticias.json` y `noticias-historico.json`

Fuente del tab Noticias.

`noticias.json`:

- `fecha`: fecha de referencia.
- `generado`: fecha/hora de generación.
- `inmobiliario[]`: noticias del área inmobiliaria.
- `ia[]`: noticias de IA y negocio.

Cada noticia usa `{ titulo, fuente, url, fecha, impacto, porque }`. `impacto` se mapea visualmente con valores como `alto`, `medio` o `bajo`.

`noticias-historico.json`:

- `actualizado`: fecha/hora de actualización.
- `items[]`: objetos `{ captura, area, titulo, fuente, url, fecha, impacto, porque }`.

El histórico agrupa por `captura` y separa `area` en `inmobiliario` e `ia`.

## `economia.json`

Fuente del tab Economía. El tab valida que existan métricas numéricas mínimas en `cap`.

Campos consumidos:

- `schema_version`: versión del schema.
- `generado`: fecha de generación.
- `actualizacion`: texto de actualización.
- `nota_supuestos`: texto del aviso principal.
- `dolar`: `{ valor, fuente, fecha }`.
- `cap`: objeto del bucket principal:
  - `bucket`
  - `gasto_usd`
  - `escribieron`
  - `calificados`
  - `agendaron`
  - `ventas_esperadas`
  - `costo_por_calificado_usd`
  - `breakeven_por_calificado_usd`
  - `roas_esperado`
  - `comision_clp`
  - `deptos_por_venta_max`
  - `tasa_calificado_venta`
  - `ingreso_esperado_usd`
  - `nota_comision`
- `formulario`: `{ gasto_usd, leads, cpl_usd, nota }`.
- `sub`: `{ estado, comision_clp, tasa_calificado_venta, nota }`.

## `acciones-matias.json`

Fuente de la sección “Acciones propuestas por Matías” dentro del tab Meta. La sección valida título, bajada, fecha y lista no vacía de acciones.

Campos consumidos:

- `schema_version`: versión del schema.
- `generado`: fecha de generación.
- `titulo`: título de la sección.
- `bajada`: descripción de la sección.
- `acciones[]`: objetos:
  - `tipo`: uno de `pausar`, `decidir`, `revisar`.
  - `anuncio`: nombre del anuncio.
  - `motivo`: explicación breve.
  - `numeros`: objeto de pares etiqueta/valor ya formateados.
  - `estado`: texto de estado.
  - `fecha`: fecha de la acción.

## `roadmap-anuncios.json`

Fuente de la sección “Roadmap de anuncios” dentro del tab Meta. La sección valida título, bajada, regla, fecha de generación, ciclos y cierre.

Campos consumidos:

- `schema_version`: versión del schema.
- `generado`: fecha de generación.
- `titulo`: título de la sección.
- `bajada`: descripción de la sección.
- `regla`: regla general del roadmap.
- `ciclos[]`: objetos:
  - `n`: número de ciclo.
  - `desde`: fecha de inicio.
  - `hasta`: fecha de término.
  - `titulo`: título del ciclo.
  - `variable`: variable principal del ciclo.
  - `detalle`: descripción.
  - `estado`: uno de `en_curso`, `proximo`, `planificado`.
  - `pregunta`: opcional; si existe se muestra.
- `cierre`: `{ fecha, texto }`.
