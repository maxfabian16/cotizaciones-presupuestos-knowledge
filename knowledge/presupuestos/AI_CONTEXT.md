# Contexto canónico — Módulo Presupuestos

> Fuente de verdad: modelo Power BI abierto `Marketing_Fidelizacion_Medicos`, inspeccionado en modo read-only el 14/09/2026. No se procesó ni modificó ningún objeto del modelo durante esta auditoría.

## 1. Alcance y snapshot

- Módulo: Presupuestos / Cotizaciones.
- Universo operativo materializado: presupuestos con `ppto_fecha_48h >= DATE(2026,7,1)`.
- Tabla central del dashboard: `tbl_Presupuesto_Resumen`.
- Grano central: una fila por `fk_id_presupuesto`.
- Filas: 82.652.
- Presupuestos distintos: 82.652.
- Duplicados de la clave: 0.
- Tabla de medidas: `06 DAX Presupuestos`, 21 medidas `Ready`.
- Moneda de los importes documentados: soles peruanos (S/).
- Zona horaria de documentación: America/Lima.

### Frescura observable del snapshot

| Fuente/campo | Máximo cargado |
|---|---:|
| `tbl_Presupuestos[ppto_fecha_48h]` | 12/09/2026 |
| `tbl_Atenciones[atencion_fecha_hora]` | 09/09/2026 23:45:45 |
| `tbl_Trinidad[trin_atencion_fecha_registro]` | 09/09/2026 |
| `tbl_Solicitud_All[solicitud_all_fecha_produccion]` | 09/09/2026 |
| `tbl_Resultados[resultado_fecha_registro]` | 04/09/2026 |

`tbl_Resultados` está cinco días detrás de Atenciones/Trinidad/Solicitud en este snapshot. Esa diferencia explica la existencia de evidencia provisional y exige cautela con las fechas recientes.

## 2. Inventario de tablas

### A. Tablas fuente / fact

| Tabla | Filas | Grano y clave | Papel en Presupuestos | No usar para |
|---|---:|---|---|---|
| `tbl_Presupuestos` | 1.126.086 | Una fila por presupuesto; `pk_id_presupuesto` | Cabecera, paciente, sede, consentimiento y temporalidad base | Medidas del dashboard; el dashboard usa Resumen |
| `tbl_Presupuestos_Detalle` | 4.180.843 | Una línea por `pk_id_presupuesto_detalle`; el par presupuesto×análisis es único en el snapshot | Ítems y precios originalmente presupuestados | Probar ejecución; no tiene estado/anulación de línea |
| `tbl_Atenciones` | 2.671.428 | Una fila por `pk_id_atencion` | Vincula uno o varios eventos de atención con un presupuesto | Equiparar atención con conversión |
| `tbl_Resultados` | 9.647.605 | Una fila por atención×análisis; par único en el snapshot | Confirmación con hito operativo y fuente exclusiva de `resultado_precio` | Confirmar por mera existencia de fila sin hito |
| `tbl_Trinidad` | 9.762.450 | Atención×análisis×médico; 9.693.673 pares no blancos, 37.475 pares duplicados, máximo histórico 4 | Señal operacional provisional, deduplicada a atención×análisis | Confirmación autónoma o precio real |
| `tbl_Solicitud_All` | 18.663.607 | Movimiento de solicitud/producción; 10.914.427 pares atención×análisis, máximo 246 movimientos por par | Respaldo de solicitud vigente después de netear `solicitud_all_estado` | Probar ejecución por sí sola |
| `tbl_Descuentos` | 1.222.433 | Registros de descuento asociados a atención | Evidencia del código 243 | Probar llamadas sin éxito o gestiones sin descuento |

### B. Tablas auxiliares existentes

| Tabla | Filas | Clave/grano | Uso |
|---|---:|---|---|
| `tbl_Paciente_Cluster_Ppto` | 522.251 | Una fila por `fk_id_paciente` en el lado uno de la relación | Fuente de `cluster_paciente`, materializado como `ppto_tipo_paciente` en Resumen |
| `tbl_Excluidos` | 94 | Una fila por `id_analisis` | Catálogo de análisis administrativos excluidos; alimenta `solicitud_all_excluido` |
| `tbl_Analisis` | 8.983 | Una fila por `pk_id_analisis` | Catálogo de nombres para análisis adicionales |

`tbl_Pacientes`, `tbl_Sedes` y `tbl_Calendario` general existen en el modelo, pero no forman parte de la ruta vigente de cálculo o filtro del dashboard de Presupuestos. El dashboard usa paciente, sede y calendario ya materializados/dedicados en Resumen.

### C. Tablas creadas para el proyecto

#### `tbl_Calendario_Presupuestos`

- Propósito: eje temporal operativo exclusivo de Presupuestos.
- Grano/clave: una fila por `fecha`.
- Filas/rango: 74; 01/07/2026 a 12/09/2026.
- Origen lógico: `CALENDAR(DATE(2026,7,1), MAX(tbl_Presupuestos[ppto_fecha_48h]))`.
- Columnas: `fecha`, `anio`, `mes_num`, `mes`, `anio_mes`, `orden_anio_mes`.
- Dependencia: `tbl_Presupuestos[ppto_fecha_48h]`.
- No usar como timestamp exacto de ejecución ni reemplazar `ppto_fecha_hora_registro + 2`.

