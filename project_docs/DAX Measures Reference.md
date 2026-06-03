# DAX Performance Measures — Rhenus Air & Ocean (SA Division)
## logistics_co demo | Power BI Semantic Model

> **How to use**: For each measure below, go to Power BI Desktop → select the target table →
> **New Measure** → paste the DAX expression. Use **Display Folder** values to organize in Model View.
>
> **Recommended setup**: Create a dedicated measure table for each category by going to
> **Enter Data** → name the table (e.g., `_Measures - Ocean`) → Load → then create measures on that table.
> Alternatively, all measures can be added to the existing `Measures (2)` table using Display Folders.

---

## 🚢 OCEAN FREIGHT MEASURES
**Target Table**: `_Measures - Ocean` (or `Measures (2)` with Display Folder = `Ocean`)

---

### 1. Total Demurrage Cost
**Display Folder**: `Ocean\Costs`  
**Format**: Currency (R)  
**Description**: Sum of all container demurrage penalties incurred when containers dwell beyond free days at port terminals.
```
Total Demurrage Cost = 
SUM( Fact_Ocean_Shipments[DemurrageCost] )
```

---

### 2. Total Detention Cost
**Display Folder**: `Ocean\Costs`  
**Format**: Currency (R)  
**Description**: Sum of all container detention penalties incurred when containers are returned late after gate-out.
```
Total Detention Cost = 
SUM( Fact_Ocean_Shipments[DetentionCost] )
```

---

### 3. Total D&D Cost
**Display Folder**: `Ocean\Costs`  
**Format**: Currency (R)  
**Description**: Combined Demurrage and Detention costs — the total penalty exposure from port and container delays.
```
Total D&D Cost = 
[Total Demurrage Cost] + [Total Detention Cost]
```

---

### 4. Avg Dwell Days
**Display Folder**: `Ocean\Operational`  
**Format**: `0.0` (1 decimal)  
**Description**: Average number of days containers dwell at port terminals across all ocean shipments, dynamically respecting the carrier mode context.
```
Avg Dwell Days = 
VAR SelectedModes = VALUES( Dim_Carriers[ModeOfTransport] )
RETURN
    IF(
        "Ocean" IN SelectedModes,
        AVERAGE( Fact_Ocean_Shipments[ActualDwellDays] ),
        BLANK()
    )
```

---

### 5. Containers Exceeding Free Days %
**Display Folder**: `Ocean\Performance`  
**Format**: Percentage `0.0%`  
**Description**: Percentage of ocean shipments where the actual dwell time exceeded the carrier's free-day allowance, triggering demurrage charges.
```
Containers Exceeding Free Days % = 
VAR TotalContainers = COUNTROWS( Fact_Ocean_Shipments )
VAR ExceededContainers = 
    CALCULATE(
        COUNTROWS( Fact_Ocean_Shipments ),
        Fact_Ocean_Shipments[ActualDwellDays] > Fact_Ocean_Shipments[FreeDaysAllowed]
    )
RETURN
    DIVIDE( ExceededContainers, TotalContainers, 0 )
```

---

### 6. Demurrage Per Container
**Display Folder**: `Ocean\Costs`  
**Format**: Currency (R)  
**Description**: Average demurrage cost per ocean shipment — useful for benchmarking per-container penalty exposure.
```
Demurrage Per Container = 
DIVIDE(
    [Total Demurrage Cost],
    COUNTROWS( Fact_Ocean_Shipments ),
    0
)
```

---

### 7. Ocean Shipment Count
**Display Folder**: `Ocean\Volume`  
**Format**: `#,0` (whole number)  
**Description**: Total count of ocean freight shipment transactions in the current filter context.
```
Ocean Shipment Count = 
COUNTROWS( Fact_Ocean_Shipments )
```

---

