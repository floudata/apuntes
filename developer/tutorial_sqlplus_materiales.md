# Tutorial Completo de Oracle SQL*Plus: Informática de Materiales
## De Básico a Avanzado

**Caso de Estudio:** Sistema de Gestión de Base de Datos de Materiales y Simulaciones

Esta guía documenta el flujo completo de trabajo con Oracle SQL*Plus aplicado a informática de materiales: desde la conexión básica y exploración de datos, hasta análisis avanzados de propiedades, correlaciones estructura-propiedad y gestión de resultados de simulaciones computacionales (DFT, MD, Monte Carlo).

---

## Índice

1. [Fundamentos y Conexión](#1-fundamentos-y-conexión)
2. [Reconocimiento del Entorno (Discovery)](#2-reconocimiento-del-entorno-discovery)
3. [Consultas Básicas (SELECT)](#3-consultas-básicas-select)
4. [Relaciones entre Tablas (JOIN)](#4-relaciones-entre-tablas-join)
5. [Análisis Agregado (GROUP BY)](#5-análisis-agregado-group-by)
6. [Consultas Avanzadas (Subqueries, CTEs)](#6-consultas-avanzadas-subqueries-ctes)
7. [Funciones Analíticas (Window Functions)](#7-funciones-analíticas-window-functions)
8. [Optimización y Performance](#8-optimización-y-performance)
9. [Automatización y Persistencia](#9-automatización-y-persistencia)
10. [Casos de Uso Avanzados](#10-casos-de-uso-avanzados)

---

## 1. Fundamentos y Conexión

### 1.1 ¿Qué es SQL*Plus?

SQL*Plus es el cliente de línea de comandos oficial de Oracle Database. Es la herramienta fundamental para:

- Ejecutar consultas SQL y scripts PL/SQL
- Administrar la base de datos
- Generar reportes formateados
- Automatizar tareas mediante scripts

**Analogía:** SQL*Plus es como el terminal bash para Linux, pero específico para Oracle.

### 1.2 Conectarse a la Base de Datos

```bash
# Desde terminal
sqlplus usuario/password@hostname:puerto/servicio

# Ejemplo práctico
sqlplus materials_user/mat2024@localhost:1521/MATDB
```

**Dentro de SQL*Plus:**

```sql
-- Conectar como usuario específico
CONNECT materials_user/mat2024@MATDB

-- Ver usuario actual
SHOW USER

-- Salir
EXIT
```

**Salida:**
```
USER is "MATERIALS_USER"
```

### 1.3 Configuración Visual Básica

Al iniciar SQL*Plus, la vista por defecto es terrible. Configurémosla:

```sql
-- Ancho de línea para evitar saltos
SET LINESIZE 200

-- Filas antes de repetir cabeceras
SET PAGESIZE 50

-- Mostrar tiempo de ejecución
SET TIMING ON

-- Formato de números
SET NUMFORMAT 999999.99

-- Prompt personalizado
SET SQLPROMPT "MATDB> "
```

---

## 2. Reconocimiento del Entorno (Discovery)

Antes de analizar datos, debemos entender qué existe en nuestra base de datos.

### 2.1 Verificar Conexión y Contexto

```sql
-- ¿Dónde estoy conectado?
SELECT SYS_CONTEXT('USERENV', 'CURRENT_SCHEMA') AS ESQUEMA_ACTUAL,
       SYS_CONTEXT('USERENV', 'DB_NAME') AS BASE_DATOS,
       SYS_CONTEXT('USERENV', 'SESSION_USER') AS USUARIO
FROM DUAL;

-- Verificar versión de Oracle
SELECT * FROM v$version;
```

**Salida:**
```
ESQUEMA_ACTUAL   BASE_DATOS   USUARIO
---------------- ------------ ----------------
MATERIALS_USER   MATDB        MATERIALS_USER

BANNER
--------------------------------------------------------------------------------
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
```

### 2.2 Explorar el Catálogo de Tablas

El diccionario de datos de Oracle contiene metadata sobre todos los objetos.

```sql
-- Listar TODAS las tablas del usuario actual
SELECT table_name, num_rows, last_analyzed
FROM user_tables
ORDER BY table_name;

-- Ver solo tablas relacionadas con materiales
SELECT table_name 
FROM user_tables 
WHERE table_name LIKE '%MAT%' OR table_name LIKE '%PROP%';

-- Contar cuántas tablas tengo
SELECT COUNT(*) AS total_tablas FROM user_tables;
```

**Salida de la primera consulta:**
```
TABLE_NAME                  NUM_ROWS LAST_ANALYZED
--------------------------- -------- -------------
ESTRUCTURAS_CRISTALINAS         234  15-ENE-26
EXPERIMENTOS_LAB                156  18-ENE-26
MATERIALES                      487  20-ENE-26
PROPIEDADES_MECANICAS           412  20-ENE-26
PROPIEDADES_TERMICAS            398  20-ENE-26
REFERENCIAS_BIBLIOGRAFICAS      789  12-ENE-26
SIMULACIONES_DFT                523  22-ENE-26

7 filas seleccionadas.
```

**Salida del conteo:**
```
TOTAL_TABLAS
------------
           7
```

**Resultado esperado:**
```
MATERIALES
PROPIEDADES_MECANICAS
PROPIEDADES_TERMICAS
SIMULACIONES_DFT
EXPERIMENTOS_LAB
ESTRUCTURAS_CRISTALINAS
REFERENCIAS_BIBLIOGRAFICAS
```

### 2.3 Ingeniería Inversa: Estructura de Tablas

El comando `DESCRIBE` (o `DESC`) muestra la estructura completa de una tabla.

```sql
-- Sintaxis
DESC nombre_tabla

-- Ejemplos
DESC MATERIALES;
DESC PROPIEDADES_MECANICAS;
DESC SIMULACIONES_DFT;
```

**Salida típica:**
```
Name                    Null?    Type
----------------------- -------- ----------------
ID_MATERIAL             NOT NULL NUMBER(10)
NOMBRE_MATERIAL                  VARCHAR2(100)
FORMULA_QUIMICA                  VARCHAR2(50)
SISTEMA_CRISTALINO               VARCHAR2(30)
GRUPO_ESPACIAL                   VARCHAR2(20)
DENSIDAD                         NUMBER(8,3)
FECHA_REGISTRO                   DATE
```

### 2.4 Entender las Relaciones (Foreign Keys)

```sql
-- Ver restricciones de llave foránea
SELECT 
    a.constraint_name,
    a.table_name AS tabla_hija,
    b.table_name AS tabla_padre,
    a.constraint_type
FROM user_constraints a
JOIN user_constraints b ON a.r_constraint_name = b.constraint_name
WHERE a.constraint_type = 'R'
ORDER BY a.table_name;
```

---

## 3. Consultas Básicas (SELECT)

El 80% del trabajo en SQL es extraer y filtrar datos. Dominemos el SELECT.

### 3.1 SELECT Simple: Proyección de Columnas

```sql
-- Ver todo (NUNCA uses esto en producción con tablas grandes)
SELECT * FROM MATERIALES;

-- Mejor: Seleccionar solo lo necesario
SELECT nombre_material, formula_quimica, densidad 
FROM MATERIALES;

-- Con alias para claridad
SELECT 
    nombre_material AS Material,
    formula_quimica AS Fórmula,
    densidad AS "Densidad (g/cm³)"
FROM MATERIALES;
```

### 3.2 Filtrado con WHERE

```sql
-- Operadores de comparación
SELECT nombre_material, sistema_cristalino
FROM MATERIALES
WHERE densidad > 5.0;

-- Operadores lógicos (AND, OR, NOT)
SELECT nombre_material, formula_quimica
FROM MATERIALES
WHERE sistema_cristalino = 'Cúbico' 
  AND densidad BETWEEN 2.0 AND 8.0;

-- Búsqueda de patrones (LIKE)
SELECT nombre_material, formula_quimica
FROM MATERIALES
WHERE formula_quimica LIKE '%Fe%'  -- Materiales con hierro
   OR formula_quimica LIKE '%Cu%'; -- Materiales con cobre

-- Valores nulos
SELECT nombre_material, grupo_espacial
FROM MATERIALES
WHERE grupo_espacial IS NULL;
```

**Ejemplo de salida (materiales con densidad > 5.0):**
```
NOMBRE_MATERIAL                          SISTEMA_CRISTALINO
---------------------------------------- --------------------
Wolframio                                Cúbico
Tungsteno-Carburo                        Hexagonal
Acero AISI 4340                          Cúbico
Hierro puro                              Cúbico
Níquel                                   Cúbico
Cobalto                                  Hexagonal

6 filas seleccionadas.

Elapsed: 00:00:00.12
```

**Ejemplo de salida (búsqueda con LIKE - materiales con Fe):**
```
NOMBRE_MATERIAL                          FORMULA_QUIMICA
---------------------------------------- --------------------
Hematita                                 Fe2O3
Magnetita                                Fe3O4
Hierro puro                              Fe
Acero AISI 304                           Fe-Cr18-Ni10
Ferroportlandita                         FeSO4·7H2O

5 filas seleccionadas.
```

### 3.3 Ordenamiento (ORDER BY)

```sql
-- Orden ascendente (por defecto)
SELECT nombre_material, densidad
FROM MATERIALES
ORDER BY densidad;

-- Orden descendente
SELECT nombre_material, densidad
FROM MATERIALES
ORDER BY densidad DESC;

-- Ordenamiento múltiple
SELECT nombre_material, sistema_cristalino, densidad
FROM MATERIALES
ORDER BY sistema_cristalino ASC, densidad DESC;
```

### 3.4 Limitar Resultados

```sql
-- Top 10 materiales más densos
SELECT nombre_material, densidad
FROM MATERIALES
WHERE densidad IS NOT NULL
ORDER BY densidad DESC
FETCH FIRST 10 ROWS ONLY;

-- Con offset (saltar primeros 5)
SELECT nombre_material, densidad
FROM MATERIALES
ORDER BY densidad DESC
OFFSET 5 ROWS FETCH NEXT 10 ROWS ONLY;
```

**Salida (Top 10):**
```
NOMBRE_MATERIAL                          DENSIDAD
---------------------------------------- --------
Osmio                                      22.590
Iridio                                     22.560
Platino                                    21.450
Renio                                      21.020
Wolframio                                  19.250
Oro                                        19.320
Tántalo                                    16.690
Uranio                                     19.050
Tungsteno-Carburo                          15.630
Plutonio                                   19.840

10 filas seleccionadas.

Elapsed: 00:00:00.08
```

### 3.5 Operadores de Cálculo

```sql
-- Conversión de unidades inline
SELECT 
    nombre_material,
    densidad AS "Densidad (g/cm³)",
    densidad * 1000 AS "Densidad (kg/m³)",
    densidad / 8.96 AS "Densidad Relativa (ref: Cu)"
FROM MATERIALES;

-- Concatenación de strings
SELECT 
    nombre_material || ' (' || formula_quimica || ')' AS Identificador,
    sistema_cristalino
FROM MATERIALES;
```

**Salida (conversión de unidades):**
```
NOMBRE_MATERIAL                          Densidad (g/cm³) Densidad (kg/m³) Densidad Relativa (ref: Cu)
---------------------------------------- ---------------- ---------------- ----------------------------
Aluminio                                            2.700         2700.000                        0.301
Cobre                                               8.960         8960.000                        1.000
Titanio                                             4.506         4506.000                        0.503
Acero AISI 304                                      7.930         7930.000                        0.885
Grafeno                                             2.260         2260.000                        0.252

5 filas seleccionadas.
```

**Salida (concatenación):**
```
IDENTIFICADOR                                              SISTEMA_CRISTALINO
---------------------------------------------------------- --------------------
Aluminio (Al)                                              Cúbico FCC
Cobre (Cu)                                                 Cúbico FCC
Diamante (C)                                               Cúbico
Grafito (C)                                                Hexagonal
Silicio (Si)                                               Cúbico

5 filas seleccionadas.
```

---

## 4. Relaciones entre Tablas (JOIN)

En bases de datos normalizadas, la información está distribuida. Los JOINs conectan las piezas.

### 4.1 Modelo de Datos del Caso de Estudio

Nuestro esquema tiene las siguientes tablas principales:

```
MATERIALES (id_material, nombre_material, formula_quimica, sistema_cristalino)
    ↓ 1:N
PROPIEDADES_MECANICAS (id_propiedad, id_material, modulo_young, dureza, tenacidad)
PROPIEDADES_TERMICAS (id_propiedad, id_material, conductividad, capacidad_calorifica)
SIMULACIONES_DFT (id_simulacion, id_material, energia_formacion, band_gap, software)
EXPERIMENTOS_LAB (id_experimento, id_material, tipo_experimento, resultado, fecha)
```

### 4.2 INNER JOIN: Intersección

Solo devuelve registros que coinciden en ambas tablas.

```sql
-- Materiales con sus propiedades mecánicas
SELECT 
    m.nombre_material,
    m.formula_quimica,
    pm.modulo_young,
    pm.dureza,
    pm.tenacidad
FROM MATERIALES m
INNER JOIN PROPIEDADES_MECANICAS pm ON m.id_material = pm.id_material;
```

**Caso Real:** "Dame todos los óxidos con su módulo de Young"

```sql
SELECT 
    m.nombre_material,
    m.formula_quimica,
    pm.modulo_young AS "E (GPa)"
FROM MATERIALES m
INNER JOIN PROPIEDADES_MECANICAS pm ON m.id_material = pm.id_material
WHERE m.formula_quimica LIKE '%O%'  -- Contiene oxígeno
ORDER BY pm.modulo_young DESC;
```

**Salida:**
```
NOMBRE_MATERIAL                          FORMULA_QUIMICA              E (GPa)
---------------------------------------- ---------------------------- --------
Óxido de Aluminio (Alúmina)              Al2O3                          380.00
Dióxido de Circonio                      ZrO2                           210.00
Óxido de Magnesio                        MgO                            250.00
Hematita                                 Fe2O3                          220.00
Cuarzo                                   SiO2                            73.00
Magnetita                                Fe3O4                          180.00
Mullita                                  3Al2O3·2SiO2                   145.00
Vidrio soda-cal                          Na2O-CaO-SiO2                   70.00

8 filas seleccionadas.

Elapsed: 00:00:00.15
```

### 4.3 LEFT JOIN: Todo de la izquierda

Devuelve TODOS los materiales, aunque no tengan propiedades registradas.

```sql
-- Encontrar materiales SIN datos de propiedades mecánicas
SELECT 
    m.nombre_material,
    m.formula_quimica,
    pm.modulo_young
FROM MATERIALES m
LEFT JOIN PROPIEDADES_MECANICAS pm ON m.id_material = pm.id_material
WHERE pm.id_material IS NULL;  -- No hay match = NULL
```

**Salida:**
```
NOMBRE_MATERIAL                          FORMULA_QUIMICA              MODULO_YOUNG
---------------------------------------- ---------------------------- ------------
Aerogel de Sílice                        SiO2 (poroso)                            
Nanotubo de Carbono (teórico)            C                                        
Fullereno C60                            C60                                      
Grafeno monocapa                         C (2D)                                   
Metamaterial fotónico                    Variable                                 

5 filas seleccionadas.

-- Estos materiales necesitan caracterización mecánica
```

### 4.4 Múltiples JOINs

```sql
-- Vista completa: Material + Propiedades Mecánicas + Propiedades Térmicas
SELECT 
    m.nombre_material,
    m.sistema_cristalino,
    pm.modulo_young AS "E (GPa)",
    pm.dureza AS "HV",
    pt.conductividad AS "k (W/mK)",
    pt.capacidad_calorifica AS "Cp (J/kgK)"
FROM MATERIALES m
INNER JOIN PROPIEDADES_MECANICAS pm ON m.id_material = pm.id_material
INNER JOIN PROPIEDADES_TERMICAS pt ON m.id_material = pt.id_material
ORDER BY pm.modulo_young DESC;
```

**Salida:**
```
NOMBRE_MATERIAL                  SISTEMA_CRISTALINO    E (GPa)     HV k (W/mK) Cp (J/kgK)
-------------------------------- -------------------- -------- ------ -------- ----------
Diamante                         Cúbico                1050.00  10000  2200.00     509.00
Carburo de Silicio               Hexagonal              450.00   2800   120.00     750.00
Carburo de Boro                  Hexagonal              460.00   3200    30.00     950.00
Óxido de Aluminio (Alúmina)      Hexagonal              380.00   2100    30.00     775.00
Carburo de Tungsteno             Hexagonal              700.00   2400   110.00     203.00
Acero AISI 4340                  Cúbico                 210.00    350    44.50     475.00
Titanio (Ti-6Al-4V)              Hexagonal              113.80    350     7.40     526.00
Aluminio 6061-T6                 Cúbico FCC              68.90     95   167.00     896.00

8 filas seleccionadas.

Elapsed: 00:00:00.23
```

### 4.5 Ejemplo Avanzado: Material + Simulaciones

```sql
-- Comparar energía de formación experimental vs DFT
SELECT 
    m.nombre_material,
    m.formula_quimica,
    s.software AS "Código DFT",
    s.energia_formacion AS "E_form (eV/atom)",
    s.band_gap AS "Eg (eV)",
    s.fecha_calculo
FROM MATERIALES m
INNER JOIN SIMULACIONES_DFT s ON m.id_material = s.id_material
WHERE s.band_gap > 0  -- Solo semiconductores/aislantes
ORDER BY s.band_gap ASC;
```

---

## 5. Análisis Agregado (GROUP BY)

Cuando necesitamos **resumir**, **contar** o **calcular estadísticas**.

### 5.1 Funciones de Agregación Básicas

```sql
-- COUNT: Contar materiales
SELECT COUNT(*) AS total_materiales FROM MATERIALES;

-- AVG: Densidad promedio
SELECT AVG(densidad) AS densidad_promedio FROM MATERIALES;

-- MIN y MAX
SELECT 
    MIN(densidad) AS densidad_minima,
    MAX(densidad) AS densidad_maxima
FROM MATERIALES;

-- SUM: Suma total (ejemplo: costos)
SELECT SUM(costo_sintesis) AS costo_total_investigacion 
FROM EXPERIMENTOS_LAB;
```

### 5.2 GROUP BY: Agrupar por Categorías

**Pregunta:** "¿Cuántos materiales tengo por sistema cristalino?"

```sql
SELECT 
    sistema_cristalino,
    COUNT(*) AS cantidad
FROM MATERIALES
GROUP BY sistema_cristalino
ORDER BY cantidad DESC;
```

**Resultado:**
```
SISTEMA_CRISTALINO       CANTIDAD
----------------------- ---------
Cúbico                        145
Hexagonal                      89
Tetragonal                     56
Ortorrómbico                   34
Monoclínico                    28
Triclínico                     12
Trigonal                       18
Amorfo                         15

8 filas seleccionadas.

Elapsed: 00:00:00.05
```

### 5.3 Estadísticas por Grupo

```sql
-- Propiedades mecánicas promedio por sistema cristalino
SELECT 
    m.sistema_cristalino,
    COUNT(*) AS n_muestras,
    ROUND(AVG(pm.modulo_young), 2) AS "E_avg (GPa)",
    ROUND(STDDEV(pm.modulo_young), 2) AS "E_std (GPa)",
    ROUND(AVG(pm.dureza), 1) AS "HV_avg"
FROM MATERIALES m
INNER JOIN PROPIEDADES_MECANICAS pm ON m.id_material = pm.id_material
GROUP BY m.sistema_cristalino
ORDER BY AVG(pm.modulo_young) DESC;
```

**Salida:**
```
SISTEMA_CRISTALINO       N_MUESTRAS E_avg (GPa) E_std (GPa) HV_avg
------------------------ ---------- ------------ ----------- -------
Cúbico                           89       425.67      198.34  2850.5
Hexagonal                        67       387.23      156.89  2456.7
Tetragonal                       34       298.45      134.21  1987.3
Ortorrómbico                     21       245.12       98.76  1654.2
Trigonal                         15       312.56      145.32  2123.8
Monoclínico                      12       189.34       67.45  1234.5
Triclínico                        8       167.89       54.32  1089.2
Amorfo                           10       156.23       89.12   987.6

8 filas seleccionadas.

Elapsed: 00:00:00.18
```

### 5.4 HAVING: Filtrar Grupos

**WHERE** filtra filas ANTES de agrupar.  
**HAVING** filtra grupos DESPUÉS de agrupar.

```sql
-- Sistemas cristalinos con al menos 20 materiales
SELECT 
    sistema_cristalino,
    COUNT(*) AS cantidad
FROM MATERIALES
GROUP BY sistema_cristalino
HAVING COUNT(*) >= 20
ORDER BY cantidad DESC;
```

**Caso Real:** "Materiales con más de 5 simulaciones DFT registradas"

```sql
SELECT 
    m.nombre_material,
    COUNT(s.id_simulacion) AS num_simulaciones,
    AVG(s.energia_formacion) AS energia_promedio
FROM MATERIALES m
INNER JOIN SIMULACIONES_DFT s ON m.id_material = s.id_material
GROUP BY m.nombre_material
HAVING COUNT(s.id_simulacion) > 5
ORDER BY num_simulaciones DESC;
```

### 5.5 CASE WHEN: Lógica Condicional

Clasificar materiales según propiedades.

```sql
-- Clasificar materiales por conductividad eléctrica
SELECT 
    m.nombre_material,
    pt.conductividad_electrica AS "σ (S/m)",
    CASE 
        WHEN pt.conductividad_electrica > 1e7 THEN 'Conductor'
        WHEN pt.conductividad_electrica > 1e-2 THEN 'Semiconductor'
        ELSE 'Aislante'
    END AS clasificacion
FROM MATERIALES m
INNER JOIN PROPIEDADES_TERMICAS pt ON m.id_material = pt.id_material
ORDER BY pt.conductividad_electrica DESC;
```

**Salida:**
```
NOMBRE_MATERIAL                          σ (S/m)        CLASIFICACION
---------------------------------------- -------------- --------------
Plata                                    6.30000000E+07 Conductor
Cobre                                    5.96000000E+07 Conductor
Oro                                      4.52000000E+07 Conductor
Aluminio                                 3.77000000E+07 Conductor
Grafeno                                  1.00000000E+08 Conductor
Silicio (dopado n)                       1.56000000E+03 Semiconductor
Germanio (dopado p)                      2.17000000E+03 Semiconductor
Arseniuro de Galio                       8.50000000E+03 Semiconductor
Silicio (intrínseco)                     4.35000000E-04 Semiconductor
Diamante                                 1.00000000E-13 Aislante
Cuarzo                                   1.00000000E-17 Aislante
Teflón                                   1.00000000E-23 Aislante

12 filas seleccionadas.

Elapsed: 00:00:00.11
```

**Ejemplo: Conteo Condicional (Técnica Pivot)**

```sql
-- Contar simulaciones exitosas vs fallidas por software
SELECT 
    software,
    COUNT(*) AS total_simulaciones,
    SUM(CASE WHEN convergencia = 'SI' THEN 1 ELSE 0 END) AS exitosas,
    SUM(CASE WHEN convergencia = 'NO' THEN 1 ELSE 0 END) AS fallidas,
    ROUND(
        SUM(CASE WHEN convergencia = 'SI' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 
        1
    ) AS "% Éxito"
FROM SIMULACIONES_DFT
GROUP BY software
ORDER BY COUNT(*) DESC;
```

**Salida:**
```
SOFTWARE                 TOTAL_SIMULACIONES   EXITOSAS   FALLIDAS  % Éxito
------------------------ ------------------ ---------- ---------- --------
VASP                                    234        218         16     93.2
Quantum ESPRESSO                        187        172         15     92.0
CASTEP                                  145        138          7     95.2
ABINIT                                   89         81          8     91.0
Wien2k                                   67         59          8     88.1
SIESTA                                   54         48          6     88.9

6 filas seleccionadas.

Elapsed: 00:00:00.14

-- Análisis: CASTEP tiene la mejor tasa de convergencia (95.2%)
```

---

## 6. Consultas Avanzadas (Subqueries, CTEs)

### 6.1 Subqueries en WHERE

**Problema:** "Materiales con módulo de Young superior al promedio"

```sql
SELECT nombre_material, modulo_young
FROM MATERIALES m
JOIN PROPIEDADES_MECANICAS pm ON m.id_material = pm.id_material
WHERE pm.modulo_young > (
    SELECT AVG(modulo_young) FROM PROPIEDADES_MECANICAS
)
ORDER BY pm.modulo_young DESC;
```

### 6.2 Subqueries en SELECT

```sql
-- Comparar cada material con el máximo global
SELECT 
    m.nombre_material,
    pm.modulo_young,
    (SELECT MAX(modulo_young) FROM PROPIEDADES_MECANICAS) AS max_global,
    pm.modulo_young - (SELECT MAX(modulo_young) FROM PROPIEDADES_MECANICAS) AS diferencia
FROM MATERIALES m
JOIN PROPIEDADES_MECANICAS pm ON m.id_material = pm.id_material;
```

### 6.3 Common Table Expressions (CTEs)

Los CTEs hacen el código más legible y reutilizable.

**Sintaxis:**
```sql
WITH nombre_cte AS (
    -- Subconsulta
)
SELECT * FROM nombre_cte;
```

**Ejemplo Práctico:**

```sql
-- Paso 1: CTE con estadísticas base
WITH stats_mecanicas AS (
    SELECT 
        m.sistema_cristalino,
        AVG(pm.modulo_young) AS e_promedio,
        STDDEV(pm.modulo_young) AS e_desviacion
    FROM MATERIALES m
    JOIN PROPIEDADES_MECANICAS pm ON m.id_material = pm.id_material
    GROUP BY m.sistema_cristalino
)
-- Paso 2: Usar el CTE
SELECT 
    sistema_cristalino,
    ROUND(e_promedio, 2) AS "E_avg (GPa)",
    ROUND(e_desviacion, 2) AS "σ (GPa)",
    ROUND(e_desviacion / e_promedio * 100, 1) AS "CV (%)"
FROM stats_mecanicas
WHERE e_promedio > 100
ORDER BY e_promedio DESC;
```

**Salida:**
```
SISTEMA_CRISTALINO       E_avg (GPa)   σ (GPa)   CV (%)
------------------------ ------------ --------- --------
Cúbico                         425.67    198.34     46.6
Hexagonal                      387.23    156.89     40.5
Trigonal                       312.56    145.32     46.5
Tetragonal                     298.45    134.21     45.0
Ortorrómbico                   245.12     98.76     40.3
Monoclínico                    189.34     67.45     35.6
Triclínico                     167.89     54.32     32.4
Amorfo                         156.23     89.12     57.0

8 filas seleccionadas.

Elapsed: 00:00:00.21

-- Observación: Los materiales amorfos tienen mayor variabilidad (CV=57%)
```

### 6.4 CTEs Múltiples

```sql
WITH 
-- CTE 1: Propiedades mecánicas promedio
mech_avg AS (
    SELECT id_material, AVG(modulo_young) AS e_avg
    FROM PROPIEDADES_MECANICAS
    GROUP BY id_material
),
-- CTE 2: Propiedades térmicas promedio
therm_avg AS (
    SELECT id_material, AVG(conductividad) AS k_avg
    FROM PROPIEDADES_TERMICAS
    GROUP BY id_material
)
-- Consulta final
SELECT 
    m.nombre_material,
    ma.e_avg AS "E (GPa)",
    ta.k_avg AS "k (W/mK)"
FROM MATERIALES m
JOIN mech_avg ma ON m.id_material = ma.id_material
JOIN therm_avg ta ON m.id_material = ta.id_material
WHERE ma.e_avg > 200 AND ta.k_avg < 50;
```

---

## 7. Funciones Analíticas (Window Functions)

Las funciones ventana permiten hacer cálculos **sin colapsar las filas** como hace GROUP BY.

### 7.1 Ranking

**ROW_NUMBER:** Numeración única secuencial.

```sql
SELECT 
    nombre_material,
    modulo_young,
    ROW_NUMBER() OVER (ORDER BY modulo_young DESC) AS ranking
FROM MATERIALES m
JOIN PROPIEDADES_MECANICAS pm ON m.id_material = pm.id_material;
```

**Salida:**
```
NOMBRE_MATERIAL                          MODULO_YOUNG    RANKING
---------------------------------------- ------------ ----------
Diamante                                      1050.00          1
Carburo de Tungsteno                           700.00          2
Carburo de Boro                                460.00          3
Carburo de Silicio                             450.00          4
Óxido de Aluminio (Alúmina)                    380.00          5
Nitruro de Silicio                             310.00          6
Óxido de Magnesio                              250.00          7
Hematita                                       220.00          8
Acero AISI 4340                                210.00          9
Dióxido de Circonio                            210.00         10

10 filas seleccionadas.
```

**RANK:** Permite empates (saltos en numeración).

```sql
SELECT 
    nombre_material,
    densidad,
    RANK() OVER (ORDER BY densidad DESC) AS ranking
FROM MATERIALES
WHERE densidad IS NOT NULL;
```

**Salida (mostrando empates):**
```
NOMBRE_MATERIAL                          DENSIDAD    RANKING
---------------------------------------- -------- ----------
Osmio                                      22.590          1
Iridio                                     22.560          2
Platino                                    21.450          3
Renio                                      21.020          4
Plutonio                                   19.840          5
Oro                                        19.320          6
Wolframio                                  19.250          7
Uranio                                     19.050          8
Tántalo                                    16.690          9
Tungsteno-Carburo                          15.630         10
Mercurio                                   13.534         11
Rodio                                      12.410         12

12 filas seleccionadas.
```

**DENSE_RANK:** Sin saltos en numeración.

```sql
SELECT 
    nombre_material,
    sistema_cristalino,
    modulo_young,
    DENSE_RANK() OVER (
        PARTITION BY sistema_cristalino 
        ORDER BY modulo_young DESC
    ) AS ranking_por_sistema
FROM MATERIALES m
JOIN PROPIEDADES_MECANICAS pm ON m.id_material = pm.id_material;
```

**Salida:**
```
NOMBRE_MATERIAL                          SISTEMA_CRISTALINO   MODULO_YOUNG RANKING_POR_SISTEMA
---------------------------------------- -------------------- ------------ -------------------
Diamante                                 Cúbico                    1050.00                   1
Carburo de Tungsteno                     Cúbico                     700.00                   2
Acero AISI 4340                          Cúbico                     210.00                   3
Acero inoxidable 316L                    Cúbico                     193.00                   4
Níquel                                   Cúbico                     200.00                   5
...

Carburo de Boro                          Hexagonal                  460.00                   1
Carburo de Silicio                       Hexagonal                  450.00                   2
Óxido de Aluminio (Alúmina)              Hexagonal                  380.00                   3
Nitruro de Silicio                       Hexagonal                  310.00                   4
Cobalto                                  Hexagonal                  209.00                   5
...

Dióxido de Circonio                      Tetragonal                 210.00                   1
Titanato de Bario                        Tetragonal                 145.00                   2
Dióxido de Titanio (Rutilo)              Tetragonal                 283.00                   3

18 filas seleccionadas.

-- Nota: Cada sistema cristalino tiene su propio ranking independiente
```

### 7.2 Running Aggregates

```sql
-- Suma acumulativa de costos de experimentos
SELECT 
    fecha_experimento,
    descripcion,
    costo,
    SUM(costo) OVER (ORDER BY fecha_experimento) AS costo_acumulado
FROM EXPERIMENTOS_LAB
ORDER BY fecha_experimento;
```

### 7.3 Comparación con Valores Previos/Siguientes

**LAG:** Valor de la fila anterior.  
**LEAD:** Valor de la fila siguiente.

```sql
-- Cambio de energía de formación entre simulaciones
SELECT 
    nombre_material,
    fecha_calculo,
    energia_formacion,
    LAG(energia_formacion) OVER (
        PARTITION BY id_material 
        ORDER BY fecha_calculo
    ) AS energia_anterior,
    energia_formacion - LAG(energia_formacion) OVER (
        PARTITION BY id_material 
        ORDER BY fecha_calculo
    ) AS delta_energia
FROM MATERIALES m
JOIN SIMULACIONES_DFT s ON m.id_material = s.id_material;
```

**Salida:**
```
NOMBRE_MATERIAL          FECHA_CAL ENERGIA_FORMACION ENERGIA_ANTERIOR DELTA_ENERGIA
------------------------ --------- ----------------- ---------------- -------------
Silicio                  12-ENE-26            -5.423                                
Silicio                  15-ENE-26            -5.431           -5.423        -0.008
Silicio                  18-ENE-26            -5.429           -5.431         0.002
Silicio                  22-ENE-26            -5.430           -5.429        -0.001

Germanio                 10-ENE-26            -3.851                                
Germanio                 14-ENE-26            -3.847           -3.851         0.004
Germanio                 19-ENE-26            -3.849           -3.847        -0.002

Diamante                 08-ENE-26            -7.365                                
Diamante                 11-ENE-26            -7.368           -7.365        -0.003
Diamante                 16-ENE-26            -7.367           -7.368         0.001

11 filas seleccionadas.

-- Observación: Las energías convergen con diferencias < 0.01 eV
```

### 7.4 Percentiles

```sql
-- Identificar materiales en el top 10% de dureza
SELECT 
    nombre_material,
    dureza,
    NTILE(10) OVER (ORDER BY dureza DESC) AS decil
FROM MATERIALES m
JOIN PROPIEDADES_MECANICAS pm ON m.id_material = pm.id_material
WHERE dureza IS NOT NULL
  AND NTILE(10) OVER (ORDER BY dureza DESC) = 1;  -- Top 10%
```

---

## 8. Optimización y Performance

### 8.1 Uso de EXPLAIN PLAN

```sql
-- Activar el plan de ejecución
EXPLAIN PLAN FOR
SELECT m.nombre_material, pm.modulo_young
FROM MATERIALES m
JOIN PROPIEDADES_MECANICAS pm ON m.id_material = pm.id_material
WHERE m.sistema_cristalino = 'Cúbico';

-- Ver el plan
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

### 8.2 Índices

```sql
-- Crear índice en columna frecuentemente buscada
CREATE INDEX idx_sistema_cristalino ON MATERIALES(sistema_cristalino);

-- Índice compuesto
CREATE INDEX idx_material_prop ON PROPIEDADES_MECANICAS(id_material, modulo_young);

-- Ver índices existentes
SELECT index_name, table_name, uniqueness
FROM user_indexes
ORDER BY table_name;
```

### 8.3 Buenas Prácticas

```sql
-- ❌ MAL: SELECT *
SELECT * FROM MATERIALES;

-- ✅ BIEN: Solo columnas necesarias
SELECT nombre_material, formula_quimica FROM MATERIALES;

-- ❌ MAL: OR en grandes tablas
SELECT * FROM MATERIALES 
WHERE sistema_cristalino = 'Cúbico' OR sistema_cristalino = 'Hexagonal';

-- ✅ BIEN: IN
SELECT * FROM MATERIALES 
WHERE sistema_cristalino IN ('Cúbico', 'Hexagonal');
```

---

## 9. Automatización y Persistencia

### 9.1 Archivo de Configuración Global (`login.sql`)

Crea un archivo `login.sql` en tu directorio de trabajo:

```sql
-- login.sql
-- Configuración automática al iniciar SQL*Plus

-- Formato visual
SET LINESIZE 300
SET PAGESIZE 100
SET TIMING ON
SET TRIMSPOOL ON
SET TAB OFF

-- Formato de columnas comunes
COLUMN nombre_material FORMAT A40
COLUMN formula_quimica FORMAT A20
COLUMN sistema_cristalino FORMAT A15

-- Prompt personalizado
SET SQLPROMPT "_USER@_CONNECT_IDENTIFIER> "

-- Mensaje de bienvenida
PROMPT
PROMPT ========================================
PROMPT  Sistema de Gestión de Materiales
PROMPT  Versión 2.0 - Enero 2026
PROMPT ========================================
PROMPT
```

### 9.2 Scripts Reutilizables

**Ejemplo 1: `reporte_top_materiales.sql`**

```sql
-- reporte_top_materiales.sql
-- Top 20 materiales por módulo de Young

-- Formato específico
COLUMN ranking FORMAT 999
COLUMN "E (GPa)" FORMAT 9999.99
COLUMN "Densidad (g/cm³)" FORMAT 9999.99

-- Consulta
SELECT 
    RANK() OVER (ORDER BY pm.modulo_young DESC) AS ranking,
    m.nombre_material,
    m.sistema_cristalino,
    pm.modulo_young AS "E (GPa)",
    m.densidad AS "Densidad (g/cm³)"
FROM MATERIALES m
JOIN PROPIEDADES_MECANICAS pm ON m.id_material = pm.id_material
WHERE pm.modulo_young IS NOT NULL
FETCH FIRST 20 ROWS ONLY;
```

**Ejemplo 2: `estadisticas_simulaciones.sql`**

```sql
-- estadisticas_simulaciones.sql
-- Resumen de simulaciones DFT por software

COLUMN software FORMAT A20
COLUMN total FORMAT 9999
COLUMN "% Éxito" FORMAT 999.9

SELECT 
    software,
    COUNT(*) AS total,
    SUM(CASE WHEN convergencia = 'SI' THEN 1 ELSE 0 END) AS exitosas,
    SUM(CASE WHEN convergencia = 'NO' THEN 1 ELSE 0 END) AS fallidas,
    ROUND(
        SUM(CASE WHEN convergencia = 'SI' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 
        1
    ) AS "% Éxito"
FROM SIMULACIONES_DFT
GROUP BY software
ORDER BY COUNT(*) DESC;
```

### 9.3 Ejecución de Scripts

```sql
-- Ejecutar script
@reporte_top_materiales

-- Con ruta completa
@/home/usuario/scripts/estadisticas_simulaciones.sql

-- Ver contenido de un script sin ejecutar
GET reporte_top_materiales.sql
```

### 9.4 Generación de Reportes (SPOOL)

```sql
-- Iniciar captura a archivo
SPOOL reporte_materiales.txt

-- Ejecutar consultas
@reporte_top_materiales
@estadisticas_simulaciones

-- Detener captura
SPOOL OFF
```

### 9.5 Variables de Sustitución

```sql
-- Definir variable
DEFINE material_id = 42

-- Usar variable
SELECT * FROM MATERIALES WHERE id_material = &material_id;

-- Solicitar al usuario
ACCEPT material_nombre PROMPT 'Ingrese nombre del material: '
SELECT * FROM MATERIALES WHERE nombre_material LIKE '%&material_nombre%';
```

---

## 10. Casos de Uso Avanzados

### 10.1 Análisis de Correlación Estructura-Propiedad

**Pregunta:** "¿Existe correlación entre sistema cristalino y conductividad térmica?"

```sql
WITH stats_por_sistema AS (
    SELECT 
        m.sistema_cristalino,
        COUNT(*) AS n_muestras,
        AVG(pt.conductividad) AS k_promedio,
        STDDEV(pt.conductividad) AS k_desviacion,
        MIN(pt.conductividad) AS k_min,
        MAX(pt.conductividad) AS k_max
    FROM MATERIALES m
    JOIN PROPIEDADES_TERMICAS pt ON m.id_material = pt.id_material
    WHERE pt.conductividad IS NOT NULL
    GROUP BY m.sistema_cristalino
)
SELECT 
    sistema_cristalino,
    n_muestras,
    ROUND(k_promedio, 2) AS "k_avg (W/mK)",
    ROUND(k_desviacion, 2) AS "σ",
    ROUND(k_min, 2) AS "k_min",
    ROUND(k_max, 2) AS "k_max",
    ROUND(k_desviacion / k_promedio * 100, 1) AS "CV (%)"
FROM stats_por_sistema
WHERE n_muestras >= 5  -- Solo sistemas con datos suficientes
ORDER BY k_promedio DESC;
```

**Salida:**
```
SISTEMA_CRISTALINO       N_MUESTRAS k_avg (W/mK)     σ   k_min   k_max  CV (%)
------------------------ ---------- ------------ ------ ------- ------- -------
Cúbico                           67       245.78  98.45   15.30  401.00    40.1
Hexagonal                        54       187.34  76.23   12.80  350.00    40.7
Tetragonal                       23       124.56  54.32    8.90  210.00    43.6
Trigonal                         12        98.23  45.67    5.40  180.00    46.5
Ortorrómbico                     18        89.45  38.21    4.20  156.00    42.7
Monoclínico                       8        67.89  29.34    3.10  120.00    43.2
Amorfo                           15        45.67  32.11    1.50   95.00    70.3

7 filas seleccionadas.

Elapsed: 00:00:00.28

-- Hallazgos:
-- 1. Los materiales cúbicos tienen mayor conductividad térmica promedio
-- 2. La alta simetría (cúbico, hexagonal) correlaciona con mejor conducción
-- 3. Materiales amorfos muestran alta dispersión (CV=70.3%)
```

### 10.2 Identificación de Outliers (Valores Atípicos)

Método: Rango Intercuartílico (IQR)

```sql
WITH quartiles AS (
    SELECT 
        PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY modulo_young) AS Q1,
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY modulo_young) AS Q3
    FROM PROPIEDADES_MECANICAS
),
iqr_bounds AS (
    SELECT 
        Q1,
        Q3,
        Q3 - Q1 AS IQR,
        Q1 - 1.5 * (Q3 - Q1) AS limite_inferior,
        Q3 + 1.5 * (Q3 - Q1) AS limite_superior
    FROM quartiles
)
SELECT 
    m.nombre_material,
    pm.modulo_young AS "E (GPa)",
    b.Q1,
    b.Q3,
    b.limite_inferior,
    b.limite_superior,
    CASE 
        WHEN pm.modulo_young < b.limite_inferior THEN 'Outlier Bajo'
        WHEN pm.modulo_young > b.limite_superior THEN 'Outlier Alto'
        ELSE 'Normal'
    END AS clasificacion
FROM MATERIALES m
JOIN PROPIEDADES_MECANICAS pm ON m.id_material = pm.id_material
CROSS JOIN iqr_bounds b
WHERE pm.modulo_young < b.limite_inferior 
   OR pm.modulo_young > b.limite_superior
ORDER BY pm.modulo_young;
```

### 10.3 Tracking de Experimentos (Trazabilidad)

```sql
-- Historia completa de un material: Simulaciones + Experimentos
WITH historia_material AS (
    SELECT 
        s.id_material,
        'Simulación DFT' AS tipo,
        s.fecha_calculo AS fecha,
        s.software AS detalle,
        s.energia_formacion AS valor_principal
    FROM SIMULACIONES_DFT s
    
    UNION ALL
    
    SELECT 
        e.id_material,
        'Experimento Lab' AS tipo,
        e.fecha_experimento AS fecha,
        e.tipo_experimento AS detalle,
        e.resultado AS valor_principal
    FROM EXPERIMENTOS_LAB e
)
SELECT 
    m.nombre_material,
    h.tipo,
    h.fecha,
    h.detalle,
    h.valor_principal
FROM MATERIALES m
JOIN historia_material h ON m.id_material = h.id_material
WHERE m.nombre_material = 'Grafeno'
ORDER BY h.fecha DESC;
```

**Salida:**
```
NOMBRE_MATERIAL TIPO              FECHA     DETALLE                  VALOR_PRINCIPAL
--------------- ----------------- --------- ------------------------ ---------------
Grafeno         Experimento Lab   22-ENE-26 Raman Spectroscopy                  2680
Grafeno         Simulación DFT    20-ENE-26 VASP                              -9.234
Grafeno         Experimento Lab   18-ENE-26 AFM Thickness Measure              0.335
Grafeno         Simulación DFT    15-ENE-26 Quantum ESPRESSO                  -9.231
Grafeno         Experimento Lab   12-ENE-26 Resistividad eléctrica         1.00E-06
Grafeno         Simulación DFT    10-ENE-26 VASP                              -9.228
Grafeno         Experimento Lab   08-ENE-26 XRD Pattern Analysis               2.464
Grafeno         Simulación DFT    05-ENE-26 CASTEP                            -9.235

8 filas seleccionadas.

-- Trazabilidad completa: Alternancia entre simulaciones y validaciones experimentales
```

### 10.4 Comparación de Métodos Computacionales

```sql
-- Comparar energías de formación: DFT vs Semi-empírico
SELECT 
    m.nombre_material,
    MAX(CASE WHEN s.tipo_calculo = 'DFT' THEN s.energia_formacion END) AS E_DFT,
    MAX(CASE WHEN s.tipo_calculo = 'Semi-empirico' THEN s.energia_formacion END) AS E_SE,
    ABS(
        MAX(CASE WHEN s.tipo_calculo = 'DFT' THEN s.energia_formacion END) -
        MAX(CASE WHEN s.tipo_calculo = 'Semi-empirico' THEN s.energia_formacion END)
    ) AS diferencia_absoluta
FROM MATERIALES m
JOIN SIMULACIONES_DFT s ON m.id_material = s.id_material
WHERE s.tipo_calculo IN ('DFT', 'Semi-empirico')
GROUP BY m.nombre_material
HAVING COUNT(DISTINCT s.tipo_calculo) = 2  -- Ambos métodos presentes
ORDER BY diferencia_absoluta DESC;
```

### 10.5 Reporte Ejecutivo Final

```sql
-- Dashboard completo: Vista 360° del proyecto
WITH 
-- Métricas generales
general_stats AS (
    SELECT 
        COUNT(DISTINCT m.id_material) AS total_materiales,
        COUNT(DISTINCT s.id_simulacion) AS total_simulaciones,
        COUNT(DISTINCT e.id_experimento) AS total_experimentos
    FROM MATERIALES m
    LEFT JOIN SIMULACIONES_DFT s ON m.id_material = s.id_material
    LEFT JOIN EXPERIMENTOS_LAB e ON m.id_material = e.id_material
),
-- Top 5 sistemas cristalinos
top_sistemas AS (
    SELECT sistema_cristalino, COUNT(*) AS cantidad
    FROM MATERIALES
    GROUP BY sistema_cristalino
    ORDER BY COUNT(*) DESC
    FETCH FIRST 5 ROWS ONLY
),
-- Éxito de simulaciones
sim_stats AS (
    SELECT 
        COUNT(*) AS total,
        SUM(CASE WHEN convergencia = 'SI' THEN 1 ELSE 0 END) AS exitosas
    FROM SIMULACIONES_DFT
)
SELECT 
    'Resumen del Proyecto de Materiales' AS reporte,
    (SELECT total_materiales FROM general_stats) AS materiales_registrados,
    (SELECT total_simulaciones FROM general_stats) AS simulaciones_realizadas,
    (SELECT total_experimentos FROM general_stats) AS experimentos_completados,
    ROUND((SELECT exitosas * 100.0 / total FROM sim_stats), 1) || '%' AS tasa_exito_sim
FROM DUAL;
```

**Salida:**
```
REPORTE                                  MATERIALES_REGISTRADOS SIMULACIONES_REALIZADAS EXPERIMENTOS_COMPLETADOS TASA_EXITO_SIM
---------------------------------------- ---------------------- ----------------------- ------------------------ --------------
Resumen del Proyecto de Materiales                          487                     776                      234 92.4%

1 fila seleccionada.

Elapsed: 00:00:00.34
```

**Vista Complementaria (Top Sistemas):**
```sql
SELECT * FROM top_sistemas;
```
```
SISTEMA_CRISTALINO       CANTIDAD
------------------------ --------
Cúbico                        145
Hexagonal                      89
Tetragonal                     56
Ortorrómbico                   34
Monoclínico                    28

5 filas seleccionadas.
```

**Análisis del Dashboard:**
- **Base de datos robusta:** 487 materiales caracterizados
- **Alto volumen computacional:** 776 simulaciones DFT ejecutadas
- **Validación experimental:** 234 experimentos de laboratorio
- **Excelente convergencia:** 92.4% de éxito en simulaciones
- **Enfoque estructural:** Predominancia de sistemas de alta simetría (Cúbico + Hexagonal = 48%)

---

## Glosario Técnico

| Término | Definición |
|---------|-----------|
| **DFT** | Density Functional Theory - Método computacional para calcular propiedades electrónicas |
| **Eg** | Energy gap / Band gap - Brecha energética entre bandas de valencia y conducción |
| **HV** | Hardness Vickers - Medida de dureza del material |
| **IQR** | Interquartile Range - Rango intercuartílico para detectar outliers |
| **CTE** | Common Table Expression - Subconsulta temporal nombrada |
| **SPOOL** | Comando para capturar salida de SQL*Plus a archivo |
| **Window Function** | Función analítica que opera sobre un conjunto de filas relacionadas |
| **PARTITION BY** | Divide el conjunto de resultados en grupos para funciones ventana |

---

## Recursos Adicionales

### Scripts de Ejemplo Completos

**setup_database.sql** - Crear esquema inicial:
```sql
-- Crear tabla de materiales
CREATE TABLE MATERIALES (
    id_material NUMBER(10) PRIMARY KEY,
    nombre_material VARCHAR2(100) NOT NULL,
    formula_quimica VARCHAR2(50),
    sistema_cristalino VARCHAR2(30),
    grupo_espacial VARCHAR2(20),
    densidad NUMBER(8,3),
    fecha_registro DATE DEFAULT SYSDATE
);

-- Crear secuencia para IDs
CREATE SEQUENCE seq_materiales START WITH 1 INCREMENT BY 1;

-- Crear tabla de propiedades mecánicas
CREATE TABLE PROPIEDADES_MECANICAS (
    id_propiedad NUMBER(10) PRIMARY KEY,
    id_material NUMBER(10) REFERENCES MATERIALES(id_material),
    modulo_young NUMBER(10,2),
    dureza NUMBER(10,2),
    tenacidad NUMBER(10,2),
    fecha_medicion DATE
);

-- Índices para performance
CREATE INDEX idx_mat_sistema ON MATERIALES(sistema_cristalino);
CREATE INDEX idx_prop_material ON PROPIEDADES_MECANICAS(id_material);
```

### Comandos Útiles de SQL*Plus

```sql
-- Ver configuración actual
SHOW ALL

-- Limpiar pantalla
CLEAR SCREEN

-- Editar última consulta
EDIT

-- Ejecutar última consulta
/

-- Historial de comandos
HISTORY

-- Guardar consulta actual en archivo
SAVE mi_consulta.sql

-- Ver descripciones de todas las tablas
SELECT table_name FROM user_tables;
DESC [nombre_tabla]

-- Cancelar ejecución
Ctrl+C
```

---

## Ejercicios Prácticos

### Nivel Básico
1. Listar todos los materiales del sistema cúbico con densidad mayor a 5 g/cm³
2. Encontrar el material con mayor módulo de Young
3. Contar cuántos materiales contienen oxígeno en su fórmula

### Nivel Intermedio
4. Calcular el promedio de conductividad térmica por sistema cristalino
5. Identificar materiales sin simulaciones DFT registradas
6. Top 10 de materiales más duros que sean semiconductores (0 < Eg < 4 eV)

### Nivel Avanzado
7. Encontrar materiales cuyo módulo de Young esté en el percentil 95
8. Comparar convergencia de simulaciones entre VASP y Quantum ESPRESSO
9. Crear un reporte que identifique materiales "candidatos prometedores" (alta dureza + baja densidad + conductor)

---

**Última actualización:** 23 de Enero de 2026  
**Autor:** Tutorial de Informática de Materiales  
**Versión:** 1.0
