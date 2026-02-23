
# Descripción de la supervisión de rendimiento en Azure SQL

## ¿Qué es una línea de base?

- **Definición:** Una línea de base es un conjunto representativo de valores de rendimiento (métricas y comportamientos) recogidos durante un periodo estable de operación; sirve como referencia para detectar desviaciones, regresiones y anomalías.
- **Propósito:** Identificar el comportamiento "normal" del servicio para detectar anomalías, dimensionar recursos y definir umbrales de alerta realistas.

### Cómo establecer una línea de base (pasos prácticos)
1. **Definir el periodo representativo:** mínimo 1–2 semanas; ideal incluir picos de carga esperados.
2. **Seleccionar métricas clave:** CPU, I/O (latencia y throughput), memoria, uso de log, transacciones/s, waits, latencias de consulta.
3. **Recolectar datos:** activar Query Store, métricas de Azure Monitor, Extended Events y/o Diagnostic Settings hacia Log Analytics o Storage.
4. **Calcular percentiles y rangos:** p50, p75, p90, p95, p99 y máximos; documentar horarios pico.
5. **Guardar y versionar la línea base:** exportar a CSV, Log Analytics o Azure Storage para comparaciones.
6. **Revisar periódicamente:** actualizar tras cambios de arquitectura o de patrón de carga.

## Azure Monitor

- **Qué es:** Servicio central de telemetría en Azure que recopila métricas, logs y diagnósticos de recursos, incluyendo Azure SQL Database.
- **Retención por defecto:** Azure Monitor almacena métricas durante 93 días (≈3 meses).
- **Archivado y análisis a largo plazo:** exportar métricas y logs a:
	- **Azure Storage (Blob):** almacenamiento económico y retención personalizada.
	- **Log Analytics:** análisis con KQL y paneles, retención configurable.
	- **Event Hub:** para integración con SIEM o consumidores externos.

### Integración con Azure SQL
- En el recurso de la base de datos -> **Diagnostic settings**: enviar `Metrics`, `SQLInsights`, `Errors`, `Deadlocks`, `Timeouts`, `QueryStoreRuntimeStatistics` a Log Analytics / Storage / Event Hub.
- Usar **Azure Monitor Alerts** (métricas y consultas KQL) e **Azure SQL Insights** / **Query Performance Insight** para análisis guiado.

## Métricas importantes de SQL Server (relevantes en Azure SQL)
- **CPU (%)**
- **vCore/DTU Utilization**
- **Memory / Buffer pool usage**
- **IO throughput / IOPS / Read & Write latency**
- **Log write latency**
- **Transactions/sec / Batch requests/sec**
- **Active connections / Sessions**
- **TempDB usage**
- **Lock waits / Deadlocks / Blocking time**
- **Wait statistics (top waits)**
- **Query duration (p95/p99)**
- **Errors / Timeouts**
- **Storage percent used**

## ¿Qué son los Eventos Extendidos?

- **Definición:** Mecanismo ligero y flexible de recolección de eventos en SQL Server para diagnóstico, auditoría y rendimiento.
- **Ventajas frente a SQL Trace / Profiler:** menor sobrecarga, mayor flexibilidad en filtros (predicates), múltiples targets y mejor extensibilidad.

### Componentes de Eventos Extendidos
- **Event:** unidad de información (ej. `sql_batch_completed`).
- **Action:** datos adicionales asociados (ej. `sql_text`).
- **Predicate:** filtro que limita la captura.
- **Target:** destino de almacenamiento (`event_file`, `ring_buffer`, `histogram`, `event_counter`).
- **Session:** contenedor que agrupa eventos, acciones, filtros y targets.
- **Package:** agrupación por funcionalidad (p. ej. `sqlserver`).

### Canales (categorías)
- **Administrativo (Administrative):** cambios de configuración, inicio/parada.
- **Operativo (Operational):** eventos de operación diaria (conexiones, autenticación, errores).
- **Analítico (Analytic):** telemetría orientada a rendimiento.
- **Depurar (Debug):** trazas detalladas para diagnóstico profundo (activar temporalmente).

## Impacto en PaaS e IaaS
- **IaaS (SQL en VM):** control total; puede ejecutar sesiones XE y guardar archivos localmente.
- **PaaS (Azure SQL Database / Managed Instance):** en Single/Elastic Pool acceso limitado: usar Diagnostic Settings, Query Store y eventos expuestos por la plataforma; Managed Instance ofrece mayor control que Single Database.
- **Recomendación:** en PaaS activar Diagnostic Settings + Query Store; en Managed Instance o IaaS usar sesiones XE según necesidad.

## ¿Qué se puede supervisar con Eventos Extendidos? (ejemplos)
- Consultas y sentencias: `sql_batch_completed`, `sql_statement_completed` (duración, plan, texto).
- RPC y procedimientos: `rpc_completed`, `sp_statement_completed`.
- Bloqueos y deadlocks: `lock_acquired`, `deadlock_graph`.
- Waits y latencias: `wait_info`.
- Errores: `error_reported`.
- Recursos: memoria, I/O y eventos de sistema.
- Seguridad: logins y cambios de permisos.

## Monitores de base de datos

- **Definición:** Conjunto de herramientas y procesos que vigilan salud, rendimiento, integridad y disponibilidad de bases de datos.
- **Ejemplos en Azure:** Query Performance Insight, Azure SQL Insights / Intelligent Insights, Automatic Tuning, Azure Monitor Alerts, Log Analytics Workbooks.
- **Funcionalidades:** detección de anomalías, alertas, historial de métricas, recomendaciones de optimización y auditoría.

## Información del rendimiento de las consultas y cómo verla en el Portal de Azure

- **Qué es:** Conjunto de capacidades basadas en `Query Store` y herramientas del portal (Query Performance Insight) que muestran las consultas con mayor impacto, duración, ejecución y planes históricos.
- **Requisitos:** `Query Store` activado (por defecto suele estar habilitado en Azure SQL DB).

### Pasos para acceder desde el Portal de Azure
1. Abrir la base de datos en el Portal de Azure.
2. En el menú lateral seleccionar **Query Performance Insight** (o Intelligent Performance → Query Performance Insight).
3. Elegir rango temporal y la métrica a analizar (CPU, duración, ejecuciones).
4. Ordenar la lista de consultas por impacto y seleccionar una consulta para ver: texto SQL, número de ejecuciones, tendencia y plan (vía Query Store).
5. Para análisis profundo abrir **Query Store**: ver runtime statistics, history de planes y forzar planes si es necesario.

### Consejos prácticos
- Correlacionar picos en métricas (I/O, CPU) con aumentos en duración de consultas para encontrar causa raíz.
- Exportar datos a Log Analytics o Storage si se necesita retención histórica superior a 93 días.
- Usar Query Store para identificar regresiones por cambios de plan y aplicar `forced plan` sólo tras validación.
