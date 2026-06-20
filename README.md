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

El análisis consideró 95,824 pedidos entregados que contaban con información de entrega y calificación. Dentro de esta muestra, 12,272 pedidos, equivalentes al 12.81%, presentaron baja satisfacción. Debido a que las reseñas de 1 y 2 estrellas son minoría, el problema presenta desbalance de clases.

Variables predictoras utilizadas
  Las variables incorporadas en el modelo fueron agrupadas en cuatro bloques:

| Bloque | Variables |
|---|---|
| Logística | `[delivery_time`, `estimated_delivery_days`, `days_to_approval`, `freight_value`, `ratio flete/precio]` 
|Precio y pedido| `[price]`, número de productos, número de cuotas, peso del producto, cantidad de fotos |
| Producto y ubicación | categoría, estado del cliente, tipo de pago | 
| Contexto temporal | mes de compra, día de la semana, periodo Black Friday y periodo navideño |

El indicador exacto is_holiday no se incluyó directamente porque no presentaba variación útil en la tabla final. En su lugar, se utilizaron variables temporales que sí permiten capturar estacionalidad y periodos comerciales de alta demanda, como el mes de compra, Black Friday y Navidad.

También se debe precisar que el dataset no contiene una calificación histórica directa del vendedor. Por esta razón, no se utilizó una “calificación del vendedor” como predictor. En una futura versión del modelo se recomienda crear un score histórico del vendedor utilizando únicamente información previa al pedido, por ejemplo: tasa de retrasos anteriores, satisfacción histórica promedio y volumen de ventas reciente.

Modelo seleccionado

| Modelo | AUC ROC | Precision | Recall | F1-score |
|---|---|---|---|---|
| Regresión logística balanceada | 0.6974 | 0.2738 | 0.3811 | 0.3186 
| Random Forest balanceado | 0.6998 | 0.3241 | 0.3881 | 0.3532

El modelo seleccionado fue Random Forest Classifier con balanceo de clases.

  La elección se justifica porque obtuvo mejores resultados que la regresión logística en precisión, recall y F1-score. Además, Random Forest permite capturar relaciones no lineales, especialmente relevantes en este caso, ya que el riesgo de una mala calificación no aumenta de forma constante: se incrementa con fuerza cuando el tiempo de entrega supera determinados umbrales.

Para atender el desbalance de clases se utilizó:

class_weight = "balanced_subsample"

Esta técnica asigna mayor peso a los pedidos con baja satisfacción durante el entrenamiento. No se aplicó SMOTE, ya que el uso de pesos de clase permite preservar los registros reales y evita generar combinaciones sintéticas poco realistas entre variables numéricas y categóricas.

Validación temporal
La validación se realizó mediante una separación temporal para simular un escenario real de negocio:

| Conjunto | Periodo de compra | Pedidos | 
|---|---|---|
| Entrenamiento | Antes del 1 de abril de 2018 | 63,818 
| Prueba | Desde el 1 de abril de 2018 | 32,006 

Este enfoque evita entrenar el modelo con información futura y permite evaluar si los patrones aprendidos en pedidos pasados se mantienen en pedidos posteriores.

### 5.2 Métricas de desempeño

| Métrica | Valor | Interpretación para el negocio |
|---|---|---|
| Accuracy | `85.26%` | El modelo clasifica correctamente la mayoría de los pedidos. Sin embargo, esta métrica debe interpretarse con cautela porque los pedidos con baja satisfacción son minoría. |
| Precision, clase insatisfecho | `32.41%` | De cada 100 pedidos que el modelo marca como de alto riesgo, aproximadamente 32 realmente terminan con una calificación de 1 o 2 estrellas. |
| Recall, clase insatisfecho | `38.81%` | De cada 100 pedidos que efectivamente reciben una mala calificación, el modelo identifica cerca de 39 antes de la reseña. |
| F1-score | `35.32%` | El modelo logra un equilibrio moderado entre detectar clientes insatisfechos y evitar demasiadas alertas falsas. |
| AUC ROC | `69.98%` | El modelo tiene una capacidad aceptable para distinguir entre pedidos con mayor y menor riesgo de baja satisfacción. |
| Average Precision | `30.73%` | El modelo mejora de forma importante la priorización de pedidos riesgosos frente a una selección aleatoria. |

