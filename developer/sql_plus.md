# Guía de Estudio: Análisis de Datos y Reporting en Oracle SQL

**Caso de Estudio:** Análisis de Cumplimiento de SLA

Esta guía documenta el flujo de trabajo completo: desde la exploración inicial y la limpieza visual, hasta el análisis de datos avanzado y la automatización del entorno en Oracle SQL*Plus.

---

## 1. Fase de Reconocimiento (Discovery)

Antes de lanzar consultas complejas, debemos entender el "terreno": dónde estamos conectados y qué estructuras de datos existen.

### 1.1 Verificación del Entorno

En Oracle, las "bases de datos" suelen referirse a instancias completas (CDB/PDB). Los datos de usuario viven en **Esquemas**.

* **Comando clave:** `show con_name` (Verifica el contenedor actual).
* **Error común:** `ORA-00942`. Ocurre al intentar consultar vistas protegidas del sistema (`v$pdbs`) sin permisos de DBA.

### 1.2 Listado de Tablas (El Catálogo)

Para ver los objetos que pertenecen a nuestro usuario actual:

```sql
SELECT table_name FROM user_tables;

```

**Objetos identificados:**

1. `SLA_ANALISTAS` (Maestro de empleados/agentes).
2. `SLA_INCIDENCIAS` (Transaccional de tickets).

### 1.3 Ingeniería Inversa de la Estructura

Para entender cómo relacionar las tablas sin documentación, usamos `DESC` (Describe).

```sql
DESC SLA_ANALISTAS;
DESC SLA_INCIDENCIAS;

```

* **Hallazgo:** Ambas tablas comparten el campo `ID_ANALISTA`. Esta será nuestra **Llave de Unión (Join Key)**.

---

## 2. Fase de Relación y Formato (Join & Display)

El objetivo es crear una vista unificada que cruce a los responsables con sus tareas.

### 2.1 La Consulta Relacional (INNER JOIN)

Unimos las tablas para obtener un listado legible.

```sql
SELECT
    a.NOMBRE_ANALISTA,
    a.AREA,
    i.NRO_INCIDENCIA,
    i.NOMBRE_CLIENTE,
    i.CUMPLE_SLA
FROM SLA_ANALISTAS a
JOIN SLA_INCIDENCIAS i ON a.ID_ANALISTA = i.ID_ANALISTA;

```

---

## 3. Fase de Análisis Agregado (Analytics)

Convertimos los datos crudos ("raw data") en información útil mediante agrupación y lógica condicional.

### 3.1 Conteo Condicional (Pivot)

En lugar de contar filas totales, queremos contar "éxitos" y "fracasos" por separado. Usamos `SUM(CASE ...)` para evaluar condiciones fila por fila.

```sql
SELECT
    a.NOMBRE_ANALISTA,
    a.AREA,
    COUNT(*) AS TOTAL_INCIDENCIAS,
    -- Contador de Éxitos
    SUM(CASE WHEN i.CUMPLE_SLA = 'S' THEN 1 ELSE 0 END) AS CUMPLIO,
    -- Contador de Fallos
    SUM(CASE WHEN i.CUMPLE_SLA = 'N' THEN 1 ELSE 0 END) AS NO_CUMPLIO
FROM SLA_ANALISTAS a
JOIN SLA_INCIDENCIAS i ON a.ID_ANALISTA = i.ID_ANALISTA
GROUP BY a.NOMBRE_ANALISTA, a.AREA
ORDER BY NO_CUMPLIO DESC;

```

### 3.2 Cálculo de KPIs (Porcentajes)

Para normalizar el rendimiento, calculamos la efectividad porcentual.

* *Fórmula:* `(Éxitos / Total) * 100`

```sql
SELECT
    a.NOMBRE_ANALISTA,
    a.AREA,
    COUNT(*) AS TOTAL,
    -- Cálculo matemático + concatenación de símbolo '%'
    ROUND((SUM(CASE WHEN i.CUMPLE_SLA = 'S' THEN 1 ELSE 0 END) / COUNT(*)) * 100, 1) || '%' AS EFECTIVIDAD
FROM SLA_ANALISTAS a
JOIN SLA_INCIDENCIAS i ON a.ID_ANALISTA = i.ID_ANALISTA
GROUP BY a.NOMBRE_ANALISTA, a.AREA
ORDER BY EFECTIVIDAD DESC;

```

---

## 4. Hallazgos del Caso (Insights)

Tras ejecutar las consultas, detectamos los siguientes patrones:

| Patrón Detectado | Evidencia (Data) | Acción Recomendada |
| --- | --- | --- |
| **Fallo Sistémico en Averías** | Área `SVA AVERIAS` tiene 0% de efectividad. | Auditar procesos técnicos de esa área específica. |
| **Sobrecarga de Trabajo** | `Chapoñan` tiene 2x carga que el promedio. | Redistribuir tickets para evitar burnout. |
| **Excelencia en Reclamos** | `Gutiérrez` mantiene 100% efectividad en área crítica. | Usar como mentor para otros analistas. |

---

## 5. Glosario Rápido

* **`user_tables`**: Diccionario de datos con las tablas del usuario.
* **`JOIN`**: Operación para combinar conjuntos de datos.
* **`GROUP BY`**: Colapsar múltiples filas en una sola por categoría.
* **`CASE WHEN`**: Estructura `IF-THEN-ELSE` dentro de SQL.
* **`login.sql`**: Archivo de configuración de inicio (ver sección 6).
* **`@archivo`**: Comando para ejecutar un script SQL externo.

---

## 6. Automatización y Persistencia (Workflow)

SQL*Plus olvida la configuración visual al cerrar sesión. Para trabajar como un profesional (DevOps style), usamos archivos de configuración persistentes.

### 6.1 Configuración Global (`login.sql`)

Este archivo actúa como el `.bashrc` de la base de datos. SQL*Plus lo busca y ejecuta automáticamente al iniciar.

**Instrucciones:**
Crea un archivo llamado `login.sql` en tu carpeta de trabajo con el siguiente contenido:

```sql
-- login.sql
-- Configuración visual base para cualquier sesión
SET LINESIZE 300    -- Ancho de línea (evita saltos feos)
SET PAGESIZE 100    -- Filas antes de repetir cabeceras
SET TRIMSPOOL ON    -- Elimina espacios en blanco al final
SET TAB OFF         -- Usa espacios en lugar de tabuladores

-- Cambia el prompt para saber quién eres
SET SQLPROMPT "_USER > "

```

### 6.2 Scripts de Reporte (`.sql`)

Para no reescribir consultas complejas o formatos específicos de tablas, guárdalos en archivos `.sql`.

**Ejemplo:** Crea un archivo `reporte_sla.sql`:

```sql
-- reporte_sla.sql
-- 1. Formatos específicos (Solo aplican a estas columnas)
COLUMN NOMBRE_ANALISTA FORMAT A35
COLUMN AREA FORMAT A15
COLUMN NOMBRE_CLIENTE FORMAT A35
COLUMN CUMPLE_SLA FORMAT A10

-- 2. La consulta compleja
SELECT
    a.NOMBRE_ANALISTA,
    a.AREA,
    i.NRO_INCIDENCIA,
    i.NOMBRE_CLIENTE,
    i.CUMPLE_SLA
FROM SLA_ANALISTAS a
JOIN SLA_INCIDENCIAS i ON a.ID_ANALISTA = i.ID_ANALISTA;

```

### 6.3 Ejecución

Dentro de SQL*Plus, simplemente invoca el script con una arroba (`@`):

```sql
SLA_USER > @reporte_sla

```

---

*Guía actualizada el 23 de Enero de 2026.*