#### `tbl_Gestion_Diaria_Paciente_Item`

- Propósito: consolidar oportunidades diarias redundantes.
- Grano/clave lógica: paciente × `ppto_fecha_48h` × análisis.
- Filas: 151.761.
- Duplicados del grano: 0.
- Origen: `tbl_Presupuestos_Detalle` + atributos relacionados de `tbl_Presupuestos`.
- Excluye paciente blanco, análisis blanco, fecha blanca y fechas anteriores al 01/07/2026.
- Ganador: presupuesto con `ppto_fecha_hora_registro` más reciente; desempates por `pk_id_presupuesto` y luego `pk_id_presupuesto_detalle`, ambos descendentes.
- Controles: `cant_presupuestos_origen`, `cant_ocurrencias_item`.
- No filtra por validez, consentimiento, atención, contacto o call center; esos flags se conservan como atributos del ganador.
- No usar para ejecución ni conversión.

#### `tbl_Presupuesto_Detalle_Ejecucion`

- Propósito: enriquecer cada línea presupuestada original con evidencia de ejecución.
- Grano/clave: una fila por `pk_id_presupuesto_detalle`.
- Filas: 322.233; claves distintas: 322.233; duplicados: 0.
- Pares presupuesto×análisis: 322.233; no hay líneas repetidas por par en el universo actual.
- Estados de línea: 50.577 `CONFIRMADO`, 2.878 `PROVISIONAL`, 24 `EXCEPCION_SIN_HITO`, 268.754 `NO_EJECUTADO`.
- Dependencias: Presupuestos, Detalle, Atenciones, Resultados, Trinidad y Solicitud_All.
- Consolida todas las atenciones válidas de un presupuesto antes de enriquecer el detalle.
- No incluye análisis adicionales ni valores económicos finales.
- Está desconectada físicamente; se consume dentro de tablas calculadas.

#### `tbl_Presupuesto_Adicional_Ejecucion`

- Propósito: conservar análisis ejecutados que no existían en el detalle original.
- Grano físico: una fila por atención válida × análisis adicional.
- Clave lógica: `pk_id_atencion + fk_id_analisis`.
- Filas: 2.732; duplicados atención×análisis: 0.
- Pares presupuesto×análisis: 2.709; 23 filas adicionales repiten un par presupuesto×análisis porque ocurrió en más de una atención.
- Estados conservados: 2.610 `CONFIRMADO` y 122 `PROVISIONAL`. Las excepciones vencidas se excluyen y no existe `NO_EJECUTADO` en esta tabla.
- Dependencias: Análisis, Atenciones, Presupuestos, Detalle, Resultados, Trinidad y Solicitud_All.
- No debe unirse por detalle ni sumarse directamente como si su grano fuera presupuesto×análisis. Resumen deduplica los pares antes de valorar producción adicional.

#### `tbl_Presupuesto_Resumen`

- Propósito: tabla central y autosuficiente del dashboard táctico.
- Grano/clave: una fila por `fk_id_presupuesto`.
- Filas: 82.652; claves distintas: 82.652; duplicados: 0.
- Universo: `ppto_fecha_48h >= 01/07/2026`.
- Origen lógico directo: Presupuestos, Detalle, Atenciones, Resultados, Descuentos, Cluster de paciente, Detalle de ejecución y Adicionales.
- Contiene sede y tipo de paciente físicamente materializados.
- Contiene flags de funnel, estados de conversión, conteos de ítems y valores económicos.
- Es la única tabla que deben escanear las medidas tácticas actuales.
- No incluye producción adicional dentro de `ppto_valor_convertido_real` ni de sus tramos temporales.

#### Tablas intermedias de legado

| Tabla | Filas | Propósito histórico | Estado actual |
|---|---:|---|---|
| `tbl_Presupuesto_Conversion` | 82.652 | Capa 4A: estado final por presupuesto | `Ready`, desconectada, reemplazada para el dashboard |
| `tbl_Presupuesto_Valor` | 82.652 | Capa 4B: valores por presupuesto | `Ready`, desconectada, reemplazada para el dashboard |
| `tbl_Presupuesto_Momento_Conversion` | 82.652 | Capa 4C: corte exacto de 48 horas | `Ready`, desconectada, reemplazada para el dashboard |
| `tbl_Presupuesto_Funnel` | 82.652 | Capa 5A: funnel operativo | `Ready`, desconectada, reemplazada para el dashboard |
| `tbl_Presupuesto_Gestion_CallCenter` | 82.652 | Capa 5B: marcador 243 y clasificación | `Ready`, desconectada, reemplazada para el dashboard |

Estas cinco tablas siguen presentes, pero la expresión vigente de `tbl_Presupuesto_Resumen` no las referencia. No deben ser la base de nuevas medidas tácticas.

### D. Tabla DAX de medidas