La tasa real de baja satisfacción en el conjunto de prueba fue de aproximadamente 10.37%. En cambio, dentro de los pedidos clasificados como de mayor riesgo, la tasa de baja satisfacción fue considerablemente más alta.

Esto demuestra que el modelo no debe utilizarse como una decisión automática definitiva, sino como una herramienta de priorización para enfocar la atención de operaciones y servicio al cliente en los pedidos más vulnerables.

### 5.3 Interpretación de resultados

El modelo confirma que sí es posible identificar un perfil de pedido con mayor probabilidad de baja satisfacción. Los pedidos más riesgosos suelen combinar tiempos de entrega prolongados, flete elevado, plazos prometidos largos, mayor peso del producto y determinadas categorías o ubicaciones geográficas.
Las variables con mayor importancia en el modelo fueron las siguientes:

| Variable | Importancia estimada | 
|---|---|
| Tiempo real de entrega (delivery_time) | 47.8% 
| Valor del flete | 8.2%
| Número de ítems del pedido | 6.9%
| Días estimados de entrega | 5.3%
| Ratio flete / precio | 5.2%
| Precio del pedido | 5.0%
| Peso del producto | 5.0%
| Categoría del producto | 3.6%
| Mes de compra | 3.5%
| Estado del cliente | 2.9%

El resultado más relevante es que el tiempo de entrega representa aproximadamente el 47.8% de la importancia predictiva total del modelo. Esto confirma que la logística es el principal factor asociado a la baja satisfacción, por encima del precio, la categoría o el contexto temporal.

Magnitud del efecto del tiempo de entrega
La relación entre tiempo de entrega y riesgo de recibir una calificación baja es clara:

| Tiempo de entrega | Pedidos | Porcentaje con calificación 1–2 estrellas |
|---|---|---|
| 0 a 3 días | 8,565 | 7.1%
| 4 a 7 días | 24,974 | 7.1%
| 8 a 10 días | 18,300 | 8.9%
| 11 a 14 días | 17,890 | 9.7%
| 15 a 21 días | 15,264 | 12.5%
| 22 a 30 días | 6,824 | 27.2%
| Más de 30 días | 4,007 | 65.3%

Los pedidos entregados en un plazo de hasta 14 días presentan una tasa de baja satisfacción de aproximadamente 8.5%. En contraste, cuando la entrega supera los 30 días, el riesgo aumenta a 65.3%.

Esto representa una diferencia de aproximadamente 56.8 puntos porcentuales y un riesgo casi 7.7 veces mayor frente a un pedido entregado dentro de los primeros 14 días.

Por lo tanto, se identifican dos umbrales operativos relevantes:

    1).15 días: umbral preventivo, ya que la baja satisfacción comienza a crecer de manera sostenida.
    2).22 días: umbral crítico, ya que el riesgo se eleva a más del doble respecto a entregas menores a 15 días.
    3).Más de 30 días: zona de riesgo extremo, donde cerca de dos de cada tres pedidos terminan con una calificación de 1 o 2 estrellas.

Zona de alto riesgo
Para convertir la predicción en una herramienta operativa, se definió como zona de alto riesgo al 10% de pedidos con mayor probabilidad estimada de baja satisfacción.

| Indicador de priorización | Resultado |
|---|---|
| Pedidos evaluados en el conjunto de prueba | 32,006
| Pedidos ubicados en el top 10% de riesgo | 3,200 
| Pedidos con baja satisfacción dentro del top 10% | 1,137 
| Tasa de baja satisfacción dentro del top 10% | 35.5% 
| Casos totales de baja satisfacción detectados por este grupo | 34.3%

