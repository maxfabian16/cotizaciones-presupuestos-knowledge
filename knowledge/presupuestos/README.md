# Módulo Presupuestos — Power BI

## Propósito

Este directorio conserva el contexto técnico y funcional del módulo de Presupuestos del modelo Power BI de Marketing/Fidelización Médicos. El snapshot fue generado en modo read-only el 14/09/2026 sobre la copia abierta del PBIX.

Sirve como referencia persistente para:

- crear y corregir medidas DAX;
- interpretar filtros y relaciones;
- diagnosticar resultados;
- construir el dashboard táctico;
- trabajar sin acceso directo al PBIX.

## Arquitectura canónica

```text
tbl_Presupuestos
  ├─ tbl_Presupuestos_Detalle
  ├─ tbl_Atenciones
  │    ├─ tbl_Resultados
  │    ├─ tbl_Trinidad
  │    ├─ tbl_Solicitud_All
  │    └─ tbl_Descuentos
  ├─ tbl_Presupuesto_Detalle_Ejecucion
  ├─ tbl_Presupuesto_Adicional_Ejecucion
  └─ tbl_Presupuesto_Resumen
        ├─ tbl_Calendario_Presupuestos
        └─ 06 DAX Presupuestos
```

La tabla central del dashboard es `tbl_Presupuesto_Resumen`: una fila por presupuesto desde `ppto_fecha_48h >= 01/07/2026`, 82.652 filas, 82.652 presupuestos distintos y cero duplicados.

## Estado actual

- `tbl_Calendario_Presupuestos`: 74 fechas, del 01/07/2026 al 12/09/2026.
- `tbl_Gestion_Diaria_Paciente_Item`: 151.761 oportunidades únicas paciente × fecha de gestión × análisis.
- `tbl_Presupuesto_Detalle_Ejecucion`: 322.233 líneas originales, sin duplicados de detalle.
- `tbl_Presupuesto_Adicional_Ejecucion`: 2.732 evidencias adicionales atención × análisis.
- `tbl_Presupuesto_Resumen`: 82.652 presupuestos únicos, tabla central del dashboard.
- `06 DAX Presupuestos`: 21 medidas `Ready`; `05 Comparativos` está vacío.

## Documentos

- [AI_CONTEXT.md](AI_CONTEXT.md): documento canónico autocontenido para personas y asistentes de IA.
- [DAX_CATALOG.md](DAX_CATALOG.md): expresiones exactas y metadatos de las 21 medidas.
- [model_snapshot.json](model_snapshot.json): snapshot machine-readable de tablas, relaciones, reglas, benchmarks y medidas.

## Reglas de uso

- Para el dashboard usar `tbl_Presupuesto_Resumen`, no reconstruir el universo desde las facts.
- Para pacientes de presupuestos usar `cant_pax_ppto`, no la medida histórica `01_Dax_Base[cant_pax]`.
- Año, mes y día deben venir de `tbl_Calendario_Presupuestos`.
- Sede y tipo de paciente deben venir directamente de `tbl_Presupuesto_Resumen`.
- El código 243 es evidencia positiva de gestión con descuento; su ausencia no demuestra ausencia de llamada.
- Las cinco tablas intermedias `tbl_Presupuesto_Conversion`, `tbl_Presupuesto_Valor`, `tbl_Presupuesto_Momento_Conversion`, `tbl_Presupuesto_Funnel` y `tbl_Presupuesto_Gestion_CallCenter` permanecen en el modelo como legado. El Resumen vigente no depende de ellas.

## Fuente de verdad

El estado real del modelo al momento del snapshot prevalece sobre documentos históricos. Los benchmarks y expresiones de medidas incluidos aquí fueron reconsultados directamente del modelo sin procesarlo ni modificarlo.
