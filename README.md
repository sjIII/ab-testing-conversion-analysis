# Auditoría Estadística y Análisis de Resultados: Prueba A/B para Sistema de Recomendaciones (`recommender_system_test`) 🛒🧪

## 🎯 1. Contexto y Problema de Negocio
Una tienda en línea internacional implementó un sistema de recomendaciones mejorado con el objetivo de incrementar las ventas en la región de la Unión Europea (UE). El equipo original lanzó el experimento controlado (Prueba A/B) pero abandonó el proyecto antes de consolidar y validar los resultados, dejando únicamente las especificaciones técnicas crudas y las bases de datos de eventos.

**El Desafío:** Como Analista de Datos, la tarea principal consiste en realizar un control de calidad exhaustivo de los datos (*Data Sanity Check*) para comprobar si el experimento se ejecutó correctamente según los parámetros del negocio, analizar el comportamiento del usuario en el embudo de conversión y evaluar estadísticamente si el nuevo sistema de recomendaciones (Grupo B) generó el impacto esperado frente al diseño actual (Grupo A).

---

## 🚀 2. Especificaciones Técnicas y Objetivos
### Parámetros del Experimento:
* **Nombre de la prueba:** `recommender_system_test`
* **Grupos de control:** Grupo А (Base/Control), Grupo B (Nuevo embudo de pago con sistema de recomendaciones mejorado).
* **Fecha de lanzamiento:** 7 de diciembre de 2020.
* **Fecha límite de registro de usuarios:** 21 de diciembre de 2020.
* **Fecha de finalización del experimento:** 1 de enero de 2021.
* **Audiencia objetivo:** 15% de los nuevos usuarios de la región de la UE (Tamaño previsto: 6,000 participantes).

### Resultados Esperados por el Negocio:
Dentro de los 14 días posteriores a la inscripción, los usuarios deberían mostrar una mejor conversión en cada etapa del embudo de compras (`product_page` → `product_cart` → `purchase`), esperando **al menos un 10% de aumento** en las tasas de conversión de cada paso en el Grupo B en comparación con el Grupo A.

---

## 📊 3. Datos y Metodología Analítica
El análisis se realizó integrando cuatro fuentes de datos transaccionales independientes:
* `ab_project_marketing_events_us.csv`: Calendario de campañas de marketing globales para 2020.
* `final_ab_new_users_upd_us.csv`: Registro de usuarios inscritos entre el 7 y 21 de diciembre de 2020.
* `final_ab_events_upd_us.csv`: Historial de eventos de los usuarios hasta el 1 de enero de 2021.
* `final_ab_participants_upd_us.csv`: Tabla de asignación de usuarios a los grupos del experimento.

### Fases del Estudio:
1. **Preparación y Saneamiento de Datos:** Conversión de tipos de datos (marcas de tiempo), manejo de valores ausentes y detección de anomalías.
2. **Análisis Exploratorio de Datos (EDA):** 
   * Evaluación de la distribución de eventos por usuario entre ambos grupos.
   * Análisis de la distribución de interacciones a lo largo de los días.
   * Detección de usuarios duplicados (usuarios presentes en ambas muestras simultáneamente).
   * Identificación de interferencias externas (campañas de marketing concurrentes).
3. **Análisis del Embudo de Conversión:** Mapeo de la pérdida de usuarios en la ruta `login` ➔ `product_page` ➔ `product_cart` ➔ `purchase`.
4. **Evaluación Estadística:** Implementación de una **Prueba Z para Proporciones (Z-test)** con un nivel de significancia (α) de 0.05 para verificar si las diferencias en las tasas de conversión son estadísticamente significativas.

---

## 🛠️ 4. Herramientas y Decisiones Técnicas
* **Python (Pandas y NumPy):** Utilizados para la limpieza de datos, uniones avanzadas de tablas transaccionales por `user_id` y segmentación de la ventana de tiempo de 14 días de actividad por usuario.
* **SciPy (Stats):** Biblioteca matemática seleccionada para automatizar el cálculo del *Z-score* y el *p-value* en la prueba de proporciones de las etapas del embudo.
* **Matplotlib y Seaborn:** Utilizados para la visualización del embudo de conversión y las curvas de distribución de eventos diarios.