Esto significa que, revisando solamente el 10% de pedidos con mayor riesgo, Olist puede concentrar su atención en un grupo donde más de uno de cada tres pedidos termina con una calificación de 1 o 2 estrellas.

Si este criterio se aplicara sobre el total de pedidos analizados, aproximadamente 9,582 pedidos entrarían en una zona de seguimiento prioritario.

Conclusión predictiva
La hipótesis H5 queda respaldada: existe un perfil de pedido con mayor probabilidad de terminar en baja satisfacción.

El principal predictor es el tiempo de entrega. El modelo demuestra que los retrasos prolongados, especialmente los superiores a 22 días, aumentan de forma considerable el riesgo de recibir una calificación negativa.

Por ello, el modelo puede ser utilizado para identificar pedidos que requieren intervención prioritaria. La utilidad no está en reemplazar al equipo de operaciones, sino en ordenar la atención: primero deben revisarse los pedidos con mayor probabilidad de generar insatisfacción, reclamos y una mala calificación.

En términos de negocio, Olist debe pasar de una gestión reactiva, que actúa después de recibir una reseña negativa, a una gestión preventiva basada en riesgo logístico y probabilidad de baja satisfacción.

---

## 6. Nivel 4 – Analítica prescriptiva: ¿qué debe hacer Olist?

El análisis prescriptivo transforma los hallazgos descriptivos, diagnósticos y predictivos en decisiones operativas concretas. El objetivo no es únicamente identificar qué factores explican una mala experiencia, sino determinar qué acciones debe implementar Olist para reducir la baja satisfacción antes de que el cliente deje una calificación negativa.

Los escenarios presentados son simulaciones de impacto basadas en los promedios observados por segmento y en el desempeño del modelo predictivo. Por tanto, representan estimaciones de negocio que deben validarse posteriormente mediante una prueba piloto o experimento controlado.

### 6.1 Escenarios "¿qué pasaría si…?"

**Escenario A – Reducir la cola de entregas críticas**
El análisis diagnóstico identificó que la satisfacción empieza a deteriorarse de forma sostenida desde los 15 días de entrega y se vuelve crítica a partir de los 22 días. Los pedidos con más de 30 días representan la zona de mayor riesgo, ya que concentran una proporción muy alta de calificaciones de 1 y 2 estrellas.

Para este escenario, se consideran como pedidos críticos aquellos con un tiempo de entrega superior a 22 días.

| Indicador | Valor observado | 
|---|---|
| Pedidos con entrega entre 22 y 30 días | 6,824
| Pedidos con entrega superior a 30 días | 4,007
| Total de pedidos críticos, más de 22 días | 10,831
| Tasa promedio de baja satisfacción en pedidos críticos | 41.3%
| Satisfacción promedio, 4–5 estrellas, en pedidos críticos | 46.5%
| Rating promedio estimado de pedidos críticos | 5.0%
| Peso del producto | 3.00
| Tasa de baja satisfacción en entregas de 15 a 21 días | 12.5%
| Satisfacción promedio, 4–5 estrellas, en entregas de 15 a 21 días | 3.5%
| Estado del cliente | 77.4%
| Rating promedio de entregas de 15 a 21 días | 4.10

Supuesto del escenario
Se asume que Olist implementa mejoras logísticas y logra reducir en 25% la cantidad de pedidos que actualmente supera el umbral crítico de 22 días.

Se asume además que los pedidos recuperados alcanzan un desempeño similar al segmento de entregas de 15 a 21 días.

Impacto estimado