#### `06 DAX Presupuestos`

- Tabla exclusiva de medidas.
- 21 medidas `Ready`, sin errores.
- Una fila técnica generada por `ROW("__dummy", 1)`; `__dummy` está oculta y fuera de MDX.
- Carpetas: `01 Base`, `02 Funnel`, `03 Conversion 48h`, `04 Post 48h`.
- `05 Comparativos` no tiene medidas. Una carpeta vacía no se materializa como objeto independiente.
- No tiene relaciones; las medidas operan sobre Resumen.

## 3. Relaciones relevantes

En las relaciones unidireccionales listadas como many→one, el filtro fluye desde el lado one hacia el lado many.

| Relación | Lado many | Lado one | Activa | Filtro | Propósito / advertencia |
|---|---|---|---|---|---|
| `AutoDetected_692d47a1-4ef3-4baf-8e0a-5939cad8632e` | `tbl_Atenciones[fk_id_presupuesto]` | `tbl_Presupuestos[pk_id_presupuesto]` | Sí | Presupuesto → Atenciones | Base de atribución |
| `AutoDetected_b5f26ac0-295c-4853-8b9c-349d43ea6a1a` | `tbl_Presupuestos_Detalle[fk_id_presupuesto]` | `tbl_Presupuestos[pk_id_presupuesto]` | Sí | Presupuesto → Detalle | Cabecera a líneas |
| `a65d8aea-5de2-7491-884d-e8cf9cc6cd9f` | `tbl_Presupuestos[fk_id_paciente]` | `tbl_Paciente_Cluster_Ppto[fk_id_paciente]` | Sí | Cluster → Presupuestos | Permite materializar tipo de paciente |
| `7feadbf8-5bc7-4bb9-7d7d-f9a7ab8a82cc` | `tbl_Solicitud_All[fk_id_atencion]` | `tbl_Atenciones[pk_id_atencion]` | Sí | Atenciones → Solicitud | Relación de soporte; la ejecución DAX usa filtro explícito |
| `363d6217-56d6-6eb3-222c-638bd9849880` | `tbl_Solicitud_All[fk_id_analisis]` | `tbl_Excluidos[id_analisis]` | Sí | Excluidos → Solicitud | Calcula exclusión administrativa |
| `720f4c48-6230-bfb9-e1e6-0ba10955df7a` | `tbl_Trinidad[fk_id_atencion]` | `tbl_Atenciones[pk_id_atencion]` | Sí | Bidireccional | Legado; riesgo de propagación amplia |
| `4c75503e-27a0-0a35-e375-adb840dbd94f` | `tbl_Trinidad[trin_codigo_atencion_analisis]` | `tbl_Resultados[cod_atencion_analisis]` | Sí | Bidireccional | Legado; no sustituye conciliación explícita |
| `c3f64263-175d-c1a7-f0c8-137b6959434a` | `tbl_Resultados[fk_id_analisis]` | `tbl_Analisis[pk_id_analisis]` | Sí | Bidireccional | Puede propagar filtros residuales; 3B aísla el catálogo con `REMOVEFILTERS` |
| `9fae6a49-f133-8f97-b63c-288a0e2abc04` | `tbl_Trinidad[fk_id_atencion]` | `tbl_Descuentos[fk_id_atencion]` | Sí | Bidireccional | Relación legado; Resumen detecta 243 mediante conjuntos explícitos |
| `PptoFecha48h_CalendarioPresupuestos` | `tbl_Presupuestos[ppto_fecha_48h]` | `tbl_Calendario_Presupuestos[fecha]` | Sí | Calendario → Presupuestos | Eje operativo de cabecera |
| `GestionFecha48h_CalendarioPresupuestos` | `tbl_Gestion_Diaria_Paciente_Item[ppto_fecha_48h]` | `tbl_Calendario_Presupuestos[fecha]` | Sí | Calendario → Gestión diaria | Eje operativo de oportunidades |
| `ResumenFecha48h_CalendarioPresupuestos` | `tbl_Presupuesto_Resumen[ppto_fecha_48h]` | `tbl_Calendario_Presupuestos[fecha]` | Sí | Calendario → Resumen | Ruta canónica Año/Mes/Día → medidas |

No existe relación `tbl_Presupuestos → tbl_Presupuesto_Resumen`. Sede y tipo de paciente filtran directamente columnas físicas de Resumen. Las tablas 3A, 3B y las cinco intermedias de legado permanecen desconectadas.

## 4. Reglas de negocio canónicas

### 4.1 Temporalidad del presupuesto

- `ppto_fecha_hora_registro` combina fecha y hora por componentes; si cualquiera es blanca devuelve `BLANK()`.
- `ppto_fecha_48h` es la fecha, sin hora, del día en que se cumplen 48 horas.
- Corte exacto: `ppto_fecha_hora_registro + 2` días.
- `ppto_fecha_48h` sirve como bucket de gestión y clave del calendario; no debe usarse como sustituto del timestamp exacto.

### 4.2 Presupuesto válido

`ppto_valido = TRUE` cuando:

