# DAX Measures Library

A categorized library of DAX measures for the Sales Performance star schema (see [../model/data-model.md](../model/data-model.md)). Each section groups measures by purpose, with the underlying business need noted above the code block.

## Base Measures
Core additive measures that everything else builds on.

```dax
Total Sales = SUM(FactSales[SalesAmount])

Total Cost = SUM(FactSales[CostAmount])

Total Profit = [Total Sales] - [Total Cost]

Profit Margin % = DIVIDE([Total Profit], [Total Sales], 0)

Total Quantity = SUM(FactSales[Quantity])
```

## Time Intelligence
Relies on DimDate being marked as the model's official Date Table.

```dax
Sales YTD = 
TOTALYTD([Total Sales], DimDate[Date])

Sales PY = 
CALCULATE([Total Sales], SAMEPERIODLASTYEAR(DimDate[Date]))

Sales YoY % = 
DIVIDE([Total Sales] - [Sales PY], [Sales PY], 0)

Sales MTD = 
TOTALMTD([Total Sales], DimDate[Date])

Rolling 3-Month Sales = 
CALCULATE(
    [Total Sales],
    DATESINPERIOD(DimDate[Date], MAX(DimDate[Date]), -3, MONTH)
)
```

## Ranking & Top-N
Used for leaderboards and top-performer visuals.

```dax
Product Sales Rank = 
RANKX(ALL(DimProduct[ProductName]), [Total Sales], , DESC)

Top 10 Products Sales = 
CALCULATE(
    [Total Sales],
    TOPN(10, ALL(DimProduct[ProductName]), [Total Sales], DESC)
)
```

## Running Totals
Cumulative sales that respect whatever filters/slicers are currently selected.

```dax
Running Total Sales = 
CALCULATE(
    [Total Sales],
    FILTER(
        ALLSELECTED(DimDate[Date]),
        DimDate[Date] <= MAX(DimDate[Date])
    )
)
```

## Customer Analysis
Measures supporting customer-level KPIs and new customer acquisition tracking.

```dax
Distinct Customers = DISTINCTCOUNT(FactSales[CustomerKey])

Average Order Value = DIVIDE([Total Sales], DISTINCTCOUNT(FactSales[SalesID]), 0)

New Customers = 
CALCULATE(
    DISTINCTCOUNT(DimCustomer[CustomerKey]),
    FILTER(
        DimCustomer,
        DimCustomer[SignupDate] >= MIN(DimDate[Date]) &&
        DimCustomer[SignupDate] <= MAX(DimDate[Date])
    )
)
```

## Variance vs Target
Assumes a separate Sales Target measure or table exists in the model.

```dax
Sales vs Target % = 
DIVIDE([Total Sales] - [Sales Target], [Sales Target], 0)
```
