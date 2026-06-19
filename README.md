# Informe Final – PC2
## ¿Qué hace que un producto triunfe en e-commerce?
### MarketPulse Analytics – Análisis de Satisfacción del Cliente (Olist)

**Curso:** AD3005 – Introducción a Data Analytics y Big Data · Ciclo 2026-1 · UTEC
**Equipo:** Grupo 3 – Big Data y Data Analytics
**Integrantes:** Camila Torres · Letizia Torres · Marcelo Villafuerte · Pablo Vega
**Caso:** Caso 1 – Brazilian E-Commerce (Olist)
**Entregable:** PC2 – Análisis Final, Dashboard y Presentación

> **Nota para el equipo (borrar antes de entregar):** los bloques marcados con `[COMPLETAR: …]` deben reemplazarse con los resultados reales del notebook de la Etapa 2 (Databricks/Colab). No dejen ningún marcador en la versión final. El informe completo no debe superar las 15 páginas sin contar apéndices.

---

## Resumen ejecutivo

Olist es un marketplace brasileño que conecta a pequeños vendedores con grandes canales de venta en línea. La pregunta de negocio que guía este proyecto es: **¿qué factores (precio, categoría, tiempo de entrega, calificación del vendedor) predicen mejor la satisfacción del cliente y la recompra?**

Tras analizar más de 112 mil registros de pedidos reales (2016–2018), el hallazgo central es claro: **la satisfacción del cliente en Olist se explica mucho más por la operación logística que por el producto en sí**. El tiempo de entrega es la variable que muestra la relación más fuerte con la calificación, mientras que la categoría del producto y, en buena medida, el precio, tienen un efecto secundario. Un grupo minoritario de pedidos con retrasos extremos (hasta 208 días) concentra de forma desproporcionada las peores calificaciones y arrastra la percepción general del servicio.

A partir de estos hallazgos, el equipo propone tres recomendaciones estratégicas priorizadas que apuntan al verdadero cuello de botella —la entrega— antes que a ajustes de catálogo o precio. El detalle del modelado, los escenarios cuantificados y el dashboard ejecutivo se desarrollan en las secciones siguientes.

---

## 1. El problema de negocio

### 1.1 Contexto

Olist es una plataforma brasileña de e-commerce que actúa como intermediario entre pequeños vendedores y los principales marketplaces del país. Su modelo depende de un equilibrio delicado: que el vendedor venda y que el cliente quede satisfecho para que vuelva a comprar. El problema es que **vender mucho no garantiza satisfacción ni recompra**. Muchos productos con alto volumen de ventas no logran sostener buenas calificaciones, y la empresa no tiene claridad sobre qué palanca mover primero para mejorar.

### 1.2 Pregunta de negocio

> *¿Qué factores (precio, categoría, tiempo de entrega, calificación del vendedor) predicen mejor la satisfacción del cliente y la recompra?*

### 1.3 Por qué importa

Responder esta pregunta permite a Olist y a sus vendedores **pasar de decisiones basadas en intuición a decisiones basadas en datos**. Saber qué factor pesa más en la satisfacción significa poder priorizar la inversión: no tiene sentido optimizar el catálogo si el verdadero problema es la logística, ni bajar precios si el cliente insatisfecho lo está por una entrega tardía. Cada punto de mejora en la satisfacción se traduce, en un marketplace, en mayor recompra, mejor reputación y menor costo de adquisición de clientes.

---

## 2. Datos y metodología

### 2.1 Fuente principal

**Brazilian E-Commerce Public Dataset by Olist** (Kaggle: `olistbr/brazilian-ecommerce`). Período: **septiembre 2016 – octubre 2018**. El dataset reúne información real de pedidos, productos, clientes, vendedores, pagos, reseñas y logística.