| Resultado esperado | Cálculo | Impacto |
|---|---|---|
| Malas calificaciones evitadas, 1–2 estrellas | 2,708 × (41.3% - 12.5%) | 780 pedidos aproximadamente
| Reseñas satisfactorias adicionales, 4–5 estrellas | 2,708 × (77.4% - 46.5%) | 837 pedidos aproximadamente
| Mejora del review score en los pedidos intervenidos | 4.10 - 3.00 | +1.10 puntos
| Mejora estimada del review score promedio global | Impacto ponderado sobre 95,824 pedidos | +0.03 puntos aproximadamente

Interpretación de negocio

Reducir una cuarta parte de los pedidos que superan los 22 días permitiría evitar aproximadamente 780 experiencias de baja satisfacción y generar alrededor de 837 reseñas adicionales de 4 o 5 estrellas.

Aunque una mejora global de 0.03 puntos en el review score promedio puede parecer pequeña, su relevancia está en que se concentra en los clientes con peor experiencia. Es decir, la intervención no busca modificar marginalmente a todos los clientes, sino recuperar los casos de mayor riesgo reputacional, mayor probabilidad de reclamo y peor percepción del servicio.

**Escenario B – Sistema de alerta temprana con el modelo predictivo**

El modelo Random Forest identificó que el 10% de pedidos con mayor probabilidad de baja satisfacción concentra una proporción importante de pedidos que finalmente reciben una calificación de 1 o 2 estrellas.

| Indicador del modelo | Resultado | 
|---|---|
| Pedidos del conjunto de prueba | 32,006 
| Pedidos ubicados en el top 10% de riesgo | 3,200
| Pedidos con baja satisfacción dentro del top 10% | 1,137
| Tasa de baja satisfacción en el top 10% | 35.5%
| Casos de baja satisfacción capturados por el top 10% | 34.3%

Supuesto del escenario

Se propone que Olist use el modelo para identificar el 10% de pedidos con mayor riesgo y active un protocolo de intervención temprana que incluya:

    1). Verificación del estado de envío.
    2). Priorización del pedido con el operador logístico.
    3). Comunicación proactiva y transparente al cliente.
    4). Actualización de la fecha estimada de entrega.
    5). Compensación comercial cuando el retraso ya no pueda evitarse.
    6). Revisión de vendedor, ruta y categoría cuando se detecten patrones recurrentes.
Sobre el total de 95,824 pedidos analizados, el 10% de mayor riesgo representaría aproximadamente: 9,582
Si este grupo mantiene una tasa de baja satisfacción de 35.5%, se esperaría que concentre aproximadamente: 3,402
Se asume que la intervención operativa logra recuperar el 25% de estos casos de riesgo: 851

| Resultado esperado | Impacto | 
|---|---|
| Pedidos ubicados en zona de alto riesgo | 9,582
| Casos esperados de baja satisfacción dentro de la zona de riesgo | 3,402
| Casos potencialmente recuperados con intervención del 25% | 851
| Malas reseñas de 1–2 estrellas potencialmente evitadas | 851
| Mejora esperada | Menor cantidad de reclamos y mayor satisfacción en pedidos críticos

Interpretación de negocio

El modelo permite que Olist no tenga que intervenir todos los pedidos, sino que concentre recursos en aquellos con mayor probabilidad de generar una experiencia negativa.

En lugar de revisar de manera uniforme los más de 95 mil pedidos, el equipo puede priorizar aproximadamente 9,582 pedidos de alto riesgo. Bajo el supuesto de una intervención efectiva en el 25% de los casos críticos, Olist podría recuperar aproximadamente 851 clientes que, de otra forma, tendrían alta probabilidad de dejar una calificación de 1 o 2 estrellas.

No es posible calcular el efecto real sobre la recompra porque el dataset no contiene compras posteriores por cliente ni una variable de retención. Sin embargo, reducir las malas experiencias es una señal razonable de protección de la confianza del cliente y de mejora potencial de la relación futura con la plataforma.

### 6.2 Recomendaciones estratégicas

