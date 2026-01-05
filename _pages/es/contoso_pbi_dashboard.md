---
layout: default
lang: es
permalink: /contoso_pbi_dashboard/es/
title: Contoso Power BI Dashboard
---

# Dashboard de Ventas Contoso en Power BI

## Análisis de estructura

Modelo fuente: Contoso Sales Sample para Power BI Data Model

<img width="550" height="393" alt="image" src="https://github.com/user-attachments/assets/090237de-2860-467d-807c-d9399e9ba881" />

A primera vista, podría asumirse que **SalesAmount** es igual a **UnitPrice × SalesQuantity**.

  Verificación SalesAmount = CALCULATE(SUM('Product'[UnitPrice])*SUM(Sales[SalesQuantity]))

Sin embargo, al probar esta lógica, el resultado excede el valor real de **SalesAmount**, lo que sugiere que los descuentos ya están considerados. Esto implica que **SalesAmount** representa el ingreso neto después de aplicar **DiscountAmount**.

  SalesAmount Cal = [Check SalesAmount]-Sum(Sales[DiscountAmount])

<img width="500" height="141" alt="image" src="https://github.com/user-attachments/assets/bab0161c-eac4-491d-8fcd-694777031289" />

## Análisis de métricas en Excel

<img width="320" height="439" alt="image" src="https://github.com/user-attachments/assets/bf6ecf70-1e95-419d-87e6-50a3c28a3b66" />

## 📘 Glosario de métricas – Dashboard de Ventas Contoso

Este glosario define las métricas clave utilizadas en el análisis de ventas de Contoso. Cada entrada incluye una descripción, una fórmula sugerida y un ejemplo de implementación en DAX para Power BI.

### 🧮 Métricas base (Medidas explícitas)

Esta sección define las métricas fundamentales utilizadas en el Dashboard de Ventas Contoso. Todas las métricas están implementadas como medidas explícitas en DAX para asegurar compatibilidad con los *Calculation Groups* y mantener claridad en las visualizaciones.

| Métrica              | Descripción                                                                 | Fórmula sugerida               | Ejemplo DAX |
|----------------------|-----------------------------------------------------------------------------|--------------------------------|-------------|
| **UnitPrice**        | Precio unitario del producto. Recuperado de la tabla `Product` para consistencia. | `(Product[UnitPrice])` | `UnitPrice = SUM('Product'[UnitPrice])` |
| **UnitCost**         | Costo unitario del producto. Recuperado de la tabla `Product`.              | `(Product[UnitCost])` | `UnitCost = SUM(Product[UnitCost])` |
| **SalesQuantity**    | Cantidad total de productos vendidos. Definida como una medida explícita.   | `SUM(Sales[SalesQuantity])`    | `SalesQuantity = SUM(Sales[SalesQuantity])` |
| **ReturnQuantity**   | Cantidad total de productos devueltos.                                      | `SUM(Sales[ReturnQuantity])`   | `ReturnQuantity = SUM(Sales[ReturnQuantity])` |
| **DiscountAmount**   | Monto total de descuentos aplicados a las ventas.                          | `SUM(Sales[DiscountAmount])`   | `DiscountAmount = SUM(Sales[DiscountAmount])` |
| **DiscountQuantity** | Cantidad de productos vendidos con descuento aplicado.                     | `SUM(Sales[DiscountQuantity])` | `DiscountQuantity = SUM(Sales[DiscountQuantity])` |

---
#### 🧠 Notas

- Todas las métricas anteriores están definidas como **medidas explícitas en DAX** para asegurar compatibilidad con los *Calculation Groups* y la lógica avanzada de *Time Intelligence*.  
- Evita usar columnas sin procesar directamente en visualizaciones o cálculos cuando se requiera lógica dinámica (ej. YTD, YoY).  
- Estas medidas sirven como base para métricas derivadas como ingresos, costos y rentabilidad.  

### 💰 Métricas de ingresos y costos