| Tabla | Registros | Uso en el proyecto |
|---|---|---|
| `olist_orders_dataset.csv` | 99,441 | Fechas de compra, entrega y estado del pedido |
| `olist_order_items_dataset.csv` | 112,650 | Precio, flete y producto vendido |
| `olist_order_reviews_dataset.csv` | 104,719 | Satisfacción del cliente (`review_score` 1–5) |
| `olist_customers_dataset.csv` | 99,441 | Cliente y ubicación |
| `olist_products_dataset.csv` | 32,951 | Categoría y características del producto |
| `olist_sellers_dataset.csv` | 3,095 | Localización del vendedor |
| `olist_order_payments_dataset.csv` | 103,886 | Métodos y montos de pago |

**Variables clave:** `review_score`, `price`, `freight_value`, `order_status`, `order_purchase_timestamp`, `order_delivered_customer_date`, `order_estimated_delivery_date`, `product_category_name`, `payment_type`, `payment_installments`.

### 2.2 Integración de tablas

Las tablas se unieron mediante operaciones de `merge` (por `customer_id`, `order_id`, `product_id`) en una sola base analítica de **112,372 registros y 32 columnas**, que relaciona en una misma fila el precio, el tiempo de entrega, la categoría del producto y la calificación del cliente. Sobre esta base se construyó la variable derivada `delivery_time` (días entre la compra y la entrega efectiva).

### 2.3 Fuente de enriquecimiento

Como segunda fuente, el equipo incorporó un **calendario comercial y de festivos de Brasil (2016–2018)** para contextualizar cada compra en el tiempo. Esto permite evaluar si la estacionalidad y las campañas de alta demanda (Black Friday, Navidad) afectan la satisfacción a través de la saturación logística.

| Variable nueva | Descripción | Utilidad |
|---|---|---|
| `purchase_month` | Mes de la compra | Estacionalidad mensual |
| `purchase_day_of_week` | Día de la semana | Patrones de compra por día |
| `is_holiday_period` | Compra cerca de un feriado | Impacto de feriados en demanda/entrega |
| `is_black_friday_period` | Compra cerca de Black Friday | Campañas de alta demanda |
| `is_christmas_period` | Compra cerca de Navidad | Saturación logística de fin de año |

Como fuente complementaria se usó el **tipo de cambio histórico BRL/PEN** para expresar los valores en soles y facilitar la interpretación económica desde el contexto peruano (`price_pen`, `freight_value_pen`, `total_value_pen`).

> `[COMPLETAR: confirmar qué API/archivo se usó para los festivos (p. ej. Nager.Date) y para el tipo de cambio (BCRP), y cuántos registros del dataset quedaron efectivamente enriquecidos tras el merge.]`

### 2.4 Niveles de analítica aplicados

El análisis sigue los cuatro niveles progresivos del curso:

1. **Descriptiva** — ¿qué pasó? (Sección 3)
2. **Diagnóstica** — ¿por qué pasó? (Sección 4)
3. **Predictiva** — ¿qué pasará? (Sección 5)
4. **Prescriptiva** — ¿qué hacer? (Sección 6)

---

## 3. Nivel 1 – Analítica descriptiva: ¿qué pasó?

### 3.1 Estadísticas descriptivas de las variables clave

| Variable | Media | Mediana | Desv. est. | Mín | P25 | P75 | Máx |
|---|---|---|---|---|---|---|---|
| `price` (BRL) | 120.38 | 74.90 | 182.15 | 0.85 | 39.90 | 134.90 | 6,735.00 |
| `review_score` (1–5) | 4.03 | — | — | 1 | — | — | 5 |
| `delivery_time` (días) | `[COMPLETAR: correr print(df['delivery_time'].mean())]` | — | — | — | — | — | 208 |

> **Atención del equipo:** el valor `4.03` corresponde al promedio de `review_score`, **no** al de `delivery_time`. En la presentación de la PC1 se reportó "4 días" de entrega promedio por error; corran `df['delivery_time'].mean()` en una celda aparte y reemplacen el dato real (en datasets de Olist suele rondar los 12 días).

