# Informe Final – PC2
## ¿Qué hace que un producto triunfe en e-commerce?
### MarketPulse Analytics – Análisis de Satisfacción del Cliente (Olist)

**Curso:** AD3005 – Introducción a Data Analytics y Big Data · Ciclo 2026-1 · UTEC
**Equipo:** Grupo 3 – Big Data y Data Analytics
**Integrantes:** Camila Torres · Letizia Torres · Marcelo Villafuerte · Pablo Vega
**Caso:** Caso 1 – Brazilian E-Commerce (Olist)
**Entregable:** PC2 – Análisis Final, Dashboard y Presentación

---

## Resumen ejecutivo

Olist es un marketplace brasileño que conecta a pequeños vendedores con grandes canales de venta en línea. La pregunta que guía el proyecto es: **¿qué factores (precio, categoría, tiempo de entrega, calificación del vendedor) predicen mejor la satisfacción del cliente y la recompra?**

Tras integrar y analizar **114,092 registros** de pedidos reales (2016–2018), enriquecidos con calendario comercial y tipo de cambio, el hallazgo central es contundente: **la satisfacción del cliente en Olist se explica por la logística, no por el producto.** Un pedido entregado a tiempo obtiene en promedio **4.14 estrellas**; uno entregado tarde cae a **2.26**. La categoría del producto y el precio casi no mueven la aguja.

A partir de ese hallazgo, el equipo construyó un **modelo predictivo** que estima la probabilidad de que un pedido llegue tarde (y por tanto reciba mala calificación), y un **análisis de escenarios** que cuantifica el impacto de intervenir los pedidos de mayor riesgo. El resultado se traduce en tres recomendaciones priorizadas que apuntan al verdadero cuello de botella —la entrega— antes que al catálogo o al precio.

---

## 1. El problema de negocio

### 1.1 Contexto

Olist actúa como intermediario entre pequeños vendedores y los principales marketplaces de Brasil. Su modelo depende de un equilibrio: que el vendedor venda y que el cliente quede satisfecho para recomprar. El problema es que **vender mucho no garantiza satisfacción ni recompra**, y la empresa no sabe qué palanca mover primero para mejorar la experiencia.

### 1.2 Pregunta de negocio

> *¿Qué factores (precio, categoría, tiempo de entrega, calificación del vendedor) predicen mejor la satisfacción del cliente y la recompra?*

### 1.3 Por qué importa

Responderla permite a Olist **priorizar inversión con evidencia**: no tiene sentido optimizar el catálogo si el problema es logístico, ni bajar precios si el cliente insatisfecho lo está por una entrega tardía. En un marketplace, cada punto de satisfacción se traduce en mayor recompra, mejor reputación y menor costo de adquisición.

---

## 2. Datos y metodología

### 2.1 Fuente principal

**Brazilian E-Commerce Public Dataset by Olist** (Kaggle: `olistbr/brazilian-ecommerce`). Período: **septiembre 2016 – octubre 2018**. Reúne información real de pedidos, productos, clientes, vendedores, pagos, reseñas y logística.

| Tabla | Uso en el proyecto |
|---|---|
| `olist_orders_dataset.csv` | Fechas de compra, entrega y estado del pedido |
| `olist_order_items_dataset.csv` | Precio, flete y producto vendido |
| `olist_order_reviews_dataset.csv` | Satisfacción del cliente (`review_score` 1–5) |
| `olist_customers_dataset.csv` | Cliente y ubicación |
| `olist_products_dataset.csv` | Categoría y características del producto |
| `olist_order_payments_dataset.csv` | Métodos y montos de pago |

### 2.2 Integración y enriquecimiento

Las tablas se unieron mediante `merge` en una base analítica de **114,092 filas y 51 columnas**. Sobre ella se construyeron variables derivadas de logística (`delivery_time`, `delivery_delay_days`, `is_late_delivery`), de contexto temporal y de conversión de moneda. El resultado se exportó como `olist_enriched_final.csv` (base usada en todo el análisis y el dashboard).

**Enriquecimiento 1 – Calendario comercial y festivos de Brasil.** Permite ubicar cada compra en su contexto temporal:

| Variable | Descripción |
|---|---|
| `purchase_month`, `purchase_day_of_week`, `is_weekend` | Estacionalidad y patrones por día |
| `is_holiday`, `is_black_friday`, `is_christmas_period` | Marca compras en fechas comerciales clave |
| `purchase_context` | Clasifica cada pedido en *Regular, Feriado, Navidad o Black Friday* |

Distribución de `purchase_context`: **Regular 105,223 · Navidad 5,521 · Feriado 1,958 · Black Friday 1,390**.