| Métrica         | Descripción                                                                 | Fórmula sugerida                                  | Ejemplo DAX |
|-----------------|-----------------------------------------------------------------------------|--------------------------------------------------|-------------|
| **NetSales**    | Ingreso bruto antes de descuentos.                                          | `UnitPrice × SalesQuantity`                      | `NetSales = [SalesAmount]+[DiscountAmount]` |
| **SalesAmount** | Ingreso neto después de descuentos.                                         | `SUM(Sales[SalesAmount])`                        | `SalesAmount = SUM(Sales[SalesAmount]` |
| **ReturnAmount**| Valor monetario de los productos devueltos.                                | `SUM(Sales[ReturnAmount]`                        | `ReturnAmount = SUM(Sales[DiscountAmount]` |
| **TotalCost**   | Costo total de los productos vendidos (excluyendo devoluciones).            | `SUM(Sales[TotalCost]`                           | `TotalCost = (SUM(Sales[TotalCost]` |

### 📊 Métricas de rentabilidad

| Métrica            | Descripción                                                              | Fórmula sugerida                                  | Ejemplo DAX |
|--------------------|--------------------------------------------------------------------------|--------------------------------------------------|-------------|
| **GrossProfit**    | Ganancia bruta antes de devoluciones.                                    | `SalesAmount - TotalCost`                        | `GrossProfit = [SalesAmount] - [TotalCost]` |
| **NetProfit**      | Ganancia neta después de devoluciones.                                   | `SalesAmount - TotalCost - ReturnAmount`         | `NetProfit = [SalesAmount] - [TotalCost] - [ReturnAmount]` |
| **GrossMargin %**  | Margen bruto como porcentaje de las ventas.                              | `GrossProfit / SalesAmount`                      | `GrossMargin % = DIVIDE([GrossProfit], [SalesAmount])` |
| **NetMargin %**    | Margen neto como porcentaje de las ventas.                               | `NetProfit / SalesAmount`                        | `NetMargin % = DIVIDE([NetProfit], [SalesAmount])` |

### 📉 Métricas de descuentos y devoluciones

| Métrica             | Descripción                                                             | Fórmula sugerida                                  | Ejemplo DAX |
|---------------------|-------------------------------------------------------------------------|--------------------------------------------------|-------------|
| **DiscountRate %**  | Porcentaje de descuento aplicado sobre NetSales.                        | `DiscountAmount / NetSales`                      | `DiscountRate % = DIVIDE([DiscountAmount], [NetSales])` |
| **ReturnRate %**    | Porcentaje de devoluciones sobre la cantidad vendida.                   | `ReturnQuantity / SalesQuantity`                 | `ReturnRate % = DIVIDE([ReturnQuantity], [SalesQuantity])` |

### ⏱️ Grupo de cálculos de inteligencia temporal

Esta sección define transformaciones reutilizables basadas en tiempo usando *Calculation Groups* en Power BI. Estas permiten aplicar dinámicamente lógica (YTD, MTD, YoY, etc.) a cualquier medida utilizando `SELECTEDMEASURE()`.

| Elemento de cálculo | Descripción                                               | Fórmula sugerida                                  | Ejemplo DAX |
|---------------------|-----------------------------------------------------------|--------------------------------------------------|-------------|
| YTD                 | Total acumulado desde el 1 de enero hasta la fecha actual.| `TOTALYTD(SELECTEDMEASURE(), 'Date'[Date])`      | `YTD = TOTALYTD(SELECTEDMEASURE(), 'Calendar'[DateKey])` |
| MTD                 | Total acumulado desde el inicio del mes hasta la fecha actual.| `TOTALMTD(SELECTEDMEASURE(), 'Date'[Date])`   | `MTD = TOTALMTD(SELECTEDMEASURE(), 'Calendar'[DateKey])` |
| QTD                 | Total acumulado desde el inicio del trimestre hasta hoy.  | `TOTALQTD(SELECTEDMEASURE(), 'Date'[Date])`      | `QTD = TOTALQTD(SELECTEDMEASURE(), 'Calendar'[DateKey])` |
| YoY                 | Mismo periodo del año anterior.                           | `CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR('Date'[Date]))` | `YoY = CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR('Calendar'[DateKey]))` |
| Previous Month      | Mismo periodo del mes anterior.                           | `CALCULATE(SELECTEDMEASURE(), PREVIOUSMONTH('Date'[Date]))` | `PreviousMonth = CALCULATE(SELECTEDMEASURE(), PREVIOUSMONTH('Calendar'[DateKey]))` |
| YoY % Change        | Cambio porcentual año contra año.                         | `(Current - LastYear) / LastYear`                | `YoY % = DIVIDE(SELECTEDMEASURE() - [YoY], [YoY])` |
| MoM % Change        | Cambio porcentual mes contra mes.                         | `(Current - PreviousMonth) / PreviousMonth`      | `MoM % = DIVIDE(SELECTEDMEASURE() - [PreviousMonth], [PreviousMonth])` |