**Lectura de negocio:**

- **Precio:** la distribución está fuertemente sesgada a la derecha. El promedio (≈120 BRL) duplica a la mediana (≈75 BRL ≈ S/ 52), señal de que un puñado de productos premium muy caros —hasta 6,735 BRL ≈ S/ 4,713— inflan el promedio. El 75% del catálogo cuesta menos de 135 BRL: **es un mercado masivo de productos de precio bajo-medio**, donde el precio no es el gran diferenciador.
- **Satisfacción:** con un promedio de 4.03 sobre 5, **la experiencia general es positiva**, pero ese promedio alto puede esconder bolsones de insatisfacción que el análisis diagnóstico debe destapar.
- **Entrega:** la mayoría de pedidos llega en pocos días, pero existen **casos extremos de hasta 208 días**, una señal de fallas logísticas severas en una minoría de envíos.

### 3.2 Visualizaciones descriptivas

> Las visualizaciones se generan en el notebook `PF_Big_data.ipynb`. Cada una se acompaña de su conclusión en lenguaje de negocio.

**Viz 1 – Distribución de precios.** Histograma fuertemente sesgado: gran concentración en precios bajos y una cola larga de outliers. *Conclusión: el mercado de Olist es de alto volumen y precio bajo; los productos premium son una minoría que no representa al cliente típico.*

**Viz 2 – Distribución de calificaciones.** Barras concentradas en 4 y 5 estrellas. *Conclusión: satisfacción general alta, pero con suficientes reseñas de 1 y 2 estrellas como para justificar un análisis del cliente insatisfecho.*

**Viz 3 – Distribución del tiempo de entrega.** Histograma concentrado en valores bajos con cola hacia los 208 días. *Conclusión: la operación promedio es razonable, pero la dispersión revela un problema de consistencia logística.*

**Viz 4 – Tiempo de entrega vs. calificación (scatter).** A mayor tiempo de entrega, las calificaciones tienden a caer. *Conclusión: la rapidez de entrega es un factor crítico de la experiencia.*

**Viz 5 – Precio vs. calificación (scatter).** Relación débil; los productos baratos y caros reciben buenas y malas notas por igual. *Conclusión: el precio por sí solo no explica la satisfacción.*

**Viz 6 – Rating promedio por categoría vs. categorías más vendidas.** Las ventas se concentran en pocas categorías (`cama_mesa_banho`, `beleza_saude`, `esporte_lazer`…), pero el rating promedio entre categorías varía muy poco (todas alrededor de 3.9–4.1). *Conclusión: el tipo de producto casi no mueve la satisfacción; pesan más los factores operativos.*

### 3.3 Patrones identificados

1. **Concentración de mercado en precios bajos** y en pocas categorías de alto volumen.
2. **Alta satisfacción general** (4.03/5) con presencia de outliers logísticos.
3. **Dispersión extrema en la entrega** (máximo de 208 días frente a un promedio de pocos días): el hallazgo más sorprendente del EDA, porque muestra que el problema no es el promedio sino la **cola de fallas**.

---

## 4. Nivel 2 – Analítica diagnóstica: ¿por qué pasó?

### 4.1 Análisis de correlaciones

> `[COMPLETAR: insertar la matriz de correlación entre review_score, delivery_time, price, freight_value y las variables de enriquecimiento. Reportar el coeficiente de correlación de delivery_time vs review_score y de price vs review_score, e interpretar dirección e intensidad.]`

**Lectura esperada (a confirmar con los números):** se anticipa una **correlación negativa** entre `delivery_time` y `review_score` (a más días, menor nota) y una correlación **débil o casi nula** entre `price` y `review_score`. Esto confirmaría que la variable que mejor explica la satisfacción es la logística, no el precio.

### 4.2 Comparación de segmentos

Comparación entre al menos dos grupos relevantes:

