# Role and Context
You are acting as an expert BI Consultant, Enterprise Data Architect, and Prompt Engineer with deep domain expertise in third-party logistics (3PL), multi-modal global freight forwarding (Air & Ocean), and South African supply chain network optimization. 

The goal is to design a comprehensive portfolio demonstration project engineered to secure a Power BI Freelancer role at Rhenus Air & Ocean (South African Division). The technical evaluation evaluates proficiency in processing messy raw logistics data, architecting high-performance Star Schemas, writing complex DAX/M expressions, integrating external databases via Microsoft Fabric/SQL Server, and building executive-ready dashboards.

# Execution Constraints
- DO NOT execute visual mockups or build BI reports. 
- PROVIDE only the highly structured, actionable blueprints and the complete, self-contained T-SQL deployment script.
- Ensure all outputs strictly mirror the operations, regions, and naming conventions relevant to a global logistics provider operating out of South Africa.

---

# Phase 1: Project Ideas & Blueprinting
Brainstorm exactly 3 distinct, high-impact Power BI dashboard concepts that address complex supply chain visibility and performance bottlenecks. One dashboard must explicitly cover the **Warehousing Solutions Division**, and the remaining two must cover the **Multi-Modal Forwarding Division**.

For each of the 3 concepts, you must use the following exact structural headers:

### 1. Project Title & Persona
- Define the operational scenario (e.g., South African Cross-Border Distribution & Warehouse Efficiency, Global Ocean Freight Congestion & Capacity Optimizer, Air Cargo Transit Performance).
- Specify the exact executive or operational user persona it serves (e.g., Regional Logistics Director, Global Ocean Freight Procurement Manager, Warehouse Operations Head).

### 2. Core Business Value
- Detail how this dashboard transforms raw transactional logistics data into prescriptive insights. Explain the cost-saving levers (e.g., minimizing demurrage, optimizing container load factors, identifying low-productivity labor shifts).

### 3. Key Metrics & DAX Logic
Detail at least 4 advanced business metrics per dashboard. For each metric, provide a clear business definition and write out the logical or pseudo-DAX expression to calculate it. You must explicitly target the following division-specific KPIs within your 3 concepts:
- **Warehousing Solutions Division KPIs**: Cost per unit, Spatial utilization (volumetric/pallet positions), Throughput (Inbound/Outbound lines handled), and Labour productivity.
- **Multi-Modal Forwarding Division KPIs**: Cargo transit variance (Actual vs. Estimated Time of Arrival), Route fuel consumption/CO2 footprint estimation, and Carrier capacity utilization.
- *Technical requirement*: Include at least one complex DAX pattern involving time intelligence (e.g., warehouse inventory aging buckets) and one involving dynamic operational threshold logic (e.g., rolling averages or tiered Demurrage & Detention charge scaling) that account for regional delays.

### 4. Dimensional Modeling Layout
- Define the Star Schema required to build a performant DirectLake or Import model.
- Map out the primary Fact table (e.g., `Fact_Shipment_Transactions`, `Fact_Warehouse_Movement`) and all supporting Dimension tables (e.g., `Dim_Date`, `Dim_Location_Hubs`, `Dim_Carriers`, `Dim_Equipment_Containers`). Explicitly state the grain of the Fact table.

---

# Phase 2: Complete T-SQL Data Generation Script
Generate a comprehensive, production-grade T-SQL script optimized for Microsoft SQL Server and fully compatible with Microsoft Fabric (via Data Factories or Data Warehouse shortcuts). 

The script must be entirely self-contained and ready for out-of-the-box execution in SQL Server Management Studio (SSMS). It must fulfill the following technical parameters:

### 1. DDL Setup & Relational Integrity
- Include clean `CREATE DATABASE`, `CREATE SCHEMA` (e.g., `whse`, `fwd`), and `CREATE TABLE` statements.
- Use explicit and optimized data types (`DATETIME`, `DECIMAL(18,4)` for monetary/weight elements, `INT`, `VARCHAR`).
- Define strict relational integrity with explicit `PRIMARY KEY` and `FOREIGN KEY` constraints reflecting the Star Schema outlined in Phase 1.

### 2. Reference & Dimension Data Master Population
- Populate the dimensional lookup tables with realistic, high-fidelity South African and international logistics data. Do not use generic placeholders like 'Carrier A' or 'Location 1'.
- Include transport modes: `Air Freight`, `Ocean - FCL`, `Ocean - LCL`, and `Road Cross-Border`.
- Include precise global transport hubs and South African gateways (e.g., Port of Durban, Johannesburg OR Tambo International Airport, Port of Cape Town, City Deep Inland Container Terminal, Beitbridge Border Post, Port of Rotterdam, Port of Shanghai).
- Include distinct carrier types and vessel/aircraft capacity classifications.

### 3. Transactional Fact Data & Localized South African Anomalies
- Generate a minimum of 1,000 transactional rows distributed across the last 12 to 24 months using advanced T-SQL generation structures (`WHILE` loops, Common Table Expressions (CTEs), or system cross-joins).
- **Injected Localized Anomalies**: The generated transaction history must intentionally bake in real-world South African operational bottlenecks to test the analytics resilience of the Power BI model. Programmatically inject:
  - **Port Congestion Backlogs**: Substantial delays between scheduled arrival and actual berthing/offloading at the Port of Durban during peak shipping months, triggering cascading Demurrage & Detention variance.
  - **Border Post Bottlenecks**: Multi-day transit time spikes exclusively affecting the `Road Cross-Border` mode at the Beitbridge Border Post, impacting transit variance metrics.
  - **Utility Interruptions**: Periodic, localized inventory processing drops and labor productivity drops in the `whse` transactions to simulate the historical operational impacts of severe load-shedding or municipal water interruptions at Gauteng and Durban fulfillment hubs.

### 4. Logical & Economic Data Consistency
- **Temporal Logic**: Ensure all milestone timestamps are strictly sequential (`BookingDate` < `CargoReceivedDate` < `DepartureDate` < `ArrivalDate` < `DeliveryDate` < `InvoicingDate`), even when expanded by the injected regional anomalies.
- **Physical & Economic Scaling**: Ensure physical metrics (Weight, Volume) and monetary metrics (Freight Charges, Fuel Surcharges, Handling Fees) match real-world economic realities. Air freight must exhibit significantly higher per-kilogram costs and faster transit times than Ocean freight. Cross-Border road freight must scale logically by distance from Johannesburg to neighboring SADC hubs.