**Enriquecimiento 2 – Tipo de cambio BRL/PEN** (factor 0.70). Genera `price_pen`, `freight_value_pen` y `payment_value_pen` para interpretar los montos en soles (ej. la mediana de precio de 74.9 BRL ≈ **S/ 52**).

### 2.3 Niveles de analítica aplicados

El análisis cubre los cuatro niveles: **descriptiva** (¿qué pasó?), **diagnóstica** (¿por qué?), **predictiva** (¿qué pasará?) y **prescriptiva** (¿qué hacer?).

---

## 3. Nivel 1 – Analítica descriptiva: ¿qué pasó?

### 3.1 Estadísticas descriptivas de las variables clave

| Variable | Media | Mediana | Desv. est. | Mín | P25 | P75 | Máx |
|---|---|---|---|---|---|---|---|
| `price` (BRL) | 120.48 | 74.90 | 183.28 | 0.85 | 39.90 | 134.90 | 6,735.00 |
| `review_score` (1–5) | 4.02 | 5 | 1.39 | 1 | — | — | 5 |
| `delivery_time` (días) | **12.01** | 10 | — | 0 | — | — | 209 |

**Lectura de negocio:**

- **Precio:** distribución muy sesgada a la derecha. El promedio (≈120 BRL) duplica a la mediana (≈75 BRL), señal de un puñado de productos premium —hasta 6,735 BRL ≈ S/ 4,713— que inflan el promedio. El 75% del catálogo cuesta menos de 135 BRL: **es un mercado masivo de precio bajo-medio**.
- **Satisfacción:** promedio de 4.02/5; **la experiencia general es positiva**, pero ese promedio esconde bolsones de insatisfacción que el nivel diagnóstico destapa.
- **Entrega:** la mayoría de pedidos llega en ~10–12 días, pero existen **casos extremos de hasta 209 días**: fallas logísticas severas en una minoría de envíos.

> **Corrección respecto a la PC1:** en la PC1 se reportó un tiempo de entrega promedio de "4 días". Ese valor era en realidad el promedio de `review_score` (4.02). El promedio real de entrega es **12 días** (mediana 10), corregido en este informe.

### 3.2 Visualizaciones descriptivas

**Ventas en el tiempo.** Las ventas crecen de forma sostenida durante 2017 y se estabilizan en 2018. La caída abrupta al final corresponde a meses con datos incompletos, no a una baja real de demanda.

![Ventas por mes](images/01_ventas_por_mes.png)

**Categorías que generan más ingresos.** Las ventas se concentran en pocas categorías: `cama_mesa_banho`, `beleza_saude` e `informatica_acessorios` lideran. *Conclusión: demanda focalizada en un grupo reducido de categorías.*

![Top 10 categorías por ventas](images/02_top_categorias.png)

**Distribución de la satisfacción.** Fuerte concentración en 4 y 5 estrellas. *Conclusión: satisfacción general alta, pero con suficientes reseñas de 1–2 estrellas como para justificar el análisis del cliente insatisfecho.*

![Distribución de calificaciones](images/03_distribucion_reviews.png)

**Concentración geográfica.** São Paulo (SP) domina las ventas, seguido de Río de Janeiro (RJ) y Minas Gerais (MG). *Conclusión: el negocio depende fuertemente del sudeste de Brasil.*

![Top estados por ventas](images/04_top_estados.png)

**Contexto comercial de las compras.** La enorme mayoría de pedidos ocurre en contexto "Regular"; las fechas comerciales (Navidad, Feriado, Black Friday) son una fracción pequeña del volumen total.

![Compras por contexto comercial](images/05_compras_contexto.png)

**Distribución de precios y de tiempos de entrega.** Ambas variables muestran colas largas de outliers que conviene tratar con cuidado al interpretar promedios.

![Distribución de precios](images/06_distribucion_precios.png)

![Distribución del tiempo de entrega](images/07_distribucion_entrega.png)

### 3.3 Patrones identificados

1. **Mercado concentrado** en precios bajos, pocas categorías y el estado de São Paulo.
2. **Alta satisfacción general** (4.02/5) con presencia de outliers logísticos.
3. **Dispersión extrema en la entrega** (máximo de 209 días frente a una mediana de 10): el problema no está en el promedio, sino en la cola de fallas.

---

## 4. Nivel 2 – Analítica diagnóstica: ¿por qué pasó?

### 4.1 La entrega tardía es el factor decisivo

La correlación entre tiempo de entrega y calificación es **negativa (≈ −0.30)**: a más días, menor nota. Pero el efecto se ve con total claridad al separar pedidos a tiempo de pedidos tardíos:

| Estado de la entrega | Review promedio |
|---|---|
| **A tiempo** (`is_late_delivery = 0`) | **4.14** |
| **Tardío** (`is_late_delivery = 1`) | **2.26** |

Una entrega tardía hace caer la satisfacción en **casi 1.9 estrellas**. El boxplot lo confirma: los pedidos a tiempo se concentran en 4–5 estrellas, mientras que los tardíos se desploman hacia 1–2.

![Impacto de la entrega tardía en la satisfacción](images/09_impacto_retraso.png)

Visto pedido a pedido, la relación es la misma tendencia: los tiempos de entrega largos arrastran las calificaciones hacia abajo.

![Tiempo de entrega vs. satisfacción](images/10_entrega_vs_satisfaccion.png)

### 4.2 El retraso concentra las peores notas

Al cruzar el estado de la entrega con la distribución de calificaciones, el contraste es brutal:

| Estado | 1★ | 2★ | 3★ | 4★ | 5★ |
|---|---|---|---|---|---|
| **A tiempo** | 10.3% | 3.1% | 8.2% | 19.5% | **58.9%** |
| **Tardío** | **54.4%** | 8.5% | 10.6% | 10.0% | 16.5% |

Más de la **mitad de los pedidos tardíos reciben 1 estrella**, frente al 10% en los pedidos a tiempo. La insatisfacción no está repartida: se concentra casi por completo en las entregas que fallan.

### 4.3 El precio y el contexto comercial casi no afectan la satisfacción

El ticket promedio apenas varía entre contextos comerciales (Black Friday 187 BRL, Regular 181, Navidad 169, Feriado 168), y las diferencias de rating entre categorías son mínimas. Esto refuerza que **el tipo de producto, el precio y la fecha no son los grandes determinantes de la satisfacción**: lo es la operación de entrega.

![Ticket de compra por contexto comercial](images/08_ticket_por_contexto.png)

### 4.4 Validación de las hipótesis de la PC1

| Hipótesis | Resultado | Evidencia |
|---|---|---|
| **H1 – Precio óptimo** | **Refutada / débil** | El precio tiene relación casi nula con la satisfacción; no hay un rango "óptimo" claro. |
| **H2 – Umbral crítico de entrega** | **Validada** | La satisfacción cae de 4.14 a 2.26 cuando la entrega pasa de "a tiempo" a "tardía". |
| **H3 – Retrasos extremos** | **Validada** | El 54% de los pedidos tardíos recibe 1 estrella; la insatisfacción se concentra ahí. |
| **H4 – Sensibilidad por categoría** | **Refutada / débil** | Las diferencias de rating entre categorías son pequeñas; domina el factor logístico. |
| **H5 – Perfil del cliente insatisfecho** | **Validada (vía modelo)** | Se desarrolla en el Nivel 3: el retraso es el predictor central de la mala experiencia. |

---

## 5. Nivel 3 – Analítica predictiva: ¿qué pasará?

### 5.1 Planteamiento

Dado que la entrega tardía es el principal detonante de la insatisfacción (Nivel 2), el equipo entrenó un modelo que **predice la probabilidad de que un pedido se entregue tarde** (`is_late_delivery`). Anticipar el retraso equivale, en la práctica, a anticipar la mala calificación: es la traducción operativa de la hipótesis H5.

- **Modelo:** Regresión logística con `class_weight='balanced'` (corrige el desbalance: solo el **6.77%** de los pedidos llegan tarde).
- **Base de modelamiento:** 96,478 pedidos entregados.
- **Variables predictoras:** `payment_value`, `payment_installments`, `price`, `freight_value`, `product_weight_g` (numéricas) y `purchase_context`, `purchase_month`, `is_weekend`, `is_holiday`, `is_black_friday`, `is_christmas_period`, `customer_state` (categóricas).
- **Validación:** división 80/20 estratificada.

### 5.2 Métricas de desempeño

| Métrica | Valor | Lectura de negocio |
|---|---|---|
| ROC-AUC | **0.728** | Capacidad moderada-buena de distinguir un pedido que llegará tarde de uno que no. |
| Accuracy | 0.837 | Alta, pero inflada por el desbalance (la mayoría llega a tiempo). |
| Recall (tardíos) | 0.425 | Detecta ~42 de cada 100 retrasos reales: sirve como filtro de alerta temprana. |
| Precision (tardíos) | 0.189 | De los pedidos marcados como riesgo, ~19% terminan siendo tardíos (genera falsos positivos). |
| F1-score | 0.261 | Refleja el compromiso entre cobertura y precisión en un problema desbalanceado. |

