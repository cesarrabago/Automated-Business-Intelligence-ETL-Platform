# Automated Business Intelligence ETL Platform

Plataforma automatizada de ETL y BI que extrae datos, los procesa con flujos en n8n (ETL), los almacena en una base de datos relacional (PostgreSQL) y los visualiza en Power BI. Simula una arquitectura de datos real para análisis y reporting empresarial.

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![n8n](https://img.shields.io/badge/n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

> Pipeline de BI de extremo a extremo: ingesta automatizada de CSV desde Gmail → transformación en JavaScript → UPSERT en PostgreSQL → dashboard ejecutivo en Power BI. Cero pasos manuales.

---

## Arquitectura del Sistema

![Arquitectura ETL & BI](architecture.png)

---

## Vista General del Proyecto


![Vista General del Proyecto](screenshots/preview.png)

---

## Problema de Negocio

Una red de concesionarios de motos Honda en **5 estados de India** no contaba con un sistema centralizado de reportes. Los gerentes regionales enviaban archivos CSV de forma ad-hoc por correo electrónico, y un analista dedicaba ~40 horas/mes a consolidarlos manualmente en Excel antes de poder realizar cualquier análisis.

| Punto de Dolor | Impacto |
|---|---|
| Consolidación manual de CSV | ~40 horas de analista / mes |
| Latencia de reportes | 3 días de retraso hasta la visibilidad ejecutiva |
| Tasa de registros duplicados | 15–20% por fusión manual de archivos |

Las decisiones de negocio sobre inventario, promociones financieras y venta cruzada de seguros se tomaban sobre **₹37.41M en ventas** con 3 días de antigüedad.

---

## Solución

| Paso | Herramienta | Acción |
|---|---|---|
| Disparador | Gmail API / OAuth2 | Detectar nuevo correo con adjunto CSV |
| Validar | Nodo Filter de n8n | Lista blanca de remitentes + regex de asunto |
| Extraer | Nodo Extract from File de n8n | Decodificación Base64 → más de 1.500 filas JSON |
| Transformar | Nodo JavaScript Code | Eliminación de nulos, conversión de tipos, normalización de fechas |
| Cargar | PostgreSQL | `INSERT ... ON CONFLICT DO UPDATE` (idempotente) |
| Notificar | Gmail | Correo con resumen de ejecución (filas ingresadas / actualizadas / omitidas) |

---


## Stack Tecnológico

| Capa | Tecnología |
|---|---|
| Orquestación | n8n (auto-hospedado) |
| Fuente | Gmail API / OAuth2 |
| Transformación | JavaScript |
| Almacenamiento | PostgreSQL 15 |
| Visualización | Power BI |
| Infraestructura | Docker Compose |

---

## Esquema de Base de Datos

Esquema en estrella con 1 tabla de hechos, 3 dimensiones, 1 tabla de auditoría y vistas de capa semántica consumidas por Power BI.

| Objeto | Tipo | Descripción |
|---|---|---|
| `sales_orders` | Hecho | Granularidad por transacción — una fila por pedido |
| `dim_products` | Dimensión | Catálogo de modelos de motos |
| `dim_geography` | Dimensión | Estado / región / nivel |
| `dim_payment` | Dimensión | Metadatos del método de pago |
| `pipeline_runs` | Auditoría | Cada ejecución registrada con conteo de filas + estado |
| `vw_monthly_kpis` | Vista | KPIs pre-agregados para Power BI |

*El esquema completo y los flujos de n8n están disponibles bajo solicitud.*

---

## Resultados

| Métrica | Antes | Después |
|---|---|---|
| Horas de analista / mes | ~40h | **0h** |
| Latencia de reportes | 3 días | **< 1 hora** |
| Tasa de duplicados | 15–20% | **0%** |
| Ventas netas rastreadas | Fragmentadas | **₹37.41M unificadas** |

---


## Hoja de Ruta

| Prioridad | Funcionalidad |
|---|---|
| P1 | Capa de transformación con dbt (modelos staging + mart) |
| P1 | Dashboard de monitoreo del pipeline sobre `pipeline_runs` |
| P2 | Manejo de errores + cola de mensajes fallidos + alertas en Slack |
| P2 | Despliegue en la nube (GCP + Cloud SQL + Power BI Service) |
| P3 | Migración a Snowflake / BigQuery |