### 8. Tiered Demurrage Rate
**Display Folder**: `Ocean\Performance`  
**Format**: Currency (R) `"R"#,0.00`  
**Description**: Effective daily demurrage rate calculated as total demurrage cost divided by total excess days beyond the free-day allowance. Benchmark against the R1,650/day contractual rate.
```
Tiered Demurrage Rate = 
VAR ExcessDays = 
    SUMX(
        Fact_Ocean_Shipments,
        MAX( Fact_Ocean_Shipments[ActualDwellDays] - Fact_Ocean_Shipments[FreeDaysAllowed], 0 )
    )
RETURN
    DIVIDE( [Total Demurrage Cost], ExcessDays, 0 )
```

---

### 9. Port Congestion Rate %
**Display Folder**: `Ocean\Performance`  
**Format**: Percentage `0.0%`  
**Description**: Percentage of ocean shipments experiencing significant port congestion (dwell time exceeding 10 days). This threshold isolates systemic congestion backlogs, particularly at the Port of Durban.
```
Port Congestion Rate % = 
VAR TotalShipments = COUNTROWS( Fact_Ocean_Shipments )
VAR CongestedShipments = 
    CALCULATE(
        COUNTROWS( Fact_Ocean_Shipments ),
        Fact_Ocean_Shipments[ActualDwellDays] > 10
    )
RETURN
    DIVIDE( CongestedShipments, TotalShipments, 0 )
```

---

### 10. Avg Excess Dwell Days
**Display Folder**: `Ocean\Operational`  
**Format**: `0.0` (1 decimal)  
**Description**: Average number of days containers dwell beyond the free-day allowance (only for shipments that exceeded it). Measures the severity of delays, not just frequency.
```
Avg Excess Dwell Days = 
AVERAGEX(
    FILTER(
        Fact_Ocean_Shipments,
        Fact_Ocean_Shipments[ActualDwellDays] > Fact_Ocean_Shipments[FreeDaysAllowed]
    ),
    Fact_Ocean_Shipments[ActualDwellDays] - Fact_Ocean_Shipments[FreeDaysAllowed]
)
```

---
---

## ✈️ AIR FREIGHT MEASURES
**Target Table**: `_Measures - Air` (or `Measures (2)` with Display Folder = `Air`)

---

### 1. Air Shipment Count
**Display Folder**: `Air\Volume`  
**Format**: `#,0` (whole number)  
**Description**: Total count of air freight shipment transactions in the current filter context.
```
Air Shipment Count = 
COUNTROWS( Fact_Air_Shipments )
```

---

### 2. Total Actual Weight (KG)
**Display Folder**: `Air\Physical`  
**Format**: `#,0.00` with suffix ` kg`  
**Description**: Sum of actual cargo weight across all air shipments.
```
Total Actual Weight (KG) = 
SUM( Fact_Air_Shipments[ActualWeightKG] )
```

---

### 3. Total Volume (CBM)
**Display Folder**: `Air\Physical`  
**Format**: `#,0.0000` with suffix ` m³`  
**Description**: Sum of cargo volume (cubic metres) across all air shipments.
```
Total Volume (CBM) = 
SUM( Fact_Air_Shipments[VolumeCBM] )
```

---

### 4. Total Chargeable Weight
**Display Folder**: `Air\Financial`  
**Format**: `#,0.00` with suffix ` kg`  
**Description**: Total chargeable weight across all air shipments, dynamically respecting the carrier mode context.
```
Total Chargeable Weight = 
VAR SelectedModes = VALUES( Dim_Carriers[ModeOfTransport] )
RETURN
    IF(
        "Air" IN SelectedModes,
        SUMX(
            Fact_Air_Shipments,
            MAX( Fact_Air_Shipments[ActualWeightKG], Fact_Air_Shipments[VolumeCBM] * 167 )
        ),
        BLANK()
    )
```

---

### 5. Total Air Revenue
**Display Folder**: `Air\Financial`  
**Format**: Currency (R)  
**Description**: Total client-billed revenue from air freight, calculated as chargeable weight × client sell rate per shipment.
```
Total Air Revenue = 
SUMX(
    Fact_Air_Shipments,
    VAR ChargeableWeight = 
        MAX( Fact_Air_Shipments[ActualWeightKG], Fact_Air_Shipments[VolumeCBM] * 167 )
    RETURN
        ChargeableWeight * Fact_Air_Shipments[ClientSellRatePerKG]
)
```

