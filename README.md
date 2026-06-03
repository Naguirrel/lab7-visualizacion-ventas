# Lab 7 — Visualización de Datos · RetailMax

**Curso:** CC3088 Bases de Datos 1 — Universidad del Valle de Guatemala  
**Área de negocio:** Ventas  
**Ciclo:** 1, 2026

---

## Video de presentación

> **https://youtu.be/CEzIMJ4tyAs?si=0gyU5Fa3wwnEIsuA**

---

## Cómo levantar el ambiente

```bash
docker compose up -d
```

Esperar ~60 segundos a que Metabase inicialice, luego abrir **http://localhost:3000**.

**Credenciales de calificación:**

| Campo | Valor |
|-------|-------|
| Correo | calificar@uvg.edu.gt |
| Contraseña | secret123+ |

> El dashboard aparece precargado al abrir Metabase. No se requiere ningún paso adicional.

---

## Estructura del repositorio

```
├── docker-compose.yml       # Ambiente completo (PostgreSQL + Metabase)
├── DDL.sql                  # Esquema de la base de datos
├── DATA.sql                 # Datos de prueba
├── metabase-data/           # Volumen persistido con el dashboard construido
├── informe.pdf              # Documentación completa de los 12+ indicadores
└── README.md
```

---

## Dashboard

El dashboard **"Dashboard Ventas"** contiene 2 tabs con 6 indicadores cada uno.

---

### Tab 1 — Rendimiento de Ventas

---

#### KPI 1 — Ventas Totales

**Qué representa:**  
Total de ingresos generados por todos los pedidos completados.

**Importancia para el área:**  
Permite medir el desempeño general del área de ventas y evaluar el volumen total de ingresos del negocio.

**Visualización:** Big Number — permite visualizar rápidamente el indicador financiero principal del dashboard.

![KPI 1 - Ventas Totales](images/kpi1-ventas-totales.png)

**Consulta SQL:**
```sql
SELECT
    ROUND(SUM(
        dp.cantidad * dp.precio_unitario * (1 - dp.descuento / 100.0)
    ), 2) AS ventas_totales
FROM detalle_pedido dp
JOIN pedido p
    ON dp.id_pedido = p.id_pedido
WHERE p.estado = 'completado';
```

---

#### KPI 2 — Ventas por Mes

**Qué representa:**  
Evolución mensual de las ventas generadas por pedidos completados.

**Importancia para el área:**  
Permite identificar tendencias, crecimiento, caídas y comportamientos estacionales en las ventas.

**Visualización:** Line Chart — ideal para visualizar cambios y tendencias a lo largo del tiempo.

![KPI 2 - Ventas por Mes](images/kpi2-ventas-mes.png)

**Consulta SQL:**
```sql
SELECT
    DATE_TRUNC('month', p.fecha) AS mes,
    ROUND(SUM(
        dp.cantidad * dp.precio_unitario * (1 - dp.descuento / 100.0)
    ), 2) AS ventas
FROM detalle_pedido dp
JOIN pedido p
    ON dp.id_pedido = p.id_pedido
WHERE p.estado = 'completado'
GROUP BY mes
ORDER BY mes;
```

---

#### KPI 3 — Top 10 Productos Más Vendidos

**Qué representa:**  
Los productos que generan mayores ingresos dentro de los pedidos completados.

**Importancia para el área:**  
Identifica qué productos tienen mejor desempeño comercial y cuáles generan más ingresos para la empresa.

**Visualización:** Bar Chart — facilita comparar visualmente el rendimiento entre distintos productos.

![KPI 3 - Top 10 Productos](images/kpi3-top-productos.png)

**Consulta SQL:**
```sql
SELECT
    pr.nombre AS producto,
    ROUND(SUM(
        dp.cantidad * dp.precio_unitario * (1 - dp.descuento / 100.0)
    ), 2) AS ingresos
FROM detalle_pedido dp
JOIN pedido p
    ON dp.id_pedido = p.id_pedido
JOIN producto pr
    ON dp.id_producto = pr.id_producto
WHERE p.estado = 'completado'
GROUP BY pr.nombre
ORDER BY ingresos DESC
LIMIT 10;
```

---

#### KPI 4 — Valor Promedio por Venta

**Qué representa:**  
Valor promedio generado por cada pedido completado. Este indicador calcula primero el total de cada venta y luego obtiene el promedio general de todas las ventas completadas.

**Importancia para el área:**  
Permite conocer cuánto ingresa la empresa, en promedio, por cada transacción. Esto ayuda a evaluar si las estrategias comerciales están aumentando el valor de compra de los clientes y no solo la cantidad de ventas.

**Visualización:** Big Number — permite visualizar rápidamente un valor promedio clave para medir el rendimiento comercial de cada venta.

![KPI 4 - Valor Promedio por Venta](images/kpi4-promedio-venta.png)

