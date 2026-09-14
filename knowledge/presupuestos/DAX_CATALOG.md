# Catálogo DAX — Presupuestos

> Snapshot read-only del modelo abierto, generado el 14/09/2026. Las expresiones de esta página son copias exactas de las medidas del modelo.

## Estado

- Tabla: `06 DAX Presupuestos`
- Medidas: **21**
- Todas las medidas: `Ready`, sin errores
- Columna técnica: `__dummy`, oculta
- Carpetas con medidas: `01 Base`, `02 Funnel`, `03 Conversion 48h`, `04 Post 48h`
- `05 Comparativos`: vacío; Power BI no materializa una carpeta vacía como objeto independiente

## Índice

| Medida | Carpeta | Formato | Dependencias | Benchmark global |
|---|---|---|---|---|
| `cant_ppto` | `01 Base` | `#,0` | — | 82.652 |
| `cant_pax_ppto` | `01 Base` | `#,0` | — | 25.865 |
| `monto_ppto_bruto` | `01 Base` | `S/ #,0` | — | S/ 39.802.181,03 |
| `monto_conv_real` | `01 Base` | `S/ #,0` | — | S/ 5.056.912,07 |
| `cant_ppto_evaluable` | `02 Funnel` | `#,0` | `[cant_ppto]` | 75.875 |
| `cant_ppto_valido` | `02 Funnel` | `#,0` | `[cant_ppto]` | 45.200 |
| `cant_ppto_consentimiento` | `02 Funnel` | `#,0` | `[cant_ppto]` | 8.449 |
| `tasa_ppto_valido` | `02 Funnel` | `0.0%` | `[cant_ppto_valido]`, `[cant_ppto_evaluable]` | 59,5717 % |
| `tasa_ppto_consentimiento` | `02 Funnel` | `0.0%` | `[cant_ppto_consentimiento]`, `[cant_ppto_valido]` | 18,6925 % |
| `cant_ppto_conv_48h` | `03 Conversion 48h` | `#,0` | `[cant_ppto]` | 663 |
| `cant_pax_conv_48h_ppto` | `03 Conversion 48h` | `#,0` | `[cant_pax_ppto]` | 561 |
| `tasa_ppto_conv_48h` | `03 Conversion 48h` | `0.0%` | `[cant_ppto_conv_48h]`, `[cant_ppto_consentimiento]` | 7,8471 % |
| `monto_conv_48h_real` | `03 Conversion 48h` | `S/ #,0` | — | S/ 288.545 |
| `cant_ppto_pend_48h` | `04 Post 48h` | `#,0` | `[cant_ppto]` | 7.786 |
| `cant_ppto_conv_post_48h` | `04 Post 48h` | `#,0` | `[cant_ppto]` | 233 |
| `cant_pax_conv_post_48h_ppto` | `04 Post 48h` | `#,0` | `[cant_pax_ppto]` | 191 |
| `tasa_ppto_conv_post_48h` | `04 Post 48h` | `0.0%` | `[cant_ppto_conv_post_48h]`, `[cant_ppto_pend_48h]` | 2,9926 % |
| `monto_conv_post_48h_real` | `04 Post 48h` | `S/ #,0` | — | S/ 106.927 |
| `cant_ppto_post_48h_243` | `04 Post 48h` | `#,0` | `[cant_ppto]` | 25 |
| `cant_ppto_post_48h_sin_243` | `04 Post 48h` | `#,0` | `[cant_ppto_conv_post_48h]`, `[cant_ppto_post_48h_243]` | 208 |
| `porc_post_48h_243` | `04 Post 48h` | `0.0%` | `[cant_ppto_post_48h_243]`, `[cant_ppto_conv_post_48h]` | 10,7296 % |

## Definiciones exactas

### `cant_ppto`

- Display Folder: `01 Base`
- Format string: `#,0`
- Descripción del modelo: Cantidad de presupuestos visibles en tbl_Presupuesto_Resumen bajo el contexto de filtros actual.
- Propósito funcional: Cantidad de presupuestos visibles en tbl_Presupuesto_Resumen bajo el contexto de filtros actual.
- Dependencias de medidas: ninguna
- Columnas utilizadas directamente: ninguna; utiliza una tabla o medidas dependientes
- Benchmark global actual: **82.652**

~~~DAX
COUNTROWS('tbl_Presupuesto_Resumen')
~~~

### `cant_pax_ppto`

- Display Folder: `01 Base`
- Format string: `#,0`
- Descripción del modelo: Cantidad de pacientes distintos con identificador no blanco en los presupuestos visibles.
- Propósito funcional: Cantidad de pacientes distintos con identificador no blanco en los presupuestos visibles.
- Dependencias de medidas: ninguna
- Columnas utilizadas directamente: `tbl_Presupuesto_Resumen[fk_id_paciente]`
- Benchmark global actual: **25.865**