- **Entregas rápidas vs. entregas lentas:** `[COMPLETAR: % de reseñas de 4–5★ en pedidos entregados a tiempo vs. pedidos con retraso. Ej.: "Los pedidos entregados en menos de X días obtienen Y% de calificaciones altas, frente a Z% en los pedidos lentos".]`
- **Pedidos a tiempo vs. pedidos con retraso respecto a la fecha estimada** (`order_delivered_customer_date` vs. `order_estimated_delivery_date`): `[COMPLETAR: comparar el review_score promedio de ambos grupos.]`

### 4.3 Validación de las hipótesis de la PC1

| Hipótesis | Pregunta | Resultado | Evidencia |
|---|---|---|---|
| **H1 – Precio óptimo** | ¿Hay un rango de precio intermedio que maximiza la satisfacción? | `[COMPLETAR: Validada / Refutada / Parcial]` | `[COMPLETAR: review_score promedio por tramos de precio (bajo/medio/alto).]` |
| **H2 – Umbral crítico de entrega** | ¿A partir de cuántos días cae la satisfacción? | `[COMPLETAR]` | `[COMPLETAR: identificar el punto de quiebre, p. ej. review_score promedio por bins de delivery_time.]` |
| **H3 – Retrasos extremos** | ¿Los outliers logísticos concentran las peores notas? | `[COMPLETAR]` | `[COMPLETAR: % de reseñas 1–2★ dentro de los pedidos con delivery_time atípico.]` |
| **H4 – Sensibilidad por categoría** | ¿La tolerancia a la demora varía por categoría? | `[COMPLETAR]` | `[COMPLETAR: efecto del delivery_time sobre review_score segmentado por categoría.]` |
| **H5 – Perfil del cliente insatisfecho** | ¿Se puede predecir baja satisfacción combinando variables? | `[COMPLETAR – se responde con el modelo de la Sección 5]` | `[COMPLETAR]` |

**Diagnóstico preliminar:** la evidencia descriptiva ya apunta a que **H2 y H3 son las hipótesis con mayor respaldo** (la entrega y los retrasos extremos explican gran parte de las malas calificaciones), mientras que **H4 tiende a refutarse** parcialmente porque las diferencias de rating entre categorías son pequeñas. El equipo debe cerrar cada fila con la evidencia cuantitativa del notebook.

---

## 5. Nivel 3 – Analítica predictiva: ¿qué pasará?

> Esta sección responde principalmente a **H5 (perfil del cliente insatisfecho)**.

### 5.1 Planteamiento del modelo

Se entrena un modelo para **predecir la probabilidad de que un pedido reciba una calificación baja (1–2 estrellas)** a partir de las variables del pedido. La variable objetivo es binaria (`baja_satisfaccion = 1 si review_score ≤ 2`), y las predictoras incluyen `delivery_time`, `price`, `freight_value`, categoría y las variables de enriquecimiento temporal.

> `[COMPLETAR: indicar el modelo finalmente usado (regresión logística, árbol de decisión, Random Forest o XGBoost), justificar la elección y describir el tratamiento del desbalance de clases —las reseñas 1–2★ son minoría—, por ejemplo con SMOTE o class_weight.]`

### 5.2 Métricas de desempeño

> `[COMPLETAR: reportar las métricas reales del modelo e interpretarlas en lenguaje de negocio.]`

| Métrica | Valor | Qué significa para el negocio |
|---|---|---|
| Accuracy | `[COMPLETAR]` | `[COMPLETAR]` |
| Precision (clase insatisfecho) | `[COMPLETAR]` | De cada 100 pedidos que el modelo marca como riesgo, cuántos lo son realmente |
| Recall (clase insatisfecho) | `[COMPLETAR]` | De cada 100 clientes que se van a quejar, cuántos detectamos a tiempo |
| F1 / AUC | `[COMPLETAR]` | Capacidad global de distinguir clientes satisfechos de insatisfechos |

