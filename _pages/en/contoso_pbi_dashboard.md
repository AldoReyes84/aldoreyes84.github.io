---
layout: default
lang: en
permalink: /contoso_pbi_dashboard/en/
title: Contoso Power BI Dashboard
---


# Contoso Sales PowerBI Dashboard 

## Structure analysis

Source Model, Contoso Sales Sample for Power BI data Model 

<img width="550" height="393" alt="image" src="https://github.com/user-attachments/assets/090237de-2860-467d-807c-d9399e9ba881" />

At first glance, one might assume that SalesAmount equals UnitPrice × SalesQuantity.

  Check SalesAmount = CALCULATE(SUM('Product'[UnitPrice])*SUM(Sales[SalesQuantity]))

However, when testing this logic, the result exceeds the actual SalesAmount, suggesting that discounts are already factored in. This implies that SalesAmount represents net revenue after applying DiscountAmount.

  SalesAmount Cal = [Check SalesAmount]-Sum(Sales[DiscountAmount])

<img width="500" height="141" alt="image" src="https://github.com/user-attachments/assets/bab0161c-eac4-491d-8fcd-694777031289" />

## Excel Metrics Analysis

<img width="320" height="439" alt="image" src="https://github.com/user-attachments/assets/bf6ecf70-1e95-419d-87e6-50a3c28a3b66" />


## 📘 Metrics Glossary – Contoso Sales Dashboard

This glossary defines the key metrics used in the Contoso Sales analysis. Each entry includes a description, a suggested formula, and a DAX implementation example for Power BI.

### 🧮 Base Metrics (Explicit Measures)

This section defines the foundational metrics used in the Contoso Sales Dashboard. All metrics are implemented as explicit DAX measures to ensure compatibility with Calculation Groups and maintain clarity across visuals.

| Metric              | Description                                                                 | Suggested Formula               | DAX Example |
|---------------------|-----------------------------------------------------------------------------|----------------------------------|-------------|
| **UnitPrice**        | Unit price of the product. Retrieved from the `Product` table for consistency. | `(Product[UnitPrice])` | `UnitPrice = SUM('Product'[UnitPrice])` |
| **UnitCost**         | Unit cost of the product. Retrieved from the `Product` table.              | `(Product[UnitCost])` | `UnitCost = SUM(Product[UnitCost])` |
| **SalesQuantity**    | Total quantity of products sold. Defined as an explicit measure.           | `SUM(Sales[SalesQuantity])`      | `SalesQuantity = SUM(Sales[SalesQuantity])` |
| **ReturnQuantity**   | Total quantity of products returned.                                       | `SUM(Sales[ReturnQuantity])`     | `ReturnQuantity = SUM(Sales[ReturnQuantity])` |
| **DiscountAmount**   | Total discount amount applied to sales.                                    | `SUM(Sales[DiscountAmount])`     | `DiscountAmount = SUM(Sales[DiscountAmount])` |
| **DiscountQuantity** | Quantity of products sold with discount applied.                          | `SUM(Sales[DiscountQuantity])`   | `DiscountQuantity = SUM(Sales[DiscountQuantity])` |

---

#### 🧠 Notes

- All metrics above are defined as **explicit DAX measures** to ensure compatibility with Calculation Groups and advanced Time Intelligence logic.
- Avoid using raw columns directly in visuals or calculations when dynamic logic (e.g., YTD, YoY) is required.
- These measures serve as the foundation for derived metrics such as revenue, cost, and profitability.

### 💰 Revenue and Cost Metrics

| Metric         | Description                                                                 | Suggested Formula                                  | DAX Example |
|----------------|-----------------------------------------------------------------------------|----------------------------------------------------|-------------|
| **NetSales**     | Gross revenue before discounts.                                            | `UnitPrice × SalesQuantity`                        | `NetSales = [SalesAmount]+[DiscountAmount]` |
| **SalesAmount**  | Net revenue after discounts.                                               | `SUM(Sales[SalesAmount])`                      | `SalesAmount = SUM(Sales[SalesAmount]` |
| **ReturnAmount** | Monetary value of returned products.                                       | `SUM(Sales[ReturnAmount]`                       | `ReturnAmount = SUM(Sales[DiscountAmount]` |
| **TotalCost**    | Total cost of sold products (excluding returns).                           | `SUM(Sales[TotalCost]`      | `TotalCost = (SUM(Sales[TotalCost]` |

### 📊 Profitability Metrics

| Metric           | Description                                                               | Suggested Formula                                  | DAX Example |
|------------------|---------------------------------------------------------------------------|----------------------------------------------------|-------------|
| **GrossProfit**     | Gross profit before returns.                                              | `SalesAmount - TotalCost`                          | `GrossProfit = [SalesAmount] - [TotalCost]` |
| **NetProfit**       | Net profit after returns.                                                 | `SalesAmount - TotalCost - ReturnAmount`           | `NetProfit = [SalesAmount] - [TotalCost] - [ReturnAmount]` |
| **GrossMargin %**   | Gross margin as a percentage of sales.                                    | `GrossProfit / SalesAmount`                        | `GrossMargin % = DIVIDE([GrossProfit], [SalesAmount])` |
| **NetMargin %**     | Net margin as a percentage of sales.                                      | `NetProfit / SalesAmount`                          | `NetMargin % = DIVIDE([NetProfit], [SalesAmount])` |

### 📉 Discount and Return Metrics

| Metric             | Description                                                              | Suggested Formula                                  | DAX Example |
|--------------------|--------------------------------------------------------------------------|----------------------------------------------------|-------------|
| **DiscountRate %**   | Discount percentage applied over NetSales.                               | `DiscountAmount / NetSales`                        | `DiscountRate % = DIVIDE([DiscountAmount], [NetSales])` |
| **ReturnRate %**     | Return percentage over quantity sold.                                    | `ReturnQuantity / SalesQuantity`                   | `ReturnRate % = DIVIDE([ReturnQuantity], [SalesQuantity])` |