~~~DAX
DISTINCTCOUNTNOBLANK('tbl_Presupuesto_Resumen'[fk_id_paciente])
~~~

### `monto_ppto_bruto`

- Display Folder: `01 Base`
- Format string: `S/ #,0`
- Descripción del modelo: Suma del valor bruto originalmente presupuestado en el contexto visible.
- Propósito funcional: Suma del valor bruto originalmente presupuestado en el contexto visible.
- Dependencias de medidas: ninguna
- Columnas utilizadas directamente: `tbl_Presupuesto_Resumen[ppto_valor_presupuestado_bruto]`
- Benchmark global actual: **S/ 39.802.181,03**

~~~DAX
SUM('tbl_Presupuesto_Resumen'[ppto_valor_presupuestado_bruto])
~~~

### `monto_conv_real`

- Display Folder: `01 Base`
- Format string: `S/ #,0`
- Descripción del modelo: Suma del valor real confirmado de ítems presupuestados ejecutados en el contexto visible.
- Propósito funcional: Suma del valor real confirmado de ítems presupuestados ejecutados en el contexto visible.
- Dependencias de medidas: ninguna
- Columnas utilizadas directamente: `tbl_Presupuesto_Resumen[ppto_valor_convertido_real]`
- Benchmark global actual: **S/ 5.056.912,07**

~~~DAX
SUM('tbl_Presupuesto_Resumen'[ppto_valor_convertido_real])
~~~

### `cant_ppto_evaluable`

- Display Folder: `02 Funnel`
- Format string: `#,0`
- Descripción del modelo: Cantidad de presupuestos evaluables, definidos por ppto_evaluable = TRUE.
- Propósito funcional: Cantidad de presupuestos evaluables, definidos por ppto_evaluable = TRUE.
- Dependencias de medidas: `[cant_ppto]`
- Columnas utilizadas directamente: `tbl_Presupuesto_Resumen[ppto_evaluable]`
- Benchmark global actual: **75.875**

~~~DAX
CALCULATE(
    [cant_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[ppto_evaluable] = TRUE())
)
~~~

### `cant_ppto_valido`

- Display Folder: `02 Funnel`
- Format string: `#,0`
- Descripción del modelo: Cantidad de presupuestos del funnel que cumplen la condición de validez.
- Propósito funcional: Cantidad de presupuestos del funnel que cumplen la condición de validez.
- Dependencias de medidas: `[cant_ppto]`
- Columnas utilizadas directamente: `tbl_Presupuesto_Resumen[funnel_valido]`
- Benchmark global actual: **45.200**

~~~DAX
CALCULATE(
    [cant_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_valido] = TRUE())
)
~~~

### `cant_ppto_consentimiento`

- Display Folder: `02 Funnel`
- Format string: `#,0`
- Descripción del modelo: Cantidad de presupuestos válidos del funnel que además tienen consentimiento.
- Propósito funcional: Cantidad de presupuestos válidos del funnel que además tienen consentimiento.
- Dependencias de medidas: `[cant_ppto]`
- Columnas utilizadas directamente: `tbl_Presupuesto_Resumen[funnel_consentimiento]`
- Benchmark global actual: **8.449**

~~~DAX
CALCULATE(
    [cant_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_consentimiento] = TRUE())
)
~~~

### `tasa_ppto_valido`

- Display Folder: `02 Funnel`
- Format string: `0.0%`
- Descripción del modelo: Presupuestos válidos / presupuestos evaluables.
- Propósito funcional: Presupuestos válidos / presupuestos evaluables.
- Dependencias de medidas: `[cant_ppto_valido]`, `[cant_ppto_evaluable]`
- Columnas utilizadas directamente: ninguna; utiliza una tabla o medidas dependientes
- Benchmark global actual: **59,5717 %**

~~~DAX
DIVIDE(
    [cant_ppto_valido],
    [cant_ppto_evaluable]
)
~~~

### `tasa_ppto_consentimiento`

- Display Folder: `02 Funnel`
- Format string: `0.0%`
- Descripción del modelo: Presupuestos con consentimiento / presupuestos válidos.
- Propósito funcional: Presupuestos con consentimiento / presupuestos válidos.
- Dependencias de medidas: `[cant_ppto_consentimiento]`, `[cant_ppto_valido]`
- Columnas utilizadas directamente: ninguna; utiliza una tabla o medidas dependientes
- Benchmark global actual: **18,6925 %**

~~~DAX
DIVIDE(
    [cant_ppto_consentimiento],
    [cant_ppto_valido]
)
~~~

### `cant_ppto_conv_48h`