Tres recomendaciones priorizadas por impacto, cada una trazada a un hallazgo del análisis:

| # | Recomendación estratégica | Acción concreta | Responsable sugerido | Indicador de éxito | Hallazgo que la respalda |
|---|---|---|---|---|---|
| 1 | **Atacar la cola logística antes que el catálogo** | Crear una cola operativa para pedidos con más de 15 días; activar alerta naranja desde los 22 días y prioridad máxima desde los 30 días. Auditar vendedores, rutas y regiones que concentran entregas críticas. | Operaciones / Logística | Reducir el porcentaje de pedidos con más de 22 días; reducir las reseñas 1–2 estrellas en pedidos críticos; mejorar el review score promedio. | Nivel 1: tiempos extremos de hasta 208 días. Nivel 2: H2 y H3 muestran que la satisfacción cae de forma crítica después de 22 días. |
| 2 | **Implementar alerta temprana de insatisfacción** | Aplicar el modelo predictivo diariamente; revisar primero el top 10% de pedidos con mayor riesgo; asignar responsable y acción correctiva en menos de 24 horas. | Data Analytics / Customer Experience / Operaciones | Recall@10%; porcentaje de pedidos de riesgo intervenidos; porcentaje de clientes de alto riesgo recuperados; reducción de malas reseñas. | Nivel 3: H5 confirma que existe un perfil de pedido con alta probabilidad de baja satisfacción. |
| 3 | No priorizar el catálogo como principal palanca de satisfacción |Mantener mejoras de catálogo y surtido, pero reasignar la inversión principal hacia logística, comunicación proactiva y promesas de entrega realistas. Aplicar monitoreo por categoría solo para detectar categorías con alta sensibilidad a retrasos. | Estrategia / Marketplace / Marketing |Mantener o mejorar el rating promedio con menor gasto en ajustes generalizados de catálogo; reducción de reclamos logísticos. | Nivel 1 y Nivel 2: el rating promedio varía poco entre categorías, mientras que el tiempo de entrega presenta un efecto mucho mayor. |

Recomendación 1: Gestión activa de la cola logística

La principal prioridad de Olist debe ser reducir las entregas que superan los umbrales críticos. La evidencia muestra que el problema no está distribuido de forma uniforme: una proporción relativamente pequeña de pedidos con tiempos excesivos concentra una gran parte de las calificaciones negativas.

Se recomienda implementar tres niveles de control:

| Nivel | Condición | Acción |
|---|---|---|
| Preventivo | Entre 15 y 21 días | Revisar estado de envío y validar cumplimiento de fecha prometida.
| Crítico | Entre 22 y 30 días | Escalar el pedido, informar al cliente y revisar transportista, vendedor y ruta.
| Extremo | Más de 30 días | Prioridad máxima; intervención logística, compensación comercial y análisis de causa raíz.

Recomendación 2: Modelo de riesgo como herramienta de priorización

El modelo predictivo no debe reemplazar al equipo de operaciones ni decidir automáticamente qué cliente recibirá una compensación. Su función principal es priorizar.

La lógica de uso sería:

El modelo asigna una probabilidad de baja satisfacción a cada pedido.
Olist selecciona el top 10% de pedidos con mayor riesgo.
Operaciones revisa el estado real de los pedidos priorizados.
Customer Experience contacta de manera preventiva a los clientes afectados.
Se registra la acción tomada y se mide si la mala experiencia fue evitada.
El resultado se incorpora como retroalimentación para mejorar el modelo.

La meta no es eliminar todas las malas reseñas, sino reducir las que son prevenibles mediante intervención temprana

Recomendación 3: Reorientar el presupuesto hacia la experiencia de entrega

Los resultados muestran que las diferencias de satisfacción entre categorías son relativamente pequeñas, mientras que el tiempo de entrega tiene un efecto mucho más fuerte sobre la probabilidad de una mala calificación.