---

### 6. Total Air Cost
**Display Folder**: `Air\Financial`  
**Format**: Currency (R)  
**Description**: Total airline procurement cost, calculated as chargeable weight × airline buy rate per shipment.
```
Total Air Cost = 
SUMX(
    Fact_Air_Shipments,
    VAR ChargeableWeight = 
        MAX( Fact_Air_Shipments[ActualWeightKG], Fact_Air_Shipments[VolumeCBM] * 167 )
    RETURN
        ChargeableWeight * Fact_Air_Shipments[AirlineBuyRatePerKG]
)
```

---

### 7. Air Gross Profit
**Display Folder**: `Air\Financial`  
**Format**: Currency (R)  
**Description**: Gross profit from air freight operations (Revenue minus Cost).
```
Air Gross Profit = 
[Total Air Revenue] - [Total Air Cost]
```

---

### 8. Air Gross Profit Margin %
**Display Folder**: `Air\Performance`  
**Format**: Percentage `0.0%`  
**Description**: Gross profit margin on air freight — the percentage of revenue retained after airline procurement costs. Measures the effectiveness of buy/sell rate negotiation.
```
Air Gross Profit Margin % = 
DIVIDE(
    [Total Air Revenue] - [Total Air Cost],
    [Total Air Revenue],
    0
)
```

---

### 9. Avg Buy Rate Per KG
**Display Folder**: `Air\Pricing`  
**Format**: Currency (R) `"R"#,0.00`  
**Description**: Average airline procurement rate per kilogram across all air shipments.
```
Avg Buy Rate Per KG = 
AVERAGE( Fact_Air_Shipments[AirlineBuyRatePerKG] )
```

---

### 10. Avg Sell Rate Per KG
**Display Folder**: `Air\Pricing`  
**Format**: Currency (R) `"R"#,0.00`  
**Description**: Average client-billed rate per kilogram across all air shipments.
```
Avg Sell Rate Per KG = 
AVERAGE( Fact_Air_Shipments[ClientSellRatePerKG] )
```

---
---

## 🚛 CROSS-BORDER / ROAD FREIGHT MEASURES
**Target Table**: `_Measures - Border` (or `Measures (2)` with Display Folder = `Border`)

---

### 1. Border Shipment Count
**Display Folder**: `Border\Volume`  
**Format**: `#,0` (whole number)  
**Description**: Total count of cross-border road freight shipment transactions in the current filter context.
```
Border Shipment Count = 
COUNTROWS( Fact_Border_Shipments )
```

---

### 2. On-Time Delivery %
**Display Folder**: `Border\Performance`  
**Format**: Percentage `0.0%`  
**Description**: Percentage of shipments delivered within the target transit time plus a 30-minute (0.5 hour) schedule buffer, dynamically respecting the carrier mode context.
```
On-Time Delivery % = 
VAR SelectedModes = VALUES( Dim_Carriers[ModeOfTransport] )
RETURN
    IF(
        "Road" IN SelectedModes,
        VAR TotalShipments = COUNTROWS( Fact_Border_Shipments )
        VAR OnTimeShipments = 
            CALCULATE(
                COUNTROWS( Fact_Border_Shipments ),
                Fact_Border_Shipments[TotalTransitHours] <= ( Fact_Border_Shipments[TargetTransitHours] + 0.5 )
            )
        RETURN
            DIVIDE( OnTimeShipments, TotalShipments, 0 ),
        BLANK()
    )
```

---

### 3. OTD vs Target Gap
**Display Folder**: `Border\Performance`  
**Format**: Percentage `+0.0%;-0.0%` (show sign)  
**Description**: Variance between actual On-Time Delivery % and the corporate 95% OTD target. Positive = above target, Negative = below target.
```
OTD vs Target Gap = 
[On-Time Delivery %] - 0.95
```

---