- Display Folder: `03 Conversion 48h`
- Format string: `#,0`
- Descripción del modelo: Cantidad de presupuestos con consentimiento y conversión orgánica dentro de las primeras 48 horas.
- Propósito funcional: Cantidad de presupuestos con consentimiento y conversión orgánica dentro de las primeras 48 horas.
- Dependencias de medidas: `[cant_ppto]`
- Columnas utilizadas directamente: `tbl_Presupuesto_Resumen[funnel_conversion_organica_48h]`
- Benchmark global actual: **663**

~~~DAX
CALCULATE(
    [cant_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_conversion_organica_48h] = TRUE())
)
~~~

### `cant_pax_conv_48h_ppto`

- Display Folder: `03 Conversion 48h`
- Format string: `#,0`
- Descripción del modelo: Pacientes distintos no blancos con presupuestos convertidos dentro de las primeras 48 horas.
- Propósito funcional: Pacientes distintos no blancos con presupuestos convertidos dentro de las primeras 48 horas.
- Dependencias de medidas: `[cant_pax_ppto]`
- Columnas utilizadas directamente: `tbl_Presupuesto_Resumen[funnel_conversion_organica_48h]`
- Benchmark global actual: **561**

~~~DAX
CALCULATE(
    [cant_pax_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_conversion_organica_48h] = TRUE())
)
~~~

### `tasa_ppto_conv_48h`

- Display Folder: `03 Conversion 48h`
- Format string: `0.0%`
- Descripción del modelo: Presupuestos con conversión orgánica dentro de 48 horas / presupuestos con consentimiento.
- Propósito funcional: Presupuestos con conversión orgánica dentro de 48 horas / presupuestos con consentimiento.
- Dependencias de medidas: `[cant_ppto_conv_48h]`, `[cant_ppto_consentimiento]`
- Columnas utilizadas directamente: ninguna; utiliza una tabla o medidas dependientes
- Benchmark global actual: **7,8471 %**

~~~DAX
DIVIDE(
    [cant_ppto_conv_48h],
    [cant_ppto_consentimiento]
)
~~~

### `monto_conv_48h_real`

- Display Folder: `03 Conversion 48h`
- Format string: `S/ #,0`
- Descripción del modelo: Valor real confirmado dentro de 48 horas, limitado a presupuestos de conversión orgánica del funnel.
- Propósito funcional: Valor real confirmado dentro de 48 horas, limitado a presupuestos de conversión orgánica del funnel.
- Dependencias de medidas: ninguna
- Columnas utilizadas directamente: `tbl_Presupuesto_Resumen[ppto_valor_convertido_48h_real]`, `tbl_Presupuesto_Resumen[funnel_conversion_organica_48h]`
- Benchmark global actual: **S/ 288.545**

~~~DAX
CALCULATE(
    SUM('tbl_Presupuesto_Resumen'[ppto_valor_convertido_48h_real]),
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_conversion_organica_48h] = TRUE())
)
~~~

### `cant_ppto_pend_48h`

- Display Folder: `04 Post 48h`
- Format string: `#,0`
- Descripción del modelo: Cantidad de presupuestos con consentimiento que permanecían sin conversión al cumplir 48 horas.
- Propósito funcional: Cantidad de presupuestos con consentimiento que permanecían sin conversión al cumplir 48 horas.
- Dependencias de medidas: `[cant_ppto]`
- Columnas utilizadas directamente: `tbl_Presupuesto_Resumen[funnel_pendiente_48h]`
- Benchmark global actual: **7.786**

~~~DAX
CALCULATE(
    [cant_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_pendiente_48h] = TRUE())
)
~~~

### `cant_ppto_conv_post_48h`

- Display Folder: `04 Post 48h`
- Format string: `#,0`
- Descripción del modelo: Cantidad de presupuestos pendientes a las 48 horas que presentan ejecución posterior.
- Propósito funcional: Cantidad de presupuestos pendientes a las 48 horas que presentan ejecución posterior.
- Dependencias de medidas: `[cant_ppto]`
- Columnas utilizadas directamente: `tbl_Presupuesto_Resumen[funnel_ejecucion_posterior]`
- Benchmark global actual: **233**

~~~DAX
CALCULATE(
    [cant_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_ejecucion_posterior] = TRUE())
)
~~~

### `cant_pax_conv_post_48h_ppto`

- Display Folder: `04 Post 48h`
- Format string: `#,0`
- Descripción del modelo: Pacientes distintos no blancos con presupuestos recuperados mediante ejecución posterior a las 48 horas.
- Propósito funcional: Pacientes distintos no blancos con presupuestos recuperados mediante ejecución posterior a las 48 horas.
- Dependencias de medidas: `[cant_pax_ppto]`
- Columnas utilizadas directamente: `tbl_Presupuesto_Resumen[funnel_ejecucion_posterior]`
- Benchmark global actual: **191**