```text
TRIM(COALESCE(ppto_paciente_nombre_completo, "")) <> ""
AND
TRIM(COALESCE(ppto_paciente_telefono, "")) <> ""
```

No incluye consentimiento, contacto, call center, cumplimiento de 48 horas ni atención.

### 4.3 Atención válida

Una atención es válida para un presupuesto cuando:

1. `fk_id_presupuesto` está informado y resuelve por la relación existente;
2. `atencion_fecha_hora` no es blanca;
3. `ppto_fecha_hora_registro` no es blanca;
4. `atencion_fecha_hora > ppto_fecha_hora_registro`.

La comparación es estricta. Una atención anterior o en el mismo instante es inválida. `ppto_atencion_valida` solo indica que existe al menos una atención válida; no significa conversión.

### 4.4 Evidencia de ejecución

Puerta obligatoria: `tbl_Atenciones[atencion_presupuesto_valida] = TRUE()`.

- **CONFIRMADO**: existe atención×análisis en `tbl_Resultados`, con claves informadas y al menos un hito no blanco entre:
  - `resultado_fecha_tm`;
  - `resultado_fecha_envio_tubo`;
  - `resultado_fecha_recepcion_tubo`;
  - `resultado_fecha_emision`;
  - `resultado_fecha_liberacion_net`;
  - `resultado_fecha_liberacion_net_uro_copro`.
- **PROVISIONAL**: no existe confirmación; el par aparece en Trinidad efectiva, deduplicada a atención×análisis; el mismo par tiene Solicitud_All con suma de `solicitud_all_estado > 0`, no está excluido y se encuentra dentro del SLA de cinco días.
- **EXCEPCION_SIN_HITO**: existe señal Trinidad + Solicitud admisible sin Resultado y la antigüedad supera cinco días. No cuenta como ejecución.
- **NO_EJECUTADO**: no existe evidencia confirmada ni provisional vigente.

El SLA usa como referencia `INT(MAX(tbl_Resultados[resultado_fecha_registro]))`, no el reloj actual. Esto evita crear excepciones falsas si Resultados deja de actualizarse.

`CONFIRMADO` prevalece sobre `PROVISIONAL`. Solicitud_All sola nunca declara ejecución. La fecha de ejecución es `tbl_Atenciones[atencion_fecha_hora]`. Se elige la primera evidencia cronológica del presupuesto×análisis; el desempate es `pk_id_atencion`.

### 4.5 Conversión y estados

Conversión no equivale a tener atención. Un presupuesto convierte cuando al menos un análisis originalmente presupuestado tiene `item_ejecutado = TRUE` en 3A.

Sea:

- `pres`: cantidad de líneas originalmente presupuestadas;
- `ejec`: cantidad de líneas ejecutadas, confirmadas o provisionales vigentes.

Estado final:

| Estado | Condición |
|---|---|
| `NO_EVALUABLE` | `pres = 0` |
| `NO_CONVERTIDO` | `pres > 0` y `ejec = 0` |
| `PARCIAL` | `0 < ejec < pres` |
| `TOTAL` | `ejec >= pres` |

Distribución actual: 6.777 NO_EVALUABLE; 66.225 NO_CONVERTIDO; 1.350 PARCIAL; 8.300 TOTAL.

El estado a 48 horas aplica la misma regla usando únicamente líneas cuya primera ejecución ocurrió `<= ppto_fecha_hora_registro + 2`. Distribución: 6.777 NO_EVALUABLE; 67.965 NO_CONVERTIDO; 1.027 PARCIAL; 6.883 TOTAL.

### 4.6 Precio real

- Fuente exclusiva: `tbl_Resultados[resultado_precio]`.
- Solo evidencia `CONFIRMADO`.
- Provisional no tiene importe real.
- No se usa precio presupuestado como fallback.
- No se usa Trinidad, Solicitud_All ni producción adicional como sustituto.
- Si un presupuesto×análisis aparece confirmado en varias atenciones, la lógica de Resumen toma el precio de la primera atención válida confirmada, con desempate por ID de atención.
- `resultado_precio = 0` es un cero real; no se imputa valor.

### 4.7 Código 243

Código: `tbl_Descuentos[fk_id_descuento] = "243"`.

Nombre literal exacto en el modelo:

```text
PRESUPUESTOS NO CONVERTIDOS CALL CENTER 48H  10%
```

Hay dos espacios entre `48H` y `10%`.

- Presencia: evidencia positiva de una gestión que terminó en atención válida con el descuento aplicado.
- Ausencia: no demuestra ausencia de llamada; no cubre llamadas fallidas, gestiones sin descuento ni periodos sin uso del código.
- No aporta denominador de llamadas o gestiones.
- No permite denominar `porc_post_48h_243` como efectividad del Call Center; es participación con evidencia 243.
- `POST_48H_SIN_243` es descriptivo, no causal ni necesariamente orgánico.
- Estado actual en Resumen: 40 presupuestos con 243, 10 usos fuera de la regla de 48 horas y 25 recuperaciones gestionables del funnel con 243.

### 4.8 Oportunidad diaria

Unidad: paciente + fecha de gestión + análisis.