**Consulta SQL:**
```sql
SELECT 
    ROUND(AVG(total_venta), 2) AS valor_promedio_por_venta
FROM (
    SELECT 
        p.id_pedido,
        SUM(dp.cantidad * dp.precio_unitario * (1 - dp.descuento / 100.0)) AS total_venta
    FROM pedido p
    JOIN detalle_pedido dp 
        ON p.id_pedido = dp.id_pedido
    WHERE p.estado = 'completado'
    GROUP BY p.id_pedido
) ventas_por_pedido;
```

---

#### KPI 5 — Volumen de Pedidos Mensual

**Qué representa:**  
Cantidad de pedidos completados por cada mes. A diferencia del indicador de ventas por mes, este KPI no mide ingresos, sino el número de transacciones realizadas.

**Importancia para el área:**  
Permite analizar si el movimiento comercial aumenta o disminuye a lo largo del tiempo. También ayuda a interpretar si los ingresos crecen porque hay más pedidos o porque cada venta tiene un valor promedio más alto.

**Visualización:** Line Chart — ideal para observar la evolución mensual del volumen de pedidos e identificar tendencias, aumentos o caídas en la actividad comercial.

![KPI 5 - Volumen de Pedidos Mensual](images/kpi5-pedidos-mensuales.png)

**Consulta SQL:**
```sql
SELECT 
    DATE_TRUNC('month', p.fecha) AS mes,
    COUNT(p.id_pedido) AS total_pedidos
FROM pedido p
WHERE p.estado = 'completado'
GROUP BY DATE_TRUNC('month', p.fecha)
ORDER BY mes;
```

---

#### KPI 6 — Ventas por Categoría de Producto

**Qué representa:**  
Ingresos generados por cada categoría de producto dentro de los pedidos completados.

**Importancia para el área:**  
Permite identificar qué categorías generan más ingresos para la empresa. Esta información ayuda a priorizar inventario, promociones y estrategias comerciales según las categorías con mejor desempeño.

**Visualización:** Bar Chart — facilita comparar visualmente el rendimiento de cada categoría de producto y reconocer cuáles aportan más ingresos.

![KPI 6 - Ventas por Categoría de Producto](images/kpi6-ventas-categoria.png)

**Consulta SQL:**
```sql
SELECT
    c.nombre AS categoria,
    ROUND(SUM(dp.cantidad * dp.precio_unitario * (1 - dp.descuento / 100.0)), 2) AS ingresos
FROM pedido p
JOIN detalle_pedido dp 
    ON p.id_pedido = dp.id_pedido
JOIN producto pr 
    ON dp.id_producto = pr.id_producto
JOIN categoria c
    ON pr.id_categoria = c.id_categoria
WHERE p.estado = 'completado'
GROUP BY c.nombre
ORDER BY ingresos DESC;
```

---

### Tab 2 — Análisis de Clientes y Canales

---

#### KPI 1 — Clientes activos por mes

**Qué representa:**  
La cantidad de clientes que completaron al menos una compra en el mes más reciente, comparado con el mesa anterior.
**Importancia para el área:**  
Permite que la empresa pueda monitorear que la cantidad de clientes generando transacciones esté creciendo, puesto que la caida de los clientes podria ayudar a identificar de forma temprana algún problema que más adelante pueda afectar los ingresos.
**Visualización:** Trend (Big Number) — permite ver de un vistazo la métrica más crítica del tab con contexto del mes anterior.

![KPI 7 - Kpi7-clientes-mes](images/kpi7-clientes-mes.png)

**Consulta SQL:**
```sql
SELECT 
DATE_TRUNC('month', p.fecha) as mes,
COUNT(DISTINCT p.id_cliente) as clientes_activos
FROM pedido p
WHERE p.estado = 'completado'
GROUP BY mes
ORDER BY mes;
```

---

#### KPI 2 — Segmento de Cliente por Canal

**Qué representa:**  
Cuántos clientes de cada segmento (VIP, regular, nuevo) compran en cada canal, mostrando si el perfil del cliente varía según el canal de compra.

**Importancia para el área:**  
Para saber si los clientes VIP prefieren comprar en tienda física u online, ya que eso puede ser útil para definir dónde enfocar promociones, atención personalizada y recursos comerciales por canal.

**Visualización:** Row Chart - porque permite comparar dos dimensiones simultáneas (canal y segmento) de forma compacta y clara.

![kpi8-segmento-por-canal](images/kpi8-segmento-por-canal.png)

**Consulta SQL:**
```sql
SELECT
c.segmento, 
p.canal,
COUNT(DISTINCT p.id_cliente) AS clientes
FROM pedido p 
JOIN cliente c ON c.id_cliente = p.id_cliente
WHERE p.estado = 'completado'
GROUP BY p.canal, c.segmento
ORDER BY p.canal, clientes;
```

---

#### KPI 3 — ARPU mensual por segmento

**Qué representa:**  
El ingreso promedio generado por cada cliente activo en un mes, desglosado por segmento (VIP, regular, nuevo).

**Importancia para el área:**  
Permite que se pueda conocer cuánto de ingreso nos genera cada tipo de cliente mensualmente para priorizar esfuerzos comerciales y detectar si algún segmento está perdiendo valor a lo largo del tiempo.