~~~DAX
CALCULATE(
    [cant_pax_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_ejecucion_posterior] = TRUE())
)
~~~

### `tasa_ppto_conv_post_48h`

- Display Folder: `04 Post 48h`
- Format string: `0.0%`
- Descripción del modelo: Presupuestos con ejecución posterior al corte / presupuestos pendientes a las 48 horas.
- Propósito funcional: Presupuestos con ejecución posterior al corte / presupuestos pendientes a las 48 horas.
- Dependencias de medidas: `[cant_ppto_conv_post_48h]`, `[cant_ppto_pend_48h]`
- Columnas utilizadas directamente: ninguna; utiliza una tabla o medidas dependientes
- Benchmark global actual: **2,9926 %**

~~~DAX
DIVIDE(
    [cant_ppto_conv_post_48h],
    [cant_ppto_pend_48h]
)
~~~

### `monto_conv_post_48h_real`

- Display Folder: `04 Post 48h`
- Format string: `S/ #,0`
- Descripción del modelo: Valor real confirmado posterior a 48 horas, limitado a los presupuestos recuperados del funnel.
- Propósito funcional: Valor real confirmado posterior a 48 horas, limitado a los presupuestos recuperados del funnel.
- Dependencias de medidas: ninguna
- Columnas utilizadas directamente: `tbl_Presupuesto_Resumen[ppto_valor_convertido_post_48h_real]`, `tbl_Presupuesto_Resumen[funnel_ejecucion_posterior]`
- Benchmark global actual: **S/ 106.927**

~~~DAX
CALCULATE(
    SUM('tbl_Presupuesto_Resumen'[ppto_valor_convertido_post_48h_real]),
    KEEPFILTERS('tbl_Presupuesto_Resumen'[funnel_ejecucion_posterior] = TRUE())
)
~~~

### `cant_ppto_post_48h_243`

- Display Folder: `04 Post 48h`
- Format string: `#,0`
- Descripción del modelo: Cantidad de presupuestos recuperados post 48 horas con evidencia gestionable del código 243.
- Propósito funcional: Cantidad de presupuestos recuperados post 48 horas con evidencia gestionable del código 243.
- Dependencias de medidas: `[cant_ppto]`
- Columnas utilizadas directamente: `tbl_Presupuesto_Resumen[conversion_post_48h_con_243_gestionable]`
- Benchmark global actual: **25**

~~~DAX
CALCULATE(
    [cant_ppto],
    KEEPFILTERS('tbl_Presupuesto_Resumen'[conversion_post_48h_con_243_gestionable] = TRUE())
)
~~~

### `cant_ppto_post_48h_sin_243`

- Display Folder: `04 Post 48h`
- Format string: `#,0`
- Descripción del modelo: Presupuestos recuperados post 48 horas sin evidencia gestionable del código 243.
- Propósito funcional: Presupuestos recuperados post 48 horas sin evidencia gestionable del código 243.
- Dependencias de medidas: `[cant_ppto_conv_post_48h]`, `[cant_ppto_post_48h_243]`
- Columnas utilizadas directamente: ninguna; utiliza una tabla o medidas dependientes
- Benchmark global actual: **208**

~~~DAX
[cant_ppto_conv_post_48h]
    - [cant_ppto_post_48h_243]
~~~

### `porc_post_48h_243`

- Display Folder: `04 Post 48h`
- Format string: `0.0%`
- Descripción del modelo: Presupuestos recuperados post 48 horas con evidencia 243 / total de presupuestos recuperados post 48 horas; representa participación, no efectividad del Call Center.
- Propósito funcional: Presupuestos recuperados post 48 horas con evidencia 243 / total de presupuestos recuperados post 48 horas; representa participación, no efectividad del Call Center.
- Dependencias de medidas: `[cant_ppto_post_48h_243]`, `[cant_ppto_conv_post_48h]`
- Columnas utilizadas directamente: ninguna; utiliza una tabla o medidas dependientes
- Benchmark global actual: **10,7296 %**

~~~DAX
DIVIDE(
    [cant_ppto_post_48h_243],
    [cant_ppto_conv_post_48h]
)
~~~

## Advertencia de nombres

La medida histórica `01_Dax_Base[cant_pax]` no pertenece a este catálogo. Usa `tbl_Trinidad[trin_pax_nro_documento]` sobre atenciones efectivas y tiene una semántica diferente. Para Presupuestos deben utilizarse `cant_pax_ppto`, `cant_pax_conv_48h_ppto` y `cant_pax_conv_post_48h_ppto`.