![Curva ROC](images/11_curva_roc.png)

![Matriz de confusión](images/12_matriz_confusion.png)

**Interpretación honesta.** El modelo no es un predictor perfecto —su precisión es baja porque los retrasos son eventos raros y difíciles de anticipar solo con variables del pedido—, pero con un AUC de 0.73 **sí ordena bien el riesgo**. Eso es suficiente para su propósito real: **priorizar**. En lugar de vigilar 96 mil pedidos por igual, Olist puede concentrar recursos en el grupo de mayor probabilidad.

### 5.3 Segmentación operativa del riesgo

Fijando el umbral de alto riesgo en **0.631** (el 15% de mayor probabilidad), el modelo separa dos grupos muy distintos:

| Nivel de riesgo | Pedidos | Probabilidad promedio | % tardíos reales |
|---|---|---|---|
| **Alto riesgo** | 14,472 | 0.73 | **18.3%** |
| Riesgo regular | 82,006 | 0.38 | 4.7% |

El grupo de alto riesgo concentra una tasa de retraso casi **4 veces mayor** que el resto: es exactamente donde conviene intervenir.

---

## 6. Nivel 4 – Analítica prescriptiva: ¿qué debe hacer Olist?

### 6.1 El tamaño del problema, en cifras

| Estado de la entrega | Pedidos | Review promedio | % reviews bajas (1–2★) |
|---|---|---|---|
| **Entrega tardía** | 6,381 | 2.27 | **62.4%** |
| **Entrega a tiempo** | 89,451 | 4.29 | 9.3% |

Evitar un retraso mejora la calificación de ese pedido en ~**2.0 estrellas** y reduce su probabilidad de reseña baja en ~**53 puntos porcentuales**. Esa es la palanca que activan los escenarios.

### 6.2 Escenario A – Sistema de alerta temprana con el modelo

**Supuesto:** Olist usa el modelo para vigilar los **14,336** pedidos de alto riesgo y, con seguimiento proactivo, evita el **30%** de los retrasos de ese grupo.

| Indicador | Resultado |
|---|---|
| Pedidos intervenidos | 14,336 |
| Retrasos evitados estimados | ≈ 776 |
| Reviews bajas evitadas estimadas | ≈ 412 |
| Tasa de reviews bajas | 12.80% → **12.37%** |
| Review promedio | 4.16 → 4.17 |

El efecto sobre el promedio global es modesto (porque los retrasos son minoría), pero **recupera a cientos de clientes** que de otro modo habrían quedado insatisfechos.

### 6.3 Escenario B – Intervenir el contexto comercial más crítico

Al medir la tasa de retraso por contexto comercial aparece un patrón claro:

| Contexto | Pedidos | % retrasos | Review promedio |
|---|---|---|---|
| **Black Friday** | 1,139 | **17.3%** | 3.78 |
| Navidad | 4,698 | 8.1% | 4.07 |
| Regular | 88,343 | 6.5% | 4.17 |
| Feriado | 1,652 | 5.6% | 4.10 |

**Black Friday es el contexto con mayor tasa de retraso y peor satisfacción.** El escenario plantea reducir un 25% de sus retrasos reforzando la logística en esa campaña.

> **Matiz analítico:** aunque Black Friday tiene la *tasa* más alta, en *volumen absoluto* la mayoría de los retrasos ocurre en pedidos "Regular" (5,712 retrasos). Una estrategia completa debe atacar ambos: la campaña crítica por intensidad y la operación regular por volumen.

### 6.4 Recomendaciones estratégicas

| # | Recomendación | Acción concreta | Responsable | Indicador de éxito | Hallazgo que la respalda |
|---|---|---|---|---|---|
| 1 | **Atacar la entrega antes que el catálogo o el precio** | Auditar y rediseñar el proceso de los envíos en riesgo de retraso; priorizar SP y los vendedores con peor desempeño logístico | Operaciones / Logística | Reducir el % de pedidos tardíos y subir el review promedio | Nivel 2 (4.14 vs 2.26) y Nivel 4 (62% de reviews bajas en tardíos) |
| 2 | **Implementar alerta temprana con el modelo** | Integrar el modelo predictivo para marcar el grupo de alto riesgo (14.3k pedidos) y activar seguimiento proactivo | Data / Customer Experience | Recall del modelo y nº de clientes en riesgo recuperados | Nivel 3 (alto riesgo 18.3% vs 4.7%) y Escenario A |
| 3 | **Reforzar la logística en Black Friday y la operación regular** | Capacidad logística extra en campañas de alta demanda + mejora continua del grueso "Regular" | Estrategia / Operaciones | % de retrasos en Black Friday y en pedidos Regular | Nivel 4 (Black Friday 17.3% de retrasos; Regular concentra el volumen) |