Si el mismo análisis aparece varias veces el mismo día para el mismo paciente, se conserva una sola fila y gana el presupuesto con registro más reciente. Desempates: presupuesto y detalle descendentes. El precio/tarifa y los flags corresponden a la ocurrencia ganadora. Esta tabla consolida oportunidad, no ejecución.

## 5. Funnel canónico

| Etapa | Conteo | Denominador de tasa | Tasa |
|---|---:|---:|---:|
| Evaluables | 75.875 | — | — |
| Válidos | 45.200 | 75.875 evaluables | 59,5717 % |
| Con consentimiento | 8.449 | 45.200 válidos | 18,6925 % |
| Conversión ≤48h dentro del funnel | 663 | 8.449 con consentimiento | 7,8471 % |
| Pendientes a 48h | 7.786 | — | — |
| Recuperados post 48h | 233 | 7.786 pendientes | 2,9926 % |
| Con evidencia 243 gestionable | 25 | 233 recuperados | 10,7296 % |
| Sin evidencia 243 | 208 | 233 recuperados | 89,2704 % |

Reconciliaciones:

- `663 + 7.786 = 8.449`.
- `25 + 208 = 233`.

`funnel_pendiente_48h` exige consentimiento y estado `NO_CONVERTIDO` a las 48 horas. Por ello, un presupuesto parcialmente convertido a las 48 horas no entra al denominador de recuperación post 48h.

## 6. Valores económicos

| Campo/concepto | Definición | Incluye | Excluye | Total actual |
|---|---|---|---|---:|
| `ppto_valor_presupuestado_bruto` | Suma de `ppto_analisis_precio` de todas las líneas originales | Todo lo presupuestado | Adicionales | S/ 39.802.181,03 |
| `ppto_valor_presupuestado_ejecutado` | Precio presupuestado de líneas con `item_ejecutado=TRUE` | Confirmados y provisionales vigentes, valorados a precio presupuestado | Excepciones y no ejecutados | No se usa como precio real |
| `ppto_valor_no_convertido` | Precio presupuestado de líneas con `item_ejecutado=FALSE` | No ejecutados y excepciones | Ejecutados | No reconcilia con precio real |
| `ppto_valor_convertido_real` | Suma de `resultado_precio` para líneas originales confirmadas | Ítems presupuestados confirmados | Provisionales y adicionales | S/ 5.056.912,07 |
| `ppto_valor_convertido_48h_real` | Tramo de convertido real cuya primera ejecución fue ≤ corte exacto | Confirmados originales ≤48h | Post48h, provisionales y adicionales | S/ 4.091.405,96 |
| `ppto_valor_convertido_post_48h_real` | Tramo de convertido real cuya primera ejecución fue > corte exacto | Confirmados originales post48h | ≤48h, provisionales y adicionales | S/ 965.506,11 |
| Producción adicional / `ppto_valor_adicional_real` | Pares adicionales confirmados valorados con `resultado_precio` | Análisis no presupuestados confirmados, deduplicados a presupuesto×análisis | Ítems presupuestados y provisionales | S/ 260.654,06 |

Reconciliación estructural:

```text
S/ 4.091.405,96 + S/ 965.506,11 = S/ 5.056.912,07
```

Los montos del dashboard son subconjuntos del funnel:

- `monto_conv_48h_real = S/ 288.545`: solo presupuestos con `funnel_conversion_organica_48h=TRUE`.
- `monto_conv_post_48h_real = S/ 106.927`: solo presupuestos con `funnel_ejecucion_posterior=TRUE`.

No son iguales a los totales estructurales porque estos incluyen todos los presupuestos del universo, sin exigir evaluabilidad, validez, consentimiento ni pertenencia a las etapas del funnel.

## 7. Tipo de paciente

Campo del dashboard: `tbl_Presupuesto_Resumen[ppto_tipo_paciente]`.

Origen lógico: `tbl_Paciente_Cluster_Ppto[cluster_paciente]`, resuelto durante la construcción de Resumen mediante la relación física de paciente. Un valor blanco o la ausencia de correspondencia se convierte en `"Sin historial"`.

| Categoría | Presupuestos |
|---|---:|
| Estratégico | 4.642 |
| Recurrente | 2.304 |
| Potencial | 1.941 |
| Bajo Impacto | 5.800 |
| Sin historial | 67.965 |
| **Total** | **82.652** |

No deben crearse medidas por categoría; el desglose se obtiene con esta columna en el contexto del visual.

## 8. Diccionario funcional de `tbl_Presupuesto_Resumen`

