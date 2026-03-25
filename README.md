# 🛒 Proyecto ETL: Instacart Market Basket Analysis

![PySpark](https://img.shields.io/badge/PySpark-3.x-orange.svg)
![Databricks](https://img.shields.io/badge/Databricks-Azure-blue.svg)
![Architecture](https://img.shields.io/badge/Architecture-Medallion-gold.svg)

## 📖 Descripción General
Este repositorio contiene el pipeline de datos (ETL/ELT) desarrollado en **PySpark** para procesar los conjuntos de datos transaccionales y de catálogo de Instacart. 

El proyecto implementa una **Arquitectura Medallón (Medallion Architecture)** orientada a construir un Data Lakehouse robusto. El objetivo es ingerir los datos crudos, limpiarlos, modelarlos y generar Data Marts listos para el consumo de los equipos de Business Intelligence (BI) y Data Science.

---

## 🏗️ Arquitectura de Datos (Flujo ETL)

El proceso de transformación está dividido en tres capas lógicas progresivas:

### 🥉 Capa Bronze (Ingesta / Raw Data)
**Objetivo:** Mantener un historial inmutable de los datos crudos tal como llegan del origen.
* **Fuentes:** `orders.csv`, `order_products__prior.csv`, `products.csv`, `aisles.csv`, `departments.csv`.
* **Proceso:** Los datos se leen y se escriben en formato Parquet/Delta sin alteraciones lógicas.
* **Metadatos:** Se agrega automáticamente la columna `ingestion_date` (timestamp) para auditoría y trazabilidad.

### 🥈 Capa Silver (Datos Limpios y Conformados)
**Objetivo:** Filtrar, limpiar, estandarizar y modelar los datos para facilitar su cruce.
* **Manejo de Nulos:** En `fact_orders`, se imputan los valores nulos de `days_since_prior_order` por `0.0` y se crea un flag booleano `is_first_order`.
* **Estandarización de Textos:** Funciones `TRIM()` y `LOWER()` aplicadas a nombres de productos, pasillos y departamentos para evitar duplicidades tipográficas.
* **Modelado Dimensional:** Denormalización cruzando (`JOIN`) `products`, `aisles` y `departments` para crear una única tabla de dimensión robusta: `dim_products`.

### 🥇 Capa Gold (Datos de Negocio / Data Marts)
**Objetivo:** Entregar datos agregados, pre-calculados y optimizados para tableros de control y modelos predictivos.
* **`dm_user_profiling`:** Perfilado de clientes (Customer 360) con total de pedidos y frecuencia promedio de compra.
* **`dm_department_performance`:** Volumen de ventas agrupado por departamento (Category Management).
* **`dm_top_products`:** Ranking histórico de los productos más vendidos en la plataforma.

---

## 🌍 Ambientes de Despliegue

El ciclo de vida del desarrollo y la ejecución de este pipeline se gestiona a través de dos entornos (Workspaces) separados en Azure Databricks:

### 🛠️ Desarrollo (DEV) - `adbpryargd01`
* **Propósito:** Creación de nuevos features, pruebas unitarias, exploración de datos (EDA) y validación de transformaciones.
* **Datos:** Utiliza un subconjunto de datos (sample data) para agilizar el procesamiento y reducir costos de cómputo.
* **Acceso:** Data Engineers y desarrolladores del proyecto.

### 🚀 Producción (PROD) - `adbproyargd01`
* **Propósito:** Ejecución automatizada y programada (Jobs/Workflows) del pipeline validado.
* **Datos:** Carga el volumen total de la información hacia las capas Gold consumibles por el negocio.
* **Acceso:** Estrictamente controlado. Los despliegues se realizan únicamente a través de procesos de integración y despliegue continuo (CI/CD) como Azure DevOps o GitHub Actions.

---

## ⚙️ Estructura del Repositorio

├── data/                   # (Solo en local) Muestras de archivos CSV
├── notebooks/              # Notebooks exploratorios
├── src/                    # Scripts de código fuente (PySpark)
│   ├── bronze_ingestion.py
│   ├── silver_transform.py
│   └── gold_aggregations.py
└── README.md               # Documentación del proyecto