Por ello, no se recomienda eliminar la inversión en catálogo, variedad o segmentación de productos. Sin embargo, estas acciones no deben ser la principal inversión destinada a mejorar la satisfacción.

La prioridad debe ser:

    1). Mejorar el cumplimiento de la promesa de entrega.
    2). Ajustar fechas estimadas para que sean realistas.
    3). Reducir retrasos en rutas y regiones críticas.
    4). Gestionar vendedores con alto historial de demoras.
    5). Comunicar de forma transparente cualquier cambio en el envío.
    6). Ofrecer soluciones rápidas cuando el retraso no pueda evitarse.
    
### 6.3 Trazabilidad dato → insight → decisión

El proyecto sigue un hilo lógico desde los datos hasta la recomendación estratégica.

El hilo argumental del proyecto es consistente: los datos muestran que **la categoría apenas mueve la satisfacción** y que **la entrega sí**, en especial sus casos extremos; el modelo confirma qué pedidos están en riesgo; y las recomendaciones redirigen la inversión hacia donde el dato indica que está el verdadero problema. Cada recomendación cita explícitamente el nivel de análisis que la sustenta.

| Nivel de análisis | Dato observado | Insight obtenido | Decisión recomendada | 
|---|---|---|---|
| Nivel 1: Descriptivo | Existen pedidos con tiempos de entrega de hasta 208 días y una distribución con outliers logísticos. | La operación presenta una cola extrema de pedidos con desempeño anormal. | Monitorear la distribución completa y no solo el promedio de entrega. 
| Nivel 2: Diagnóstico, H2 | La baja satisfacción aumenta desde los 15 días y se vuelve crítica después de 22 días. | El tiempo de entrega tiene un umbral operativo claro. | Crear alertas preventivas desde los 15 días y escalamiento crítico desde los 22 días. 
| Nivel 2: Diagnóstico, H3 | Los pedidos con retrasos extremos concentran ratings mucho más bajos. | Los casos extremos tienen impacto desproporcionado en la experiencia del cliente. | Los casos extremos tienen impacto desproporcionado en la experiencia del cliente. | Crear una cola crítica y un protocolo especial para pedidos de más de 30 días. 
|Nivel 2: Diagnóstico, H4 | Las diferencias de satisfacción entre categorías son menores que las diferencias explicadas por el tiempo de entrega. | La categoría influye, pero no es la principal causa de insatisfacción. | La categoría influye, pero no es la principal causa de insatisfacción. | No priorizar ajustes generalizados de catálogo como solución principal. 
| Nivel 3: Predictivo, H5 | El modelo identifica una zona de alto riesgo donde la baja satisfacción es mucho mayor que en la población general. | Es posible anticipar pedidos con probabilidad elevada de recibir una mala reseña. | Integrar un sistema de alerta temprana y priorización de pedidos. 
| Nivel 4: Prescriptivo | Los escenarios muestran que intervenir pedidos críticos puede evitar cientos de malas experiencias. | La inversión en logística y seguimiento proactivo tiene potencial de alto impacto. | Implementar piloto de intervención y medir resultados antes de escalar la solución. 

Conclusión prescriptiva

Los datos muestran que el principal problema de satisfacción no se explica por el precio ni por la categoría del producto, sino por los retrasos logísticos, especialmente aquellos que superan los 22 días.

El modelo predictivo permite identificar qué pedidos tienen mayor riesgo de terminar con una baja calificación. A partir de ello, Olist puede intervenir antes de que el cliente deje una reseña negativa.

La recomendación central es redirigir la inversión hacia una gestión preventiva de la experiencia logística: alertas tempranas, control de rutas críticas, seguimiento de vendedores, comunicación proactiva y protocolos para pedidos extremos.

De esta manera, Olist pasa de una gestión reactiva, donde responde después de recibir una mala calificación, a una gestión prescriptiva basada en riesgo, priorización e intervención oportuna.

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