| Columna | Tipo | Definición/fuente | Uso y valores |
|---|---|---|---|
| `fk_id_presupuesto` | String | `tbl_Presupuestos[pk_id_presupuesto]` | Clave única del Resumen |
| `fk_id_paciente` | String | Cabecera del presupuesto | Distinct count con exclusión de blancos |
| `ppto_sede` | String | `tbl_Presupuestos[ppto_sede]` | Slicer/desglose directo; existen 6 filas con literal `"NULL"` |
| `ppto_tipo_paciente` | String | Cluster de paciente, con blanco/no match → Sin historial | Cinco categorías canónicas |
| `ppto_fecha_hora_registro` | DateTime | Fecha + hora de registro por componentes | Inicio temporal exacto |
| `ppto_fecha_48h` | DateTime de fecha | Día calendario de cumplimiento de 48h | Relación con Calendario |
| `ppto_fecha_hora_limite_48h` | DateTime | `ppto_fecha_hora_registro + 2` | Corte temporal exacto |
| `ppto_evaluable` | Boolean | Cantidad de líneas originales > 0 | Primera etapa del funnel |
| `ppto_valido` | Boolean | Nombre y teléfono no vacíos tras TRIM/COALESCE | Atributo atómico |
| `ppto_atencion_valida` | Boolean | Existe al menos una atención válida | No equivale a conversión |
| `ppto_consentimiento` | Boolean | Campo de cabecera | Requisito del funnel después de validez |
| `ppto_cant_items_presupuestados` | Int64 | Conteo de líneas originales | Denominador de estados |
| `ppto_cant_items_ejecutados` | Int64 | Líneas 3A con `item_ejecutado=TRUE` | Confirmado + provisional vigente |
| `ppto_estado_conversion` | String | Comparación ejecutados vs presupuestados | NO_EVALUABLE, NO_CONVERTIDO, PARCIAL, TOTAL |
| `ppto_cant_items_ejecutados_48h` | Int64 | Ejecutados con primera ejecución ≤ límite exacto | Estado al corte |
| `ppto_cant_items_pendientes_48h` | Int64 | `MAX(0, presupuestados - ejecutados_48h)` | Volumen pendiente de ítems |
| `ppto_estado_conversion_48h` | String | Estado usando solo ejecutados ≤48h | Mismos cuatro estados |
| `ppto_cant_items_ejecutados_post_48h` | Int64 | Ejecutados con primera ejecución > límite | Seguimiento tardío |
| `ppto_tuvo_ejecucion_post_48h` | Boolean | Conteo post48h > 0 | Evidencia temporal |
| `ppto_valor_presupuestado_bruto` | Double | Suma de precios presupuestados | Monto base |
| `ppto_valor_presupuestado_ejecutado` | Double | Precio presupuestado de líneas ejecutadas | No es precio real |
| `ppto_valor_no_convertido` | Double | Precio presupuestado de líneas no ejecutadas | No es resta contra precio real |
| `ppto_valor_convertido_real` | Double | Resultado precio, solo confirmados originales | Monto real estructural |
| `ppto_valor_convertido_48h_real` | Double | Resultado precio confirmado del tramo ≤48h | Monto estructural temporal |
| `ppto_valor_convertido_post_48h_real` | Double | Resultado precio confirmado del tramo post48h | Monto estructural temporal |
| `ppto_valor_adicional_real` | Double | Resultado precio de adicionales confirmados | Producción adicional separada |
| `funnel_valido` | Boolean | Evaluable y `ppto_valido` | Etapa válida |
| `funnel_consentimiento` | Boolean | Evaluable, válido y consentimiento | Universo de tasas de conversión |
| `funnel_conversion_organica_48h` | Boolean | Funnel consentimiento y estado48h PARCIAL/TOTAL | Numerador ≤48h |
| `funnel_pendiente_48h` | Boolean | Funnel consentimiento y estado48h NO_CONVERTIDO | Denominador post48h |
| `funnel_ejecucion_posterior` | Boolean | Pendiente a 48h y conteo post48h > 0 | Recuperados post48h |
| `gestion_call_center_243` | Boolean | Alguna atención válida del presupuesto tiene descuento 243 | Evidencia positiva, no denominador |
| `uso_243_fuera_regla_48h` | Boolean | Tiene 243 pero estado48h distinto de NO_CONVERTIDO | Control de ruido |
| `conversion_post_48h_con_243_gestionable` | Boolean | Consentimiento, NO_CONVERTIDO a 48h, post ejecución y 243 | Numerador de participación 243 |
| `ppto_tipo_conversion` | String | Clasificación con precedencia | NO_EVALUABLE, NO_CONVERTIDO, ORGANICA_48H, POST_48H_CON_243, POST_48H_SIN_243; SIN_CLASIFICAR es fallback y no aparece actualmente |

Distribución actual de `ppto_tipo_conversion`: 6.777 NO_EVALUABLE; 66.225 NO_CONVERTIDO; 7.910 ORGANICA_48H; 30 POST_48H_CON_243; 1.710 POST_48H_SIN_243; 0 SIN_CLASIFICAR. Esta clasificación cubre todo el universo y no equivale al funnel táctico, que además exige consentimiento.

## 9. Lineage y flujo de datos

