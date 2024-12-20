# Financial Reporting Project - PowerBI Requirements Document

## 1. Project Overview
This document outlines the comprehensive requirements for the PowerBI build of our financial reporting project.

## 2. Source Systems
- Microsoft SQL Server Database

## 3. Data Sources
### 3.1 Tables
1. fact_financial
2. Dim_GLCode
3. Dim_Department

### 3.2 Sample Data
sql
-- Sample data for fact_financial
SELECT TOP 5 * FROM fact_financial;

-- Sample data for Dim_GLCode
SELECT TOP 5 * FROM Dim_GLCode;

-- Sample data for Dim_Department
SELECT TOP 5 * FROM Dim_Department;


## 4. Sample Queries
sql
-- Query to get total revenue by department
SELECT d.DepartmentName, SUM(f.Amount) as TotalRevenue
FROM fact_financial f
JOIN Dim_Department d ON f.DepartmentID = d.DepartmentID
WHERE f.GLCodeID IN (SELECT GLCodeID FROM Dim_GLCode WHERE Category = 'Revenue')
GROUP BY d.DepartmentName;

-- Query to get labor utilization
SELECT d.DepartmentName,
       SUM(CASE WHEN g.Category = 'Labor' THEN f.Amount ELSE 0 END) as LaborCost,
       SUM(f.Amount) as TotalCost,
       (SUM(CASE WHEN g.Category = 'Labor' THEN f.Amount ELSE 0 END) / SUM(f.Amount)) * 100 as LaborUtilizationPercentage      
FROM fact_financial f
JOIN Dim_Department d ON f.DepartmentID = d.DepartmentID
JOIN Dim_GLCode g ON f.GLCodeID = g.GLCodeID
GROUP BY d.DepartmentName;


## 5. DAX Measures

// Total Revenue
Total Revenue = SUMX(fact_financial, IF(RELATED(Dim_GLCode[Category]) = "Revenue", fact_financial[Amount], 0))

// Labor Utilization
Labor Cost = SUMX(fact_financial, IF(RELATED(Dim_GLCode[Category]) = "Labor", fact_financial[Amount], 0))
Total Cost = SUM(fact_financial[Amount])
Labor Utilization % = DIVIDE([Labor Cost], [Total Cost], 0)

// YoY Growth
Revenue LY = CALCULATE([Total Revenue], SAMEPERIODLASTYEAR(Dim_Date[Date]))
YoY Growth % = DIVIDE([Total Revenue] - [Revenue LY], [Revenue LY], 0)


## 6. Visual Layout
### 6.1 P&L Report Components
1. Revenue Overview
   - Total Revenue (Card visual)
   - Revenue by Departent (Bar chart)
   - Revenue Trend (Line chart)

2. Cost Analysis
   - Total Cost (Card visual)
   - Cost Breakdown by Category (Pie chart)
   - Cost Trend (Line chart)

3. Profitability
   - Gross Profit
   (Card visual)
   - Profit Margin % (Gauge visual)
   - Profit by Department (Bar chart)

4. Labor Utilization
   - Labor Utilization % (Card visual)
   - Labor Cost vs Total Cost (Stacked bar chart)
   - Labor Utilization Trend (Line chart)

### 6.2 Formatting Requirements
- Use corporate color scheme (primary colors: #007bff, #28a745, #dc3545)
- Consistent font usage across all visuals (Recommended: Arial or Calibri)
- Clear and concise titles for each visual
- Include data labels where appropriate
- Use slicers for date ranges and departments

## 7. Interactivity Requirements
- Cross-filtering between visuals
- Drill-down capability in hierarchical data (e.g., Department > Sub-department)
- Tooltips with additional details on hover

## 8. Refresh and Data Update Requirements
- Daily automated refresh of the report
- Last refresh date/time to be displayed on the report

## 9. Security and Access Control
- Row-level security based on user departments
- Different views for executives and department managers

## 10. Performance Considerations
- Optimize DAX measures for quick calculations
- Use aggregations where possible to improve query performance
- Limit visuals per page to ensure quick loading times