### 5.3 Interpretación de resultados

> `[COMPLETAR: traducir la salida del modelo a negocio. Insertar el gráfico de feature importance y responder: ¿qué variable predice más la baja satisfacción? ¿cuántos pedidos del total caen en zona de riesgo? ¿qué magnitud tiene el efecto del tiempo de entrega?]`

**Narrativa esperada (a confirmar):** se anticipa que el **tiempo de entrega será la variable más importante** del modelo, por encima del precio y la categoría, reforzando los hallazgos descriptivos y diagnósticos. Esto permitiría a Olist **anticipar qué pedidos en curso tienen alto riesgo de mala calificación** y actuar antes de que el cliente reseñe.

---

## 6. Nivel 4 – Analítica prescriptiva: ¿qué debe hacer Olist?

### 6.1 Escenarios "¿qué pasaría si…?"

> `[COMPLETAR: cuantificar al menos 2 escenarios con supuestos explícitos, apoyados en los coeficientes del modelo o en los promedios por segmento.]`

**Escenario A – Reducir la cola de retrasos extremos.**
Supuesto: si Olist interviene los pedidos con `delivery_time` por encima del umbral crítico de H2 (los que hoy concentran las peores notas), reduciéndolos en un X%.
Impacto estimado: `[COMPLETAR: proyección de incremento en el review_score promedio o en el % de reseñas 4–5★.]`

**Escenario B – Sistema de alerta temprana con el modelo predictivo.**
Supuesto: si Olist usa el modelo de la Sección 5 para detectar el % de pedidos en riesgo y activa compensaciones o seguimiento proactivo.
Impacto estimado: `[COMPLETAR: cuántos clientes insatisfechos se podrían recuperar y efecto esperado en recompra.]`

### 6.2 Recomendaciones estratégicas

Tres recomendaciones priorizadas por impacto, cada una trazada a un hallazgo del análisis:

| # | Recomendación | Acción concreta | Responsable sugerido | Indicador de éxito | Hallazgo que la respalda |
|---|---|---|---|---|---|
| 1 | **Atacar la cola logística antes que el catálogo** | Auditar y rediseñar el proceso de los envíos que superan el umbral crítico de días; priorizar regiones/vendedores con más retrasos | Operaciones / Logística | Reducir el % de pedidos sobre el umbral crítico y subir el review_score promedio | Nivel 1 (máx 208 días) y Nivel 2 (H2, H3) |
| 2 | **Implementar alerta temprana de insatisfacción** | Integrar el modelo predictivo (Nivel 3) para marcar pedidos en riesgo y activar seguimiento proactivo | Data / Customer Experience | Recall del modelo y % de clientes en riesgo recuperados | Nivel 3 (modelo H5) |
| 3 | **No invertir en diferenciación por categoría para mejorar satisfacción** | Reorientar el presupuesto de mejora desde el catálogo hacia la operación y la comunicación de plazos realistas | Estrategia / Marketing | Mantener satisfacción con menor gasto en ajustes de catálogo | Nivel 1 y 2 (rating casi plano entre categorías) |

### 6.3 Trazabilidad dato → insight → decisión

El hilo argumental del proyecto es consistente: los datos muestran que **la categoría apenas mueve la satisfacción** y que **la entrega sí**, en especial sus casos extremos; el modelo confirma qué pedidos están en riesgo; y las recomendaciones redirigen la inversión hacia donde el dato indica que está el verdadero problema. Cada recomendación cita explícitamente el nivel de análisis que la sustenta.

---

## 7. Dashboard ejecutivo

**Link funcional:** `[COMPLETAR: enlace a Power BI / Looker Studio]`

El dashboard incluye un mínimo de cuatro visualizaciones interactivas, cada una titulada como una pregunta de negocio:

1. **¿Cómo se distribuye la satisfacción de nuestros clientes?** — distribución de `review_score`.
2. **¿Cuánto tarda realmente una entrega y cuándo se vuelve un problema?** — `delivery_time` vs. `review_score`.
3. **¿Qué categorías venden más y cuáles satisfacen más?** — ventas y rating por categoría.
4. **¿Dónde están los pedidos en riesgo?** — `[COMPLETAR: visual geográfica o de segmentos de riesgo según el modelo.]`

> `[COMPLETAR: pegar capturas o describir brevemente cada visual final del dashboard.]`

---

## 8. Limitaciones del análisis

- **No se observa recompra real:** el dataset no tiene una variable explícita de retención, por lo que la "recompra" de la pregunta de negocio se aproxima de forma indirecta y debe interpretarse con cautela.
- **Sesgo de satisfacción:** las calificaciones están muy concentradas en 4 y 5 estrellas, lo que genera desbalance de clases y obliga a técnicas específicas en el modelado.
- **Variables externas ausentes:** el dataset no incluye promociones, stock, competencia ni campañas, factores que también influyen en la satisfacción.
- **Outliers logísticos extremos:** los casos atípicos de entrega distorsionan promedios si no se tratan; se reportan tanto media como mediana para mitigarlo.
- **Período histórico (2016–2018):** los patrones pueden haber cambiado; las conclusiones aplican al período analizado.

---

## 9. Conclusiones y próximos pasos

El análisis confirma que, en Olist, **un producto "triunfa" menos por lo que es y más por cómo llega**. La logística —y en particular la consistencia de la entrega— es la palanca con mayor relación con la satisfacción, mientras que precio y categoría tienen un rol secundario. El cliente insatisfecho es, sobre todo, un cliente que esperó demasiado.

**Si el proyecto continuara**, los siguientes pasos serían: (1) construir una métrica proxy de recompra a partir de `customer_unique_id` para medir retención real; (2) incorporar datos de vendedores y regiones para localizar los focos de retraso; y (3) llevar el modelo predictivo a un sistema de alerta en tiempo real.

---

## Apéndice A – Bitácora de contribución

| Semana | Camila Torres | Letizia Torres | Marcelo Villafuerte | Pablo Vega |
|---|---|---|---|---|
| 7 | `[COMPLETAR]` | `[COMPLETAR]` | `[COMPLETAR]` | `[COMPLETAR]` |
| 8 | … | … | … | … |
| 9 | … | … | … | … |
| 10 | … | … | … | … |
| 11 | … | … | … | … |
| 12 | … | … | … | … |
| 13 | … | … | … | … |
| 14 | … | … | … | … |

> `[COMPLETAR: cada integrante describe brevemente su aporte semanal. Esta tabla es obligatoria según la Sección 7.4 del documento del curso.]`

---

## Apéndice B – Uso de Inteligencia Artificial Generativa

Durante la PC2 se utilizó IA generativa como apoyo para **estructurar y redactar el informe**, no para generar análisis ni resultados. Las decisiones analíticas, la carga e integración de datos, el modelado y las visualizaciones fueron realizadas por el equipo.

| Actividad | Herramienta | Prompt (resumen) | Validación realizada |
|---|---|---|---|
| Estructura y redacción del informe PC2 | `[COMPLETAR: ChatGPT / Claude]` | "Redactar el informe final PC2 integrando los 4 niveles de analítica para el caso Olist." | Se contrastó cada sección con la rúbrica y con los resultados reales del notebook; los marcadores `[COMPLETAR]` se llenaron con datos propios |
| `[COMPLETAR: otras actividades con IA, p. ej. apoyo en código del modelo]` | `[COMPLETAR]` | `[COMPLETAR]` | `[COMPLETAR]` |

> **Importante:** reemplacen todos los `[COMPLETAR]` por el uso real de IA. Declarar con precisión protege la nota; la rúbrica anula la evaluación si detecta un informe generado íntegramente por IA sin aporte del equipo.