1. **Presupuesto** — `tbl_Presupuestos`: cabecera, paciente, sede, consentimiento y registro.
2. **Detalle presupuestado** — `tbl_Presupuestos_Detalle`: análisis y precio esperado.
3. **Atención válida** — `tbl_Atenciones`: FK de presupuesto y comparación estricta de timestamps.
4. **Solicitud/señal operacional** — Trinidad efectiva deduplicada + Solicitud_All neta positiva y no excluida.
5. **Resultado** — `tbl_Resultados`: hito operativo confirmado y `resultado_precio`.
6. **Evidencia por detalle** — `tbl_Presupuesto_Detalle_Ejecucion`: una fila por línea original con estado y primera atención.
7. **Adicionales** — `tbl_Presupuesto_Adicional_Ejecucion`: atención×análisis ejecutado no presupuestado.
8. **Conversión y momento** — Resumen consolida líneas a presupuesto, compara conteos y separa el corte exacto.
9. **Funnel y 243** — Resumen materializa validez, consentimiento, recuperación y marcador.
10. **Resumen** — `tbl_Presupuesto_Resumen`: tabla autosuficiente de una fila por presupuesto.
11. **Medidas** — `06 DAX Presupuestos`: agregaciones simples sobre Resumen.
12. **Dashboard** — calendario filtra Resumen; sede y tipo filtran columnas directas.

## 10. Dashboard táctico

### Panel 01 — Funnel

- KPI: evaluables, válidos, consentimiento.
- Evolución: mismas medidas con Año/Mes/Día de `tbl_Calendario_Presupuestos`.
- Perfil: `ppto_tipo_paciente` como categoría.
- Benchmarks: 75.875; 45.200; 8.449.

### Panel 02 — ≤48h

- Presupuestos: `cant_ppto_conv_48h` = 663.
- Pacientes: `cant_pax_conv_48h_ppto` = 561.
- Tasa: `tasa_ppto_conv_48h` = 7,8471 %.
- Monto real del funnel: `monto_conv_48h_real` = S/ 288.545.
- Evolución diaria y perfil reutilizan las mismas medidas bajo el contexto de calendario/tipo.

### Panel 03 — Post48h

- Pendientes: `cant_ppto_pend_48h` = 7.786.
- Recuperados: `cant_ppto_conv_post_48h` = 233.
- Pacientes: `cant_pax_conv_post_48h_ppto` = 191.
- Tasa de recuperación: `tasa_ppto_conv_post_48h` = 2,9926 %.
- Monto recuperado: `monto_conv_post_48h_real` = S/ 106.927.
- Con 243: 25; sin 243: 208.
- Participación 243: 10,7296 %; no llamar efectividad.
- Evolución y perfil reutilizan las mismas medidas.

No crear medidas independientes por mes, día, sede o tipo de paciente mientras el contexto de filtros normal sea suficiente.

## 11. DAX canónico de `06 DAX Presupuestos`

Las expresiones siguientes son copias completas del modelo. Metadatos y dependencias detalladas: [DAX_CATALOG.md](DAX_CATALOG.md).

### `cant_ppto` — `01 Base` — `#,0`

~~~DAX
COUNTROWS('tbl_Presupuesto_Resumen')
~~~

### `cant_pax_ppto` — `01 Base` — `#,0`

~~~DAX
DISTINCTCOUNTNOBLANK('tbl_Presupuesto_Resumen'[fk_id_paciente])
~~~

### `monto_ppto_bruto` — `01 Base` — `S/ #,0`

~~~DAX
SUM('tbl_Presupuesto_Resumen'[ppto_valor_presupuestado_bruto])
~~~

### `monto_conv_real` — `01 Base` — `S/ #,0`

~~~DAX
SUM('tbl_Presupuesto_Resumen'[ppto_valor_convertido_real])
~~~

### `cant_ppto_evaluable` — `02 Funnel` — `#,0`

~~~DAX
CALCULATE(
    [cant_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[ppto_evaluable] = TRUE())
)
~~~

### `cant_ppto_valido` — `02 Funnel` — `#,0`

~~~DAX
CALCULATE(
    [cant_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_valido] = TRUE())
)
~~~

### `cant_ppto_consentimiento` — `02 Funnel` — `#,0`

~~~DAX
CALCULATE(
    [cant_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_consentimiento] = TRUE())
)
~~~

### `tasa_ppto_valido` — `02 Funnel` — `0.0%`

~~~DAX
DIVIDE(
    [cant_ppto_valido],
    [cant_ppto_evaluable]
)
~~~

### `tasa_ppto_consentimiento` — `02 Funnel` — `0.0%`

~~~DAX
DIVIDE(
    [cant_ppto_consentimiento],
    [cant_ppto_valido]
)
~~~

### `cant_ppto_conv_48h` — `03 Conversion 48h` — `#,0`

~~~DAX
CALCULATE(
    [cant_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_conversion_organica_48h] = TRUE())
)
~~~

### `cant_pax_conv_48h_ppto` — `03 Conversion 48h` — `#,0`

~~~DAX
CALCULATE(
    [cant_pax_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_conversion_organica_48h] = TRUE())
)
~~~

### `tasa_ppto_conv_48h` — `03 Conversion 48h` — `0.0%`

~~~DAX
DIVIDE(
    [cant_ppto_conv_48h],
    [cant_ppto_consentimiento]
)
~~~

### `monto_conv_48h_real` — `03 Conversion 48h` — `S/ #,0`