---
## 📈 5. Resultados y Hallazgos Principales (Basados en Datos Reales)

Tras procesar los datos de participación de la prueba `recommender_system_test`, se identificó una asimetría crítica en el volumen de usuarios asignados a las muestras: **Grupo A (2,747 usuarios únicos)** frente al **Grupo B (928 usuarios únicos)**.

Al analizar el embudo de conversión (`login` ➔ `product_page` ➔ `product_cart` ➔ `purchase`) y aplicar la Prueba Z para proporciones, se obtuvieron las siguientes métricas clave:

*   **Paso 1: login ➔ product_page (Página de Producto):**
    *   La conversión en el **Grupo A fue del 64.80%** frente al **56.42% del Grupo B**.
    *   La diferencia fue de **8.38% a favor del grupo de control**.
    *   **Resultado de la Prueba Z:** El `p-value` obtenido fue de **0.000005**, indicando que la caída en la conversión del Grupo B es **estadísticamente significativa**. El nuevo sistema de recomendaciones empeoró el paso inicial del embudo.
*   **Paso 2: product_page ➔ product_cart (Carrito de Compras):**
    *   La conversión del **Grupo A fue del 46.29%** frente al **48.76% del Grupo B**.
    *   **Resultado de la Prueba Z:** El `p-value` fue de **0.320620**. Al ser significativamente mayor a 0.05, se concluye que **no existe una diferencia estadística real** en este paso; la variación se debió al azar.
*   **Anomalía Crítica Detectada en el Paso Final (purchase):**
    *   El análisis del dataset arrojó métricas inconsistentes en la etapa de compra, donde el Grupo A mostró una tasa de conversión superior al 100% (**105.83%**) y el Grupo B un **100.39%**. 
    *   Esto representa un **error de telemetría o contaminación de datos** (usuarios ejecutando compras sin pasar por el flujo lineal registrado o inconsistencias en las marcas de tiempo entre tablas), lo que invalida la confiabilidad estadística del evento de compra final y provocó un valor `NaN` en la prueba estadística.

---

## 💡 6. Conclusiones y Recomendaciones Estratégicas

### Conclusión General:
**El experimento no fue exitoso.** El sistema de recomendaciones mejorado implementado en el Grupo B no solo no alcanzó el incremento del 10% esperado por el negocio en cada etapa, sino que provocó una **pérdida estadísticamente significativa del 8.38% en la transición a la página del producto** en comparación con el diseño actual.

### Recomendaciones para el Equipo de Producto y Negocio:
1. **Rechazar el Despliegue del Grupo B:** Se recomienda suspender inmediatamente la implementación del nuevo sistema de recomendaciones en el embudo de pago para la región de la UE, manteniendo el diseño actual (Grupo A).
2. **Auditoría Urgente al Sistema de Registro de Eventos:** El hallazgo de tasas de conversión superiores al 100% en el evento `purchase` evidencia una falla grave en el tracking o en la extracción de datos. Es mandatorio que el equipo de Ingeniería de Software revise la lógica de disparo de eventos de compra antes de lanzar una nueva prueba.
3. **Optimización del Criterio de Asignación (A/B Splitter):** El experimento sufrió una fuerte asimetría de volumen (2,747 vs 928). Los futuros experimentos deben contar con un sistema de distribución balanceado para mitigar el sesgo por tamaño de muestra.


## 📂 7. Instrucciones para la Ejecución del Proyecto

### Estructura de Archivos:
```text
├── datasets/           # Contiene los archivos CSV del proyecto
├── notebooks/          # Jupyter Notebook con el código completo documentado (EDA y pruebas Z)
├── requirements.txt    # Dependencias necesarias para ejecutar el entorno
└── README.md           # Reporte ejecutivo del experimento
```