### ⏱️ Time Intelligence Calculation Group

This section defines reusable time-based transformations using Calculation Groups in Power BI. These allow dynamic application of logic (YTD, MTD, YoY, etc.) to any measure using `SELECTEDMEASURE()`.

| Calculation Item     | Description                                               | Suggested Formula                                  | DAX Example |
|----------------------|-----------------------------------------------------------|----------------------------------------------------|-------------|
| YTD                  | Year-to-date total from Jan 1 to current date.            | `TOTALYTD(SELECTEDMEASURE(), 'Date'[Date])`        | `YTD = TOTALYTD(SELECTEDMEASURE(), 'Calendar'[DateKey])` |
| MTD                  | Month-to-date total from 1st of month to current date.    | `TOTALMTD(SELECTEDMEASURE(), 'Date'[Date])`        | `MTD = TOTALMTD(SELECTEDMEASURE(), 'Calendar'[DateKey])` |
| QTD                  | Quarter-to-date total from start of quarter to today.     | `TOTALQTD(SELECTEDMEASURE(), 'Date'[Date])`        | `QTD = TOTALQTD(SELECTEDMEASURE(), 'Calendar'[DateKey])` |
| YoY                  | Same period last year.                                    | `CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR('Date'[Date]))` | `YoY = CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR('Calendar'[DateKey]))` |
| Previous Month       | Same period in previous month.                            | `CALCULATE(SELECTEDMEASURE(), PREVIOUSMONTH('Date'[Date]))` | `PreviousMonth = CALCULATE(SELECTEDMEASURE(), PREVIOUSMONTH('Calendar'[DateKey]))` |
| YoY % Change         | Year-over-year percentage change.                         | `(Current - LastYear) / LastYear`                  | `YoY % = DIVIDE(SELECTEDMEASURE() - [YoY], [YoY])` |
| MoM % Change         | Month-over-month percentage change.                       | `(Current - PreviousMonth) / PreviousMonth`        | `MoM % = DIVIDE(SELECTEDMEASURE() - [PreviousMonth], [PreviousMonth])` |

---

> **Technical Notes**  
> - Metrics are calculated using data from the `Product` and `Sales` tables.  
> - Ensure consistency between `UnitPrice` and `UnitCost` across tables.  
> - Formulas are adaptable to DAX, SQL, or other BI environments.

  ## Dashboard Desing & Story Telling  



 KIP´s  that could provide us with a simple Overview of the company Sales statement.  
-------------------------------------------------------------------------------------
<img width="500" height="289" alt="image" src="https://github.com/user-attachments/assets/fc621312-15e8-489b-a94b-8334787ec451" />
                                                                                                                                    
 Yearly Table to have a better perspective of the metrics comparing throuh time.                                                   
                                                                                                                                   
 <img width="500" height="186" alt="image" src="https://github.com/user-attachments/assets/ebc830f8-22d7-43d4-b64e-464e8ebfbed0" /> 
                                                                                                                                    
 Histogram for 2013 vs LY Sales                                                                                                     
                                                                                                                                     
 <img width="500" height="124" alt="image" src="https://github.com/user-attachments/assets/d5db401b-2e53-46f1-852e-7e4cf5a82cc4" /> 
                                                                                                                                    
 Time Intelligence Measure Group with YTD, 
                                           
 <img width="500" height="133" alt="image" src="https://github.com/user-attachments/assets/6b9b0eb6-5131-4996-9c7b-f3110f95459c" /> 
                                                                                                                                    
 YOY displays 2012 vs 2011 Sales 
                                 
 <img width="500" height="114" alt="image" src="https://github.com/user-attachments/assets/aa93ef1f-0d67-4c9e-9fc8-75e0cb17519b" /> 
                                                                                                                                    
 YOY% disable the KIP's and table but privides a comparative line for 2012 vs 2013 Sales on our histogram visualization 
                                                                                                                        
 <img width="500" height="289" alt="image" src="https://github.com/user-attachments/assets/9f014f26-abde-4a93-86e1-6d1c5276b67b" /> 
                                                                                                                                     
 Channel, Category and Subcategory Sales Bar Chart interactive filters for the rest of the dashboard. 
                                                                                                      
 <img width="500" height="289" alt="image" src="https://github.com/user-attachments/assets/849279d9-d5c5-450b-aaf5-cdecc3f1dc4e" /> 
                                                                                                                                     
 Bubble Map drilled down by Continent/Contry, interactive for the rest of the dashboard. 
                                                                                         
 <img width="500" height="289" alt="image" src="https://github.com/user-attachments/assets/dd734d8a-3d1e-4ae3-a769-9a78b5b3753b" /> 
                                                                                                                                      
 Most profitable products interactive with the rest of the dashboard
                                                                    
 <img width="500" height="289" alt="image" src="https://github.com/user-attachments/assets/cbdfc629-cda1-410e-9268-0f35a25b0d9e" /> 
                                                                                                                                     


The Story here is, KPI's are showing a healty preformance for 2013 55.79% NetMargin with low ReturnRate and DiscountAmount

But if we take a deeper look into the Data Table we can notice YOY% SalesAmount decreased -15.96% in 2012 and the negative tendance was contenined in 2013 up to -3.33%

Altought the Margins fell it's not a percentage to be concider. Same goes for DiscountRate and ReturnRate is also lower. 

In conclution, Sales has drop significantly vs LY in 2012 and the tendance has been containde in 2013 