**Visualización:** Line Chart — porque es la mejor opción para comparar la evolución mensual de múltiples segmentos simultáneamente en el tiempo.

![KPI 9 - ARPU mensual por segmento](images/kpi9-arpu-mensual.png)

**Consulta SQL:**
```sql
SELECT 
DATE_TRUNC('month', p.fecha) AS mes,
c.segmento,
ROUND(
SUM(dp.cantidad * dp.precio_unitario *(1- (dp.descuento/100))) / COUNT(DISTINCT p.id_cliente), 2
) AS arpu
FROM pedido p 
JOIN detalle_pedido dp ON p.id_pedido = dp.id_pedido
JOIN cliente c ON p.id_cliente = c.id_cliente
WHERE p.estado = 'completado'
GROUP BY mes, c.segmento
ORDER BY mes, c.segmento;
```
---

#### KPI 4 — Ticket Promedio por Segmento de Cliente

**Qué representa:**  
Valor promedio de cada pedido completado, agrupado por segmento de cliente (VIP, regular, nuevo).

**Importancia para el área:**  
Permite identificar cuánto gasta en promedio cada tipo de cliente y priorizar esfuerzos de retención y upselling en los segmentos de mayor valor.

**Visualización:** Bar Chart — ideal para comparar valores discretos entre categorías.

![KPI 4 - Ticket Promedio por Segmento](images/kpi10-ticket-promedio.png)

**Consulta SQL:**
```sql
SELECT
    c.segmento,
    COUNT(DISTINCT p.id_pedido) AS total_pedidos,
    ROUND(
        AVG(sub.valor_pedido)::numeric, 2
    ) AS ticket_promedio
FROM cliente c
JOIN pedido p ON c.id_cliente = p.id_cliente
JOIN (
    SELECT
        id_pedido,
        SUM(cantidad * precio_unitario * (1 - descuento / 100.0)) AS valor_pedido
    FROM detalle_pedido
    GROUP BY id_pedido
) sub ON p.id_pedido = sub.id_pedido
WHERE p.estado = 'completado'
GROUP BY c.segmento
ORDER BY ticket_promedio DESC;
```

---

#### KPI 5 — Tendencia de Ventas Mensual con Crecimiento MoM

**Qué representa:**  
Ingresos netos por mes y el porcentaje de crecimiento respecto al mes anterior (Month over Month).

**Importancia para el área:**  
Permite detectar estacionalidad, tendencias de aceleración o desaceleración, y meses críticos para anticipar acciones comerciales.

**Visualización:** Line Chart — muestra la evolución temporal; el % de crecimiento añade una segunda dimensión de análisis sobre el mismo gráfico.

![KPI 5 - Tendencia MoM](images/kpi11-tendencia-mom.png)

**Consulta SQL:**
```sql
WITH ventas_mensuales AS (
    SELECT
        DATE_TRUNC('month', p.fecha) AS mes,
        SUM(dp.cantidad * dp.precio_unitario * (1 - dp.descuento / 100.0)) AS ingresos
    FROM pedido p
    JOIN detalle_pedido dp ON p.id_pedido = dp.id_pedido
    WHERE p.estado = 'completado'
    GROUP BY DATE_TRUNC('month', p.fecha)
)
SELECT
    TO_CHAR(mes, 'YYYY-MM') AS mes,
    ROUND(ingresos::numeric, 2) AS ingresos,
    ROUND(
        (
            (ingresos - LAG(ingresos) OVER (ORDER BY mes))
            / NULLIF(LAG(ingresos) OVER (ORDER BY mes), 0)
        ) * 100,
        2
    ) AS crecimiento_pct
FROM ventas_mensuales
ORDER BY mes;
```

---

#### KPI 6 — Participación de Ingresos por Canal de Venta

**Qué representa:**  
Ingresos totales y porcentaje de contribución de cada canal de venta (tienda física vs. online).

**Importancia para el área:**  
Informa decisiones de inversión entre canales: si el canal online crece, hay argumento para ampliar capacidad digital; si la tienda física domina, para optimizar operaciones presenciales.

**Visualización:** Pie/Donut Chart — comunica composición proporcional de forma visual e inmediata.

![KPI 6 - Participación por Canal](images/kpi12-canal-venta.png)

**Consulta SQL:**
```sql
WITH ingresos_canal AS (
    SELECT
        p.canal,
        SUM(dp.cantidad * dp.precio_unitario * (1 - dp.descuento / 100.0)) AS ingresos
    FROM pedido p
    JOIN detalle_pedido dp ON p.id_pedido = dp.id_pedido
    WHERE p.estado = 'completado'
    GROUP BY p.canal
)
SELECT
    canal,
    ROUND(ingresos::numeric, 2) AS ingresos,
    ROUND(
        (ingresos / SUM(ingresos) OVER ()) * 100,
        2
    ) AS participacion_pct
FROM ingresos_canal
ORDER BY ingresos DESC;
```