### 6.5 Trazabilidad dato → insight → decisión

El hilo es consistente: los datos muestran que **la categoría y el precio apenas mueven la satisfacción** y que **la entrega sí, sobre todo cuando falla**; el modelo identifica qué pedidos están en riesgo; los escenarios cuantifican el beneficio de intervenir; y las recomendaciones redirigen la inversión hacia donde el dato indica que está el problema. Cada recomendación cita explícitamente el nivel que la sustenta.

---

## 7. Dashboard ejecutivo

**Entregable:** archivo de Power BI `Version_Final_Dashboard.pbix` (incluido en el repositorio).

> **Pendiente del equipo:** publicar el `.pbix` en Power BI Service (*Publicar → obtener enlace*) y pegar aquí el **link funcional** para cumplir el requisito de la rúbrica. Verificar que el dashboard incluya al menos 4 visualizaciones interactivas, cada una titulada como una pregunta de negocio, por ejemplo:
> 1. ¿Cómo se distribuye la satisfacción de nuestros clientes?
> 2. ¿Cuánto afecta la entrega tardía a la calificación?
> 3. ¿Qué estados y categorías concentran las ventas?
> 4. ¿En qué contexto comercial se concentran los retrasos?

---

## 8. Limitaciones del análisis

- **No se observa recompra real:** el dataset no tiene una variable explícita de retención; la "recompra" de la pregunta de negocio se aproxima de forma indirecta y debe interpretarse con cautela.
- **Sesgo de satisfacción:** las calificaciones se concentran en 4–5 estrellas, lo que genera desbalance de clases y obliga a técnicas específicas (`class_weight='balanced'`).
- **Precisión del modelo:** el retraso es un evento raro y difícil de predecir solo con variables del pedido; el modelo sirve para priorizar, no para garantizar.
- **Variables externas ausentes:** no hay datos de promociones, stock, competencia ni capacidad logística real.
- **Período histórico (2016–2018) y meses incompletos:** los últimos meses tienen datos parciales; las conclusiones aplican al período analizado.

---

## 9. Conclusiones y próximos pasos

En Olist, **un producto triunfa menos por lo que es y más por cómo llega.** La consistencia de la entrega es la variable con mayor relación con la satisfacción; el precio, la categoría y la fecha tienen un rol secundario. El cliente insatisfecho es, sobre todo, un cliente cuyo pedido llegó tarde.

**Si el proyecto continuara:** (1) construir una métrica proxy de recompra a partir de `customer_unique_id` para medir retención real; (2) incorporar datos de capacidad logística y de vendedores para localizar el origen de los retrasos; y (3) llevar el modelo a un sistema de alerta en tiempo real integrado al flujo operativo.

---

## Apéndice A – Bitácora de contribución

| Semana | Camila Torres | Letizia Torres | Marcelo Villafuerte | Pablo Vega |
|---|---|---|---|---|
| 7 | _(completar)_ | _(completar)_ | _(completar)_ | _(completar)_ |
| 8 | … | … | … | … |
| 9 | … | … | … | … |
| 10 | … | … | … | … |
| 11 | … | … | … | … |
| 12 | … | … | … | … |
| 13 | … | … | … | … |
| 14 | … | … | … | … |

> _Cada integrante describe brevemente su aporte semanal (obligatorio según la Sección 7.4 del documento del curso)._

---

## Apéndice B – Uso de Inteligencia Artificial Generativa

Durante la PC2 se utilizó IA generativa como apoyo para **estructurar y redactar el informe** y para **organizar la presentación de resultados**, no para generar el análisis. La carga e integración de datos, el enriquecimiento, el modelado y las visualizaciones fueron desarrollados por el equipo en el notebook `pf_big_data.py` / `PF_Big_data.ipynb`. Todas las cifras de este informe provienen de la ejecución real de ese código sobre `olist_enriched_final.csv`.

| Actividad | Herramienta | Uso | Validación |
|---|---|---|---|
| Redacción y estructura del informe PC2 | _(completar: ChatGPT / Claude)_ | Organizar secciones e integrar los 4 niveles de analítica | Cada cifra se contrastó con la salida del notebook |
| _(completar otras, p. ej. apoyo en código del modelo)_ | _(completar)_ | _(completar)_ | _(completar)_ |

> _Reemplazar los "(completar)" por el uso real de IA. Declararlo con precisión protege la nota: la rúbrica anula la evaluación si detecta un informe generado íntegramente por IA sin aporte del equipo._