~~~DAX
CALCULATE(
    SUM('tbl_Presupuesto_Resumen'[ppto_valor_convertido_48h_real]),
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_conversion_organica_48h] = TRUE())
)
~~~

### `cant_ppto_pend_48h` — `04 Post 48h` — `#,0`

~~~DAX
CALCULATE(
    [cant_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_pendiente_48h] = TRUE())
)
~~~

### `cant_ppto_conv_post_48h` — `04 Post 48h` — `#,0`

~~~DAX
CALCULATE(
    [cant_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_ejecucion_posterior] = TRUE())
)
~~~

### `cant_pax_conv_post_48h_ppto` — `04 Post 48h` — `#,0`

~~~DAX
CALCULATE(
    [cant_pax_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_ejecucion_posterior] = TRUE())
)
~~~

### `tasa_ppto_conv_post_48h` — `04 Post 48h` — `0.0%`

~~~DAX
DIVIDE(
    [cant_ppto_conv_post_48h],
    [cant_ppto_pend_48h]
)
~~~

### `monto_conv_post_48h_real` — `04 Post 48h` — `S/ #,0`

~~~DAX
CALCULATE(
    SUM('tbl_Presupuesto_Resumen'[ppto_valor_convertido_post_48h_real]),
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_ejecucion_posterior] = TRUE())
)
~~~

### `cant_ppto_post_48h_243` — `04 Post 48h` — `#,0`

~~~DAX
CALCULATE(
    [cant_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[conversion_post_48h_con_243_gestionable] = TRUE())
)
~~~

### `cant_ppto_post_48h_sin_243` — `04 Post 48h` — `#,0`

~~~DAX
[cant_ppto_conv_post_48h]
    - [cant_ppto_post_48h_243]
~~~

### `porc_post_48h_243` — `04 Post 48h` — `0.0%`

~~~DAX
DIVIDE(
    [cant_ppto_post_48h_243],
    [cant_ppto_conv_post_48h]
)
~~~


## 12. Riesgos y advertencias

### NO HACER

- No agregar columnas pesadas de conversión a `tbl_Presupuestos`.
- No ejecutar Process Calculate/Full/Data/Automatic general sobre `tbl_Presupuestos` para cambios tácticos.
- No reconstruir temporalidad en medidas cuando ya está materializada en Resumen.
- No usar `TREATAS` para Año/Mes/Día: la relación física Calendario → Resumen ya existe.
- No usar producción adicional como monto convertido de ítems presupuestados.
- No usar 243 como prueba de todas las llamadas ni como tasa de efectividad.
- No llamar orgánico a `POST_48H_SIN_243`.
- No confundir `ppto_fecha_48h` con `ppto_fecha_hora_registro + 2`.
- No usar `01_Dax_Base[cant_pax]` para Presupuestos; usar `cant_pax_ppto`.
- No sumar directamente 3B a grano presupuesto×análisis sin deduplicar las múltiples atenciones.
- No usar las cinco tablas intermedias de legado para nuevas medidas del dashboard.
- No usar precio presupuestado como fallback de precio real.
- No contar `EXCEPCION_SIN_HITO` como ejecución.

### Riesgos observados

1. **Rezago de Resultados**: cinco días en el snapshot. Los montos reales recientes están sujetos a confirmación posterior.
2. **Reclasificación por SLA**: un provisional puede convertirse en confirmado o excepción en un cálculo futuro.
3. **Relaciones bidireccionales de legado**: Trinidad–Resultados–Análisis–Atenciones–Descuentos pueden propagar filtros amplios. Las medidas tácticas evitan esa ruta usando Resumen.
4. **Grano de adicionales**: 23 repeticiones de presupuesto×análisis por múltiples atenciones; son trazabilidad válida, no duplicado de atención×análisis.
5. **Sede literal `"NULL"`**: 6 presupuestos. No es `BLANK()` y aparecerá como categoría en slicers.
6. **Código 243 incompleto**: evidencia positiva de descuento, no registro exhaustivo de gestión.
7. **Calendario calculado**: termina en la máxima fecha de gestión disponible; no contiene fechas futuras hasta recalcularse.
8. **Tablas legado**: permanecen `Ready` y pueden inducir a crear medidas sobre una ruta sustituida.
9. **Importes Double**: presentar redondeados como moneda; la reconciliación económica es exacta a centavos.
10. **Snapshot importado**: los conteos pueden cambiar cuando se actualicen fuentes y se recalculen las capas.

## DISCREPANCIAS DETECTADAS

No se detectaron discrepancias entre los objetos, las 21 medidas y los benchmarks solicitados frente al modelo actual.

Precisiones del modelo que deben preservarse:

- El nombre literal del descuento 243 contiene dos espacios antes de `10%`.
- 3B tiene 2.732 filas a grano atención×análisis, pero 2.709 pares presupuesto×análisis.
- Las cinco tablas intermedias históricas siguen presentes, aunque Resumen ya no depende de ellas.
- `05 Comparativos` está vacío; Power BI no conserva una carpeta vacía como objeto independiente.