> **Notas técnicas**  
> - Las métricas se calculan utilizando datos de las tablas `Product` y `Sales`.  
> - Asegurar consistencia entre `UnitPrice` y `UnitCost` en todas las tablas.  
> - Las fórmulas son adaptables a DAX, SQL u otros entornos de BI.  

## Diseño del Dashboard y Storytelling  

| KPI´s que pueden brindarnos una visión general simple del estado de ventas de la compañía. |  
|--------------------------------------------------------------------------------------------|  
|<img width="500" height="289" alt="image" src="https://github.com/user-attachments/assets/fc621312-15e8-489b-a94b-8334787ec451" />|  
|                                                                                            |  
| Tabla anual para tener una mejor perspectiva de las métricas comparadas a lo largo del tiempo. |  
|                                                                                            |  
| <img width="500" height="186" alt="image" src="https://github.com/user-attachments/assets/ebc830f8-22d7-43d4-b64e-464e8ebfbed0" /> |  
|                                                                                            |  
| Histograma de Ventas 2013 vs LY (año anterior).                                            |  
|                                                                                            |  
| <img width="500" height="124" alt="image" src="https://github.com/user-attachments/assets/d5db401b-2e53-46f1-852e-7e4cf5a82cc4" /> |  
|                                                                                            |  
| Grupo de medidas de Inteligencia Temporal con YTD.                                         |  
|                                                                                            |  
| <img width="500" height="133" alt="image" src="https://github.com/user-attachments/assets/6b9b0eb6-5131-4996-9c7b-f3110f95459c" /> |  
|                                                                                            |  
| YOY muestra Ventas 2012 vs 2011.                                                           |  
|                                                                                            |  
| <img width="500" height="114" alt="image" src="https://github.com/user-attachments/assets/aa93ef1f-0d67-4c9e-9fc8-75e0cb17519b" /> |  
|                                                                                            |  
| YOY% desactiva los KPI´s y la tabla, pero proporciona una línea comparativa para Ventas 2012 vs 2013 en la visualización del histograma. |  
|                                                                                            |  
| <img width="500" height="289" alt="image" src="https://github.com/user-attachments/assets/9f014f26-abde-4a93-86e1-6d1c5276b67b" /> |  
|                                                                                            |  
| Gráfico de barras de Ventas por Canal, Categoría y Subcategoría con filtros interactivos para el resto del dashboard. |  
|                                                                                            |  
| <img width="500" height="289" alt="image" src="https://github.com/user-attachments/assets/849279d9-d5c5-450b-aaf5-cdecc3f1dc4e" /> |  
|                                                                                            |  
| Mapa de burbujas con desglose por Continente/País, interactivo para el resto del dashboard. |  
|                                                                                            |  
| <img width="500" height="289" alt="image" src="https://github.com/user-attachments/assets/dd734d8a-3d1e-4ae3-a769-9a78b5b3753b" /> |  
|                                                                                            |  
| Productos más rentables, interactivos con el resto del dashboard.                          |  
| <img width="500" height="289" alt="image" src="https://github.com/user-attachments/assets/cbdfc629-cda1-410e-9268-0f35a25b0d9e" /> |  
|                                                                                            |  

La historia aquí es que los KPI´s muestran un desempeño saludable en 2013 con un **NetMargin de 55.79%**, acompañado de una baja tasa de devoluciones (**ReturnRate**) y un bajo monto de descuentos (**DiscountAmount**).

Pero si observamos más de cerca la tabla de datos, podemos notar que el **YOY% SalesAmount** disminuyó **-15.96% en 2012**, y la tendencia negativa fue contenida en 2013 hasta **-3.33%**.

Aunque los márgenes cayeron, no es un porcentaje significativo a considerar. Lo mismo ocurre con la tasa de descuentos (**DiscountRate**) y la tasa de devoluciones (**ReturnRate**), que también son más bajas.

En conclusión, las ventas cayeron significativamente frente al año anterior en 2012, y la tendencia fue contenida en 2013.
                                                                                           