### 4. Avg Border Dwell Hours
**Display Folder**: `Border\Operational`  
**Format**: `0.0` with suffix ` hrs`  
**Description**: Average time spent at border posts/customs clearance. Isolates the customs processing bottleneck from line-haul driving time.
```
Avg Border Dwell Hours = 
AVERAGE( Fact_Border_Shipments[BorderDwellHours] )
```

---

### 5. Avg Line Haul Hours
**Display Folder**: `Border\Operational`  
**Format**: `0.0` with suffix ` hrs`  
**Description**: Average pure driving/line-haul time excluding border and road delays.
```
Avg Line Haul Hours = 
AVERAGE( Fact_Border_Shipments[LineHaulDrivingHours] )
```

---

### 6. Avg Total Transit Hours
**Display Folder**: `Border\Operational`  
**Format**: `0.0` with suffix ` hrs`  
**Description**: Average total door-to-door transit time including driving, border dwell, and road delays.
```
Avg Total Transit Hours = 
AVERAGE( Fact_Border_Shipments[TotalTransitHours] )
```

---

### 7. Transit Time Variance (Hours)
**Display Folder**: `Border\Performance`  
**Format**: `+0.0;-0.0` (show sign) with suffix ` hrs`  
**Description**: Average variance between actual total transit and target transit time. Positive = slower than target (delays), Negative = faster than target.
```
Transit Time Variance (Hours) = 
AVERAGEX(
    Fact_Border_Shipments,
    Fact_Border_Shipments[TotalTransitHours] - Fact_Border_Shipments[TargetTransitHours]
)
```

---

### 8. Transit Time Variance %
**Display Folder**: `Border\Performance`  
**Format**: Percentage `+0.0%;-0.0%`  
**Description**: Average transit time variance expressed as a percentage of target transit hours. Isolates the proportional impact of delays relative to planned journey times.
```
Transit Time Variance % = 
AVERAGEX(
    Fact_Border_Shipments,
    DIVIDE(
        Fact_Border_Shipments[TotalTransitHours] - Fact_Border_Shipments[TargetTransitHours],
        Fact_Border_Shipments[TargetTransitHours],
        0
    )
)
```

---

### 9. Late Delivery Count
**Display Folder**: `Border\Volume`  
**Format**: `#,0` (whole number)  
**Description**: Count of shipments that exceeded the target transit time plus the 30-minute buffer — i.e., shipments that failed the OTD threshold.
```
Late Delivery Count = 
CALCULATE(
    COUNTROWS( Fact_Border_Shipments ),
    Fact_Border_Shipments[TotalTransitHours] > ( Fact_Border_Shipments[TargetTransitHours] + 0.5 )
)
```

---

### 10. Border Delay Rate %
**Display Folder**: `Border\Performance`  
**Format**: Percentage `0.0%`  
**Description**: Proportion of total transit time attributable to border dwell. High values indicate customs/clearance is the primary bottleneck (particularly at Beitbridge Border Post).
```
Border Delay Rate % = 
DIVIDE(
    AVERAGE( Fact_Border_Shipments[BorderDwellHours] ),
    AVERAGE( Fact_Border_Shipments[TotalTransitHours] ),
    0
)
```

---
---

## 📊 CROSS-MODAL SUMMARY MEASURES
**Target Table**: `_Measures - Summary` (or `Measures (2)` with Display Folder = `Summary`)

---

### 1. Total Shipments (All Modes)
**Display Folder**: `Summary\Volume`  
**Format**: `#,0` (whole number)  
**Description**: Combined shipment count across all three transport modes (Ocean + Air + Border), dynamically respecting the carrier transport mode filter context.
```
Total Shipments (All Modes) = 
VAR SelectedModes = VALUES( Dim_Carriers[ModeOfTransport] )
VAR OceanCount = IF( "Ocean" IN SelectedModes, COUNTROWS( Fact_Ocean_Shipments ), 0 )
VAR AirCount = IF( "Air" IN SelectedModes, COUNTROWS( Fact_Air_Shipments ), 0 )
VAR RoadCount = IF( "Road" IN SelectedModes, COUNTROWS( Fact_Border_Shipments ), 0 )
RETURN
    OceanCount + AirCount + RoadCount
```

---

### 2. Total Logistics Cost
**Display Folder**: `Summary\Financial`  
**Format**: Currency (R)  
**Description**: Total logistics cost exposure across all modes — combines ocean D&D penalties with air freight procurement costs, dynamically respecting the carrier transport mode filter context.
```
Total Logistics Cost = 
VAR SelectedModes = VALUES( Dim_Carriers[ModeOfTransport] )
VAR OceanCost = IF( "Ocean" IN SelectedModes, [Total D&D Cost], 0 )
VAR AirCost = IF( "Air" IN SelectedModes, [Total Air Cost], 0 )
RETURN
    OceanCost + AirCost
```

---

### 3. Total Logistics Revenue
**Display Folder**: `Summary\Financial`  
**Format**: Currency (R)  
**Description**: Total revenue from all revenue-bearing modes. Currently only Air Freight generates direct buy/sell revenue in the data model.
```
Total Logistics Revenue = 
VAR SelectedModes = VALUES( Dim_Carriers[ModeOfTransport] )
VAR AirRevenue = IF( "Air" IN SelectedModes, [Total Air Revenue], 0 )
RETURN
    AirRevenue
```

---

### 4. Overall Gross Margin %
**Display Folder**: `Summary\Performance`  
**Format**: Percentage `0.0%`  
**Description**: Overall gross margin percentage across the logistics operation (Total logistics revenue minus all costs, divided by revenue).
```
Overall Gross Margin % = 
DIVIDE(
    [Total Logistics Revenue] - [Total Logistics Cost],
    [Total Logistics Revenue],
    0
)
```

---

### 5. Rolling 3-Month Demurrage
**Display Folder**: `Summary\Time Intelligence`  
**Format**: Currency (R)  
**Description**: Rolling 3-month total demurrage cost. Uses DATESINPERIOD for a trailing-window calculation anchored to the last date in the current filter context. Smooths out monthly spikes and reveals trends.

> ⚠️ **Note**: This measure requires an **active relationship** between `Fact_Ocean_Shipments[VesselArrivalDateKey]` and `Dim_Dates[DateKey]` in the current context (or use `USERELATIONSHIP` if the active relationship is on a different date key).

```
Rolling 3-Month Demurrage = 
CALCULATE(
    [Total Demurrage Cost],
    DATESINPERIOD(
        Dim_Dates[CalendarDate],
        MAX( Dim_Dates[CalendarDate] ),
        -3,
        MONTH
    )
)
```

---

### 6. YoY Demurrage Delta
**Display Folder**: `Summary\Time Intelligence`  
**Format**: Currency (R) with sign `+"R"#,0.00;-"R"#,0.00`  
**Description**: Year-over-Year change in demurrage costs. Positive = costs increased vs prior year, Negative = costs decreased. Uses SAMEPERIODLASTYEAR for accurate time-intelligence comparison.

> ⚠️ **Note**: Requires `Dim_Dates[CalendarDate]` to be marked as a **Date table** for time intelligence functions to work correctly. If not yet done, right-click the `Dim_Dates` table → **Mark as Date Table** → select `CalendarDate`.

```
YoY Demurrage Delta = 
VAR CurrentPeriod = [Total Demurrage Cost]
VAR PriorYear = 
    CALCULATE(
        [Total Demurrage Cost],
        SAMEPERIODLASTYEAR( Dim_Dates[CalendarDate] )
    )
RETURN
    CurrentPeriod - PriorYear
```

## ⚙️ PREREQUISITES

Before using the Time Intelligence measures (#5 and #6 in Summary), ensure:

1. **Mark Dim_Dates as Date Table**:  
   Right-click `Dim_Dates` → **Mark as Date Table** → select `CalendarDate` as the date column.

2. **Verify active relationships**:  
   The active relationship from `Fact_Ocean_Shipments[VesselArrivalDateKey]` → `Dim_Dates[DateKey]` must be active for the Rolling and YoY measures to work on Ocean data.
